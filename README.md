# nsclojure

O projeto `nsclojure` é uma tentativa de ampliar os conteúdos disponíveis
em português sobre Clojure.

É muito fácil encontrar bons materiais sobre a linguagem e bibliotecas em inglês,
mas é uma tarefa um pouco mais complicada encontrar em português. Apesar de já
existirem ótimos conteúdos no nosso idioma, essa é mais uma contribuição para
oferecer uma nova alternativa livre e gratuita.

Existe uma ressalva porém. Como é possível notar já no [Conteúdo](#conteúdo) abaixo, não se pode
dizer que o material é 100% em português. Eu decidi não traduzir termos consolidados
em inglês, como `map`, `vector`, e `multimethods`, e _todo código_. O motivo para isso
é simples - é muito mais fácil realizar buscas e ter sucesso em inglês do que em qualquer
outro idioma. Isso também facilta a comunicação com outros desenvolvedores,
a assimilação de novos conteúdos no futuro, e buscar ajuda em fóruns online. Não acredito
que isso se torne um problema no entendimento deste projeto.

## Objetivo

O objetivo deste material é ser um curso rápido sobre Clojure. Serão abordados
os principais tópicos da linguagem, funcionalidades e boas práticas, atendo-se
ao essencial para se desenvolver na linguagem.

Este projeto é colaborativo e provavelmente nunca estará terminado. Sinta-se
livre para propor melhorias, ideias e novas sugestões.

## Público-alvo

Este material destina-se a qualquer um que deseja aprender Clojure.

Pessoas que já desenvolvem certamente terão uma facilidade maior em seguir o conteúdo,
já que alguns passos como instalação do ambiente, configuração de IDE e outras tarefas
cotidianas para um dev não serão apresentadas.

O material também pode não trazer grandes novidades para desenvolvedores Clojure
experientes, mas eu acredito que sempre há algo novo para se aprender. Por si só,
este material também não é indicado como primeiro contato com programação.

## Convenções

As seguintes convenções são utilizadas:

1. `;;=>` indica o retorno da avaliação de uma expressão.
2. `;;#` indica uma saída no `stdout`.
3. `;;!` indica a ocorrência de uma exceção.

Exemplos:

```clojure
(println "Hi there!")
;;# Hi there!
;;=> nil

(println (/ 1 0))
;;! Execution error (ArithmeticException) at nsclojure.core/eval1657 (form-init9742472599147237557.clj:1).
;;! Divide by zero
```

## Conteúdo

01. [Apresentando Clojure](doc/01-apresentando-clojure.md)
02. [Tipos de dados](doc/02-tipos-de-dados.md)
03. [Controle de fluxo](doc/03-controle-de-fluxo.md)
04. [Estrutura do código](doc/04-estrutura-do-codigo.md)
05. [Funções](doc/05-funcoes.md)
06. [Estruturas de dados](doc/06-estruturas-de-dados.md)
07. [Vectors e lists](doc/07-vectors-e-lists.md)
08. Maps
09. Sequences
10. Records
11. Multimethods
12. Protocols
13. Namespaces
14. Estado e concorrência
15. Java interop
16. core.async
17. Macros
18. Pensando funcional

## License

MIT License