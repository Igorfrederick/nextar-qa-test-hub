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
| `400` | Validação de payload (schema, no middleware) |
| `401` | Sem token ou token inválido |
| `403` | Perfil autenticado sem permissão |
| `404` | Recurso não encontrado |
| `409` | Violação de regra de domínio (no service) |

A distinção entre `401` e `403` é verificada: token ausente ou inválido é `401`; token válido com perfil insuficiente é `403`.

## Regras de negócio e seus erros

Cada regra falha com `code` próprio e status adequado. Regra nova exige `code` novo.

**Critério de classificação: a dependência de estado.** Invariante de entrada — julgável olhando apenas o payload — valida por schema no middleware e retorna `400`. Invariante de domínio — só julgável consultando o estado do sistema — valida no service e retorna `409`.

| # | Regra | Camada | Status |
|---|---|---|---|
| 1 | `externalRef` único dentro da suíte — importação deduplica | Service | `409` |
| 2 | Suíte com TestRun registrado não pode ser excluída | Service | `409` |
| 3 | TestRun `closed` não aceita Execution | Service | `409` |
| 4 | Execution `pass` ou `fail` exige ao menos uma evidência no payload | Validação | `400` |
| 5 | Execution `fail` ou `blocked` exige `notes` preenchido | Validação | `400` |
| 6 | No máximo uma Execution por (`testRunId`, `testCaseId`) — atualiza, não duplica | Service | `409` |
| 7 | TestRun com casos não executados não pode ser fechado | Service | `409` |
| 8 | Rascunho só é gerado com todos os casos executados | Service | `409` |

Corpo do erro de validação:

```json
{ "error": { "code": "VALIDATION_ERROR", "message": "...", "details": [{ "field": "notes", "issue": "..." }] } }
```

**Regras `400` asserem `VALIDATION_ERROR` mais o campo em `details`, não um `code` próprio. Regras `409` têm `code` específico da violação.**

## Perfis

| Ação | `qa` | `lead` |
|---|---|---|
| Ler projetos, suítes, casos | ✅ | ✅ |
| Criar/editar/excluir projeto, suíte, caso | ❌ | ✅ |
| Importar cenários | ❌ | ✅ |
| Abrir e fechar ciclo | ❌ | ✅ |
| Registrar execução | ✅ | ✅ |
| Gerar rascunho de comentário | ✅ | ✅ |
