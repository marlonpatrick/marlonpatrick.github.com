---
title: "PrimeFaces: around problems with resizable/draggable dataTable"
date: 2014-04-26 16:29
comments: true
lang: en
categories:
 - PrimeFaces
tags:
 - JSF
 - PrimeFaces
 - DataTable
---

I will leave here workarounds for some PrimeFaces bugs when using dataTable with features resizable or draggable. Some of these bugs have been corrected, but if for some reason you like me works with any version of PrimeFaces containing the error and can not update it will be of great help.

<!-- more -->

**1 - NullPointerException to resize the size of a column (fixed in version 3.5)**

The problem occurs in a dataTable which is resizable and draggable at the same time. I found the following tickets in the PrimeFaces issue tracker to address the problem: <a href="https://code.google.com/p/primefaces/issues/detail?id=3674" title="Issue 3674" target="_blank">3674</a>, <a href="https://code.google.com/p/primefaces/issues/detail?id=4012" title="Issue 4012" target="_blank">4012</a>,<a href="https://code.google.com/p/primefaces/issues/detail?id=4796" title="Issue 4796" target="_blank">4796</a> and <a href="https://code.google.com/p/primefaces/issues/detail?id=5167" title="Issue 5167" target="_blank">5167</a>. In version 3.4.1 of PrimeFaces the error log is something like:

	01:40:49,907 INFO  [javax.enterprise.resource.webcontainer.jsf.context] (http--0.0.0.0-8080-1) java.lang.NullPointerException: java.lang.NullPointerException
	at org.primefaces.component.datatable.feature.ResizableColumnsFeature.decode(ResizableColumnsFeature.java:35) [primefaces-3.4.1.jar:]
	at org.primefaces.component.datatable.DataTableRenderer.decode(DataTableRenderer.java:53) [primefaces-3.4.1.jar:]
	at javax.faces.component.UIComponentBase.decode(UIComponentBase.java:787) [jboss-jsf-api_2.1_spec-2.0.1.Final.jar:2.0.1.Final]
	at org.primefaces.component.api.UIData.processDecodes(UIData.java:224) [primefaces-3.4.1.jar:]
	at com.sun.faces.context.PartialViewContextImpl$PhaseAwareVisitCallback.visit(PartialViewContextImpl.java:506) [jsf-impl-2.1.7-jbossorg-2.jar:]
	at com.sun.faces.component.visit.PartialVisitContext.invokeVisitCallback(PartialVisitContext.java:183) [jsf-impl-2.1.7-jbossorg-2.jar:]
	at org.primefaces.component.api.UIData.visitTree(UIData.java:635) [primefaces-3.4.1.jar:]

Briefly the error happens because when you resize a column the PrimeFaces processes both resizing (resize) and the reordering (reorder) of the column. This can be easily realized if you use the colResize and colReorder event to generate a log.

A detailed explanation is as follows: to create the effect of the resize a column PrimeFaces creates a tag `<span>` in the header of the column and makes this tag an object <a href="http://jqueryui.com/draggable/" title="jQuery UI - Draggable" target="_blank">draggable</a>, thus when the user drags the "column" for her to stay greater or minor and loose there is a "collision" between the resizer and reorder feature of dataTable. This is because the report that the dataTable is draggable the PrimeFaces makes all draggable columns so that they can be dragged and reordered and makes the whole table header <a href = "http://jqueryui.com/droppable/" title =" jQuery UI - Droppable" target ="_blank">droppable</a>. The problem is that the implementation of PrimeFaces missed configuring the droppable to accept only reordering columns and thus to release the `<span>` that resizes to the drop reordering of the column is used and the PrimeFaces understands that the column is being reordered and resized, with this method you should adjust the size of the column does not receive a parameter (the id of the resized column) it needs to function properly and then causes the error.

To solve this problem I added a script after closing the tag `<p:dataTable>` that defines a specific scope for '<span>' which is used in resizing columns. So, to resize a column even if the user release the mouse on the table header of the drop column reordering will not be called because the draggable object '<span>' has a different scope of droppable object (table header). The code is as follows:

	<p:dataTable draggableColumns="true" resizableColumns="true">
	       <!-- your code -->
	</ p: dataTable>
	
	<script type="text/javascript">
	    $( ".ui-column-resizer" ).draggable( "option", "scope", 'resizer');//resizer is the name of the scope that chose, could be any other name
	</script>

