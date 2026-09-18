# Checklist — E2E

Aplicar a qualquer diff que toque `e2e/**`. Base: `conventions/e2e_conventions.md`.

> Page Object Model é a decisão deste projeto, registrada em `docs/decisions.md`. Page Object **não** é violação aqui.

## Camadas (CRITICAL)

- [ ] Nenhum Page Object contém asserção
- [ ] Nenhum Page Object cria massa de dados
- [ ] Nenhum Page Object faz chamada HTTP
- [ ] Um Page Object por tela — sem page de componente nem de fluxo
- [ ] Setup por API (service layer), nunca pela interface

## Seletores

- [ ] Exclusivamente `data-cy`
- [ ] Nenhum seletor por classe CSS, texto visível, posição no DOM ou hierarquia de tags

## Asserções

- [ ] Vivem no arquivo do teste, não no Page Object
- [ ] **Verificam a coisa certa** — o efeito real, não um sintoma lateral
- [ ] Específicas: valor esperado, não `toBeVisible()` genérico
- [ ] Todo fluxo termina em asserção
- [ ] Erro de API asseverado por `code`, nunca por mensagem em português

## Massa de dados

- [ ] Gerada por factory com faker
- [ ] **Zero dado hardcoded**
- [ ] Cada teste gera a própria massa
- [ ] Identificador que precisa ser único carrega entropia — sem nome fixo que colida entre workers

## Independência

- [ ] Nenhum teste depende de outro
- [ ] Nenhum teste depende da ordem de execução
- [ ] Nenhum dado compartilhado entre testes
- [ ] A suíte passaria com `--shuffle` e em paralelo

## Setup e teardown

- [ ] Autenticação por fixture, por perfil
- [ ] Page objects injetados por fixture, sem herança de BasePage
- [ ] O teste limpa o que criou

## Determinismo

- [ ] Nenhum `waitForTimeout`
- [ ] Sem `if/else` no caminho principal do teste
- [ ] Sem dependência de dado pré-existente no ambiente

## Cobertura das regras de negócio

Caminho de erro tem o mesmo peso do caminho feliz. As oito regras do `CLAUDE.md` precisam de cobertura, mas **nem todas se testam da mesma forma**.

### Testam-se violando — a regra deve falhar

| # | O que o teste provoca | Espera |
|---|---|---|
| 1 | Importar cenário com `externalRef` já existente na suíte | `409`, e o caso **não** duplicado |
| 2 | Excluir suíte que possui TestRun registrado | `409` |
| 3 | Registrar execution em TestRun `closed` | `409` |
| 4 | Marcar `pass` ou `fail` sem evidência | `400` com `VALIDATION_ERROR` e o campo em `details` |
| 5 | Marcar `fail` ou `blocked` sem `notes` | `400` com `VALIDATION_ERROR` e o campo em `details` |
| 7 | Fechar TestRun com casos não executados | `409` |
| 8 | Pedir rascunho com casos pendentes | `409` |

### Testa-se observando o comportamento — a regra deve funcionar

| # | O que o teste faz | Espera |
|---|---|---|
| 6 | Marcar status duas vezes para o mesmo par (`testRunId`, `testCaseId`) | A segunda marcação **atualiza**; permanece uma única execution, com o status novo |

A regra 6 não é violação — é idempotência. Testar "o que acontece se violar" não se aplica: o que se verifica é que a segunda chamada não duplica.

### Além das regras

- [ ] Acesso negado por perfil coberto — `qa` tentando ação de `lead` responde `403`
- [ ] Token ausente ou inválido responde `401`, distinto do `403`
- [ ] Erro asseverado pelo `code`, nunca pela mensagem em português
