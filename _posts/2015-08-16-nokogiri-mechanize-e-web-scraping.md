---
layout: post
title: Nokogiri, Mechanize e Web Scraping
---

Uma das minhas motivações para desenvolver o <a href="http://gynmovies.herokuapp.com" target="_blank">gynmovies+</a> foi a falta de informação na internet das sessões dos cinemas da minha cidade. Sempre que eu estava afim de assistir algum filme em cartaz eu tinha que entrar em todos os sites e verificar quais os horários das sessões. Bem, acontece que eu não gostava nem um pouco disso, e decidi tentar pegar essas informações diariamente, guardar num banco de dados e disponibilizar pra galera.

A questão agora era: Como?

## Nokogiri e Mechanize

<div class="message">
Data scraping is a technique in which a computer program extracts data from human-readable output coming from another program.
</div>

`Data scraping` sempre foi algo que gostei muito. Fazer isso utilizando as gems `Nokogiri` e `Mechanize` então, é de uma facilidade tão grande que até quem não curte pode mudar de ideia.

[Nokogiri](https://github.com/sparklemotion/nokogiri) é um leitor e parser de XML, HTML, SAX que suporta manipulação dos nós usando XPATH e CSS. Já o [Mechanize](https://github.com/sparklemotion/mechanize) é uma gem que automatiza a interação com websites. Ela te da acesso a métodos de manipulação de requisições HTTP, manipulação de formulários, etc.

## Começando com Nokogiri

Um exemplo prático, pesquisei no Google por cinema e peguei uma página de cinema qualquer, no caso essa [http://www.bourbonshopping.com.br/site/programacao.jsp](http://www.bourbonshopping.com.br/site/programacao.jsp).

O primeiro passo a ser executado é abrir o console do seu navegador e inspecionar o que é interessante. Ao abrir o console identificamos que o nome do filme fica dentro de um `<h1>` que por sua vez se encontra dentro de uma `div` com id `descricaoConteudo`. Depois de instalar o `nokogiri`.

{% highlight ruby %}
gem install nokogiri
{% endhighlight %}

Podemos abrir o `irb`, carregar as gems com `require` e começar a usar:

{% highlight ruby %}
> require 'nokogiri'
> require 'open-uri'
> url = http://www.bourbonshopping.com.br/site/programacao.jsp
> page = Nokogiri::HTML(open(url))
{% endhighlight %}

Note que para trabalhar com um site da internet, deve-se usar a gem `open-uri`. Sem ela, só conseguimos abrir arquivos `HTML`, `XML` locais na máquina. Ao rodar o último comando acima, `page = Nokogiri::HTML(open(url))`, teremos toda a página `HTML` carregada na memória pela variável `page` para começarmos a trabalhar. Como eu disse anteriormente, já sabemos onde encontrar o nome dos filmes e também já sabemos que o `nokogiri` trabalha com `XPATH` e `CSS`. Ao rodar uma pesquisa na página usando o seletor de CSS `#descricaoConteudo h1`, o resultado é um array de elementos com os nomes dos filmes dentro.

{% highlight ruby %}
> filmes = page.css('#descricaoConteudo h1')
> filmes.first.text
=> "Nome do filme"

# Ou por XPATH, é simples como clicar com o botão direito no console e 'Copiar XPATH'
> page.xpath('/html/body/div[3]/div[1]/div[1]/div[4]/h1').text
> "Nome do filme"
{% endhighlight %}

Bem simples, não é? Como selecionar elementos da `DOM` usando `jQuery`. Dessa maneira analisando a página e a sua estrutura, e usando o `nokogiri` para interpretá-la, conseguimos qualquer informação.  

Na maioria das vezes, a utilização somente do `nokogiri` é o suficiente para se extrair dados de onde você quiser, porém em alguns casos necessitamos de uma maior interação com a página para conseguir informações mais relevantes, como submeter um formulário com alguns dados, manipular o clique de um botão que mostra diferentes elementos na página, etc. Para essa interação, o `mechanize` é essencial.  

Como exemplo podemos pegar a página da CBF [http://bid.cbf.com.br](http://bid.cbf.com.br). Na parte de cima temos um formulário com 4 campos e um botão de filtrar. Ao selecionar uma data e clicar em filtrar, a página carrega a div interna com vários dados de jogadores. Como simular essa submissão de formulário com parâmetros da página? 

{% highlight ruby %}
> require 'mechanize'
> url = 'http://bid.cbf.com.br'
> agent = Mechanize.new
> page = agent.get(url)
{% endhighlight %}

Nesse momento, a variável `page` guarda informações relevantes ao site como todos os links, todos os formulários, todos os botões e assim por diante. Logo podemos capturar o único formulário da página assim:  

{% highlight ruby %}
> form = page.form
{% endhighlight %}

Com o formulário em mãos, setar valores para os campos é bem simples, basta checar qual a posição do campos no array `fields` do formulário, colocar o valor desejado e pedir ao `agent` submeter o formulário guardando essa nova página em uma outra variável:  

{% highlight ruby %}
> form.fields[0].value = '15/08/2015'
> new_page = agent.submit(form)
{% endhighlight %}

Com a nova página em mãos, inspecionamos as informações que queremos coletar e então podemos usar os seletores `XPATH` ou `CSS` assim como usamos anteriormente com o `nokogiri`.  

{% highlight ruby %}
> new_page.search('.mbottom-plus35 h1.nameplayer')
{% endhighlight %}

Simples e prático. 