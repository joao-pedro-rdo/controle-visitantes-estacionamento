# Checklist Operacional

Baseado em `docs/backend-roadmap.md`.

## Status Geral

- [ ] Fase 0 concluida
- [ ] Fase 1 concluida
- [ ] Fase 2 concluida
- [ ] Fase 3 concluida
- [ ] Fase 4 concluida
- [ ] Fase 5 concluida
- [ ] Fase 6 concluida
- [ ] Fase 7 concluida
- [ ] Fase 8 concluida
- [ ] Fase 9 concluida
- [ ] Fase 10 concluida

## Fase 0: Padrao Tecnico

- [ ] Confirmar stack: `Fastify + TypeScript + Zod + Vitest + Prisma`
- [ ] Confirmar que `NestJS` esta fora do escopo por enquanto
- [ ] Definir regra: toda feature nova nasce em TypeScript
- [ ] Definir regra: toda rota nova tem schema `Zod`
- [ ] Definir regra: controller sem regra de negocio
- [ ] Definir regra: acesso Prisma fora do controller
- [ ] Definir regra: erros padronizados
- [ ] Definir estrutura alvo:
  - [ ] `src/app.ts`
  - [ ] `src/server.ts`
  - [ ] `src/routes/`
  - [ ] `src/controllers/`
  - [ ] `src/services/`
  - [ ] `src/repositories/`
  - [ ] `src/schemas/`
  - [ ] `src/lib/`
  - [ ] `src/types/`
  - [ ] `src/test/`

## Fase 1: Infraestrutura Do Backend

- [ ] Instalar `typescript`
- [ ] Instalar `tsx` ou decidir executor de dev para TS
- [ ] Criar `backend/tsconfig.json`
- [ ] Criar `src/app.ts`
- [ ] Criar `src/server.ts`
- [ ] Mover bootstrap do Fastify para `app.ts`
- [ ] Deixar `server.ts` responsavel apenas por `listen`
- [ ] Ajustar script `dev`
- [ ] Ajustar script `build`
- [ ] Ajustar script `start`
- [ ] Criar script `test`
- [ ] Criar script `test:watch`
- [ ] Instalar `zod`
- [ ] Avaliar integracao com Fastify para schemas tipados
- [ ] Instalar `vitest`
- [ ] Criar config minima do `vitest`
- [ ] Criar `src/test/setup.ts`
- [ ] Criar helper para criar app de teste
- [ ] Criar helper para autenticacao em teste
- [ ] Validar que o backend sobe apos a transicao inicial
- [ ] Validar que `fastify.inject()` funciona

## Fase 2: Fundacoes Do Backend

- [ ] Criar `AppError`
- [ ] Padronizar codigos HTTP
- [ ] Padronizar mensagens de erro para frontend
- [ ] Criar handler global de erro
- [ ] Mapear `ZodError` para `400`
- [ ] Mapear auth para `401` e `403`
- [ ] Esconder stack em producao
- [ ] Criar utilitario de validacao para `body`
- [ ] Criar utilitario de validacao para `params`
- [ ] Criar utilitario de validacao para `query`
- [ ] Centralizar Prisma em um client unico
- [ ] Reduzir imports diretos de Prisma espalhados
- [ ] Centralizar parsing de token
- [ ] Centralizar leitura de cookie
- [ ] Centralizar verificacao de usuario e role
- [ ] Padronizar respostas de sucesso
- [ ] Padronizar respostas de erro

## Fase 3: Auth

- [ ] Criar schema Zod de login
- [ ] Criar schema Zod de signup
- [ ] Criar schema Zod de update user
- [ ] Criar schema Zod de update password
- [ ] Criar service de auth
- [ ] Mover validacao de credenciais para service
- [ ] Mover emissao de token para service
- [ ] Mover logout para service
- [ ] Mover check de sessao para service
- [ ] Enxugar controller de auth
- [ ] Revisar `httpOnly`
- [ ] Revisar `sameSite`
- [ ] Revisar `secure`
- [ ] Validar compatibilidade com Nginx e proxy
- [ ] Revisar `verifyToken`
- [ ] Revisar `verifyS2Role`
- [ ] Revisar `verifyGuardaRole`
- [ ] Criar teste: login sucesso
- [ ] Criar teste: login invalido
- [ ] Criar teste: auth check autenticado
- [ ] Criar teste: auth check sem cookie
- [ ] Criar teste: logout

## Fase 4: Vehicles

- [ ] Criar schema Zod de create vehicle
- [ ] Criar schema Zod de update vehicle
- [ ] Criar schema Zod de get by id
- [ ] Criar schema Zod de get by plate
- [ ] Validar placa
- [ ] Validar CPF ou RG conforme regra definida
- [ ] Validar campos obrigatorios
- [ ] Validar tamanho maximo dos campos
- [ ] Validar caracteres especiais indevidos
- [ ] Criar service de vehicles
- [ ] Criar repository de vehicles
- [ ] Remover regra de negocio do controller
- [ ] Criar teste: create success
- [ ] Criar teste: create invalid body
- [ ] Criar teste: get by id
- [ ] Criar teste: update
- [ ] Criar teste: conflict quando aplicavel
- [ ] Criar teste: auth nas rotas protegidas

