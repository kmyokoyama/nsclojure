# nsclojure

O projeto `nsclojure` é uma tentativa de ampliar os conteúdos disponíveis
em português sobre Clojure.

É muito fácil encontrar bons materiais sobre a linguagem e bibliotecas em inglês,
mas é uma tarefa um pouco mais complicada encontrar em português. Apesar de já
existirem ótimos conteúdos no nosso idioma, essa é mais uma contribuição para
oferecer uma nova alternativa.

Existe uma ressalva porém. Como é possível notar já no [Conteúdo](#conteúdo) abaixo, não se pode
dizer que o material é 100% em português. Eu decidi não traduzir termos consolidados
em inglês, como `map`, `vector`, e `multimethods`, e _todo código_. O motivo para isso
é simples - é muito mais fácil realizar buscas e ter sucesso em inglês do que em qualquer
outro idioma. Isso também facilta a comunicação com outros desenvolvedores,
a assimilação de novos conteúdos no futuro, e buscar ajuda em fóruns online. Não acredito
que isso se torne um problema no entendimento deste projeto.

## Convenções

As seguintes convenções são utilizadas:

1. `;;=>` indica o retorno da avaliação de uma expressão.
2. `;;#` indica uma saída no `stdout`.
3. `;;!` indica a ocorrência de uma exceção.

Exemplos:

```clojure
(println "Hi there!")
;;# "Hi there!"
;;=> nil

(println (/ 1 0))
;;! Execution error (ArithmeticException) at nsclojure.core/eval1657 (form-init9742472599147237557.clj:1).
;;! Divide by zero
```

## Conteúdo

1. Introdução
2. Apresentando Clojure
3. [Tipos de dados](doc/tipos-de-dados.md)
4. Controle de fluxo
5. Estrutura do código
6. Funções
7. Estrutudas de dados
8. Vectors e lists
9. Maps
10. Sequences
11. Records
12. Multimethods
13. Protocols
14. Namespaces
15. Estado e concorrência
16. Java interop
17. core.async
18. Macros
19. Pensando funcional

## License

MIT License