# Apresentando Clojure

Clojure é simplicidade, expressividade e produtividade.

Clojure é uma linguagem comparativamente nova (2007), fruto do trabalho de Rich
Hickey, seu criador, e de vários contribuidores ao longo dos anos. Você pode
dar uma boa olhada no próprio código no [repositório oficial no GitHub](https://github.com/clojure/clojure).

Ela é mais uma linguagem na família do Lisp, o que significa que parênteses estarão por toda parte.
Mais do que isso, ser um Lisp significa estar alinhada com o conceito de
homoiconicidade (calma, não vai embora) e suporte natural e elegante à macros.


Além disso, Clojure roda sobre a JVM (Java Virtual Machine), o que possibilita que
se tenha acesso à imensa gama de bibliotecas e frameworks do Java e desfrute de
toda robustez, confiabilidade e tooling já disponíves para a plataforma.
Veremos mais a frente como é simples interoperar com o Java e tirar proveito da JVM,
quando necessário.

Clojure também abraça fortemente o paradigma funcional, enfatizando vários dos seus
melhores aspectos como imutabilidade (através de estruturas de dados persistentes),
funções de primeira classe, funções de ordem superior, funções puras, transparência
referencial, laziness, recursão, além do filosofia

> "_It is better to have 100 functions operate on one data structure than 10 functions on 10 data structures._"  
> — Alan Perlis  
>  
> (_É melhor ter 100 funções que operam sobre uma estrutura de dados do que 10 funções que operam sobre 10 estruturas de dados._)

Em termos de compilação e execução, Clojure é uma linguagem compilada e dinamicamente tipada.
Apesar de ser compilada, ela oferece interatividade com o desenvolvedor
e o ambiente de desenvolvimento sem igual. É muito comum passar a maior parte
do tempo de desenvolvimento em Clojure no seu REPL (Read-Eval-Print Loop),
prototipando e experimentando trechos de código com feedbacks imediatos, que poupam
tempo de desenvolvimento e trazem mais segurança.

Todas essas caraterísticas criam um excelente cenário para explorar multithreading.
E é exatamente isso que Clojure faz. A linguagem possui bom suporte à programação
concorrente e assíncrona e ao modelo CSP (Communicating Sequential Processes).

## Não tenha medo dos parênteses

Sério, eles estão para te ajudar. Por ser um Lisp, Clojure é cheio de parênteses
para todos os lados. Com o tempo, você se acostuma com eles e eles se tornam
invisíveis!

A estrutura básica do Clojure gira em torna das chamadas symbolic expressions, ou s-expressions.
Uma s-expression pode ser um _atom_, ou seja, um literal ou um símbolo:

```clojure
1
1.5
"this is a string"
\a
:some-keyword
[1 2 3 4]
{:a 1 :b 2}
```
ou uma _list_, ou seja, uma s-expression não-atômica

```clojure
(f arg1 arg2 arg3 ,,,)
```

Quando uma s-expression existe para ser avaliada, a chamamos de _form_.
__Toda expressão avaliada tem um valor como resultado__.

O primeiro elemento de uma form deve resolver para um atom de um dos três tipos:

1. Forma especial (_special form_)
2. Macro
3. Função

Explicando brevemente cada uma delas:

* Formas especiais possuem suas próprias regras de avaliação e são tratadas
diretamente pelo compilador. Elas podem ser entendidas como as formas fundamentais
da linguagem, sobre as quais todo o restante é construído. Elas não podem ser
criadas por desenvolvedores.

* Macros são formas que permitem o desenvolvedor estender a linguagem. Elas possuem
regras de avaliação diferentes das funções e possibilitam manipular o próprio código
em tempo de compilação. Se você já programou em outra linguagem que possui "macros"
como C ou C++, é melhor evitar a tentação de fazer analogias. Essas macros são coisas
bem diferentes e qualquer tentativa de analogia não seria fiel e traria mais confusão.

* Por fim, funções são o conceito mais familiar para quase todos desenvolvedores.
Elas possuem regras de avaliação usuais e se comportam exatamente como o esperado.

Podemos aninhar múltiplas formas:

```clojure
(active-user? (json->user (get-user-from-service! (name-by-id! 123) ;; This is a comment.
                                                  (url-from-config! *config* service-name))))
```

Nesse exemplo:

* `active-user?` é uma função que recebe um argumento, um 
usuário, e retorna se este está ativo ou não.
* O usuário é o retorno da função `json->user`, que converte
uma resposta em formato JSON para um formato de usuário da aplicação.
* Esse usuário é obtido através da função `get-user-from-service!`, que por
sua vez recebe dois argumentos:
    * O nome do usuário, obtido através da função `name-by-id!`,
    que devolve um nome dado um ID (`123`).
    * Uma URL, obtida através da função `url-from-config!`,
    que recebe dois argumentos:
        * Uma estrutura de configuração, `*config*`.
        * O nome do serviço que se quer criar a URL, `service-name`.

Obviamente esse é um exemplo inventado e mais a frente veremos uma forma mais
idiomática de reescrevê-lo. Por enquanto, ele serve para ilustrar
algumas convenções interessantes.

## Algumas convenções iniciais

No exemplo anterior os nomes das funções utilizam `-` e `!`. Isso pode soar
estranho para alguém vindo de outra linguagem como JavaScript ou Python,
onde identificadores têm regras bastante específicas sobre os caracteres aceitos.

Em Clojure, muitos caracteres "estranhos" são aceitos em identificadores. O mais
comum é certamente `-`. O Clojure segue uma convenção de nomenclatura apelidada
de _kebab-case_, onde partes do nome são separadas por `-`.

Isso significa que em vez de escrever `mySimpleFunction` (camel-case)
ou `my_simply_function` (snake-case), em Clojure escrevemos `my-simple-function`
(kebab-case).

Comentários em Clojure são marcados por `;`. Tudo que vier na mesma linha após
`;` é comentário e é ignorado pelo compilador. Uma prática bastante comum é
usar dois (`;;`), como feito no exemplo, para enfatizar a presença do comentário.

Além disso, o caractere de vírgula é tratado exatamente igual a um espaço
em branco. Isso significa que `[,,,]` e `[   ]` são exatamente iguais
para o compilador do Clojure.

Assim como em outras linguagens, algumas funções retornam valores booleanos
(verdadeiro ou falso) e portanto são chamadas funções predicado. Funções predicado
em Clojure são convencionalmente sufixadas com `?`, como a função `active-user?`
do exemplo.

Funções que terminam com `!` costumam denotar funções não puras, que produzem
algum side-effect ou que dependem de um estado global. Porém, nem sempre esse é
o caso, e o melhor é sempre verificar a convenção adotada pelo projeto.

Algumas funções convertem entre formatos e tipos. Como o caractere `-` (como já vimos)
e o `>` podem ser usados em nomes, convencionou-se utilizar `->` para explícita
e concisamente expressar funções de conversão. Assim, `json->user` indica uma função
que recebe algum JSON e o converte para uma representação de usuário, provavelmente
interna da aplicação.

No exemplo, também se vê o `*config*`. Esses asteriscos antes e depois do nome
são conhecidos como _earmuffs_. Essa é uma convenção para denotar _vars_ dinâmicas
(dynamic vars). Por enquanto, você pode pensar em dynamic vars como variáveis
(mutáveis) globais. Como sempre em Clojure, estado global é altamente desencorajado
e portanto você verá earmuffs muito raramente.

Por fim, note como todos os parênteses de fechamento, `)`, estão agrupados no
final de toda expressão. Isso também é uma convenção e, em teoria, você poderia, __mas não
deveria__, reescrever o código assim:

```clojure
;; BAD: Don't do this!
(active-user?
  (json->user
    (get-user-from-service!
      (name-by-id! 123) ;; This is a comment.
      (url-from-config!
        *config*
        service-name
      )
    )
  )
)
```

Não se preocupe. Editores de texto com suporte a Clojure irão te ajudar a
formatar o código corretamente.

Existem várias outras convenções e boas práticas. Um bom começo é dar uma olhada
em [Clojure Style Guide](https://github.com/bbatsov/clojure-style-guide).

## REPL

Uma das grandes vantagens do Clojure é seu incrível suporte ao desenvolvimento
interativo através do REPL (Read-Eval-Print Loop). O REPL é um console que permite
a entrada de expressões pelo desenvolvedor, as avalia, e retorna o resultado. O
conceito não é novo para desenvolvedores Python ou Ruby, mas Clojure leva o conceito
para um novo nível.

Existem algumas formas de abrir o REPL. Você pode simplesmente executar

```shell script
$ clj
```

na linha de comando ou, se estiver no diretório de um projeto Clojure criado
pelo lein, você pode executar

```shell script
$ lein repl
```

que também traz o REPL, mas com alguns benefícios (e.g., acesso às libs do projeto).

Uma boa dica é configurar o seu editor de texto ou IDE para acesso rápido ao REPL.
O IntelliJ com Cursive possui uma tab dedicada ao REPL e é possível configurar
diversos atalhos para maior produtividade.

Uma vez no REPL, lhe é dado um prompt como

```shell script
user=>
```

Neste caso, `user` indica o namespace atual (falaremos sobre namespaces em outra
seção). A partir de então, você pode entrar qualquer expressão para ser avaliada

```shell script
user=> (+ 1 1)
2
user=>
```

O REPL guarda o valor retornado pela última expressão avaliada na variável `*1`.
Continuando o exemplo anterior, é possível fazer algo como

```shell script
user=> (+ *1 3)
5
```

O REPL também aceita expressões com múltiplas linhas e as trata corretamente:

```shell script
user=> (defn f
  #_=> [x y]
  #_=> (+ x y))
#'user/f
user=> (f 1 2)
3
```

Note como a indentação não ficou das melhores. Isso acontece com REPL iniciado
com `clj` ou `lein clj` sem configurações adicionais. Esse é outro argumento
a favor de configurar o REPL no seu editor de texto ou IDE, onde a indentação
(e qualquer outra questão de _style_) segue as mesmas configurações do próprio
editor e trazem uma experiência ainda melhor.

Através do REPL também é possível ler docstrings e até mesmo o próprio código
de uma função ou macro. Para ver a docstring, use a função `doc`:

```shell script
user=> (doc if)
-------------------------
if
  (if test then else?)
Special Form
  Evaluates test. If not the singular values nil or false,
  evaluates and yields then, otherwise, evaluates and yields else. If
  else is not supplied it defaults to nil.

  Please see http://clojure.org/special_forms#if
nil


user=> (doc when)
-------------------------
clojure.core/when
([test & body])
Macro
  Evaluates test. If logical true, evaluates body in an implicit do.
nil


user=> (doc +)
-------------------------
clojure.core/+
([] [x] [x y] [x y & more])
  Returns the sum of nums. (+) returns 0. Does not auto-promote
  longs, will throw on overflow. See also: +'
nil
```

Note como o output indica qual é o tipo da form. Se for uma forma especial ou macro,
isso estará bem explícito, como no caso do `if` e do `when`, respectivamente. Se
nada for indicado, então trata-se de uma função regular, como no caso do `+`.

Para dar uma olhada no código, use a função `source`:

```shell script
user=> (source empty?)
(defn empty?
  "Returns true if coll has no items - same as (not (seq coll)).
  Please use the idiom (seq x) rather than (not (empty? x))"
  {:added "1.0"
   :static true}
  [coll] (not (seq coll)))
nil
```

De agora em diante, não será mais representado o prompt do REPL (`user=>`) nos
exemplos. Para manter mais limpo e conciso, apenas as próprias expressões
serão mostradas. [Veja as convenções adotadas nos exemplos](../README.md#convenções).

## Buscando ajuda

Clojure é uma linguagem comparativamente nova e sua base de desenvolvedores cresce
mais a cada dia. A sua comunidade é muito aberta e suportiva a novos membros.

Um ótimo lugar para buscar ajuda é no [Slack dos Clojurians](https://clojurians.slack.com).
Lá existe um canal `#beginners` onde você pode postar suas dúvidas (em inglês)
e certamente alguém tentará te ajudar.

A comunidade Clojure também faz um bom trabalho, contribuindo com vários exemplos
na documentação aberta da linguagem em [ClojureDocs](https://clojuredocs.org/).

Ao contrário de quando desenvolvia em outras linguagens, com Clojure eu me vejo
utilizando cada vez menos o StackOverflow. Não porque a comunidade é fraca lá ou
por causa do StackOverflow em si, mas sim porque a simplicidade da linguagem
torna esse tipo de tarefa menos frequente e necessário.