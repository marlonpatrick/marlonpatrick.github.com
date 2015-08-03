---
title: "Seam Faces: TransactionPhaseListener and the error JBAS010152"
date: 2012-10-31 16:40
comments: true
lang: en
categories:
 - JBoss Seam
tags:
 - JSF
 - JBoss Seam
 - Seam Faces
 - CDI
 - Seam Solder
---

If you are using Seam Faces you may have already come across the following message in the log of your application:
<strong>JBAS010152: APPLICATION ERROR: transaction still active in request with status 0</strong>

<!-- more -->

This message is caused by the malfunction of the transaction control of your system, more specifically, by the malfunction of TransactionPhaseListener class, contained in the Seam Faces jar. Basically, TransactionPhaseListener implements a common JSF PhaseListener at the beginning of every request opens a new transaction and the end of the request performs the transaction commit or rollback.

The problem is that the TransactionPhaseListener is enabled by default since your project has the Seam Faces, Seam Seam Transaction Persistence and thus the mere fact of placing these libs in your project already adds the behavior described above in the system, and most cases, has conflicting with the transaction control chosen to be used in the application. In my case, the TransactionPhaseListener behavior did not meet and worse, interfered. The solution I found was to disable this behavior through the following setting in beans.xml file:

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

With this setting in beans.xml the TransactionPhaseListener class will be ignored by the CDI and so is disabled, thus eliminating the problem in the transaction control and that annoying message (JBAS010152). In fact, when I got rid of this problem appeared others in my project because he was hiding some defects in my transaction control, then I recommend trying to validate its strategy after disable TransactionPhaseListener and make sure it is in working order.
