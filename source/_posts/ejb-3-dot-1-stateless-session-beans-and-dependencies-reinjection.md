---
title: "EJB 3.1: Stateless Session Beans and reinjection dependencies"
date: 2013-09-04 22:50
comments: true
lang: en
categories:
 - EJB
tags:
 - Stateless Session Bean
 - Dependency Injection
 - CDI
---

When you start working with EJB one of the first things to learn is about the different types of beans, which at 3.1 specification are: Stateful Session Beans (SFSB), Stateless Session Beans (SLSB), Singleton Sessions Beans and Message-Drive Beans. With this comes one of the most common questions: "What is the difference between Stateful and Stateless beans?". Well, this post is not one to explain these differences, if you need that follows a good <a target = "_ blank" href="http://www.theserverside.com/tutorial/Which-EJB-to-use-Stateful-stateless-and-singleton-session-beans-compared">link</a>.

<!-- more -->

But to my post need to make sense to put the standard response:. "SFSBs preserve the state even if they occur several calls methods in the bean as the container ensures that the customer will always be serviced by the same instance. Already SLSBs not preserve state because, for each method call the customer makes it can be serviced by a different instance."

However, with the advent of CDI and its powerful dependency injection mechanism that changes a bit, because even a SLSB can have a "pseudo conversational state" when using this feature. Most developers (including me until this afternoon) believes that the injection of dependencies for an EJB happens only during instance creation and thus assume things like, "I can not inject a managed bean @SessionScoped in an SLSB, for as I have no guarantee of being always attended by the same instance I can get one that has the managed bean to another session".

In fact it is not so, I found it today and if he knew me before would have avoided big headache, so I decided to share here. The EJB 3.1 specification on page 74 item 4.3.2 says the following:
> If a session bean makes use of dependency injection, the container injects these references after the bean instance is created, and before any business methods are invoked on the bean instance.

Ie the dependency injection is done after creating the bean instance and also before each call of a business method. This was very useful for me, as was already racking their brains on how to keep the EntityManager to inject in my EJBs (through Seam Managed Persistence Context my EntityManager is a @ConversationScoped) according to the EJB client context. As the container reinjected dependencies before each method call actually I do not even need to have worried about it.

Somehow, this definition can give SLSBs the ability to have a "conversational state". For example, if you inject in a SLSB a bean with session scope (@SessionScoped), regardless of which pool instance process your request the bean is session-scoped always according to the current client session. In other words, there is no possibility of a SLSB have a session bean another client. This allows a SLSB receive the user logged in as an instance field through @Inject without causing any problem if two users use the same instance of SLSB alternately. For example:


	@Stateless
	public class StatelessSessionBean{
	    @Inject
	    @LoggedInUser
	    protected User loggedInUser;//@SessionScoped

	    public void test(){
	        System.out.println("ejb:"+this);
	        System.out.println("user:"+this.loggedInUser);
	    }
	}

Then the result of several calls the method "test" by different users is as follows:

	User 1 invoke
	17:02:20,800 INFO  [stdout] (http--0.0.0.0-8080-5) ejb:StatelessSessionBean@406189
	17:02:20,801 INFO  [stdout] (http--0.0.0.0-8080-5) user:User 1

	User 2 invoke
	17:02:56,227 INFO  [stdout] (http--0.0.0.0-8080-8) ejb:StatelessSessionBean@406189
	17:02:56,228 INFO  [stdout] (http--0.0.0.0-8080-8) user:User 2

	User 2 invoke
	17:03:24,376 INFO  [stdout] (http--0.0.0.0-8080-8) ejb:StatelessSessionBean@406189
	17:03:24,378 INFO  [stdout] (http--0.0.0.0-8080-8) user:User 2

	User 1 invoke
	17:03:24,517 INFO  [stdout] (http--0.0.0.0-8080-6) ejb:StatelessSessionBean@1c05227
	17:03:24,518 INFO  [stdout] (http--0.0.0.0-8080-6) user:User 1

	User 1 invoke
	17:04:24,045 INFO  [stdout] (http--0.0.0.0-8080-1) ejb:StatelessSessionBean@406189
	17:04:24,047 INFO  [stdout] (http--0.0.0.0-8080-1) user:User 1

	User 2 invoke
	17:04:24,179 INFO  [stdout] (http--0.0.0.0-8080-8) ejb:StatelessSessionBean@1c05227
	17:04:24,179 INFO  [stdout] (http--0.0.0.0-8080-8) user:User 2

Note that even being serviced by the same instance, the field "loggedInUser" varies correctly according to the user that invoked the method.

