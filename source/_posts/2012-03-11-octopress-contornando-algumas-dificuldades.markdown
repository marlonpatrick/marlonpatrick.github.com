---
layout: post
title: "Octopress: Contornando algumas dificuldades"
date: 2012-03-11 23:51
comments: true
categories: [Octopress]
tags: [Octopress]
---

Bem, como j&aacute; tem v&aacute;rios posts em outros blogs com tutoriais de como instalar e configurar o Octopress al&eacute;m da 
pr&oacute;pria documenta&ccedil;&atilde;o ser muito clara, eu resolvi fazer esse post para tratar de alguns probleminhas pouco mensionados:

**1 - RVM/rbenv para usu&aacute;rios Windows**: No site do <a href="http://octopress.org/" title="Octopress" target="_blank">Octopress</a> 
voc&ecirc; &eacute; instru&iacute;do a instalar o <a href="http://rvm.beginrescueend.com/" title="RVM" target="_blank">RVM</a> ou o 
<a href="https://github.com/sstephenson/rbenv" title="rbenv" target="_blank">rbenv</a>, ferramentas estas que s&oacute; 
est&atilde;o dispon&iacute;veis para Linux. No entanto, eu fiz toda a configura&ccedil;&atilde;o sem elas, ent&atilde;o, pode seguir em frente. Pelo pouco que vi 
servem apenas para dizer: "Ruby, eu quero que voc&ecirc; execute como se fosse a vers&atilde;o XYZ", nesse caso, baixei a vers&atilde;o do 
Ruby utilizada no tutorial e pronto, problema resolvido.

<!-- more -->

**2 - Internacionaliza&ccedil;&atilde;o**: Esse ponto &eacute; minha &uacute;nica reclama&ccedil;&atilde;o com o Octopress(pelo menos at&eacute; 
agora), pois, n&atilde;o h&aacute; suporte nenhum para esse tipo de mecanismo, voc&ecirc; tem que ir p&aacute;gina por p&aacute;gina
e alterar as palavras direto nos html's.

**3 - Internacionaliza&ccedil;&atilde;o(datas)**: Se voc&ecirc; observar os nomes dos meses e dias da semana por padr&aacute;o 
s&atilde;o ingl&ecirc;s, mas, fica tranquilo, &eacute; s&oacute; copiar a solu&ccedil;&atilde;o de 
<a href="http://hugolyra.com/" title="Hugo Lyra" target="_blank">Hugo</a>:  
3.1 - Abra o arquivo octopress/plugins/date.rb  
3.2 - Acrescente o seguinte codigo no inicio do arquivo:
		Date::MONTHNAMES = [nil] + %w(Janeiro Fevereiro Marco Abril Maio Junho Julho Agosto Setembro Outubro Novembro Dezembro)  
		Date::DAYNAMES = %w(Domingo Segunda-Feira Terca-Feira Quarta-Feira Quinta-Feira Sexta-Feira Sabado)  
		Date::ABBR_MONTHNAMES = [nil] + %w(Jan Fev Mar Abr Mai Jun Jul Aug Sep Out Nov Dez)  
		Date::ABBR_DAYNAMES = %w(Dom Seg Ter Qua Qui Sex Sab)
		class Time  
			alias :strftime_nolocale :strftime  
			def strftime(format)  
				format = format.dup  
				format.gsub!(/%a/, Date::ABBR_DAYNAMES[self.wday])  
				format.gsub!(/%A/, Date::DAYNAMES[self.wday])  
				format.gsub!(/%b/, Date::ABBR_MONTHNAMES[self.mon])  
				format.gsub!(/%B/, Date::MONTHNAMES[self.mon])  
				self.strftime_nolocale(format)  
			end  
		end

**4 - Categorias**: N&atilde;o h&aacute; um template pronto para exibir as categorias na barra lateral, portanto criei um da seguinte 
forma:  
4.1 - Crie um arquivo octopress/source/_includes/custom/asides/categorias.html  
4.2 - O conteudo do arquivo sera:  
		<section>
			<h1>Categorias</h1>
			<span id="todas_categorias">
				{% raw %}
					{% for category in site.categories %}
						<a href="{{"/"}}{{site.category_dir}}{{"/"}}{{ category | first | downcase }}">{{ category | first }}</a>
					{% endfor %}
				{% endraw %}
			</span>
		</section>
4.3 - No arquivo octopress/_config.yml adicione o novo modulo no atributo default_asides:
default_asides: [custom/asides/versiculo.html, asides/recent_posts.html, custom/asides/categorias.html, custom/asides/about.html]

A mesma l&oacute;gica das categorias pode ser aplicada para tags, al&eacute;m disso voc&ecirc; pode construir outros 
m&oacute;dulos, como o que criei para postar um vers&iacute;culo b&iacute;blico(tudo bem que a galera de TI n&atilde;o se liga muito nisso), 
mas, ta&iacute; pra quem tiver um tempinho.

**Update: Encontrei uma forma mais fácil e mais segura neste <a href="http://anthonydigirolamo.github.com/blog/2011/09/21/octopress-category-listing/" target="_blank" title="Octopress Category Listing">link</a> para exibir as categorias. Você só tem que acrescentar uma linha de código no arquivo category_generator.rb:
	def category_links(categories)
	    dir = @context.registers[:site].config['category_dir']
	    categories = categories.keys if categories.class == Hash <<------- Adicione esta linha
	    categories = categories.sort!.map do |item|

Com isso, você agora pode listar as categorias da seguinte forma:
	<section>
      	    <h1>Categorias</h1>
               <span id="todas_categorias">
               {% raw %}
		   {{ site.categories | category_links }}
               {% endraw %}
               </span>
	 </section>
	
**5 - Customizando a url do GitHub para o seu pr&oacute;prio dom&iacute;nio**: Apesar de est&aacute; bem explicado no site do 
<a href="http://github.com/" title="GitHub" target="_blank">GitHub</a>, eu apanhei um 
pouco nessa etapa devido a minha total e absoluta falta de conhecimento em gerencimanto de dom&iacute;nio, mas, basicamente o que se tem 
de fazer &eacute;:  
5.1 - Criar um arquivo CNAME em octopress/source, onde o conteudo sera o seu dominio: meudominio.com  
5.2 - Criar um registro(zone file) A (host) com o ip do GitHub:207.97.227.245  
5.3 - Criar um registro(zone file) CNAME(host alias) com o link da sua pagina no GitHub: meuusuario.github.com  
	
Para essa etapa recomendo que depois que as altera&ccedil;&otilde;es forem comitadas de fato(rake deploy) voc&ecirc; espere 
pelo menos umas 2h para ent&atilde;o a nova configura&ccedil;&atilde;o ser espalhada pelos DNS's do mundo.

<strong>Sofonias 3.15:</strong> O SENHOR afastou as sentenças que eram contra ti e lançou fora o teu inimigo. O Rei de Israel, o SENHOR, está no meio de ti; tu já não verás mal algum.
