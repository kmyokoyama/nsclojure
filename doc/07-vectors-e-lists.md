# Vectors e lists

Nesta seção, falaremos sobre as nossas primeiras estruturas de dados, os vectors (vetores)
e lists (listas). Eu não vou traduzir esses termos de propósito. Eles são bastante difundidos
e tratá-los por seus nomes originais em inglês vai facilitar a assimilação mais a frente, principalmente
dos nomes das funções.

Ambas estruturas representam sequências ordenadas de elementos, mas com algumas diferenças
entre elas. Vamos começar pelos vectors.

## Vectors

Em Clojure, um vector literal pode ser criado com colchetes `[ ]`:

```clojure
[] ;;=> []
[1 2 3] ;;=> [1 2 3]
[:a "this" 1 10.2M] ;;=> [:a "this" 1 10.2M]
```

No primeiro exemplo, foi criado um vector vazio. No segundo exemplo, foi criado um vector
onde seus três elementos são numéricos. Mas vectors não precisam ser homogêneos quanto aos
tipos de dados dos seus elementos. Isso é mostrado no terceiro elemento, onde temos quatro elementos,
sendo uma keyword, uma string, um Long e um BigDecimal.

Note como os elementos **não** são separados por vírgulas como de costume em outras linguagens.
Em Clojure, vírgulas são interpretadas como espaços em branco pelo compilador e ignoradas:

```clojure
[,,,] ;;=> []
```

Vectors em Clojure são estruturas de dados adequadas para acesso na sua ponta direita, i.e., no fim do vector.
Isso significa que operações que adicionam, removem ou acessam no final do vector possuem bom desempenho.
Em termos de acesso  aleatório (e.g., por um índice), vectors também são bastante indicados.
Por outro lado, adicionar ou remover no começo (ponta esquerda) não é uma tarefa muito performática. 

Vamos conhecer as principais funções relativas a vectors na ordem CRUD (Create–Read–Update–Delete). 
Para todos os exemplos a seguir, usaremos os seguintes dados:

```clojure
(def v [10 20 30 40])

(def v-nil nil)

(def v-nested [[10 20] [30 40]])
```

### Construindo vectors

Vectors também podem ser construídos com funções. As duas principais são `vector` e `vec`.
A função `vector` recebe um número arbitrário de argumentos e retorna um vector contendo esses
argumentos, na mesma ordem em que eles são passados:

```clojure
(vector :first 2 "third") ;;=> [:first 2 "third"]
```

A função `vec` por sua vez recebe uma coleção de elementos (como uma list ou até mesmo um map)
e retorna um vector:

```clojure
(vec '(:first 2 "third")) ;;=> [:first 2 "third"]
```

Não se preocupe com o `'()` por agora. Já veremos o que ele significa quando falarmos de lists.

### Acessando vectors

Temos basicamente três formas de acessar elementos em um vector:

1. Usando o vector como uma função:

```clojure
(v 1) ;;=> 20

(v 3)
;;! Execution error (IndexOutOfBoundsException) at nsclojure.core/eval1587 (form-init7967620353117370647.clj:1).
;;! null

(v-nil 1)
;;! Execution error (NullPointerException) at nsclojure.core/eval1593 (form-init7967620353117370647.clj:1).
;;! Cannot invoke "clojure.lang.IFn.invoke(Object)" because the return value of "clojure.lang.Var.getRawRoot()" is null
```

Como pode ver, índices começam em zero. Se tentarmos acessar um índice fora dos limites do
vector, recebemos uma exception. Também recebemos uma exception se por acaso o vector for `nil`.

Um ponto interessante é que neste caso, o vector se comporta como uma collection associativa (como um map), onde
suas chaves são os seus índices válidos e os valores são os elementos nesses índices.

2. Usando a função `nth`:

```clojure
(nth v 1) ;;=> 20

(nth v 3)
;;! Execution error (IndexOutOfBoundsException) at nsclojure.core/eval1601 (form-init7967620353117370647.clj:1).
;;! null

(nth v-nil 1) ;;=> nil
```

