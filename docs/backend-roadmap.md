# Roadmap Backend

## Decisao Tecnica

- Manter `Fastify`
- Adotar `TypeScript`
- Usar `Zod` para validacao estilo DTO
- Usar `Vitest` para testes
- Manter `Prisma`
- Deixar migracao para `NestJS` fora do escopo por enquanto

## Opiniao Sobre Vitest

- `Vitest` e a melhor escolha para este backend hoje.
- E mais leve e rapido que `Jest` para uma base nova.
- Como o backend ainda nao tem suite consolidada, nao existe custo de migracao.
- Para testes de API com Fastify, a combinacao recomendada e:
  - `vitest`
  - `fastify.inject()`
  - `supertest` apenas se algum dia houver necessidade de testar HTTP externo de outra forma

## Stack Alvo

- `Fastify + TypeScript + Zod + Vitest + Prisma`

## Fase 0: Congelar O Padrao Tecnico

Objetivo: decidir o stack e evitar retrabalho.

### Tasks

1. Confirmar stack alvo
   - `Fastify`
   - `TypeScript`
   - `Zod`
   - `Vitest`
   - `Prisma`
2. Definir convencoes
   - toda feature nova nasce em TS
   - toda rota nova tem schema Zod
   - controller sem regra de negocio
   - acesso Prisma fora do controller
   - erros padronizados
3. Definir estrutura alvo de pastas
   - `src/app.ts`
   - `src/server.ts`
   - `src/routes/`
   - `src/controllers/`
   - `src/services/`
   - `src/repositories/`
   - `src/schemas/`
   - `src/lib/`
   - `src/types/`
   - `src/test/`

### Entregavel

- arquitetura alvo decidida
- padrao escrito e compartilhado

## Fase 1: Preparar Infraestrutura Do Backend

Objetivo: habilitar TS e testes sem quebrar o sistema atual.

### Tasks

1. Adicionar TypeScript no backend
   - instalar `typescript`
   - instalar `tsx` ou manter execucao atual durante transicao
   - criar `tsconfig.json`
2. Separar bootstrap da aplicacao
   - criar `app.ts` para montar Fastify
   - criar `server.ts` so para iniciar `listen`
   - isso facilita testes com `inject`
3. Ajustar scripts do backend
   - `dev`
   - `build`
   - `start`
   - `test`
   - `test:watch`
4. Instalar Zod
   - `zod`
   - opcional integracao com Fastify via provider
5. Instalar Vitest
   - `vitest`
   - cobertura opcional depois
   - criar config minima
6. Criar base de testes
   - `src/test/setup.ts`
   - helper para criar app de teste
   - helper para autenticar usuario

### Entregavel

- backend sobe com TS habilitado
- backend aceita testes com `fastify.inject()`

## Fase 2: Padronizar Fundacoes Do Backend

Objetivo: criar as pecas reutilizaveis antes de refatorar features.

### Tasks

1. Criar camada de erro padrao
   - `AppError`
   - codigos HTTP consistentes
   - mensagens claras para frontend
2. Criar handler global de erro
   - mapear `ZodError` para `400`
   - mapear auth para `401/403`
   - esconder stack em producao
3. Criar utilitario de validacao
   - parse de `body`
   - parse de `params`
   - parse de `query`
4. Isolar Prisma
   - criar um client unico
   - remover import espalhado quando possivel
5. Padronizar auth helpers
   - extrair token parsing
   - extrair cookie access
   - centralizar verificacao de usuario e role
6. Padronizar resposta de rotas
   - sucesso consistente
   - erros consistentes

### Entregavel

- base pronta para refatorar features sem repetir codigo

## Fase 3: Refatorar Autenticacao Primeiro

Objetivo: atacar a area mais critica do sistema.

### Tasks

1. Criar schemas Zod de auth
   - login
   - signup
   - update user
   - update password
2. Criar service de auth
   - validar credenciais
   - emitir token
   - limpar cookie
   - checar sessao
3. Enxugar controller de auth
   - controller so recebe `req`, chama service e responde
4. Revisar regras de cookie
   - `httpOnly`
   - `sameSite`
   - `secure`
   - compatibilidade com Nginx e proxy
5. Revisar middleware de auth
   - `verifyToken`
   - `verifyS2Role`
   - `verifyGuardaRole`
6. Adicionar testes de auth
   - login sucesso
   - login invalido
   - auth check autenticado
   - auth check sem cookie
   - logout

### Entregavel

- auth com schema, service e testes

## Fase 4: Refatorar Vehicles

Objetivo: comecar pelos cadastros mais usados e mais simples que entries.

### Tasks

1. Criar schemas Zod
   - create vehicle
   - update vehicle
   - get by id
   - get by plate
2. Incluir validacoes reais
   - placa
   - CPF ou RG se aplicavel
   - campos obrigatorios
   - tamanho maximo
   - caracteres especiais indevidos
3. Criar service de vehicles
   - criar
   - listar
   - buscar por id
   - buscar por placa
   - atualizar
   - excluir
4. Criar repository de vehicles
   - isolar Prisma
5. Refatorar rota e controller
   - remover regra do controller
6. Adicionar testes
   - create success
   - create invalid body
   - get by id
   - update
   - duplicate ou conflict se existir regra
   - auth nas rotas protegidas

### Entregavel

- modulo vehicles estruturado e testado

## Fase 5: Refatorar Entries E Exits

Objetivo: organizar o fluxo mais sensivel da regra de negocio.

### Tasks

1. Mapear fluxos reais
   - entrada visitante
   - entrada permissionario
   - agendamento
   - confirmacao
   - saida
