---
layout: post
title: "PrimeFaces: contornando problemas com dataTable resizable/draggable"
date: 2014-04-26 16:29
comments: true
categories: [JSF,PrimeFaces] 
tags: [JSF, PrimeFaces, DataTable]
---

Vou deixar aqui soluções alternativas para alguns bugs do PrimeFaces quando se usa dataTable com as features resizable ou draggable. Alguns desses bugs já foram corrigidos, mas, se por algum motivo você assim como eu trabalha com alguma versão do PrimeFaces que contém o erro e não pode atualizá-la será de grande ajuda.

**1 - NullPointerException ao redimensionar o tamanho de uma coluna (corrigido na versão 3.5)**

O problema acontece num dataTable que é resizable e draggable ao mesmo tempo. Encontrei os seguintes tickets no issue tracker do PrimeFaces para tratar do problema: <a href="https://code.google.com/p/primefaces/issues/detail?id=3674" title="Issue 3674" target="_blank">3674</a>, <a href="https://code.google.com/p/primefaces/issues/detail?id=4012" title="Issue 4012" target="_blank">4012</a>,<a href="https://code.google.com/p/primefaces/issues/detail?id=4796" title="Issue 4796" target="_blank">4796</a> e <a href="https://code.google.com/p/primefaces/issues/detail?id=5167" title="Issue 5167" target="_blank">5167</a>. Na versao 3.4.1 do PrimeFaces o log do erro é algo como:

	01:40:49,907 INFO  [javax.enterprise.resource.webcontainer.jsf.context] (http--0.0.0.0-8080-1) java.lang.NullPointerException: java.lang.NullPointerException
	at org.primefaces.component.datatable.feature.ResizableColumnsFeature.decode(ResizableColumnsFeature.java:35) [primefaces-3.4.1.jar:]
	at org.primefaces.component.datatable.DataTableRenderer.decode(DataTableRenderer.java:53) [primefaces-3.4.1.jar:]
	at javax.faces.component.UIComponentBase.decode(UIComponentBase.java:787) [jboss-jsf-api_2.1_spec-2.0.1.Final.jar:2.0.1.Final]
	at org.primefaces.component.api.UIData.processDecodes(UIData.java:224) [primefaces-3.4.1.jar:]
	at com.sun.faces.context.PartialViewContextImpl$PhaseAwareVisitCallback.visit(PartialViewContextImpl.java:506) [jsf-impl-2.1.7-jbossorg-2.jar:]
	at com.sun.faces.component.visit.PartialVisitContext.invokeVisitCallback(PartialVisitContext.java:183) [jsf-impl-2.1.7-jbossorg-2.jar:]
	at org.primefaces.component.api.UIData.visitTree(UIData.java:635) [primefaces-3.4.1.jar:]

<!-- more -->

De maneira resumida o erro acontece porque ao redimensionar uma coluna o PrimeFaces processa tanto o redimensionamento (resize) quanto a reordenação da coluna (reorder). Isso pode ser facilmente percebido se você usar os eventos colResize e colReorder para gerar algum log.

A explicação detalhada é a seguinte: para criar o efeito de redimensionar uma coluna o PrimeFaces cria uma tag `<span>` no cabeçalho da coluna e torna essa tag um objeto <a href="http://jqueryui.com/draggable/" title="jQuery UI - Draggable" target="_blank">draggable</a>, dessa forma, quando o usuário arrasta a "coluna" para ela ficar maior ou menor e a solta ocorre uma "colisão" entre a funcionalidade resizer e reorder do dataTable. Isso porque, ao informar que o dataTable é draggable o PrimeFaces torna todas as colunas draggable de modo que as mesmas possam ser arrastadas e reordenadas bem como torna todo o cabeçalho da tabela <a href="http://jqueryui.com/droppable/" title="jQuery UI - Droppable" target="_blank">droppable</a>. O problema é que a implementação do PrimeFaces esqueceu de configurar o droppable para aceitar apenas reordenação de colunas e com isso ao soltar o `<span>` que redimensiona a coluna o drop da reordenação é invocado e o PrimeFaces entende que a coluna está sendo reordenada e redimensionada, com isso o método que deveria ajustar o tamanho da coluna não recebe um parâmetro (o id da coluna redimensionada) que precisa para funcionar corretamente e então provoca o erro.

Para solucionar esse problema adicionei um script após o fechamento da tag `<p:dataTable>` que define um escopo específico para o `<span>` que é usado no redimensionamento das colunas. Assim, ao redimensionar uma coluna mesmo que o usuário solte o mouse sobre o cabeçalho da tabela o drop da reordenação de colunas não será chamado, pois, o objeto draggable `<span>` possui um escopo diferente do objeto droppable (cabeçalho da tabela). O código fica da seguinte forma:

	<p:dataTable draggableColumns="true" resizableColumns="true">
	       <!-- your code -->
	</ p: dataTable>
	
	<script type="text/javascript">
	    $( ".ui-column-resizer" ).draggable( "option", "scope", 'resizer');//resizer é o nome do escopo que escolhi, poderia ser qualquer outro nome
	</script>

**2 - NullPointerException ao redimensionar uma coluna dinâmica (corrigido na versão 3.5.7)**

