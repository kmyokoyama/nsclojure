# Funções

Clojure ẽ um Lisp com um forte gosto funcional. Como deve ser óbvio, funções
desempenham um papel central no paradigma funcional.

## Invocando funções

Invocar uma função em Clojure é tão simples quanto colocá-la entre parênteses
junto com seus argumentos. A função invocada deve estar sempre na posição inicial
(_header position_) de uma form.

Por exemplo, considere a função `+:

```clojure
(+ 1 2 3 -5 4.5) ;;=> 5.5
```

Não existe surpresa para invocar uma função sem passar argumentos, simplesmente
ponha o nome da função entre parênteses, como esperado:

```clojure
(+) ;;=> 0
```

A função `+` aceita um número qualquer de argumentos (incluindo nenhum argumento)
e retorna a soma de todos eles. Se nenhum argumento for passado, ela retorna zero.

Sempre que uma form for avaliada, Clojure espera que um dos três - forma especial,
macro ou função - esteja na posição inicial da form.

Para ser qualificada como função, basta que a entidade em questão implemente
a interface `clojure.lang.IFn`. Como veremos mais a frente, algumas coisas
pouco usuais, como maps e vectors, também implementam essa interface em Clojure.

Tentar avaliar uma form cuja posição inicial não é ocupada por uma forma especial,
macro ou função causa uma exceção ser lançada em tempo de execução.

```clojure
("not-a-function" 1 2) ;;! Execution error (ClassCastException) at nsclojure.core/eval1559 (form-init12043839654965885206.clj:1).
                       ;;! java.lang.String cannot be cast to clojure.lang.IFn
```

## Definindo funções

Clojure já vêm com centenas de funções muito úteis, mas mais cedo do que tarde,
é preciso criar as suas próprias funções.

Existem algumas formas de fazer isso, e a mais comum delas é com a macro `defn`.
Essa macro tem a seguinte forma:

```clojure
(defn <fn-name>
    <optional-docstring>
    [<parameters>]
    <fn-body>)
```

Vamos criar uma função extremamente simples que soma dois argumentos:

```clojure
(defn sum-two
    "Sums its two arguments"
    [x y]
    (println "Executing the function")
    (+ x y))

(sum-two 2 3)
;;# Executing the function
;;=> 5
```

No exemplo acima, `sum-two` é o nome da função. Adicionamos uma docstring,
`"Sums its two arguments"`, para explicar o que a função faz (docstrings
são opcionais e podem conter múltiplas linhas). A lista de parâmetros formais tem
dois parâmetros posicionais `x` e `y`. Por fim, o corpo da função é simplesmente
as forms ` (println "Executing the function") (+ x y)`.

Note duas coisas:

1. O corpo da função tem um `do` implícito, ou seja, podemos sequenciar
várias forms para execução sem a necessidade de incluir um `do`.

2. Não há instrução de `return`. __O retorno de qualquer função é o valor
da última expressão avaliada dentro da função__. Por conseguinte, toda função tem
retorno (mesmo que este seja implicitamente `nil`).

Em Clojure, todos parâmetros são posicionais. Não existem parâmetros nomeados,
opcionais* ou mesmo com valor default.

> *: Não existir parâmetros opcionais é apenas uma meia verdade. Existe a possibilidade
> de se definir uma função que aceite um número indefinido de parâmetros, os quais
> podem ser usados como workaround para passar argumentos adicionais. Mas isso
> não é tão comum, nem idiomático.

## Funções anônimas

Em uma linguagem funcional, criar funções e utilizá-las em todos os lugares é
algo corriqueiro. Seria uma pena se funções, por mais simples que fossem, tivessem
que ser sempre previamente definidas, nomeadas, para então serem utilizadas.

Obviamente esse não é o caso em Clojure. Para facilitar o uso de funções de
forma menos burocrática, existem duas formas de criar funções ad-hoc.

### `fn`

A primeira é usando a forma especial `fn`. Essa forma especial tem exatamente
a mesma constituição do `defn`. Porém, ao contrário do que acontece com `defn`,
a `fn` não cria um nome no namespace atual que possa ser referenciado depois.

Sua utilidade maior é quando precisamos definir e utilizar a função no mesmo lugar,
ao mesmo tempo. Esse é um cenário bastante comum quando temos funções
de ordem superior, i.e., funções que recebem e retornam funções.

Por exemplo, considere o seguinte exemplo onde criamos uma função `apply-to-double`,
que recebe uma função `f` de um único argumento e um número e aplica a função
ao dobro do número:

```clojure
(defn apply-to-double
    [f x]
    (f (* 2 x)))
