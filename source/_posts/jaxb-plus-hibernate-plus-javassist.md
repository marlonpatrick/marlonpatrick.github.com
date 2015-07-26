---
title: "JAXB + Hibernate + Javassist"
date: 2012-07-12 13:08
comments: true
lang: pt-br
alias: blog/2012/07/12/jaxb-plus-hibernate-plus-javassist/index.html
categories:
 - APIs Java
 - Hibernate
tags:
 - JAXB
 - Hibernate
 - Javassist
---

Se você está tentando converter um objeto java em XML usando JAXB pode ter alguns problemas caso esse objeto esteja sendo gerenciado pelo Hibernate. Em alguns casos o Hibernate reescreve as classes para fazer algumas melhorias e adiconar funcionalidades extras, como por exemplo, objetos com mapeamentos Lazy. Nesses casos o Hibernate reescreve a classe do objeto para que ele então possa fazer o carregamento tardio dos dados.

<!-- more -->

Até aí tudo bem, nós nem temos conhecimento, pois, trabalhamos com a interface do objeto real, porém, quando usamos uma api como JAXB que usa reflection para obter os campos e métodos do objeto, isso pode causar alguns problemas. No meu caso, o erro foi mais ou menos assim:

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

O caso é que o Hibernate (usando a api Javassist) fez um proxy na minha classe User para adicionar comportamento Lazy numa das propriedades da classe e adicionou um atributo chamado handler, o qual, não há nenhuma anotação JAXB definindo como deve ser tratado esse atributo na hora do marshal e por isso o erro acima ocorreu.

Bem, dando uma pesquisada a minha solução é fazer o unproxy na classe proxyada pelo Hibernate e então faço marshal no meu objeto real. Para obter o objeto correto com Hibernate é só copiar o código abaixo:

	public Object  getUnproxyModel(Object model) {
		if (HibernateProxy.class.isAssignableFrom(model.getClass())) {
			return ((HibernateProxy)model).getHibernateLazyInitializer().getImplementation();
		}

		return model;
	}

Então, é só fazer assim:

	marshaller.marshal(getUnproxyModel(user), outputStream);

É isso. Vlw!
