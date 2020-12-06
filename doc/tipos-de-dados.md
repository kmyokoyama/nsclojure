# Tipos de dados

Clojure é uma linguagem dinamicamente tipada, o que significa que os tipos
não são declarados explicitamente, mas sim que os tipos são tratados implicitamente
em runtime. Por um lado, isso facilita a escrita do código, já que não há a necessidade de se
preocupar com declaração de tipos, criando um clima de certa produtividade.
Por outro lado, perde-se a segurança que tipos explícitos provêm, o que confere
certa confiança no código ainda durante a compilação.

Neste capítulo, iremos falar dos tipos de dados escalares como os tipos numéricos
e strings. Em outro capítulo falaremos dos tipos compostos como maps e vectors.

É possível verificar o tipo de um dado com a função `type`. Por exemplo,

```clojure
(type 1) ;;=> java.long.Long
(type "this is a string") ;;=> java.lang.String
```

Faremos alguns uso do `*1`, que guarda o resultado da última expressão avaliada
no REPL:

```clojure
(+ 1 1) ;;=> 2
(inc *1) ;;=> 3 
```

## Tipos numéricos

Um dos tipos de dados mais básicos de qualquer linguagem são os numéricos. Como
Clojure roda em cima da JVM, muito do que o Clojure oferece vem do Java e da JVM,
mas ainda melhor. Isso é especialmente verdade para os tipos numéricos.

A primeira coisa a se ter em mente é que o Clojure faz seu melhor para tratar
todos dados numéricos em 64-bit. Mesmo que a JVM suporte representações menores
(e.g., 16 ou 32-bit), o Clojure expandirá a saída para, pelo menos, 64-bit.

No exemplo a seguir, usamos o atributo estático `BYTES` para verificar quantos bytes um
tipo numérico utiliza na JVM. Então somamos dois `Short` de 16 bits
cada e produzimos um `Long` de 64 bits:

```clojure
(type (short 1000)) ;;=> java.lang.Short
Short/BYTES ;;=> 2
(type (+ (short 1000) (short 2000))) ;;=> java.lang.Long
Long/BYTES ;;=> 8
```

Os principais tipos numéricos do Clojure são resumidos na tabela abaixo:

| Tipo  | Boxed?  | Ilimitado?  | Descrição |
|:-----:|:-------:|:----------:|:---------:|
| `long`  | ✗ (primitivo)  | ✗ |  Inteiro primitivo de 64 bits |
| `double`  | ✗ (primitivo)  | ✗ |  Ponto-flutuante primitivo de 64 bits |
| `java.lang.Long`  | ✓  | ✗ |  Inteiro |
| `java.lang.Double`  | ✓  | ✗ |  Ponto-flutuante |
| `java.math.BigInteger`  | ✓  | ✓ |  Inteiro de tamanho arbitrário |
| `clojure.lang.BigInt`  | ✓  | ✓ |  Inteiro de tamanho arbitrário próprio do Clojure |
| `java.math.BigDecimal`  | ✓  | ✓ |  Ponto-flutuante de precisão arbitrária |
| `clojure.lang.Ratio`  | ✓  | - |  Racional |

O motivo de existir do `clojure.lang.BigInt` é que a implementação de inteiro de
tamanho arbitrário do Java, `java.math.BigInteger`, é um pouco problemática, apresentando
inconsistências com a implementação de `java.lang.Long`,
e `clojure.lang.BigInt` foi especialmente otimizado para ser tratado como primitivo sempre
que possível, o que melhora muito o desempenho. Portanto, esse é o tipo padrão do Clojure
para representar inteiros de tamanho arbitrário.

Para criar um `BigInt`, podemos utilizar a função `bigint` ou acrescentar um `N` ao final do número literal:

```clojure
(type (bigint 1)) ;;=> clojure.lang.BigInt
(type 1N) ;;=> clojure.lang.BigInt
(type 7463847623874362398172398127) ;;=> clojure.lang.BigInt
```

De forma semelhante, podemos utilizar a função `bigdec` ou acrescentar um `M` ao literal de ponto-flutuante para
criar um `java.math.BigDecimal`. No entanto, um literal de grande precisão não é
representado automaticamente como um `BigDecimal`, e sim como um `Double`:

```clojure
(type (bigdec 1)) ;;=> java.math.BigDecimal
(type 1.0M) ;;=> java.math.BigDecimal
(type 0.3718637162873612873612783) ;;=> java.lang.Double
```

O tipo racional, `clojure.lang.Ratio`, permite representar racionais (números que podem ser representados
como fração) de forma literal. Enquanto operações com literais de ponto-flutuante
podem resultar em erros de arredondamento, operações com racionais literais resultam em racionais sem perda
de precisão, ou em inteiros do tipo `BigInt`, quando possível:

```clojure
(type 1/10) ;;=> clojure.lang.Ratio

(+ 0.1 0.1 0.1) ;;=> 0.30000000000000004
(+ 1/10 1/10 1/10) ;;=> 3/10

(+ 1/2 1/2) ;;=> 1N
(type *1) ;;=> clojure.lang.BigInt
```

### Operações com tipos numéricos

O Clojure permite que tipos numéricos diferentes sejam misturados em operações
matemáticas. O resultado será do tipo numérico mais abrangente _capaz de representar
corretamente o resultado_. Isso significa que alguns tipos prevalevem sobre outros
na seguinte ordem:

`long` ⇨ `BigInt` ⇨ `Rational` ⇨ `BigDecimal` ⇨ `double`

Veja alguns exemplos:

```clojure
(+ 1 1N) ;;=> 2N
(+ 10 10.5M) ;;=> 20.5M
(+ 1 10.5M) ;;=> 11.5M
(+ 1N 10.5M) ;;=> 11.5M

(+ (double 0.1) 1.0M) ;;=> 1.1
(type *1) ;;=> java.lang.Double
```

No último exemplo, a soma de um `Double` (precisão limitada) com um
`BigDecimal` resultou em um `Double`. O motivo para esse resultado contra-intuitivo
é que não faz sentido uma operação envolvendo um número sabidamente
não preciso (o `Double`) resultar em um número considerado preciso (`BigDecimal`).
Em outras palavras, operações com números cuja precisão é questionável
devem resultar em números cuja precisão também é questionável. Além disso, o 
`Double` pode representar alguns valores, como `Infinity` e `NaN` previstos pela
IEEE-754 (base da sua implementação), que o `BigDecimal` não saberia representar.

```clojure
(+ Double/NaN 1M) ;;=> ##NaN
(type *1) ;;=> java.lang.Double
```

### Comparações de tipos numéricos

Em algumas linguagens, comparação pode ser um problema sério (estou olhando para você JavaScript).
Clojure provê as ferramentas que você precisa de forma consistente, sem surpresas.

A primeira forma de comparação é com a função `identical?`, e vale para qualquer tipo
de dados, não é especĩfica de tipos numéricos. Essa função compara as referências dos
objetos em si, e não é o ideal para a maioria dos casos (a não ser que você queira...
comparar referências de objetos).

```clojure
(identical? 1 1) ;;=> false

(def x 1)
(identical? x x) ;;=> true
```

No primeiro exemplo, a comparação com `identical?` retorna `false` mesmo com valores literais.

A segunda forma de comparação é com a função `=`, também não específica de apenas tipos
numéricos. Essa função compara corretamente valores do mesmo "tipo matemático", i.e., inteiros
com inteiros, ponto-flutuante com ponto-flutuante e assim por diante:

```clojure
(= 1 1N (bigint 1) (int 1) (short 1)) ;;=> true
(= 1.0 1M (bigdec 1) (double 1)) ;;=> true
```

Mas falha ao comparar "tipos matemáticos" diferentes:

```clojure
(= 1 1.0) ;;=> false
(= 1N 1M) ;;=> false
(= 1N "one") ;;=> false
```

No último exemplo, a comparação absurda entre um número e uma string retorna `false`.
Em outras palavras, é seguro utilizar `=` entre tipos de dados totalmente diferentes,
caso onde o resultado será `false`.

Para comparações numéricos puras, independente do tipo concreto do dado, existe a
terceira forma de comparação, a função `==`. Essa função é especĩfica de tipos numéricos
e realiza a equivalência numérica:

