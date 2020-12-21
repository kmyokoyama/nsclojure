# Controle de fluxo

Como toda linguagem, é necessário ter as ferramentas para controlar o fluxo
de execução do programa. Os principais mecanismos de controle de fluxo que
veremos nesta seção são do tipo condicional.

Controles de loop são menos necessários em linguagens como Clojure, onde outros
mecanismos mais interessantes e idiomáticos estão disponíveis. Clojure oferece
boas abstrações de coleções e uma vasta gama de funções que atuam sobre elas,
o que reduz a necessidade de estruturas como `for` e `while`, tão presentes em
outras linguagens, para próxima de zero. Para além disso, recursão e algumas outras
estruturas de iteração cobrem o restante das necessidades.

> Toda expressão em Clojure retorna um valor (ainda que este seja `nil`)! Isso é particularmente verdade
> para todas expressões de controle de fluxo que serão mostradas abaixo.

## Operadores lógicos

Antes de entrar no assunto de controle de fluxo, vale falar sobre operadores lógicos.

Qualquer função (ou macro) que tem como resultado um valor truthy ou falsey, pode
ser usada em contextos condicionais (como no `if` que veremos a seguir). A principal
finalidade dos operadores lógicos é criar condições lógicas compostas.

### `and`

O operador lógico `and` é uma macro que aceita um número variável de argumentos
e os avalia da esquerda para a direita. Ele retorna o primeiro valor falsey
encontrado, se houver, ou o valor do último argumento, se todos avaliarem truthy.
Veja alguns exemplos:

```clojure
(and 0 [] 1 2) ;;=> 2
(and 0 [] (seq []) 10) ;;=> nil
(and 1 2 (= 1 2) (seq [])) ;;=> false
(and) ;;=> true
``` 

No primeiro exemplo, todos valores são logicamente truthy, e portanto `and`
retorna o último valor avaliado, `2`. No segundo exemplo, `(seq [])` é `nil`, então
`nil` é retornado. No terceiro exemplo, o primeiro valor falsey avaliado é a
comparação `(= 1 2)` que retorna `false`, sendo este então o valor retornado por `and`.
Por fim, `(and)` avaliado sem argumentos retorna, por default, `true`.

### `or`

O operador lógico `or` também é uma macro que aceita um número variável de argumentos
e os avalia da esquerda para a direita. Ele retorna o primeiro valor truthy encontrado,
se houver, ou o valor do último argumento, se todos avaliarem falsey. Veja alguns exemplos:

```clojure
(or 0 1 (= 1 1) 2) ;;=> 0
(or false nil 1 2) ;;=> 1
(or (seq []) (empty? [])) ;;=> true
(or) ;;=> nil
```

No primeiro exemplo, o primeiro argumento já é o primeiro valor truthy, `0`, e
portanto é o valor retorno pelo `or`. No segundo exemplo, o primeiro valor encontrado
avaliado como truthy é `1`, que é o valor retornado. No terceiro exemplo, `(seq [])`
avalia falsey (`nil`), mas `(empty? [])` avalia `true`, que é então retornado.
Por fim, `(or)` avaliado sem argumentos retorna, por default, `nil`.

### `not`

A macro `not` recebe apenas um argumento e retorna a sua negação lógica.
Exemplos:

```clojure
(not false) ;;=> true
(not nil) ;;=> true
(not true) ;;=> false
(not 0) ;;=> false
(not []) ;;=> false
(not not) ;;=> false
```

Para entender os exemplos, basta lembrar que apenas `false` e `nil` são considerados
falsey (logicamente falsos), enquanto _qualquer_ outro valor é considerado truthy
(logicamente verdadeiro).

### `complement`

A função `complement` não é um operador lógico e é um pouco mais complexa, mas
vale falar dela logo nesta seção. Essa função recebe outra função, `f`, e retorna
uma nova função que recebe os mesmos parâmetros que `f` e tem exatamente o mesmo
comportamento que `f`, mas retorna seu valor lógico negado. Veja alguns exemplos:

```clojure
(def not-zero? (complement zero?))
(not-zero? 1) ;;=> true
(not-zero? 0) ;;=> false

(defn print-empty?
    [coll]
    (if (seq coll)
      (do (println "It is not empty")
          false)
      (do (println "It is empty")
          true)))

(print-empty? [1 2])
;;# It is not empty
;;=> false

(print-empty? [])
;;# It is empty
;;=> true

(def not-print-empty? (complement print-empty?))

(not-print-empty? [1 2])
;;# It is not empty
;; true

(not-print-empty? [])
;;# It is empty
;; false
```

