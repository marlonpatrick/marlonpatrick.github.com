---
title: "JasperReports: Solving NoSuchMethodError to use an image"
date: 2014-04-28 09:21
lang: en
categories:
 - JasperReports
 - iReport
tags:
 - JasperReports
 - iReport
 - iText
---

The error occurs when you try to generate a report made with Jasper/iReport using an image. The error message is something like: <strong>Caused by: java.lang.NoSuchMethodError: com.lowagie.text.Image.getPlainWidth ()F</ strong>.

<!-- more -->

The problem is that you are using a version of iText incompatible with the version of JasperReports. Until the version 3.0.0 of the Jasper/iReport the correct version of iText is 1.3.1. Starting with version 3.1.0 of the Jasper/iReport the correct version of iText is 2.1.0.

A better tip is to check the lib folder of your iReport which will contain all the libs with their correct versions, so you can ensure you are using the appropriate version of each dependency of JasperReports.
