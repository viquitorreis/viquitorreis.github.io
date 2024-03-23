---
title: "Binary search"
date: 2024-02-21T13:56:32-03:00
draft: false
authors: [Victor]
categories: [Algoritmos]
tags: [Algoritmos, Busca, Desenvolvimento]
---
# Binary Search ( Pesquisa binária )

## Tabela de conteúdos
  * [O que é o Binary Search](#o-que-é-o-binary-search)
  * [Binary search x Linear search](#binary-search-x-linear-search)
  * [Runtime Binary Search](#runtime-do-binary-search)
  * [Implementando o Binary Search: Golang](#implementando-o-binary-search-golang)

## O que é o Binary Search
O Binary search é um algoritmo que faz uma **busca eficiente** para achar um item qualquer em uma ***lista ordenada***. A forma que funciona é muito simples: ele vai dividir a lista ao meio ( na parte que pode conter o elemento buscado ), até reduzir a lista o suficiente para achar o elemento ( ou não ). 

## Binary search x Linear search
Em 2023, é estimado que a população do planeta terra seja de 8 bilhões de pessoas. Imagine um cenário onde teriamos que fazer uma busca em todas essas pessoas do nosso planeta para encontrar um *Arquiteto de Cidades Invisíveis*. Vamos simular 2 cenários: busca linear e binary search, considerando que a ordem dos nomes na lista estão **ordenados**.

**Busca linear:**

Supondo que criaremos um algoritmo para fazer uma busca simples (linear search) nessa lista com todos os indivíduos:
![Linear Search](/binary_search/Busca.png "Linear Search")

No **pior cenário** teriamos que buscar todas as pessoas da lista e adotando como base que cada busca + checagem gastaria 1 milisegundo, a busca completa iria nos custar pouco mais de **3 meses**. Isso seria tempo demais, será que temos uma solução melhor para esse problema?

**Melhor forma de busca:**

Se começarmos na *metade da lista*, ou seja, na pessoa número 4 bilhão, **eliminamos metade das tentativas** já de início:

![Chute na metade](/binary_search/2.png "Inicio binary search")

Como a resposta foi que nosso chute estava baixo demais, todos os elementos do começo até a metade serão desconsiderados. Por mais que erramos na nossa tentativa *eliminamos metade das possiblidades*

* No próximo chute vamos **chutar a metade do restante**.

![Buscando no binary search](/binary_search/binary-search-3.png "Buscando no binary search")

No binary search, continuamos esse processo de dividir as possibilidades restantes pela metade até acharmos o elemento desejado

Fazendo o Binary Search, levaríamos no máximo **33 buscas** para achar o valor desejado, se cada busca = 1 ms, gastaríamos 33 milisegundos para fazer essa busca.

* **Logo:**
  - Linear Search => gastaríamos **3 meses** para fazer a busca
  - Binary Search => gastaríamos **33 milisegundos** para buscar

## Runtime do Binary Search
Para calcularmos quanto tempo pode gastar a busca do Binary Search ( no pior cenário possível - o último elemento a buscar ser o que queremos ), basta fazermos uma simples operação. Geralmente, quando falamos de runtime de algoritmos quase sempre iremos calcular o logaritmo na *base 2*:

![Calculo Runtime](/binary_search/binary-search-3.png "Cálculo runtime")

No nosso caso, log de 8 bilhões na base 2 ≃ 33

## Implementando o Binary Search: Golang
Para implementarmos o Binary search podemos simplesmente seguir esses passos:

- [x] Precisamos do valor do menor elemento e do maior elemento
- [x] Fazemos um loop, enquanto o menor elemento for menor ou igual que o maior, continuamos a busca ( caso não acharmos o elemento desejado)
- [x] Pegar o valor do meio
- [x] 'Chutar' com o valor do meio
- [x] Se o chute for muito baixo, o menor elemento vai passar a ser maior que elemento do meio
- [x] Se o chute for muito baixo, o maior elemento vai passar a ser menor que o elemento do meio
- [x] O loop continua até acharmos o elemento desejado ( ou não )

* Implementação em golang:

```go
package main

import (
	"sort"
)

func binary_search(item int) int {
	arr := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99}
	// arr deve estar ordenada...
	sort.Ints(arr)

	lo := 0
	hi := len(arr) - 1
	for lo <= hi {
		mid := (hi + lo) / 2
		guess := arr[mid]

		if guess == item {
			return mid
		}

		if guess > item {
			hi = mid - 1
		} else {
			lo = mid + 1
		}
	}

	return 0
}

func main() {
	binary_search(7)
}
```