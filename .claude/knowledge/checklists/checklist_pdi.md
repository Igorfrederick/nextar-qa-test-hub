# Checklist — Critérios de avaliação do PDI

Usado pelo `revisor-pdi` para ler o repositório inteiro com os olhos de quem vai avaliar no GitHub.

Os quatro critérios abaixo são os declarados na seção 1 do `CLAUDE.md`. Os itens sob cada um são a leitura verificável desses critérios, derivada das convenções do projeto.

> Se a rubrica formal do PDI divergir deste arquivo, a rubrica vence e este arquivo é corrigido.

## Frontend — estrutura, componentização e reutilização, boas práticas

- [ ] Estrutura de pastas corresponde à seção 8 do `CLAUDE.md`
- [ ] Uma pasta por tela em `pages/`
- [ ] Componentes reutilizáveis existem e são de fato reutilizados
- [ ] Nenhuma duplicação de JSX que já exista como componente
- [ ] Formulários com validação por schema
- [ ] Estados de carregamento e erro tratados
- [ ] Responsividade real na tela de execução
- [ ] Nenhum elemento interativo sem `data-cy`

## Backend — camadas, JWT, hash, env, modelagem

- [ ] Separação de camadas visível na estrutura de pastas e respeitada no código
- [ ] Controller sem regra, service sem `req`/`res`, model sem service
- [ ] JWT com geração **e** validação corretas, expiração definida
- [ ] Senha com bcrypt; `passwordHash` nunca exposto
- [ ] Segredos exclusivamente por variável de ambiente
- [ ] `.env.example` presente e completo, com valores fictícios
- [ ] Modelagem coerente com as regras de negócio — unicidade e índices onde a regra exige
- [ ] Formato de erro único, com `code` e status HTTP adequado

## E2E — organização, seletores, asserções, cobertura, independência, setup/teardown

- [ ] Testes organizados por jornada do usuário
- [ ] Um Page Object por tela
- [ ] Seletores exclusivamente `data-cy`
- [ ] Asserções específicas, no arquivo do teste
- [ ] Cobertura das jornadas mínimas: login e acesso negado por perfil, CRUD de suíte e caso, importação com deduplicação, ciclo completo, regras que falham
- [ ] Independência real — sem dependência de ordem ou de estado
- [ ] Setup via API e teardown limpando o que criou

## Aplicação da solução — executável, organizado, README, facilidade de uso

- [ ] O projeto sobe seguindo apenas o README, sem conhecimento tácito
- [ ] README com setup, seed e execução das três frentes
- [ ] Seed com dado fictício
- [ ] Tabela mapeando cada entregável do PDI ao lugar no repositório
- [ ] Organização geral coerente entre as três frentes

## Transversais

- [ ] Nenhum dado real da Nextar: nome de cliente, chave real de tarefa, conteúdo de bug real
- [ ] Nenhum segredo commitado
- [ ] `docs/decisions.md` cobre as decisões relevantes, em ordem cronológica inversa
- [ ] Histórico de commits coerente com `conventions/commit_conventions.md`
- [ ] Nenhum `.gitkeep` remanescente em pasta que já tem arquivo real
- [ ] Nada da lista de não-escopo do v1 implementado