```

Note que, por ser esperado que `f` seja uma função passada como argumento, podemos
utilizar `f` na primeira posição da form.

Podemos definir algumas funções para serem passadas no lugar de `f`:

```clojure
(defn print-double-msg
    [x]
    (println "The double is" x))


(defn increment
    [x]
    (inc x))


(apply-to-double print-double-msg 2) ;;=> "The double is 4"

(apply-to-double increment 2) ;;=> 5
```

No entanto, é um tanto chato ter que definir essas funções simplesmente para
utilizá-las logo em seguida. Podemos usar então `fn`:

```clojure
(apply-to-double (fn [x] (println "The double is" x)) 2) ;;=> "The double is 4"

(apply-to-double (fn [x] (inc x)) 2) ;;=> 5
```

No exemplo acima definimos as funções e as utilizamos no mesmo lugar.

No caso do segundo exemplo, poderíamos ter simplificado ainda mais. Como tudo que
essa função está fazendo é repassar o seu parâmetro para a função `inc`, podemos
tirar essa camada intermediária e usar `inc` diretamente:

```clojure
(apply-to-double inc 2) ;;=> 5
```

Funções criadas com `fn` não são exatamente anônimas. É possível sim especificar
um nome para as funções criadas dessa forma com o intuito de facilitar o debug.
Veja a diferença com o exemplo a seguir:

```clojure
(apply-to-double (fn [x] (/ x 0)) 2)
;;! Execution error (ArithmeticException) at nsclojure.core/eval1591$fn (form-init12043839654965885206.clj:1).
;;! Divide by zero

(apply-to-double (fn bad-division [x] (/ x 0)) 2)
;;! Execution error (ArithmeticException) at nsclojure.core/eval1603$bad-division (form-init12043839654965885206.clj:1).
;;! Divide by zero
```

Olhe atentamente as exceções e note como no primeiro caso aparece apenas `fn`
sem especificar mais detalhes (imagine debugar isso em uma aplicação com milhares
de linhas de código), enquanto no segundo caso aparece `bad-division`. Apesar
desse benefício, essa funcionalidade do `fn` é bastante rara de ser usada.

Por fim, podemos atribuir o resultado de uma expressão `fn` a um nome com `def`
e utilizar esse nome posteriormente. De fato, é exatamente isso que `defn` (o nome
é uma contração de `def` com `fn`) faz para nós:

```clojure
(def print-double-message (fn [x] (println "The double is" x)))

(defn print-double-message
    [x]
    (println "The double is" x))
```

As duas formas acima produzem o mesmo resultado. No entanto, prefira sempre usar
`defn` em vez de `def` + `fn`.

### `#()`

A segunda forma de criar funções anônimas é com o reader `#()`. Poderíamos
ter escrito os exemplos acima com `#()`:

```clojure
(apply-to-double #(println "The double is" %) 2) ;;=> "The double is 4"

(apply-to-double #(inc %) 2) ;;=> 5
```

Com `#()` não se tem mais uma lista de parâmetros formais como se tinha com ambos
`defn` e `fn`. O primeiro parâmetro passado para `#()` é representado pelo `%`,
ou `%1`. Se a função receber mais de um parâmetro, os demais parâmetros são
representados por `%2`, `%3`, `%4`, e assim por diante:

```clojure
(defn apply-then-double
  [f x y z w]
  (* 2 (f x y z w)))

(apply-then-double #(+ (* %1 %2) (* %3 %4)) 10 20 30 40) ;;=> 2800
```

É bom ter atenção ao usar `#()`, pois seu caso de uso ideal é para funções
anônimas que possuam apenas um form em seu corpo. Isso significa que ele é ideal
para escrever funções que teriam a seguinte estrutura equivalente com `fn`:

```clojure
(fn some-fn
    [,,,]
    (some-form ,,,)) ;; <- Single form!

#(some-form ,,,)
```

Obviamente podemos contornar essa restrição com `do`, __mas isso não é uma
boa prática__:

```clojure
(fn other-fn
    [,,,]
    (first-form ,,,)
    (second-form ,,,)) ;; <- Two (or more) forms.

;; BAD: Don't do this!
#(do (first-form ,,,) (second-form ,,,))
```

A regra de ouro é: se tiver muitos parâmetros ou se dar nome aos parâmetros
fizer uma diferença na legibilidade, prefira usar `fn`. Reserve `#()` para funções
simples, com apenas uma form e poucos argumentos (idealmente um ou dois).

