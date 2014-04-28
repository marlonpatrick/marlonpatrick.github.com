---
layout: post
title: "JasperReports: Resolvendo NoSuchMethodError ao usar uma imagem"
date: 2014-04-28 09:21
categories: [JasperReports, iReport]
tags: [JasperReports, iReport, iText]
---

O erro acontece quando você tenta gerar um relatório feito com Jasper/iReport que usa uma imagem. A mensagem de erro é algo como: <strong>Caused by: java.lang.NoSuchMethodError: com.lowagie.text.Image.getPlainWidth()F</strong>.

O problema é que você está usando uma versão do iText imcompatível com a versão do JasperReports. Até a versão 3.0.0 do Jasper/iReport a versão correta do iText é 1.3.1. A partir da versão 3.1.0 do Jasper/iReport a versão correta do iText é 2.1.0.

Uma dica melhor é verificar a pasta lib do seu iReport o qual irá conter todas as libs com suas versões corretas, assim, pode garantir que está usando a versão adequada de cada dependência do Jasper.

<strong>Efésios 2.8-9:</strong>Pois vocês são salvos pela graça, por meio da fé, e isto não vem de vocês, é dom de Deus; não por obras, para que ninguém se glorie.
