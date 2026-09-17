# Log de decisões técnicas

Registro das decisões tomadas no projeto, na data em que foram tomadas. Existe porque as decisões serão questionadas na avaliação, e resposta escrita na data vale mais do que justificativa reconstruída depois.

Formato de cada entrada: decisão, motivo, alternativa descartada.

**Ordem cronológica inversa** — entrada mais recente no topo. O log cresce ao longo do projeto e a decisão mais nova é a que tem maior chance de ser consultada; nova entrada vai logo abaixo deste cabeçalho.

---

## [17/09/2026] Agentes apontam para as convenções em vez de transcrevê-las

**Decisão:** as instruções dos agentes deixam de reproduzir convenção por extenso e passam ao formato **o que verificar + ponteiro para a fonte** (`arquivo.md §Seção`). O agente continua sabendo o que garantir; a regra por extenso vive num lugar só.

**Motivo:** transcrição cria cópia, e cópia depende de execução manual perfeita para permanecer sincronizada. A regra de propagação reduz o risco mas não o elimina — ela já falhou uma vez com a regra no lugar, no mesmo dia em que foi escrita.

O dado que sustenta a decisão: os blocos do `code-reviewer` foram convertidos para esse formato antes dos demais agentes, e **nenhum achado das revisões seguintes caiu nos blocos convertidos** — todos caíram em trechos ainda transcritos. Apontar elimina a classe de achado; corrigir transcrição elimina a instância.

**Alternativa descartada:** manter a transcrição e confiar na regra de propagação. Descartada pela evidência acima. A regra continua valendo — ela cobre o que não dá para transformar em ponteiro, como a tabela de critérios reproduzida no `revisor-pdi`.

**Cuidado de execução registrado:** ponteiro sem contexto vira indireção vazia, e ponteiro quebrado é pior que transcrição desatualizada — a transcrição ao menos diz algo errado que dá para notar; o ponteiro quebrado não diz nada. Por isso cada linha mantém **o que verificar** antes da fonte, e todo `§` referenciado é conferido contra o heading real.

---

## [17/09/2026] Verificação por leitor sem contexto é parte do mecanismo

**Decisão:** a regra de propagação se completa com uma revisão feita por um leitor **sem histórico da conversa que produziu a mudança**, recebendo apenas os arquivos. Não é etapa opcional nem cerimônia — é o que torna a regra verificável.

**Motivo:** **regra nova falha primeiro com quem a escreveu.** Quem acabou de escrever tem o conteúdo na cabeça e lê a intenção em vez do texto — confirma o que quis dizer, não o que ficou escrito. Foi o que aconteceu aqui: a regra de propagação foi escrita, aplicada a um agente, e violada em outro no mesmo commit, sem que a auto-revisão percebesse.

A evidência está na curva das revisões independentes: 15 achados, depois 12, depois 5 — cada rodada encontrando a regra da rodada anterior aplicada incompletamente. Nenhuma delas teria acontecido em auto-revisão.

**Alternativa descartada:** confiar na revisão de quem escreveu, com checklist. Descartada porque o checklist também é lido com a intenção na cabeça. O problema não é falta de rigor; é que o autor não consegue simular desconhecimento do próprio texto.

---

## [17/09/2026] Regra de propagação passa a cobrir `.claude/agents/`

**Decisão:** alteração na rubrica do PDI, na tabela de critérios da seção 1 do `CLAUDE.md` ou em `docs/decisions.md` exige verificar a propagação para `.claude/knowledge/` **e** `.claude/agents/`, no mesmo commit. Omissão é achado HIGH no BLOCO 9 do `code-reviewer` e item da varredura do `revisor-pdi`.

**Motivo — a causa raiz, registrada:** o mecanismo de propagação anterior cobria apenas artefatos de **conhecimento** (convenções e checklists). Os **agentes são consumidores desse conhecimento e não eram verificados** — e vários deles transcrevem critérios em prosa dentro das próprias instruções, o que os torna cópias que envelhecem.

A falha se materializou no mesmo dia em que a regra foi escrita: a correção da transcrição da rubrica chegou à tabela do `CLAUDE.md` e ao `checklist_pdi.md`, mas não à lista de critérios do `revisor-pdi` — justamente o agente encarregado de auditar contra a rubrica. Nove entregáveis ficaram fora do único lugar onde seriam cobrados.

O gatilho também foi ampliado: a regra anterior disparava apenas em "decisão nova em `docs/decisions.md`". O que falhou hoje não foi uma decisão, e sim a correção de uma **fonte externa** — a rubrica. Gatilho estreito deixa a próxima variante escapar.

**Alternativa descartada:** confiar em que o autor lembre de propagar. Foi o que falhou duas vezes no mesmo dia, uma delas enquanto se corrigia a primeira. Regra sem verificação automática é intenção, não mecanismo.

**Consequência:** o `revisor-pdi` ganha também a verificação de paráfrase — checklist que apenas repete a convenção em outras palavras, sem acrescentar verificabilidade, deve ser removido. É dessincronia que aparece com o tempo, não no diff de um PR.

---

## [17/09/2026] Classificação das regras de negócio por camada e status

**Decisão:** invariante de entrada valida por schema e retorna `400`; invariante de domínio valida no service e retorna `409`.

