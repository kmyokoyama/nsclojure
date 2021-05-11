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

Vamos conhecer as principais funções relativas a vectors na ordem CRUD (Create-Read-Update-Delete).

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

Temos basicamente três formas de acessar elementos em um vector. Para exemplificá-las, usaremos
os seguintes dados:

```clojure
(def v [10 20 30])

(def v-nil nil)

(def v-nested [[10 20] [30 40]])
```

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

### Modificando vectors

Se você leu a seção anterior, provavelmente está se pergunta "Mas não é tudo imutável?".
E a resposta é "Sim!". Tudo continua imutável. Na verdade, não estaremos modificando
a estrutura original, mas sim criando uma nova estrutura com as modificação a partir da original.

Temos algumas formas de modificar elementos em vectors. A primeira delas é com `assoc`:

```clojure
(assoc v 1 2000) ;;=> [10 2000 30 40]

v ;;=> [10 20 30 40]
```

Note no exemplo acima como o retorno do `assoc` é um novo vetor com as modificações desejadas. No entanto, o vector
original, no caso `v`, não foi alterado. Isso também vale para qualquer uma dos casos a seguir.

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

### Removendo elementos

Remover elementos de um vector não é uma tarefa tão usual. Talvez por isso, Clojure não tenha uma forma mais direta de realizar
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