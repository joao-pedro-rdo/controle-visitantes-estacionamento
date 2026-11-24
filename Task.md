# Checklist Operacional

Baseado em `docs/backend-roadmap.md`.

## Status Geral

- [x] Fase 0 concluida
- [ ] Fase 1 concluida
- [x] Fase 2 concluida
- [x] Fase 3 concluida
- [x] Fase 4 concluida
- [x] Fase 5 concluida
- [x] Fase 6 concluida
- [ ] Fase 7 concluida (codigo feito; faltam smoke tests manuais Docker/Nginx)
- [ ] Fase 8 concluida
- [x] Fase 9 concluida
- [ ] Fase 10 concluida

## Fase 0: Padrao Tecnico

- [x] Confirmar stack: `Fastify + TypeScript + Zod + Vitest + Prisma`
- [x] Confirmar que `NestJS` esta fora do escopo por enquanto
- [x] Definir package manager: `npm`
- [x] Confirmar motivo do `npm`: `Dockerfile`, `package-lock.json` e scripts atuais ja usam `npm`
- [x] Definir regra: toda feature nova nasce em TypeScript
- [x] Definir regra: toda rota nova tem schema `Zod`
- [x] Definir regra: controller sem regra de negocio
- [x] Definir regra: acesso Prisma fora do controller
- [x] Definir regra: erros padronizados
- [ ] Definir estrutura alvo:
  - [x] `src/app.ts`
  - [x] `src/server.ts`
  - [x] `src/routes/`
  - [x] `src/controllers/`
  - [x] `src/services/`
  - [x] `src/repositories/`
  - [x] `src/schemas/`
  - [x] `src/lib/`
  - [x] `src/types/`
  - [x] `src/test/`

## Fase 1: Infraestrutura Do Backend

- [x] Instalar `typescript`
- [x] Instalar `tsx` ou decidir executor de dev para TS
- [x] Criar `backend/tsconfig.json`
- [x] Criar `src/app.ts`
- [x] Criar `src/server.ts`
- [x] Mover bootstrap do Fastify para `app.ts`
- [x] Deixar `server.ts` responsavel apenas por `listen`
- [x] Ajustar script `dev`
- [x] Ajustar script `build`
- [x] Ajustar script `start`
- [x] Criar script `test`
- [x] Criar script `test:watch`
- [x] Instalar `zod`
- [ ] Avaliar integracao com Fastify para schemas tipados
- [x] Instalar `vitest`
- [x] Criar config minima do `vitest`
- [x] Criar `src/test/setup.ts`
- [x] Criar helper para criar app de teste
- [x] Criar helper para autenticacao em teste
- [x] Validar que o backend sobe apos a transicao inicial
- [x] Validar que `fastify.inject()` funciona

## Fase 2: Fundacoes Do Backend

- [x] Criar `AppError`
- [x] Padronizar codigos HTTP
- [x] Padronizar mensagens de erro para frontend
- [x] Criar handler global de erro
- [x] Mapear `ZodError` para `400`
- [x] Mapear auth para `401` e `403`
- [x] Esconder stack em producao
- [x] Criar utilitario de validacao para `body`
- [x] Criar utilitario de validacao para `params`
- [x] Criar utilitario de validacao para `query`
- [x] Centralizar Prisma em um client unico
- [x] Reduzir imports diretos de Prisma espalhados
- [x] Centralizar parsing de token
- [x] Centralizar leitura de cookie
- [x] Centralizar verificacao de usuario e role
- [x] Padronizar respostas de sucesso
- [x] Padronizar respostas de erro

## Fase 3: Auth

- [x] Criar schema Zod de login
- [x] Criar schema Zod de signup
- [x] Criar schema Zod de update user
- [x] Criar schema Zod de update password
- [x] Criar service de auth
- [x] Mover validacao de credenciais para service
- [x] Mover emissao de token para service
- [x] Mover logout para service
- [x] Mover check de sessao para service
- [x] Enxugar controller de auth
- [x] Revisar `httpOnly`
- [x] Revisar `sameSite`
- [x] Revisar `secure`
- [x] Validar compatibilidade com Nginx e proxy
- [x] Revisar `verifyToken`
- [x] Revisar `verifyS2Role`
- [x] Revisar `verifyGuardaRole`
- [x] Criar teste: login sucesso
- [x] Criar teste: login invalido
- [x] Criar teste: auth check autenticado
- [x] Criar teste: auth check sem cookie
- [x] Criar teste: logout

## Fase 4: Vehicles

- [x] Criar schema Zod de create vehicle
- [x] Criar schema Zod de update vehicle
- [x] Criar schema Zod de get by id
- [x] Criar schema Zod de get by plate
- [x] Validar placa
- [x] Validar CPF ou RG conforme regra definida
- [x] Validar campos obrigatorios
- [x] Validar tamanho maximo dos campos
- [x] Validar caracteres especiais indevidos
- [x] Criar service de vehicles
- [x] Criar repository de vehicles
- [x] Remover regra de negocio do controller
- [x] Criar teste: create success
- [x] Criar teste: create invalid body
- [x] Criar teste: get by id
- [x] Criar teste: update
- [x] Criar teste: conflict quando aplicavel
- [x] Criar teste: auth nas rotas protegidas

