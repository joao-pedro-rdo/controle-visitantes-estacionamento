# GitHub Actions runners

## Homologação (ativo)

O runner está instalado diretamente no servidor de homologação. O workflow `.github/workflows/deploy-homolog.yml` seleciona-o pelos labels:

- `self-hosted`
- `linux`
- `homolog` (label customizado)

O GitHub também mantém labels padrão de sistema operacional/arquitetura. Confirme em **Settings → Actions → Runners** que o runner está online e tem esses labels.

O usuário que executa o serviço do runner deve:

- executar `docker` e `docker compose` sem `sudo` (permissão no socket Docker do host);
- ter escrita no caminho registrado como variável `DEPLOY_PATH` do GitHub Environment `homolog`;
- conseguir manter nesse caminho o `.env`, `ssl/server.crt`, `ssl/server.key` e os diretórios bind-mounted pelo Compose.

O workflow autentica no Docker Hub usando os repository secrets `DOCKERHUB_USERNAME` e `DOCKERHUB_TOKEN`, então não é necessário manter senha DockerHub na máquina.

## Produção (ainda não configurado)

O workflow `.github/workflows/deploy-prod.yml` seleciona o runner pelos labels `self-hosted`, `linux` e `prod` (label customizado `prod`). Crie o Environment `production` com `DEPLOY_PATH` e, de preferência, *Required reviewers*. O job de deploy só roda quando a variável `PROD_DEPLOY_ENABLED` for `true`; até lá o merge em `main` apenas publica as imagens. Siga [`docs/migracao-producao.md`](../docs/migracao-producao.md) antes de habilitar.

Não habilite deploy de produção antes de identificar e preservar/restaurar o volume PostgreSQL e os uploads/imagens do sistema que já estão em uso.
