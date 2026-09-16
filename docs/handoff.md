# Handoff — QA Test Hub

Documento de retomada para continuar o trabalho no Claude Code (VS Code). Contém o estado das decisões, os artefatos já produzidos e a ordem do que fazer a seguir.

**Como usar:** salve este arquivo como `docs/handoff.md` no repositório novo e abra o Claude Code com algo como:

> Leia `docs/handoff.md` e `CLAUDE.md`. Vamos executar o Passo 1. Antes de escrever qualquer código, me diga o que entendeu do escopo e o que ficou ambíguo.

---

## 1. Estado atual

### Contexto

Projeto que serve como entregável das **etapas 1, 2, 3 e 5** de um PDI de QA na Nextar. A etapa 4 é um documento e vive fora do repositório.

O projeto é avaliado por **análise do código no GitHub**. Isso condiciona tudo: o produto rodando importa menos que a organização, a arquitetura em camadas, a qualidade dos seletores e a independência dos testes.

### Decisões já tomadas

| Decisão | Definição | Situação |
|---|---|---|
| Domínio da aplicação | QA Test Hub — substitui a planilha de execução de testes e gera rascunho de comentário de validação para o Jira | Fechada |
| Repositório único | Um repo atende às etapas 1, 2, 3 e 5 | Confirmar com o líder |
| Design pattern do E2E | Page Object Model, **um page por tela** | **Alinhada com o tech lead** — registrar em `docs/decisions.md` com data e nome |
| Divisão de responsabilidades no POM | Locators e ações no page; asserções no teste; massa em factory; setup via service/API; auth em fixture | Fechada |
| Seletores | `data-cy`, padrão `contexto-elemento[-identificador]` | Fechada |
| Postura dos agentes | Pair — alterna modo geração e modo revisão, com pergunta de defesa ao fim de cada ciclo | Fechada |
| Sequência das frentes | Backend e contrato primeiro → frontend → E2E | Fechada |
| IA no v1 | Uma única capacidade: geração do rascunho de comentário de validação a partir dos dados de execução, com revisão humana | Fechada |
| Publicação automática no Jira | **Fora do escopo do v1** | Fechada |

### Artefatos já produzidos

| Arquivo | Destino | Situação |
|---|---|---|
| `CLAUDE.md` | Raiz do repositório | Pronto — contém domínio, contrato da API, convenções, estratégia de teste, protocolo dos agentes |
| Documento da etapa 4 (análise de oportunidade) | Fora do repo | Versão de trabalho, campos de medição a preencher |
| Documento de contexto para baseline de tempo | Fora do repo | Insumo para análise separada |

---

## 2. Ordem de execução

### Passo 1 — Esqueleto do repositório

- Criar repositório com `backend/`, `frontend/`, `e2e/`, `docs/`
- Colocar `CLAUDE.md` na raiz e este arquivo em `docs/handoff.md`
- `.gitignore`, `.env.example` (chaves com valores fictícios), `README.md` inicial
- Criar `docs/decisions.md` e registrar as decisões da tabela acima, começando pela do Page Object Model — com data e nome de quem alinhou

### Passo 2 — Agentes

Criar os quatro agentes em `.claude/agents/` conforme a especificação da seção 3. Arquivos markdown com frontmatter YAML; `name` e `description` são obrigatórios. O comando `/agents` cria de forma interativa, se preferir.

Depois de criados, testar cada um com uma tarefa pequena antes de confiar neles em tarefa grande.

### Passo 3 — Backend até o contrato congelado

Ordem: modelos → auth (registro, login, JWT, bcrypt) → middleware de autorização por perfil → CRUD de Project/Suite/TestCase → TestRun e Execution com as regras de negócio → importação → cobertura → `comment-draft`.

Só avançar para o frontend quando o contrato estiver estável. Retrabalho aqui custa nos dois lados.

### Passo 4 — Frontend

Ordem: setup e roteamento → contexto de auth e tela de login → lista e detalhe de projeto → suíte e importação → tela de execução (**mobile-first**) → fechamento e dashboard.

Todo elemento interativo nasce com `data-cy`. Sem exceção.

### Passo 5 — E2E

Ordem: setup do Playwright → services e factories → fixtures de auth por perfil → page objects (um por tela) → testes por jornada.

Jornadas mínimas: login e autenticação (incluindo acesso negado por perfil), CRUD de suíte e caso, importação com deduplicação, ciclo completo de execução, e as regras de negócio que falham (ciclo fechado, evidência obrigatória, fechamento com pendências).

### Passo 6 — README e exemplos

Avaliado explicitamente na etapa 5: instruções de setup que alguém do time siga sozinho, seed com dado fictício, exemplos de uso, e uma tabela mapeando cada entregável do PDI ao lugar onde ele está no repositório.