## Funções privadas

Ainda não abordamos namespaces, mas vale apresentar um assunto que serã repetido
quando abordarmos. Esse assunto são as funções privadas do namespace.

Brevemente, um namespace é uma forma de criar um agrupamento lógico de nomes
e evitar colisões de nomes. Uma aplicação normalmente possui algumas dezenas de namespaces,
que separam de forma lógica partes de sua implementação (funções, macros, constantes etc).

Como em outras linguagens, Clojure provê um jeito de tornar alguns nomes privadas,
acessíveis apenas dentro do namespace onde foram definidos.

Uma forma geral de tornar privado um nome é utilizando metadata como no exemplo
a seguir:

```clojure
(defn ^:private some-private-fn
    [,,,]
    (,,,))

(def ^:private some-constant 3.14159265M)
```

No exemplo acima, ambos `some-private-fn` e `some-constant` só são acessíveis
por seus nomes dentro do namespace onde foram definidos.

Essa anotação estranha `^:private` (ou de forma mais verbosa `^{:private true}`)
é uma forma de adicionar metadados a Var (veremos exatamente o que é um Var mais a frente).
Podemos até verificar que de fato o metadata foi criado usando a função
`meta` e passando a Var (com `#'`):

```clojure
(meta #'some-private-fn)
;;=> {:private true,
;;=>  :arglists ([]),
;;=>  :line 1,
;;=>  :column 1,
;;=>  :file "/tmp/form-init12043839654965885206.clj",
;;=>  :name some-private-fn,
;;=>  :ns #object[clojure.lang.Namespace 0x1894a65a "nsclojure.core"]}
```

Na verdade, existe até uma macro, syntatic sugar, análoga a `defn` para definir
funções privadas do namespace, `defn-`. No exemplo a seguir, a definição alternativa
de `some-private-fn` usando `defn-` é exatamente equivalente a do exemplo anterior:

```clojure
(defn- some-private-fn
    [,,,]
    (,,,))
```

Não existe consenso de quando se deve usar uma forma ou outra, mas parece que
adicionar metadata com `^:private` está ganhando. Um dos motivos é que ele é
mais geral e permite tornar privadas várias entidades para as quais não existe
um syntatic sugar análogo, criando código mais padrão e consistente.
Por exemplo, não existe um `def-` para criar constantes privadas,
sendo necessário utilizar `^:private`.

Quando falarmos de namespaces, veremos uma forma de contornar o modo privado
e acessar nomes privados de outros namespaces. Isso pode ser útil para testes.

## Multiaridade (multiarity)

Uma feature de funções bastante útil é permitir definir a mesma função múltiplas
vezes com diferentes números de parâmetros. Por exemplo, é possível criar
uma função que recebe um _ou_ dois argumentos:

```clojure
(defn print-multiarity
    ([x]
     (println "I've received only one argument:" x)
     x)

    ([x y]
     (println "I've received two arguments:" x "and" y)
     (+ x y)))

(print-multiarity 10)
;;# I've received only one argument: 10
;;=> 10

(print-multiarity 10 20)
;;# I've received two arguments: 10 and 20
;;=> 30
```

Repare como cada definição da função possui sua própria s-expression.

Essa feature é útil para emular valores default:

```clojure
(defn greet
  ([name]
   (greet name "Hi"))

  ([name greeting]
   (println (str greeting ",") name)))

(greet "Ann") ;;# Hi, Ann
(greet "Ann" "Hello") ;;# Hello, Ann
```

Note como no caso onde a função recebe apenas um parâmetro, a função invoca a
ela mesma, passando um valor default (`"Hi"`) como segundo argumento. Essa técnica
é bastante comum e útil.

## Funções variádicas

Funções variádicas são aquelas que recebem um número variável (não definido) de
parâmetros. Em Clojure, funções variádicas representam seus parâmetros adicionais
declarando-os após o símbolo `&` como no exemplo a seguir:

```clojure
(defn variadic-fn
    [x y & zs]
    (println "x:" x "y:" y "rest:" zs))

(variadic-fn 1 2)
;;# x: 1 y: 2 rest: nil
;;=> nil

(variadic-fn 1 2 3 4 5)
;;# x: 1 y: 2 rest: (3 4 5)
;;=> nil
```

No exemplo acima, a função `variadic-fn` recebe dois parâmetros posicionais obrigatórios,
`x` e `y`, e um número indefinido de argumentos adicionais, opcionais.