Na primeira parte, simplesmente negamos o resultado da função predicado `zero?`.
A função `not-zero?` retorna `true` se o valor não for zero, ou `false`, caso
contrário.

Na segunda parte, as coisas complicam um pouco (e ficam até confusas). A função
`print-empty?` exibe `"It is not empty"` e retorna `false` se seu argumento não
está vazio, e exibe `"It is empty"` e retorna `true` caso seu argumento esteja vazio.

A função `not-print-empty?` tem exatamente o mesmo comportamento da `print-empty?`,
incluindo mesmos prints, mas seu resultado é negado logicamente. Repare como isso
pode gerar uma certa confusão (e aparente inconsistência). De qualquer forma, a
função `complement` tem suas aplicações e vale a pena conhecê-la.

### `<`, `<=`, `=`, `>=`, `>`

Os comparadores de ordem também são bastante utilizados em contextos de condicionais.

A única novidade aqui em relação a outras linguagens é que eles são variádicos, e
isso traz a vantagem de poder comparar inúmeros valores de uma só vez:

```clojure
(< 1 2 3 4) ;;=> true
(< 1 2 4 3) ;;=> false
```

(Você pode pular as próximas sentenças, se quiser). Há uma forma elegante em Clojure
de verificar se uma dada coleção está ordenada (por exemplo, em ordem crescente)
usando `apply` e um comparador:

```clojure
(def ordered-coll [1 2 3 4])
(def not-ordered-coll [1 2 4 3])

(apply < ordered-col) ;;=> true
(apply < not-ordered-col) ;;=> false
```

A função `apply` simplesmente invoca a função passada como seu primeiro argumento
(no caso dos exemplos `<`) passando cada um dos elementos da coleção como argumentos
individuais dessa função. Isso é melhor explicado com um exemplo simples:

```clojure
(defn sum
  [x y]
  (+ x y))

(apply sum [1 2]) ;;=> 3
(sum 1 2) ;;=> 3
```

A função `sum` soma seus dois argumentos. A função `apply` "explode" o vector `[1 2]`
e passa `1` e `2` como argumentos individuais para `sum`. As duas invocações
do exemplo são equivalentes neste caso.

Agora podemos ver as estruturas de controle de fluxo.

## `if` e `if-not`

O controle de fluxo mais básico é feito com a forma especial (_special form_) `if`
e sua contraparte conveniente `if-not`.

O `if` (e `if-not`) recebe, na sua forma mais comumente usada, três expressões:

1. Uma condição (lembrando: apenas `false` e `nil` são logicamente falsos).
2. Uma expressão a ser executada, se a condição em 1 for verdadeira.
3. Uma expressão a ser executa, se a condição em 1 for falsa.

Dois exemplos:

```clojure
(if (= (+ 1 1) 2) "Still know how to do math" "Wat") ;;=> "Still know how to do math"

(if (= (* 2 2) 5) "Still know how to do math" "Wat") ;;=> "Wat"
```

No primeiro exemplo, a condição é dada pela expressão `(= (+ 1 1) 2)` que retorna `true`.
A expressão dada pela string (lembre: literais são expressões válidas) `"Still know how to do math"`
é executada se a condição for verdadeira, e a expressão `"Wat"` seria executada se a condição
fosse falsa.

No segundo exemplo, a condição é `(= (* 2 2) 5)` e as expressão para condição verdadeira e falsas
são as mesmas. Como o resultado da condição é `false`, o resultado do `if` é `"Wat"`. 

Vale lembrar, a string resultante se torna de fato o valor de toda expressão do `if`.

A contraparte `if-not` agora se torna intuitiva. Se a condição for falsey, i.e.
`false` ou `nil`, é executada a expressão seguinte à condição. Caso contrário, a
última expressão é executada. Ela é equivalente a `(if (not <condition>) <expr1> <expr2>)`

```clojure
(if-not (seq [1 2]) "Empty" "Not empty") ;;=> "Not empty"

(if-not (seq []) "Empty" "Not empty") ;;=> "Empty"
```

Importante tomar cuidado ao se querer executar múltiplas expressões tanto nas expressões
truthy, quanto falsey. Veja o exemplo __errado__ a seguir:

```clojure
(if (seq [1 2])
  (println "Oh yeah, it is...") ;; Truthy expressions.
  (println "not empty")
  
  (println "It would be bad, if...") ;; Falsey expressions.
  (println "it was empty"))
;;! Syntax error compiling if at (/tmp/form-init9742472599147237557.clj:1:1).
;;! Too many arguments to if
```

