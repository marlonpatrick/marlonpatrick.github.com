---
title: "PrimeFaces: Menu Dinâmico e o erro Cannot remove the same component twice"
date: 2013-01-24 22:39
comments: true
lang: pt-br
alias: blog/2013/01/24/primefaces-menu-dinamico-e-o-erro-cannot-remove-the-same-component-twice/index.html
categories:
 - JSF
 - PrimeFaces
tags:
 - JSF
 - PrimeFaces
 - Dynamic Menu
 - Menu Dinâmico
 - MenuBar
---

Se você está usando o componente MenuBar do PrimeFaces para geração de menus dinâmicos pode ter se deparado com o erro:<strong>Cannot remove the same component twice</strong>. Esse erro ocorre pelo mesmo motivo explicado no post <a href="/blog/2013/01/24/primefaces-menu-dinamico-e-a-inofensiva-mensagem-unable-to-find-component-dot-dot-dot">PrimeFaces:Menu Dinâmico E a Inofensiva Mensagem Unable to Find Component…</a>.

<!-- more -->

A diferença é que a mensagem "Unable to find component..." é apenas um warnning no log do servidor que num caso muito específico pode representar um problema de performance, já a mensagem "Cannot remove the same component twice" é um erro de fato que impede a renderização da tela JSF. No meu caso, o erro ocorre quando um Validator JSF lança uma ValidatorException numa tela que tem um menu dinâmico (PrimeFaces MenuBar).

Outra coisa importante é que o erro ocorre na versão 3.4.1 do PrimeFaces(não cheguei a verificar outras versões) em conjunto com a implementação JSF Mojarra 2.1.9 ou superior. Caso a versão do Mojarra seja menor que a 2.1.9 ao invés de ocorrer erro é jogado no log do servidor a mensagem "Unable to find component...".

A solução colocada no post referido anteriormente também serve para esse caso e lá explico melhor o porque do problema. É só clicar <a href="/blog/2013/01/24/primefaces-menu-dinamico-e-a-inofensiva-mensagem-unable-to-find-component-dot-dot-dot">aqui</a> e dá uma conferida. Já até abri uma issue no bug tracking do PrimeFaces no seguinte <a target="_blank" href="http://code.google.com/p/primefaces/issues/detail?id=4431">link</a>. Por enquanto é isso, vlw!
