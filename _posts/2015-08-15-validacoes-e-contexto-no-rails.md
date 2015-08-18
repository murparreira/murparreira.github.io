---
layout: post
title: Validações e contexto no Rails
---

Muitas vezes quem está trabalhando com `Rails` se pergunta: **Como eu posso fazer uma validação de um model que só funciona em uma parte do sistema?**. Um bom exemplo desse caso de uso seria uma aplicação de agendamento de pacientes. Vejamos o seguinte problema.

Nossa tabela de paciente tem os seguintes dados:

{% highlight ruby %}
# timestamp_create_pacientes.rb
class CreatePacientes < ActiveRecord::Migration
  def change
    create_table :pacientes do |t|
      t.string :nome
      t.date :nascimento
      t.string :cpf
      t.integer :sexo
      t.string :nome_mae
      t.string :endereco
    end
  end
end
{% endhighlight %}

E as seguintes validações:

{% highlight ruby %}
# paciente.rb
class Paciente < ActiveRecord::Base
  validates :nome, :cpf, :sexo, :nascimento, presence: true
end
{% endhighlight %}

Dessa maneira ao tentar salvar um paciente sem nome, CPF, sexo ou data de nascimento, o `ActiveRecord` vai reclamar e falar que esses campos precisam ser preenchidos. Até aí tudo bem, se tivermos um cadastro de paciente separado na aplicação. Porém, em um novo requisito do cliente aparece a necessidade de se criar um pré-cadastro ao realizar um agendamento, pois o usuário, no caso uma secretária, não terá tempo de preencher todos os dados ao telefone com o paciente. O requisito é o seguinte: **Na tela de agendamento, deve aparecer a opção de se pré-cadastrar um novo paciente informando o nome, o telefone.**. E agora??

## Refatorando o código

No nosso `model` acabamos de falar que não se pode salvar um paciente sem nome, CPF, sexo e data de nascimento. Porém se o usuário usar a tela de agendamento para um pré-cadastro, como devemos proceder.

A `API` de validações do `ActiveRecord` nos dá a possibilidade de realizar validações condicionais.

{% highlight ruby %}
# paciente.rb
class Paciente < ActiveRecord::Base
  attr_accessor :tela_paciente
  validates :nome, presence: true
  validates :cpf, presence: true, if: Proc.new{|p| p.tela_paciente?}
  validates :sexo, presence: true, if: Proc.new{|p| p.tela_paciente?}
  validates :nascimento, presence: true
end

# pacientes_controller.rb
def create
  @paciente = Paciente.new paciente_params
  @paciente.tela_paciente = true
  if @paciente.save
    #...
  end
end

# agendamentos_controller.rb
def create
  @agendamento = Agendamento.new agendamento_params
  @paciente = Paciente.new paciente_params
  @paciente.tela_paciente = false
  if @paciente.save
    @agendamento.paciente_id = @paciente.id
    if @agendamento.save
      #...
    end
  end
end
{% endhighlight %}

Criamos um atributo virtual (que não é persistido no banco) para informar de onde o paciente está sendo salvo e manipulado. Antes disparar a validação, o `Proc` é executado e a condição é checada para verificar se o fluxo continua ou não. Bem simples, porém um pouco poluído.

## Contextos

Uma outra maneira de trabalhar com validações condicionais, é indicar o contexto ao realizar uma operação como salvar ou atualizar um `model`. Como assim contextos?

O método `save` pode receber um hash de contexto da seguinte maneira:

{% highlight ruby %}
# pacientes_controller.rb
def create
  @paciente = Paciente.new paciente_params
  @paciente.tela_paciente = true
  if @paciente.save(context: :tela_paciente)
    #...
  end
end

# paciente.rb
class Paciente < ActiveRecord::Base
  validates :endereco, presence: true, on: :tela_paciente
  #...
end
{% endhighlight %}

Essa validação só será executada quando chamarmos o `save` de pacientes passando o contexto `tela_paciente`. Você pode se perguntar, **Mas e as outras validações?**. Todas elas serão executadas, mas a que requer o contexto só será disparada da maneira que mostramos.