Os poossíveis valores adicionais passados para `variadic-fn` são coletados
em uma sequence representada por `zs`. Se nenhum argumento adicional for passado,
`zs` é `nil`.

Importante notar que apenas um símbolo pode vir depois de `&` para representar
a sequence de argumentos adicionais. Além disso, ela deve ser a última coisa na lista
de parâmetros formais da função.

A mesma sintaxe se repete para funções anônimas definidas com `fn`:

```clojure
(def anon-variadic-fn (fn [x y & zs] (println "x:" x "y:" y "rest:" zs)))

(anon-variadic-fn 1 2)
;;# x: 1 y: 2 rest: nil
;;=> nil

(anon-variadic-fn 1 2 3 4 5)
;;# x: 1 y: 2 rest: (3 4 5)
;;=> nil
```

No caso de `#()`, a sintaxe é um pouco diferente. A sequence nesse caso é representada por `%&`:

```clojure
(def another-anon-variadic-fn #(println "x:" % "y:" %2 "rest:" %&))

(another-anon-variadic-fn 1 2)
;;# x: 1 y: 2 rest: nil
;;=> nil

(another-anon-variadic-fn 1 2 3 4 5)
;;# x: 1 y: 2 rest: (3 4 5)
;;=> nil
```

## Funções de ordem superior

Funções de ordem superior são funções que recebem outras funções como parâmetro
ou retornam uma função como resultado. Já criamos algumas funções de ordem
superior nesta seção - `apply-to-double` e `apply-then-double` são dois exemplos.

Podemos também criar uma função que retorna outra função, aproveitando a vantagem
de closures e escopo léxico do Clojure (não vamos entrar em detalhes):

```clojure
(defn inc-by
    [n]
    (fn [x] (+ x n)))

(def inc-by-2 (inc-by 2))
(inc-by-2 5) ;;=> 7
(inc-by-2 8) ;;=> 10

(def inc-by-3 (inc-by 3))
(inc-by-3 5) ;;=> 8
```

A função `inc-by` recebe um número, `n`, que representa o incremento e retorna _uma
nova função_, que por sua vez recebe um número, `x`, e retorna `x` incrementado de `n`.
Note como as invocações `(inc-by 2)` e `(inc-by 3)` retornam funções, logo
`inc-by-2` e `inc-by-3` são duas novas funções.

Isso é possível por causa que Clojure implementa o conceito de _closure_. O valor
de `n` passado para `inc-by` é "lembrado" pela função anônima sendo retornada.
Por exemplo, no caso de `(inc-by 2)`, a função anônima retornada é `(fn [x] (+ x 2))`.
Nesse caso, dizemos que a função anônima retornada é uma closure.

> Trivia: o nome da linguagem é um trocadilho de closure com J de Java.

Existem diversas funções de ordem superior úteis no Clojure e em diversas libs.
Aqui, serão apresentadas três das mais populares em programação funcional: `map`,
`filter` e `reduce`.

### `map`

Clojure não provê nem utiliza muitas estruturas de iteração clássicas de outras
linguagens, como `for` (existe em Clojure, mas é outra ideia), `while` e `until`.
Em vez disso, segue-se a filosofia de transformação de dados, utilizando
funções de ordem superior e abstrações de dados consistentes.

A função `map` é uma dessas funções de ordem superior. Ela aceita uma função e
um número qualquer de collections. Veremos o que são collections mais pra frente,
mas por ora, considere um exemplo de collection um vector `[,,,]`.

O uso mais popular de `map` é com apenas uma collection. Neste caso, `map`
atravessa a collection aplicando a função em cada elemento individualmente, criando
uma nova collection de saída com os resultados.

Por exemplo, se ela recebe uma função qualquer `f` (de um parâmetro) e uma
collection `[1 2 3 4]`, o resultado do `map` será uma nova collection `((f 1) (f 2) (f 3) (f 4))`:

```clojure
(map #(* % %) [1 2 3 4]) ;;=> (1 4 9 16)
```

Note como o resultado não foi um vector, mas sim algo com parênteses. Isso é
basicamente uma sequence. Falaremos mais sobre sequence e porque `map` se comporta
assim mais a frente.

Se mais de uma collection for passada para `map`, ela tem o comportamento exemplificado
a seguir. Considere que `map` recebeu uma função `f` (de dois parâmetros) e duas
collections `[1 2 3 4]` e `[10 20 30 40]`. O resultado será `[(f 1 10) (f 2 20) (f 3 30) (f 4 40)]`.
Veja um exemplo simples (lembre-se que `+` é uma função variádica):

