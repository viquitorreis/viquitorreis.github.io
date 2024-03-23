---
title: "HTTP Server em GO com testes"
date: 2024-03-23T13:56:32-03:00
draft: false
authors: [Victor]
categories: [Unit-testing, GO]
tags: [TDD, Unit-testing, GO, HTTP, API]
---

Vamos criar um server HTTP para o seguinte:

**GET /players/{name}** deve retornar um número indicando o total de vitórias

**POST /players/{name}** deve retornar um registro daquele nome, incrementando para cada POST subsequente

- Vamos seguir a abordagem **TDD,** ter um software funcionando o mais rápido que pudermos e então fazermos melhorias pequenas até que cheguemos à solução. Com essa abordagem vamos:

1. Manter o espaço de problema pequeno a cada momento

2. Não vamos cair rabbit holes

3. Caso fiquemos perdidos ou travados, fazer um revert não vai perder muito trabalho


## Red, green, refactor

O processo de TDD consiste em:

1. Escrever um teste e ver ele falhar ( red )

2. Escrever o mínimo de código possível para fazer passar no teste ( green )

3. Refatorar ( melhorar o código )

**A meta é sair do red o mais rápido possível**

Assim que o teste passar **você pode commitar as mudanças** 


### E se eu não fizer isso?

Quanto mais código você adicionar enquanto estiver em red, mais você vai adicionar problemas que não estão cobertos pelos testes

A ideia é escrever código útil em passos pequenos, dirigidos por testes para que **não caia em rabbit holes durante horas**


#### Galinha e ovo

Como podemos fazer isso? Não podemos fazer um **GET** nos users sem armazenar algo, e é complicado dizer se o **POST** funciona se não conseguimos averiguar com o **GET**

É ai que existe o **mocking:**

- O **GET** vai precisar de um **PlayerStore** para pegar as pontuações de um jogador. Isso deve ser uma **interface** para quando testarmos vamos poder criar um um esboço simples para testar nosso código sem precisar ter implementado nenhum código de storage ( banco de dados )

- Para o **POST** podemos “espiar” nas chamadas em **PlayerStore** para garantir que armazenou os jogadores corretamente. Nossa implementação de salvar não vai estar acoplada para recuperar nada

- Para ter um software que funcione rapidamente podemos fazer uma implementação e de muito simples **na memória** e depois podemos criar uma implementação com storage se quisermos


### Escreva o teste primeiro

Podemos escrever um teste para passar ao retornar um valor hard coded para começarmos. Depois que tivermos um teste funcionando podemos escrever mais testes 

- Para criarmos um web server em Go geralmente vamos chamar o **ListenAndServe**

