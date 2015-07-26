---
title: "EJB 3.1: Stateless Session Beans e a reinjeção de dependências"
date: 2013-09-04 22:50
comments: true
lang: pt-br
alias: blog/2013/09/04/ejb-3-dot-1-stateless-session-beans-e-a-reinjecao-de-dependencias/index.html
categories:
 - EJB
tags:
 - EJB
 - Stateless Session Bean
 - SLSB
 - Reinjeção de dependências
 - Injeção de dependências
---

Quando se começa a trabalhar com EJB uma das primeiras coisas a se aprender é sobre os diferentes tipos de beans, que na especificação 3.1 são: Stateful Session Beans (SFSB), Stateless Session Beans (SLSB), Singleton Sessions Beans e os Message-Drive Beans. Com isso vem uma das dúvidas mais comuns: "Qual a diferença entre beans Stateful e Stateless?". Bem, esse post não é mais um para explicar essas diferenças, se você precisa disso segue um bom <a target="_blank" href="http://www.theserverside.com/tutorial/Which-EJB-to-use-Stateful-stateless-and-singleton-session-beans-compared">link</a>.

<!-- more -->

Mas, para o meu post fazer sentido preciso colocar a resposta padrão: "SFSBs preservam o estado mesmo que ocorram várias chamadas de métodos no bean, pois, o container garante que o cliente vai ser sempre atendido pela mesma instância. Já os SLSBs não preservam estado, pois, para cada chamada de método que o cliente faz ele pode ser atendido por uma instância diferente."

Porém, com o advento do CDI e o seu poderoso mecanismo de injeção de dependências isso muda um pouco, pois, mesmo um SLSB pode ter um "pseudo conversational state" quando usa esse recurso. A maioria dos desenvolvedores (inclusive eu até hoje a tarde) acredita que a injeção de dependências de um EJB acontece apenas durante a criação da instância e com isso supõem coisas como: "Não posso injetar um managed bean @SessionScoped num SLSB, pois, como não tenho garantia de ser atendido sempre pela mesma instância posso obter uma que tem o managed bean de uma outra sessão".

Na verdade não é bem assim, descobri isso hoje e se soubesse antes teria me evitado uma grande dor de cabeça, por isso resolvi compartilhar aqui. A especificação EJB 3.1 na página 74 item 4.3.2 fala o seguinte:
> If a session bean makes use of dependency injection, the container injects these references after the bean instance is created, and before any business methods are invoked on the bean instance.

Ou seja, a injeção de dependências é feita depois de criar a instância do bean e também antes de cada chamada de um método de negócio. Isso foi de grande valia para mim, pois, já estava quebrando a cabeça em como manteria o EntityManager que injeto nos meus EJBs (através do Seam Managed Persistence Context meu EntityManager é um @ConversationScoped) de acordo com o contexto do cliente do EJB. Como o container reinjeta as dependências antes de cada chamada de método na verdade eu nem precisa ter me preocupado com isso.

De alguma forma, essa definição pode dar aos SLSBs a capacidade de ter um "estado de conversação". Por exemplo, se você injetar em um SLSB um bean com escopo de sessão (@SessionScoped), independentemente de qual instância do pool processar sua solicitação o bean com escopo de sessão sempre estará de acordo com a sessão do cliente atual. Em outras palavras, não existe a possibilidade de um SLSB ter um bean de sessão de outro cliente. Isso permite que uma SLSB receba o usuário logado como um campo de instância através de @Inject sem causar nenhum problema caso dois usuários usem a mesma instância do SLSB alternadamente. Por exemplo:

	@Stateless
	public class StatelessSessionBean{
	    @Inject
	    @LoggedInUser
	    protected User loggedInUser;//@SessionScoped

	    public void test(){
	        System.out.println("ejb:"+this);
	        System.out.println("user:"+this.loggedInUser);
	    }
	}

Então o resultado de várias chamadas do método "test" por usuários diferentes é a seguinte:

	User 1 invoke
	17:02:20,800 INFO  [stdout] (http--0.0.0.0-8080-5) ejb:StatelessSessionBean@406189
	17:02:20,801 INFO  [stdout] (http--0.0.0.0-8080-5) user:User 1

	User 2 invoke
	17:02:56,227 INFO  [stdout] (http--0.0.0.0-8080-8) ejb:StatelessSessionBean@406189
	17:02:56,228 INFO  [stdout] (http--0.0.0.0-8080-8) user:User 2

	User 2 invoke
	17:03:24,376 INFO  [stdout] (http--0.0.0.0-8080-8) ejb:StatelessSessionBean@406189
	17:03:24,378 INFO  [stdout] (http--0.0.0.0-8080-8) user:User 2

	User 1 invoke
	17:03:24,517 INFO  [stdout] (http--0.0.0.0-8080-6) ejb:StatelessSessionBean@1c05227
	17:03:24,518 INFO  [stdout] (http--0.0.0.0-8080-6) user:User 1

	User 1 invoke
	17:04:24,045 INFO  [stdout] (http--0.0.0.0-8080-1) ejb:StatelessSessionBean@406189
	17:04:24,047 INFO  [stdout] (http--0.0.0.0-8080-1) user:User 1

	User 2 invoke
	17:04:24,179 INFO  [stdout] (http--0.0.0.0-8080-8) ejb:StatelessSessionBean@1c05227
	17:04:24,179 INFO  [stdout] (http--0.0.0.0-8080-8) user:User 2

Perceba que mesmo sendo atendido pela mesma instância, o campo "loggedInUser" varia corretamente de acordo com o usuário que invocou o método.
