---
layout: post
title: "Seam Faces: TransactionPhaseListener e o erro JBAS010152"
date: 2012-10-31 16:40
comments: true
categories: [JSF, JBoss Seam]
tags: [JSF, JBoss Seam, Seam Faces, CDI, Seam Solder] 
---

Se você está usando Seam Faces pode já ter se deparado com a seguinte mensagem no log de sua aplicação:
<strong>JBAS010152: APPLICATION ERROR: transaction still active in request with status 0</strong>

Essa mensagem é provocada pelo mal funcionamento do controle de transação do seu sistema, mais especificamente, pelo mal funcionamento da classe TransactionPhaseListener, contida no jar do Seam Faces. Basicamente, TransactionPhaseListener implementa um PhaseListener comum do JSF que no início de toda requisição abre uma nova transação e no final da requisição realiza o commit ou rollback da transação.

O problema é que o TransactionPhaseListener é habilitado por padrão desde que o seu projeto tenha o Seam Faces, Seam Persistence e Seam Transaction, assim, o simples fato de colocar essas libs no seu projeto já adiciona o comportamento acima descrito no sistema, e na maioria dos casos, acaba entrando em conflito com o controle de transação escolhido para ser usado na aplicação. No meu caso, o comportamento do TransactionPhaseListener não atendia e pior, atrapalhava. A solução que encontrei foi desabilitar esse comportamento através da seguinte configuração no arquivo beans.xml:

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:s="urn:java:ee" xmlns:t="urn:java:org.jboss.seam.transaction"
	       xmlns:ft="urn:java:org.jboss.seam.faces.transaction"
	       xmlns:sc="urn:java:org.jboss.solder.core"
	       xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://docs.jboss.org/cdi/beans_1_0.xsd">
		
	    <!-- Others Configurations -->

	    <sc:Veto>
	    	<!-- Defines @Veto as a Qualifier -->
		<s:Qualifier/>
	    </sc:Veto>

	    <ft:TransactionPhaseListener>
	    	 <!--  Preventing class TransactionPhaseListener from being installed as CDI bean.
	    		 This class is responsible by message: JBAS010152: APPLICATION ERROR: transaction still active in request with status 0. 
	    		--> 
		<s:replaces/>
		<sc:Veto/>
	    </ft:TransactionPhaseListener>
	</beans>


Com essa configuração no beans.xml a classe TransactionPhaseListener será ignorada pelo CDI e portanto ficará desabilitada, eliminando assim o problema no controle de transação e aquela mensagem chata (JBAS010152). Na verdade, quando consegui me livrar desse problema apareceram outros no meu projeto, pois, ele estava ocultando alguns defeitos no meu controle de transação, então, recomendo tentar validar sua estratégia após desabilitar o TransactionPhaseListener e se certificar que está em perfeito funcionamento. Vlw!


<strong>Provérbios 1.10:</strong>Filho meu, se os pecadores procuram te atrair com agrados, não aceites.