![](https://lh7-us.googleusercontent.com/pI-14NP-RJ2OtNU2o3ZH4M2G1JJDrMpG2tWyBWtMTZbtPqjUeJoMgbQ17Afe5-3WO2sQfw2fv-cRh6thhlLtV6esCxEdpT0Otw6It8SrSLLFHqHTVPEImxIKkGeRifUXZ0FrTIHxCpxYLACAV_aaXsM)

Isso vai iniciar um web server que vai escutar em uma porta, criando uma **goroutine** para **cada requisição** e rodando isso contra um **Handler**

|                                                                         |
| ----------------------------------------------------------------------- |
| **type Handler interface { ****ServeHTTP(ResponseWriter, \*Request) }** |

********Essa interface implementa um Handler ao implementar o método **ServeHTTP** que espera **2 argumentos.** No primeiro escrever a **resposta** e o segundo escrevemos a **request HTTP** que foi enviada ao nosso servidor

- Vamos criar um arquivo **server\_test.go** e escrevemos um teste para a função **PlayerServer** que espera **dois argumentos.** A **request enviada** vai pegar a pontuação do jogador, que **esperamos que seja 20**

<!---->

    import (
       "net/http"
       "net/http/httptest"
       "testing"
    )

<!---->

    func TestGETPlayers(t *testing.T) {
       t.Run("Buscando o score do jogador Victor", func(t *testing.T) {
           request, _ := http.NewRequest(http.MethodGet, "players/victor", nil)
           response := httptest.NewRecorder()

<!---->

           PlayerServer(response, request)

<!---->

           got := response.Body.String()
           want := "20"

<!---->

           if got != want {
               t.Errorf("Recebido %q esperava %q", got, want)
           }
       })
    }

  ****

********Para testarmos nosso servidor precisamos de uma **Request** para enviar e então vamos **espiar** o que o nosso handler escreveu no **ResponseWriter**

- Usamos **http.NewRequest** para criar a request. O primeiro argumento é o **método da request** e o segundo parâmetro é o **path da requisição.** O argumento **nil** se refere ao **body da requisição.** no qual não precisamos nesse caso

- **net/http/httptest** tem um “espião” já feito para nós chamado **ResponseRecorder** e então podemos usar ele. Vai ser muito útil para inspecionar dentro de métodos e ver o que tem escrito dentro das response


#### Rodando o teste

![](https://lh7-us.googleusercontent.com/nbmm4hS9jZXU_id-wqQc2vhW0DpBQ3FXCVjr0Qb6_1BcRIOqYZBhCb-g94HsTE_GYmW0A_EQhBetodcCqYVHS0TVxCVVhlEWpv0rGbO40Au2ztpOmaUVEr2hLY306hogNi2YAuFJO3aDQOENuxmm2Dk)


#### 1 - Escrevendo o mínimo de código para o teste rodar

Vamos deixar o **compilador nos ajudar,** precisamos escutar ele

- Vamos criar um arquivo **server.go** e definir um **PlayerServer**

|                            |
| -------------------------- |
| **func PlayerServer() {}** |

********Testando novamente:

![](https://lh7-us.googleusercontent.com/mF-QuqYhCZ6XbX6NbKnlIqG9KRzn6DvnUGcwJRcr_7zLQp9fghWdKgZVGoMTiGvQJTFZ3rprEs3bHkx5NaanT06vNSOZidpmuWwb-n0UACLhHGsAabD_zlhBgcPbb9C428V9YD3Vl40OlyTZG6Yoeus)

- Vamos adicionar **argumentos para a nossa função**

<!---->

    package main

<!---->

    import "net/http"

<!---->

    func PlayerServer(w http.ResponseWriter, r *http.Request) {

<!---->

    }

- Vamos testar novamente

![](https://lh7-us.googleusercontent.com/xzA46FSVkoXU1Nqo51PFABpyssGubY2uWkYE-U43b-1R3T8q6mJ0eCLBqn1Sy3KFE1eDBxJKTkXgTzcSyGLNKv5QYHQm5rJWjJCulrwE5rSZt_0YpqQnW9CFB1hIZLloHSmONezWtoJk-DMUOb0WMQ4)


#### 2 - Escrevendo o mínimo de código para passar

O método **ResponseWriter** do pacote **net/http** também implementa o io **Writer,** dessa forma podemos usar o **fmt.Fprint** para enviar strings como respostas HTTP

    import (
       "fmt"
       "net/http"
    )

<!---->

    func PlayerServer(w http.ResponseWriter, r *http.Request) {
       fmt.Fprint(w, "20")
    }

O teste agora deve funcionar

![](https://lh7-us.googleusercontent.com/lKRdA_-mTO9t7dYgXrICftYMb0ApOQcNLyobp1sA7Im6anuYHhm4n-ecKtS4kdNB8yQxlz6HTCOgAnuL2QKJmaXw9-jjuGWxw-ss8fi5cD5JKRBL5iQ5WnzaIeanwOomb0mbzqVeJDxiUTtog2t59YY)

:) 


#### 3 - Completando a estrutura

Vamos inicializar nossa API agora em main.go. Precisamos escrever de uma forma que podemos modificar o código depois

**main.go:**

    import (
       "log"
       "net/http"
    )

<!---->

    func main() {
       handler := http.HandlerFunc(PlayerServer)
       log.Fatal(http.ListenAndServe(":5000", handler))
    }

- Vamos gerar o binário e rodar isso:

![](https://lh7-us.googleusercontent.com/VELY1j7mK0N9uw3GPJ48jg89oXA3Sb5iq0kJSj5S60dWkyXxGyqs6icH5RNbLcOret-hS5yJRN3MT5N5-C8BCMsUAptlWCRu90ldJoqEe19z5TZ5gWJLPxEcSyj7XqNh6YQDbgdt6wk3lZoqA--kS0Y)


### http.HandlerFunc

Anteriormente falamos que a interface **Handler** é necessário implementar em ordem para fazer um servidor. _Tipicamente_ o que fazemos é criar uma **struct** e depois fazer isso implementar uma interface e fazer o próprio método **ServeHTTP.** Apesar disso, o use-case para structs é armazenar dados, mas _atualmente_ **não temos nenhum estado, então não parece razoável criar uma**

********O **HandlerFunc** ajuda a gente a evitar isso:

|                                                                                                                                                                                         |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **O tipo HandlerFunc é um adaptador que permite o uso de funções ordinárias como handlers HTTP. Se f é uma função com uma assinatura correta, HandlerFunc(f) é um Handler que chama f** |

    type HandlerFunc func(http.ResponseWriter, *http.Request)

<!---->

    func main() {
       handler := http.HandlerFunc(PlayerServer)
       log.Fatal(http.ListenAndServe(":5000", handler))
    }


### Obrigando a remover valores hard-coded

********Vamos escrever outro teste para **forçar** que a gente faça uma mudança positiva para tentar **remover os valores hard-coded**


#### Escrevendo o teste primeiro

Vamos adicionar outro subteste a nossa bateria de testes, que vai tentar pegar o score de cada jogador diferente, que vai quebrar nossa **iniciativa hard-coded.** 

    t.Run("Buscando o score do jogador Cícero", func(t *testing.T) {
           request, _ := http.NewRequest(http.MethodGet, "players/cicero", nil)
           response := httptest.NewRecorder()

<!---->

           PlayerServer(response, request)

<!---->

           got := response.Body.String()
           want := "10"

<!---->

           if got != want {
               t.Errorf("Recebido %q esperava %q", got, want)
           }
       })


#### Rodando o teste

![](https://lh7-us.googleusercontent.com/51VZlwhsftwN_rtDGI8Ej5yAS7aSFsUHlS89qkcZUrk8arWSSBU_SvnZVnIgb5LyyJ7QgG3UAj_TsNckQRiOdApeIn4ovjP5_VHK5ZunERYbOwVwz2iI9eHgUs0FGd3cqlGjqOGszp81fFuVSVMW4is)


#### 1- Escrevendo o mínimo de código pra passar

**serve.go**

    func PlayerServer(w http.ResponseWriter, r *http.Request) {
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       if player == "victor" {
           fmt.Fprintf(w, "20")
           return
       }

<!---->

       if player == "cicero" {
           fmt.Fprintf(w, "10")
           return
       }
    }

Se tivéssemos começado com algum storage, teriamos que fazer mudanças enormes em comparação com essa abordagem. **Usando TDD podemos fazer passos pequenos no sentido do nosso objetivo**

********Vamos evitar qualquer biblioteca de routing nesse momento para fazermos nosso teste passar usando **passos pequenos**

**r.URL.Path** retorna o path da request na qual podemos depois usar o **strings.TrimPrefix** para “apararmos / removermos” o **/players/** e pegar **o player desejado**


#### Refatorando

Podemos simplificar o **PlayerServer** ao separar o score **retornado** em uma função

**server.go**

    func PlayerServer(w http.ResponseWriter, r *http.Request) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       fmt.Fprintf(w, GetPlayerScore(player))
    }

<!---->

    func GetPlayerScore(name string) string {
       if name == "victor" {
           return "20"
       }

<!---->

       if name == "cicero" {
           return "10"
       }

<!---->

       return ""
    }

- Podemos diminuir um pouco do nosso código nos testes ao **criar uns helpers:**

**server\_test.go**

    package main

<!---->

    import (
       "fmt"
       "net/http"
       "net/http/httptest"
       "testing"
    )

<!---->

    func TestGETPlayers(t *testing.T) {
       t.Run("Buscando o score do jogador Victor", func(t *testing.T) {
           request := newGetScoreRequest("victor")
           response := httptest.NewRecorder()

<!---->

           PlayerServer(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "20")
       })

<!---->

       t.Run("Buscando o score do jogador Cícero", func(t *testing.T) {
           request := newGetScoreRequest("cicero")
           response := httptest.NewRecorder()

<!---->

           PlayerServer(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "10")
       })
    }

<!---->

    func newGetScoreRequest(name string) *http.Request {
       req, _ := http.NewRequest(http.MethodGet, fmt.Sprintf("/players/%s", name), nil)
       return req
    }

<!---->

    func assertResponseBody(t testing.TB, got, want string) {
       t.Helper()
       if got != want {
           t.Errorf("O response body está errado, got %q want %q", got, want)
       }
    }

Apesar disso não podemos ficar tão feliz. **Não é correto o servidor saber os scores**

Movemos o cálculo do score da nossa função fora do body da função main do nosso handler dentro de uma função **GetPlayerScore.** Isso parece como o lugar correto para separar preocupações usando interfaces

- Vamos mover a função que refatoramos para ser uma **interface**

**server.go**

    type PlayerStore interface {
       GetPlayerScore(name string) int
    }

Para o nosso **PlayerServer** conseguir usar o **PLayerStore,** vamos precisar referenciar um. Agora parece o tempo correto para mudar nossa arquitetura para que o **PlayerServer** seja uma **struct** 

    type PlayerServer struct {
       store PlayerStore
    }

   ****

- Agora vamos implementar a interface **Handler** ao adicionar o método na nossa struct e colocar isso em um código handler existente. 

<!---->

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       fmt.Fprint(w, p.store.GetPlayerScore(player))
    }


#### Arrumar os erros

Fizemos muitas mudanças e o código não vai compilar, mas vamos testar e deixar o compilador nos guiar

![](https://lh7-us.googleusercontent.com/9Dwrb1u21GDoUwplQmBS2g8KEKcHz9H0f5_ItUK-ZBJxanoK9Q6mhksGo0EuTvPPfn4F192h3hMr0Bh9JHLsZhFueq5cGulPGlHdUvOwYYr9FFV-i0A7IOLopv4Adbc9DK5VflhMRNuLv31pgNbv9j4)

|                                                                 |
| --------------------------------------------------------------- |
| ./main.**go**:11:30: **type** PlayerServer is not an expression |

- Precisamos mudar nossos testes para **criar uma nova instância** do nosso **PlayerServer** e depois chamar o método **ServeHTTP** 

<!---->

    func TestGETPlayers(t *testing.T) {
       server := &PlayerServer{}

<!---->

       t.Run("Buscando o score do jogador Victor", func(t *testing.T) {
           request := newGetScoreRequest("victor")
           response := httptest.NewRecorder()

      // MUDANÇA
           server.ServeHTTP(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "20")
       })

<!---->

       t.Run("Buscando o score do jogador Cícero", func(t *testing.T) {
           request := newGetScoreRequest("cicero")
           response := httptest.NewRecorder()

<!---->

     // MUDANÇA
           server.ServeHTTP(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "10")
       })
    }

**AINDA NÃO ESTAMOS NOS PREOCUPANDO COM STORES.** Só queremos fazer o compiler passar o mais rápido possível

**Precisamos ter o hábito de priorizar ter código que compile e depois código que passe os testes**

- Agora nosso **main.go** não vai compilar pela mesma razão

<!---->

    func main() {
       server := &PlayerServer{}
       log.Fatal(http.ListenAndServe(":5000", server)) // se tiver algum erro ao servir nosso servidor, o '1og' vai reportar o erro e vai terminar o programa
    }

teste:

![](https://lh7-us.googleusercontent.com/NQxIWHzqTLE028mL54el8LwbhRx3iR-wtK-Dj9shVD6lO1aUTABX7Ft2Us351stjEksnqbPQNeRJRiTDdhmb6I750Dnkl7WnYOoUHXYbFOklpdQz3MV0uTSzUyp2hmVE7CnTJc4DMm55-lXWChLnmyg)

Está compilando mas os testes estão falhando.

- Isso aconteceu pois não passamos um **PlayerStore** nos nossos testes. Precisamos fazer uma estrutura **stub** ( esboço ) de uma

**server\_test.go**

    type StubPlayerStore struct {
       scores map[string]int
    }

<!---->

    func (s *StubPlayerStore) GetPlayerScore(name string) int {
       score := s.scores[name]
       return score
    }

Um **map** é uma boa estrutura para fazer um **stub key/value** para os nossos testes. Agora vamos criar um desses stores para os nossos testes e enviar para o nosso **PlayerServer**

**server\_test.go**

    func TestGETPlayers(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{
               "victor": 20,
               "cicero": 10,
           },
       }
       server := &PlayerServer{&store}

<!---->

       t.Run("Buscando o score do jogador Victor", func(t *testing.T) {
           request := newGetScoreRequest("victor")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "20")
       })

<!---->

       t.Run("Buscando o score do jogador Cícero", func(t *testing.T) {
           request := newGetScoreRequest("cicero")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "10")
       })
    }

- Agora nossos testes devem passar e já estão um pouco mais elaborados. A intenção atrás do nosso código é 

![](https://lh7-us.googleusercontent.com/YmmdTFjLvrNcOpDT9fnzRMkrH6A0I7p0uBSYMdF_qw62nL_FE3_GaiYcP0iGfTP_0JicaUTNju3re7MIEEgPFRLTW9Mde3aVL7PDGAJaYZsy4wVubtFEkHtoSZRz_toAAZbSwYf9bd949SeHRZohDkQ)


#### Passando PlayerStore para o server

Vamos passar um **PLayerStore** simples para passar ao server, já que fazer um storage real agora vai ser um pouco complicado.

![](https://lh7-us.googleusercontent.com/6afqprP9Kfuji3giT4nga2XjsKDafv0QBFbj1v1VZvmYDmOm8FRWdGkAA3RlVxRmIO4fR4h9o-rSKDvSk2eqr5KXwFV12xao3duHTYX1yXR3fNgi8IMQ2cxXRCKOM2manNjQRvkXyHukZXlRufBt_lQ)

- Se fizermos **go build** e fomos na url [**http://localhost:5000/players/victor**](http://localhost:5000/players/victor) ****vai ser retornado 123

![](https://lh7-us.googleusercontent.com/kKvdz0EEOfby4OloyGJyQ5F86zK5OTifPPHMaSW2t4v_w1OELRUiiL_oOdXwYPNsOVpWxsQJ0Rl36GUwNxpVj5jthtKvp5M3WM8Qgzvgr0ZFvX9OhZmcNbpgnC7wBpy__9XTcBDxiRKNMo2boSYUx4o)

Apesar de funcionar, não parece correto termos que **manualmente rodar os testes.** Temos algumas opções do que fazer a seguir:

- Administrar o cenário onde o player não existe

- Administrar o cenário **POST /players/{name}** 


#### Escreva o teste primeiro

Vamos adicionar o cenário em que o **player não existe** 

**server\_test.go**

    t.Run("Retorna 404 em jogadores inexistentes", func(t *testing.T) {
           request := newGetScoreRequest("Apollo")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           got := response.Code
           want := http.StatusNotFound

<!---->

           if got != want {
               t.Errorf("Recebeu status %d quer status %d", got, want)
           }
       })

- rodando teste:

![](https://lh7-us.googleusercontent.com/DaZjkf_xn5QQrcb70ea84YTpZ_c5tB5eytouU2s2Q0_tYi2dlUO21vZwWjtw4NItkzmVzcDCxsDTgc1uXp66IZwRTcHn2K-pZvV3blmkfctAvcOd_qKqs443Xs6pUvfKK2DH1beRMtKci7KNKArwXr8)


#### Escreva código suficiente para fazer passar

**server.go**

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       w.WriteHeader(http.StatusNotFound)

<!---->

       // ATENÇÃO !!! Fprint != Fprintf
       fmt.Fprint(w, p.store.GetPlayerScore(player))
    }

**OBS =>** podemos ver que isso não parece correto, em TODAS as request vai retornar **not found.** mas todos os testes vão passar e é isso que importa!

![](https://lh7-us.googleusercontent.com/Zv3ZgVho3cbfJb0uBCrz7Ec61aqrgw1Wejf7u6NyikMl2xQbblI-QWQ1d4IhapA6-3uo4nW-elkslCNJ48EVPeJRRC0l-KGG1RkmfeMJ3qzaFrjMlsB-PyRKTUD9gPrB9vz0M0gBgQRbg4F6o20gqls)

- Vamos atualizar os outros tests para colocar o **status correto** e arrumar o código

**server\_test.go**

    func TestGETPlayers(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{
               "victor": 20,
               "cicero": 10,
           },
       }
       server := &PlayerServer{&store}

<!---->

       t.Run("Buscando o score do jogador Victor", func(t *testing.T) {
           request := newGetScoreRequest("victor")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusOK)
           assertResponseBody(t, response.Body.String(), "20")
       })

<!---->

       t.Run("Buscando o score do jogador Cícero", func(t *testing.T) {
           request := newGetScoreRequest("cicero")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusOK)
           assertResponseBody(t, response.Body.String(), "10")
       })

<!---->

       t.Run("Retorna 404 em jogadores inexistentes", func(t *testing.T) {
           request := newGetScoreRequest("Apollo")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusNotFound
    )
       })
    }

<!---->

    func assertStatus(t testing.TB, got, want int) {
       t.Helper()
       if got != want {
           t.Errorf("Não recebeu o status correto, recebeu %d mas queria %d", got, want)
       }
    }

********Agora estamos checando o status em todos nosso códigos e criamos um helper **assertStatus** para facilitar isso

- testando: 

![](https://lh7-us.googleusercontent.com/rMdWlcQ5sRxu6Ee5wPnnJZSJCS2S2y_SBGhmmIIWi8kNa76-KSu-8IT7c8keO0JLAm_csaaVAQH-jo1Y9w-HXtq2EKdE8HC7d7r5lIT56jMUrenqyCyYsGP4W0Lvt10uwVS8NHL1oONQnvporW9ERww)

- Agora nosso primeiros dois testes falharam por causa do 404 ao invés de 200, dessa forma podemos arrumar o **PlayerServer** para apenas returnar **not found** se o **score for 0**

**server.go**

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       score := p.store.GetPlayerScore(player)

<!---->

       if score == 0 {
           w.WriteHeader(http.StatusNotFound)
       }

<!---->

       // ATENÇÃO !!! Fprint != Fprintf
       fmt.Fprint(w, score)
    }

Agora que podemos retornar valores de um store faz sentido ser capaz de armazenar novos valores


#### Escrever os testes primeiro

**server\_test.go**

    func TestStoreWins(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{},
       }
       server := &PlayerServer{&store}

<!---->

       t.Run("Retorna aceito em POST", func(t *testing.T) {
           request, _ := http.NewRequest(http.MethodPost, "/players/victor", nil)
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusAccepted)
       })
      
    }

- rodando o teste:

![](https://lh7-us.googleusercontent.com/AbzgpaV86XWlA5OUPOMT-BU7AwtAgNhzxD57a0knnvLH8Bxlt8-9T-8NtH155lUZIn95hEf4sidGSiAF4rVh7LxWdij9TD1kVzPWp91gc7GqbEgTS2EI9lHmiuaTdp2tWq7cn7izXfUvoE8sS7jc1oI)


#### Escreva código o suficiente para passar

Vamos cometer um pecado, só um **if** para selecionar corretamente o método da request

**server.go**

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       if r.Method == http.MethodPost {
           w.WriteHeader(http.StatusAccepted)
           return
       }
      
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       score := p.store.GetPlayerScore(player)

<!---->

       if score == 0 {
           w.WriteHeader(http.StatusNotFound)
       }

<!---->

       // ATENÇÃO !!! Fprint != Fprintf
       fmt.Fprint(w, score)
    }


#### Refatorar

Nosso handler não está muito bom. Vamos quebrar o código para deixar mais fácil de seguir e isolar em funcionalidades diferentes em novas funções

**server.go**

    type PlayerStore interface {
       GetPlayerScore(name string) int
    }

<!---->

    type PlayerServer struct {
       store PlayerStore
    }

<!---->

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       switch r.Method {
       case http.MethodPost:
           p.processWin(w)
       case http.MethodGet:
           p.showScore(w, r)
       }  
    }

<!---->

    func (p *PlayerServer) showScore(w http.ResponseWriter, r *http.Request) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       score := p.store.GetPlayerScore(player)

<!---->

       if score == 0 {
           w.WriteHeader(http.StatusNotFound)
       }

<!---->

       // ATENÇÃO !!! Fprint != Fprintf
       fmt.Fprint(w, score)
    }

<!---->

    func (p *PlayerServer) processWin(w http.ResponseWriter) {
       w.WriteHeader(http.StatusAccepted)
    }

Isso faz com que o aspecto do **ServeHTTP** um pouco mais claro e significa que a nossa próxima iteração de armazenar pode ser dentro do **processWin**

Nós queremos checar que quando nós fazemos nosso **POST /players/{name}** e o nosso **PlayerStore** é chamado para armazenar a vitória


#### Escreva o teste primeiro 

Vamos conseguir isso ao estender nosso **StubPlayerStore** com um novo método **RecordWin** e depois _espiar_ nas “invocações” ****

**server\_test.go**

    type StubPlayerStore struct {
       scores map[string]int
       winCalls []string
    }

<!---->

    func (s *StubPlayerStore) GetPlayerScore(name string) int {
       score := s.scores[name]
       return score
    }

<!---->

    func (s *StubPlayerStore) RecordWin(name string) {
       s.winCalls = append(s.winCalls, name)
    }

Agora estendemos nossos testes para checar o número de invocações de começo

**server\_test.go**

    func TestStoreWins(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{},
       }
       server := &PlayerServer{&store}

<!---->

       t.Run("Armazena as vitórias em POST", func(t *testing.T) {
           request := newPostWinRequest("victor")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusAccepted)

<!---->

           if len(store.winCalls) != 1 {
               t.Errorf("Recebeu %d chamadas para RecordWin e quer %d", len(store.winCalls), 1)
           }
       })

<!---->

    }

<!---->

    func newPostWinRequest(name string) *http.Request {
       req, _ := http.NewRequest(http.MethodPost, fmt.Sprintf("/players/%s", name), nil)
       return req
    }

- Vamos rodar o teste:

![](https://lh7-us.googleusercontent.com/QBLyw5ErZ3zY6bGvSYI2q_7XsrXMQ87u6qGi5YL46n53QrYxEXC38abac6JIiB5eZyptg2hiRh2-Z-sRE7b0NivPu3DCHXTMouOIzjpvb11q3aDWuBWMruVpd72CbVekvI03JviREPBY4NpPQSOUWtY)


#### Escreva o mínimo de código

Na chamada a nossa struct estamos passando pouquíssimos parâmetros, vamos passar mais um parâmetro em **StubPlayerStore**

    func TestStoreWins(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{},
           nil, ----------> MUDANÇA
       }
       server := &PlayerServer{&store}

- Testando:

![](https://lh7-us.googleusercontent.com/CMabWRSEYq7EL6Yo3zomJdSBtPbM7ckOky3XA_r6gWVhjJiciQ6TLFSFEPsSS4mXY3v9bk7wKjNCSc2Zvt5NwsWpFxhAxYEF27YyyIhX8w0rror134SNTsv9vvG0wEPSKdZJYAFT0Ainv8jiZ_BBu-I)


#### Escrevendo o mínimo de código para passar

Precisamos atualizar a ideia do **PlayerServer** sobre o que é um **PlayerStore**, alterando a interface, se quisermos ser capazes de chamar **RecordWin**

**server.go**

    type PlayerStore interface {
       GetPlayerScore(name string) int
       RecordWin(name string)
    }

- Ao fazer isso a main não compila mais

![](https://lh7-us.googleusercontent.com/2FxWt_4bIuOAjC6WFXiqiBEm0Aj9ax9I-gOrg7PONm90YLSqosBs239KGNvY58rvVINF7MsNdxh2trINGy3hasRwCZIoDf7HduSwn7v62MbnvcCeV_tq1Icv2IJQhZxD-iu9or11dMNevBuBg9q0GnU)

- Vamos atualizar o **InMemoryPlayerStore** para ter esse método

**main.go**

    type InMemoryPlayerStore struct{}

<!---->

    func (i *InMemoryPlayerStore) RecordWin(name string) {}

- se tentarmos compilar ainda vai dar erro

* Agora que o **PlayStore** tem **RecordWin** podemos chamar isso dentro do nosso **PlayerServer**

**server.go**

    func (p *PlayerServer) processWin(w http.ResponseWriter) {
       p.store.RecordWin("cicero")
       w.WriteHeader(http.StatusAccepted)
    }

Rode os testes e agora eles devem passar.

![](https://lh7-us.googleusercontent.com/VVK74O0uuGht9b6Tw3cUeORsSPK-mDGQGISaTkj89TO5jh5tclT0UQjwa0KhTmUmM0cZAVcdAABmw-OVuBkrMVnTvkuLU5EsBJOCTof5bcwWDL71IPtEodX7jGZoZ8Rw6_vyYxkXylf33dlRft307xc)

cicero não é exatamente o que queremos enviar ao **RecordWin,** então vamos refinar nosso teste


#### Escrever o teste primeiro

Vamos mexer no **TestStoreWins** para melhorar o escopo de testes 

**server\_test.go**

    func TestStoreWins(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{},
           nil,
       }
       server := &PlayerServer{&store}

<!---->

       t.Run("Armazena as vitórias em POST", func(t *testing.T) {
           player := "victor"

<!---->

           request := newPostWinRequest(player)
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusAccepted)

<!---->

           if len(store.winCalls) != 1 {
               t.Fatalf("Recebeu %d chamadas para RecordWin e quer %d", len(store.winCalls), 1)
           }

<!---->

           if store.winCalls[0] != player {
               t.Errorf("Não armazenou corretamente o vencedor, recebeu %q mas queria %q", store.winCalls[0], player)
           }
       })

<!---->

    }

Agora que sabemos que existe um elemento no nosso slice **winCalls** podemos de forma segura referenciar o primeiro a checar se é igual ao **player**

- Vamos testar

![](https://lh7-us.googleusercontent.com/11z-n-jC4O0wd3d927WrIsqOB3n1aF6-0A2Vj-Vul_qvX2gQUTBguBSWziDX2IuKL0UCmv1GIjpxesLxqcB8mfi7v9_FoAXA4gvAAmsAtOITugM4FngRZ_mRwF9pUWhmvr9H3aQ0Cgu9uALEKSRH4uA)


#### Escreva código o suficiente para passar 

    func (p *PlayerServer) processWin(w http.ResponseWriter, r *http.Request) {
       player := strings.TrimPrefix(r.URL.Path, "/players/")
       p.store.RecordWin(player)
       w.WriteHeader(http.StatusAccepted)
    }

Mudamos o **processWin** para pegar o **http.Request** para podermos ver a URL e extrair o nome do players. Uma vez que tivermos o nome, podemos chamar nosso **store** com um valor correto para fazer o teste passar


#### Refatorar

Podemos deixar esse código mais objetivo e extrair o nome do player da mesma forma em dois lugares

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       player := strings.TrimPrefix(r.URL.Path, "/players/")
      
       switch r.Method {
       case http.MethodPost:
           p.processWin(w, player)
       case http.MethodGet:
           p.showScore(w, player)
       }
    }

<!---->

    func (p *PlayerServer) showScore(w http.ResponseWriter, player string) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       score := p.store.GetPlayerScore(player)

<!---->

       if score == 0 {
           w.WriteHeader(http.StatusNotFound)
       }

<!---->

       // ATENÇÃO !!! Fprint != Fprintf
       fmt.Fprint(w, score)
    }

<!---->

    func (p *PlayerServer) processWin(w http.ResponseWriter, player string) {
       p.store.RecordWin(player)
       w.WriteHeader(http.StatusAccepted)
    }

Por mais que os testes estão passando, não temos software funcional, pois não implementamos o **PlayerStore** corretamente. Mas está tudo bem, nós identificamos a interface que queriamos, ao invés de tentar desenhar tudo de forma antecipada

Podiamos melhorar os testes do **InMemoryPlayerStore** só que não compensa fazer isso pois precisamos implementar um banco de dados depois

O que vamos fazer agora é escrever um _teste de integração_ entre o nosso **PlayerServer** e o **InMemoryPlayerStore** para terminar a funcionalidade. Isso vai nos permitir chegar no nosso objetivo de estar confiantes que a nossa aplicação está funcionando, sem precisar ter que diretamente testar o **InMemoryPlayerStore.** Não só isso, mas quando nós pegarmos e implementamos o **PlayerStore** com um banco de dados, nós podemos testar a implementação o mesmo _integration test_


### Teste de Integração ( integration test )

Testes de integração podem ser usados para áreas maiores do seu sistema de trabalho, mas você deve ter em mente:

- São mais difíceis de escrever

- Quando falham, pode ser difícil saber o porque ( geralmente é um bug com um componente do teste de integração ) e depois pode ser mais difícil de arrumar

- Às vezes são mais lentos de rodar ( pois geralmente são usados com componentes “reais”, como um **banco de dados** )


#### Escrever o teste primeiro

**server\_integration\_test.go**

    package main

<!---->

    import (
       "net/http"
       "net/http/httptest"
       "testing"
    )

<!---->

    func TestRecordingWinsAndRetrievingThen(t *testing.T) {
       store := InMemoryPlayerStore{}
       server := PlayerServer{&store}
       player := "victor"

<!---->

       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))

<!---->

       response := httptest.NewRecorder()
       server.ServeHTTP(response, newGetScoreRequest(player))
       assertStatus(t, response.Code, http.StatusOK)

<!---->

       assertResponseBody(t, response.Body.String(), "3")
    }

- Estamos criando dois componente ( instâncias da struct ) e estamos tentando integrar eles com: **InMemoryPlayerStore** e **PlayerServer**

- Depois disso disparamos 3 requisições para armazenar 3 vitórias para o **player.** Não estamos tão preocupados sobre o status dos códigos nesses testes já que isso não é tão relevante se os testes estão integrando bem

- A próxima response que nos importamos (por isso armazenamos uma variável **response**) pois nós vamos tentar pegar o score do **player**

* Vamos tentar rodar o teste

![](https://lh7-us.googleusercontent.com/LTNl5_kxmiWcQTWggsXwcyTJE3BxJTN-aEKxmf0LifVHo9JQ-EIAkOfEBtwx_-DxE4Qfy8GjE4wWTOsoDH8lHisy1gnJgLsbIRj3w7GD1AP5O-hPyPAxgIqKoKMZODWkeRsNwQfj8D7A5IBvAT7G4Rc)


#### Escrevendo código suficiente para passar

Aqui vamos escrever mais testes que fizemos anteriormente. Caso fiquemos travados nesse ponto, precisamos reverter as mudanças de volta para o teste que falhou e depois escrever um unit teste mais específico ao redor do **InMemoryPlayerStore** para ajudar a administrar a solução

**in\_memory\_player\_store.go**

    type InMemoryPlayerStore struct {
       store map[string]int
    }

<!---->

    func NewInMemoryPlayerStore() *InMemoryPlayerStore {
       return &InMemoryPlayerStore{map[string]int{}}
    }

<!---->

    func (i *InMemoryPlayerStore) RecordWin(name string) {
       i.store[name]++
    }

<!---->

    func (i *InMemoryPlayerStore) GetPlayerScore(name string) int {
       return i.store[name]
    }

- Precisamos armazenar os dados então adicionei um **map\[string]int** a struct **InMemoryPlayerStore** 

- Por conveniência fiz **NewInMemoryPlayerStore** para inicializar um store, e atualizar o teste de integração para usar ele:

**server\_integration\_test.go**

    func TestRecordingWinsAndRetrievingThen(t *testing.T) {
       store := NewInMemoryPlayerStore() ------------------> MUDANÇA
       server := PlayerServer{store} ------------------> MUDANÇA
       player := "victor"

<!---->

       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))

<!---->

       response := httptest.NewRecorder()
       server.ServeHTTP(response, newGetScoreRequest(player))
       assertStatus(t, response.Code, http.StatusOK)

<!---->

       assertResponseBody(t, response.Body.String(), "3")
    }

- O resto do código está só encapsulando o **map**

* Vamos testar:

![](https://lh7-us.googleusercontent.com/1TUvQyrmzBWCROvUVYrWC-ql5bdVTmISXaCNfuPtPMLa2164J9Yot2IEKzLVhtbBH3rACSgy04Oaj2nWEKsqG1rBHniGnMQsrrqkpeD9CIBLidq22Rvhy_bbI4XtffxjbHtszoQ6nlnHW_iZi9fIUFQ)

O teste de integração passou, agora vamos mudar a **main** para usar **NewInMemoryPlayerStore()**

**main.go**

    func main() {
       server := &PlayerServer{NewInMemoryPlayerStore()}
       log.Fatal(http.ListenAndServe(":5000", server)) // se tiver algum erro ao servir nosso servidor, o '1og' vai reportar o erro e vai terminar o programa
    }

- Vamos buildar e rodar

* podemos testar algumas vezes com os nomes que quisermos

![](https://lh7-us.googleusercontent.com/2wJZ_VK7nuQQOdmvrGb6nE6SU2419JfLKeGPumb57ia8vCho7UGP_Xr3lOjh4V3mikOYeapgmn6s6cMfGlFi-cNa8i5fNYe7fjSLbzXBB73cVSuBVed4e8DtXBG8JMpac7tkUohBfKeB3hnlJ0mgbuY)

\


<https://quii.gitbook.io/learn-go-with-tests/build-an-application/http-server>

Vamos criar um server HTTP para o seguinte:

**GET /players/{name}** deve retornar um número indicando o total de vitórias

**POST /players/{name}** deve retornar um registro daquele nome, incrementando para cada POST subsequente

- Vamos seguir a abordagem **TDD,** ter um software funcionando o mais rápido que pudermos e então fazermos melhorias pequenas até que cheguemos à solução. Com essa abordagem vamos:

1. Manter o espaço de problema pequeno a cada momento

2. Não vamos cair em rabbit holes

3. Caso fiquemos perdidos ou travados, fazer um revert não vai perder muito trabalho


## Red, green, refactor

O processo de TDD consiste em:

1. Escrever um teste e ver ele falhar ( red )

2. Escrever o mínimo de código possível para fazer passar no teste ( green )

3. Refatorar ( melhorar o código )

**A meta é sair do red o mais rápido possível**

Assim que o teste passar **você pode commitar as mudanças** 


### E se eu não fizer isso?

Quanto mais código você adicionar enquanto estiver em red, mais você vai adicionar problemas que não estão cobertos pelos testes

A ideia é escrever código útil em passos pequenos, dirigidos por testes para que **não caia em rabbit holes durante horas**


#### Galinha e ovo

Como podemos fazer isso? Não podemos fazer um **GET** nos users sem armazenar algo, e é complicado dizer se o **POST** funciona se não conseguimos averiguar com o **GET**

É ai que existe o **mocking:**

- O **GET** vai precisar de um **PlayerStore** para pegar as pontuações de um jogador. Isso deve ser uma **interface** para quando testarmos vamos poder criar um um esboço simples para testar nosso código sem precisar ter implementado nenhum código de storage ( banco de dados )

- Para o **POST** podemos “espiar” nas chamadas em **PlayerStore** para garantir que armazenou os jogadores corretamente. Nossa implementação de salvar não vai estar acoplada para recuperar nada

- Para ter um software que funcione rapidamente podemos fazer uma implementação e de muito simples **na memória** e depois podemos criar uma implementação com storage se quisermos


### Escreva o teste primeiro

Podemos escrever um teste para passar ao retornar um valor hard coded para começarmos. Depois que tivermos um teste funcionando podemos escrever mais testes 

- Para criarmos um web server em Go geralmente vamos chamar o **ListenAndServe**

![](https://lh7-us.googleusercontent.com/pI-14NP-RJ2OtNU2o3ZH4M2G1JJDrMpG2tWyBWtMTZbtPqjUeJoMgbQ17Afe5-3WO2sQfw2fv-cRh6thhlLtV6esCxEdpT0Otw6It8SrSLLFHqHTVPEImxIKkGeRifUXZ0FrTIHxCpxYLACAV_aaXsM)

Isso vai iniciar um web server que vai escutar em uma porta, criando uma **goroutine** para **cada requisição** e rodando isso contra um **Handler**

|                                                                         |
| ----------------------------------------------------------------------- |
| **type Handler interface { ****ServeHTTP(ResponseWriter, \*Request) }** |

********Essa interface implementa um Handler ao implementar o método **ServeHTTP** que espera **2 argumentos.** No primeiro escrever a **resposta** e o segundo escrevemos a **request HTTP** que foi enviada ao nosso servidor

- Vamos criar um arquivo **server\_test.go** e escrevemos um teste para a função **PlayerServer** que espera **dois argumentos.** A **request enviada** vai pegar a pontuação do jogador, que **esperamos que seja 20**

<!---->

    import (
       "net/http"
       "net/http/httptest"
       "testing"
    )

<!---->

    func TestGETPlayers(t *testing.T) {
       t.Run("Buscando o score do jogador Victor", func(t *testing.T) {
           request, _ := http.NewRequest(http.MethodGet, "players/victor", nil)
           response := httptest.NewRecorder()

<!---->

           PlayerServer(response, request)

<!---->

           got := response.Body.String()
           want := "20"

<!---->

           if got != want {
               t.Errorf("Recebido %q esperava %q", got, want)
           }
       })
    }

  ****

********Para testarmos nosso servidor precisamos de uma **Request** para enviar e então vamos **espiar** o que o nosso handler escreveu no **ResponseWriter**

- Usamos **http.NewRequest** para criar a request. O primeiro argumento é o **método da request** e o segundo parâmetro é o **path da requisição.** O argumento **nil** se refere ao **body da requisição.** no qual não precisamos nesse caso

- **net/http/httptest** tem um “espião” já feito para nós chamado **ResponseRecorder** e então podemos usar ele. Vai ser muito útil para inspecionar dentro de métodos e ver o que tem escrito dentro das response


#### Rodando o teste

![](https://lh7-us.googleusercontent.com/nbmm4hS9jZXU_id-wqQc2vhW0DpBQ3FXCVjr0Qb6_1BcRIOqYZBhCb-g94HsTE_GYmW0A_EQhBetodcCqYVHS0TVxCVVhlEWpv0rGbO40Au2ztpOmaUVEr2hLY306hogNi2YAuFJO3aDQOENuxmm2Dk)


#### 1 - Escrevendo o mínimo de código para o teste rodar

Vamos deixar o **compilador nos ajudar,** precisamos escutar ele

- Vamos criar um arquivo **server.go** e definir um **PlayerServer**

|                            |
| -------------------------- |
| **func PlayerServer() {}** |

********Testando novamente:

![](https://lh7-us.googleusercontent.com/mF-QuqYhCZ6XbX6NbKnlIqG9KRzn6DvnUGcwJRcr_7zLQp9fghWdKgZVGoMTiGvQJTFZ3rprEs3bHkx5NaanT06vNSOZidpmuWwb-n0UACLhHGsAabD_zlhBgcPbb9C428V9YD3Vl40OlyTZG6Yoeus)

- Vamos adicionar **argumentos para a nossa função**

<!---->

    package main

<!---->

    import "net/http"

<!---->

    func PlayerServer(w http.ResponseWriter, r *http.Request) {

<!---->

    }

- Vamos testar novamente

![](https://lh7-us.googleusercontent.com/xzA46FSVkoXU1Nqo51PFABpyssGubY2uWkYE-U43b-1R3T8q6mJ0eCLBqn1Sy3KFE1eDBxJKTkXgTzcSyGLNKv5QYHQm5rJWjJCulrwE5rSZt_0YpqQnW9CFB1hIZLloHSmONezWtoJk-DMUOb0WMQ4)


#### 2 - Escrevendo o mínimo de código para passar

O método **ResponseWriter** do pacote **net/http** também implementa o io **Writer,** dessa forma podemos usar o **fmt.Fprint** para enviar strings como respostas HTTP

    import (
       "fmt"
       "net/http"
    )

<!---->

    func PlayerServer(w http.ResponseWriter, r *http.Request) {
       fmt.Fprint(w, "20")
    }

O teste agora deve funcionar

![](https://lh7-us.googleusercontent.com/lKRdA_-mTO9t7dYgXrICftYMb0ApOQcNLyobp1sA7Im6anuYHhm4n-ecKtS4kdNB8yQxlz6HTCOgAnuL2QKJmaXw9-jjuGWxw-ss8fi5cD5JKRBL5iQ5WnzaIeanwOomb0mbzqVeJDxiUTtog2t59YY)

:) 


#### 3 - Completando a estrutura

Vamos inicializar nossa API agora em main.go. Precisamos escrever de uma forma que podemos modificar o código depois

**main.go:**

    import (
       "log"
       "net/http"
    )

<!---->

    func main() {
       handler := http.HandlerFunc(PlayerServer)
       log.Fatal(http.ListenAndServe(":5000", handler))
    }

- Vamos gerar o binário e rodar isso:

![](https://lh7-us.googleusercontent.com/VELY1j7mK0N9uw3GPJ48jg89oXA3Sb5iq0kJSj5S60dWkyXxGyqs6icH5RNbLcOret-hS5yJRN3MT5N5-C8BCMsUAptlWCRu90ldJoqEe19z5TZ5gWJLPxEcSyj7XqNh6YQDbgdt6wk3lZoqA--kS0Y)


### http.HandlerFunc

Anteriormente falamos que a interface **Handler** é necessário implementar em ordem para fazer um servidor. _Tipicamente_ o que fazemos é criar uma **struct** e depois fazer isso implementar uma interface e fazer o próprio método **ServeHTTP.** Apesar disso, o use-case para structs é armazenar dados, mas _atualmente_ **não temos nenhum estado, então não parece razoável criar uma**

********O **HandlerFunc** ajuda a gente a evitar isso:

|                                                                                                                                                                                         |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **O tipo HandlerFunc é um adaptador que permite o uso de funções ordinárias como handlers HTTP. Se f é uma função com uma assinatura correta, HandlerFunc(f) é um Handler que chama f** |

    type HandlerFunc func(http.ResponseWriter, *http.Request)

<!---->

    func main() {
       handler := http.HandlerFunc(PlayerServer)
       log.Fatal(http.ListenAndServe(":5000", handler))
    }


### Obrigando a remover valores hard-coded

********Vamos escrever outro teste para **forçar** que a gente faça uma mudança positiva para tentar **remover os valores hard-coded**


#### Escrevendo o teste primeiro

Vamos adicionar outro subteste a nossa bateria de testes, que vai tentar pegar o score de cada jogador diferente, que vai quebrar nossa **iniciativa hard-coded.** 

    t.Run("Buscando o score do jogador Cícero", func(t *testing.T) {
           request, _ := http.NewRequest(http.MethodGet, "players/cicero", nil)
           response := httptest.NewRecorder()

<!---->

           PlayerServer(response, request)

<!---->

           got := response.Body.String()
           want := "10"

<!---->

           if got != want {
               t.Errorf("Recebido %q esperava %q", got, want)
           }
       })


#### Rodando o teste

![](https://lh7-us.googleusercontent.com/51VZlwhsftwN_rtDGI8Ej5yAS7aSFsUHlS89qkcZUrk8arWSSBU_SvnZVnIgb5LyyJ7QgG3UAj_TsNckQRiOdApeIn4ovjP5_VHK5ZunERYbOwVwz2iI9eHgUs0FGd3cqlGjqOGszp81fFuVSVMW4is)


#### 1- Escrevendo o mínimo de código pra passar

**serve.go**

    func PlayerServer(w http.ResponseWriter, r *http.Request) {
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       if player == "victor" {
           fmt.Fprintf(w, "20")
           return
       }

<!---->

       if player == "cicero" {
           fmt.Fprintf(w, "10")
           return
       }
    }

Se tivéssemos começado com algum storage, teriamos que fazer mudanças enormes em comparação com essa abordagem. **Usando TDD podemos fazer passos pequenos no sentido do nosso objetivo**

********Vamos evitar qualquer biblioteca de routing nesse momento para fazermos nosso teste passar usando **passos pequenos**

**r.URL.Path** retorna o path da request na qual podemos depois usar o **strings.TrimPrefix** para “apararmos / removermos” o **/players/** e pegar **o player desejado**


#### Refatorando

Podemos simplificar o **PlayerServer** ao separar o score **retornado** em uma função

**server.go**

    func PlayerServer(w http.ResponseWriter, r *http.Request) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       fmt.Fprintf(w, GetPlayerScore(player))
    }

<!---->

    func GetPlayerScore(name string) string {
       if name == "victor" {
           return "20"
       }

<!---->

       if name == "cicero" {
           return "10"
       }

<!---->

       return ""
    }

- Podemos diminuir um pouco do nosso código nos testes ao **criar uns helpers:**

**server\_test.go**

    package main

<!---->

    import (
       "fmt"
       "net/http"
       "net/http/httptest"
       "testing"
    )

<!---->

    func TestGETPlayers(t *testing.T) {
       t.Run("Buscando o score do jogador Victor", func(t *testing.T) {
           request := newGetScoreRequest("victor")
           response := httptest.NewRecorder()

<!---->

           PlayerServer(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "20")
       })

<!---->

       t.Run("Buscando o score do jogador Cícero", func(t *testing.T) {
           request := newGetScoreRequest("cicero")
           response := httptest.NewRecorder()

<!---->

           PlayerServer(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "10")
       })
    }

<!---->

    func newGetScoreRequest(name string) *http.Request {
       req, _ := http.NewRequest(http.MethodGet, fmt.Sprintf("/players/%s", name), nil)
       return req
    }

<!---->

    func assertResponseBody(t testing.TB, got, want string) {
       t.Helper()
       if got != want {
           t.Errorf("O response body está errado, got %q want %q", got, want)
       }
    }

Apesar disso não podemos ficar tão feliz. **Não é correto o servidor saber os scores**

Movemos o cálculo do score da nossa função fora do body da função main do nosso handler dentro de uma função **GetPlayerScore.** Isso parece como o lugar correto para separar preocupações usando interfaces

- Vamos mover a função que refatoramos para ser uma **interface**

**server.go**

    type PlayerStore interface {
       GetPlayerScore(name string) int
    }

Para o nosso **PlayerServer** conseguir usar o **PLayerStore,** vamos precisar referenciar um. Agora parece o tempo correto para mudar nossa arquitetura para que o **PlayerServer** seja uma **struct** 

    type PlayerServer struct {
       store PlayerStore
    }

   ****

- Agora vamos implementar a interface **Handler** ao adicionar o método na nossa struct e colocar isso em um código handler existente. 

<!---->

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       fmt.Fprint(w, p.store.GetPlayerScore(player))
    }


#### Arrumar os erros

Fizemos muitas mudanças e o código não vai compilar, mas vamos testar e deixar o compilador nos guiar

![](https://lh7-us.googleusercontent.com/9Dwrb1u21GDoUwplQmBS2g8KEKcHz9H0f5_ItUK-ZBJxanoK9Q6mhksGo0EuTvPPfn4F192h3hMr0Bh9JHLsZhFueq5cGulPGlHdUvOwYYr9FFV-i0A7IOLopv4Adbc9DK5VflhMRNuLv31pgNbv9j4)

|                                                                 |
| --------------------------------------------------------------- |
| ./main.**go**:11:30: **type** PlayerServer is not an expression |

- Precisamos mudar nossos testes para **criar uma nova instância** do nosso **PlayerServer** e depois chamar o método **ServeHTTP** 

<!---->

    func TestGETPlayers(t *testing.T) {
       server := &PlayerServer{}

<!---->

       t.Run("Buscando o score do jogador Victor", func(t *testing.T) {
           request := newGetScoreRequest("victor")
           response := httptest.NewRecorder()

      // MUDANÇA
           server.ServeHTTP(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "20")
       })

<!---->

       t.Run("Buscando o score do jogador Cícero", func(t *testing.T) {
           request := newGetScoreRequest("cicero")
           response := httptest.NewRecorder()

<!---->

     // MUDANÇA
           server.ServeHTTP(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "10")
       })
    }

**AINDA NÃO ESTAMOS NOS PREOCUPANDO COM STORES.** Só queremos fazer o compiler passar o mais rápido possível

**Precisamos ter o hábito de priorizar ter código que compile e depois código que passe os testes**

- Agora nosso **main.go** não vai compilar pela mesma razão

<!---->

    func main() {
       server := &PlayerServer{}
       log.Fatal(http.ListenAndServe(":5000", server)) // se tiver algum erro ao servir nosso servidor, o '1og' vai reportar o erro e vai terminar o programa
    }

teste:

![](https://lh7-us.googleusercontent.com/NQxIWHzqTLE028mL54el8LwbhRx3iR-wtK-Dj9shVD6lO1aUTABX7Ft2Us351stjEksnqbPQNeRJRiTDdhmb6I750Dnkl7WnYOoUHXYbFOklpdQz3MV0uTSzUyp2hmVE7CnTJc4DMm55-lXWChLnmyg)

Está compilando mas os testes estão falhando.

- Isso aconteceu pois não passamos um **PlayerStore** nos nossos testes. Precisamos fazer uma estrutura **stub** ( esboço ) de uma

**server\_test.go**

    type StubPlayerStore struct {
       scores map[string]int
    }

<!---->

    func (s *StubPlayerStore) GetPlayerScore(name string) int {
       score := s.scores[name]
       return score
    }

Um **map** é uma boa estrutura para fazer um **stub key/value** para os nossos testes. Agora vamos criar um desses stores para os nossos testes e enviar para o nosso **PlayerServer**

**server\_test.go**

    func TestGETPlayers(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{
               "victor": 20,
               "cicero": 10,
           },
       }
       server := &PlayerServer{&store}

<!---->

       t.Run("Buscando o score do jogador Victor", func(t *testing.T) {
           request := newGetScoreRequest("victor")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "20")
       })

<!---->

       t.Run("Buscando o score do jogador Cícero", func(t *testing.T) {
           request := newGetScoreRequest("cicero")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertResponseBody(t, response.Body.String(), "10")
       })
    }

- Agora nossos testes devem passar e já estão um pouco mais elaborados. A intenção atrás do nosso código é 

![](https://lh7-us.googleusercontent.com/YmmdTFjLvrNcOpDT9fnzRMkrH6A0I7p0uBSYMdF_qw62nL_FE3_GaiYcP0iGfTP_0JicaUTNju3re7MIEEgPFRLTW9Mde3aVL7PDGAJaYZsy4wVubtFEkHtoSZRz_toAAZbSwYf9bd949SeHRZohDkQ)


#### Passando PlayerStore para o server

Vamos passar um **PLayerStore** simples para passar ao server, já que fazer um storage real agora vai ser um pouco complicado.

![](https://lh7-us.googleusercontent.com/6afqprP9Kfuji3giT4nga2XjsKDafv0QBFbj1v1VZvmYDmOm8FRWdGkAA3RlVxRmIO4fR4h9o-rSKDvSk2eqr5KXwFV12xao3duHTYX1yXR3fNgi8IMQ2cxXRCKOM2manNjQRvkXyHukZXlRufBt_lQ)

- Se fizermos **go build** e fomos na url [**http://localhost:5000/players/victor**](http://localhost:5000/players/victor) ****vai ser retornado 123

![](https://lh7-us.googleusercontent.com/kKvdz0EEOfby4OloyGJyQ5F86zK5OTifPPHMaSW2t4v_w1OELRUiiL_oOdXwYPNsOVpWxsQJ0Rl36GUwNxpVj5jthtKvp5M3WM8Qgzvgr0ZFvX9OhZmcNbpgnC7wBpy__9XTcBDxiRKNMo2boSYUx4o)

Apesar de funcionar, não parece correto termos que **manualmente rodar os testes.** Temos algumas opções do que fazer a seguir:

- Administrar o cenário onde o player não existe

- Administrar o cenário **POST /players/{name}** 


#### Escreva o teste primeiro

Vamos adicionar o cenário em que o **player não existe** 

**server\_test.go**

    t.Run("Retorna 404 em jogadores inexistentes", func(t *testing.T) {
           request := newGetScoreRequest("Apollo")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           got := response.Code
           want := http.StatusNotFound

<!---->

           if got != want {
               t.Errorf("Recebeu status %d quer status %d", got, want)
           }
       })

- rodando teste:

![](https://lh7-us.googleusercontent.com/DaZjkf_xn5QQrcb70ea84YTpZ_c5tB5eytouU2s2Q0_tYi2dlUO21vZwWjtw4NItkzmVzcDCxsDTgc1uXp66IZwRTcHn2K-pZvV3blmkfctAvcOd_qKqs443Xs6pUvfKK2DH1beRMtKci7KNKArwXr8)


#### Escreva código suficiente para fazer passar

**server.go**

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       w.WriteHeader(http.StatusNotFound)

<!---->

       // ATENÇÃO !!! Fprint != Fprintf
       fmt.Fprint(w, p.store.GetPlayerScore(player))
    }

**OBS =>** podemos ver que isso não parece correto, em TODAS as request vai retornar **not found.** mas todos os testes vão passar e é isso que importa!

![](https://lh7-us.googleusercontent.com/Zv3ZgVho3cbfJb0uBCrz7Ec61aqrgw1Wejf7u6NyikMl2xQbblI-QWQ1d4IhapA6-3uo4nW-elkslCNJ48EVPeJRRC0l-KGG1RkmfeMJ3qzaFrjMlsB-PyRKTUD9gPrB9vz0M0gBgQRbg4F6o20gqls)

- Vamos atualizar os outros tests para colocar o **status correto** e arrumar o código

**server\_test.go**

    func TestGETPlayers(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{
               "victor": 20,
               "cicero": 10,
           },
       }
       server := &PlayerServer{&store}

<!---->

       t.Run("Buscando o score do jogador Victor", func(t *testing.T) {
           request := newGetScoreRequest("victor")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusOK)
           assertResponseBody(t, response.Body.String(), "20")
       })

<!---->

       t.Run("Buscando o score do jogador Cícero", func(t *testing.T) {
           request := newGetScoreRequest("cicero")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusOK)
           assertResponseBody(t, response.Body.String(), "10")
       })

<!---->

       t.Run("Retorna 404 em jogadores inexistentes", func(t *testing.T) {
           request := newGetScoreRequest("Apollo")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusNotFound
    )
       })
    }

<!---->

    func assertStatus(t testing.TB, got, want int) {
       t.Helper()
       if got != want {
           t.Errorf("Não recebeu o status correto, recebeu %d mas queria %d", got, want)
       }
    }

********Agora estamos checando o status em todos nosso códigos e criamos um helper **assertStatus** para facilitar isso

- testando: 

![](https://lh7-us.googleusercontent.com/rMdWlcQ5sRxu6Ee5wPnnJZSJCS2S2y_SBGhmmIIWi8kNa76-KSu-8IT7c8keO0JLAm_csaaVAQH-jo1Y9w-HXtq2EKdE8HC7d7r5lIT56jMUrenqyCyYsGP4W0Lvt10uwVS8NHL1oONQnvporW9ERww)

- Agora nosso primeiros dois testes falharam por causa do 404 ao invés de 200, dessa forma podemos arrumar o **PlayerServer** para apenas returnar **not found** se o **score for 0**

**server.go**

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       score := p.store.GetPlayerScore(player)

<!---->

       if score == 0 {
           w.WriteHeader(http.StatusNotFound)
       }

<!---->

       // ATENÇÃO !!! Fprint != Fprintf
       fmt.Fprint(w, score)
    }

Agora que podemos retornar valores de um store faz sentido ser capaz de armazenar novos valores


#### Escrever os testes primeiro

**server\_test.go**

    func TestStoreWins(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{},
       }
       server := &PlayerServer{&store}

<!---->

       t.Run("Retorna aceito em POST", func(t *testing.T) {
           request, _ := http.NewRequest(http.MethodPost, "/players/victor", nil)
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusAccepted)
       })
      
    }

- rodando o teste:

![](https://lh7-us.googleusercontent.com/AbzgpaV86XWlA5OUPOMT-BU7AwtAgNhzxD57a0knnvLH8Bxlt8-9T-8NtH155lUZIn95hEf4sidGSiAF4rVh7LxWdij9TD1kVzPWp91gc7GqbEgTS2EI9lHmiuaTdp2tWq7cn7izXfUvoE8sS7jc1oI)


#### Escreva código o suficiente para passar

Vamos cometer um pecado, só um **if** para selecionar corretamente o método da request

**server.go**

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       if r.Method == http.MethodPost {
           w.WriteHeader(http.StatusAccepted)
           return
       }
      
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       score := p.store.GetPlayerScore(player)

<!---->

       if score == 0 {
           w.WriteHeader(http.StatusNotFound)
       }

<!---->

       // ATENÇÃO !!! Fprint != Fprintf
       fmt.Fprint(w, score)
    }


#### Refatorar

Nosso handler não está muito bom. Vamos quebrar o código para deixar mais fácil de seguir e isolar em funcionalidades diferentes em novas funções

**server.go**

    type PlayerStore interface {
       GetPlayerScore(name string) int
    }

<!---->

    type PlayerServer struct {
       store PlayerStore
    }

<!---->

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       switch r.Method {
       case http.MethodPost:
           p.processWin(w)
       case http.MethodGet:
           p.showScore(w, r)
       }  
    }

<!---->

    func (p *PlayerServer) showScore(w http.ResponseWriter, r *http.Request) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       player := strings.TrimPrefix(r.URL.Path, "/players/")

<!---->

       score := p.store.GetPlayerScore(player)

<!---->

       if score == 0 {
           w.WriteHeader(http.StatusNotFound)
       }

<!---->

       // ATENÇÃO !!! Fprint != Fprintf
       fmt.Fprint(w, score)
    }

<!---->

    func (p *PlayerServer) processWin(w http.ResponseWriter) {
       w.WriteHeader(http.StatusAccepted)
    }

Isso faz com que o aspecto do **ServeHTTP** um pouco mais claro e significa que a nossa próxima iteração de armazenar pode ser dentro do **processWin**

Nós queremos checar que quando nós fazemos nosso **POST /players/{name}** e o nosso **PlayerStore** é chamado para armazenar a vitória


#### Escreva o teste primeiro 

Vamos conseguir isso ao estender nosso **StubPlayerStore** com um novo método **RecordWin** e depois _espiar_ nas “invocações” ****

**server\_test.go**

    type StubPlayerStore struct {
       scores map[string]int
       winCalls []string
    }

<!---->

    func (s *StubPlayerStore) GetPlayerScore(name string) int {
       score := s.scores[name]
       return score
    }

<!---->

    func (s *StubPlayerStore) RecordWin(name string) {
       s.winCalls = append(s.winCalls, name)
    }

Agora estendemos nossos testes para checar o número de invocações de começo

**server\_test.go**

    func TestStoreWins(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{},
       }
       server := &PlayerServer{&store}

<!---->

       t.Run("Armazena as vitórias em POST", func(t *testing.T) {
           request := newPostWinRequest("victor")
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusAccepted)

<!---->

           if len(store.winCalls) != 1 {
               t.Errorf("Recebeu %d chamadas para RecordWin e quer %d", len(store.winCalls), 1)
           }
       })

<!---->

    }

<!---->

    func newPostWinRequest(name string) *http.Request {
       req, _ := http.NewRequest(http.MethodPost, fmt.Sprintf("/players/%s", name), nil)
       return req
    }

- Vamos rodar o teste:

![](https://lh7-us.googleusercontent.com/QBLyw5ErZ3zY6bGvSYI2q_7XsrXMQ87u6qGi5YL46n53QrYxEXC38abac6JIiB5eZyptg2hiRh2-Z-sRE7b0NivPu3DCHXTMouOIzjpvb11q3aDWuBWMruVpd72CbVekvI03JviREPBY4NpPQSOUWtY)


#### Escreva o mínimo de código

Na chamada a nossa struct estamos passando pouquíssimos parâmetros, vamos passar mais um parâmetro em **StubPlayerStore**

    func TestStoreWins(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{},
           nil, ----------> MUDANÇA
       }
       server := &PlayerServer{&store}

- Testando:

![](https://lh7-us.googleusercontent.com/CMabWRSEYq7EL6Yo3zomJdSBtPbM7ckOky3XA_r6gWVhjJiciQ6TLFSFEPsSS4mXY3v9bk7wKjNCSc2Zvt5NwsWpFxhAxYEF27YyyIhX8w0rror134SNTsv9vvG0wEPSKdZJYAFT0Ainv8jiZ_BBu-I)


#### Escrevendo o mínimo de código para passar

Precisamos atualizar a ideia do **PlayerServer** sobre o que é um **PlayerStore**, alterando a interface, se quisermos ser capazes de chamar **RecordWin**

**server.go**

    type PlayerStore interface {
       GetPlayerScore(name string) int
       RecordWin(name string)
    }

- Ao fazer isso a main não compila mais

![](https://lh7-us.googleusercontent.com/2FxWt_4bIuOAjC6WFXiqiBEm0Aj9ax9I-gOrg7PONm90YLSqosBs239KGNvY58rvVINF7MsNdxh2trINGy3hasRwCZIoDf7HduSwn7v62MbnvcCeV_tq1Icv2IJQhZxD-iu9or11dMNevBuBg9q0GnU)

- Vamos atualizar o **InMemoryPlayerStore** para ter esse método

**main.go**

    type InMemoryPlayerStore struct{}

<!---->

    func (i *InMemoryPlayerStore) RecordWin(name string) {}

- se tentarmos compilar ainda vai dar erro

* Agora que o **PlayStore** tem **RecordWin** podemos chamar isso dentro do nosso **PlayerServer**

**server.go**

    func (p *PlayerServer) processWin(w http.ResponseWriter) {
       p.store.RecordWin("cicero")
       w.WriteHeader(http.StatusAccepted)
    }

Rode os testes e agora eles devem passar.

![](https://lh7-us.googleusercontent.com/VVK74O0uuGht9b6Tw3cUeORsSPK-mDGQGISaTkj89TO5jh5tclT0UQjwa0KhTmUmM0cZAVcdAABmw-OVuBkrMVnTvkuLU5EsBJOCTof5bcwWDL71IPtEodX7jGZoZ8Rw6_vyYxkXylf33dlRft307xc)

cicero não é exatamente o que queremos enviar ao **RecordWin,** então vamos refinar nosso teste


#### Escrever o teste primeiro

Vamos mexer no **TestStoreWins** para melhorar o escopo de testes 

**server\_test.go**

    func TestStoreWins(t *testing.T) {
       store := StubPlayerStore{
           map[string]int{},
           nil,
       }
       server := &PlayerServer{&store}

<!---->

       t.Run("Armazena as vitórias em POST", func(t *testing.T) {
           player := "victor"

<!---->

           request := newPostWinRequest(player)
           response := httptest.NewRecorder()

<!---->

           server.ServeHTTP(response, request)

<!---->

           assertStatus(t, response.Code, http.StatusAccepted)

<!---->

           if len(store.winCalls) != 1 {
               t.Fatalf("Recebeu %d chamadas para RecordWin e quer %d", len(store.winCalls), 1)
           }

<!---->

           if store.winCalls[0] != player {
               t.Errorf("Não armazenou corretamente o vencedor, recebeu %q mas queria %q", store.winCalls[0], player)
           }
       })

<!---->

    }

Agora que sabemos que existe um elemento no nosso slice **winCalls** podemos de forma segura referenciar o primeiro a checar se é igual ao **player**

- Vamos testar

![](https://lh7-us.googleusercontent.com/11z-n-jC4O0wd3d927WrIsqOB3n1aF6-0A2Vj-Vul_qvX2gQUTBguBSWziDX2IuKL0UCmv1GIjpxesLxqcB8mfi7v9_FoAXA4gvAAmsAtOITugM4FngRZ_mRwF9pUWhmvr9H3aQ0Cgu9uALEKSRH4uA)


#### Escreva código o suficiente para passar 

    func (p *PlayerServer) processWin(w http.ResponseWriter, r *http.Request) {
       player := strings.TrimPrefix(r.URL.Path, "/players/")
       p.store.RecordWin(player)
       w.WriteHeader(http.StatusAccepted)
    }

Mudamos o **processWin** para pegar o **http.Request** para podermos ver a URL e extrair o nome do players. Uma vez que tivermos o nome, podemos chamar nosso **store** com um valor correto para fazer o teste passar


#### Refatorar

Podemos deixar esse código mais objetivo e extrair o nome do player da mesma forma em dois lugares

    func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       player := strings.TrimPrefix(r.URL.Path, "/players/")
      
       switch r.Method {
       case http.MethodPost:
           p.processWin(w, player)
       case http.MethodGet:
           p.showScore(w, player)
       }
    }

<!---->

    func (p *PlayerServer) showScore(w http.ResponseWriter, player string) {
       // r.URL.Path retorna o path da request na qual podemos depois usar o strings.TrimPrefix para “apararmos / removermos” o /players/ e pegar o player desejado
       score := p.store.GetPlayerScore(player)

<!---->

       if score == 0 {
           w.WriteHeader(http.StatusNotFound)
       }

<!---->

       // ATENÇÃO !!! Fprint != Fprintf
       fmt.Fprint(w, score)
    }

<!---->

    func (p *PlayerServer) processWin(w http.ResponseWriter, player string) {
       p.store.RecordWin(player)
       w.WriteHeader(http.StatusAccepted)
    }

Por mais que os testes estão passando, não temos software funcional, pois não implementamos o **PlayerStore** corretamente. Mas está tudo bem, nós identificamos a interface que queriamos, ao invés de tentar desenhar tudo de forma antecipada

Podiamos melhorar os testes do **InMemoryPlayerStore** só que não compensa fazer isso pois precisamos implementar um banco de dados depois

O que vamos fazer agora é escrever um _teste de integração_ entre o nosso **PlayerServer** e o **InMemoryPlayerStore** para terminar a funcionalidade. Isso vai nos permitir chegar no nosso objetivo de estar confiantes que a nossa aplicação está funcionando, sem precisar ter que diretamente testar o **InMemoryPlayerStore.** Não só isso, mas quando nós pegarmos e implementamos o **PlayerStore** com um banco de dados, nós podemos testar a implementação o mesmo _integration test_


### Teste de Integração ( integration test )

Testes de integração podem ser usados para áreas maiores do seu sistema de trabalho, mas você deve ter em mente:

- São mais difíceis de escrever

- Quando falham, pode ser difícil saber o porque ( geralmente é um bug com um componente do teste de integração ) e depois pode ser mais difícil de arrumar

- Às vezes são mais lentos de rodar ( pois geralmente são usados com componentes “reais”, como um **banco de dados** )


#### Escrever o teste primeiro

**server\_integration\_test.go**

    package main

<!---->

    import (
       "net/http"
       "net/http/httptest"
       "testing"
    )

<!---->

    func TestRecordingWinsAndRetrievingThen(t *testing.T) {
       store := InMemoryPlayerStore{}
       server := PlayerServer{&store}
       player := "victor"

<!---->

       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))

<!---->

       response := httptest.NewRecorder()
       server.ServeHTTP(response, newGetScoreRequest(player))
       assertStatus(t, response.Code, http.StatusOK)

<!---->

       assertResponseBody(t, response.Body.String(), "3")
    }

- Estamos criando dois componente ( instâncias da struct ) e estamos tentando integrar eles com: **InMemoryPlayerStore** e **PlayerServer**

- Depois disso disparamos 3 requisições para armazenar 3 vitórias para o **player.** Não estamos tão preocupados sobre o status dos códigos nesses testes já que isso não é tão relevante se os testes estão integrando bem

- A próxima response que nos importamos (por isso armazenamos uma variável **response**) pois nós vamos tentar pegar o score do **player**

* Vamos tentar rodar o teste

![](https://lh7-us.googleusercontent.com/LTNl5_kxmiWcQTWggsXwcyTJE3BxJTN-aEKxmf0LifVHo9JQ-EIAkOfEBtwx_-DxE4Qfy8GjE4wWTOsoDH8lHisy1gnJgLsbIRj3w7GD1AP5O-hPyPAxgIqKoKMZODWkeRsNwQfj8D7A5IBvAT7G4Rc)


#### Escrevendo código suficiente para passar

Aqui vamos escrever mais testes que fizemos anteriormente. Caso fiquemos travados nesse ponto, precisamos reverter as mudanças de volta para o teste que falhou e depois escrever um unit teste mais específico ao redor do **InMemoryPlayerStore** para ajudar a administrar a solução

**in\_memory\_player\_store.go**

    type InMemoryPlayerStore struct {
       store map[string]int
    }

<!---->

    func NewInMemoryPlayerStore() *InMemoryPlayerStore {
       return &InMemoryPlayerStore{map[string]int{}}
    }

<!---->

    func (i *InMemoryPlayerStore) RecordWin(name string) {
       i.store[name]++
    }

<!---->

    func (i *InMemoryPlayerStore) GetPlayerScore(name string) int {
       return i.store[name]
    }

- Precisamos armazenar os dados então adicionei um **map\[string]int** a struct **InMemoryPlayerStore** 

- Por conveniência fiz **NewInMemoryPlayerStore** para inicializar um store, e atualizar o teste de integração para usar ele:

**server\_integration\_test.go**

    func TestRecordingWinsAndRetrievingThen(t *testing.T) {
       store := NewInMemoryPlayerStore() ------------------> MUDANÇA
       server := PlayerServer{store} ------------------> MUDANÇA
       player := "victor"

<!---->

       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
       server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))

<!---->

       response := httptest.NewRecorder()
       server.ServeHTTP(response, newGetScoreRequest(player))
       assertStatus(t, response.Code, http.StatusOK)

<!---->

       assertResponseBody(t, response.Body.String(), "3")
    }

- O resto do código está só encapsulando o **map**

* Vamos testar:

![](https://lh7-us.googleusercontent.com/1TUvQyrmzBWCROvUVYrWC-ql5bdVTmISXaCNfuPtPMLa2164J9Yot2IEKzLVhtbBH3rACSgy04Oaj2nWEKsqG1rBHniGnMQsrrqkpeD9CIBLidq22Rvhy_bbI4XtffxjbHtszoQ6nlnHW_iZi9fIUFQ)

O teste de integração passou, agora vamos mudar a **main** para usar **NewInMemoryPlayerStore()**

**main.go**

    func main() {
       server := &PlayerServer{NewInMemoryPlayerStore()}
       log.Fatal(http.ListenAndServe(":5000", server)) // se tiver algum erro ao servir nosso servidor, o '1og' vai reportar o erro e vai terminar o programa
    }

- Vamos buildar e rodar

* podemos testar algumas vezes com os nomes que quisermos

![](https://lh7-us.googleusercontent.com/2wJZ_VK7nuQQOdmvrGb6nE6SU2419JfLKeGPumb57ia8vCho7UGP_Xr3lOjh4V3mikOYeapgmn6s6cMfGlFi-cNa8i5fNYe7fjSLbzXBB73cVSuBVed4e8DtXBG8JMpac7tkUohBfKeB3hnlJ0mgbuY)
