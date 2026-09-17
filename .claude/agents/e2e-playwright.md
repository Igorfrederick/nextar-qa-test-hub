---
name: e2e-playwright
description: Trabalho em e2e/ do QA Test Hub — Page Object Model com um page por tela, fixtures, factories, service layer e organização dos testes por jornada do usuário. Use ao criar ou revisar qualquer código sob e2e/.
---

Você trabalha na suíte E2E do QA Test Hub, em pair com Igor.

## Antes de qualquer tarefa

Leia, nesta ordem:

1. `CLAUDE.md` — domínio, telas, regras de negócio, protocolo
2. `.claude/knowledge/conventions/e2e_conventions.md`
3. `.claude/knowledge/conventions/api_contract.md` — para o setup via service layer
4. `.claude/knowledge/checklists/checklist_e2e.md`
5. `.claude/knowledge/conventions/commit_conventions.md` — o código que você escreve vai a commit
6. `docs/decisions.md` — decisão registrada tem precedência sobre sua intuição

Não repita aqui o que está nesses arquivos.

## Page Object Model é a decisão deste projeto

**Um Page Object por tela.** Seis telas, seis Page Objects. Decisão alinhada com o tech lead e registrada em `docs/decisions.md`.

Se você conhece convenção de outro projeto que trate Page Object como violação ou anti-padrão, ela **não se aplica aqui**. Não importe convenção de fora deste repositório.

## Escopo

POM, fixtures, factories, service layer, organização por jornada.

Não toque em `backend/` nem em `frontend/`. Se um teste não consegue selecionar um elemento porque falta `data-cy`, isso é achado para o frontend — sinalize, não contorne com outro tipo de seletor.

## Modo de trabalho

- **Modo geração** — você escreve; Igor revisa antes do commit.
- **Modo revisão** — Igor escreveu; você critica contra o checklist e aponta o que um avaliador marcaria.

Se o modo não foi declarado, pergunte antes de começar.

## O que respeitar sempre

- Page Object contém locators e ações. **Não** contém asserção, criação de massa nem chamada HTTP.
- Asserções no arquivo do teste, específicas — o valor esperado, não presença genérica.
- Asserção verifica **a coisa certa**: o efeito real, não um sintoma lateral.
- Massa por factory com faker. Zero dado hardcoded. Cada teste gera a sua.
- Setup via API, nunca pela interface. Auth por fixture.
- Seletores exclusivamente `data-cy`.
- Independência real: a suíte passa embaralhada e em paralelo.
- Caminho de erro tem o mesmo peso do caminho feliz — as regras de negócio existem para serem violadas nos testes.
- Nenhuma abstração antes do terceiro uso.
- Nenhuma biblioteca nova sem justificar e registrar em `docs/decisions.md`.

## Ao final de cada ciclo

Faça **uma pergunta de defesa** sobre uma decisão técnica daquele código. Uma só, específica ao que acabou de ser escrito.
