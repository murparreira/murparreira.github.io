---
layout: post
title: Sinatra, quando Rails é muito grande
---

Certa vez tive a necessidade de automatizar uma pequena tarefa no meu trabalho. Eu era responsável por igualar o código de testes e de produção, checando por diferenças nos commits dos dois repositórios que a gente tinha internamente. Era uma tarefa bem manual, eu tinha duas cópias dos projetos na minha máquina, então eu entrava, atualizava os dois diretórios pra última versão, rodava um `diff` entre os dois, copiava os arquivos diferentes com `rsync` sobreescrevendo os de produção e enviava o código para o repositório central. Acontece que pra grandes commits de arquivos isso era um saco, a lista era muito grande.  

Foi então que tive a ideia de fazer um interface que manipulasse esse meu problema e qualquer pessoa que commitasse o código poderia analisar os 2 repositórios e enviar seus arquivos para o repositório central de produção.  

Acontece que esse foi um problema muito simples pra se usar um framework tão robusto quanto o `Rails`. Eu não iria usar conexão com bancos de dados, consequentemente eu não iria usar a camada M do MVC, não precisaria de autenticação, não precisaria de todas as convenções que são impostas ao desenvolvedor que o usa. Era uma coisa bem simples, algumas rotas, uma resposta HTML, algumas ações a serem executadas. Usar `Rails` era inviável.  

Nesse momento entra em cena o [Sinatra](http://www.sinatrarb.com/). Tão simples como o exemplo da página inicial, você instala a gem, cria um arquivo .rb, coloca o require, cria uma rota baseada em um verbo HTTP, executa esse arquivo no console e BOOM, mágico. `Sinatra` roda em cima do [Rack](http://rack.github.io/), que é uma interface para servidores web em `Ruby`.  

O que exatamente eu precisava era de 2 rotas. Uma que listasse uma página inicial com as opções disponíveis para o pessoal. Outra que fizesse o `diff` entre os dois diretórios e pegasse a lista dos arquivos diferentes mostrando o nome dele e um checkbox do lado para ser selecionado, e ao fim da listagem um botão de realizar a operação de cópia de um diretório para outro.  

{% highlight ruby %}
# test.rb
get '/' do
  erb :index
end
get '/diff' do
  cmd = "diff --exclude-from='public/exclude.txt' --brief -r #{trunk_path} #{production_path}"
  string = `#{cmd}`
  @files = string.split(/\n/)
  # Código omitido para simplificar
  erb :diff
end
post '/send' do
  params[:parametro]
  # Código omitido para simplificar
  redirect '/'
end
{% endhighlight %}

{% highlight html %}
# diff.erb
<form action="/send" method="POST">
  <% @files.each do |file| %>
    # Código omitido para simplificar
  <% end %>
  <input id="submit" type="submit" value="Enviar">
</form>
{% endhighlight %}

Aqui criamos um arquivo com as rotas raiz `/`, diff `/diff` e send `send`. A rota raiz apenas renderiza o arquivo `views/index.erb`. A rota diff tem um script que quando executado usa o `diff` do sistema e cria um array de arquivos baseado no retorno dessa função. Repare que colocamos o `@` na variável `@files` para podermos acessá-la na view, assim como no `Rails`. No final da ação especificamos qual view renderizar, `erb :diff`, procurando por `views/diff.erb` no diretório do projeto. A rota `send` só é chamada por POST, e reparem que ela tem um objeto `params` assim como os métodos dos controllers no `Rails`. Após realizar as ações ele redireciona o usuário para a rota raiz.  

## Customizando o Sinatra

A ideia inicial do Sinatra é dar suporte a pequenas aplicações como essa. Porém ele é um framework que pode ser altamente customizado. Por exemplo, se vc precisa de sessões, basta ativar o suporte a sessões no seu arquivo:  

{% highlight ruby %}
enable :sessions
get '/' do
  session[:mensagem] = 'Mensagem!'
  redirect to('/diff')
end
get '/diff' do
  mensagem = session[:mensagem] # => 'Mensagem!'
  erb :diff
end
{% endhighlight %}

O Sinatra também aceita helpers que podem ser definidos em qualquer lugar, e usados nas views.  

{% highlight ruby %}
# teste.rb
helpers do
  def adicionar_titulo_ao_nome(text)
    "<strong>Sr. #{text}</strong>"
  end
end

# view.erb
<%= adicionar_titulo_ao_nome(@nome) %>
{% endhighlight %}

Além de vários outros middlewares que podem ser adicionados e usados como autenticação, log, debug, roteamento, tudo ao gosto do freguês. Além do mais existem várias gems de integração com os ORM's mais famosos do `ruby` como `ActiveRecord`, `Sequel` e `DataMapper`, basta pesquisar por sinatra e o nome do ORM.  

Links de ajuda: 

[Introdução ao Sinatra do próprio site](http://www.sinatrarb.com/intro.html)  

[Boas práticas com o Sinatra](http://blog.carbonfive.com/2013/06/24/sinatra-best-practices-part-one/)  

[Rack](http://rack.github.io/)  

[O que é Rack?](http://southdesign.de/blog/rack.html)  