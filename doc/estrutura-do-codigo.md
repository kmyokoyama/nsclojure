# Estrutura do código

Toda linguagem precisa de algumas estruturas básicas que tornem possível escrever
código de forma fluida. Nesta seção, serão apresentadas algumas estruturas
básicas, mas extremamente úteis e cotidianas na linguagem Clojure. Elas serão
a base para começarmos a escrever código mais compĺexo e realístico.

## `let`

Até agora foi usado `def` para definir alguns nomes nos exemplos. Na verdade,
essa é uma prática longe da ideal. Será falado mais do `def` mais adiante, mas por ora
basta saber que `def` define um nome global, e estado global é ruim e deve ser
evitado.

> De agora em diante, a palavra _nome_ será substituída pelo termo mais preciso
> _symbol_.

No entanto, em muitos casos é útil e até imprescindível declarar symbols com
escopos locais e bem definidos. Por exemplo, se uma mesma expressão deve ser avaliada
somente uma vez e seu resultado utilizado em duas ou mais expressões subsequentes,
então deve haver um jeito de guardar o seu valor com um dado nome, e
posteriormente acessar esse valor através desse nome. Entra em jogo o `let`.

O `let` é uma forma especial que permite criar symbols locais. Veja um exemplo:

```clojure
(let [x 1
      y (* 2 x)]
    (println "x =" x " y =" y)
    (+ x y))
;;# "x = 1 y = 2"
;;=> 3
```

Neste exemplo, criamos dois symbols locais `x` e `y` com os valores `1` e `2`, respectivamente.
Note que o valor de `y` é derivado de `x`. Qualquer valor atribuído dentro do
vector de bindings do `let` fica disponível imediatamente após seu binding. 

Após o vector de bindings, o `let` recebe uma série de expressões
como se tivesse um `do` implícito e as avalia em ordem. Essas expressões
podem ser qualquer coisa, incluindo outros `let`. O resultado da última expressão
avaliada é o valor final da expressão `let`.

No último exemplo, o `let` recebeu duas expressões (a do `println` e do `+`)
e seu valor final é aquele resultante da avaliação de `(+ x y)`.

Os valores atribuídos no `let` têm escopo limitado à expressão `let`:

```clojure
(let [x 1]
  (let [z (* 2 x)] ;; z exists from here...
    (println "x =" x)
    (println "z =" z)) ;; to here.
  (println "z =" z)) ;; But not here.
;;! Syntax error compiling at (/tmp/form-init12980331161769918786.clj:5:3).
;;! Unable to resolve symbol: z in this context
```

O que aconteceu é que o symbol `z` existe apenas dentro da expressão do `let` mais
interno, mas nâo no `let` mais externo. Essa expressão não pôde nem ao menos
ser avaliada, a própria compilação já falhou.

As expressões que são atribuídas no `let` podem ser tão complexas quanto se queira.
Na verdade, podemos ter até chamadas de rede ou ao banco de dados:

```clojure
(let [movie-name (get-movie-by-id! user-id db) ;; Query to the database.
      rating (get-rating-by-name! movie-name url)] ;; Request through the network.
    (do-something-with-rating))
```

## `if-let` e `when-let`

Já vimos `if`, `when`, e agora o `let`. Um cenário bastante comum é atribuir
um dado valor a um nome, e em seguida testar se esse valor é verdadeiro (com
`if` ou `when`), prosseguindo para alguma ação caso o seja:

```clojure
(let [numbers [1 2 3 4]]
    (if (seq numbers)
      (str "It has " (count numbers) " elements")
      "It is empty"))
;;=> "It has 4 elements"
```

Isso é ok e funciona, mas é um tanto quanto trabalhoso e aninhado. Clojure gosta
de simplificar as coisas, não complicar. Existem duas macros que servem bem nessa
situação - `if-let` e `when-let`. É possível reescrever o exemplo assim:

```clojure
(if-let [numbers (seq [1 2 3 4])]
    (str "It has " (count numbers) " elements")
    "It is empty")
;;=> "It has 4 elements"

(if-let [numbers (seq [])]
    (str "It has " (count numbers) " elements")
    "It is empty")
;;=> "It is empty"

(when-let [numbers (seq [1 2 3 4])]
    (str "It has " (count numbers) " elements"))
;;=> "It has 4 elements"

(when-let [numbers (seq [])]
    (str "It has " (count numbers) " elements"))
;;=> nil
```

O que essas macros fazem é realizar o binding (como no `let`) se, e somente se,
o valor a ser atribuído for truthy, i.e., não for `false` ou `nil`.

Também existem as variantes `if-some` e `when-some`. Elas funcionam quase
exatamente como a `if-let` e `when-let` com a diferença que o valor
só é atribuído se não for `nil`, i.e., até o valor `false` pode ser atribuído.