Nesse caso basta apenas ter um dataTable resizable com colunas dinâmicas para que o erro aconteça. Encontrei os seguintes tickets no issue tracker do PrimeFaces para tratar do problema: <a href="https://code.google.com/p/primefaces/issues/detail?id=4238" title="Issue 4238" target="_blank">4238</a> e <a href="https://code.google.com/p/primefaces/issues/detail?id=4797" title="Issue 4797" target="_blank">4797</a>. Engraçado que mesmo com várias pessoas relatando o problema o time do PrimeFaces marcou ambos issues como "Não foi possível replicar". Segundo um dos observadores o problema foi resolvido na versão 3.5.7, mas, não pude comprovar.

O ponto onde acontece o erro é o mesmo do caso descrito acima, assim, o log é semelhante e só o que muda é a causa raiz do problema. No item anterior, o motivo do NullPointer é que o parâmetro com o id da coluna redimensionada não é enviado para o servidor, já no caso das colunas dinâmicas, mesmo quando o id é enviado o PrimeFaces não consegue encontrar a coluna correspondente. O método que é usado para buscar a coluna pelo id é o `DataTable.findColumn(String clinetId)` o qual só procura por colunas simples e desconsidera as dinâmicas.

No meu caso, não era tão importante que o objeto Column fosse achado e que o seu campo width fosse setado pois o width usado na minha tela fica armazenado num outro objeto específico do meu sistema. Por isso minha solução foi apenas sobrescrever a classe que processa o evento resize do dataTable (org.primefaces.component.datatable.feature.ResizableColumnsFeature), mas especificamente o método decode:

	@Override
	public void decode(FacesContext context, DataTable table) {
		Map<String, String> params = FacesContext.getCurrentInstance().getExternalContext().getRequestParameterMap();
		String clientId = table.getClientId();

		String columnId = params.get(clientId + "_columnId");
		String width = params.get(clientId + "_width");
		Column column = table.findColumn(columnId);

		if(column != null){ // adicionei esse if para evitar a NullPointerException 
			column.setWidth(Integer.parseInt(width));
		}
	}
	
Dessa forma o evento colResize pode ser invocado e é através dele que resolvo meu problema. Talvez o código seja difícil de aplicar em outro sistema, mas, de toda forma vou deixar aqui pelo menos para você ter uma idéia de como fiz e poder adaptar a solução:

	public void onResizeColumn(ColumnResizeEvent event) {
		if (event.getColumn() == null) {
			String index = getColumnIndex(event);
			DynaColumn dynaColumn = this.getColumns().get(Integer.parseInt(index));//getColumns() é a lista que uso para gerar as colunas dinâmicas
			dynaColumn.setWidth(event.getWidth());//DynaColumn é um objeto específico do meu sistema
		}
	}

	private String getColumnIndex(ColumnResizeEvent event) {
		String columnId = FacesContext.getCurrentInstance().getExternalContext().getRequestParameterMap().get(event.getComponent().getClientId() + "_columnId");
		String[] columnIdSplited = columnId.split(":");
		return columnIdSplited[columnIdSplited.length - 1];
	}

Na minha página JSF uso o campo DynaColumn.width para setar o tamanho da coluna mesmo que o objeto Column do PrimeFaces não tenha sido atualizado.

	<p:dataTable resizableColumns="true">
	       <!-- your code -->
		<p:columns value="#{viewManager.columns}" var="dynaColumn">
		       <!-- your code -->
		</p:columns>
	</ p: dataTable>
	
	<ui:repeat var="dynaColumn" value="#{viewManager.columns}" varStatus="status">
	    <script type="text/javascript">
	    	var columnWidth = #{dynaColumn.width};
	    	if(columnWidth > 0){
				$('[id="idDoDataTable:dynamicColumns:#{status.index}"]').
	    			children('.ui-dt-c').css('width',columnWidth);
			}
		</script>				
	</ui:repeat>


A issue <a href="https://code.google.com/p/primefaces/issues/detail?id=4238" title="Issue 4238" target="_blank">4238</a> propõe uma outra forma de contornar o problema, além disso, outras soluções possíveis seriam ajustar o método ResizableColumnsFeature.decode para procurar a coluna da maneira certa ou então no onResize setar o objeto Column ao invés de um objeto próprio do sistema. Caso alguma dessas correções seja implementada não seria necessário o javascript para ajustar o width das colunas na página JSF o que simplificaria a solução. No meu caso específico há um requisito de negócio que me forçou a fazer da forma aqui apresentada.

**3 - Evento colReorder é executado para qualquer chamada ajax e não apenas na reordenação de colunas**

Para simular esse erro basta apenas ter um dataTable draggable e fazer qualquer chamada ajax na tela. Encontrei o ticket <a href="https://code.google.com/p/primefaces/issues/detail?id=4985" title="Issue 4985" target="_blank">4985</a> no issue tracker do PrimeFaces que aborda o problema. O mesmo foi marcado como "Não será corrigido", então, acredito que ainda aconteça em qualquer versão.

Para resolver o problema sobrescrevi a classe org.primefaces.component.datatable.feature.DraggableColumnsFeature para ajustar o método shouldDecode:

	@Override
	public boolean shouldDecode(FacesContext context, DataTable table) {
	  return table.isDraggableColumns() && this.isDragRequest(context);
	}
	
	private boolean isDragRequest(FacesContext context) {
	  String eventName =  context.getExternalContext().getRequestParameterMap().get("javax.faces.behavior.event");
	
	  if("colReorder".equals(eventName)){
	    return true;
	  }
			
	  return false;
	}

Dessa forma, o evento colReorder agora será chamado no momento apropriado.

<strong>Mateus 10.28:</strong>Não tenham medo dos que matam o corpo, mas não podem matar a alma. Antes, tenham medo daquele que pode destruir tanto a alma como o corpo no inferno.
