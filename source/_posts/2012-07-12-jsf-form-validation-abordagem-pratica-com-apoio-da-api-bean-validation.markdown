---
layout: post
title: "JSF Form Validation:Abordagem prática com apoio da API Bean Validation + Seam Faces"
date: 2012-07-12 19:08
comments: true
categories: [JSF, JBoss Seam]
tags: [JSF, JBoss Seam, JSF Form Validation, Bean Validation, JSR 303, JSF Validation, Seam Faces] 
---

Bem, se você já trabalha com JSF a algum tempo sabe que criar telas com fluxos um pouco mais complexos usando as validações built-in ou até mesmo Bean Validation sabe que isso pode se tornar um tanto complicado. Basicamente o problema é que muitas vezes as validações são acionadas em momentos inoportunos, momentos em que não é necessário a tal validação, mas, isso acontece devido ao tão mencionado ciclo de vida JSF.

Um exemplo bem simples pra quem usa Primefaces é ao se trabalhar com abas, onde, cada vez que você muda de aba as validações são chamadas pelo JSF, o que em muitos casos é desnecessário. Para saber mais: http://code.google.com/p/primefaces/issues/detail?id=3423.

Sim, eu sei que dá pra contornar a maioria desses problemas com o processamento parcial (atributo execute nas tags padrões e process nas tags Primefaces), mas, a verdade é que o código começa a ficar complexo e em alguns casos não tem jeito mesmo e muita gente acaba abidicando dessas validações e colocam as validações no próprio back bean.

Pois bem, esses dias estava fazendo uma dessas telas e acabei me deparando com esses problemas. Deixar o código das páginas complexo não é uma boa opção para esse trabalho e ficar fazendo as validações diretamente nos back beans também não me pareceu a solução mais elegante(apesar de funcionar bem). Nos tempos de Struts existia o conceito de validação de form o que não é natural no JSF, pois, os validadores são determinados componente a componente. Mas aí, encontrei essa funcionalidade no Seam Faces, com a tag <s:validateForm />. Com essa tag do Seam Faces podemos definir um JSF Validator para o form e não apenas para um campo específico e esse validador é acionado assim que o form é submetido.

Obs: Qualquer dificuldade de adicionar o Seam Faces no seu projeto leia este outro post [Adicionando Seam Faces No Seu Projeto Maven](/blog/2012/06/14/adicionando-seam-faces-no-seu-projeto-maven/).

Com isso, já temos uma maneira de validar o form apenas no momento que o mesmo for submetido, mas, ainda teria de fazer as validações manualmente para cada campo, foi aí que me veio a idéia de usar a api Bean Validation para fazer esse trabalho para mim. A idéia é simples, eu coloco anotações Bean Validation no meu modelo, desabilito as validações do ciclo de vida JSF com a tag <f:validateBean /> (JSF 2 e Bean Validation são integradas e as validações são ativadas por padrão) e valido o modelo no meu form validator chamando as validações Bean Validation programaticamente. Dessa forma meu form é validado com a api Bean Validation no momento exato que me interessa, ou seja, no submit.

