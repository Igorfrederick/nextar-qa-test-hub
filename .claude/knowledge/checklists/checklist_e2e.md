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

## Cobertura

- [ ] Caminho de erro coberto, não só o caminho feliz
- [ ] Regras de negócio violadas nos testes: ciclo fechado, evidência obrigatória, `notes` obrigatório, fechamento com pendências
- [ ] Acesso negado por perfil coberto
