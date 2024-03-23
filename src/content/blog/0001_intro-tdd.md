---
title: "Introdução a TDD"
date: 2024-01-21T12:25:49-03:00
draft: false
authors: [Victor]
categories: [TDD]
tags: [TDD, Testing, Desenvolvimento]
---
# Introdução a TDD ( Test Driven Development )

## Tabela de conteúdos
  * [O que é o TDD](#o-que-é-o-tdd)
  * [Por quê usar o TDD](#por-quê-usar-o-tdd)
  * [Contras do TDD](#contras-do-tdd)
  * [Ciclo do TDD - RED, GREEN, REFACTOR](#ciclo-do-tdd---red-green-refactor)
  * [Começando de pé direito](#começando-de-pé-direito)
  * [Hello Victor](#hello-victor)
  * [Iterações: Testes de Benchmark](#iterações-teste-de-benchmark)

## O que é o TDD?

TDD significa Desenvolvimento Orientado a Testes ( Test driven development ). Então ele consiste na modalidade de desenvolvimento em que escrevemos os testes unitários primeiro

## Por quê usar o TDD
#### Melhora a qualidade do código

- [x] Códigos escritos usando TDD tem maior qualidade
- [x] Tem menor quantidade de erros
- [x] Vai te ajudar a criar um código mais objetivo ( que vai diretamente ao problema específico ).
- [x] O processo de fazer TDD vai automaticamente te ajudar a criar práticas que vai deixar seu código melhor.
- [x] Vai melhorar exponencialmente a sua forma lógica de ver e resolver um problema
- [x] Você vai concentrar seus esforços para resolver partes de código mais curtos e fáceis de ler, auxiliando a criar códigos mais robustos 


#### Melhora o design do seu sistema

Como para escrever os testes antes, escrevemos o teste antes da funcionalidade, o código escrito posteriormente é mais fácil de checar, o que deixa seu sistema melhor desenvolvido. 

Dessa forma, nós desenvolvedores conseguimos alcançar um sistema mais modularizado, fácil de entender, manter, expandir posteriormente ( escalar o sistema ), testar e refatorar ( já que não vai permitir erros em funções criadas posteriormente )

#### Aumenta a produtividade do desenvolvedor

O TDD aumenta a velocidade de desenvolvimento, já que você vai gastar menos tempo debugando ( pensa ficar debugando um sistema pesado o tempo inteiro, que custa rodar na sua máquina para arrumar um bug ).

Nos estágios iniciais do projeto pode gastar mais tempo para criar testes e código que vai para a produção. Apesar disso, a medida que o projeto se desenvolve, adicionar e testar novas funcionalidades vai ir mais rápido com menos trabalho.

``
Quatro equipes de desenvolvimento participaram de um estudo conjunto da Microsoft e IBM. O estudo concluiu que a adoção do TDD reduziu a quantidade de bugs em 40 a 90%. O tempo necessário para concluir os projetos também aumentou em 15 a 35 por cento, APESAR DISSO os custos de manutenção abaixaram exponencialmente e compensaram isso devido à melhoria na qualidade.
``

**obs:** essa pesquisa foi feita em 2008

*fonte:*
https://www.microsoft.com/en-us/research/wp-content/uploads/2009/10/Realizing-Quality-Improvement-Through-Test-Driven-Development-Results-and-Experiences-of-Four-Industrial-Teams-nagappan_tdd.pdf


![Calculo Runtime](/intro-tdd/Cost-of-Maintenance.jpg "Custos gerais em softwares")
*fonte:*
https://community.nasscom.in/communities/project-management/software-maintenance-step-step-guide


#### Reduz os custos do projeto ao longo do tempo

Como o TDD diminui a necessidade de manutenção no código, ele diminui os custos ao longo do tempo e aumenta a produtividade da equipe.

Menos problemas significa menos horas em desenvolvimento, o que afeta diretamente os custos do projeto. Deve-se ter em mente que o TDD quase sempre vai sair um pouco mais caro no período inicial do projeto, mas a medida que o projeto for se desenvolvendo o custo benefício será melhor com o TDD.

#### Obter assistência para prevenção de bugs

Como rodar testes é o foco principal do TDD, o desenvolvedor pode garantir que a aplicação vai funcionar como deve, e precisar apenas de consertos pequenos posteriormente. É muito importante entender que no método do TDD os desenvolvedores colocam ênfase em desenvolver testes antes dos erros acontecerem do que consertar eles depois que o código foi desenvolvido.

#### O TDD funciona como documentação	

Os testes escritos podem servir como documentação, pois quando rodamos e lemos os testes entendemos qual era o objetivo do desenvolvedor que implementou a funcionalidade. Baseado nos testes o desenvolvedor pode entender qual era o input esperado por uma funcionalidade e os resultados desejados

#### Mantém sustentabilidade em processos dependentes

Com o TDD, nós podemos ficar confiantes de que as partes dependentes umas das outras no código irão continuar funcionando como deveriam. Depois de refatorar ou introduzir uma nova funcionalidade, os testes vão te ajudar a avaliar se tudo está funcionando como deveria. Sem o TDD, nós desenvolvedores não conseguimos acompanhar se o código que está sendo desenvolvido atualmente está atrapalhando o funcionamento de funcionalidades criadas anteriormente.

## Contras do TDD

- O TDD vai diminuir a velocidade de desenvolvimento em estágios iniciais do projeto.
- O TDD, a parte de teste em si pode ficar difícil de manter. O desenvolvedor terá que constantemente melhorar ou mudar os testes, já que as funcionalidades do sistema podem mudar a medida que o tempo passa. Então é preciso adaptações constantes nos testes.
- É difícil aprender o método do TDD, pois implementa-se o teste primeiro e depois o código. Logo, é uma zona de desconforto muito grande inicialmente.
- Código de TDD as vezes é difícil de entender.

## Ciclo do TDD - RED, GREEN, REFACTOR 

O processo do TDD de forma simples, consiste em deixar que o compilador te guie na hora de desenvolver os testes. Então o que o compilador reclamar de erro você vai e arruma, depois disso você terá que melhorar o escopo dos testes logicamente.

#### Red

Se escreve o teste para uma funcionalidade específica, deve ser algo simples, um passo por vez. **O teste vai falhar**.

#### Green

Escrever o **mínimo** de código possível para fazer o teste passar

#### Refactor

Após os testes passarem, deve-se **refatorar**, ou seja, diminuir redundâncias e repetições de código

*O processo se repete…*

![Ciclo do TDD](/intro-tdd/tdd_flow1-1.jpeg "Ciclo do TDD")
*fonte:*
https://www.nimblework.com/pt-br/agile/desenvolvimento-orientado-a-testes-tdd/


## Começando de pé direito

Todo desenvolvedor sabe, sempre que começamos algo deve ser feito o ritual sagrado para começarmos de pé direito: printar “Hello world!”

Vamos escrever um teste que irá testar se estamos corretamente printando hello world usando golang.

**OBS:** Em golang, todo arquivo de teste deve terminar com o sufixo “test”

**OBS2:** Em go, toda função de teste deve ter o prefixo “Test”, a não ser que seja testes de benchmark

* Vamos testar uma função que será implementada, e terá o nome de Hello()

**hello_test.go**
```go
package main

import (
   "fmt"
   "testing"
)

func TestHelloWorld(t *testing.T) {
   t.Run("Testando Hello world", func(t *testing.T) {
       got := Hello()
       want := "Hello world!"


       if got != want {
           t.Errorf("Não recebeu a benção ainda, recebeu %q, queria %q", got, want)
       }
   })
}
```

**Rodando:** go test -v

```bash
# testando [testando.test]
./hello_test.go:9:10: undefined: Hello
FAIL    testando [build failed]
```

Podemos ver que o compilador nos indicou que a função Hello está indefinida, vamos implementar ela

* Implementando o mínimo de código possível possível para o teste passar

**main.go**
```go
package main

import "fmt"

func Hello() string {
   helloWorld := "Hello world!"


   return helloWorld
}

func main() {
   fmt.Println(Hello())
}
```

* **Vamos rodar o teste novamente**: go test -v

```
=== RUN   TestHelloWorld
=== RUN   TestHelloWorld/Testando_Hello_world
--- PASS: TestHelloWorld (0.00s)
    --- PASS: TestHelloWorld/Testando_Hello_world (0.00s)
PASS
ok      testando        0.001s
```

## Hello Victor

Nosso próximo passo agora é conseguir especificar a pessoa que queremos cumprimentar. Vamos implementar essa funcionalidade.

Vamos testar um Hello, Victor.

Red -> vamos escrever um teste que vai falhar. Nesse primeiro momento a intenção é deixar que o compilador nos guie

**hello_test.go**
```go
func TestHelloWorld(t *testing.T) {
   t.Run("Testando Hello world", func(t *testing.T) {
       got := Hello("Victor")
       want := "Hello, Victor"

       if got != want {
           t.Errorf("Não cumprimentou corretamente, recebeu %q, queria %q", got, want)
       }
   })
}
```

* **Agora vamos testar isso:**: go test -v

```bash
# testando [testando.test]
./hello_test.go:9:16: too many arguments in call to Hello
        have (string)
        want ()
FAIL    testando [build failed]
```

O compilador está nos dizendo que tem mais argumentos na função Hello do que deveria. Vamos arrumar nossa função

Green -> vamos implementar um código simples que passe no teste

**main.go**
```go
package main

import "fmt"

func Hello(name string) string {
   helloWorld := fmt.Sprintf("Hello, %s", name)


   return helloWorld
}
```

* **Vamos testar esse código:**: go test -v

```bash
=== RUN   TestHelloWorld
=== RUN   TestHelloWorld/Testando_Hello_world
--- PASS: TestHelloWorld (0.00s)
    --- PASS: TestHelloWorld/Testando_Hello_world (0.00s)
PASS
ok      testando        0.002s
```

## Iterações: Teste de Benchmark
	
Uma boa forma de testar Benchmarks é testar a quantidade de iterações ou operações que uma determinada funcionalidade em nosso sistema está fazendo.

Para a nossa sorte, na biblioteca padrão de Go já vêm com a funcionalidade testing.B que é feita pensando em testes de benchmark, ou seja, não precisamos de nenhuma biblioteca externa para testar benchmarks em Go!

Vamos simular isso de forma simples com iterações.

**Red ->** Vamos escrever um teste que será reprovado. Vamos implementar um teste que confere se uma função repete o caractere V 10 vezes

**benchmark_test.go**
```go
package benchmark

import "testing"

func TestRepeat(t *testing.T) {
   got := Repeat("v")
   want := "vvvvvvvvvv"

   if got != want {
       t.Errorf("Esperava %q mas recebeu %q", want, got)
   }
}
```

* **Testando isso:**: go test -v

```bash
# testando [testando.test]
./hello_test.go:19:9: undefined: Repeat
FAIL    testando [build failed]
```

**Green ->** Escrevendo o mínimo de código para passar no teste

Em go, não existe while e nem do. Todos os loops são feitos com for, obviamente existem formas diferentes de se usar o for na linguagem

**benchmark.go**
```go
package benchmark

func Repeat(character string) string {
   var repeated string

   for i := 0; i < 10; i++ {
       repeated = repeated + character
   }

   return repeated
}
```

***Note que:*** também podemos armazenar variável em go usando o var. Perceba também que no nosso loop não usamos nenhum parênteses para a condição, como em outras linguagens conhecidas


**Testando:** go test -v

```bash
=== RUN   TestRepeat
--- PASS: TestRepeat (0.00s)
PASS
ok      testando/benchmark      0.001s
```

Como podemos ver o teste passou, e ao contrário do anterior ele veio com menos informações, pois não usamos o t.Run() para fazer o teste

Refatorando -> vamos refatorar isso agora. Vamos também usar outro operador para fazer a soma na nossa variável var, vamos usar o operador +=

**benchmark.go**
```go
package benchmark

const repeatCount = 10

func Repeat(character string) string {
   var repeated string

   for i := 0; i < repeatCount; i++ {
       repeated += character
   }

   return repeated
}
```

* **Testando novamente**: go test -v

```bash
=== RUN   TestRepeat
--- PASS: TestRepeat (0.00s)
PASS
ok      testando/benchmark      0.001s
```

***Note que:*** declarei a constante sem especificar o tipo, Go inferiu automáticamente para mim, mas eu podia ter inferido o tipo também se quisesse:

```go
const repeatCount int = 10
```

Essa nomenclatura também é válida

#### Benchmarking

Agora vamos simular um benchmark em go. Escrever benchmarks é bem parecido com escrever testes normais em Go, e não precisamos de nenhuma biblioteca externa para isso.

```go
func BenchmarkTest(b *testing.B) {
   b.Run("Testando benchmark", func(b *testing.B) {
       for i := 0; i < b.N; i++ {
           Repeat("v")
       }
   })
}
```

Para testar *benchmarks* usamos o comando: **go test -bench=.**

```bash
goos: linux
goarch: amd64
pkg: testando/benchmark
cpu: Intel(R) Core(TM) i7-7500U CPU @ 2.70GHz
BenchmarkTest/Testando_benchmark-4               3777550               302.8 ns/op
PASS
ok      testando/benchmark      1.469
```