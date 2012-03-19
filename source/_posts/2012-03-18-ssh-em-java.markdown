---
layout: post
title: "SSH em Java"
date: 2012-03-18 21:22
comments: true
categories: [API's Java]
tags: [SSH, Java]
---

Java + SSH = JSCH

N&atilde;o sei se voc&ecirc;s j&aacute; precisaram acessar alguma m&aacute;quina usando SSH atrav&eacute;s de um c&oacute;digo Java, mas, se precisarem vai a dica: 
<a href="http://www.jcraft.com/jsch/" target="_blank" title="JSCH">JSCH</a>. JSCH &eacute; uma api (pure Java) bastante completa 
que implementa o SSH2, voc&ecirc; n&atilde;o precisar&aacute; de nenhuma lib nativa ou qualquer coisa parecida, 
basta apenas incluir a lib do JSCH no seu projeto e PRONTO.

Fazendo uma analogia, o JSCH &eacute; um Putty em Java que voc&ecirc; pode executar comandos programaticamente na m&aacute;quina remota, como por exemplo,
um scp remote-to-local ou local-to-remote. Assim como o Putty, o JSCH pode acessar m&aacute;quinas Linux via Windows sem problemas!

Vou deixar aqui um exemplo bem simples:

1 - Primeiro voc&ecirc; precisa criar uma classe para guardar as informa&ccedil;&otilde;es do usu&aacute;rio. Para isso &eacute; necess&aacute;rio
implementar a interface com.jcraft.jsch.UserInfo.

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

2 - Depois, &eacute; s&oacute; conectar na m&aacute;quina remota

	public class TesteJschMPInfo {

		public static void main(String[] args) throws Exception {
			try{
				JSch jsch=new JSch();
				Session session = jsch.getSession("root", "192.168.1.1", 22);
				session.setUserInfo(new MPInfoUser());
				session.connect();
				
				ChannelExec channel = (ChannelExec) session.openChannel("exec");
				channel.setCommand("echo \"testando...\" >> testempinfo;");
				channel.connect();
				channel.disconnect();
				
			}catch(Exception e){
				e.printStackTrace();
			}
			
		}
	}
			
				
Tenho usado bastante essa api no trabalho e recomendo fortemente, muito simples e completa, al&eacute;m disso, voc&ecirc;s podem encontrar diversos exemplos 
<a href="http://www.jcraft.com/jsch/examples/" target="_blank" title="Exemplos JSCH">aqui</a>.Abra&ccedil;os!