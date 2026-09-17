# Checklist — Backend

Aplicar a qualquer diff que toque `backend/**`. Base: `conventions/backend_conventions.md` e `conventions/api_contract.md`.

## Camadas (CRITICAL)

- [ ] Nenhum controller contém regra de negócio
- [ ] Nenhum service importa ou recebe `req`/`res`
- [ ] Nenhum model conhece service
- [ ] Nenhuma lógica dentro da definição de rota

## Segurança (CRITICAL)

- [ ] Nenhum segredo literal no código — tudo por variável de ambiente
- [ ] Nenhum `.env` real versionado
- [ ] Senha com hash bcrypt, custo vindo de env
- [ ] `passwordHash` não aparece em nenhuma resposta de API
- [ ] Senha não aparece em log, nem em texto puro

## Autenticação e autorização

- [ ] JWT com geração e validação explícitas
- [ ] Expiração do token definida
- [ ] Segredo do JWT em variável de ambiente
- [ ] Validação do token em middleware, não repetida em controller
- [ ] Toda rota marcada `[lead]` no contrato tem o middleware de perfil
- [ ] Perfis como constante única, sem string mágica espalhada

## Validação

- [ ] Schema Zod em `validators/` para toda entrada
- [ ] Validação aplicada por middleware, antes do service
- [ ] Falha de validação retorna `400` com `details` preenchido

## Erros

- [ ] Formato único `{ error: { code, message, details } }` em toda resposta de erro
- [ ] `code` em `SCREAMING_SNAKE_CASE`, vindo de catálogo único
- [ ] Middleware de erro centralizado; sem `try/catch` repetido por controller
- [ ] Status HTTP correto: `400` validação, `401` sem token, `403` perfil, `404` ausente, `409` regra
- [ ] Regra de negócio nova tem `code` correspondente no catálogo

## Modelagem

- [ ] Unicidade declarada onde a regra exige: `User.email`, `Project.key`
- [ ] `TestCase.externalRef` único **dentro da suíte** (índice composto), não global
- [ ] Par (`testRunId`, `testCaseId`) único em Execution
- [ ] `Evidence` como subdocumento de Execution, não coleção própria
- [ ] Referência entre entidades por `ObjectId`, campo terminando em `Id`

## Contrato

- [ ] Rota, verbo e formato conforme `api_contract.md`
- [ ] Nenhuma rota fora do contrato sem sinalização prévia
