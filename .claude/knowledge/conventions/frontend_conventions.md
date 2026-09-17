# Convenções — Frontend

Fonte única para qualquer trabalho em `frontend/`. Extraído do `CLAUDE.md`.

## Estrutura

```
pages/        uma pasta por tela
components/   componentes reutilizáveis, sem regra de negócio
hooks/
services/     cliente HTTP; única camada que fala com a API
schemas/      schemas Zod de formulário
contexts/     auth
utils/
```

São seis telas, seis pastas em `pages/`. Tela nova exige sinalização antes.

## Componentização

- Componente em `components/` é reutilizável e **não** carrega regra de negócio
- Regra de negócio vive na página ou em hook, nunca dentro do componente genérico
- JSX que já existe como componente não é reescrito — duplicação de marcação é achado
- **Abstração só no terceiro uso.** Dois lugares parecidos não justificam componente genérico

## Formulários

- React Hook Form com schema Zod de `schemas/`
- Erro exibido por campo, não em bloco único no topo
- O schema é a fonte da validação; validação manual paralela ao schema é achado

## Chamadas à API

- Exclusivamente por `services/`. `fetch` ou cliente HTTP dentro de componente é quebra de camada
- Toda chamada trata **carregamento** e **erro**, não só o caminho feliz
- O erro exibido ao usuário vem do `message` da API; o `code` é o que a lógica consome

## Responsividade

- A aplicação é usada em **desktop**, ao lado de outras ferramentas de QA da equipe. Não há requisito mobile-first.
- Responsividade real, com layout que reflui; `overflow` escondendo conteúdo é achado
- A tela de execução (`/runs/:id`) é a de maior uso: fica aberta por longos períodos, lado a lado com outras janelas. Priorize densidade de informação útil e ação rápida sobre caso de teste, não área de toque

## Seletores

Todo elemento interativo nasce com `data-cy`. Componente novo sem `data-cy` está incompleto.

Padrão: `contexto-elemento[-identificador]`, kebab-case.

```
login-email-input
login-submit-button
project-list-row-{projectKey}
test-case-row-{externalRef}
execution-status-pass-button
run-close-button
coverage-percent-value
comment-draft-textarea
error-toast
```

O identificador dinâmico usa o dado de negócio (`projectKey`, `externalRef`), não índice de posição — índice muda quando a ordenação muda.

## Idioma

Identificadores, componentes e campos em inglês; texto de interface em português. Sem mistura dentro de um mesmo identificador.

## Anti-padrões

| Anti-padrão | Por quê |
|---|---|
| Componente de 300 linhas | Faz muitas coisas; nenhuma delas testável isoladamente |
| Prop drilling de quatro níveis | O dado deveria estar em contexto ou ser derivado |
| `useEffect` para valor derivável do render | Cria estado duplicado que dessincroniza |
| Abstração antes do terceiro uso | Generaliza sobre dois exemplos e acerta o formato errado |
| Token em `localStorage` sem justificativa | Decisão de segurança que exige entrada em `docs/decisions.md` |
| Elemento interativo sem `data-cy` | Componente incompleto; quebra a suíte E2E |
