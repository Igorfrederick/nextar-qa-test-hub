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
| Frontend | Estrutura de pastas, componentização e reutilização, boas práticas, interface responsiva (mobile e desktop), formulários com validação, tela de login integrada com o backend |
| Backend | API REST funcional, arquitetura em camadas, JWT correto (geração e validação), middleware de validação e autorização, conexão com MongoDB, hash de senhas, variáveis de ambiente, modelagem de dados |
| E2E | Suíte cobrindo login e autenticação, suíte cobrindo funcionalidades principais, testes isolados de backend, organização por feature ou jornada, qualidade dos seletores, asserções específicas, independência entre testes, setup e teardown apropriados |
| Aplicação da solução | Código executável, organização, README claro, facilidade de uso |

Esta tabela é a transcrição da rubrica formal do PDI. Onde ela divergir da rubrica, a rubrica vence e esta tabela é corrigida — ela é a fonte de `checklists/checklist_pdi.md`, e o que falta aqui fica invisível para o `revisor-pdi`.

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
| Execução do ciclo | `/runs/:id` | Tela de maior uso — o QA a mantém aberta ao lado de outras ferramentas |
| Fechamento do ciclo e rascunho | `/runs/:id/summary` | Dashboard de cobertura + geração do comentário |

Seis telas, seis Page Objects. Não criar tela nova sem sinalizar.

---
## 4. Stack

- **Frontend:** React + Vite, React Router, React Hook Form + Zod
- **Backend:** Node.js + Express, MongoDB + Mongoose, JWT, bcrypt
- **E2E:** Playwright + TypeScript
- **Idioma:** identificadores de código, entidades e campos em **inglês**; textos de interface em **português**. Sem mistura dentro de um mesmo identificador.

### Segurança — não negociável

- Senha sempre com hash (bcrypt). Nunca em texto puro, nunca em log, nunca em resposta de API.
- Segredos só via variável de ambiente. Nenhum valor real commitado; `.env.example` com chaves e valores fictícios.
- Nenhum dado real no repositório: sem nome de cliente, sem chave real de tarefa do Jira, sem conteúdo de bug real. Seed usa dado fictício.

---

## 5. Convenções — base de conhecimento

As convenções detalhadas vivem em `.claude/knowledge/`, fonte única lida por todos os agentes. Este arquivo permanece como camada de contexto: domínio, escopo, telas, perfis, regras de negócio e protocolo de trabalho.

### Convenções

| Arquivo | Cobre |
|---|---|
| `conventions/backend_conventions.md` | Camadas, JWT, bcrypt, env, erros, modelagem |
| `conventions/frontend_conventions.md` | Estrutura, componentização, formulários, responsividade, `data-cy` |
| `conventions/e2e_conventions.md` | POM, fixtures, factories, service layer, independência |
| `conventions/api_contract.md` | Rotas, formato de erro, códigos, status HTTP |
| `conventions/commit_conventions.md` | Tipos, critério de corte, branches, `.gitkeep` |

### Checklists

| Arquivo | Usado por |
|---|---|
| `checklists/checklist_backend.md` | `backend-api`, `code-reviewer` |
| `checklists/checklist_frontend.md` | `frontend-react`, `code-reviewer` |
| `checklists/checklist_e2e.md` | `e2e-playwright`, `code-reviewer` |
| `checklists/checklist_pdi.md` | `revisor-pdi` |

**Regra de manutenção:** quando uma decisão nova entrar em `docs/decisions.md`, verificar se ela altera algum arquivo de `.claude/knowledge/`. Convenção que evolui sem o reviewer saber vira ruído silencioso.

### Dois pontos que não saem daqui

**Seletores.** Todo elemento com o qual o teste interage carrega `data-cy`, no padrão `contexto-elemento[-identificador]`, kebab-case. Componente novo sem `data-cy` está incompleto. Detalhe em `frontend_conventions.md`.

**Page Object Model, um Page Object por tela.** Decisão alinhada com o tech lead, registrada em `docs/decisions.md`. Seis telas, seis Page Objects. Convenção de outro projeto que trate Page Object como violação não se aplica aqui. Detalhe em `e2e_conventions.md`.

---

## 6. Estrutura de pastas

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
  tests/           testes automatizados de service e API, sem UI

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

.claude/
  agents/            cinco agentes
  knowledge/         convenções e checklists

docs/
  decisions.md       log de decisões técnicas
```

**Regra de camada no backend:** controller não contém regra de negócio; service não conhece `req`/`res`; model não conhece service. Violação disso é o erro mais visível na avaliação de "arquitetura em camadas".

---

## 7. Agentes

Cinco agentes em `.claude/agents/`. A divisão é por **unidade de análise e momento**, não por assunto.

| Agente | Analisa | Quando | Escreve código? |
|---|---|---|---|
| `backend-api` | Código em construção em `backend/` | Durante o trabalho, em pair | Sim |
| `frontend-react` | Código em construção em `frontend/` | Durante o trabalho, em pair | Sim |
| `e2e-playwright` | Código em construção em `e2e/` | Durante o trabalho, em pair | Sim |
| `code-reviewer` | O **diff** de um PR, nas três frentes | Antes do merge | Não |
| `revisor-pdi` | O **repositório inteiro** contra a rubrica | Antes de fechar uma frente | Não |

Todos leem `.claude/knowledge/` — nenhum carrega checklist próprio.

---

## 8. Log de decisões

Toda decisão técnica relevante vai para `docs/decisions.md`, uma entrada curta:

```
## [data] Título da decisão
**Decisão:** o que foi decidido
**Motivo:** por quê
**Alternativa descartada:** o que não foi escolhido e por quê
```

Isso existe porque as decisões serão questionadas na avaliação. Resposta escrita na data em que foi tomada vale mais do que justificativa reconstruída depois.

**Ordem cronológica inversa** — entrada mais recente no topo. O log cresce ao longo do projeto e a decisão mais nova é a que tem maior chance de ser consultada.

Quando a decisão foi alinhada com outra pessoa, registrar nome, papel e data do alinhamento.

---

## 9. Protocolo de trabalho com os agentes

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

## 10. Decisões em aberto

Confirmar antes de implementar a parte correspondente:

- Modelo e provedor de LLM para a rota `comment-draft`, conforme política interna de uso de IA
- Formato exato do arquivo de importação (colunas esperadas no CSV/JSON gerado pelos agentes atuais)
- Formato-padrão do comentário de validação no Jira, a ser usado como referência na geração do rascunho
- Ferramenta de teste automatizado para `backend/tests/` — a convenção é agnóstica; a escolha entra no Passo 3 e exige registro em `docs/decisions.md`
