---
title: "Running Maven project as a Web application in Eclipse"
date: 2012-03-28 18:49
comments: true
lang: en
categories:
 - Maven
 - Eclipse
tags:
 - m2eclipse
---

These days I was setting my Eclipse to work with <a target="_blank" href="http://www.frameworkdemoiselle.gov.br/" title="Demoiselle Framework"> Demoiselle Framework </a> and how this framework is fully managed by Maven one of the necessary plugins is the <a target="_blank" href="http://eclipse.org/m2e/download/" title="m2eclipse"> m2eclipse </a> to do integration between the Eclipse and Maven.

<!-- more -->

So far so good, you can open the Eclipse go the menu Help > Install New Software and use the update site http://download.eclipse.org/technology/m2e/releases to install the m2eclipse. With that, I could use one of the Demoiselle archetypes to download an application already fully configured.

But one problem I had was to make this project run as a web application via Eclipse and then after much poking around, I would need another plugin to make this integration (maven/eclipse/web application) work, is the m2e-extras. Simply you do the same procedure as above and use the update site http://m2eclipse.sonatype.org/sites/m2e-extras.

Just in case, go to the menu Project > Properties > Project Facets and select the Dynamic Web Module option. After that, just click the right button in the project going in option Run as > Run on Server and ready, Maven project running as a web application usually!

**Update:** In case of compatibility problems with the version of m2e-extras, then you must use the update site http://m2eclipse.sonatype.org/sites/m2e-webby/ that supports the latest version of Maven.
