---
title: "JAXB + Hibernate + Javassist"
date: 2012-07-12 13:08
comments: true
lang: en
categories:
 - JAXB
 - Hibernate
tags:
 - JAXB
 - Hibernate
 - Javassist
---

If you are trying to convert a java object to XML using JAXB may have some problems if the object is being managed by Hibernate. In some cases Hibernate rewrites classes to make some improvements and add extra features, such as objects with Lazy mappings. In such cases Hibernate rewrites the class of the object so that it can do lazy loading of data.

<!-- more -->

So far so good, we do not have knowledge as we work with the real object interface, however, when using an api as JAXB using reflection for the fields and methods of the object, it may cause some problems. In my case, the error was something like this:

	com.sun.xml.bind.v2.runtime.IllegalAnnotationsException: 2 counts of IllegalAnnotationExceptions
	javassist.util.proxy.MethodHandler is an interface, and JAXB can't handle interfaces.
		this problem is related to the following location:
			at javassist.util.proxy.MethodHandler
			at private javassist.util.proxy.MethodHandler User_$$_javassist_19.handler
			at User_$$_javassist_19
	javassist.util.proxy.MethodHandler does not have a no-arg default constructor.
		this problem is related to the following location:
			at javassist.util.proxy.MethodHandler
			at private javassist.util.proxy.MethodHandler User_$$_javassist_19.handler
			at User_$$_javassist_19

The case is that Hibernate (using Javassist api) made a proxy in my User class to add Lazy behavior in one of the class properties and added an attribute called handler, which there is no JAXB annotation defining how it should be treated this attribute the marshal time and so the above error occurred.

Well, giving a searched my solution is to unproxy in class wrapped by Hibernate and then marshal do in my real object. For the correct object with Hibernate simply copy the code below:


	public Object  getUnproxyModel(Object model) {
		if (HibernateProxy.class.isAssignableFrom(model.getClass())) {
			return ((HibernateProxy)model).getHibernateLazyInitializer().getImplementation();
		}

		return model;
	}

So, just do this:

	marshaller.marshal(getUnproxyModel(user), outputStream);
