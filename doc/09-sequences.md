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
```