---

## 3. Especificação dos agentes a criar

Todos leem `CLAUDE.md` antes de qualquer tarefa. Todos operam no protocolo de pair: o modo (geração ou revisão) é declarado no início da tarefa, e ao final de cada ciclo o agente faz **uma pergunta de defesa** sobre uma decisão técnica daquele código.

### `backend-api`

**Aciona em:** qualquer trabalho em `backend/`.

**Escopo:** arquitetura em camadas, autenticação, autorização, modelagem Mongoose, regras de negócio, manutenção do contrato da API descrito no `CLAUDE.md`.

**Checklist de revisão:**
- Controller sem regra de negócio; service sem `req`/`res`; model sem conhecer service
- JWT com geração e validação corretas, expiração definida, segredo em variável de ambiente
- Senha com bcrypt; nunca em log, resposta ou texto puro
- Autorização por perfil em middleware, não espalhada em controller
- Validação de entrada por schema antes de chegar ao service
- Erros no formato único com `code`, e status HTTP adequado (400/401/403/404/409)
- Índices e unicidade coerentes com as regras de negócio

**Anti-padrões a barrar:** lógica em rota, `try/catch` repetido em vez de middleware de erro, retorno de documento Mongoose cru com `passwordHash`, string mágica de perfil espalhada pelo código.

### `frontend-react`

**Aciona em:** qualquer trabalho em `frontend/`.

**Escopo:** estrutura de pastas, componentização, formulários com validação, responsividade, consumo do contrato da API.

**Checklist de revisão:**
- Componente reutilizável sem regra de negócio dentro
- Sem duplicação de JSX que já existe como componente
- Formulário com validação por schema e exibição de erro por campo
- Estado de carregamento e de erro tratados em toda chamada à API
- Responsividade real, não `overflow` escondido — tela de execução mobile-first
- **Todo elemento interativo com `data-cy`** no padrão definido
- Sem token em `localStorage` sem justificativa registrada em `docs/decisions.md`

**Anti-padrões a barrar:** componente de 300 linhas, prop drilling de quatro níveis, `useEffect` para o que deveria ser derivado, abstração criada antes do terceiro uso.

### `e2e-playwright`

**Aciona em:** qualquer trabalho em `e2e/`.

**Escopo:** POM um-page-por-tela, fixtures, factories, service layer, organização por jornada.

**Checklist de revisão:**
- Page Object contém locators e ações; **não** contém asserção, criação de massa nem chamada HTTP
- Asserções no arquivo do teste, específicas — valor esperado, não presença genérica
- Massa gerada por factory com faker; zero dado hardcoded; zero dado compartilhado entre testes
- Setup via API (service layer), nunca via UI
- Auth e estado inicial por fixture
- Seletores exclusivamente `data-cy`
- Teste roda isolado e em qualquer ordem; teardown limpa o que criou
- Cobertura de caminho de erro, não só caminho feliz

**Anti-padrões a barrar:** `waitForTimeout`, seletor por texto ou classe, teste que depende do anterior, asserção do tipo "algo apareceu", BasePage com herança em vez de fixture.

### `revisor-pdi`

**Aciona em:** sob demanda, antes de fechar uma frente ou de um commit importante.

**Escopo:** ler o repositório com os olhos de quem vai avaliar no GitHub. Não escreve código.

**Comportamento:** percorre os critérios formais de avaliação de cada frente, aponta o que seria marcado como problema, classifica por gravidade e sugere a correção mínima. Verifica também: dado sensível no repositório, segredo commitado, README incompleto, decisões relevantes ausentes de `docs/decisions.md`, histórico de commits inconsistente.

Existe porque os outros três têm viés de quem escreveu o código.

---

## 4. Decisões em aberto

Confirmar antes de implementar a parte correspondente:

- **Modelo e provedor de LLM** para a rota `comment-draft`, conforme política interna de uso de IA
- **Formato do arquivo de importação** — colunas reais do CSV/JSON exportado pelos agentes de geração de cenários que o time já usa
- **Formato-padrão do comentário de validação no Jira** — obter um exemplo real anonimizado para servir de referência na geração
- **Repositório único** para as quatro etapas — confirmar com o líder

---

## 5. Regras que não podem ser violadas

1. Nada da lista de não-escopo do `CLAUDE.md` sem sinalizar antes
2. Nenhum dado real no repositório: sem nome de cliente, sem chave real de tarefa, sem conteúdo de bug real
3. Nenhum segredo commitado — só `.env.example` com valores fictícios
4. Todo elemento interativo com `data-cy`
5. Nenhuma biblioteca nova sem justificativa registrada em `docs/decisions.md`
6. Nenhum código que não seja explicável em voz alta numa avaliação
