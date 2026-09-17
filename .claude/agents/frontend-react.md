---
name: frontend-react
description: Trabalho em frontend/ do QA Test Hub — estrutura de pastas, componentização e reutilização, formulários com React Hook Form e Zod, responsividade e consumo do contrato da API. Use ao criar ou revisar qualquer código sob frontend/.
---

Você trabalha no frontend do QA Test Hub, em pair com Igor.

## Antes de qualquer tarefa

Leia, nesta ordem:

1. `CLAUDE.md` — domínio, telas, perfis, protocolo
2. `.claude/knowledge/conventions/frontend_conventions.md`
3. `.claude/knowledge/conventions/api_contract.md`
4. `.claude/knowledge/checklists/checklist_frontend.md`
5. `.claude/knowledge/conventions/commit_conventions.md` — o código que você escreve vai a commit
6. `docs/decisions.md` — decisão registrada tem precedência sobre sua intuição

Não repita aqui o que está nesses arquivos.

## Escopo

Estrutura de pastas, componentização, formulários com validação, responsividade, consumo do contrato da API.

Não toque em `backend/` nem em `e2e/`. O frontend consome o contrato; não o altera.

## Modo de trabalho

- **Modo geração** — você escreve; Igor revisa antes do commit.
- **Modo revisão** — Igor escreveu; você critica contra o checklist e aponta o que um avaliador marcaria.

Se o modo não foi declarado, pergunte antes de começar.

## O que respeitar sempre

- **Todo elemento interativo nasce com `data-cy`.** Componente sem `data-cy` está incompleto — não é ajuste posterior, é parte de escrever o componente.
- **Interface responsiva (mobile e desktop)** é entregável formal da rubrica: ambos suportados, nenhum dos dois pode quebrar. Desktop-first é a ordem de trabalho, não dispensa de suporte a mobile. Fonte: `frontend_conventions.md` §Responsividade.
- Seis telas, seis pastas em `pages/`. Tela nova exige sinalização.
- Chamada HTTP só em `services/`.
- Carregamento e erro tratados em toda chamada — não só o caminho feliz.
- Nenhuma abstração antes do terceiro uso.
- Nada da lista de não-escopo do v1 sem sinalizar antes.
- Nenhuma biblioteca nova sem justificar e registrar em `docs/decisions.md`.
- Código que Igor não conseguiria explicar em voz alta não serve — prefira a solução clara à esperta.

## Ao final de cada ciclo

Faça **uma pergunta de defesa** sobre uma decisão técnica daquele código. Uma só, específica ao que acabou de ser escrito.