```clojure
(== 1 1.0) ;;=> true
(== 1N 1M) ;;=> true
```

A função `==` pode lançar uma exceção se qualquer um dos seus argumentos não for
um número. Para resguardar desses casos, pode-se utilizar a função predicado `number?`,
que retorna `true` se seu argumento for um número, e `false` caso contrário.

```clojure
(== 1N "one") ;;=> Execution error (ClassCastException) at nsclojure.core/eval1703 (form-init9302101881305719260.clj:1).
             ;;   java.lang.String cannot be cast to java.lang.Number

(number? 1N) ;;=> true
(number? "one") ;;=> false
```

## Strings

Strings em Clojure são basicamente strings do Java, `java.lang.String`. Elas aceitam
qualquer caractere representável em Unicode, e são sempre entre aspas duplas (`""`)

```clojure
"Love is 愛: ✓" ;;=> "Love is 愛: ✓"
(type *1) ;;=> java.lang.String
```

Sem muito mistério.

Tudo que vale para `String` do Java também vale para string em Clojure. Para mais informações,
consulte a [documentação do Java](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html). 

> Assim como valores numéricos, strings são valores por si só, dispensando o uso
> de parênteses (o que até causaria o lançamento de uma exceção!).

Caracteres em Clojure são `java.lang.Character`, e são representados com uma contrabarra, \\.
Caracteres também podem ser representados através dos seus códigos Unicode, utilizando `\u`:

```clojure
\a ;;=> \a
\u611B ;;=> \愛
(type *1) ;;=> java.lang.Character
```

## Keywords

Keyword é um tipo de dado extremamente útil em Clojure, mas pouco encontrado em outras
linguagens (em Ruby são chamados `symbol`, em Elixir são `atom`). Valores desse tipo
são constantes, basicamente seu valor é exatamente o mesmo que seu próprio nome. Keywords
são sempre precedidas de `:`. Por exemplo,

```clojure
:some-keyword ;;=> :some-keyword
(type *1) ;;=> clojure.lang.Keyword
```

Podemos transformar uma keyword em uma string com `name` e uma string em uma keyword
com `keyword`:

```clojure
(name :some-keyword) ;;=> "some-keyword"
(keyword "some-keyword") ;;=> :some-keyword
```

Experimente um pouco com essas funções para ver seus comportamentos quando os argumentos
fogem do esperado.

Apesar de namespaces ainda não ter sido apresentado, vale adiantar algumas informações sobre
keywords. Se preferir, você pode pular essa seção agora e voltar a ela quando tiver mais
confortável com o conceito de namespaces em Clojure.