Novamente, ao acessar um índice fora dos limite do vector, recebemos uma exceção de `IndexOutOfBoundsException`.
Porém, ao acessar um vector `nil`, agora recebemos `nil`, o que um pouco menos agressivo.

3. Usando a função `get`:

```clojure
(get v 1) ;;=> 20

(get v 3) ;;=> nil

(get v-nil 1) ;;=> nil
```

Com a função `get`, recebemos `nil` em ambos casos, quando tentamos acessar um índice fora
dos limites do vector e quando usamos um vector `nil` por acaso.

Também é possível acessar vectors aninhados com a função `get-in`:

```clojure
(get-in v-nested [0 1]) ;;=> 20
```

No exemplo acima, acessar o índice 0 do vector externo e o índice 1 do vector interno, retornando o elemento 20.

Clojure possui uma função para acessar o último elemento de um vector através da função `peek`:

```clojure
(peek v) ;;=> 40
```

### Modificando vectors

Se você leu a seção anterior, provavelmente está se pergunta "Mas não é tudo imutável?".
E a resposta é "Sim!". Tudo continua imutável. Na verdade, não estaremos modificando
a estrutura original, mas sim criando uma nova estrutura com as modificação a partir da original.

Temos algumas formas de modificar elementos em vectors. A primeira delas é com `conj`:

```clojure
(conj v 50) ;;=> [10 20 30 40 50]
```

Quando aplicado a um vector, `conj` faz a operação de append no vector, i.e., adiciona o elemento ao final do vector.
Também é possível adicionar múltiplos elementos de uma só vez:

```clojure
(conj v 50 60 70 80) ;;=> [10 20 30 40 50 60 70 80]

v ;;=> [10 20 30 40]
```

Note no exemplo acima como o retorno do `conj` é um novo vetor com as modificações desejadas. No entanto, o vector
original, no caso `v`, não foi alterado. Isso também vale para qualquer uma dos casos a seguir.

Uma função similar é `cons`:

```clojure
(cons 0 v) ;;=> (0 10 20 30 40)
```

A função `cons` sempre adiciona o novo elemento no começo da collection e retorna uma nova sequência.

Uma outra função bastante útil é `assoc`:

```clojure
(assoc v 1 2000) ;;=> [10 2000 30 40]
```

Também podemos usar a função `assoc-in` para modificar vectors aninhados:

```clojure
(assoc-in v-nested [0 1] 2000) ;;=> [[10 2000] [30 40]]
```

Outra função muito útil é a `update`, que permite modificarmos um elemento através da aplicação de uma função:

```clojure
(update v 1 #(* % 1000)) ;;=> [10 2000 30 40]
```

E também existe sua versão aninhada `update-in`:

```clojure
(update-in v-nested [0 1] #(* % 1000)) ;;=> [[10 2000] [30 40]]
```

Note que passamos uma função de um argumento como último argumento de `update` e `update-in`. Essa função
receberá o elemento naquele determinado índice e o seu retorno será o novo elemento naquele determinado índice.

Outra função útil para modificar vector é `replace`. Essa função recebe um map com as modificações a serem
feitas como primeiro argumento e o vector como segundo argumento:

```clojure
(replace {20 2000} v) ;;=> [10 2000 30 40]
```

### Removendo elementos

Como dito, operações que atuam sobre o final do vector são mais performáticas. A função `pop` remove um
elemento do final do vector e retorna o vector sem este elemento:

```clojure
(pop v) ;;=> [10 20 30]
```

Remover elementos do meio de um vector não é uma tarefa tão usual. Talvez por isso, Clojure não tenha uma forma mais direta de realizar
essa tarefa. Uma das formas mais indicadas para isso é utilizando as funções `subvec` e `concat`.

A função `subvec` retorna um subvector do vector original. Essa função recebe o vector, um índice de ínicio, inclusivo,
e um índice de fim, exclusivo:

```clojure
(subvec v 1) ;;=> [20 30]

(subvec v 0 2) ;;=> [10 20]
```

A função `concat` simplesmente concatena sequências (e.g., vectors) e retorna uma nova sequência:

