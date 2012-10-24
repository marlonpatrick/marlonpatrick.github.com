---
layout: post
title: "Executando projeto Maven como aplica&ccedil;&atilde;o Web no Eclipse"
date: 2012-03-28 18:49
comments: true
categories: [Maven, Eclipse]
tags: [Maven, Eclipse, m2eclipse]
---

Esses dias estava configurando meu Eclipse para trabalhar com <a href="http://www.frameworkdemoiselle.gov.br/" target="_blank" title="Demoiselle Framework">Demoiselle Framework</a> e como esse framework &eacute; totalmente gerenciado pelo Maven um dos plugins necess&aacute;rios &eacute; o <a href="http://eclipse.org/m2e/download/" target="_blank" title="m2eclipse">m2eclipse</a> para fazer integra&ccedil;&atilde;o entre Eclipse e Maven.

At&eacute; a&iacute; tudo bem, voc&ecirc; pode abrir o Eclipse ir no menu Help > Install New Software e o usar o update site http://download.eclipse.org/technology/m2e/releases para instalar o m2eclipse. Com isso, eu pude usar um dos arqu&eacute;tipos do Demoiselle para baixar uma aplica&ccedil;&atilde;o j&aacute; totalmente configurada.

Por&eacute;m, um problema que eu tive foi o de fazer esse projeto executar como uma aplica&ccedil;&atilde;o web via Eclipse e a&iacute; depois de muito fu&ccedil;ar, vi que precisaria de um outro plugin para fazer essa integra&ccedil;&atilde;o maven/eclipse/aplica&ccedil;&atilde;o web funcionar, &eacute; o m2e-extras. Para isso, basta voc&ecirc; fazer o mesmo procedimento acima e usar o update site http://m2eclipse.sonatype.org/sites/m2e-extras.

S&oacute; para garantir, v&aacute; no menu Project > Properties > Project Facets e marque a op&ccedil;&atilde;o Dynamic Web Module. Depois disso, &eacute; s&oacute; clicar com o bot&atilde;o direito no projeto ir na op&ccedil;&atilde;o Run as > Run on Server e pronto, projeto Maven executando como uma aplica&ccedil;&atilde;o web normalmente!



**Update:**No caso de haver problemas de compatibilidade com a versão do m2e-extras, então, será necessário usar o update site http://m2eclipse.sonatype.org/sites/m2e-webby/ que suporta a versão mais recente do Maven.

<strong>2 João 1.9:</strong>Todo aquele que prevarica e não persevera na doutrina de Cristo não tem a DEUS; quem persevera na doutrina de Cristo, esse tem tanto o Pai como o Filho.
