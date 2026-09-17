# Log de decisões técnicas

Registro das decisões tomadas no projeto, na data em que foram tomadas. Existe porque as decisões serão questionadas na avaliação, e resposta escrita na data vale mais do que justificativa reconstruída depois.

Formato de cada entrada: decisão, motivo, alternativa descartada.

**Ordem cronológica inversa** — entrada mais recente no topo. O log cresce ao longo do projeto e a decisão mais nova é a que tem maior chance de ser consultada; nova entrada vai logo abaixo deste cabeçalho.

---

## [17/09/2026] Convenções extraídas para `.claude/knowledge/`

**Decisão:** as convenções do projeto saem do `CLAUDE.md` e passam a viver em `.claude/knowledge/`, divididas em cinco arquivos de convenção e quatro checklists. O `CLAUDE.md` permanece como camada de contexto — domínio, escopo, telas, perfis, regras de negócio e protocolo — e referencia os arquivos em vez de descrevê-los por extenso.

**Motivo:** na especificação original, cada agente carregava seu próprio checklist de revisão. Isso duplicava a mesma convenção em vários arquivos, e convenção duplicada diverge em silêncio: corrigir em um lugar e esquecer o outro produz dois agentes com regras diferentes sem que ninguém perceba. Com fonte única, a atualização chega a todos os leitores ao mesmo tempo.

**Alternativa descartada:** manter tudo no `CLAUDE.md`. O arquivo já passava de trezentas linhas e continuaria crescendo; misturar contexto de domínio com detalhe de convenção torna caro localizar qualquer um dos dois.

**Regra de manutenção associada:** decisão nova em `docs/decisions.md` exige verificar se altera algum arquivo de `.claude/knowledge/`.

---

## [17/09/2026] Cinco agentes, divididos por unidade de análise

**Decisão:** o projeto passa a ter cinco agentes em vez de quatro. Entra um `code-reviewer` dedicado à análise de diff de PR, ao lado do `revisor-pdi` que já existia. A divisão entre os cinco é por **unidade de análise e momento**, não por assunto: três construtores atuam durante o trabalho, em pair, cada um em sua frente; o `code-reviewer` analisa o diff antes do merge; o `revisor-pdi` analisa o repositório inteiro antes de fechar uma frente.

**Motivo:** revisar um diff e avaliar um repositório são tarefas diferentes com referências diferentes. O `code-reviewer` compara código novo contra as convenções; o `revisor-pdi` compara o conjunto contra a rubrica de avaliação e pergunta o que seria marcado numa primeira leitura. Juntar os dois num agente só produziria um revisor que faz mal as duas coisas, porque o escopo de leitura e o critério de saída são incompatíveis.

**Alternativa descartada:** manter quatro agentes, com o `revisor-pdi` acumulando a revisão de PR. Descartada porque a revisão de diff acontece com frequência e precisa ser barata, enquanto a leitura do repositório inteiro é cara e acontece poucas vezes — misturar as duas faria uma delas ser sempre executada no ritmo errado.

**Consequência:** os três agentes construtores perdem os checklists embutidos, que passam a ser lidos de `.claude/knowledge/`.

---

## [15/09/2026] Sequência de construção: backend → frontend → E2E

**Decisão:** construir o backend até o contrato da API estar estável antes de iniciar o frontend, e o E2E por último.

**Motivo:** frontend e E2E consomem o contrato. Mudança de contrato depois de ambos escritos custa retrabalho em três lugares em vez de um.

**Alternativa descartada:** construir as três frentes em paralelo por fatia vertical de funcionalidade. Seria defensável com contrato fechado antecipadamente, mas aqui o contrato ainda tem pontos em aberto — notadamente o formato do arquivo de importação.

---

## [15/09/2026] Escopo de IA restrito à geração de rascunho de comentário

**Decisão:** no v1, a única capacidade que usa LLM é a rota `POST /runs/:id/comment-draft`, que gera rascunho de comentário de validação a partir dos dados de execução do ciclo. O rascunho passa por revisão humana e é copiado à mão para o Jira.

**Motivo:** manter uma única superfície de contato com LLM deixa o custo, a latência e o risco concentrados em um ponto auditável. A revisão humana obrigatória mantém a responsabilidade pelo conteúdo publicado com o QA, não com o modelo.

**Alternativa descartada:** publicação automática do comentário via API do Jira. Descartada para o v1 — adiciona superfície de integração e de credencial sem resolver o problema central, que é redigir o comentário. Escrita no Jira está na lista de não-escopo.

---

## [15/09/2026] Erros da API asseverados por `code`, não por mensagem

**Decisão:** toda resposta de erro da API segue o formato único `{ error: { code, message, details } }`. Os testes E2E asseveram sobre o campo `code`; nunca sobre a mensagem em português.

**Motivo:** a mensagem é camada de apresentação e o `code` é contrato. Asseverar a mensagem tornaria a suíte frágil — uma alteração cosmética de texto quebraria testes sem que nada de comportamental tivesse mudado. O `code` é inequívoco e nasce de um único lugar. A mensagem continua testada, mas em nível de unidade, sobre o catálogo de erros: a cobertura não se perde, muda de lugar.

**Alternativa descartada:** asseverar a mensagem exibida ao usuário, por ser o que ele de fato vê. Descartada por acoplar o teste E2E a texto de interface, que é o tipo de acoplamento que mais gera falso positivo em suíte de regressão.

---

## [15/09/2026] Repositório único para as etapas 1, 2, 3 e 5 do PDI

**Decisão:** um único repositório atende às etapas 1, 2, 3 e 5. A etapa 4 é um documento de análise de oportunidade e vive fora daqui.

**Motivo:** a avaliação é por análise do código no GitHub. Com backend, frontend e E2E no mesmo repositório, a integração entre as frentes fica visível — dá para ver o contrato da API sendo consumido pelo frontend e asseverado pelo E2E, o que é justamente o que se quer demonstrar. Repositórios separados esconderiam essa costura e obrigariam o avaliador a correlacionar três históricos de commits independentes.

**Alternativa descartada:** um repositório por etapa. Fragmentaria o histórico de commits, que é parte do que será avaliado, e exigiria duplicar CLAUDE.md, README e log de decisões em três lugares, com risco de divergirem.

**Decidido por:** Igor Frederick, em 15/09/2026.

---

## [15/09/2026] Page Object Model com um Page Object por tela

**Decisão:** adotar Page Object Model na suíte E2E, com exatamente um Page Object por tela da aplicação — seis telas, seis Page Objects. O Page Object contém locators e ações de baixo nível; asserções ficam no arquivo do teste, massa de dados em factory, setup via service layer e autenticação em fixture.

**Motivo:** o critério de avaliação da frente E2E cita explicitamente organização dos testes e qualidade dos seletores. Um Page Object por tela cria correspondência direta entre a estrutura de `e2e/pages/` e as telas da aplicação, o que torna a organização legível sem precisar de explicação. Manter asserção fora do Page Object preserva o Arrange-Act-Assert visível no arquivo do teste — quem lê o teste entende o que está sendo verificado sem abrir outro arquivo.

**Alternativa descartada:** Page Object por componente ou por fluxo. Por componente fragmenta demais e faz um teste depender de muitos objetos, perdendo a legibilidade que o padrão deveria trazer. Por fluxo mistura navegação de telas diferentes no mesmo objeto e reintroduz o acoplamento que o POM existe para evitar.

**Alinhado com:** Murilo Morato, tech lead, em 15/09/2026 às 11:22.
