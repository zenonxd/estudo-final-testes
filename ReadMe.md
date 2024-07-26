# Estudo Testes Subcutâneos

## Referência: Testes automatizados na prática com Spring Boot
[Compre aqui](https://www.udemy.com/course/testes-automatizados-na-pratica-com-spring-boot/)
<hr>

## Dica para leitura:
Durante o estudo, os códigos mudam pois geralmente é ensinado algo básico onde depois iremos implementar o que de fato
é utilizado no mercado de trabalho. Tome cuidado ao considerar códigos do início do estudo, se atente ao código final.

## Tópicos

<!-- TOC -->
* [Estudo Testes Subcutâneos](#estudo-testes-subcutâneos)
  * [Referência: Testes automatizados na prática com Spring Boot](#referência-testes-automatizados-na-prática-com-spring-boot)
  * [Dica para leitura:](#dica-para-leitura)
  * [Tópicos](#tópicos)
* [Introdução](#introdução)
  * [Separando as fases de teste](#separando-as-fases-de-teste)
  * [Testes de cobertura com o Jacobo](#testes-de-cobertura-com-o-jacobo)
  * [Teste Mutantes com Pitest](#teste-mutantes-com-pitest)
  * [Testes de integração com Testcontainers](#testes-de-integração-com-testcontainers)
  * [Testes Parametrizados](#testes-parametrizados)
* [Resumo](#resumo)
  * [Fim](#fim)
<!-- TOC -->
<hr>


# Introdução

Agora, faremos alguns toques finais no projeto, pensando no ambiente de produção. Alguns detalhes a mais, para termos
certeza de que testamos tudo o que era possível, evitando erros.

<hr>

## Separando as fases de teste

Separaremos os testes mais pesados em uma fase específica de testes e os testes mais leves em outra fase.

A ideia é pensar no build do projeto otimizando ele, para que seja rápida a entrega contínua do produto nos ambientes.
E só executar em alguns casos específicos, as tarefas mais pesadas.

Para isso, iremos separar o build em fases, utilizando o Maven, a execução dos testes terão duas fases distantes: 
**leves e pesados**.

- Testes mais leves: criaremos uma fase de testes de unidade.

- Testes mais pesados: criaremos uma fase de testes de integração.

Isso será útil, pois, quando formos configurar a esteira de entrega contínua podemos colocar o teste de integração
somente no ambiente de desenvolvimento e os unitários em ambiente de produção, por exemplo.

### Para fazermos isso com o Maven, utilizaremos plugins

Por padrão, os testes que possuem o sufixo "Test", rodam na mesma fase. Portanto, teremos um plugin para testes de 
unidade e outro para de integração.


- #### Testes de Unidade (mais leves) - Plugin SureFire
```

```

#### Testes de Integração (mais pesados) - Plugin failsafe
```
```

<hr>

## Testes de cobertura com o Jacobo

<hr>

## Teste Mutantes com Pitest

<hr>

## Testes de integração com Testcontainers

<hr>

## Testes Parametrizados

<hr>

# Resumo


<hr>

## Fim

