# Contrato da API

Fonte única do contrato. Extraído do `CLAUDE.md`. Consumido pelo backend (implementa), pelo frontend (chama) e pelo E2E (assere).

Base: `/api`. Autenticação via header `Authorization: Bearer <token>`.

## Rotas

```
POST   /auth/register
POST   /auth/login                        → { token, user }
GET    /auth/me

GET    /projects
POST   /projects                          [lead]
GET    /projects/:id
PATCH  /projects/:id                      [lead]
DELETE /projects/:id                      [lead]

GET    /projects/:id/suites
POST   /suites                            [lead]
GET    /suites/:id
PATCH  /suites/:id                        [lead]
DELETE /suites/:id                        [lead]

GET    /suites/:id/test-cases
POST   /test-cases                        [lead]
PATCH  /test-cases/:id                    [lead]
DELETE /test-cases/:id                    [lead]
POST   /suites/:id/import                 [lead]  → { imported, duplicates, errors }

GET    /suites/:id/runs
POST   /runs                              [lead]
GET    /runs/:id                          → run + casos + execuções
POST   /runs/:id/close                    [lead]
GET    /runs/:id/coverage                 → { total, pass, fail, blocked, pending, percent }

PUT    /runs/:id/executions/:testCaseId   → cria ou atualiza (regra 6)

POST   /runs/:id/comment-draft            → { draft }   ← única rota que usa LLM
```

`[lead]` exige middleware de autorização por perfil. Ausência é falha de segurança.

`PUT` em execution é deliberado: a operação é idempotente por par (`testRunId`, `testCaseId`) — nova marcação atualiza, não duplica.

## Formato de erro

Resposta única para todo erro:

```json
{ "error": { "code": "RUN_CLOSED", "message": "Ciclo encerrado não aceita novas execuções", "details": [] } }
```

| Campo | Papel |
|---|---|
| `code` | Contrato. Estável, `SCREAMING_SNAKE_CASE`, catálogo único |
| `message` | Apresentação. Português, livre para mudar |
| `details` | Lista de falhas de validação; vazia quando não se aplica |

**O `code` é o que os testes E2E asseveram. Nunca a mensagem em português** — decisão registrada em `docs/decisions.md`.

## Status HTTP

| Status | Quando |
|---|---|
| `400` | Falha de validação de entrada |
| `401` | Sem token ou token inválido |
| `403` | Perfil autenticado sem permissão |
| `404` | Recurso não encontrado |
| `409` | Violação de regra de negócio |

A distinção entre `401` e `403` é verificada: token ausente ou inválido é `401`; token válido com perfil insuficiente é `403`.

## Regras de negócio e seus erros

Cada regra falha com `code` próprio e status adequado. Regra nova exige `code` novo.

| # | Regra | Status |
|---|---|---|
| 1 | `externalRef` único dentro da suíte — importação deduplica | `409` |
| 2 | Suíte com TestRun registrado não pode ser excluída | `409` |
| 3 | TestRun `closed` não aceita Execution | `409` |
| 4 | Execution `pass` ou `fail` exige ao menos uma evidência | `409` |
| 5 | Execution `fail` ou `blocked` exige `notes` preenchido | `409` |
| 6 | No máximo uma Execution por par (`testRunId`, `testCaseId`) | — (atualiza) |
| 7 | TestRun com casos não executados não pode ser fechado | `409` |
| 8 | Rascunho só é gerado com todos os casos executados | `409` |

## Perfis

| Ação | `qa` | `lead` |
|---|---|---|
| Ler projetos, suítes, casos | ✅ | ✅ |
| Criar/editar/excluir projeto, suíte, caso | ❌ | ✅ |
| Importar cenários | ❌ | ✅ |
| Abrir e fechar ciclo | ❌ | ✅ |
| Registrar execução | ✅ | ✅ |
| Gerar rascunho de comentário | ✅ | ✅ |