O código para isso segue abaixo:

	import java.util.ArrayList;
	import java.util.LinkedHashSet;
	import java.util.Set;

	import javax.faces.application.FacesMessage;
	import javax.faces.component.UIComponent;
	import javax.faces.component.UIInput;
	import javax.faces.context.FacesContext;
	import javax.faces.validator.BeanValidator;
	import javax.faces.validator.FacesValidator;
	import javax.faces.validator.Validator;
	import javax.faces.validator.ValidatorException;

	import org.primefaces.component.tabview.Tab;

	@FacesValidator("formBeanValidator")
	public class FormBeanValidatorTeste implements Validator {


		protected Set<FacesMessage> allMessages;

		@Override
		public void validate(FacesContext context, UIComponent component, Object object) throws ValidatorException {

			ArrayList<FacesMessage> messages = new ArrayList<FacesMessage>(validateInputs(context, component,
					new BeanValidator()));

			if (!messages.isEmpty()) {
				if (isThrowsValidatorException()) {
					throw new ValidatorException(messages);
				}

				//As mensagens de erro ficam disponíveis para uma possível subclasse fazer uso
				this.allMessages = new LinkedHashSet<FacesMessage>(messages);
			}
		}

		/**
		 * Varre todos os campos do form e caso seja um 
		 * UIInput chama o BeanValidator padrão do JSF. 
		 *
		 *@author marlonpatrick10
		 */
		private Set<FacesMessage> validateInputs(FacesContext context, UIComponent rootComponent,
				BeanValidator beanValidator) {

			Set<FacesMessage> messages = new LinkedHashSet<FacesMessage>();

			for (UIComponent children : rootComponent.getChildren()) {
				if (children == rootComponent) {
					continue;
				}

				if (!children.isRendered()) {
					continue;
				}

				if (Tab.class.isAssignableFrom(children.getClass())) {//Primefaces
					Tab tab = (Tab)children;
					if (tab.isDisabled()) {
						continue;
					}
				}

				if (UIInput.class.isAssignableFrom(children.getClass())) {
					if (isValidatable(context, (UIInput)children)) {
						validate(context, beanValidator, (UIInput)children, messages);
					}
				}

				messages.addAll(validateInputs(context, children, beanValidator));
			}

			return messages;
		}

		/**
		 * Chama o BeanValidator padrão do JSF
		 * para um determinado UIInput. 
		 *
		 *@author marlonpatrick10
		 */
		protected void validate(FacesContext context, BeanValidator beanValidator, UIInput component,
				Set<FacesMessage> messages) {
			try {
				beanValidator.validate(context, component, component.getValue());
			} catch (ValidatorException exception) {
				if (exception.getFacesMessages() == null) {
					messages.add(exception.getFacesMessage());
				} else {
					messages.addAll(exception.getFacesMessages());
				}
			}
		}

		/**
		 * O comportamento padrão é validar todos os UIInput,
		 * mas, pode ser que não seja necessário validar 
		 * um determinado componente numa tela.
		 * 
		 * Assim, esse validator pode ser extendido e o 
		 * método usado para identificar se o componente 
		 * deve ser ou não validado.
		 *
		 *@author marlonpatrick10
		 */
		protected Boolean isValidatable(FacesContext context, UIInput input) {
			return true;
		}

		/**
		 * O comportamento padrão é validar todos os UIInput,
		 * e lançar uma ValidatorException caso haja erros.
		 * Esse validator pode ser extendido e o 
		 * método sobrescrito para não lançar a exceção,
		 * assim o novo validator pode fazer validações extras.
		 * 
		 * As mensagens de erro do Bean Validation
		 * ficam disponíveis no atributo this.allMessages.
		 *
		 *@author marlonpatrick10
		 */
		protected Boolean isThrowsValidatorException() {
			return true;
		}
	}

O FormBeanValidator pode servir para a maioria das telas, mas, ainda sim há casos que queremos especializar mais as validações. Abaixo segue um exemplo:

	import java.util.Set;

	import javax.faces.application.FacesMessage;
	import javax.faces.component.UIInput;
	import javax.faces.context.FacesContext;
	import javax.faces.validator.BeanValidator;
	import javax.faces.validator.FacesValidator;
	import javax.inject.Inject;

	import GenericUtils;
	import FormBeanValidator;

	@FacesValidator("userFormBeanValidator")
	public class UserFormBeanValidator extends FormBeanValidator {

		@Inject
		UserView userView;//back bean
	
		@Override
		protected Boolean isValidatable(FacesContext context, UIInput input) {
			//No momento do submit da tela de cadastro de usuário 
			//não é preciso validar os campos abaixo
		
			if (input.getClientId().endsWith("fieldX")) {
				return false;
			}

			if (input.getClientId().endsWith("fieldY")) {
				return false;
			}

			return true;
		}

		@Override
		protected void validate(FacesContext context, BeanValidator beanValidator, UIInput component,
				Set<FacesMessage> messages) {

			//email2 não pode ser anotado com @NotNull, pois, sua obrigatoriedade é condicional
			if (this.userView.isAdmin()) {
				if (component.getClientId().endsWith("email2")) {
					if (GenericUtils.isBlankOrNull((String)component.getValue())) {
						this.allMessages.add(new FacesMessage(FacesMessage.SEVERITY_ERROR,
								"Usuário admin deve informar 2 emails.",null));
					}
				}
			}

			//chama Bean Validation mesmo para email2, pois, 
			//User.email2 está anotado com @Email, então, 
			//ainda precisa ser validado
			super.validate(context, beanValidator, component, messages);
		}
	}

Na página(xhtml) deve ficar algo como segue abaixo:

	<html xmlns="http://www.w3.org/1999/xhtml"
		xmlns:h="http://java.sun.com/jsf/html"
		xmlns:f="http://java.sun.com/jsf/core"
		xmlns:s="http://jboss.org/seam/faces">

		<h:form id="formInsertUser" prependId="false">
			<f:validateBean disabled="true">
				<!-- Os campos anotados com Bean Validation devem ficar aqui -->
			</f:validateBean>

			<h:commandLink value="Salvar" action="#{userView.insert}" />
	
			<!-- O validator será chamado no click do commandLink -->
			<s:validateForm validatorId="userFormBeanValidator" />
		</h:form>

	</html>

Essa validação de form aliada ao Bean Validation vai me ajudar muito a construir páginas mais complexas com JSF sem ter tanta dor de cabeça com validações fora de hora, espero que ajude você também. Vlw!

<strong>1 João 5.14:</strong>E esta é a confiança que temos nele (Jesus), que se pedirmos alguma coisa segundo a sua vontade, ele nos ouve.
