# GitHub Actions runners

## Homologação (ativo)

O runner está instalado diretamente no servidor de homologação. O workflow `.github/workflows/deploy-homolog.yml` seleciona-o pelos labels customizados:

- `selfhosted-homolog-gda`
- `homolog-gda`

O GitHub também mantém labels padrão de sistema operacional/arquitetura. Confirme em **Settings → Actions → Runners** que o runner está online e tem ambos os labels.

O usuário que executa o serviço do runner deve:

- executar `docker` e `docker compose` sem `sudo` (permissão no socket Docker do host);
- ter escrita no caminho registrado como variável `DEPLOY_PATH` do GitHub Environment `homolog`;
- conseguir manter nesse caminho o `.env`, `ssl/server.crt`, `ssl/server.key` e os diretórios bind-mounted pelo Compose.

O workflow autentica no Docker Hub usando os repository secrets `DOCKERHUB_USERNAME` e `DOCKERHUB_TOKEN`, então não é necessário manter senha DockerHub na máquina.

## Produção (ainda não configurado)

O workflow está preparado, mas desabilitado em `.github/workflows/deploy-prod.yml.disabled`. Quando o servidor e o runner de produção estiverem configurados, defina os labels (planejados: `selfhosted-prod-gda` e `prod-gda`), crie o Environment `production`, revise `DEPLOY_PATH`, `.env`, TLS e a migração dos dados atuais; depois habilite o arquivo renomeando-o para `deploy-prod.yml`.

Não habilite deploy de produção antes de identificar e preservar/restaurar o volume PostgreSQL e os uploads/imagens do sistema que já estão em uso.