**Motivo:** o critério é a dependência de estado, não a natureza da regra. Validação que precisa consultar o banco não pertence ao middleware. Sem esse critério o contrato ficava ambíguo: as oito regras estavam todas marcadas `409`, mas as regras 4 e 5 são forma do payload e o middleware de validação devolveria `400` antes de o service chegar ao `409` declarado — backend e E2E divergiriam.

**Alternativa descartada:** tratar todas as oito como `409` no service, o que exigiria duplicar no service validações que o schema já garante.

**Consequência:** regras `400` asserem `VALIDATION_ERROR` mais o campo em `details`; regras `409` têm `code` específico. A numeração das oito regras não muda — já está referenciada neste log, nos agentes e em commits.

---

## [17/09/2026] Suporte a mobile e desktop, com desktop-first na ordem de trabalho

**Decisão:** a interface suporta mobile e desktop, e nenhum dos dois pode quebrar. Desktop-first é a ordem de trabalho — o alvo que guia as decisões de layout —, não dispensa de suporte a mobile.

**Motivo:** a entrada anterior deste mesmo dia excedeu o escopo. Ela negou o requisito mobile ("não há requisito mobile-first") quando o entregável formal da rubrica pede **interface responsiva (mobile e desktop)**. A distinção entre **ordem de design** e **suporte de plataforma** é o que torna a correção defensável: abandonar mobile-first como ordem de trabalho é legítimo e continua valendo; abandonar o suporte a mobile eliminaria um critério pelo qual o projeto será avaliado.

**Alternativa descartada:** editar a entrada anterior para corrigi-la. Descartada pela convenção do próprio log — decisão revista nunca é apagada; entra entrada nova declarando o que substitui. O erro e a correção são ambos parte do registro.

**Substitui:** a frase "a prioridade de design passa a ser densidade (…) não área de toque" da entrada "Tela de execução deixa de ser mobile-first", na parte em que ela foi lida como dispensa de suporte a mobile. A prioridade de densidade permanece válida; ela convive com o suporte responsivo.

---

## [17/09/2026] Transcrição da rubrica do PDI corrigida no `CLAUDE.md`

**Decisão:** a tabela de critérios da seção 1 do `CLAUDE.md` passa a transcrever a rubrica formal do PDI por inteiro. Entram os entregáveis que faltavam: interface responsiva, formulários com validação e tela de login integrada (frontend); API REST funcional, middleware de validação e autorização e conexão com MongoDB (backend); suíte de login e autenticação, suíte de funcionalidades principais, **testes isolados de backend**, organização por feature ou jornada, independência entre testes e setup/teardown apropriados (E2E).

**Motivo:** a transcrição original estava incompleta, e a incompletude era invisível por construção: o `checklist_pdi.md` deriva dessa tabela, e o `revisor-pdi` — único agente encarregado de auditar contra a rubrica — lê o checklist, não a rubrica. Entregável ausente da tabela ficava ausente de toda a cadeia de verificação. O caso mais grave era **testes isolados de backend**, que não tinha convenção, checklist, ferramenta nem agente responsável, e cuja ausência tornava inexequível a regra de commit "regra de negócio e o teste que a prova viajam no mesmo commit".

**O que permitiu a correção:** a cláusula de precedência escrita no próprio `checklist_pdi.md` — *"se a rubrica formal divergir deste arquivo, a rubrica vence e este arquivo é corrigido"*. Fonte externa vence derivado interno. A cláusula foi escrita justamente por se suspeitar que a transcrição pudesse estar incompleta, e esta é a primeira vez que é acionada.

**Alternativa descartada:** corrigir apenas os checklists, sem mexer na tabela de critérios. Descartada porque deixaria a fonte errada e o derivado certo — a próxima pessoa a regenerar um checklist a partir da tabela reintroduziria a lacuna.

**Consequência registrada:** os testes de backend passam a ser escopo do `backend-api`, e o service ganha requisito explícito de testabilidade — exercitável sem HTTP e sem subir a aplicação. A ferramenta de teste permanece decisão em aberto para o Passo 3.

---

## [17/09/2026] Tela de execução deixa de ser mobile-first

**Decisão:** a aplicação é usada em desktop, ao lado de outras ferramentas de QA da equipe. A tela de execução (`/runs/:id`) deixa de ter requisito mobile-first. Responsividade continua exigida — layout que reflui, sem `overflow` escondendo conteúdo — mas a prioridade de design passa a ser densidade de informação e ação rápida sobre caso de teste, não área de toque.

**Motivo:** a premissa original era que o QA executaria o teste em dispositivo físico, com as mãos ocupadas. Na prática, a aplicação é usada no computador, em complemento a outras ferramentas de QA desenvolvidas pela equipe. Projetar para um cenário de uso que não existe custaria decisões de layout — alvos de toque grandes, uma coluna, menos informação por tela — que pioram o uso real.

**Alternativa descartada:** manter mobile-first por segurança, caso o uso em dispositivo apareça depois. Descartada porque mobile-first não é um extra que se carrega sem custo: ele determina a ordem das decisões de layout desde o primeiro componente.

**Substitui:** a observação "Mobile-first — QA executa teste em dispositivo físico com as mãos ocupadas" da tabela de telas do `CLAUDE.md`. O `docs/handoff.md` preserva a premissa original como registro histórico; onde houver conflito, esta entrada vence.

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
