---
layout: post
title: Atualização de models com paperclip
---

[Paperclip](https://github.com/thoughtbot/paperclip) é uma gem que facilita a integração de upload de arquivos em uma aplicação Rails. Existem vários tutoriais de como usar o `Paperclip` para anexar uma imagem a um `model` no momento da criação, mas quase nenhum que explique como manipular os anexos em caso de edição e atualização do `model`. Para esse exemplo usaremos as classes `Exame` e `Arquivo`.  

{% highlight ruby %}
# exame.rb
class Exame < ActiveRecord::Base
  has_many :arquivos, dependent: :destroy
  accepts_nested_attributes_for :arquivos
end
# arquivo.rb
class Arquivo < ActiveRecord::Base
  belongs_to :exame
  
  has_attached_file :asset, 
    :path => ":rails_root/storage/#{Rails.env}#{ENV['RAILS_TEST_NUMBER']}/attachments/:id/:style/:basename.:extension"
  validates_with AttachmentSizeValidator, :attributes => :asset, 
    :less_than => 5.megabytes

  do_not_validate_attachment_file_type :asset
end
{% endhighlight %}

No nosso exemplo, `Exame` tem vários `Arquivos` que por sua vez carrega a configuração do `Paperclip`. Na nossa `view` de edição, criaremos uma listagem dos arquivos previamente anexados, e um campo para anexar novos arquivos. Na listagem, em frente ao nome de cada arquivo, adicionaremos um checkbox que carregará a informação de qual arquivo está sendo selecionado para ser apagado:  

{% highlight erb %}
<div class="form-group">
  <%= label_tag "Arquivos já anexados", nil, class: 'col-lg-2 control-label' %>
  <div class="col-lg-10">
    <% if @exame.arquivos.empty? %>
      Nenhum arquivo disponibilizado.
    <% else %>
      <table class='table table-bordered'>
        <% @exame.arquivos.each do |a| %>
            <tbody>
              <tr>
                <td><%= asset_file_name %></td>
                <td>
                  <%= check_box_tag "delete_arquivos[]", a.id, false %> 
                  Deletar Arquivo
                </td>
              </tr>
            </tbody>
        <% end %>
      </table>
    <% end %>
  </div>
</div>
<div class="form-group">
  <%= label_tag "Novos Arquivos", nil, class: 'col-lg-2 control-label' %>
  <div class="col-lg-10">
    <%= file_field_tag "assets[]", type: :file, multiple: true, 
      class: 'form-control' %>
  </div>
</div>
{% endhighlight %}

No controller, após passar pelas validações do `Exame`, checamos o `params[:assets]` para anexar os novos arquivos ao `Exame` e também checamos o `params[:delete_arquivos]` para deletar os arquivos selecionados. Esse parâmetro trará os ids dos selecionados.

{% highlight ruby %}
# exames_controller.rb
def update
  @exame = Exame.find(params[:id])
  if @exame.update_attributes(exame_params)
    if params[:assets]
      params[:assets].each { |asset|
        Arquivo.create asset: asset, exame_id: @exame.id
      }
    end
    if params[:delete_arquivos]
      Arquivo.destroy params[:delete_arquivos]
    end
    flash[:success] = 'Exame atualizado!'
    redirect_to exames_url
  else
    render 'edit'
  end
end
{% endhighlight %}

Simples e fácil!