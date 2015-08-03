---
title: "Seam Persistence: Configuring FlushMode"
date: 2012-11-02 20:19
comments: true
lang: en
categories:
 - JBoss Seam
tags:
 - Seam Persistence
 - Seam Managed Persistence Context
---

One of the great advantages of using Seam Persistence is the Seam Managed Persistence Context (SMPC) which allows better management of the EntityManager in your application. With the SMPC is possible to place the EntityManager in a CDI scope which can greatly ease the life of developers helping us to sharply minimize errors as the dreaded LazyInitializationException.

<!-- more -->

But one of the things I had trouble finding in the Seam Persistence documentation was setting the FlushMode of my EntityManager so I decided to make this post. Giving poke around in Seam code, I found the FlushModeManager class and its standard FlushModeManagerImpl implementation, which are just used to set the flushMode in the EntityManager managed by SMPC.

By default, Seam Persistence defines flushMode to AUTO. With flushMode the problem set to AUTO is that the developer has no control over the time the ORM will synchronize the in-memory objects with the database, which can often produce unexpected results. If you agree with me and want to change the flushMode used by SMPC Simply copy the code below:

	@Produces
	@ApplicationScoped
	protected FlushModeManager createDefaultFlushModeManager() {
		FlushModeManagerImpl flushModeManager = new FlushModeManagerImpl();
		flushModeManager.setFlushModeType(FlushModeType.COMMIT);
		return flushModeManager;
	}

This method will produce a CDI bean of type FlushModeManager to be used by SMPC to set the flushMode in the EntityManager. The new bean will override the default bean created by Seam Persistence, and with it, you can set the flushMode in three ways: AUTO, MANUAL and COMMIT.
