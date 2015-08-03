---
title: "PrimeFaces: Dynamic Menu and harmless message Unable to find component ..."
date: 2013-01-24 12:00
comments: true
lang: en
categories:
 - JSF
 - PrimeFaces
tags:
 - JSF
 - PrimeFaces
 - Dynamic Menu
 - MenuBar
 - Mojarra
---

I am particularly hardcore fan of PrimeFaces and one of the components I like most is to generate dynamic menus. Virtually all applications have the concept of access profile which ends up making the MenuBar component extremely useful for this type of requirement. Basically, the system will only render the menus which point to screens that the user can access.

<!-- more -->

Well, I'm using that component precisely in the above scenario with the 3.4.1 version of PrimeFaces and the implementation of JSF reference (Mojarra) in the version 2.1.7. With this combination (PrimeFaces/Mojarra) n the versions described above (believe in others as well) realized in the log my application every time performing an ajax call for each menu item generated with the dynamic MenuBar, there was a message something like this:<strong>Unable to find component with clientId 'menuitem200', the need to remove it</strong>.

Up to a certain moment nothing affected me such a message, and is probably so for 99% of cases. Now, if you like me, it fell to the remaining 1%, this message can cause a serious performance problem in ajax calls. This performance problem will only happen if you have a very large page, with many components generated as was my case.

Every time I made a ajax call that page the procedure performed by PrimeFaces that generated the message "Unable to find component ..." came to take about 1 minute and 30 seconds to finish, which is totally impractical. It's not that the code runs on the call was super heavy, these more than 1 minute refer only method execution that generated the messages as shown in the log below:

        02:40:00,876 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'submenu1', no need to remove it.
        02:40:01,298 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'submenu2', no need to remove it.
        02:40:01,656 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'menuitem45', no need to remove it.
        02:40:02,014 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'menuitem4', no need to remove it.
        02:40:02,361 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'submenu20', no need to remove it.

        ...many others messages here...

        02:40:44,240 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'menuitemespacobefore', no need to remove it.
        02:40:44,917 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'submenuuseractions', no need to remove it.
        02:40:45,249 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'menuitemchangepassword', no need to remove it.
        02:40:45,574 INFO  [stdout] (http--0.0.0.0-8080-4) Unable to find component with clientId 'menuitemlogout', no need to remove it.

In the above example, the total time was 46 seconds. Basically what happens is that the JSF Restore View phase runs the method that restores the state of the dynamic menus, where for each menu item is done a search on the entire tree of JSF components and due to page be great takes a considerable time. The worst is that as the message itself says, "the need to remove it," is not necessary to remove this component of the JSF tree because in fact it has already been removed.

In my case, what I realized is that when the dynamic menu item uses a link is not necessary that the removal PrimeFaces tries, as when the menu item uses an Action Expression, then yes, this procedure is necessary.

                // It is not necessary to try to remove this MenuItem therefore it uses a link
                MenuItem changePassword = new MenuItem();
                changePassword.setId("menuitemchangepassword");
                changePassword.setValue("Change Password");
                changePassword.setUrl("/user/changePassword.xhtml?faces-redirect=true");
                submenuUser.getChildren().add(changePassword);

                //In fact it is necessary to remove the item menu to add the new object in the Restore View phase, because it uses a Action Expression
                MethodExpression logoutAction = factory.createMethodExpression(context.getELContext(), "#{loginView.logout}",
                                null, new Class[] {});
                MenuItem logout = new MenuItem();
                logout.setId("menuitemlogout");
                logout.setValue("Logout");
                logout.setActionExpression(logoutAction);
                submenuUser.getChildren().add(logout);

To solve this problem I created a PhaseListener that removes the menu items that use this link unnecessary restoration phase, leaving only those who use action expression.

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
		    
			    //These are the only items that use Action Expression, and therefore should not be scrambled
			    if(ComponentStruct.ADD.equals(action.action) && 
		    		(action.clientId.equals("menuitemlogout") || action.clientId.equals("submenuuseractions"))){
			    	continue;
			    }

			    //Other items use link, and so are removed from this collection
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

With PhaseListener up, I let only the menu items that use Action Expression in the collection of dynamic actions to restore. In my case, I have a menu with hundreds of items, of which only 2 or 3 using action expression and thus the time spent on the restoration of dynamic shares fell sharply with that adjustment, getting unnoticeable (as it should be). This is because the PhaseListener runs before restoring leaving only what is truly necessary. Oh, another thing, the message "Unable to find component ..." should also disappear from your log.