## Fase 5: Entries E Exits

- [x] Mapear fluxo de entrada de visitante
- [x] Mapear fluxo de entrada de permissionario
- [x] Mapear fluxo de agendamento
- [x] Mapear fluxo de confirmacao
- [x] Mapear fluxo de saida
- [x] Criar schema Zod de create entry
- [x] Criar schema Zod de create exit
- [x] Criar schema Zod de query by date
- [x] Criar schema Zod de create scheduled entry
- [x] Criar schema Zod de confirm scheduled entry
- [x] Preservar ordem das rotas especificas antes das genericas
- [x] Criar service de entries
- [x] Criar repository de entries
- [x] Validar regra: pessoa ja esta dentro
- [x] Validar regra: saida sem entrada
- [x] Validar regra: agendamento em data invalida
- [x] Validar regra: tipo inconsistente
- [x] Criar teste: entrada visitante
- [x] Criar teste: saida com permissao correta
- [x] Criar teste: consulta por data
- [x] Criar teste: confirmacao de agendamento
- [x] Criar teste: unauthorized
- [x] Criar teste: forbidden

## Fase 6: Permissionarios E Pessoas Nao Autorizadas

- [x] Criar schemas Zod
- [x] Criar services
- [x] Criar repositories
- [x] Validar CPF corretamente
- [x] Validar upload e imagem quando aplicavel
- [x] Criar testes principais de permissionarios
- [x] Criar testes principais de pessoas nao autorizadas

## Fase 7: Uploads, Imagens E Settings

- [x] Mapear comportamento de `/public`
- [x] Mapear comportamento de `/system-images`
- [x] Mapear comportamento de `/images/...`
- [x] Confirmar acoplamento com bind mount de `frontend/public`
- [x] Validar extensao de upload
- [x] Validar mime type de upload
- [x] Validar tamanho de upload
- [x] Padronizar nome de arquivo
- [x] Revisar upload de logo
- [x] Revisar upload de background
- [x] Revisar reset de imagens
- [x] Revisar leitura das imagens atuais
- [x] Validar fallback de logo padrao
- [x] Validar fallback de background padrao
- [x] Criar teste: settings
- [x] Criar teste: images
- [x] Criar teste: autorizacao S2
- [x] Criar teste: fallback sem imagem customizada
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

- [x] Validar CPF apenas numerico
- [x] Validar tamanho correto do CPF
- [x] Bloquear caracteres especiais indevidos
- [x] Padronizar nomes de campos entre frontend e backend
- [x] Resolver padronizacao `CPF` vs `IDT`
- [x] Criar mascaras de frontend onde fizer sentido
- [x] Garantir que a validacao real continue no backend
- [x] Revisar padrao de CNH ou habilitacao
- [x] Revisar campo cor com dropdown controlado
- [x] Revisar secoes e destinos com dropdown controlado
- [x] Validar upload de imagem em permissionario/pessoa nao autorizada (assinatura real do arquivo)
- [x] Importacao CSV de veiculos com o mesmo schema do cadastro
- [ ] Rodar `npm run normalize:documents` na homolog (simular, revisar invalidos, depois `--apply`)

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

- [x] Instalar `typescript`, `zod` e `vitest`
- [x] Separar `app` de `server`
- [x] Refatorar `auth` primeiro
- [x] Escrever testes de `login`, `checkAuth` e `logout`

## Notas E Decisoes Pendentes

### Identidade visual movida para o deploy (2026-11-24)

- **Decisao:** logo, fundo, titulo e cores deixam de ser configurados pela tela (upload S2) e passam a ser configuracao de ambiente: pasta `branding/` no host + variaveis `APP_*` no `.env`, renderizadas em `/config.js` na subida do container do frontend. Ver `docs/deploy.md#identidade-visual`.
- **Removido:** endpoints `/system-images/*`, `/settings/system*`, `/settings/colors`, static `/public/*` do backend e bind mount `frontend/public`. O model `SystemSettings` continua no schema (sem uso) para nao exigir migration agora.
- **Efeito na Fase 7:** os itens de smoke test de upload de logo/background deixam de existir; o smoke test passa a ser "trocar arquivo em `branding/`, `up -d`, conferir em dois navegadores".
- As notas abaixo descrevem o comportamento anterior e ficam como historico.

### Validacao de cadastros (Fase 9, 2026-11-24)

