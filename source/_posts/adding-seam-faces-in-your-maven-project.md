---
title: "Adding Seam Faces in your Maven project"
date: 2012-06-14 22:02
comments: true
lang: en
categories:
 - JBoss Seam 
tags:
 - JSF
 - Seam Faces
 - Maven
---

In Maven times the title of this post seems to kind of stupid, but the team JBoss Seam 3 managed to make something that should be simple in a painful activity. Well, add the Seam Faces as a dependency of your Maven project is not as simple as suggested in <a target="_blank" href="http://seamframework.org/Seam3/FacesModule" title="Seam Faces">official website</a>. There they show what should be added to your pom.xml for the Seam Faces:

<!-- more -->

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

It is still true the libs Seam Faces will be downloaded by Maven, however, its dependencies not, so I set out to make this post. If you run your project with just the above code snippet in your pom.xml will get errors of all kinds going ClassNotFoundException to "Bean name is ambiguous", that because Seam Faces not low all its dependencies (for what serving Maven then?).

Well, after much trying and searching, I came to a minimal configuration so that the Seam Faces can be added to your project without causing errors. It is worth noting that my tests were done on JBoss AS 7. Follow the pom:

	<project xmlns=="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<groupId>test.seam</groupId>
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
	
			<!-- Virtually all of Seam 3 modules need the Solder -->
			<dependency>
				<groupId>org.jboss.solder</groupId>
				<artifactId>solder-impl</artifactId>
				<version>3.1.1.Final</version>
			</dependency>
	
			<!-- Seam Faces needs seam-international (even if you do not use for anything) -->	
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
	
			<!-- Seam International needs joda-time -->		
			<dependency>
				<groupId>joda-time</groupId>
				<artifactId>joda-time</artifactId>
				<version>2.1</version>
				<scope>compile</scope>
			</dependency>
	
			<!-- Seam Faces needs seam-transaction (even if you do not use for anything) -->	
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
	
			<!-- Seam Faces needs seam-persistence (even if you do not use for anything) -->
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
	
			<!-- Seam Persistence needs Hibernate (even if you do not use for anything) -->	
			<dependency>
				<groupId>org.hibernate</groupId>
				<artifactId>hibernate-entitymanager</artifactId>
				<version>4.1.4.Final</version>
			</dependency>
	
			<!-- Seam Faces needs seam-security (even if you do not use for anything) -->
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
	
			<!-- Seam Faces needs prettyfaces (even if you do not use for anything) -->
			<dependency>
				<groupId>com.ocpsoft</groupId>
				<artifactId>prettyfaces-jsf2</artifactId>
				<version>3.3.3</version>
				<scope>compile</scope>
			</dependency>
	
			<!-- Seam Faces needs drools (even if you do not use for anything) -->
			<dependency>
				<groupId>org.drools</groupId>
				<artifactId>knowledge-api</artifactId>
				<version>5.4.0.Final</version>
				<scope>compile</scope>
			</dependency>
	
			<!-- At last, Seam Faces -->
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

Note that I used quite the artifacts exclusion tags, the case is that the Solder, for example, is downloaded along with several modules such as Seam Faces, Seam International, Seam Transaction and etc. I did it to avoid conflicts and explain to the reader which dependencies are necessary. You can remove any of these "exclusions" and go testing to see if there will be no problem.

With this pom, my web project was run on JBoss AS 7 without errors at startup and then I could use Seam Faces to do some things as:

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


Well that's it, I hope it works in first attempt. One tip I leave is to check out Seam BOM, it provides all modules of Seam 3 in a more organized way and you can adding to your project as you need.
