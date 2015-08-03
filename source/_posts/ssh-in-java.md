---
title: "SSH in Java"
date: 2012-03-18 21:22
comments: true
lang: en
categories:
 - JSCH
tags:
 - SSH
 - Java
 - JSCH
---

Java + SSH = JSCH

I do not know if you ever needed to access a machine using SSH through a Java code, but if you need's the trick: jsch. Jsch is an api (pure Java) fairly complete that implements the SSH2, you do not need no native lib or anything like that, just include lib jsch in your project and READY.

<!-- more -->

To make an analogy, the jsch is a Putty in Java that you can execute commands programmatically in the remote machine, for example, a remote-to-local and local-to-remote scp. As Putty, the jsch can access machines Linux via Windows smoothly!

I will leave here an example quite simple:

1 - First you need to create a class to store user information. This requires implementing interface com.jcraft.jsch.UserInfo.

	public class MPInfoUser implements UserInfo{

		@Override
		public String getPassword(){ 
			return "123456"; 
		}

		@Override
		public boolean promptYesNo(String str){
			return true;
		}

		@Override
		public void showMessage(String message){

		}

		@Override
		public String getPassphrase() {
			return null;
		}

		@Override
		public boolean promptPassphrase(String paramString) {
			return true;
		}

		@Override
		public boolean promptPassword(String message){
			return true;
		}	
	}

2 - Then you can just connect to the remote machine

	public class TesteJschMPInfo {

		public static void main(String[] args) throws Exception {
			try{
				JSch jsch=new JSch();
				Session session = jsch.getSession("root", "192.168.1.1", 22);
				session.setUserInfo(new MPInfoUser());
				session.connect();
				
				ChannelExec channel = (ChannelExec) session.openChannel("exec");
				channel.setCommand("echo \"testing...\" >> testmpinfo;");
				channel.connect();
				channel.disconnect();
				
			}catch(Exception e){
				e.printStackTrace();
			}
			
		}
	}
			
I have often used this api at work and strongly recommend, very simple and complete, in addition, you can find several examples <a href="http://www.jcraft.com/jsch/examples/" target="_blank" title="JSCH Examples">here</a>. Hugs!
