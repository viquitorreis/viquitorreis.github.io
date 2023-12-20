---
title: "Binary search"
date: 2023-12-19T13:34:32-03:00
draft: false
authors: [Victor]
categories: [Algoritmos]
---
# Binary Search ( Pesquisa binária )

## Tabela de conteúdos
  * [O que é o Binary Search](#chapter-1)
  * [Binary search x Linear search](#chapter-2)
  * [Chapter 3](#chapter-3)

## O que é o Binary Search
<a id="chapter-1"></a>
O Binary search é um algoritmo que faz uma **busca eficiente** para achar um item qualquer em uma ***lista ordenada***. A forma que funciona é muito simples: ele vai dividir a lista ao meio ( na parte que pode conter o elemento buscado ), até reduzir a lista o suficiente para achar o elemento, caso exista. 

## Binary search x Linear search
<a id="chapter-2"></a>
Em 2023, é estimado que a população do planeta terra seja de 8 bilhões de pessoas. Imagine um cenário onde teriamos que fazer uma busca em todas essas pessoas do nosso planeta para encontrar um *Arquiteto de Cidades Invisíveis*. Vamos simular 2 cenários: busca linear e binary search, considerando que a ordem dos nomes na lista estão **ordenados**.

**Busca linear:**

Supondo que criaremos um algoritmo para fazer uma busca simples (linear search) nessa lista com todos os indivíduos:
![Linear Search](/images/Busca.png "Linear Search")

No **pior cenário** teriamos que buscar todas as pessoas da lista e adotando como base que cada busca + checagem gastaria 1 milisegundo, a busca completa iria nos custar pouco mais de **3 meses**. Isso seria tempo demais, será que temos uma solução melhor para esse problema?