## `do`

O `do` é uma forma especial que aceita um número qualquer de expressões e
as executa em ordem. O valor final da expressão `do` é o valor da última expressão
que ele avaliou:

```clojure
(do (+ 10 1)
    (* 2 5)
    (- 3 1))
;;=> 2
```

Como apenas o valor da última expressão é retornado pelo `do`, ele geralmente
é utilizado para realizar side-effects, i.e., modificar o estado externo ao
contexto onde ele executa (e.g., exibir algo na tela ou fazer uma requisição externa).

## `->` e `->>`

Essas duas macros são extremamente convenientes e populares em código Clojure.
A primeira, `->` (leia-se _thread first_), funciona como uma agulha e linha, costurando
resultados e primeiros argumentos de expressões. A segunda, `->>` (leia-se _thread last_),
funciona de forma semelhante, mas costura resultados e os últimos argumentos.
Em outras linguagens, temos algo parecido
(mas não exatamente igual) como o pipe do Elixir( `|>`), do R (`%>%`), e do F# (também `|>`).

A primeira expressão passada para `->` é passada
como primeiro argumento (daí o nome _thread **first**_) da próxima expressão.
O resultado dessa expressão é passado como primeiro argumento da próxima expressão, e assim
por diante. O valor da última expressão avaliada é o valor final do `->`.
Isso é melhor explicado com um exemplo:

```clojure
(-> 1
    (* 4) ;; (* 1 4) => 4
    (- 3) ;; (- 4 3) => 1
    dec)  ;; (dec 1) => 0
;;=> 0
```

O `->>` é bem parecido, mas ao invés de passar o resultado da expressão anterior
como primeiro argumento da próxima expressão, ele passa como último argumento
(daí o nome _thread **last**_). A seguir, tem-se o comportamento do exemplo anterior
se fosse usado `->>`:

```clojure
(->> 1
    (* 4) ;; (* 4 1)  =>  4
    (- 3) ;; (- 3 4)  => -1
    dec)  ;; (dec -1) => -2
;;=> -2
```

Se a expressão não tiver outros argumentos, como é o caso da expressão `dec` nos
exemplos, então os parênteses podem ser dispensados (mas se usados não fazem diferença).

Além disso, as expressões em `->` e `->>` podem ser tão complexas quanto se queira,
desde que possam receber pelo menos um argumento.

Ainda não falamos de estruturas de dados, mas já vale deixar notado uma convenção
útil aqui. Funções que atuam sobre sequences costumam esperar a sequence como último
argumento, e portanto, devem ser usadas com `->>`. Por outro lado, funções que atuam
sobre outras estruturas de dados (como maps), costumam esperar essas estruturas
como primeiro argumento, e logo devem ser usadas com `->`.

## Código é sobre transformação de dados

Podemos demorar a nos dar conta disso, mas programar e escrever código é sobre
transformar dados. Programas são fluxos contínuos de transformação de dados
de uma forma para outra.

Clojure facilita nossa vida ao tornar esse fluxo de transformação de dados
o mais claro e sem surpresas quanto possível. Pensar sobre o código, o que ele
está fazendo, e o que ele _deveria_ estar fazendo se torna uma tarefa muito menos
exaustiva e mais divertida.

Note que eu não usei a palavra _variável_ uma vez sequer em uma seção que é
suposta falar da estrutura básica de um programa. Em uma seção semelhante
de uma linguagem como C ou Python, seria possível encontrar dezenas de referências
às palavras variável, declaração, definição e atribuição.

Isso é possível, porque a maior parte dos dados do Clojure é imutável. Isso
significa que uma vez dado determinado valor a um nome, esse nome se refere a
esse valor para sempre.

Mas estamos falando de transformação de dados e portanto, obviamente, esses dados
precisam e vão mudar! De fato, é possível manipular os dados da forma como bem quiser,
mas para o resultado ser útil no futuro, você deve dar um novo nome para ele.

As ferramentas que foram vistas aqui são o alicerce de se trabalhar com dados em
Clojure. Apesar da aparente simplicidade, com o tempo é possível perceber o quão
poderosas e suficientes essas ferramentas são. Em alguns casos, porém, é necessário
um outro conjunto de ferramentas para lidar com estado em nossos programas,
mas chegaremos lá.

<!--
## Destructuring (mover para Estruturas de dados)

Clojure não tem pattern-matching built-in, mas possui um mecanismo quase tão
poderoso quanto, que permite associar symbols a partes de uma estrutura de dados
complexa durante um binding. Esse mecanismo é chamado destructuring, porque
ele se assemelha a desconstrução de uma estrutura de dados complexa em suas partes.
-->