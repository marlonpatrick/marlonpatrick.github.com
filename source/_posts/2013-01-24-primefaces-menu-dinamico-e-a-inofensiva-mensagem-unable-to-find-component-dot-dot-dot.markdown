---
layout: post
title: "PrimeFaces:Menu dinâmico e a inofensiva mensagem Unable to find component..."
date: 2013-01-24 12:00
comments: true
categories: [JSF]
tags: [JSF, PrimeFaces, Dynamic Menu, Menu Dinâmico, MenuBar] 
---

Eu sou particularmente fã incondicional do PrimeFaces e um dos componentes que mais gosto é o de gerar menus dinâmicos. Praticamente todas as aplicações tem o conceito de perfil de acesso o que acaba tornando o componente MenuBar extremamente útil para esse tipo de requisito. Basicamente, o sistema só irá renderizar os menus o qual apontam para telas que o usuário pode acessar.

Bem, estou usando esse componente exatamente no cenário descrito acima com a versão 3.4.1 do PrimeFaces e a implementação de referência do JSF(Mojarra) na sua versão 2.1.7. Com essa combinação (PrimeFaces/Mojarra) nas versões descritas acima (acredito que em outras também) percebi no log de minha aplicação que todas as vezes que executava uma chamada ajax, para cada item do menu gerado com o MenuBar dinâmico, havia uma mensagem mais ou menos assim:<strong>Unable to find component with clientId 'menuitem200', no need to remove it.</strong>.

Até um determinado momento em nada me afetava tal mensagem, e provavelmente é assim para 99% dos casos. Agora, se você assim como eu, caiu no 1% restante, essa mensagem pode causar um sério problema de performance nas chamadas ajax. Esse problema de performance só irá acontecer se você tiver uma página muito grande, com muitos componentes gerados como foi o meu caso.

Toda vez que eu fazia uma chamada ajax nessa página o procedimento executado pelo PrimeFaces que gerava as mensagens "Unable to find component..." chegava a levar cerca de 1 minuto e 30 segundos para terminar, o que é totalmente inviável. Não é que o código executado na chamada era super pesado, esses mais de 1 minuto se referem apenas execução do método que gerava as mensagens como mostra o log abaixo:

        02:40:00,876 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'submenu1', no need to remove it.
        02:40:01,298 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'submenu2', no need to remove it.
        02:40:01,656 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'menuitem45', no need to remove it.
        02:40:02,014 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'menuitem4', no need to remove it.
        02:40:02,361 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'submenu20', no need to remove it.

        ...muitas outras mensagens aqui...

        02:40:44,240 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'menuitemespacobefore', no need to remove it.
        02:40:44,917 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'submenuuseractions', no need to remove it.
        02:40:45,249 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'menuitemchangepassword', no need to remove it.
        02:40:45,574 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'menuitemlogout', no need to remove it.

No exemplo acima, o tempo total foi de 46 segundos. Basicamente o que acontece é que na fase JSF Restore View é executado o método que restaura o estado dos menus dinâmicos, onde, para cada item do menu é feito uma busca em toda a árvore de componentes do JSF e devido a página ser grande leva um tempo considerável. O pior é que, como a própria mensagem diz, "no need to remove it", não é necessário remover tal componente da árvore JSF porque na verdade ele já foi removido.

No meu caso, o que percebi é que quando o item do menu dinâmico usa um link não é necessário essa remoção que o PrimeFaces tenta fazer, já quando o item do menu usa uma Action Expression, aí sim, esse procedimento é necessário.

                //Não é necessário tentar remover esse MenuItem, pois, ele usa um link
                MenuItem changePassword = new MenuItem();
                changePassword.setId("menuitemchangepassword");
                changePassword.setValue("Mudar Senha");
                changePassword.setUrl("/user/changePassword.xhtml?faces-redirect=true");
                submenuUser.getChildren().add(changePassword);

                //De fato é preciso remover o menu item para adicionar o novo objeto na fase de Restore View, pois, ele usa uma Action Expression
                MethodExpression logoutAction = factory.createMethodExpression(context.getELContext(), "#{loginView.logout}",
                                null, new Class[] {});
                MenuItem logout = new MenuItem();
                logout.setId("menuitemlogout");
                logout.setValue("Sair");
                logout.setActionExpression(logoutAction);
                submenuUser.getChildren().add(logout);

Pra resolver esse problema criei um PhaseListener que retira os itens de menu que usam link dessa fase de restauração desnecessária, deixando apenas os que usam action expression.

	public class RestoreDynamicActionsObserver implements PhaseListener{

		public static final String DYNAMIC_ACTIONS_RENDERKIT = "HTML_BASIC";

		@Override
		public PhaseId getPhaseId() {
			return PhaseId.RESTORE_VIEW;
		}

		@Override
		public void beforePhase(PhaseEvent event) {

			ResponseStateManager rsm = RenderKitUtils.getResponseStateManager(event.getFacesContext(), DYNAMIC_ACTIONS_RENDERKIT);

			HttpServletRequest httpRequest = WebUtils.getRequest();
		
			Object[] rawState = (Object[]) rsm.getState(event.getFacesContext(), httpRequest.getRequestURI().replaceFirst(httpRequest.getContextPath(), "").split("\\?")[0]);
			if (rawState == null) {
				return;
			}
	    
			Map<String, Object> state = (Map<String,Object>) rawState[1];
		
			if(state == null){
				return;
			}
		
			List<Object> savedActions = (List<Object>) state.get(RIConstants.DYNAMIC_ACTIONS);

			if(savedActions == null){
				return;
			}
		
			for (Iterator<Object> iterator = savedActions.iterator(); iterator.hasNext();) {
				Object object = iterator.next();
			    ComponentStruct action = new ComponentStruct();
			    action.restoreState(event.getFacesContext(), object); 
		    
			    //Esses são os únicos itens que usam Action Expression, e portanto, não devem ser mexidos
			    if(ComponentStruct.ADD.equals(action.action) && 
		    		(action.clientId.equals("menuitemlogout") || action.clientId.equals("submenuuseractions"))){
			    	continue;
			    }

			    //Os demais itens usam link, e por isso, são retirados dessa coleção
			    if (action.clientId.startsWith("menuitem") 
		    		|| action.clientId.startsWith("submenu")) {            	
			    	iterator.remove();
			    }			
			}

		}
	
		@Override
		public void afterPhase(PhaseEvent event) {
			//
		}
	}


Com o PhaseListener acima, eu deixo apenas os itens de menu que usam Action Expression na coleção de ações dinâmicas a restaurar. No meu caso, tenho um menu com centenas de itens, dos quais, apenas 2 ou 3 usam action expression e com isso o tempo gasto na restauração das ações dinâmicas caiu bruscamente com  esse ajuste, ficando imperceptível(como deve ser). Isso acontece porque o PhaseListener executa antes da restauração deixando apenas o que realmente é necessário. Ah, outra coisa, as mensagens "Unable to find component..." também devem sumir do seu log. É isso, vlw!



<strong>Provérbios 19.23:</strong>O temor do SENHOR encaminha para a vida; aquele que o tem ficará satisfeito, e não o visitará mal nenhum.
