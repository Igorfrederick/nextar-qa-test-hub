# QA Test Hub

Contexto base do projeto. Este arquivo é lido por todos os agentes antes de qualquer tarefa. Regras aqui valem sobre qualquer preferência padrão do agente.

---

## 1. O que é este projeto

Aplicação web que substitui a planilha como camada de execução de testes de QA, cobrindo o intervalo entre a geração de cenários (já automatizada por agentes existentes no time) e a publicação do resultado no Jira.

Este repositório é o entregável das **etapas 1, 2, 3 e 5** de um PDI. A etapa 4 (documento de análise de oportunidade) vive fora daqui.

### Por que isso importa para quem escreve código aqui

O projeto é avaliado por **análise do código no GitHub**, não pelo produto rodando. Os critérios declarados são:

| Frente | Critérios de avaliação |
|---|---|
| Frontend | Estrutura de pastas, componentização e reutilização, boas práticas |
| Backend | Arquitetura em camadas, JWT correto (geração e validação), hash de senhas, variáveis de ambiente, modelagem de dados |
| E2E | Organização dos testes, qualidade dos seletores, asserções específicas, cobertura dos fluxos, independência entre testes, setup e teardown |
| Aplicação da solução | Código executável, organização, README claro, facilidade de uso |

Toda decisão técnica deve ser defensável em voz alta. Código que funciona mas não é explicável não serve aqui.

---

## 2. Escopo

### v1 — o que entra

- CRUD de Projeto, Suíte e Caso de Teste
- Autenticação JWT com dois perfis (QA e Líder) e autorização por rota
- Importação de cenários por arquivo (CSV/JSON) com deduplicação
- Ciclo de execução: marcação de status por caso, com evidência vinculada
- Dashboard de cobertura e progresso
- Geração de rascunho de comentário de validação para o Jira via LLM

### v1 — o que **não** entra

Não implementar, mesmo que pareça natural:

- Integração direta com o agente gerador de cenários (importação é por arquivo)
- Integração com o orquestrador de engenharia reversa
- Leitura ou escrita via API do Jira (o rascunho é copiado à mão)
- Upload real de arquivos para storage externo (evidência é referência + metadados)
- Análise histórica de recorrência de falhas
- Notificações, e-mail, websockets, tempo real
- Multi-tenant, billing, recuperação de senha, OAuth

Se uma tarefa exigir algo desta lista, pare e sinalize antes de implementar.

---

## 3. Domínio

### Entidades

```
User
Project  →  Suite  →  TestCase
                  ↓
              TestRun  →  Execution  →  Evidence[]
```

| Entidade | Campos principais |
|---|---|
| `User` | name, email (único), passwordHash, role (`qa` \| `lead`) |
| `Project` | name, key (único, ex. `NEXWEB`), description |
| `Suite` | projectId, name, description |
| `TestCase` | suiteId, externalRef (único na suíte), title, preconditions, steps[], expectedResult, tags[] |
| `TestRun` | suiteId, name, version, status (`open` \| `closed`), startedAt, closedAt, createdBy |
| `Execution` | testRunId, testCaseId, status (`pass` \| `fail` \| `blocked`), notes, evidence[], executedBy, executedAt |
| `Evidence` | (subdocumento de Execution) type (`video` \| `screenshot`), filename, description |

### Perfis e permissões

| Ação | QA | Líder |
|---|---|---|
| Ler projetos, suítes, casos | ✅ | ✅ |
| Criar/editar/excluir projeto, suíte, caso | ❌ | ✅ |
| Importar cenários | ❌ | ✅ |
| Abrir e fechar ciclo (TestRun) | ❌ | ✅ |
| Registrar execução | ✅ | ✅ |
| Gerar rascunho de comentário | ✅ | ✅ |

### Regras de negócio

Estas regras existem para serem violadas nos testes. Toda regra tem que falhar com erro claro e status HTTP adequado.