**2 - NullPointerException to resize a dynamic column (fixed in version 3.5.7)**

In this case simply just have a resizable dataTable with dynamic columns so that the error happens. I found the following tickets in the PrimeFaces issue tracker to address the problem: <a href = "https://code.google.com/p/primefaces/issues/detail?id=4238" title = "Issue 4238" target = "_blank">4238</a> and <a href="https://code.google.com/p/primefaces/issues/detail?id=4797" title="Issue 4797" target="_blank">4797</a>. Funny that even with several people reporting the problem the PrimeFaces the team scored both issues as "Unable to replicate." According to one observer the problem was resolved in version 3.5.7, but could not prove.

The point where the error occurs is the same as the case described above, so the log is similar and only what changes is the root cause of the problem. Previously, the NullPointer reason is that the parameter with the id of the resized column is not sent to the server, as in the case of dynamic columns, even when the id is sent PrimeFaces can not find the corresponding column. The method that is used to fetch the column by id is `DataTable.findColumn (String clinetId)` which only looks for simple columns and ignores the dynamics.

In my case, it was not so important that the Column object were found and that their width field was setted as the width used on my screen is stored in another specific object of my system. So my solution was only override the class that processes the resize the dataTable event (org.primefaces.component.datatable.feature.ResizableColumnsFeature), but specifically the decode method:

	@Override
	public void decode(FacesContext context, DataTable table) {
		Map<String, String> params = FacesContext.getCurrentInstance().getExternalContext().getRequestParameterMap();
		String clientId = table.getClientId();

		String columnId = params.get(clientId + "_columnId");
		String width = params.get(clientId + "_width");
		Column column = table.findColumn(columnId);

		if(column != null){ //added this if that to avoid NullPointerException
			column.setWidth(Integer.parseInt(width));
		}
	}
	
Dessa forma o evento colResize pode ser invocado e é através dele que resolvo meu problema. Talvez o código seja difícil de aplicar em outro sistema, mas, de toda forma vou deixar aqui pelo menos para você ter uma idéia de como fiz e poder adaptar a solução:

Thus the colResize event can be invoked and it is through him that solve my problem. Perhaps the code is difficult to apply in another system, but anyway I will leave here at least you get an idea of how I did and be able to adapt the solution:

	public void onResizeColumn(ColumnResizeEvent event) {
		if (event.getColumn() == null) {
			String index = getColumnIndex(event);
			DynaColumn dynaColumn = this.getColumns().get(Integer.parseInt(index));//getColumns() is the list you use to generate the dynamic columns
			dynaColumn.setWidth(event.getWidth());//DynaColumn is a particular object of my system
		}
	}

	private String getColumnIndex(ColumnResizeEvent event) {
		String columnId = FacesContext.getCurrentInstance().getExternalContext().getRequestParameterMap().get(event.getComponent().getClientId() + "_columnId");
		String[] columnIdSplited = columnId.split(":");
		return columnIdSplited[columnIdSplited.length - 1];
	}

In my JSF page using the DynaColumn.width field to set the size of the column even if the Column object PrimeFaces has not been updated.

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
				$('[id="dataTableID:dynamicColumns:#{status.index}"]').
	    			children('.ui-dt-c').css('width',columnWidth);
			}
		</script>				
	</ui:repeat>


The issue <a href="https://code.google.com/p/primefaces/issues/detail?id=4238" title="Issue 4238" target="_blank">4238</a> proposes another way around the problem, in addition, other possible solutions would adjust the ResizableColumnsFeature.decode method to search the column the right way or else the onResize set the Column object rather than a system object itself. If any of these fixes will be implemented javascript would not be necessary to adjust the width of columns in the JSF page thus simplifying the solution. In my case there is a business requirement that forced me to make the form presented here.

**3 - colReorder event runs for any ajax call and not only in the column reordering**

To simulate this error simply just have a draggable dataTable and make any ajax call on the screen. I found the ticket <a href="https://code.google.com/p/primefaces/issues/detail?id=4985" title="Issue 4985" target="_blank">4985</a> in the rimeFaces issue tracker that addresses the problem. The same was marked "not be corrected," then I believe still happen in any version.

To solve the problem I overwrite the org.primefaces.component.datatable.feature.DraggableColumnsFeature class to adjust the shouldDecode method:

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

Thus, the colReorder event will now be called at the appropriate time.