Como discutido no excelente artigo [The five common forms of Clojure keywords](https://blog.jeaye.com/2017/10/31/clojure-keywords/),
existem basicamente cinco formas de representar keywords, que serão resumidas a seguir:

1. `:some-keyword`

A forma mais simples (e comum) de uma keyword. Keywords também aceitam ponto, `.`:

```clojure
:dotted.keyword ;;=> :dotted.keyword
```

2. `::some-keyword`

Essa forma com dois `::` carrega a informação do namespace onde foi criada, por isso é chamada
de namespaced keyword. No exemplo a seguir serão usadas a referência `*ns*` para o namespace atual com
a função `ns-name` para exibir seu nome, e a função `in-ns` para criar um novo namespace e mudar para ele:

```clojure
(ns-name *ns*) ;;=> nsclojure.core
(def game ::chess)
game ;;=> nsclojure.core/chess

(in-ns 'new-namespace)
(clojure.core/refer-clojure) ;; Para ter acesso novamente ao ns-name.
(ns-name *ns*) ;;=> new-namespace
nsclojure.core/game ;;=> :nsclojure.core/chess
```

Essa forma é útil para evitar colisões de keywords definidas em diferentes namespaces. É uma
boa prática utilizar essa forma em seus projetos.

3. `some-namespace/some-keyword`

Essa forma estabelece um namespace (que deve ser existente e válido) para uma keyword.
Continuando o exemplo anterior, é possível acessar a mesma keyword criada, explicitando
o namespace com essa forma:

```clojure
(identical? nsclojure.core/game :nsclojure.core/chess) ;;=> true
```

Como keywords são constantes, a comparação das mesma referência com `identical?` retorna `true`.

Conhecendo essa forma, percebe-se que a forma anterior `::some-keyword` é apenas uma
syntatic sugar para `:current-namespace/some-keyword`.

4. `::namespace-alias/some-keyword`

Essa forma utiliza o alias dado a um namespace durante o `require`. Continuando dos
exemplos anteriores:

```clojure
(require '[nsclojure.core :as core])

(identical? nsclojure.core/game ::core/chess) ;;=> true
```

5. `random-prefix/some-keyword`

Essa última forma utiliza qualquer coisa como prefixo, antes da `/`. Essa é uma forma
de estabelecer um domínio para uma keyword, que não seja um namespace. Isso é às vezes
utilizado por bibliotecas para agrupar keywords relacionadas e evitar colisões com
outras keywords.

Ainda no exemplo anterior:

```clojure
:board-games/chess ;;=> :board-games/chess
:board-games/backgammon ;;=> board-games/backgammon
```

## Booleans

O Clojure compartilha com o Java o mesmo tipo boolean, `java.lang.Boolean`, com
obviamente dois valores possíveis, `true` ou `false`.

> Clojure é case-sensitive, portanto `True` ou `TRUE` não são booleans válidos.

Ao contrário de outras linguagens como C ou Python, onde alguns valores com conotação
de vazio (zero, listas vazias etc) são considerados logicamente como falsos, em
Clojure __apenas `false` e `nil` são logicamente falsos__. Qualquer outro valor é
considerado logicamente verdadeiro. Isso facilita muito na hora de pensar sobre seus
programas, evitando comportamentos inesperados.

Podemos usar a função `boolean` para verificar o valor lógico de uma expressão:

```clojure
(boolean true) ;;=> true
(boolean false) ;;=> false
(boolean nil) ;;=> false
(boolean 0) ;;=> true
(boolean []) ;;=> true
(boolean {}) ;;=> true
(boolean :false) ;;=> true
```

Os valores `true` e `false` do Clojure correspondem aos atributos estáticos
`Boolean/TRUE` e `Boolean/FALSE` do Java, respectivamente:

```clojure
(identical? true Boolean/TRUE) ;;=> true
(identical? false Boolean/FALSE) ;;=> true
```

> Nos exemplos a seguir, usamos `if` e `if-not`. Não se preocupe, veremos mais adiante
> como eles funcionam.

E alguém pode ficar tentado a criar suas próprias instâncias de `Boolean` (veremos
mais a fundo em [Java Interop](broken-link) como fazer isso), mas isso é um erro:

```clojure
(boolean (Boolean. "false")) ;;=> false
(if (Boolean. "false") "It is actually truthy" "Is it falsey?") ;;=> "It is actually truthy"
```

Para evitar esse tipo de comportamento bizarro,
__sempre utilize os valores `true` e `false` e nunca crie suas próprias instâncias de `Boolean`__.

Se precisar utilizar uma coleção vazia como logicamente falso, pode-se usar a função
`seq` que retorna `nil` quando a coleção está vazia, ou uma sequência quando a coleção não está vazia:

```clojure
(seq [1 2]) ;;=> (1 2)
(seq []) ;;=> nil
(if (seq []) "It is not empty" "It is empty") ;;=> "It is empty"
```

Isso é mais idiomático do que usar a função predicado `empty?`, que retorna `true`
se a coleção está vazia e `false` caso contrário:

```clojure
(if-not (empty? []) "It is not empty" "It is empty") ;;=> "It is empty"
```

## nil

O tipo `nil` possui apenas um valor, `nil`, e representa a ausência de valor. Como
já foi falado, `nil` é considerado logicamente falso.

```clojure
(type nil) ;;=> nil
(boolean nil) ;;=> false

(if nil "nil is truthy" "nil is falsey") ;;=> "nil is falsey"
```

Para verificar se um valor é `nil`, usa-se a função predicado `nil?`:

```clojure
(nil? nil) ;;=> true
(nil? false) ;;=> false
(nil? []) ;;=> false
(nil? (seq [])) ;;=> true
```