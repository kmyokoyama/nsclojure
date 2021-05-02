# Estruturas de dados

Nos capítulos anteriores já vimos algumas estruturas de dados como strings.
Agora começaremos a explorar as estruturas de dados compostas que são a base do Clojure. Antes, dois pilares devem
ser introduzidos.

## Imutabilidade

Uma das características mais interessantes das estruturas de dados em Clojure é a imutabilidade.
Ser imutável significa que a estrutura de dados permanece _a mesma_, com os seus conteúdos originais,
desde o momento de a sua criação até deixar de ser usada e então recolhida pelo runtime.
Quando funções de modificação (como aquelas que trocam um valor por outro em determinado índice
de um vetor) são usadas numa estrutura imutável, o que acontece é que uma nova estrutura de dados
com as modificações desejadas é criada e retornada. A estrutura original permanece inalterada, e qualquer
referência à estrutura original continua a enxergar a versão sem modificações.

Isso imediatamente deve gerar preocupações em algumas pessoas a respeito de performance. Mas não com o que
se preocupar. Clojure utiliza estruturas de dados persistentes bastante sofisticadas para alcançar imutabilidade
sem degradar a performance.

Persistência neste contexto não significa armazenamento em disco, mas sim que estruturas de dados persistentes
preservam as suas versões anteriores em memória, compartilhando partes da estrutura com as demais versões e evitando
duplicação desnecessária. [ref: Okasaki 1999]

Imutabilidade traz inúmeros benefícios como tornar muito mais fácil o entendimento do código (referências não são alteradas e
funções puras são mais fáceis de escrever) e facilitar programação concorrente (estruturas imutáveis são sempre _thread-safe_).
Depois de um tempo programando com imutabilidade, você vai se perguntar como foi possível programar sem imutabilidade um dia.

## Poucas estruturas, muitas funções

Clojure segue a filosofia muito bem resumida por Alan Perlis e já apresentada na [Introdução](01-apresentando-clojure.md#apresentando-clojure):

> "_It is better to have 100 functions operate on one data structure than 10 functions on 10 data structures._"  
> — Alan Perlis
>
> (_É melhor ter 100 funções que operam sobre uma estrutura de dados do que 10 funções que operam sobre 10 estruturas de dados._)

Isso significa que em vez de termos inúmeras estruturas de dados com alguns poucos métodos e funções especializados (como
propagado pelo paradigma OO), em Clojure temos algumas poucas estruturas de dados com muitas funções operando sobre eles.

Isso permite nos focarmos nas operações que desejamos realizar (e no algoritmo em si) em vez de nos preocuparmos
com a estrutura de dados em si e como seu conteúdo é organizado e mantido. Além disso, poucas, mas versáteis,
opções de estruturas diminuem a carga na hora de decidir qual usar. Em termos de funções, fica muito mais simples
conhecer e entender como as diversas funções atuam sobre a mesma estrutura e também criar novas funções, mais específicas, 
para essa mesma estrutura conhecida.

Com esses dois pilares explicados, podemos avançar e conhecer as estruturas de dados imutáveis do Clojure e as suas
muitas funções.
