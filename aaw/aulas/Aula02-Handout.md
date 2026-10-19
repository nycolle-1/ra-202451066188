# HANDOUT — AULA 02

## Dissecando o HTTP

*6 requisições sob o microscópio — Arquitetura de Aplicações Web*

## 🎯 MISSÃO

Vocês interceptaram 6 conversas entre um app e a API de uma biblioteca. Para CADA card:

- Descrevam o que o cliente pediu (verbo + recurso na URI)
- Expliquem o que o status code da resposta informa
- Respondam: repetindo a MESMA requisição 3 vezes seguidas, o estado do servidor muda?

Ao final, preencham juntos a TABELA-SÍNTESE dos verbos na última página.

*⏱️ Tempo: 30 minutos  |  👥 Formato: em duplas  |  Dica: o card 6 esconde uma pegadinha de quem é a culpa.*

> **Nomes:** Nycolle Amaral 

## REQUISIÇÃO 01 — A prateleira inteira

```text
→ REQUISIÇÃO
GET /api/livros HTTP/1.1
Host: biblioteca.newton.br
Accept: application/json
```

```text
← RESPOSTA
HTTP/1.1 200 OK
Content-Type: application/json

[ { "id": 1, "titulo": "Clean Code", "autor": "Robert C. Martin" },
  { "id": 7, "titulo": "O Programador Pragmático", "autor": "Hunt & Thomas" } ]
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
O cliente usou o recuso de GET para consultar os livros disponivel na biblioteca /api/livros

2. O que o status code informa? Deu certo? Culpa de quem se não deu?

Os statuo 200 pertecente a classificação de respotas bem sucedidas. Mais especificamente um status que a requisição foi bem sucedida e por ser um o GET, orecuso foi carregado e transmitido no corpo da mensagem.

3. Repetindo esta requisição 3 vezes seguidas, o estado do servidor muda? E a resposta?

Por ser um metado seguro, não alteraria o estatos do servidor por não alterar nada. É apenas um requisição de resposta.

## REQUISIÇÃO 02 — O livro fantasma

```text
→ REQUISIÇÃO
GET /api/livros/99 HTTP/1.1
Host: biblioteca.newton.br
Accept: application/json
```

```text
← RESPOSTA
HTTP/1.1 404 Not Found
Content-Type: application/problem+json

{ "title": "Not Found", "status": 404 }
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
O cliente pediu como GET, um livro especifico que está idetificado pelo id 99 usando o recuso /api/livros/99

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
O statos 404 faz parte da classificação resposta de erro do cliente, mais especificamente que o cliente pode comunicar com o servidor mas o sevidor não consigiu encontrar a solicitação/pagina não existe/não pode exibir o resultado. 

3. Repetindo esta requisição 3 vezes seguidas, o estado do servidor muda? E a resposta?
Por se um metado GET, não alterar o estado do servidor mesmo que a requisção em algum momento dê certo. Apenas mudaria a respota de 404 para 200.

## REQUISIÇÃO 03 — Livro novo na estante

```text
→ REQUISIÇÃO
POST /api/livros HTTP/1.1
Host: biblioteca.newton.br
Content-Type: application/json

{ "titulo": "Domain-Driven Design", "autor": "Eric Evans" }
```

```text
← RESPOSTA
HTTP/1.1 201 Created
Location: /api/livros/8
Content-Type: application/json

{ "id": 8, "titulo": "Domain-Driven Design", "autor": "Eric Evans" }
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
O cliente solicitou ao servidor que adicionasse os dados de um novo livro com POST no resuso /api/livros

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
Os statuo 201 pertecente a classificação de respotas bem sucedidas. Mais especificamente um status que a requisição foi bem sucedida e que o novo recuso foi criado.

3. Enviando este POST 3 vezes seguidas, o que acontece na estante? Para que serve o header Location?
Caso seja feito o POST 3 vezes alterar o servidor tres vezes com cada requisição sendo adicionada(deixando repetido) por ser não seguro e o objetivo.

O herder Location é o cabeçalho que idicara a URL para a pagina deve ser redirecionado. Ele é só tem signicado quando é enviado junto de um uma respota de redirecionamento ou quando é 201.

## REQUISIÇÃO 04 — Corrigindo a ficha completa

```text
→ REQUISIÇÃO
PUT /api/livros/7 HTTP/1.1
Host: biblioteca.newton.br
Content-Type: application/json

