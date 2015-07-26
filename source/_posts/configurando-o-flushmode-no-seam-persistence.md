---
title: "Seam Persistence: Configurando o FlushMode"
date: 2012-11-02 20:19
comments: true
lang: pt-br
alias: blog/2012/11/02/configurando-o-flushmode-no-seam-persistence/index.html
categories:
 - JBoss Seam
tags:
 - JBoss Seam
 - Seam Persistence
 - Seam Managed Persistence Context
 - FlushMode
 - ORM
 - JPA
 - EntityManager
---

Uma das grandes vantagens de usar o Seam Persistence é o Seam Managed Persistence Context (SMPC) que permite um melhor gerenciamento do EntityManager na sua aplicação. Com o SMPC é possível colocar o EntityManager num escopo CDI o que pode facilitar bastante a vida de nós desenvolvedores ajudando a minimizar bruscamente erros como o temido LazyInitializationException.

<!-- more -->

Porém, uma das coisas que tive dificuldade de encontrar na documentação do Seam Persistence foi como definir o FlushMode do meu EntityManagere por isso resolvi fazer esse post. Dando uma fuçada no código do Seam, encontrei a classe FlushModeManager e sua implementação padrão FlushModeManagerImpl, que justamente são usadas para definir o flushMode do EntityManager gerenciado pelo SMPC.

Por padrão, o Seam Persistence define o flushMode como AUTO. O problema do flushMode configurado como AUTO é que o desenvolvedor não tem domínio sobre o momento que o ORM irá sincronizar os objetos em memória com a base de dados, o que muitas vezes, pode gerar resultados inesperados. Se você concorda comigo e deseja mudar o flushMode usado pelo SMPC é só copiar o código abaixo:

	@Produces
	@ApplicationScoped
	protected FlushModeManager createDefaultFlushModeManager() {
		FlushModeManagerImpl flushModeManager = new FlushModeManagerImpl();
		flushModeManager.setFlushModeType(FlushModeType.COMMIT);
		return flushModeManager;
	}

Esse método irá produzir um bean CDI do tipo FlushModeManager que será usado pelo SMPC para definir o flushMode do EntityManager que ele irá gerenciar. O novo bean irá substituir o bean padrão criado pelo Seam Persistence, e com ele, você pode definir o flushMode de três maneiras: AUTO, COMMIT e MANUAL. Vlw!
