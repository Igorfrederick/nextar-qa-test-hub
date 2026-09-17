# Checklist — Critérios de avaliação do PDI

Usado pelo `revisor-pdi` para ler o repositório inteiro com os olhos de quem vai avaliar no GitHub.

Os itens abaixo derivam da tabela de critérios da seção 1 do `CLAUDE.md`, que é a transcrição da rubrica formal do PDI.

> Se a rubrica formal divergir deste arquivo, a rubrica vence e este arquivo é corrigido. Esta cláusula já foi acionada em 17/09/2026, quando a transcrição se mostrou incompleta — ver `docs/decisions.md`.

## Convenção de marcação

| Marca | Significado |
|---|---|
| `- [ ]` | Deveria estar pronto e não está, ou não foi verificado |
| `- [x]` | Verificado e conforme |
| `[pendente: Passo N]` | Depende de etapa futura do projeto; **não é falha** |

A distinção importa: item que depende do Passo 3 não é a mesma coisa que item que deveria estar pronto e não está. Um report que não separa os dois vira ruído e deixa de ser lido.

Enquanto o repositório tiver apenas documentação, a maior parte destes itens estará marcada como pendente por etapa. Isso é o estado correto — checklist que só mostra o que já está pronto não serve para nada.

---

## Frontend

Critério: estrutura de pastas, componentização e reutilização, boas práticas, interface responsiva (mobile e desktop), formulários com validação, tela de login integrada com o backend.

- [ ] Estrutura de pastas corresponde à seção 6 do `CLAUDE.md` `[pendente: Passo 4]`
- [ ] Uma pasta por tela em `pages/` `[pendente: Passo 4]`
- [ ] Componentes reutilizáveis existem e são de fato reutilizados `[pendente: Passo 4]`
- [ ] Nenhuma duplicação de JSX que já exista como componente `[pendente: Passo 4]`
- [ ] **Formulários com validação por schema** `[pendente: Passo 4]`
- [ ] **Tela de login integrada com o backend**, consumindo `POST /auth/login` `[pendente: Passo 4]`
- [ ] **Interface responsiva: utilizável em mobile e desktop**, sem quebra de layout nem conteúdo inacessível `[pendente: Passo 4]`
- [ ] Estados de carregamento e erro tratados `[pendente: Passo 4]`
- [ ] Nenhum elemento interativo sem `data-cy` `[pendente: Passo 4]`

## Backend

Critério: API REST funcional, arquitetura em camadas, JWT correto, middleware de validação e autorização, conexão com MongoDB, hash de senhas, variáveis de ambiente, modelagem de dados.

- [ ] **API REST funcional** — todas as rotas do contrato respondendo `[pendente: Passo 3]`
- [ ] Separação de camadas visível na estrutura e respeitada no código `[pendente: Passo 3]`
- [ ] Controller sem regra, service sem `req`/`res`, model sem service `[pendente: Passo 3]`
- [ ] JWT com geração **e** validação corretas, expiração definida `[pendente: Passo 3]`
- [ ] **Middleware de validação** aplicado antes do service `[pendente: Passo 3]`
- [ ] **Middleware de autorização por perfil** em toda rota marcada `[lead]` `[pendente: Passo 3]`
- [ ] **Conexão com MongoDB** isolada em `config/`, URI por env, falha tratada `[pendente: Passo 3]`
- [ ] Senha com bcrypt; `passwordHash` nunca exposto `[pendente: Passo 3]`
- [x] Segredos exclusivamente por variável de ambiente
- [x] `.env.example` presente e completo, com valores fictícios
- [ ] Modelagem coerente com as regras de negócio — unicidade e índices onde a regra exige `[pendente: Passo 3]`
- [ ] Formato de erro único, com `code` e status HTTP adequado `[pendente: Passo 3]`

## E2E

Critério: suíte cobrindo login e autenticação, suíte cobrindo funcionalidades principais, testes isolados de backend, organização por feature ou jornada, qualidade dos seletores, asserções específicas, independência entre testes, setup e teardown apropriados.

- [ ] **Suíte cobrindo login e autenticação**, incluindo acesso negado por perfil `[pendente: Passo 5]`
- [ ] **Suíte cobrindo as funcionalidades principais:** CRUD de projeto, suíte e caso; importação com deduplicação; ciclo completo de execução; dashboard de cobertura; geração do rascunho de comentário `[pendente: Passo 5]`
- [ ] **Testes isolados de backend** em `backend/tests/` — service e API, sem navegador `[pendente: Passo 3]`
- [ ] **Organização por feature ou jornada** do usuário `[pendente: Passo 5]`
- [ ] Um Page Object por tela `[pendente: Passo 5]`
- [ ] Seletores exclusivamente `data-cy` `[pendente: Passo 5]`
- [ ] Asserções específicas, no arquivo do teste `[pendente: Passo 5]`
- [ ] Cobertura das regras de negócio que falham `[pendente: Passo 5]`
- [ ] **Independência entre testes** — sem dependência de ordem ou de estado `[pendente: Passo 5]`
- [ ] **Setup e teardown apropriados** — setup via API, teardown limpando o que criou `[pendente: Passo 5]`

## Aplicação da solução

Critério: código executável, organização, README claro, facilidade de uso.

- [ ] **Código executável** — o projeto sobe seguindo apenas o README `[pendente: Passo 6]`
- [ ] README com setup, seed e execução das três frentes `[pendente: Passo 6]`
- [ ] Seed com dado fictício `[pendente: Passo 6]`
- [ ] Tabela mapeando cada entregável do PDI ao lugar no repositório `[pendente: Passo 6]`
- [ ] Organização geral coerente entre as três frentes `[pendente: Passo 6]`

## Transversais

Verificáveis desde já, em qualquer estágio.

- [ ] Nenhum dado real da Nextar: nome de cliente, chave real de tarefa, conteúdo de bug real
- [ ] Nenhum segredo commitado
- [ ] `docs/decisions.md` cobre as decisões relevantes, em ordem cronológica inversa
- [ ] Decisão alinhada com outra pessoa registra nome, papel e data
- [ ] Decisão revista tem entrada nova declarando o que substitui, em vez de edição da anterior
- [ ] Histórico de commits coerente com `conventions/commit_conventions.md`
- [ ] Nenhum `.gitkeep` remanescente em pasta que já tem arquivo real
- [ ] Nada da lista de não-escopo do v1 implementado
- [ ] Cada decisão de `docs/decisions.md` está refletida nas convenções e nos checklists