O que aconteceu? Bem `if` espera três expressões, mas foram passadas cinco expressões
(uma condição, duas expressões que esperávamos executar o caso verdadeiro, e duas
expressões para o caso falso). De fato, isso não funciona. Como executamos múltiplas
expressões em cada caso? Uma solução é usar o `do` que será visto em
[Estrutura do código](04-estrutura-do-codigo.md#do).

Veja o exemplo corrigido a seguir:

```clojure
(if (seq [1 2])
  (do ;; Truthy expressions.
    (println "Oh yeah, it is...")
    (println "not empty"))
  
  (do ;; Falsey expressions.
    (println "It would be bad, if...")
    (println "it was empty")))
;;# Oh yeah, it is...
;;# not empty
;;=> nil
```

Não existe `else` em Clojure. Mas isso não é um problema já que podemos aninhar
expressões `if`:

```clojure
(def x 2)

(if (= x 1)
  (println "x is 1")
  (if (= x 2)
    (println "x is 2")
    (println "I don't know what x is")))
;;# x is 2
;;=> nil
```

Vale notar que somente a expressão correspondente ao resultado da condição
é avaliada. A outra expressão não é avaliada e não consome nenhum tempo
de execução:

```clojure
(if (zero? (- 1 1))
    (println "1 minus 1 is zero")
    (some-really-costly-database-requisition!))
;;# 1 minus 1 is zero
;;=> nil
```

Também é possível omitir a última forma (caso falso) do `ìf` ou `if-not`. Nesse
caso, a última expressão é implicitamente `nil`:

```clojure
(if (seq [])
  (do ;; Truthy expressions.
    (println "Oh yeah, it is...")
    (println "not empty")))
;;=> nil
```

Essa forma, no entanto, não é tão popular já que existe uma alternativa melhor disponível.
É o que será mostrado com `when` e `when-not` a seguir.

## `when` e `when-not`

O `when` e `when-not` funcionam de forma bastante similar ao `if` e `if-not`, respectivamente.
As diferenças básicas são três:

1. `when` e `when-not` são macros (e não formas especiais).
2. A expressão a ser executada no caso verdadeiro do `when` e `when-not` podem ser
várias sem necessidade do `do`.
3. No caso falso, o valor final do `when` e `when-not` é `nil` implicitamente.

Alguns exemplos:

```clojure
(when (= (+ 1 1) 2) "Math is ok") ;;=> "Math is ok"
(when-not (seq [1 2]) "It is empty") ;;=> nil

(when (seq [1 2])
    (println "First item is 1")
    (println "First item is 2")
    (* 2 2))
;;# First item is 1
;;# First item is 2
;;=> 4
```

O `when` e `when-not` são particularmente úteis e convenientes (e preferíveis
em relação ao `if` e `if-not`) quando:

1. Só interessa o que fazer no caso verdadeiro.
2. Há várias expressões (possivelmente com side-effects, veremos) que devem
ser executadas no caso verdadeiro, e não queremos poluir o código com `do`.
3. É razoável aceitar `nil` como valor implícito no caso negativo.

## `case`

Outra macro de controle de fluxo condicional é o `case`. Essa ferramenta também
é popular em diversas outras linguagens, e sua semântica é basicamente a mesma aqui.

O `case` espera

1. Uma expressão a ser avaliada, que será comparada.
2. Pares `<left> <right>`. A expressão `<left>` deve ser uma constante, e não pode ser duplicada.
3. Uma expressão opcional `<default>`. Se presente, ele deve ser a última expressão.

Alguns exemplos:

```clojure
(case (inc 2)
    1 "Increment of 2 is 1"
    2 "Increment of 2 is 2"
    3 "Increment of 2 is 3"
    4 "Increment of 2 is 4"
    "I don't know how much it is")
;;=> "Increment of 2 is 3" 
```

Nesse caso, a expressão `(inc 2)`, cujo valor é `3`, foi avaliada e então comparada
com as expressões "da esquerda" (`1`, `2`...) não necessariamente em ordem. A primeira
comparação a retornar verdadeiro indica que a respectiva expressão "da direita" (neste caso,
`Increment of 2 is 3`) deve ser executada. O valor final do `case` é o valor
da última expressão avaliada por ele.

Se nenhuma das comparações resultar em verdadeiro, a expressão `<default>` é executada,
se presente. Se a expressão `<default>` não for provida, uma exceção é lançada:

```clojure
(case (* 2 4)
    2 "2 x 4 = 2"
    3 "2 x 4 = 3"
    4 "2 x 4 = 4"
    (str "Oh, 2 x 4 = " (* 2 4)))
;;=> "Oh, 2 x 4 = 8"

(case (* 2 4)
    2 "2 x 4 = 2"
    3 "2 x 4 = 3"
    4 "2 x 4 = 4")
;;! Execution error (IllegalArgumentException) at nsclojure.core/eval1681 (form-init9742472599147237557.clj:8).
;;! No matching clause: 8
```

Como deve ter notado, a função `str` simplesmente retorna uma string construída
a partir dos seus argumentos.

> No exemplos a seguir usaremos vectors e lists. Por ora, basta saber que
> vectors literais são criados com colchetes, `[,,,]`, enquanto lists literais
> são criadas com parênteses, `(,,,)`. Sim, todas essas expressões que foram
> usadas até aqui são listas (por isso, Lisp significa _LISt Processor_).

Também podemos aproveitar a comparação de vectors (veremos mais a frente) para criar
expressões como:

```clojure
(case [(>= 1 0) (>= 1 2)]
    [false false] "1 is not greater than zero, and 1 is not greater than 2"
    [false true] "1 is not greater than zero, and 1 is greater than 2"
    [true false] "1 is greater than zero, and 1 is not greater than 2"
    [true true] "1 is greater than zero, and 1 is greater than 2")
;;=> "1 is greater than zero, and 1 is not greater than 2"
```

Uma feature interessante do `case`, mas não tão conhecida (ou popular) é a comparação
dentre um conjunto de valores com o uso de lists:

```clojure
(case (inc 3)
    (1 2 3) "It is 1, 2, or 3"
    (4 5 6) "It is 4, 5, or 6")
;;=> "It is 4, 5, or 6"
```

O valor de `(inc 3)`, que é `4`, é comparado com os valores dentro das listas constantes
`(1 2 3)` e `(4 5 6)`. Como `4` se encontra na segunda lista, sua expressão "da direita"
(a string) é avaliada e esta se torna o valor final da expressão `case`.

## `cond`

A macro `cond` recebe vários pares de expressões `<left> <right>`. A primeira
expressão "da esquerda" a ser avaliada como verdadeira determina que sua respectiva
expressão "da direita" deve ser avaliada e se tornar o valor final da expressão `cond`.

Para criar um valor default, basta acrescentar qualquer expressão "da esquerda" que
avalie para um verdadeiro lógico. Alguns valores comuns são `:else`, `:default`, ou
até mesmo o óbvio `true`. Se nenhuma comparação retornar verdadeiro, e não houver
expressão default, o `cond` toma como valor `nil`:

```clojure
(def x 2)

(cond
    (= x 1) "x is 1"
    (= x 2) "x is 2"
    :else "x is something else")
;;=> "x is 2"

(def y (* 2 x))

(cond
    (= y 1) "y is 1"
    (= y 2) "y is 2"
    :else "y is something else")
;;=> "y is something else"

(cond
    (= y 1) "y is 1"
    (= y 2) "y is 2")
;;=> nil
```

## `loop` e `recur`

Agora serão mostrados mecanismos de loop, utilizando recursão. O primeiro mecanismo
usa a forma especial `recur`. No segundo mecanismo, será mostrado como se pode combinar
a forma especial `loop` com `recur` para criar recursões ainda mais flexíveis.

O `recur` sozinho é capaz de chamar a função onde ele está definido de forma recursiva.
Funções serão mostradas profundamente mais a frente, mas por enquanto basta saber
que podemos definir uma função da seguinte forma:

```clojure
(defn <fn-name>
  [<arguments>]
  <body-exprs>)
```

Por exemplo, para criar a função `square-or-inc` que dobra seu argumento se este
for par, ou o incrementa se for ímpar:

```clojure
(defn square-or-inc
    [x]
    (if (even? x)
      (* 2 x)
      (inc x)))

(square-or-inc 2) ;;=> 4
(square-or-inc 5) ;;=> 6
```

Quando usado sozinho, `recur` aceita tantos argumentos quanto a função onde ele
se encontra. Quando a execução atinge o `recur` no corpo da função, a função
é novamente chamada, tendo como argumentos os valores das expressões no `recur`.

Para exemplificar, considere a função, `factorial`, que calcula o fatorial de um dado número:

```clojure
(defn factorial
    [fat x]
    (if (zero? x)
      fat
      (recur (* fat x) (dec x))))

(factorial 1 0) ;;=> 1
(factorial 1 1) ;;=> 1
(factorial 1 2) ;;=> 2
(factorial 1 3) ;;=> 6
(factorial 1 4) ;;=> 24
(factorial 1 5) ;;=> 120
(factorial 1 10) ;;=> 3628800
```

A função `factorial` recebe dois parâmetros, `fat` e `x`. O primeiro, `fat`, deve
ser sempre `1` (já vamos ver como resolver isso com `loop`). O segundo é o número
o qual se quer calcular o fatorial.

Vamos estrinchar a execução de `(factorial 1 3)`.

Assim que é chamada, `fat` é `1`, e `x` é `3`. Como `x` não é zero, a expressão
do `recur` é avaliada. Note que existem duas expressões `(* fat x)` e `(dec x)`
no `recur`. Ambas são avaliadas com os valores atuais de `fat` e `x`,
resultando em `3` e `2`, respectivamente. O `recur` então causa uma nova
chamada à função `factorial`, passando `3` e `2` como os novos argumentos.

Nessa segunda chamada, `fat` é `3` e `x` é 2. Novamente, como `x` não é zero,
a expressão do `recur` é novamente avaliada. Desta vez, `(* fat x)` resulta em
`6` e `(dec x)` resulta em `1`. Novamente, `recur` causa uma nova chamada à
`factorial` com os novos argumentos `6` e `1`.

Na terceira chamada, `fat` é `6` e `x` é `1`. Ainda, `x` não é zero, e `recur`
novamente é avaliada. Os resultados da avaliação de `(* fat x)` e `(dec x)` são,
respectivamente, `6` e `0`. Após avaliadas, `recur` causa uma nova chamada, passando
`6` e `0` como argumentos à `factorial`.

Na quarta (e última chamada), `fat` é 6 e `x` é `0`. Como `x` é finalmente zero,
a expressão `fat` do `if` é avaliada. Essa expressão tem valor `6`, e ela se torna o valor
final de toda expressão `if`, e por fim, de toda função `factorial`.

Um inconveniente dessa abordagem é que o valor inicial de `fat` é exposto na API
da função, obrigando seus usuários a lembrarem de passar o valor `1` para o correto
funcionamento da função.

A forma especial `loop` provê um ponto de recursão para o `recur`, dentro da função.
A função `factorial` poderia ser reescrita assim com `loop` e `recur`:

```clojure
(defn factorial-loop
    [n]
    (loop [fat 1 x n]
        (if (zero? x)
          fat
          (recur (* fat x) (dec x)))))

(factorial-loop 0) ;;=> 1
(factorial-loop 1) ;;=> 1
(factorial-loop 2) ;;=> 2
(factorial-loop 3) ;;=> 6
(factorial-loop 4) ;;=> 24
(factorial-loop 5) ;;=> 120
(factorial-loop 10) ;;=> 3628800
```

Note que `loop` aceita um vector como primeira expressão (`[fat 1 x n]`). Nesse vector, tem-se
pares `<variable> <initial-value>`. Isso significa que o valor inicial de `fat`
será `1` e o valor inicial de `x` será o valor de `n` passado para a função.

Agora, em vez de `recur` causar uma nova chamada à função, a execução simplesmente
saltará para o `loop`, associando a `fat` o valor avaliado de `(* fat x)` e a `x`
o valor avaliado de `(dec x)`. Note como o valor inicial de `fat`, que deve ser `1`,
agora é especificado dentro da função, removendo a necessidade de expor essa configuração na
API da função e de forçar os usuários a lembrar dela.

Apenas um detalhe sobre a forma especial `recur`: ela deve vir sempre na _tail position_.
Uma expressão estar na tail position significa que nenhuma outra expressão deve ser passível
de ser avaliada depois dela no contexto da função.

Por exemplo, se fosse adicionado um simples `println` após o `recur`, não seria
possível nem ao menos definir corretamente a função sem que o Clojure reclamasse,
lançando uma exceção:

```clojure
(defn factorial
    [fat x]
    (if (zero? x)
      fat
      (do
        (recur (* fat x) (dec x))
        (println "fat =" fat " x =" x))))
;;! Syntax error (UnsupportedOperationException) compiling recur at (/tmp/form-init9742472599147237557.clj:6:7).
;;! Can only recur from tail position
```