- **Regras de texto:** `backend/src/lib/text-rules.ts` (espelhadas em `frontend/src/utils/validation.js`). Nomes: letras com acento/ç, espaco, apostrofo, ponto, hifen. Textos curtos: tambem numeros, `/ , ( ) º ª &`. Observacao: pontuacao de frase. Bloqueia `<`, `>`, `{`, `}`, `$`, `|`, `;`, barra invertida e crase.
- **CPF/CNH:** gravados so com digitos (CPF com digitos verificadores; CNH 11 digitos, sem conferir DV para nao barrar documento real). Telas formatam na exibicao; busca do relatorio aceita com ou sem mascara. Registros antigos: `npm run normalize:documents` (simulacao por padrao).
- **Visitante:** o formulario enviava `completeName`/`section` e nao enviava `isVisitor`, e o agendamento enviava `scheduleDate`/`scheduleTime`; o backend espera `name`/`target`/`scheduledDate`, entao os dois fluxos davam 400. Corrigido junto com o checkbox **A pe** (`onFoot`): sem ele, placa/modelo/cor sao obrigatorios; com ele, os campos do veiculo sao gravados vazios e o relatorio mostra "A pe".
- **Cor e Secao:** dropdown no frontend (`ColorSelect` com "Outra"; `DestinationSelect` com a lista de destinos). O backend valida so o formato, para nao quebrar dados antigos fora da lista.
- **CSV de veiculos:** colunas NOME, POSTO, MODELO, PLACA, COR (+ CPF, CNH, SECAO, ESQUADRAO opcionais), separador `,` ou `;`. CPF opcional so no CSV (planilhas antigas nao tem). Nao ha tela de importacao no frontend.

### Posto da Guarda sem login por IP (2026-11-24)

- **Decisao:** maquinas da guarita entram como Guarda sem senha quando o IP esta em `GUARDA_TRUSTED_IPS` (IPs/CIDRs). Sessao com login tem prioridade; cookie invalido/expirado no posto cai no perfil de posto. Implementado em `backend/src/lib/guard-station.ts` + `requireAuth`; usuario virtual `id: 0`, `login: posto-guarda`, `station: true`.
- **Proxy:** `TRUST_PROXY` (contagem de proxies, `1` no Compose) controla quando o `X-Forwarded-For` e usado; `true` e recusado. Testes em `src/test/routes/guard-station-routes.test.ts` cobrem cabecalho forjado sem proxy confiavel e IP prefixado pelo cliente.
- **Frontend:** navbar mostra "(posto da Guarda)" e o botao Sair vira "Entrar com usuario".
- **Pendente:** validar na homolog com o IP real da guarita (smoke test da Fase 7).

### Fase 7 - comportamento de imagens e settings (historico, substituido acima)

- **Mapa de servico das imagens:** `/public/*` serve `frontend/public` via bind mount em Docker (imagens padrao `logo.png`/`bg-cover.jpg` do frontend). `/system-images/*` serve `backend/public/img` (Imagem Customizada, uploads S2). `/images/{visitors,permissionarios,pessoas-nao-autorizadas}/*` servem uploads criptografados de `backend/uploads`, com acesso autenticado.
- **Fallback explicito (Imagem Padrao):** `GET /system-images/current` retorna `null` para o tipo sem customizacao (frontend aplica `/img/logo.png` e `/img/bg-cover.jpg`). `GET /settings/system` NÃO retorna `null`: devolve o objeto completo com os defaults quando nao ha customizacao (paridade com o comportamento antigo; e o que o frontend consome para titulo/logo/fundo). Default de background do backend foi alinhado para `/img/bg-cover.jpg` (antes `/img/background.jpg`), casando com o fallback usado no frontend.
- **Validacao de upload em permissionario/pessoa finalizar (Fase 9):** a Fase 6/7 validou extensao/mime/tamanho/nome somente nas imagens do sistema; uploads de permissionario/pessoa-nao-autorizada ainda passam pela paridade antiga (`image-storage` salva qualquer arquivo como `.jpg`). Aplicar `validateImageUpload` nesses fluxos junto da ingestao de CSV.
- **`backend/docs/ENDPOINTS_IMAGENS_SISTEMA.md` desatualizado:** documenta um unico `POST /system-images/upload` com `file`+`imageType`; o contrato real (frontend `ImageUploader.js`) usa dois endpoints `POST /system-images/upload/logo` e `POST /system-images/upload/background`. Revisar a doc (nao o codigo).
- **Smoke test manual (Docker/Nginx) e leitura no frontend:** pendentes - precisam de ambiente com o stack completo (login, upload de logo/background, leitura via Nginx).

### Frontend / normalizacao de dados (aplicar na Fase 9 ou quando fizer sentido)

- [ ] **CPF mascarado vs armazenado (vehicles):** o `VehicleForm` envia `idNumber` mascarado (`999.999.999-99`); o backend aceita (valida digitos) mas armazena o valor com mascara. Se surgir relatorio/busca por CPF, sera preciso normalizar. Relacionado ao item da Fase 9 `Resolver padronizacao CPF vs IDT`.
- [ ] **`vehicles-csv-controller.js` bypassa o schema novo:** o import de CSV insere `idNumber: 'N/A'` e placa crua direto via Prisma, fora das regras de validacao de vehicles (CPF/placa normalizada). Harmonizar quando a Fase 7 (uploads/settings) ou uma futura UI de importacao CSV passar por ali.
