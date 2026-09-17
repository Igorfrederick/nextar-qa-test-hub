# Convenções — Backend

Fonte única para qualquer trabalho em `backend/`. Extraído do `CLAUDE.md`.

## Arquitetura em camadas

```
routes/       define rota e middleware; nenhuma lógica
controllers/  entrada e saída HTTP; nenhuma regra de negócio
services/     regra de negócio; não conhece req/res
models/       schema Mongoose; não conhece service
middlewares/  auth, autorização por perfil, validação, erro
validators/   schemas Zod de request
config/       conexão e variáveis de ambiente
```

`backend/tests/` acompanha essas camadas: testes automatizados de service e de API, sem navegador.

Três invariantes, nesta ordem de gravidade:

1. **Controller não contém regra de negócio.** Extrai da requisição, chama o service, devolve a resposta.
2. **Service não conhece `req` nem `res`.** Recebe dados e devolve dados ou lança erro de domínio. Se um service precisa de `req`, o controller não extraiu o suficiente.
3. **Model não conhece service.** O schema descreve a forma do dado e suas restrições. Regra que dependa de outra entidade vive no service.

Violação desses três é o erro mais visível na avaliação de "arquitetura em camadas".

## Testabilidade

A separação de camadas existe também para isto: **o service tem de ser exercitável sem HTTP e sem subir a aplicação.** Recebe dados, devolve dados ou lança erro de domínio — nada nele exige requisição, resposta ou servidor em pé.

Consequências práticas:

- Service que importa `req`/`res` não é testável isoladamente. A invariante de camada e o requisito de testabilidade são a mesma regra vista de dois lados.
- Dependência externa (conexão, relógio, gerador de identificador) entra por parâmetro ou por módulo de `config/`, nunca instanciada dentro da regra.
- **Regra de negócio nasce com teste automatizado em `backend/tests/`, sem depender de UI**, e os dois viajam no mesmo commit.

A ferramenta de teste ainda não foi escolhida — decisão em aberto para o Passo 3. A convenção é agnóstica: o que se exige é teste automatizado que exercite o service sem navegador.

## Conexão com o banco

- Conexão isolada em `config/`, nunca aberta dentro de service, controller ou model
- URI exclusivamente por variável de ambiente (`MONGODB_URI`)
- Falha de conexão tratada de forma explícita: a aplicação não sobe silenciosamente sem banco
- Encerramento controlado da conexão, para que a suíte de teste não fique pendurada

## Autenticação

- JWT com geração e validação explícitas, expiração definida
- Segredo exclusivamente em variável de ambiente; nunca literal no código
- Validação do token em middleware, não repetida em controller
- O payload do token carrega o mínimo para autorizar: identificador e perfil

## Senhas

- Sempre com hash bcrypt, custo vindo de variável de ambiente
- Nunca em texto puro, nunca em log, nunca em resposta de API
- `passwordHash` jamais retorna ao cliente — remover na serialização, não confiar em o controller lembrar

## Autorização por perfil

- Middleware dedicado, aplicado na definição da rota
- Perfis (`qa`, `lead`) como constante única; string mágica espalhada pelo código é achado
- Rotas marcadas `[lead]` no contrato exigem o middleware; ausência é falha de segurança, não esquecimento

## Validação de entrada

- Schema Zod em `validators/`, aplicado por middleware antes de chegar ao service
- O service pode assumir que recebeu dado com a forma correta
- Falha de validação retorna `400` com `details` preenchido

## Erros

Formato único, sem exceção:

```json
{ "error": { "code": "RUN_CLOSED", "message": "Ciclo encerrado não aceita novas execuções", "details": [] } }
```

- `code` em `SCREAMING_SNAKE_CASE`, estável, nascido de um único catálogo
- `message` em português, camada de apresentação
- Middleware de erro centralizado; `try/catch` repetido em cada controller é achado
- Regra de negócio nova exige `code` novo no catálogo

Mapa de status:

| Status | Significado |
|---|---|
| `400` | Falha de validação de entrada |
| `401` | Sem token ou token inválido |
| `403` | Perfil sem permissão para a ação |
| `404` | Recurso não encontrado |
| `409` | Violação de regra de negócio |

## Modelagem

- Unicidade declarada no schema quando a regra de negócio exige: `User.email`, `Project.key`, `TestCase.externalRef` dentro da suíte, par (`testRunId`, `testCaseId`) em Execution
- Índice composto onde a regra é composta — `externalRef` é único *na suíte*, não globalmente
- `Evidence` é subdocumento de `Execution`, não coleção própria
- Referência entre entidades por `ObjectId`, com o nome do campo terminando em `Id`

## Anti-padrões

| Anti-padrão | Por quê |
|---|---|
| Lógica dentro da definição de rota | Rota é roteamento; regra não é testável ali |
| `try/catch` repetido em vez de middleware de erro | Formato de erro diverge entre rotas |
| Retorno de documento Mongoose cru | Vaza `passwordHash` e campos internos |
| String de perfil espalhada pelo código | Erro de digitação vira falha de autorização silenciosa |
| Service importando `req`/`res` | Quebra a camada e impede teste sem HTTP |
