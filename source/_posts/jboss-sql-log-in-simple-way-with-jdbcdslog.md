---
title: "JBOSS: SQL log in a simple way with jdbcdslog"
date: 2014-12-14 17:36
lang: en
categories:
 - JBOSS
 - Log
tags:
 - JBOSS
 - Log
 - SQL
 - jdbcdslog
 - Hibernate
 - JPA
---

A basic necessity in almost every software project is the log of SQL statements and for this purpose there is a plethora of options. However, despite the many alternatives, the log is often not made the best way:

<!-- more -->

- Excessive log: too much is logged (queries, opening and closing connections, opening and closing resultsets) and the programmer does not have the possibility to configure what should or should not be logged.
- Lack of integration with logging frameworks: the lib caters well to the SQL Log requirements, however, does not integrate with logging frameworks (log4j, java logging, apache commons logging) which ultimately forcing the developer to create a specific log file to SQL.
- Problems to work with DataSources, in particular XA-DataSources: a well logging tool meets the above two requirements, however, is limited to use with java.sql.Connection objects, or in other cases may be used with "normal" DataSources, however, do not meet when we need to log sql XA-DataSource.

For those who use Hibernate first choice is usually set it to log the generated queries, the result is something like:

	select this_.code from employee this_ where this_.code=?
	INSERT INTO stock_transaction (CHANGE, CLOSE, DATE, OPEN, STOCK_ID, VOLUME) VALUES (?, ?, ?, ?, ?, ?)

In this case we have two problems, the first is that the values of the filters and columns are not logged in, only the character **?** is shown indicating that was replaced by a parameter. The other problem is that queries are not always carried out by Hibernate, in such cases the log would not be displayed, not even the log without parameters.

Well, after testing some sql log libs found one that met me in all these aspects: control of what is logged, integration with the most popular logging frameworks, work with Connection, DataSource and XA-DataSource, log sql with all its parameters so that I can copy and paste in the database access tool and run the script, and finally, log any sql that be through the monitored connection is made by Hibernate or otherwise. The lib name is <a href="https://code.google.com/p/jdbcdslog/" title="jdbcdslog" target="_blank">jdbcdslog</a>.

The most basic example is logging the queries of a single Connection object:

	public static void main(String[] args) throws Exception {
		Class.forName("org.jdbcdslog.DriverLoggingProxy");
		
		String url = "jdbc:jdbcdslog:postgresql://172.16.1.6:5432/mpinfo;targetDriver=org.postgresql.Driver";
		Connection con = DriverManager.getConnection(url, "mpinfo", "mpinfo");
		
		PreparedStatement ps = con.prepareStatement("select * from product where id=?");
		ps.setLong(1, 1);
		ps.execute();
		
		con.close();
	}

In this example I am accessing a PostgreSQL database and notice that the URL is a little different from the norm because jdbcdslog must add the driver as a proxy for the original driver Postgres. You can see this in **jdbc:jdbcdslog:postgresql** and also in **targetDriver** parameter. The result of this code is something like:

	[main] INFO org.jdbcdslog.StatementLogger - select * from ceos.product where id=1;

But what I want to demonstrate even here in this post is how to use this lib to log sql datasources in JBoss, here in my enviroment I will use the JBoss EAP 6.2, but I think from the version of JBoss 7 does not change almost nothing. To do this we create modules to JBoss with the Postgres driver and also jdbcdslog.

1 - First we must create the folder JBOSS_HOME/modules/system/layers/base/org/postgresql/main/. Inside that folder put the Postgres jar according to the version you use, for example, postgresql-9.3-1101-jdbc41.jar. In addition we must create a file called module.xml with the following content:

	<module xmlns="urn:jboss:module:1.1" name="org.postgresql">
	    <resources>
	        <resource-root path="postgresql-9.3-1101-jdbc41.jar"/>
	    </resources>
	    <dependencies>
	        <module name="javax.api"/>
	        <module name="javax.transaction.api"/>
	    </dependencies>
	</module>

This procedure should be done for any database you want to log the queries, either Postgres, Oracle, MySQL etc. The logic is the same, just pay attention to the names of the files and folder which will be placed. The suggestion is to use the name of the main jar package in the case of Postgres is org.postgresql.

2 - Now is the time to create the jdbcdslog module. For this we create the folder JBOSS_HOME/modules/system/layers/base/com/googlecode/usc/jdbcdslog/main/. In this folder we will put the jdbcdslog jar, for example, jdbcdslog-1.0.6.2.jar. We also need to create a file called module.xml with the following content:

	<module xmlns="urn:jboss:module:1.1" name="com.googlecode.usc.jdbcdslog">
	    <resources>
	        <resource-root path="jdbcdslog-1.0.6.2.jar"/>
	    </resources>
	    <dependencies>
	        <module name="org.slf4j"/>
	        <module name="javax.api"/>
	        <module name="javax.transaction.api"/>
		<module name="org.postgresql" optional="true"/>
	    </dependencies>
	</module>

Note that the Postgres module dependency is optional. It must include modules from other databases (Oracle, MySQL) as module dependency of jdbcdslog if you want to log them as well.

3 - Now open the file JBOSS_HOME/standalone/configuration/standalone.xml and configure the jdbcdslog logs. Look for the log subsystem tag, something like `<subsystem xmlns="urn:jboss:domain:logging:1.3">` within this tag place the following contents:

            <logger category="org.jdbcdslog.StatementLogger">
                <level name="DEBUG"/>
            </logger>
            <logger category="org.jdbcdslog.ConnectionLogger">
                <level name="OFF"/>
            </logger>
            <logger category="org.jdbcdslog.ResultSetLogger">
                <level name="OFF"/>
            </logger>
            <logger category="org.jdbcdslog.SlowQueryLogger">
                <level name="OFF"/>
            </logger>

