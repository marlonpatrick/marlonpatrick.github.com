---
title: "PrimeFaces: Dynamic Menu and the error Cannot remove the same component twice"
date: 2013-01-24 22:39
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

If you are using the MenuBar component PrimeFaces to generate dynamic menus you may have encountered the error:<strong>Can not remove the same component twice</strong>. This error occurs for the same reason explained in the post <a href="/en/2013/01/24/primefaces-dynamic-menu-and-the-harmless-menssage-unable-to-find-component-dot-dot-dot/">PrimeFaces: Dynamic Menu and harmless message Unable to Find Component ... </a>.

<!-- more -->

The difference is that the "Unable to find component ..." message is just a warnning the server log to a very specific case can be a performance problem, since the message "Cannot remove the same component twice" is an error fact that prevents the rendering of JSF screen. In my case, the error occurs when a JSF Validator throws a ValidatorException a screen that has a dynamic menu (PrimeFaces MenuBar).

Another important thing is that the error occurs in the 3.4.1 version of PrimeFaces (did not get to check other versions) in conjunction with the implementation Mojarra JSF 2.1.9 or higher. If the Mojarra version 2.1.9 is less than the error occurs instead is played on the server log the message "Unable to find component ...".

The solution placed in the post mentioned above also serves to this case and there explain better why the issue. Just click <a href="/en/2013/01/24/primefaces-dynamic-menu-and-the-harmless-menssage-unable-to-find-component-dot-dot-dot/">here</a> and gives a given. I have even opened a PrimeFaces issue in the bug tracking the following <a target="_blank" href="http://code.google.com/p/primefaces/issues/detail?id=4431">link</a>.
