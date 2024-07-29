# Estudo Final Testes 

## Referência: Testes automatizados na prática com Spring Boot
[Compre aqui](https://www.udemy.com/course/testes-automatizados-na-pratica-com-spring-boot/)
<hr>

## Dica para leitura:
Durante o estudo, os códigos mudam pois geralmente é ensinado algo básico onde depois iremos implementar o que de fato
é utilizado no mercado de trabalho. Tome cuidado ao considerar códigos do início do estudo, se atente ao código final.

## Tópicos


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
somente no ambiente de desenvolvimento e os unitários em ambiente de produção, por exemplo: 

### Para fazermos isso com o Maven, utilizaremos plugins

Por padrão, os testes que possuem o sufixo "Test", rodam na mesma fase. Portanto, teremos um plugin para testes de 
unidade e outro para de integração.


- #### Testes de Unidade (mais leves) - Plugin SureFire
```xml
<!-- Unit Tests -->
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>3.3.1</version>
  <configuration>
    <argLine>-XX:+EnableDynamicAgentLoading -Xshare:off</argLine>
  </configuration>
</plugin>
```

#### Testes de Integração (mais pesados) - Plugin Failsafe
```xml
<!-- Integration Tests -->
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-failsafe-plugin</artifactId>
  <executions>
    <execution>
      <goals>
        <goal>integration-test</goal>
        <goal>verify</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

Os testes de unidade é sempre importante executar, já são executados por padrão. Os de integração que precisam ser
configurados.

<hr>

## Testes de cobertura com o Jacoco

Como saber se testamos todos os caminhos possíveis que a nossa aplicação poderia seguir? A cobertura irá checar se 
todos os fluxos de códigos que foram implementados estão sendo exercitados pelo teste. Isso são **testes de cobertura**.

A forma padrão de fazer isso é utilizando o plugin Jacoco. Ele mede esta cobertura de testes e gera um relatório muito
útil.

O Jacoco por padrão não é executado ao inicio do teste. Precisamos dizer o que vai acontecer, os alvos de cada etapa
que está sendo executado no teste.

```xml
<!-- https://mvnrepository.com/artifact/org.jacoco/jacoco-maven-plugin -->
<plugin>
  <groupId>org.jacoco</groupId>
  <artifactId>jacoco-maven-plugin</artifactId>
  <version>0.8.8</version>
  <executions>
    <!-- Preparar a coleta de dados antes dos testes, gerando o arquivo jacoco.exec -->
    <execution>
      <id>prepare-agent</id>
      <goals>
        <goal>prepare-agent</goal>
      </goals>
    </execution>
    
    <!-- Gerar o relatório de cobertura após a execução dos testes -->
    <execution>
      <id>report</id>
      <phase>verify</phase>
      <goals>
        <goal>report</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

Para gerar o arquivo jacoco + o relatório de cobertura:
```
mvn clean verify
```

Será gerado um pacote "site", nele, abrimos o arquivo index.html que conterá o report completo.

![img_1.png](img_1.png)
<hr>

### Pequenos ajustes

#### Método main não sendo chamado
É bacana a gente verificar que, por exemplo, o sw.planetapi teve uma cobertura muito baixa, e é um pacote raiz da
aplicação.

Mas, ao adentrarmos para verificar o que houve, vemos que o que aconteceu é que o método main da classe
SwPlanetApiApplication, não foi chamado. 

![img_2.png](img_2.png)

Isso é bem comum, são chamados de falsos positivos. Podemos então configurar o jacoco para ignorar a application.

Para resolver isso, dentro do pom, abaixo da versão do plugin:

```xml
<configuration>
    <excludes>
        <exclude>com/demo/swplanetapi/SwPlanetApiApplication.class</exclude>
    </excludes>
</configuration>
```      

E agora o relatório ficará assim:

![img_3.png](img_3.png)
<hr>

#### toString não sendo coberto em Planet

![img_4.png](img_4.png)

O toString só foi criado para que pudéssemos visualizar os dados no terminal do teste. Não usamos ele para implementar
nenhuma regra de negócio. Além disso, não faz nenhum sentido considerármos o método toString para análise de cobertura.

Então, desconsideraremos o método toString da análise. O Jacoco naturalmente não permite exclusão de métodos, então
criaremos um artifício para adicionar um comportamento exclusivo para um método, para que só ele seja desconsiderado.
Então, **criaremos uma anotação personalizada**.

1. Criaremos um pacote chamado Jacoco dentro da pasta API;
2. Criaremos uma anotação chamada ExcludedFromJacocoGeneratedReport;
3. Ela terá duas propriedades, Retention e Target;
   - @Retention(RetentionPolicty.RUTIME) - Ou seja, será aplicada em tempo de execução;
   - @Target(ElementType.METHOD) - Será aplicada para um método que estiver usufrindo esta anotação.

Por padrão, o Jacoco irá procurar se existe alguma coisa com "Exclude" no nome. E se sim, ele desconsiderará da
cobertura. Depois disso é só ir até a classe Planet e passar a anotação:
```java
@ExcludedFromJacocoGeneratedReport
    @Override
    public String toString() {
        return "Planet{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", climate='" + climate + '\'' +
                ", terrain='" + terrain + '\'' +
                '}';
    }
```

Relatório após isso:

![img_5.png](img_5.png)
<hr>

#### QueryBuilder - Construtor não sendo chamado

![img_6.png](img_6.png)

A gente executa o makeQuery, mas não construímos um QueryBuilder, pois ele é estático. Nesses casos, para a análise
de cobertura funcionar, criaremos um construtor vazio e colocaremos como private, usando somente os seus métodos
estáticos.

![img_7.png](img_7.png)

Relatório:

![img_8.png](img_8.png)

<hr>

## Teste Mutantes com Pitest

É possivel aferir a qualidade de um Switch de testes? A ideia dos testes é: **houve alguma mudança no código? Se sim,
ele deve quebrar. Se mudarmos o código e ele não quebrar, nem existe motivo pro teste existir.**

Para avaliarmos isso usaremos os testes mutantes. Em suma, mutações do código original, para ver se o teste detecta essa
mutação e quebra. E bom... se algum teste sobreviver, temos um mutante.

Usaremos o **pitest**, para testar os nossos testes de unidade.

```xml
<dependencies>
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.10.2</version> <!-- Use a versão mais recente disponível -->
    <scope>test</scope>
  </dependency>
  
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.10.2</version> <!-- Use a versão mais recente disponível -->
    <scope>test</scope>
  </dependency>
</dependencies>

        <!-- Plugin PIT -->
<plugin>
  <groupId>org.pitest</groupId>
  <artifactId>pitest-maven</artifactId>
  <version>1.16.1</version> <!-- Use a versão mais recente disponível -->
  <dependencies>
    <dependency>
      <groupId>org.pitest</groupId>
      <artifactId>pitest-junit5-plugin</artifactId>
      <version>1.2.1</version>
    </dependency>
</dependencies>

  <configuration>
    <targetTests>
      <param>com.demo.swplanetapi.*Test</param> <!-- Ajuste para o seu pacote de testes -->
    </targetTests>
  </configuration>
  <executions>
    <execution>
      <goals>
        <goal>mutationCoverage</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

Os testes mutantes são muito mais lentos, pois cria combinações de código para mutar o nosso código original, por isso,
para rodar com teste de integração não vale a pena.

Para testar no terminal:
```
mvn clean test pitest:mutationCoverage
```

E depois, dentro do pacote "pit-reports", teremos uma pasta com o "YY/MM/DD/HHMM", com o index.html.

![img_9.png](img_9.png)

O que faltou exercitar?

### Dentro de Domain

![img_10.png](img_10.png)


#### Planet.java

Equals e toString não quebraram.

![img_11.png](img_11.png)


O que devemos fazer, é excluí-los dos testes mutantes dentro do pom (em configuration e excludeClasses).
```xml
<configuration>
  <targetTests>
    <param>com.demo.swplanetapi.*Test</param> <!-- Ajuste para o seu pacote de testes -->
  </targetTests>
  <excludedClasses>
    <param>com.demo.swplanetapi.domain.Planet</param>
  </excludedClasses>
</configuration>
```

E com isso, não teremos mais falsos positivos.

Pit atualizado:

![img_12.png](img_12.png)
<hr>

#### QueryBuilder.java

A única coisa que "faltou" é o construtor private QueryBuilder.

![img_13.png](img_13.png)

Isso a gente não considera, pois é um falso positivo. Não precisa ser chamado.

Isso nós dá a cobertura de 100% dos testes mutantes, cobrindo o MÁXIMO de cenários possíveis.

<hr>

## Testes de integração com Testcontainers

Nesse projeto, utilizamos um .properties dentro dos testes para utilizar um banco dedicado a testes. O problema dessa
abordagem é que dependemos de um banco DEDICADO para fazer os nossos testes. Ou seja, se alguem está mexendo no 
simultaneamente, pode ter algumas instabilidades, a ponto de ficar procurando erro que não existe.

O interessante é isolar, deixando os testes menos frágeis, para não dependerem de banco de dados externos, por exemplo.

Dentro do Spring, tem a possibilidade de usarmos **Containers**/ContainersDocker. Ou seja, ao invés de usar um banco 
externo (fisico), vamos subir um container para representar esse banco mySQL, com a biblioteca TestContainers.

```xml
<dependency>
   <groupId>org.testcontainers</groupId>
   <artifactId>junit-jupiter</artifactId>
   <scope>test</scope>
</dependency>

<dependency>
   <groupId>org.testcontainers</groupId>
   <artifactId>testcontainers-bom</artifactId>
   <version>1.20.0</version>
   <type>pom</type>
   <scope>import</scope>
   </dependency>

<dependency>
   <groupId>org.testcontainers</groupId>
   <artifactId>mysql</artifactId>
   <scope>test</scope>
</dependency>
```

Com isso, precisamos dizer no teste (PlanetIT), qual a imagem que vamos subir para representar esse banco quando
rodarmos esse teste. Para isso, instalaremos o docker.

Com ele, podemos baixar as imagens que o teste precisa (mysql) e executar o teste sem nenhum problema.

### Como utilizar o banco do test containers e não o de desenvolvimento?
1. Tiraremos username e senha;
2. Mudar URL do banco, é só colocar depois dos :
<hr>

## Testes Parametrizados

<hr>

# Resumo

Lembrar de colocar o Jacoco para ignorar a [application](#método-main-não-sendo-chamado)

Pitest para testar [mutações](#teste-mutantes-com-pitest) em testes de unidade.
<hr>

## Fim

