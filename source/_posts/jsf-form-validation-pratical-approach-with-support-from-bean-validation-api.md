---
title: "JSF Form Validation: Practical approach with support from the Bean Validation API + Seam Faces"
date: 2012-07-12 19:08
comments: true
lang: en
categories:
 - JSF
 - JBoss Seam
 - Bean Validation
tags:
 - JSR 303
 - JSF Validation
 - Seam Faces
---

Well, if you already work with JSF some time knows that creating screens with a little more complex flows using the built-in validations or even Bean Validation know that this can become somewhat complicated. Basically the problem is that often the validations are triggered at inopportune moments, times when it is not necessary to such validation, but this is due to overstated JSF lifecycle.

<!-- more -->

A simple example for those who use Primefaces is when working with tabs, where each time you change tab validations are called by JSF, which in many cases is unnecessary. To learn more: http://code.google.com/p/primefaces/issues/detail?id=3423.

Yes, I know it gives to circumvent most of these problems with partial rendering (attribute execute in the standards tags and process in the Primefaces tags), but the truth is that the code starts to get complex and in some cases has not hopeless and many people end up giving up these validations and put validations in the own back bean.

Well, those days doing one of these screens and ended up running into these problems. Leave the code of complex pages is not a good fit for the job and keep doing validations directly in back beans also did not seem the most elegant solution (although work well). In Struts times there was the concept of form validation which is not natural in JSF therefore validators are determined component to component. But then I found this functionality in Seam Faces, with the tag <s:validateForm />. With this Seam Faces tag can define a JSF Validator for form and not just for a particular field and this validator is triggered as soon as the form is submitted.

Note: Any difficulty adding Seam Faces in your project read this other post [Adding Seam Faces In Your Maven Project] (/en/2012/06/14/adding-seam-faces-in-your-maven-project/).

Thus, we already have a way to validate the form only at the time that it is submitted, but would still have to do the validations manually for each field, that's when I had the idea to use the api Bean Validation to do this work for me. The idea is simple, I put Bean Validation annotations in my model, I disable the validation of the JSF life cycle with the <f:validateBean /> (JSF 2 and Bean Validation are integrated and validations are enabled by default) and the valid model in my form validator calling validations Bean Validation programmatically. That way my form is validated with the api Bean Validation at the exact moment that interests me, that is, the submit.

The code for this is as follows:

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

				//The error messages are available for a possible subclass to use
				this.allMessages = new LinkedHashSet<FacesMessage>(messages);
			}
		}

		/**
		 * Scans all form fields and if you are a
		 * UIInput calls the standard BeanValidator JSF. 
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
		 * Call the standard JSF BeanValidator
		 * For a given UIInput.
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
		 * The default behavior is to validate all UIInput,
		 * But it may be not necessary to validate
		 * A particular component on a screen.
		 *
		 * So this validator can be extended and the
		 * Method used to identify whether the component
		 * Must be validated or not.
		 *
		 *@author marlonpatrick10
		 */
		protected Boolean isValidatable(FacesContext context, UIInput input) {
			return true;
		}

		/**
		 * The default behavior is to validate all UIInput,
		 * And throw a ValidatorException if there are errors.
		 * This validator can be extended and the
		 * Method overridden to not throw the exception,
		 * So the new validator can make extra validations.
		 *
		 * Error messages from the Bean Validation
		 * Are available in this.allMessages attribute.
		 *
		 *@author marlonpatrick10
		 */
		protected Boolean isThrowsValidatorException() {
			return true;
		}
	}

The FormBeanValidator can cater for most screens, but even so there are cases that want to specialize more validations. Here is an example:

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
			// Upon submit the user registration screen
			// No need to validate the fields below		

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

			//email2 can not be annotated with @NotNull therefore your obligation is conditional

			if (this.userView.isAdmin()) {
				if (component.getClientId().endsWith("email2")) {
					if (GenericUtils.isBlankOrNull((String)component.getValue())) {
						this.allMessages.add(new FacesMessage(FacesMessage.SEVERITY_ERROR,
								"Admin user should tell two emails.",null));
					}
				}
			}

			//Call Bean Validation even email2 therefore
			//User.email2 Is annotated with @Email then
			//Still needs to be validated
			super.validate(context, beanValidator, component, messages);
		}
	}

On page (xhtml) should be something like follows:

	<html xmlns="http://www.w3.org/1999/xhtml"
		xmlns:h="http://java.sun.com/jsf/html"
		xmlns:f="http://java.sun.com/jsf/core"
		xmlns:s="http://jboss.org/seam/faces">

		<h:form id="formInsertUser" prependId="false">
			<f:validateBean disabled="true">
				<!-- Fields annotated with Bean Validation should be here -->
			</f:validateBean>

			<h:commandLink value="Save" action="#{userView.insert}" />
	
			<!-- The validator will be called at the click commandLink -->
			<s:validateForm validatorId="userFormBeanValidator" />
		</h:form>

	</html>

This form of validation together with the Bean Validation will help me a lot to build more complex pages with JSF without much headache with validations out of turn, I hope it helps you too.