```clojure
(concat [10 20] [30 40]) ;;=> (10 20 30 40)
```

Com `subvec` e `concat` podemos realizar a remoção de um elemento em um determinado índice. Vamos escrever uma função para isso:

```clojure
(defn remove-nth
  [v idx]
  (vec (concat (subvec v 0 idx) (subvec v (inc idx) (count v)))))

(remove-nth v 1) ;;=> [10 30]
```

### Outras funções úteis

Vale conhecer o que outras funções fazem (e não fazem) com vectors. Por exemplo, a função `count` retorna
a quantidade de elementos em um vector:

```clojure
(count []) ;;=> 0

(count v) ;;=> 4
```

Apesar de existir a função `empty?`,

```clojure
(empty? []) ;;=> true

(empty? v) ;;=> false
```

a forma mais idiomática de verificar se um vector está vazio é com a função `seq`

```clojure
(seq []) ;;=> nil

(seq v) ;;=> (10 20 30 40)
```

Outras duas função interessantes são `rest` e `next`. Ambas retornam o vector original com exceção do seu primeiro
elemento:

```clojure
(rest v) ;;=> (20 30 40)

(next v) ;;=> (20 30 40)
```

A grande diferença das duas está quando aplicadas a vectors (ou sequências, como veremos mais adiante) vazios
ou `nil`:

```clojure
(rest []) ;;=> ()
(rest nil) ;;=> ()

(next []) ;;=> nil
(next nil) ;;=> nil
```

Como vocë pode ver, quando aplicado a um vector vazio ou `nil`, `rest` sempre retornará uma sequência vazia. A
função `next` por outro lado, retornará `nil` nesses casos. Como sequências vazias não são consideradas falsey
em Clojure (i.e., não são logicamente falsas), usar `rest` em condições como no exemplo abaixo não é uma boa ideia.
Nesses casos, a melhor alternativa é usar `next`:

```clojure
;; BAD!
(if (rest [10])
  "It has a tail!"
  "It does not have a tail!")
;;=> "It has a tail!"

;; GOOD!
(if (next [10])
  "It has a tail!"
  "It does not have a tail!")
;;=> "It does not have a tail!"
```

Nos exemplos acima, queremos verificar se o vector (no caso, `[10]`) possui uma "cauda", ou seja, mais elementos
além do primeiro. Usando `rest`, temos o resultado incorreto, pois `(rest [10])` retorna `[]`, que é logicamente verdadeiro.
Quando usamos `next`, temos o resultado correto, pois `(next [10])` retorna `nil`, que é logicamente falso.

Na prática, `(next v)` se comporta como `(seq (rest v))`.
Esse parece um exemplo bastante inventado, mas na verdade este é um padrão relativamente comum em funções recursivas.

O contrário da função `rest`/`next` é dado pela função `butlast`:

```clojure
(butlast v) ;;=> (10 20 30)
```

Essa função retorna toda collection, exceto o último elemento. Se a collection estiver vazia ou com apenas um elemento,
`butlast` retorna `nil`.

Também existem funções específicas para acessar o primeiro ou segundo elemento de um vector com
as funções `first` e `second`, respectivamente:

```clojure
(first v) ;;=> 10

(second v) ;;=> 20
```

Para acessar o último elemento, contamos com a função `last`:

```clojure
(last v) ;;=> 40
```

Todas essas três funções retornam `nil` caso não haja o elemento solicitado.

Por fim, falemos da função `contains?` em vectors. TL;DR: não use a função `contains?` com vectors.

Por que? Essa função verifica se um certo dado é uma _chave_ em uma collection associativa. Como vimos, vectors se comportam
como collections associativas de índices (chaves) para elementos (valores). No caso de vectors,
essa função verifica se o dado é um índice do vector, o que raramente é o que queremos.

Ok, então como verificar se um dado está no vector? Use a função `some`! A função `some` recebe um predicado
e uma collection. Ela retorna o primeiro valor logicamente positivo que encontrar ao aplicar o predicado
nos elementos da collection (em ordem) ou `nil`:

