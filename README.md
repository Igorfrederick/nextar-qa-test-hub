# QA Test Hub

Aplicação web que substitui a planilha como camada de execução de testes de QA. Cobre o intervalo entre a geração de cenários e a publicação do resultado no Jira: importa cenários por arquivo, executa o ciclo de testes com registro de evidência, acompanha cobertura e gera rascunho de comentário de validação.

> **Estado:** em construção. Estrutura e decisões definidas; implementação em andamento.

---

## Stack

| Frente | Tecnologias |
|---|---|
| Frontend | React, Vite, React Router, React Hook Form, Zod |
| Backend | Node.js, Express, MongoDB, Mongoose, JWT, bcrypt |
| E2E | Playwright, TypeScript |

## Estrutura

```
backend/     API REST em camadas (routes → controllers → services → models)
frontend/    SPA React, uma pasta por tela em src/pages
e2e/         Suíte Playwright com Page Object Model, um page por tela
docs/        Log de decisões técnicas e documento de retomada
```

## Documentação

| Documento | Conteúdo |
|---|---|
| [CLAUDE.md](CLAUDE.md) | Domínio, contrato da API, convenções, estratégia de teste |
| [docs/decisions.md](docs/decisions.md) | Log de decisões técnicas, com motivo e alternativa descartada |
| [docs/handoff.md](docs/handoff.md) | Estado do projeto e ordem de execução |

## Configuração

Cada frente tem seu próprio `.env.example`. Copie e preencha com valores locais:

```bash
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
cp e2e/.env.example e2e/.env
```

Nenhum valor real de segredo é versionado. `JWT_SECRET` deve ser gerado localmente:

```bash
openssl rand -base64 32
```

## Contexto

Este repositório é o entregável das etapas 1, 2, 3 e 5 de um PDI de QA. A etapa 4 é um documento de análise de oportunidade e vive fora daqui.

Instruções completas de setup, seed e execução serão adicionadas conforme cada frente for implementada.
