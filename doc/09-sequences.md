# Sequences

Nesta seção, falaremos sobre uma das abstrações mais poderosas do Clojure, a _sequence_.

Primeiro, vamos melhorar nossa definição sobre alguns nomes e conceitos. Em Clojure, podemos chamar de collection
qualquer estrutura de dados composta, como vectors, lists, sets e maps. Algumas dessas estruturas são sequenciais por
natureza, ou seja, os seus elementos são armazenados uns após os outros, em ordem. Vectors e lists são exemplos de
estruturas sequenciais.

Além disso, Clojure provê uma abstração bastante genérica e útil sobre diversas coleções chamada _sequence_. Essa
abstração muitas vezes é abreviadas para _seq_, mas evitarei fazer isso aqui para não confundir com a função de mesmo nome.
Uma sequence é tudo aquilo que responde à API composta pelas funções `first`, `rest` e `cons`.

A função `first` retorna o primeiro elemento de uma collection ou `nil` se a collection estiver vazia. A função
`rest` retorna todos elementos após o primeiro (a cauda da collection) ou uma sequence vazia. Por fim, a função
`cons` adiciona um novo elemento no começo da collection e retorna uma sequence resultante. Importante notar que
sequences, como quase tudo em Clojure, são imutáveis. Isso significa que o resultado de `cons` é uma nova sequence sempre.
Vamos ver alguns exemplos:

```clojure
(def v [1 2 3 4]) ;; A collection.

(first v) ;;=> 1

(rest v) ;;=> (2 3 4)

(cons 0 v) ;;=> (0 1 2 3 4)
```

Note o que acontece quando a collection é vazia:

```clojure
(first nil) ;;=> nil

(rest nil) ;;=> ()

(cons 1 nil) ;;=> (1)
```

Nos exemplos acima, conseguimos usar as funções da API de sequence sobre um vector, pois vectors obedecem essa API.
Porém, note como o resultado de aplicar `rest` ou `cons` sobre um vector não é um vector, e sim uma sequence.

> **Sequences são exibidas da mesma forma que lists**
> 
> Olhando as saídas das chamadas a `rest` e `cons` acima, podemos ser levados a acreditar que os resultados são lists.
> Isso não seria correto. Na verdade, os resultados são sequences. Acontece que sequences são exibidos da mesma forma
> como lists são (com parênteses). Para tirar essa dúvida, podemos utilizar as funções `list?` e `seq?` que retornam
> se o argumento é uma list (como tipo concreto) e se o argumento conforma com a API de sequence, respectivamente:
>
> ```clojure
> (list? (rest v)) ;;=> false
> 
> (seq? (rest v)) ;;=> true
> ```

Dizemos que uma collection é _seq-able_ (sequenciável?) se podemos "extrair" uma sequence dela. Clojure provê a função
`seq` para isso:

```clojure
(seq v) ;;=> (1 2 3 4)

(type (seq v)) ;;=> clojure.lang.PersistentVector$ChunkedSeq
```

O sufixo `Seq` no tipo acima indica que estamos agora com uma representação de sequence da collection original.

Diversas estruturas de dados em Clojure são sequenciáveis, até mesmo aquelas estruturas de dados que não são sequenciais
(como os sets e maps):

```clojure
(seq #{1 2 3 4}) ;;=> (1 4 3 2)

(seq {:name "John" :age 22}) ;;=> ([:name "John"] [:age 22])

(-> (seq {:name "John" :age 22}) first) ;;=> [:name "John"]

(-> (seq {:name "John" :age 22}) first type) ;;=> clojure.lang.MapEntry
```

No segundo exemplo acima, vemos que podemos obter uma sequence a partir de um map. O último exemplo mostra que
cada elemento dessa sequence é um `clojure.lang.MapEntry`, que é basicamente um vector de dois elementos: o primeiro
elemento é a chave e o segundo elemento seu respectivo valor. O Clojure possui as funções `key` e `val` para acessar
o primeiro e segundo elemento de um `MapEntry`, respectivamente:

