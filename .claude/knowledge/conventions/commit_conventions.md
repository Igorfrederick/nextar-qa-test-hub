# Convenções — Commits e histórico

Fonte única. O histórico é parte do que será avaliado: um avaliador lê o `git log` antes de ler o código.

## Tipos

Conventional Commits: `feat:`, `fix:`, `test:`, `docs:`, `refactor:`, `chore:`

## Critério de corte

**Unidade funcional coerente e vertical.** O teste é direto: se o commit precisa do próximo para o projeto não quebrar, foi cortado cedo demais.

Commit por camada é proibido — "todos os models", depois "todos os controllers" descreve a mecânica do trabalho, não a entrega. Um commit atravessa as camadas que a funcionalidade exige.

## Regras

- **Regra de negócio e o teste que a prova viajam no mesmo commit.** Regra sem teste é commit incompleto.
- **`refactor:` nunca misturado com `feat:`.** Refatoração não muda comportamento; se mudou, não era refatoração.
- **A mensagem descreve o resultado, não a operação.** O Git já mostra o que mudou; a mensagem diz o que passou a ser verdade.
- **Corpo com o porquê** quando o commit materializa uma decisão. No resto, título sozinho basta.
- Não agregar o dia inteiro em um commit.
- Não reescrever histórico já empurrado.

## Branches

**Uma branch por frente, fechada com PR para a `main`** — mesmo trabalhando sozinho. A cerimônia é visível no histórico e custa pouco, enquanto commit direto na `main` precisaria ser justificado caso a caso na avaliação.

> **Exceção encerrada:** o setup inicial do repositório (`chore/setup-inicial`) foi a única entrada permitida sob exceção. A partir do Passo 2, a regra de branch vale integralmente, inclusive para documentação e configuração.

## `.gitkeep`

- Pasta vazia carrega `.gitkeep` apenas para existir no versionamento
- **Cada `.gitkeep` sai no mesmo commit em que a pasta recebe seu primeiro arquivo real**
- Pasta que chegar ao fim do projeto ainda com `.gitkeep` deve ser removida, não mantida — pasta vazia no entregável final é ruído

## Relação com `docs/decisions.md`

Commit que materializa decisão técnica relevante tem entrada correspondente no log de decisões, escrita na data em que a decisão foi tomada.

O log é mantido em **ordem cronológica inversa** — entrada mais recente no topo. Decisão revista nunca é apagada: entra uma nova entrada declarando que substitui a anterior.

Decisão alinhada com outra pessoa registra nome, papel e data do alinhamento.
