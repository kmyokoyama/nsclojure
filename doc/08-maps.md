# Sets e maps

## Sets

## Maps

Nesta seção, veremos uma das estruturas de dados mais utilizadas do Clojure, o map. O map é um mapa associativo
presente em diversas outras linguagens de programação sob vários nomes, dict, hash, table, associative arrays etc.

Um map é uma estrutura composta por pares de chave (key) e valor (value), onde as keys são únicas e imutáveis e os values
podem ser qualquer coisa, inclusive podem se repetir. Maps, por padrão, não são ordenados. Isso significa que não podemos
contar com nenhuma garantia de como os dados serão dispostos. Por exemplo, ao usarmos `first` em um map (sim, podemos
fazer isso), não há nenhuma garantia que será retornado o primeiro par chave/valor inserido no map.

Para apresentar o map, também vamos seguir a ordem CRUD.

### Construindo maps

Um map literal pode ser construído com `{,,,}`, onde chaves e valores são separados por um espaço em branco. Veja um exemplo:

```clojure
{:first-key 10 "second key" 20} ;;=> {:first-key 10, "second key" 20}
```

Duas coisas para notar mesmo neste exemplo simples. Primeiro, o map permite que chaves e valores sejam de tipos diferentes.
Por exemplo, a primeira chave é uma keyword, enquanto a segunda chave é uma string. Apesar de os valores no exemplo serem
ambos numéricos, eles também poderiam ser de tipos diferentes. Segundo, o uso de vírgulas separando pares diferentes
também é opcional (lembre-se, vírgulas são ignoradas), mas sua omissão é altamente recomendada.

Também podemos usar as funções `array-map`, `hash-map` e `sorted-map` para criar novos maps. Vamos começar por `array-map`:

```clojure
(array-map :first-key 10 :second-key 20) ;;=> {:first-key 10, :second-key 20}

(type (array-map :first-key 10 :second-key 20)) ;;=> clojure.lang.PersistentArrayMap
```

A função `array-map` recebe um número par de argumentos, sendo coletados dois a dois para formar os pares de chave/valor.
A estrutura subjacente retornada é um `clojure.lang.PersistentArrayMap`. Essa implementação é a default para pequenos maps
(atualmente, 8 pares chave/valor). Para maps maiores, precisamos de uma estrutura mais adequada, como a retornada por `hash-map`:

```clojure
(hash-map :first-key 10 :second-key 20) ;;=> {:second-key 20, :first-key 10}

(type (hash-map :first-key 10 :second-key 20)) ;;=> clojure.lang.PersistentHashMap
```

Primeiro, note como a ordem no map retornado é diferente da ordem passada a `hash-map`. Isso ressalta que maps, por default,
não são estruturas ordenadas. A forma como usamos `hash-map` é igual a `array-map`, passando os pares chave/valor como
argumentos dois a dois. A estrutura subjacente retornada é um `clojure.lang.PersistentHashMap`, muito mais adequada para
grandes maps. Essa é a estrutura default para maps maiores que 8 pares chave/valor.

Se for preciso criar um map que preserve a ordem dos seus pares de chave/valor, podemos utilizar uma das duas funções
`sorted-map` ou `sorted-map-by`. A `sorted-map` funciona exatamente como `array-map` ou `hash-map`, mas retorna um
`clojure.lang.PersistentTreeMap`:

```clojure
(sorted-map :b 1 :a 2) ;;=> {:a 2, :b 1}

(type (sorted-map :b 1 :a 2)) ;;=> clojure.lang.PersistentTreeMap
```

Note como a ordem no map criado é por comparação das chaves (`:a` < `:b`) e não dos valores ou ordem de inserção. Para
mais flexibilidade sobre a ordenação, podemos usar a função `sorted-map-by`. Essa função recebe um comparador e os
pares de chave/valor e retorna um map ordenado como um `clojure.lang.PersistentTreeMap`:

```clojure
(def people {100 {:name "John" :age 20}
             80  {:name "Mary" :age 22}
             200 {:name "Alice" :age 22}
             60  {:name "Bob" :age 18}})

(into (sorted-map-by
        (fn [key1 key2] (compare [(:age (get people key1)) (:name (get people key1))]
                                 [(:age (get people key2)) (:name (get people key2))])))
      people)
;;=> {60 {:name "Bob", :age 18}, 100 {:name "John", :age 20}, 200 {:name "Alice", :age 22}, 80 {:name "Mary", :age 22}}
```

