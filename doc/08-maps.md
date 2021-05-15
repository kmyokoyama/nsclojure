# Maps

Nesta seção, veremos uma das estruturas de dados mais utilizadas do Clojure, o map. O map é um mapa associativo
presente em diversas outras linguagens de programação sob vários nomes, dict, hash, table, associative arrays etc.

Um map é uma estrutura composta por pares de chave (key) e valor (value), onde as keys são únicas e imutáveis e os values
podem ser qualquer coisa, inclusive podem se repetir. Maps, por padrão, não são ordenados. Isso significa que não podemos
contar com nenhuma garantia de como os dados serão dispostos. Por exemplo, ao usarmos `first` em um map (sim, podemos
fazer isso), não há nenhuma garantia que será retornado o primeiro par chave/valor inserido no map.

Para apresentar o map, também vamos seguir a ordem CRUD.

## Construindo maps

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
mais flexibilidade sobre a ordenação, podemos usar a função `sorted-map-by`:

```clojure

```