---
layout: post
title: "Adicionando Seam Faces no seu projeto Maven"
date: 2012-06-14 22:02
comments: true
categories: [JSF, Maven]
tags: [JSF, Seam Faces, Jboss Seam, Maven]
---

Nos tempos de Maven o título desse post parece até meio idiota, mas, a equipe do JBoss Seam 3 conseguiu tornar algo que deveria ser simples numa atividade dolorosa. Pois é, adicionar o Seam Faces como dependência do seu projeto Maven não é tão simples como é sugerido na <a href="http://seamframework.org/Seam3/FacesModule" target="_blank" title="Seam Faces">página oficial</a>. Lá eles mostram o que deve ser adicionado no seu pom.xml para obter o Seam Faces:

	<dependency>
	   <groupId>org.jboss.seam.faces</groupId>
	   <artifactId>seam-faces-api</artifactId>
	   <version>3.1.0.Final</version>
	</dependency>
	
	<dependency>
	   <groupId>org.jboss.seam.faces</groupId>
	   <artifactId>seam-faces</artifactId>
	   <version>3.1.0.Final</version>
	   <scope>runtime</scope>
	</dependency>

Não deixa de ser verdade, as libs do Seam Faces serão baixadas pelo Maven, porém, as suas dependências não, por isso que me propus a fazer este post. Se você executar seu projeto apenas com o trecho de código acima no seu pom.xml vai obter erros dos mais variados tipos indo de ClassNotFoundException até "Bean name is ambiguous", isso porque Seam Faces não baixa todas as suas dependências(pra que serve o Maven então?).