```clojure
(some #(= % 20) v) ;;=> true

(some #(= % 2000) v) ;;=> nil
```

Além dessas, todas as funções que já vimos que atuam sobre collections como `map`, `reduce` e `filter` funcionam
com vectors da forma esperada. Fique atento que elas geralmente não retornam um vector, mas sim uma sequence
(que veremos mais adiante).

Com isso, terminamos de ver os vectors e algumas das suas principais funções. Tenha em mente que muitas dessas
funções são mais gerais do que apenas para vectors. Elas podem ser aplicadas em basicamente qualquer coleção
que seja uma sequência. Veremos mais sobre isso adiante.

### Lists

Lists (listas) são estruturas de dados sequências, ordenadas, não homogêneas, assim como os vectors. Uma das grandes
diferenças para os vectors é a sua implementação e as consequências que isso traz.

Ao contrário dos vectors, onde as operações são mais performáticas na ponta direita (final), nas lists as operações
são mais performáticas na ponta esquerda (início). Isso porque lists são implementadas como linked lists (listas encadeadas).
Se você tiver feito um curso sobre estruturas de dados, talvez lembre que linked lists mantêm uma referência (geralmente
chamada _head_) para o início da lista.

Uma consequência disso, é que acessar elementos de forma indexada não é tão performático assim, pois a list deve ser
percorrida sequencialmente, a partir do seu primeiro elemento. Outra consequência disso é que adicionar ou remover do
final da list também exige que toda list seja percorrida. Por esses motivos, algumas funções, quando aplicadas a lists,
realizam suas operações sobre o primeiro elemento.

Novamente, vamos ver as principais funções que atuam sobre lists na ordem CRUD.

### Construindo lists

Lists podem ser criadas de forma literal com `'()`. Note o uso de uma aspa simples (também chamado quote). O quote
em Clojure tem a função de impedir que aquilo que o segue seja avaliado. Repare a diferença:

```clojure
(1 2 3)
;;! Execution error (ClassCastException) at nsclojure.core/eval1545 (form-init1572038123679197018.clj:1).
;;! class java.lang.Long cannot be cast to class clojure.lang.IFn (java.lang.Long is in module java.base of loader 'bootstrap'; clojure.lang.IFn is in unnamed module of loader 'app')

'(1 2 3)
;;=> (1 2 3)

(quote (1 2 3))
;;=> (1 2 3)
```

No primeiro exemplo, a expressão `(1 2 3)` foi avaliada como esperado e o `1` foi interpretado como uma função, por estar
na primeira posição da expressão, enquanto `2` e `3` são seus parâmetros. Obviamente, `1` não é uma função válida e portanto
vemos a exceção sendo lançada. No segundo exemplo, impedimos que a expressão `(1 2 3)` seja avaliada, adicionando o quote
na sua frente. O terceiro exemplo mostra que a aspa simples é apenas um syntatic sugar da função `quote`.
O uso do quote é necessário sempre que formos criar uma list dessa forma.

Outra forma de criar lists é com a função `list`:

```clojure
(list 1 2 3) ;;=> (1 2 3)
```

### Acessando lists

Como esperado, muitas das funções que atuam sobre collections (e vectors em especial) também atuam sobre lists. Por exemplo,
`nth`, `first`, `second`, `last`, `rest`, `next`, `butlast` etc todas atuam sobre lists com a mesma semântica.

### Modificando lists

Assim como com vectors, lists não são realmente modificadas, mas sim retornada uma nova list com as modificações feitas
sobre a list original. Muitas das funções que vimos para vectors também vão funcionar para lists. Uma delas merece
atenção.

A função `conj`, quando aplicada sobre uma list, adiciona o novo elemento no começo da list (ao contrário de quando
aplicada sobre um vector, que adiciona o novo elemento no final):

```clojure
;; List!
(conj '(10 20 30 40) :new-element) ;;=> (:new-element 10 20 30 40)

;; Vector!
(conj [10 20 30 40] :new-element) ;;=> [10 20 30 40 :new-element]
```