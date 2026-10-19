# HANDOUT — AULA 03

## Consultoria de Design: a API da EscolaTech

*Identifique os anti-padrões e proponha o redesenho — Arquitetura de Aplicações Web*

## 🎯 MISSÃO

A EscolaTech contratou a consultoria de vocês para auditar a API do sistema escolar. Todos os endpoints abaixo FUNCIONAM e estão em produção — mas o time novo se recusa a mexer neles. Para CADA endpoint:

- Identifiquem o(s) problema(s) de design (pode haver mais de um!)
- Proponham o redesenho: método HTTP + rota + status codes corretos

*⏱️ Tempo: 25 minutos  |  👥 Formato: em duplas  |  Dica: se a rota conta o que faz em português, algo está errado.*

> **Nomes:** Nycolle   **Turma:** ____________________   **Data:** ___ / ___ / ______

## ENDPOINT 01 — POST /api/getAlunos

**Documentação atual (extraída da wiki da EscolaTech):**

```text
POST /api/getAlunos
Retorna TODOS os alunos cadastrados (hoje: 12.482 registros).
Resposta: 200 OK + array JSON completo (~9 MB).
Obs. da wiki: "usar POST porque GET não estava funcionando".
```

1. Qual(is) problema(s) de design vocês identificam?
Os principais problemas são na na falta de organização na requisição que acaba prejuticando o desempenho depedendo da quantidade de alunos. Por não possuir filtros, paginação, ordenção e ate mesmo limite de registro.

Outro problema é usar o POST para consultar(GET), isso pode prejuticar as ferramentas intermediarias, logs, favoritos, cache.

2. Seu redesenho (método + rota + status codes):

GET /api/v1/alunos?page=1&limit=50
Status codes:
200 OK
400 Bad Request

## ENDPOINT 02 — GET /deletarAluno?id=7

**Documentação atual (extraída da wiki da EscolaTech):**

```text
GET /deletarAluno?id=7
Remove o aluno do banco de dados.
Resposta: 200 OK + "OK" (mesmo se o aluno não existir).
Obs. da wiki: "dá pra deletar pelo navegador, bem prático".
```

1. Qual(is) problema(s) de design vocês identificam?
Os principais problemas são usar o GET para poder deletar o aluno 7 invez de usar o comando DELET direto. A resposta do statos do servidor retona como 200 mesmo quando o aluno não existe e na rota tem deletar como verbo.

2. Seu redesenho (método + rota + status codes):
DELETE /api/v1/alunos/7
Status codes: 204 

## ENDPOINT 03 — POST /api/alunos (criação)

**Documentação atual (extraída da wiki da EscolaTech):**

```text
POST /api/alunos
Body: { "nome": "...", "curso": "..." }
Cria o aluno e responde: 200 OK + body "OK".
O app precisa buscar a lista inteira de novo para descobrir o ID gerado.
```

1. Qual(is) problema(s) de design vocês identificam?
A respota do servirdo deveria ser 201 Created, e mesmo assum não deveria retonar o Ok porque o aluno nçaofoi criado. Alem de nçao informar uma ID.

2. Seu redesenho (método + rota + status codes):
POST /api/v1/alunos
Content-Type: application/json
Status codes: 201 Created
400 Bad Request
409 Conflict

## ENDPOINT 04 — GET /escolas/1/turmas/3/alunos/25/matriculas/88/disciplinas/12

**Documentação atual (extraída da wiki da EscolaTech):**

```text
GET /escolas/1/turmas/3/alunos/25/matriculas/88/disciplinas/12
Retorna os dados da disciplina 12 da matrícula 88.
Para montar a URL o app precisa conhecer 5 IDs diferentes.
Resposta: 200 OK + JSON da disciplina.
```

1. Qual(is) problema(s) de design vocês identificam?
A rota está longa, obrigando o cliente saber 5 idetificadores para acessar o unico recuso. Ao longo prazo se torna impordutivo.

2. Seu redesenho (método + rota + status codes):
GET /api/v1/matriculas/88/disciplinas/12

Status codes:
200 OK
404 Not Found

## ENDPOINT 05 — GET /api/alunos/7/matriculas (erro)

**Documentação atual (extraída da wiki da EscolaTech):**

```text
GET /api/alunos/7/matriculas
Se o aluno 7 não existe, responde:
200 OK + "<html><b>Erro: aluno nao existe!</b></html>"
O app mobile quebra tentando fazer parse do JSON.
```

1. Qual(is) problema(s) de design vocês identificam?
O status do HTTP contradiz com o retorno do servidor. Alem do formato da respota não ser consistente, deveria ser JSON e não um html em uma api.

2. Seu redesenho (método + rota + status codes):
GET /api/v1/alunos/7/matriculas
Accept: application/json
status codes:
400 Bad Request
401 Unauthorized
403 Forbidden

## ENDPOINT 06 — PUT /api/atualizarNotaParcial?aluno=7&disc=12&nota=8.5

**Documentação atual (extraída da wiki da EscolaTech):**

```text
PUT /api/atualizarNotaParcial?aluno=7&disc=12&nota=8.5
Atualiza SÓ a nota parcial da disciplina, sem body.
Todos os dados vão na query string.
Resposta: 200 OK + "OK".
```

1. Qual(is) problema(s) de design vocês identificam?
Os dados não sao seguros ao enviar tudo por uma query string, ficaria exporto no historico, ferramentas de monitoramento e logs. O que usar o PUT/PACTH resolveria a questão

2. Seu redesenho (método + rota + status codes):
PATCH /api/v1/matriculas/88/disciplinas/12
Content-Type: application/json
Status codes:
200 OK
204 No Content
400 Bad Request
404 Not Found

## DESAFIO

1. A EscolaTech quer lançar mudanças na API sem quebrar o app mobile antigo, que não recebe atualização há 2 anos. Que decisão de design — que falta na API INTEIRA — resolve esse problema? Como ficariam as rotas?

Seria fazer uma versao 2 com as novas rotas. Arrumando o alinamento e o caminho. Fazendo grauamente não quebraria o antigo e com as mudnaças de turma e alunos as antigas rotas ficariam em desusos.