Bem, depois de muito tentar e pesquisar, cheguei a uma configuração mínima para que o Seam Faces possa ser adicionado no seu projeto sem causar erros. Vale observar que meus testes foram feitos no JBoss AS 7. Segue o pom:

	<project xmlns=="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<groupId>teste.seam</groupId>
		<artifactId>seam-faces</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<packaging>war</packaging>
	
		<repositories>
			<repository>
				<id>JBOSS_NEXUS</id>
				<url>http://repository.jboss.org/nexus/content/groups/public</url>
			</repository>
		</repositories>
	
	   <dependencyManagement>
	      <dependencies>
	
	        <dependency>
	            <groupId>org.jboss.spec</groupId>
	            <artifactId>jboss-javaee-web-6.0</artifactId>
	            <version>2.0.0.Final</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	      
	      </dependencies>
	   </dependencyManagement>
	
		<dependencies>
			<!-- CDI  -->		
			<dependency>
				<groupId>javax.enterprise</groupId>
				<artifactId>cdi-api</artifactId>
				<scope>provided</scope>
			</dependency>
			
			<!-- JSF  -->
			<dependency>
				<groupId>org.jboss.spec.javax.faces</groupId>
				<artifactId>jboss-jsf-api_2.0_spec</artifactId>
				<scope>provided</scope>
			</dependency>
	
			<!-- Servlet -->
			<dependency>
				<groupId>org.jboss.spec.javax.servlet</groupId>
				<artifactId>jboss-servlet-api_3.0_spec</artifactId>
				<scope>provided</scope>
			</dependency>
	
			<!-- Praticamente todos os modulos do Seam 3 precisam do Solder -->		
			<dependency>
				<groupId>org.jboss.solder</groupId>
				<artifactId>solder-impl</artifactId>
				<version>3.1.1.Final</version>
			</dependency>
	
			<!-- Seam Faces precisa do seam-international(mesmo que você não use para nada) -->		
			<dependency>
				<groupId>org.jboss.seam.international</groupId>
				<artifactId>seam-international</artifactId>
				<version>3.1.0.Final</version>
				<scope>compile</scope>
				<exclusions>
					<exclusion>
						<groupId>org.jboss.solder</groupId>
						<artifactId>solder-api</artifactId>
					</exclusion>
					<exclusion>
						<artifactId>solder-impl</artifactId>
						<groupId>org.jboss.solder</groupId>
					</exclusion>
				</exclusions>
			</dependency>
	
			<!-- Seam International precisa do joda-time -->		
			<dependency>
				<groupId>joda-time</groupId>
				<artifactId>joda-time</artifactId>
				<version>2.1</version>
				<scope>compile</scope>
			</dependency>
	
			<!-- Seam Faces precisa do seam-transaction(mesmo que você não use para nada) -->		
			<dependency>
				<groupId>org.jboss.seam.transaction</groupId>
				<artifactId>seam-transaction</artifactId>
				<version>3.1.0.Final</version>
				<exclusions>
					<exclusion>
						<groupId>org.jboss.solder</groupId>
						<artifactId>solder-api</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.jboss.solder</groupId>
						<artifactId>solder-impl</artifactId>
					</exclusion>
					<exclusion>
						<artifactId>jboss-servlet-api_3.0_spec</artifactId>
						<groupId>org.jboss.spec.javax.servlet</groupId>
					</exclusion>
				</exclusions>
			</dependency>
	
			<!-- Seam Faces precisa do seam-persistence(mesmo que você não use para nada) -->		
			<dependency>
				<groupId>org.jboss.seam.persistence</groupId>
				<artifactId>seam-persistence</artifactId>
				<version>3.1.0.Final</version>
				<exclusions>
					<exclusion>
						<groupId>org.jboss.solder</groupId>
						<artifactId>solder-api</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.jboss.solder</groupId>
						<artifactId>solder-impl</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.jboss.seam.transaction</groupId>
						<artifactId>seam-transaction</artifactId>
					</exclusion>				
					<exclusion>
						<groupId>org.jboss.seam.transaction</groupId>
						<artifactId>seam-transaction-api</artifactId>
					</exclusion>				
				</exclusions>
			</dependency>
	
			<!-- Seam Persistence precisa do Hibernate(mesmo que você não use para nada) -->		
			<dependency>
				<groupId>org.hibernate</groupId>
				<artifactId>hibernate-entitymanager</artifactId>
				<version>4.1.4.Final</version>
			</dependency>
	
			<!-- Seam Faces precisa do seam-security(mesmo que você não use para nada) -->		
			<dependency>
				<groupId>org.jboss.seam.security</groupId>
				<artifactId>seam-security</artifactId>
				<version>3.1.0.Final</version>
				<scope>compile</scope>
				<exclusions>
					<exclusion>
						<groupId>org.jboss.seam.persistence</groupId>
						<artifactId>seam-persistence</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.jboss.seam.solder</groupId>
						<artifactId>seam-solder</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.jboss.solder</groupId>
						<artifactId>solder-api</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.jboss.solder</groupId>
						<artifactId>solder-impl</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.jboss.seam.international</groupId>
						<artifactId>seam-international-api</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.jboss.seam.international</groupId>
						<artifactId>seam-international</artifactId>
					</exclusion>
				</exclusions>
			</dependency>
	
			<!-- Seam Faces precisa do prettyfaces(mesmo que você não use para nada) -->
			<dependency>
				<groupId>com.ocpsoft</groupId>
				<artifactId>prettyfaces-jsf2</artifactId>
				<version>3.3.3</version>
				<scope>compile</scope>
			</dependency>
	
			<!-- Seam Faces precisa do drools(mesmo que você não use para nada) -->
			<dependency>
				<groupId>org.drools</groupId>
				<artifactId>knowledge-api</artifactId>
				<version>5.4.0.Final</version>
				<scope>compile</scope>
			</dependency>
	
			<!-- Enfim, o Seam Faces -->
			<dependency>
				<groupId>org.jboss.seam.faces</groupId>
				<artifactId>seam-faces</artifactId>
				<version>3.1.0.Final</version>
				<scope>compile</scope>
				<exclusions>
					<exclusion>
						<groupId>org.jboss.solder</groupId>
						<artifactId>solder-api</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.jboss.solder</groupId>
						<artifactId>solder-impl</artifactId>
					</exclusion>
					<exclusion>
						<groupId>org.jboss.seam.international</groupId>
						<artifactId>seam-international-api</artifactId>
					</exclusion>
				</exclusions>
			</dependency>
			
		</dependencies>
	
	</project>


Perceba que eu usei bastante as tags de exclusão de artefatos, o caso é que o Solder, por exemplo, é baixado junto com vários módulos como o Seam Faces, Seam Internacional, Seam Transaction e etc. Fiz isso para evitar conflitos e explicitar para o leitor quais as dependências necessárias. Você pode remover algum desses "exclusions" e ir testando para vê se não vai haver problema.

Com esse pom, meu projeto web foi executado no JBoss AS 7 sem erros na inicialização e então pude usar o Seam Faces para fazer algumas coisas como:

	@Named
	@RequestScoped
	public class SeamFacesTeste {
	
	
		@Inject
		FacesContext facesContext;
	
		@Inject
		Flash flash;
	
		@Inject
		HttpServletRequest request;
	
		@Inject
		HttpServletResponse response;
	
	}


Bem é isso, espero que funcione de primeira. Uma dica que deixo é dar uma olhada no Seam BOM, é uma dependência do tipo pom que fornece todos os módulos do Seam 3 de uma forma mais organizada e você pode ir adicionando ao seu projeto a medida que for precisando.
