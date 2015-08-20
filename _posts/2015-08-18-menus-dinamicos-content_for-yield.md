---
layout: post
title: Menus dinâmicos usando content_for e yield
---

Toda applicação web precisa de um menu, seja ele lateral, fixo no topo, escondido, etc. A navegação entre os módulos e componentes do sistema deve ser fluida e sem complicações. Geralmente temos um código de layout assim, onde `nav` é a parte do menu lateral:  

{% highlight html %}
<!DOCTYPE html>
<html>
<head></head>
<body>
  <div id="wrapper">
    <nav class="navbar-default navbar-static-side" role="navigation">
      <div class="sidebar-collapse">
        <ul class="nav" id="side-menu">
          <li class="nav-header">Header</li>
          <li class="nav-header">Menu 1</li>
          <li class="nav-header">Menu 2</li>
          <li class="nav-header">Menu 3</li>
          <li class="nav-header">Menu 4</li>
          <li class="nav-header">Menu 5</li>
        </ul>
      </div>
    </nav>
    <div id="page-wrapper">
      <div class="wrapper wrapper-content">
        <%= yield %>
      </div>
    </div>
  </div>
</body>
</html>
{% endhighlight %}

Se esse é o layout da aplicação, ela estará visível em todas as telas. Mas como fazer para que em determinadas views eu tenha um menu com diferentes opções? O `ActiveView` provê os chamados `CaptureHelpers` [CaptureHelpers](http://api.rubyonrails.org/classes/ActionView/Helpers/CaptureHelper.html), que são métodos que marcam determinada parte do seu código HTML para ser usado em outro lugar no layout. Para isso temos o método `content_for`.  

Quando colocamos uma parte do código HTML dentro de um bloco `content_for` e damos um nome a ele:  

{% highlight html %}
<% content_for :menu_lateral do %>
  <nav class="navbar-default navbar-static-side" role="navigation">
    <div class="sidebar-collapse">
      <ul class="nav" id="side-menu">
        <li class="nav-header">Header</li>
        <li class="nav-header">Menu 1</li>
      </ul>
    </div>
  </nav>
<% end %>
{% endhighlight %}

Marcamos esse pedaço de código com a tag `menu_lateral`. Portanto podemos invocar no layout esse pedaço de HTML usando `yield`:  

{% highlight html %}
<!DOCTYPE html>
<html>
<head></head>
<body>
  <div id="wrapper">
    <%= yield :menu_lateral %>    
    <div id="page-wrapper">
      <div class="wrapper wrapper-content">
        <%= yield %>
      </div>
    </div>
  </div>
</body>
</html>
{% endhighlight %}

Dessa maneira, em diferentes views podemos definir diferentes menus laterais. Mas aí você se pergunta: **E se eu tiver um menu lateral padrão? Que sempre será renderizado se a view não definir um bloco de menu.**. Como vimos na documentação, temos um método chamado `content_for?` que verifica se existe algum código a ser inserido com a tag e retorna true ou false. Então adicionamos uma checagem com um menu padrão:  

{% highlight html %}
<!DOCTYPE html>
<html>
<head></head>
<body>
  <div id="wrapper">
    <% if content_for?(:menu_lateral) %>
      <%= yield :menu_lateral %>    
    <% else %>
      <nav class="navbar-default navbar-static-side" role="navigation">
        <div class="sidebar-collapse">
          <ul class="nav" id="side-menu">
            <li class="nav-header">Header</li>
            <li class="nav-header">Menu 1</li>
            <li class="nav-header">Menu 2</li>
            <li class="nav-header">Menu 3</li>
            <li class="nav-header">Menu 4</li>
            <li class="nav-header">Menu 5</li>
          </ul>
        </div>
      </nav>
    <% end %>
    <div id="page-wrapper">
      <div class="wrapper wrapper-content">
        <%= yield %>
      </div>
    </div>
  </div>
</body>
</html>
{% endhighlight %}

Podemos até jogar esse menu lateral padrão em uma partial e deixar o código mais limpo ainda :).