The various logging options available, the only one left active was the log statements, ie DML and DDL commands log.

NOTE: If your JBoss is running in domain mode rather than standalone then this setting can be made to the file JBOSS_HOME/domain/configuration/domain.xml.

4 - Still in standalone.xml file we will create the datasource. Below are two examples, one for a normal datasource and one for an XA-DataSoure, both should stay within the `<datasources>` tag:

	<datasource jta="true" jndi-name="java:/mpinfods" pool-name="MPINFO-POOL" enabled="true" use-ccm="true">
	    <connection-url>jdbc:jdbcdslog:postgresql://172.16.1.6:5432/mpinfo;targetDriver=org.postgresql.Driver</connection-url>
	    <driver>jdbcdslog</driver>
	    <new-connection-sql>select 1</new-connection-sql>
	    <transaction-isolation>TRANSACTION_READ_COMMITTED</transaction-isolation>
	    <pool>
		<min-pool-size>10</min-pool-size>
		<max-pool-size>20</max-pool-size>
	    </pool>
	    <security>
		<user-name>mpinfo</user-name>
		<password>mpinfo</password>
	    </security>
	    <validation>
		<validate-on-match>false</validate-on-match>
		<background-validation>false</background-validation>
		<background-validation-millis>1</background-validation-millis>
	    </validation>
	    <statement>
		<prepared-statement-cache-size>32</prepared-statement-cache-size>
		<share-prepared-statements>false</share-prepared-statements>
	    </statement>
	</datasource>

	<xa-datasource jndi-name="java:/mpinfoxads" pool-name="MPINFO-XA-POOL" enabled="true" use-ccm="true">
		<xa-datasource-property name="PortNumber">5432</xa-datasource-property>
		<xa-datasource-property name="DatabaseName">mpinfo</xa-datasource-property>
		<xa-datasource-property name="ServerName">172.16.1.6</xa-datasource-property>
		<xa-datasource-property name="targetDS">org.postgresql.xa.PGXADataSource</xa-datasource-property>
		<driver>jdbcdslog</driver>
		<new-connection-sql>select 1</new-connection-sql>
		<transaction-isolation>TRANSACTION_READ_COMMITTED</transaction-isolation>
		<xa-pool>
			<min-pool-size>0</min-pool-size>
			<max-pool-size>10</max-pool-size>
			<flush-strategy>IdleConnections</flush-strategy>
			<is-same-rm-override>false</is-same-rm-override>
			<interleaving>false</interleaving>
			<pad-xid>false</pad-xid>
			<wrap-xa-resource>false</wrap-xa-resource>
		</xa-pool>
		<security>
			<user-name>mpinfo</user-name>
			<password>mpinfo</password>
		</security>
		<validation>
			<valid-connection-checker class-name="org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLValidConnectionChecker"/>
			<validate-on-match>true</validate-on-match>
			<background-validation>false</background-validation>
			<exception-sorter class-name="org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLExceptionSorter"/>
		</validation>
		<timeout>
			<idle-timeout-minutes>3</idle-timeout-minutes>
		</timeout>
		<statement>
			<track-statements>true</track-statements>
			<share-prepared-statements>false</share-prepared-statements>
		</statement>
	</xa-datasource>

At this point you should pay attention to information as IPs, users, passwords and replace according to your environment, moreover, realize that the tag `<driver>` use the jdbcdslog driver name that will define the next step.

5 - Within the `<datasources>` tag (standalone.xml) add a driver setting that makes the log:

	<drivers>
		<driver name="jdbcdslog" module="com.googlecode.usc.jdbcdslog">
			<driver-class>org.jdbcdslog.DriverLoggingProxy</driver-class>
			<xa-datasource-class>org.jdbcdslog.XADataSourceProxy</xa-datasource-class>
		</driver>
	</drivers>

There, it can already use the datasource and check the log JBoss any select, update, insert which is done through the connections that datasource will be logged, the output will look like:

	11:35:00,066 INFO  [org.jdbcdslog.StatementLogger] (EJB default - 6) select u.password from user u where u.login='admin';

If you use an XA-DataSource may be confronted with the following error:

	Caused by: javax.resource.ResourceException: IJ000453: Unable to get managed connection for java:/MPINFO-XA-Datasource

	Caused by: java.lang.LinkageError: loader constraint violation: loader (instance of <bootloader>) previously initiated loading for a different type with name "javax/transaction/xa/XAResource"

The exception is due to a ClassLoader problem in JBoss, the issue is that there are two definitions for javax.transaction.xa.XAResource class: one that is in the standard JVM package and one that is in a jar inside the JBoss. Then, for some reason, JBoss loads the class from your lib and jdbcdslog loads the class from the standard JVM lib (rt.jar) and this is the exception claiming a class with the same name already have been loaded. Well, to solve this adjusted a setting in JBoss to prevent it from load the class that is contained in their own libs and load only the class from the JVM. The procedure is to change the contents of the file JBOSS_HOME/modules/system/layers/base/javax/transaction/api/main/module.xml:

	<module xmlns="urn:jboss:module:1.1" name="javax.transaction.api">
		<resources>
			<resource-root path="jboss-transaction-api_1.1_spec-1.0.1.Final-redhat-2.jar">
				<filter>
					<exclude path="javax/transaction/xa"/>
					<include path="**"/>
				</filter>
			</resource-root>
		</resources>
		<dependencies>
			<system export="true">
				<paths>
					<path name="javax/transaction/xa"/>
				</paths>
			</system>
		</dependencies>
	</module>

I recommend that you save the original file before making any change, it probably will not have the tag `<dependencies>` and also will not have the tag `<filter> '. With this setting finish all necessary settings, now test and verify that the log is generated in accordance with the executed commands.