1. `externalRef` é único dentro da suíte — importação deduplica em vez de duplicar
2. Não é possível excluir Suíte que possua TestRun registrado
3. Não é possível registrar Execution em TestRun com status `closed`
4. Execution com status `pass` ou `fail` exige ao menos uma evidência
5. Execution com status `fail` ou `blocked` exige `notes` preenchido
6. Existe no máximo uma Execution por par (`testRunId`, `testCaseId`) — nova marcação atualiza, não duplica
7. Não é possível fechar TestRun com casos ainda não executados
8. Rascunho de comentário só é gerado para TestRun com todos os casos executados

### Telas

| Tela | Rota | Observação |
|---|---|---|
| Login | `/login` | — |
| Lista de projetos | `/projects` | — |
| Detalhe do projeto (suítes) | `/projects/:id` | — |
| Detalhe da suíte (casos + importação) | `/suites/:id` | — |
| Execução do ciclo | `/runs/:id` | **Mobile-first** — QA executa teste em dispositivo físico com as mãos ocupadas |
| Fechamento do ciclo e rascunho | `/runs/:id/summary` | Dashboard de cobertura + geração do comentário |

Seis telas, seis Page Objects. Não criar tela nova sem sinalizar.

---

## 4. Stack e convenções gerais

- **Frontend:** React + Vite, React Router, React Hook Form + Zod
- **Backend:** Node.js + Express, MongoDB + Mongoose, JWT, bcrypt
- **E2E:** Playwright + TypeScript
- **Idioma:** identificadores de código, entidades e campos em **inglês**; textos de interface em **português**. Sem mistura dentro de um mesmo identificador.

### Commits e histórico

O histórico é parte do que será avaliado — um avaliador lê o `git log` antes de ler o código.

- **Conventional Commits:** `feat:`, `fix:`, `test:`, `docs:`, `refactor:`, `chore:`
- **Critério de corte:** unidade funcional coerente, **vertical**. Se o commit precisa do próximo para o projeto não quebrar, foi cortado cedo demais. Não cortar por camada — "todos os models", depois "todos os controllers" é exatamente o padrão a evitar.
- **Regra de negócio e o teste que a prova vão no mesmo commit.** Regra sem teste é commit incompleto.
- **`refactor:` nunca misturado com `feat:`.** Refatoração não muda comportamento; se mudou, não era refatoração.
- **Corpo da mensagem com o porquê** quando o commit materializa uma decisão. No resto, título sozinho basta.
- **Uma branch por frente**, fechada com PR para a `main` — mesmo trabalhando sozinho.
- Não agregar o dia inteiro em um commit. Não reescrever histórico já empurrado.

### Segurança — não negociável

- Senha sempre com hash (bcrypt). Nunca em texto puro, nunca em log, nunca em resposta de API.
- Segredos só via variável de ambiente. Nenhum valor real commitado; `.env.example` com chaves e valores fictícios.
- Nenhum dado real no repositório: sem nome de cliente, sem chave real de tarefa do Jira, sem conteúdo de bug real. Seed usa dado fictício.

---

## 5. Contrato da API

Base: `/api`. Autenticação via header `Authorization: Bearer <token>`.

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

### Formato de erro

Resposta única para todo erro da API:

```json
{ "error": { "code": "RUN_CLOSED", "message": "Ciclo encerrado não aceita novas execuções", "details": [] } }
```

Códigos HTTP: `400` validação, `401` sem token ou token inválido, `403` perfil sem permissão, `404` não encontrado, `409` violação de regra de negócio.

O campo `code` é o que os testes E2E asseguram. Nunca asserir sobre a mensagem em português.

---

## 6. Convenção de seletores

Todo elemento com o qual o teste interage carrega `data-cy`. Sem exceção — se um componente novo nasce sem `data-cy`, ele está incompleto.

Padrão: `contexto-elemento[-identificador]`, kebab-case.

```
login-email-input
login-submit-button
project-list-row-{projectKey}
test-case-row-{externalRef}
execution-status-pass-button
run-close-button
coverage-percent-value
comment-draft-textarea
error-toast
```

Nunca usar como seletor em teste: classe CSS, texto visível, posição no DOM, hierarquia de tags.

---

## 7. Estratégia de teste

Page Object Model, **um Page Object por tela**. Decisão alinhada com o tech lead; registrada em `docs/decisions.md`.