## Fase 5: Entries E Exits

- [ ] Mapear fluxo de entrada de visitante
- [ ] Mapear fluxo de entrada de permissionario
- [ ] Mapear fluxo de agendamento
- [ ] Mapear fluxo de confirmacao
- [ ] Mapear fluxo de saida
- [ ] Criar schema Zod de create entry
- [ ] Criar schema Zod de create exit
- [ ] Criar schema Zod de query by date
- [ ] Criar schema Zod de create scheduled entry
- [ ] Criar schema Zod de confirm scheduled entry
- [ ] Preservar ordem das rotas especificas antes das genericas
- [ ] Criar service de entries
- [ ] Criar repository de entries
- [ ] Validar regra: pessoa ja esta dentro
- [ ] Validar regra: saida sem entrada
- [ ] Validar regra: agendamento em data invalida
- [ ] Validar regra: tipo inconsistente
- [ ] Criar teste: entrada visitante
- [ ] Criar teste: saida com permissao correta
- [ ] Criar teste: consulta por data
- [ ] Criar teste: confirmacao de agendamento
- [ ] Criar teste: unauthorized
- [ ] Criar teste: forbidden

## Fase 6: Permissionarios E Pessoas Nao Autorizadas

- [ ] Criar schemas Zod
- [ ] Criar services
- [ ] Criar repositories
- [ ] Validar CPF corretamente
- [ ] Validar upload e imagem quando aplicavel
- [ ] Criar testes principais de permissionarios
- [ ] Criar testes principais de pessoas nao autorizadas

## Fase 7: Uploads, Imagens E Settings

- [ ] Mapear comportamento de `/public`
- [ ] Mapear comportamento de `/system-images`
- [ ] Mapear comportamento de `/images/...`
- [ ] Confirmar acoplamento com bind mount de `frontend/public`
- [ ] Validar extensao de upload
- [ ] Validar mime type de upload
- [ ] Validar tamanho de upload
- [ ] Padronizar nome de arquivo
- [ ] Revisar upload de logo
- [ ] Revisar upload de background
- [ ] Revisar reset de imagens
- [ ] Revisar leitura das imagens atuais
- [ ] Validar fallback de logo padrao
- [ ] Validar fallback de background padrao
- [ ] Criar teste: settings
- [ ] Criar teste: images
- [ ] Criar teste: autorizacao S2
- [ ] Criar teste: fallback sem imagem customizada
- [ ] Fazer smoke test manual com Docker
- [ ] Fazer smoke test manual com Nginx
- [ ] Validar leitura das imagens no frontend

## Fase 8: Observabilidade E Robustez

- [ ] Reduzir `console.log` ruidoso
- [ ] Padronizar logs relevantes
- [ ] Padronizar contexto de logs
- [ ] Garantir middleware de Prometheus antes das rotas
- [ ] Validar que refactors nao quebraram metricas
- [ ] Adicionar healthcheck claro
- [ ] Revisar timeouts de integracoes externas
- [ ] Revisar erros de vigilancia e cameras
- [ ] Revisar tratamento de excecoes Prisma

## Fase 9: Hardening De Validacao

- [ ] Validar CPF apenas numerico
- [ ] Validar tamanho correto do CPF
- [ ] Bloquear caracteres especiais indevidos
- [ ] Padronizar nomes de campos entre frontend e backend
- [ ] Resolver padronizacao `CPF` vs `IDT`
- [ ] Criar mascaras de frontend onde fizer sentido
- [ ] Garantir que a validacao real continue no backend
- [ ] Revisar padrao de CNH ou habilitacao
- [ ] Revisar campo cor com dropdown controlado
- [ ] Revisar secoes e destinos com dropdown controlado

## Fase 10: Suite Minima De Regressao

- [ ] Cobrir Auth
- [ ] Cobrir Vehicles
- [ ] Cobrir Entries
- [ ] Cobrir Permissionarios
- [ ] Cobrir Pessoas nao autorizadas
- [ ] Cobrir Settings e images
- [ ] Garantir caso `200`
- [ ] Garantir caso `400`
- [ ] Garantir caso `401`
- [ ] Garantir caso `403`
- [ ] Garantir caso `404`
- [ ] Garantir caso `409` quando aplicavel

## Proxima Execucao Recomendada

- [ ] Instalar `typescript`, `zod` e `vitest`
- [ ] Separar `app` de `server`
- [ ] Refatorar `auth` primeiro
- [ ] Escrever testes de `login`, `checkAuth` e `logout`