```clojure
(-> (seq {:name "John" :age 22}) first key) ;;=> :name

(-> (seq {:name "John" :age 22}) first val) ;;=> "John"
```

## Lazy sequences

Uma coisa bastante interessante sobre sequences é a possibilidade de trabalhar com lazy sequences (também vou manter
sem tradução). Basicamente, os elementos de uma lazy sequence não são avaliados até que seja estritamente necessário.
Isso significa que a lazy sequence está mais para uma receita de como gerar os seus elementos do que para uma coleção
completa em memória. À medida que os seus elementos vão se tornando necessários, eles vão sendo computados e retornados.

Imagine-se trabalhando com um conjunto de dados realmente grande em disco, impraticável de manter em memória.
Neste caso, poderia-se utilizar uma lazy sequence para trabalhar com elementos um a um ou em batches menores, que
caibam em memória (obviamente, se todo conjunto se fizesse necessário ao mesmo tempo em memória para determinado cálculo, lazy sequences
seriam de pouca ajuda).

Outro caso interessante de lazy sequences é trabalhar com sequences infinitas. Sequences com um número infinito de elementos
claramente não cabem em memória, mas se seus elementos são gerados e retornados um a um, então se trabalhar com sequences
infinitas se torna totalmente praticável.

Clojure possui diversas funções para se criar e manipular (lazy) sequences. Por exemplo, a função `repeat` gera uma lazy
sequence, possivelmente infinita, de um mesmo valor e a função `take` retorna o `n` primeiros elementos da sequence:

```clojure
(repeat 5 "na") ;;=> ("na" "na" "na" "na" "na")

(take 10 (repeat "na")) ;;=> ("na" "na" "na" "na" "na" "na" "na" "na" "na" "na")
```

> **Cuidado ao avaliar expressões que envolvem sequences infinitas no REPL**
> 
> Lembre-se que para mostrar na saída do REPL, é preciso computar os elementos da sequence. Isso significa que
> tentar avaliar expressões que retornam sequences infinitas, como `(repeat "na")`, no REPL pode causar um hang,
> que levará a ter que reiniciar o REPL.

Em outra situação, podemos precisar dos 10 primeiros inteiros positivos que são múltiplos de 3 e terminam em 7 (tarefa
bem cotidiana, certo?). Em Clojure, podemos simplesmente usar as funções `iterate` (com `inc`) e `filter` e `take`:

```clojure
(defn mult-3-ends-with-7?
      [n]
      (and (zero? (mod n 3))
           (-> n str (clojure.string/ends-with? "7"))))

(->> (iterate inc 1)
     (filter mult-3-ends-with-7?)
     (take 10))
;;=> (27 57 87 117 147 177 207 237 267 297)

;; Or...
(->> (iterate #(+ % 3) 27)
     (filter mult-3-ends-with-7?)
     (take 10))
;;=> (27 57 87 117 147 177 207 237 267 297)

;; Or......
(take 10 (iterate #(+ % 30) 27))
;;=> (27 57 87 117 147 177 207 237 267 297)
```

A função `iterate` recebe uma função, `f`, e um valor inicial, `x`, e retorna a sequence resultante de aplicações sucessivas
de `f` (i.e., `(x (-> x f) (-> x f f) (-> x f f f) ...)`. Note como valor inicial, `x`, é sempre o primeiro elemento da sequence).
Veja mais um exemplo, onde queremos obter as próximas 5 potências de 2, começando por 1024:

```clojure
(take 5 (iterate #(* % 2) 1024)) ;;=> (1024 2048 4096 8192 16384)
```

Clojure provê muitas outras funções para se trabalhar com (lazy) sequences.
O capítulo _Unifying Data with Sequences_ do livro Programming Clojure [Miller2018] é uma excelente fonte
sobre (lazy) sequences e suas aplicações em diversos cenários.

## Referências

* [Miller2018] Miller, Alex; Halloway, Stuart; Bedra, Aaron. Programming Clojure. 3a edição. The Pragmatic Programmers, 2018.