```clojure
(map + [1 2 3 4] [10 20 30]) ;;=> (11 22 33)
```

Note um detalhe: a segunda collection é mais curta que a primeira, fazendo com que
o `map` encerre assim que a collection mais curta se esgote.

O comportamento de `map` é análogo se três ou mais collections forem passadas como argumento.

A função `map` é um pouco mais rica do que o básico mostrado aqui. Ela também trabalha
com o conceito de transdutores, pode ser aplicada em maps (dicionários) etc.

### `filter`

Uma tarefa bastante recorrente é filtrar uma collection a partir de uma determinada
condição, mantendo apenas os elementos da collection que satisfizerem essa condição.
Para isso existe a função de ordem superior `filter`.

Na sua forma mais usual, a função `filter` recebe uma função e uma collection.
O `filter` então invoca a função em cada um dos elementos da collection,
criando uma nova collection de saída com aqueles elementos que retornaram truthy:

```clojure
(filter pos? [0 -1 1 2 -0.25 10 15]) ;;=> (1 2 10 15)
(filter identity [0 [] false nil true 1 :a]) ;;=> (0 [] true 1 :a)
(filter some? [0 [] false nil true 1 :a]) ;;=> (0 [] false true 1 :a)
```

No primeiro exemplo, a função predicado `pos?` retorna `true` se seu argumento for
estritamente positivo. O `filter` então remove todos valores não-positivos da collection.

No segundo exemplo, a função `identity` retorna exatamente seu argumento. Como
ambos `false` e `nil` são falsey, eles são removidos da collection de saída.

No terceiro exemplo, a função `some?` retorna `false` apenas quando seu argumento é `nil`.
Por conseguinte, apenas `nil` é filtrado da lista de saída do `filter`.

### `reduce`

A função `reduce` tem um funcionamento um pouco mais complexo que as duas anteriores,
então vamos destrinchar cada um dos possíveis casos com exemplos.

Essa função tem duas definições. Na primeira, ela recebe dois parâmetros: uma função, `f`,
de dois parâmetros e uma collection. Na segunda, ela recebe `f`, um valor inicial, e a collection.

1. `reduce` recebe `f` e `coll`:  
    1.1. Se `coll` é vazio, então `f` deve ser no-arg, e `reduce` retorna `(f)`:  
    ```clojure
    (reduce + []) ;;=> 0
    ;; (+) => 0
   
    (reduce * []) ;;=> 1
    ;; (*) => 1
    ```
   
    1.2. Se `coll` contém apenas um elemento, esse elemento é retornado e `f` não é invocada:
    ```clojure
    (reduce #(/ 1 0) [10]) ;;=> 10
    ```
   
    1.3. Se `coll` contém dois ou mais elementos, `reduce` invoca `f` com os dois
    primeiros elementos de `coll`, depois com o resultado dessa invocação e
    o próximo elemento, e assim por diante:
    ```clojure
    (reduce + [10 20 30 40 50]) ;;=> 150
    ;; (+ 10 20)  => 30
    ;; (+ 30 30)  => 60
    ;; (+ 60 40)  => 100
    ;; (+ 100 50) => 150
    ```
    
2. `reduce` recebe `f`, `val` e `coll`:  
    1.1. Se `coll` é vazio, `reduce` retorna `val`:
    ```clojure
    (reduce + 10 []) ;;=> 10
    ```
   
    1.2. Se `coll` é não vazio, `reduce` invoca `f` com `val` e o primeiro elemento
    de `coll`, depois o resultado dessa invocação e o segundo elemento, e assim por diante:
    ```clojure
    (reduce + 100 [10 20 30 40 50]) ;;=> 250
    ;; (+ 100 10) => 110
    ;; (+ 110 20) => 130
    ;; (+ 130 30) => 160
    ;; (+ 160 40) => 200
    ;; (+ 200 50) => 250
    ```

A maior parte desses casos pode ser considerada corner-case e só foi apresentada
por questão de completude. O que realmente importa é entender o funcionamento
do `reduce` quando `coll` tem elementos suficiente (casos 1.3 e 2.2).

Muitos problemas podem ser resolvidos de forma extremamente elegante com `reduce`.
Quem estiver interessado, vale a pena estudar essa função do ponto de vista
da linguagem Haskell e suas funções `foldr` e `foldl` com estrutura recursiva.