{ "id": 7, "titulo": "O Programador Pragmático", "autor": "D. Hunt; D. Thomas" }
```

```text
← RESPOSTA
HTTP/1.1 200 OK
Content-Type: application/json

{ "id": 7, "titulo": "O Programador Pragmático", "autor": "D. Hunt; D. Thomas" }
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
Com o PUT, ouve uma atualização/Subestituição do dados armazenaddos no id 7 em /api/livros/7

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
Os statuo 200 pertecente a classificação de respotas bem sucedidas. Mais especificamente um status que a requisição foi bem sucedida, por ser um PUT normanete aparece quando o recurso é carregado pela primeira vez(caso o contario seria 204)

3. Repetindo esta requisição 3 vezes seguidas, o estado do servidor muda? E a resposta?
Na primeria vez ele mudara o status, por isso é considerado não seguro, mas a parti do seguinte será idempotente. As segunda resposta do servidor ira para 204.

## REQUISIÇÃO 05 — Fora do catálogo

```text
→ REQUISIÇÃO
DELETE /api/livros/7 HTTP/1.1
Host: biblioteca.newton.br
```

```text
← RESPOSTA
HTTP/1.1 204 No Content
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
O cliente pediu com o DELET para remover o livro localizado no id 7.

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
Os statuo 204 pertecente a classificação de respotas bem sucedidas. Mais especificamente um status que a requisição foi bem sucedida, o cliente nao precisa sair da pagina atual ou que foi removido como solicitado

3. Repetindo o DELETE, o estado do servidor muda? Que resposta você ESPERA na segunda vez?
A primeira vez ele deletara e isso mudara o statos do servidor. Por possuir um id especifico, caso seja feito o mesmo delet sem adicionar outra coisa no lugar sera idepotente e não mudara o statos.Mas tera uma respota 404 por não ser possivel localizar.

## REQUISIÇÃO 06 — O cadastro capenga

```text
→ REQUISIÇÃO
POST /api/livros HTTP/1.1
Host: biblioteca.newton.br
Content-Type: application/json

{ "autor": "Anônimo" }
```

```text
← RESPOSTA
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json

{ "title": "Bad Request", "status": 400,
  "errors": { "Titulo": [ "O campo Titulo é obrigatório" ] } }
```

**Sua análise:**

1. O que o cliente pediu (verbo + recurso)?
O cliente pediu para adicionar novos livros,com o POST em /api/livros.

2. O que o status code informa? Deu certo? Culpa de quem se não deu?
O statos 400 faz parte da classificação resposta de erro do cliente, mais especificamente que o servidor não pode ou não irá processar a requisição devido a alguma coisa que foi entendida como um erro do cliente (por exemplo, sintaxe de requisição mal formada, enquadramento de mensagem de requisição inválida ou requisição de roteamento enganosa).


3. Repetindo esta requisição 3 vezes seguidas, o estado do servidor muda? E a resposta?
Por possui o mesmo erro não iria alterar o stado do servidor. Apenas se for corrigido o erro. A respota tambem não mudara.

## TABELA-SÍNTESE — Os verbos do HTTP

*Preencham com base nos 6 cards. “Seguro” = não altera nada no servidor. “Idempotente” = repetir N vezes deixa o servidor no mesmo estado que 1 vez.*

| **Verbo** | **Para que serve** | **Seguro?** | **Idempotente?** | **Status típicos** |
| --- | --- | --- | --- | --- |
| **`GET`** |Consultar um recurso ou uma coleção|Sim|Sim|200, 304, 400, 404|
| **`POST`** |Criar recursos ou solicitar o processamento de uma ação|Não|Não, normalmente|200, 201, 202, 400, 409|
| **`PUT`** |Substituir completamente um recurso em uma URI conhecida|Não|Sim|200, 201, 204, 400, 404|
| **`PATCH`** |Alterar parcialmente um recurso|Não|Não necessariamente|200, 204, 400, 404, 409|
| **`DELETE`** |Remover um recurso|Não|Sim|200, 202, 204, 404, 409|

## DESAFIO

1. O verbo PATCH não apareceu em nenhum card. Qual a diferença entre PATCH e PUT? Um app de banco quer alterar SÓ o apelido do usuário, entre dezenas de campos do perfil — qual dos dois você usaria e por quê?

A principal diferença entre os dois metados qual será que PUT fica legível que a alteração do dado será com referência a entidade completa.

 Enquanto o O PATCH é usado para atualização parcial, quando você não quer mandar o payload completo. 

 No caso do aplicativo do banco seria recomendado o PATCH.