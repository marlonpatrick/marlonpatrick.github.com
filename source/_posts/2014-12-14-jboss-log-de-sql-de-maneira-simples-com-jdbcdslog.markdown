---
layout: post
title: "JBOSS: Log de SQL de maneira simples com jdbcdslog"
date: 2014-12-14 17:36
categories: [JBOSS, SQL, Log, Hibernate]
tags: [JBOSS, Log, SQL, jdbcdslog, Hibernate, JPA] 
---

Uma necessidade básica em quase todos os projetos de software é o log de instruções SQL e para esse fim existe uma infinidade de opções. No entanto, apesar das várias alternativas, muitas vezes o log não é feito da melhor forma:

- Excesso de log: são logadas coisas demais como consultas, abertura e fechamento de conexões, abertura e fechamento de resultsets e etc e o programador não tem a possibilidade de configurar o que deve ou não ser logado.
- Falta de integração com frameworks de logging: a lib atende bem a os requisitos de Log de SQL, porém, não se integra com frameworks de log como log4j, java logging, apache commons logging e etc o que acaba obrigando o desenvolvedor a criar um arquivo de log específico para SQL.
- Dificuldades para se trabalhar com DataSources, em especial, XA-DataSources: a ferramenta de log atende bem os dois requisitos acima, porém, é limitada a ser usada com objetos java.sql.Connection, ou em outros casos, podem ser usados com DataSources "normais", porém, não atendem quando precisamos logar sql de XA-DataSources.

Para quem usa o Hibernate a primeira escolha normalmente é configurá-lo para logar as queries geradas, o resultado é algo como:

	select this_.code from employee this_ where this_.code=?
	INSERT INTO stock_transaction (CHANGE, CLOSE, DATE, OPEN, STOCK_ID, VOLUME) VALUES (?, ?, ?, ?, ?, ?)

Nesse caso temos dois problemas, o primeiro é que os valores dos filtros e colunas não são logados, é mostrado apenas o caracter **?** indicando que foi substituído por um parâmetro. O outro problema é que nem sempre as consultas são realizadas pelo Hibernate, nesses casos, o log não seria mostrado nem mesmo o log sem parâmetros.

Pois bem, depois de testar algumas libs de log de sql encontrei uma que me atendeu em todos esses aspectos: controle do que será logado, integração com os frameworks de log mais populares, funcionar com Connection, DataSource e XA-Datasource, logar o sql com todos os seus parâmetros de modo que eu possa copiar e colar numa ferramenta de acesso a banco e executar o script, e por fim, logar qualquer sql que se faça através da conexão monitorada seja feita pelo Hibernate ou por qualquer outro modo. O nome da lib é <a href="https://code.google.com/p/jdbcdslog/" title="jdbcdslog" target="_blank">jdbcdslog</a>.

O exemplo mais básico é logar as queries de um simples objeto Connection:

	public static void main(String[] args) throws Exception {
		Class.forName("org.jdbcdslog.DriverLoggingProxy");
		
		String url = "jdbc:jdbcdslog:postgresql://172.16.1.6:5432/mpinfo;targetDriver=org.postgresql.Driver";
		Connection con = DriverManager.getConnection(url, "mpinfo", "mpinfo");
		
		PreparedStatement ps = con.prepareStatement("select * from product where id=?");
		ps.setLong(1, 1);
		ps.execute();
		
		con.close();
	}

Nesse exemplo estou acessando um banco PostgreSQL e perceba que a URL fica um pouco diferente do padrão pois preciso acrescentar o driver do Jdbcdslog como proxy para o driver original do Postgre. Você pode perceber isso em **jdbc:jdbcdslog:postgresql** e também no parâmetro targetDriver. O resultado desse código é algo como:

	[main] INFO org.jdbcdslog.StatementLogger - select * from ceos.product where id=1;

Mas o que quero demonstrar mesmo aqui nesse post é como usar essa lib para logar sql de datasources no JBoss, aqui no meu ambiente vou usar o JBoss EAP 6.2, mas acredito que a partir da versão do JBoss 7 não muda quase nada. Para isso vamos criar módulos no JBoss com o driver do Postgre e também do jdbcdslog.

1 - Primeiro devemos criar a pasta JBOSS_HOME/modules/system/layers/base/org/postgresql/main/. Dentro dessa pasta colocaremos o jar do Postgre de acordo com a versão que você usa, por exemplo, postgresql-9.3-1101-jdbc41.jar. Além disso devemos criar um arquivo chamado module.xml com o seguinte conteúdo:

	<module xmlns="urn:jboss:module:1.1" name="org.postgresql">
	    <resources>
	        <resource-root path="postgresql-9.3-1101-jdbc41.jar"/>
	    </resources>
	    <dependencies>
	        <module name="javax.api"/>
	        <module name="javax.transaction.api"/>
	    </dependencies>
	</module>