2. Criar schemas Zod
   - create entry
   - create exit
   - query by date
   - create scheduled entry
   - confirm scheduled entry
3. Preservar ordem das rotas
   - manter rotas especificas antes das genericas
4. Criar service de entries
   - regras de entrada
   - regras de saida
   - regras de agendamento
   - regras de consulta por periodo
5. Adicionar validacoes de dominio
   - pessoa ja esta dentro
   - saida sem entrada
   - agendamento em data invalida
   - tipo inconsistente
6. Adicionar testes
   - entrada visitante
   - saida com permissao correta
   - consulta por data
   - confirmacao de agendamento
   - unauthorized e forbidden

### Entregavel

- modulo entries organizado e com risco menor de regressao

## Fase 6: Refatorar Permissionarios E Pessoas Nao Autorizadas

Objetivo: padronizar os demais modulos de cadastro sensiveis.

### Tasks

1. Criar schemas Zod
2. Criar services
3. Criar repositories
4. Validar CPF corretamente
5. Validar upload e imagem quando aplicavel
6. Adicionar testes principais

### Entregavel

- cadastros principais padronizados

## Fase 7: Uploads, Imagens E Settings

Objetivo: estabilizar a parte mais acoplada entre backend, frontend e Docker.

### Tasks

1. Mapear comportamento atual
   - `/public`
   - `/system-images`
   - `/images/...`
   - bind mount com `frontend/public`
2. Padronizar validacao de upload
   - extensao
   - mime type
   - tamanho
   - nome de arquivo
3. Revisar services de imagem
   - upload logo
   - upload background
   - reset
   - leitura das imagens atuais
4. Tratar fallback com clareza
   - logo padrao
   - background padrao
5. Adicionar testes
   - endpoints de settings
   - endpoints de imagens
   - autorizacao S2
   - fallback quando nao ha imagem customizada
6. Fazer smoke test manual com Docker e Nginx
   - login
   - upload de logo
   - upload de background
   - leitura no frontend

### Entregavel

- imagens do sistema com menos comportamento magico

## Fase 8: Observabilidade E Robustez

Objetivo: deixar o backend mais facil de manter.

### Tasks

1. Revisar logs
   - reduzir `console.log` ruidoso
   - deixar logs relevantes
   - padronizar contexto
2. Revisar Prometheus
   - manter middleware antes das rotas
   - validar que refactors nao quebraram metricas
3. Adicionar healthcheck claro
4. Revisar timeouts e erros externos
   - vigilancia e cameras
5. Revisar tratamento de excecoes Prisma

### Entregavel

- backend mais previsivel em producao

## Fase 9: Hardening De Validacao

Objetivo: atacar dores ja mapeadas no projeto.

### Tasks

1. Validar CPF apenas numerico e com tamanho correto
2. Validar campos contra caracteres especiais indevidos
3. Padronizar nome de campos entre frontend e backend
   - exemplo: `CPF` vs `IDT`
4. Criar mascaras no frontend quando fizer sentido
5. Manter validacao real no backend sempre
6. Revisar habilitacao ou CNH
7. Revisar campos de cor e secoes com dropdowns controlados

### Entregavel

- reducao de bug de input inconsistente

## Fase 10: Suite Minima De Regressao

Objetivo: ter seguranca para continuar evoluindo.

### Prioridade De Testes

1. Auth
2. Vehicles
3. Entries
4. Permissionarios
5. Pessoas nao autorizadas
6. Settings e images

### Casos Obrigatorios

1. `200` sucesso
2. `400` body invalido
3. `401` sem auth
4. `403` sem role
5. `404` nao encontrado
6. `409` conflito quando aplicavel

### Entregavel

- suite minima confiavel para refactors

## Ordem De Execucao Recomendada

1. Fase 0
2. Fase 1
3. Fase 2
4. Fase 3
5. Fase 4
6. Fase 5
7. Fase 10 parcial para consolidar
8. Fase 6
9. Fase 7
10. Fase 8
11. Fase 9

## Backlog Pratico Em Blocos

### Bloco 1

1. Adicionar TypeScript ao backend
2. Criar `app.ts` e `server.ts`
3. Instalar Zod
4. Instalar Vitest
5. Criar setup base de testes

### Bloco 2

1. Criar `AppError`
2. Criar error handler global
3. Centralizar Prisma client
4. Centralizar auth helpers
5. Definir padrao route/controller/service/repository

### Bloco 3

1. Refatorar auth schemas
2. Refatorar auth service
3. Refatorar auth controller
4. Testar login/logout/checkAuth
5. Validar cookie e proxy Nginx

### Bloco 4

1. Refatorar vehicles schemas
2. Refatorar vehicles service
3. Refatorar vehicles repository
4. Testar CRUD de vehicles
5. Adicionar validacao de campos

### Bloco 5

1. Refatorar entries schemas
2. Refatorar entries service
3. Refatorar entries repository
4. Preservar ordem das rotas
5. Testar entradas, saidas e agendamentos

### Bloco 6

1. Refatorar permissionarios
2. Refatorar pessoas nao autorizadas
3. Validar CPF e uploads
4. Testar fluxos principais

### Bloco 7

1. Refatorar settings
2. Refatorar images
3. Testar upload/reset/listagem
4. Fazer smoke test no Docker

## Proximo Passo Recomendado

1. Instalar `typescript`, `zod` e `vitest`
2. Separar `app` de `server`
3. Refatorar `auth` primeiro
4. Escrever os primeiros testes de `login`, `checkAuth` e `logout`
