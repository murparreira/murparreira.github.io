---
layout: post
title: Mechanize e Web Scraping
---

Uma das minhas motivações para desenvolver o <a href="http://gynmovies.herokuapp.com" target="_blank">gynmovies+</a> foi a falta de informação na internet das sessões dos cinemas da minha cidade. Sempre que eu estava afim de assistir algum filme em cartaz eu tinha que entrar em todos os sites e verificar quais os horários das sessões. Bem, acontece que eu não gostava nem um pouco disso, e decidi tentar pegar essas informações diariamente, guardar num banco de dados e disponibilizar pra galera.

A questão agora era: Como?

## Mechanize

<div class="message">
Data scraping is a technique in which a computer program extracts data from human-readable output coming from another program.
</div>

`Data scraping` sempre foi algo que gostei muito. Fazer isso utilizando a gem `Mechanize` então, é de uma facilidade tão grande que até quem não curte pode mudar de ideia.

[Mechanize](https://github.com/sparklemotion/mechanize) é uma gem que automatiza a interação com websites. Ela te da acesso a métodos de manipulação de requisições HTTP, manipulação de formulários, etc.

Um exemplo prático, pesquisei no Google por cinema e peguei uma página de uma outra cidade, no caso essa [http://www.bourbonshopping.com.br/site/programacao.jsp](http://www.bourbonshopping.com.br/site/programacao.jsp).

O primeiro passo a ser executado é abrir o console do seu navegador e inspecionar o que é interessante. Ao abrir o console identificamos que o nome do filme fica dentro de um `<h1>` que por sua vez se encontra dentro de uma `div` com id `descricaoConteudo`. Depois de instalar o `mechanize`.

{% highlight ruby %}
gem install mechanize
{% endhighlight %}

Podemos abrir o `irb`:

{% highlight ruby %}
> require 'mechanize'
>