Esse procedimento deverá ser feito para qualquer banco de dados que você queira logar as queries, seja Postgre, Oracle, MySQL etc. A lógica é a mesma, basta prestar atenção nos nomes dos arquivos e na pasta o qual serão colocados. A sugestão é utilizar o nome do pacote principal do jar, no caso do Postgre é o org.postgresql.

2 - Agora é a vez de criamos o módulo do jdbcdslog. Para isso criamos a pasta JBOSS_HOME/modules/system/layers/base/com/googlecode/usc/jdbcdslog/main/. Nessa pasta irems colocar o jar do jdbcdslog, por exemplo, jdbcdslog-1.0.6.2.jar. Também precisaremos criar um arquivo chamado module.xml com o seguinte conteúdo:

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

Perceba que a dependência do módulo do Postgre é opcional. Será necessário incluir os módulos de outros bancos de dados (Oracle, Mysql) como dependência do módulo do jdbcdslog caso você queira fazer o log deles também.

3 - Agora abra o arquivo JBOSS_HOME/standalone/configuration/standalone.xml e configure os logs do jdbcdslog. Procure a tag do subsistema de log, algo como `<subsystem xmlns="urn:jboss:domain:logging:1.3">`, dentro dessa tag coloque o seguinte conteúdo:

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

Das várias opções de log disponíveis, a única que deixamos ativa foi o log de statements, ou seja, log de comandos DML e DDL.

OBS:Caso seu JBoss esteja executando em modo domínio ao invés de standalone então essa configuração pode ser feita no arquivo JBOSS_HOME/domain/configuration/domain.xml

4 - Ainda no arquivo standalone.xml vamos criar o datasource. Dentro da tag `<datasources>` coloque o seguinte conteúdo:

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

Nesse ponto você deve prestar atenção na tag `<connection-url>` e substituir os valores de acordo com o seu ambiente, além disso, perceba que na tag `<driver>` usamos o nome do driver do jdbcdslog que definiremos no próximo passo.

5 - Dentro da tag `<datasources>` acrescentar a definição do driver que faz o log:

	<drivers>
		<driver name="jdbcdslog" module="com.googlecode.usc.jdbcdslog">
			<driver-class>org.jdbcdslog.DriverLoggingProxy</driver-class>
			<xa-datasource-class>org.jdbcdslog.XADataSourceProxy</xa-datasource-class>
		</driver>
	</drivers>

Pronto, com isso já poderemos usar o datasource e verificar no log do JBoss que qualquer select, update, insert etc que for feito através das conexões desse datasource serão logadas, a saída será algo como:

	11:35:00,066 INFO  [org.jdbcdslog.StatementLogger] (EJB default - 6) select u.password from user u where u.login='admin';

Se você usa um XA-DataSource talvez se depare com o seguinte erro:

	Caused by: javax.resource.ResourceException: IJ000453: Unable to get managed connection for java:/MPINFO-XA-Datasource

	Caused by: java.lang.LinkageError: loader constraint violation: loader (instance of <bootloader>) previously initiated loading for a different type with name "javax/transaction/xa/XAResource"

A exceção acontece devido a um problema de ClassLoader no JBoss, a questão é que existem duas definições para a classe javax.transaction.xa.XAResource: uma que fica no pacote padrão da JVM e outra que fica num jar dentro do JBoss. Então, por algum motivo, o JBoss carrega a classe a partir da sua lib e o jdbcdslog carrega a classe a partir da lib padrão da JVM (rt.jar) e com isso ocorre a exceção alegando que uma classe com esse mesmo nome já havia sido carregada. Pois bem, para resolver isso ajustei uma configuração no JBoss para evitar que ele carregue a classe que está contida nas suas próprias libs e carregue somente a classe a partir da JVM. O procedimento consiste em alterar o conteúdo do arquivo JBOSS_HOME/modules/system/layers/base/javax/transaction/api/main/module.xml:

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

Recomendo que você salve o arquivo original antes de fazer qualquer modificação, ele provavelmente não terá a tag `<dependencies>` e também não terá a tag `<filter>`. Com esse ajuste terminamos todas as configurações necessárias, agora é testar e verificar que o log gerado está de acordo com os comandos executados.

<strong>1 João 4.20:</strong>Se alguém afirmar: "Eu amo a Deus", mas odiar seu irmão, é mentiroso, pois quem não ama seu irmão, a quem vê, não pode amar a Deus, a quem não vê.