### O que fica no Page Object

- Locators (`data-cy`)
- Ações de baixo nível: `preencherLogin()`, `marcarStatus()`, `clicarSalvar()`
- Navegação para a própria tela

### O que **não** fica no Page Object

| Preocupação | Onde fica | Motivo |
|---|---|---|
| Asserções | No teste | Arrange-Act-Assert precisa estar legível no arquivo do teste |
| Criação de massa de dados | Factory + Service Layer (via API) | Setup por UI é lento e frágil |
| Autenticação e estado inicial | Fixture do Playwright | Injeção de dependência, não herança de BasePage |
| Chamadas HTTP | Service Layer | Page Object fala com a tela, não com a API |

### Camadas de apoio

- `factories/` — Factory com overrides usando faker. **Zero dado hardcoded.** Cada teste gera a sua própria massa, para garantir independência.
- `services/` — encapsula o CRUD HTTP da API. Todo setup acontece aqui, não pela interface.
- `fixtures/` — estende o `test` base do Playwright: autenticação por perfil, page objects injetados, cliente de API pronto.

### Independência

Nenhum teste pode depender da execução de outro, nem da ordem, nem de estado deixado por um anterior. Dado compartilhado entre dois testes é bug de arquitetura de teste, não conveniência.

---

## 8. Estrutura de pastas

```
backend/
  src/
    config/          conexão, variáveis de ambiente
    models/          schemas Mongoose
    routes/          definição de rotas
    controllers/     entrada/saída HTTP, sem regra de negócio
    services/        regra de negócio
    middlewares/     auth, autorização por perfil, validação, erro
    validators/      schemas Zod de request
    utils/
  tests/

frontend/
  src/
    pages/           uma pasta por tela
    components/      componentes reutilizáveis, sem regra de negócio
    hooks/
    services/        cliente HTTP
    schemas/         schemas Zod de formulário
    contexts/        auth
    utils/

e2e/
  tests/             organizados por jornada do usuário
  pages/             Page Objects, um por tela
  fixtures/
  factories/
  services/
  support/

docs/
  decisions.md       log de decisões técnicas
```

**Regra de camada no backend:** controller não contém regra de negócio; service não conhece `req`/`res`; model não conhece service. Violação disso é o erro mais visível na avaliação de "arquitetura em camadas".

---

## 9. Log de decisões

Toda decisão técnica relevante vai para `docs/decisions.md`, uma entrada curta:

```
## [data] Título da decisão
**Decisão:** o que foi decidido
**Motivo:** por quê
**Alternativa descartada:** o que não foi escolhido e por quê
```

Isso existe porque as decisões serão questionadas na avaliação. Resposta escrita na data em que foi tomada vale mais do que justificativa reconstruída depois.

---

## 10. Protocolo de trabalho com os agentes

O modo é declarado no início de cada tarefa:

- **Modo geração** — o agente escreve; Igor revisa antes do commit.
- **Modo revisão** — Igor escreve; o agente critica contra os critérios da seção 1 e aponta o que um avaliador marcaria.

Ao final de cada ciclo, independentemente do modo, o agente faz **uma pergunta de defesa** sobre uma decisão técnica daquele código. Se a resposta não vier, o código volta.

Regras permanentes para qualquer agente neste repositório:

1. Não implementar nada da lista de não-escopo (seção 2) sem sinalizar antes.
2. Não introduzir biblioteca nova sem justificar e registrar em `docs/decisions.md`.
3. Não gerar código que Igor não conseguiria explicar — preferir a solução clara à solução esperta.
4. Não criar abstração antes do terceiro uso.
5. Todo componente interativo nasce com `data-cy`.
6. Nenhum dado real, segredo ou credencial no código.

---

## 11. Decisões em aberto

Confirmar antes de implementar a parte correspondente:

- Modelo e provedor de LLM para a rota `comment-draft`, conforme política interna de uso de IA
- Formato exato do arquivo de importação (colunas esperadas no CSV/JSON gerado pelos agentes atuais)
- Formato-padrão do comentário de validação no Jira, a ser usado como referência na geração do rascunho