No exemplo acima, `people` é um map, não ordenado, onde as chaves são id e os valores são maps representando pessoas
(com nome e idade), e queremos um map com a mesma estrutura, mas ordenado pela idade e, em caso de empate, pelo nome.
Para isso, utilizamos a função `sorted-map-by` com uma função que usa `compare` para comparar elementos dois a dois.
Nesse caso, comparamos vetores, onde o primeiro elemento é a idade e o segundo elemento é o nome. Comparar vetores assim
nos permite tratar dos casos de empate. Note como Mary e Alice possuem a mesma idade, mas pelo critério de desempate,
Alice veio na frente de Mary. Se desejar uma ordem decrescente com `compare`, simplesmente inverta `key1` e `key2`
de lugar.

### Acessando maps

Existem várias formas de acessar os elementos de maps. Para todos os exemplos a seguir, usaremos o seguinte map:

```clojure
(def jane {:name "Jane"
           :age  22
           :job  {:company {:name "Not an Evil Corp"
                            :sector "Software"}
                  :role "Software Engineer"}})
```

Uma das principais formas de acessar um map é aproveitando-se do fato de que maps respondem à interface de funções
(`clojure.lang.IFn`)

```clojure
(jane :name) ;;=> "Jane"
```

Melhor ainda, keywords também podem se comportar como funções:

```clojure
(:name jane) ;;=> "Jane"

(:hobbies jane) ;;=> nil

(:hobbies jane #{}) ;;=> #{}
```

No exemplo acima, usamos `:name` como função para acessar esta chave do map passado como argumento (`jane`). Se tentarmos
acessar uma chave que não existe (como `:hobbies`), o valor retornado será `nil`. Podemos também passar um segundo argumento
que é o valor default a ser retornado caso a chave não exista (o set vazio `#{}` é retornado, pois `:hobbie` não é uma chave existente).

Esta última forma (usando keywords como função) tem uma vantagem sobre a primeira: keywords literais nunca são `nil`,
portanto não corremos o risco de receber `NullPointerException`. Por outro lado, maps podem ser `nil` e esse risco é real:

```clojure
(def john nil)

(john :name)
;;! xecution error (NullPointerException) at nsclojure.core/eval1639 (form-init12030100162938325497.clj:1).
;;! Cannot invoke "clojure.lang.IFn.invoke(Object)" because the return value of "clojure.lang.Var.getRawRoot()" is null
```

Uma forma idiomática de acessar elementos de maps aninhados é com `->` e keywords:

```clojure
(-> jane :job :company :name) ;;=> "Not an Evil Corp

;; Equivalent to
(:name (:company (:job jane))) ;;=> "Not an Evil Corp"
```

Para acessar elementos de um map, também podemos utilizar a função `get` e sua versão para maps aninhados `get-in`:

```clojure
(get jane :name) ;;=> "Jane"

(get jane :hobbies) ;;=> nil

(get jane :hobbies #{}) ;;=> #{}

(get john :name) ;;=> nil
```

Como dá para perceber, o comportamento dessa função é bastante parecido com o obtido usando keywords. A versão aninhada,
`get-in` é análoga:

```clojure
(get-in jane [:job :company :name]) ;;=> "Not an Evil Corp"
```

Por fim, existem duas formas bastante úteis para acessar múltiploes elementos de um map. A primeira forma é com a função
`select-keys`, que retorna um "submap" do map original:

```clojure
(select-keys jane [:name :age]) ;;=> {:name "Jane", :age 22}
```

A segunda forma é um pouco mais complicada, mas ainda assim muito útil (e versátil). Ela utiliza a função `juxt`.
A função `juxt` aceita como argumentos várias funções e retorna uma única função (higher-order function das melhores!),
que retorna um vector da aplicação daquelas funções. Confuso, não? Vamos ver um exemplo:

```clojure
(defn square [x] (* x x))

(defn cube [x] (* x x x))

(def powers (juxt square cube))

(powers 5) ;;=> [25 125]
```

Agora, podemos combinar o poder de `juxt` com keywords atuando como funções:

```clojure
((juxt :name :age) jane) ;;=> ["Jane" 22]

((juxt :name (fn [p] (-> p :age inc))) jane) ;;=> ["Jane" 23]
```

O segundo exemplo acima ilustra como podemos usar `juxt` para acessar e manipular múltiplos campos do mesmo map. De fato,
`juxt` é bastante versátil e poderoso, e usá-lo com maps é apenas uma das possibilidades. Vale a pena saber aplicá-lo

### Modificando maps

Como toda boa estrutura persistente em Clojure, modificar um map não modifica realmente o map original. Em vez disso,
um novo map, baseado no map original com as modificações efetuadas, é retornado.

Duas das funções mais usuais para modificar maps são a `assoc` (e `assoc-in`). A função `assoc` permite associar um novo
valor a uma chave. Se a chave não existir, ela será criada e associada com aquele valor:

```clojure
(assoc jane :hobbies #{"reading" "running"})
;;=> {:name "Jane",
;;=>  :age 22,
;;=>  :job {:company {:name "Not an Evil Corp", :sector "Software"}, :role "Software Engineer"},
;;=>  :hobbies #{"running" "reading"}}

;; It is Jane's birthday!
(assoc jane :age 23)
;;=> {:name "Jane", :age 23, :job {:company {:name "Not an Evil Corp", :sector "Software"}, :role "Software Engineer"}}

(assoc-in jane [:job :company :name] "A Good Corp")
;;=> {:name "Jane", :age 22, :job {:company {:name "A Good Corp", :sector "Software"}, :role "Software Engineer"}}
```

> Vale reforçar: o map original `jane` não sofreu nenhuma alteração! Novos maps foram criados a partir das modificações
> acima. Se manter essas modificações for importante, utilize `let`, `def` etc. para dar um nome aos novos maps.

Às vezes, o novo valor é derivado do valor atual, utilizando alguma função. Por exemplo, no caso do aniversário da Jane,
queremos incrementar a sua idade em 1. Isso pode ser obtido com a função `update` (e sua versão aninhada `update-in`):

```clojure
(update jane :age inc)
;;=> {:name "Jane", :age 23, :job {:company {:name "Not an Evil Corp", :sector "Software"}, :role "Software Engineer"}}
```

Obviamente, este exemplo foi o simples caso de passar `inc`, mas a função `update` aceita uma função tão complexa quanto
se queira.

A sua versão aninhada, `update-in` é análoga:

```clojure
(update-in jane [:job :company] (fn [old-company]
                                    (if (= "Software" (:sector old-company))
                                      (assoc old-company :name "Maybe an Evil Corp")
                                      old-company)))
;;=> {:name "Jane", :age 22, :job {:company {:name "Maybe an Evil Corp", :sector "Software"}, :role "Software Engineer"}}
```

As funções `update`/`update-in` aceitam argumentos extras que são passados para a função de atualização. Veja um exemplo:

```clojure
(update jane :age - 1)
;;=> {:name "Jane", :age 21, :job {:company {:name "Not an Evil Corp", :sector "Software"}, :role "Software Engineer"}}

(update jane :age (fn [old-age] (- old-age 1)))
;;=> {:name "Jane", :age 21, :job {:company {:name "Not an Evil Corp", :sector "Software"}, :role "Software Engineer"}}
```

Note que o valor a ser atualizado (no exemplo, `old-age`) é passado como primeiro argumento da função de atualização,
enquanto os argumentos extras (no exemplo, `1`) são passados como segundo, terceiro, etc.

### Removendo elementos

Às vezes, é necessário remover uma chave (e seu respectivo valor) de um map. Para isso, contamos com a função `dissoc`:

```clojure
(dissoc jane :age)
;;=> {:name "Jane", :job {:company {:name "Not an Evil Corp", :sector "Software"}, :role "Software Engineer"}}
```

Infelizmente, não existe uma função `dissoc-in`, mas sua funcionalidade pode ser facilmente alcançada com `update`/`update-in` e `dissoc`:

```clojure
(update-in jane [:job :company] dissoc :sector)
;;=> {:name "Jane", :age 22, :job {:company {:name "Not an Evil Corp"}, :role "Software Engineer"}}
```

Note que o argumento extra, `:sector`, é passado como segundo argumento para a função `dissoc`. O exemplo acima
é equivalente ao seguinte:

```clojure
(update-in jane [:job :company] (fn [company] (dissoc company :sector)))
;;=> {:name "Jane", :age 22, :job {:company {:name "Not an Evil Corp"}, :role "Software Engineer"}}
```

### Outras funções úteis

Outras três funções que podem ser úteis são `keys`, `vals` e `zipmap`. Elas podem ser combinadas para criar outras funções
ainda mais interessantes.

A função `keys` simplesmente retorna uma sequência com as chaves de um map, enquanto a função `vals` retorna uma sequência
com os valores de um map:

```clojure
(keys jane)
;;=> (:name :age :job)

(vals jane)
;;=> ("Jane" 22 {:company {:name "Not an Evil Corp", :sector "Software"}, :role "Software Engineer"})
```

A função `zipmap` recebe duas collections e retorna uma map onde as chaves vêm da primeira collection e os valores
vêm da segunda collection:

```clojure
(zipmap [:name :age :hobbies] ["Jane" 22]) ;;=> {:name "Jane", :age 22}
```

Note no exemplo acima que a collection de valores é mais curta que a collection de chaves. Isso fez com que a chave
`:hobbies` ficasse sem valor correspondente e, por isso, não fosse incluída no map final.