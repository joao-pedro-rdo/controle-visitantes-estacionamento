# Controle de Visitantes — CI/CD e Deploy

Contexto de orquestração de CI/CD e deploy do sistema. Descreve como os repos se relacionam, como imagens são construídas e como ambientes recebem novas versões.

## Language

**Repositorio Central (orquestrador)**:
O repositório raiz (`controle-visitantes-estacionamento`) que contém os submodulos e os arquivos de composição/deploy. Ele não constrói código de aplicação; ele orquestra o deploy.
_Avoid_: repositório principal, monorepo

**Submodulo de Aplicacao**:
Cada repositório que produz código de aplicação (`controle-gda-backend`, `controle-gda-frontend`), rastreado pelo Repositorio Central como submodulo.
_Avoid_: repo filho, módulo de código

**Imagem**:
O artefato Docker construído e publicado a partir de um Submodulo de Aplicacao, referenciado pelo Repositorio Central no deploy.
_Avoid_: build, pacote

**Ambiente**:
Um alvo de deploy com composição e nomenclatura próprias (homolog e prod). Cada Ambiente tem sua própria imagem publicada.
_Avoid_: stage, instância

**Deploy**:
A ação de puxar uma Imagem publicada para o servidor de um Ambiente e reiniciar o stack correspondente.
_Avoid_: release, publicação

**Tag de Imagem**:
O identificador da Imagem que determina em qual Ambiente ela será usada (`latest` para prod, `homolog` para homolog).
_Avoid_: versão, label

## Decisiones

- **Fonte da verdade do que roda**: a **Tag de Imagem** (decisão A). O Submodulo de Aplicacao constrói e publica a Imagem; o Repositorio Central apenas puxa a Tag e sobe o stack. O ref do submodulo no Central nao determina o que roda.
- **Acionamento do deploy**: `repository_dispatch` (decisão A). No fim da pipeline do Submodulo, uma chamada `POST /repos/{central}/dispatches` com evento customizado; o Central escuta esse evento e dispara o Deploy. Nao ha push de volta ao Central.
- **Topologia do runner**: um contêiner Docker self-hosted com daemon Docker interno (Docker-in-Docker). A stack de producao (postgres, backend, frontend, nginx) e orquestrada por esse daemon interno. O workflow do Central roda nesse runner e age sobre esse daemon.
- **Isolamento por Ambiente**: um workflow de deploy por Ambiente (decisão B). O Central tem `deploy-homolog` e `deploy-prod`, cada um fixo no seu compose e na sua tag. Cada Submodulo dispara o `repository_dispatch` do Ambiente correspondente.
- **Regra branch → Ambiente**: push em `develop` → homolog (imagem `:homolog`, evento `deploy-homolog`); push em `main` → prod (imagem `:latest`, evento `deploy-prod`). Promocao e feita por merge `develop → main` no proprio Submodulo.
- **Modelo de deploy**: unitario por Submodulo (decisão A). Cada `repository_dispatch` sobe apenas o servico cuja imagem mudou (`docker compose pull <svc> && up -d <svc>`); back nunca mexe no front e vice-versa.
- **Segredos e config**: arquivos sensiveis (`.env`, `ssl/`) vivem no volume persistente do contêiner do runner, fora do controle do git (decisão A). O workflow apenas garante o checkout do codigo; os bind mounts do compose apontam para esses arquivos preexistentes.
- **Autenticacao do pull**: imagens privadas; o login do Docker Hub e residente no contêiner do runner (via `~/.docker/config.json` injetado no volume), decision A. O workflow do Central nao carrega credencial de Docker Hub; o daemon interno ja autentica ao puxar.
- **Autenticacao do push**: os Submodulos usam GitHub Secrets (`DOCKERHUB_USERNAME`/`DOCKERHUB_TOKEN`) configurados em cada repo de aplicacao, numa mesma conta Docker Hub (`joaoprdo`) usada por todo o sistema.
- **Migracoes**: executadas uma unica vez no `CMD` da imagem do backend (`prisma migrate deploy && npm start`); remove-se o `command:` duplicado do compose e o mount de migrations do host. A fonte das migrations e a imagem, nao o host.
- **Tag de Imagem (estrategia)**: tag dupla (decisão A). O Submodulo publica `:latest`/`:homolog` (mutavel, usada pelo compose) **e** `:sha-<commit>` (imutavel, para rollback sem rebuild).
- **Validacao pos-deploy**: healthcheck + `docker compose up -d --wait` (decisão A). O backend ganha um healthcheck; o workflow do Central usa `--wait` e falha se o servico nao ficar saudavel no limite, sinalizando rollback.
- **Contrato do dispatch**: o payload do `repository_dispatch` e minimo — sem SHA (decisão A). O Central sempre puxa a Tag mutavel do Ambiente (`latest`/`homolog`). O SHA imutavel serve apenas a rollback manual.
- **Gate de testes por Ambiente**: rodar `test` e `build` em qualquer pipeline, mas o comportamento do gate depende do Ambiente. Em **homolog**, testes rodam mas **nao bloqueiam** o push/deploy (imagem `:homolog` e evento `deploy-homolog` publicados mesmo se testes falharem). Em **prod**, testes **bloqueiam** o deploy: se `test`/`build` falham, a imagem `:latest` nao e publicada e `deploy-prod` nao e disparado.
