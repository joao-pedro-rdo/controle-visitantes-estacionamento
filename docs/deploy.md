# CI/CD e deploy

## Fluxo de branches

- `develop`: desenvolvimento contínuo; CI roda testes/build, sem publicar imagem nem fazer deploy.
- `homolog`: promoção explícita de `develop`; CI valida, publica `homolog` e `sha-<commit>`, e implanta no servidor de homologação.
- `main`: promoção explícita de `homolog`; CI valida, publica `latest` e `sha-<commit>`, e implanta no servidor de produção.

O repositório central fixa cada versão dos submódulos pelo gitlink. Para publicar código novo, atualize o submódulo e registre o ponteiro no branch do repositório central. Assim cada imagem pode ser relacionada aos commits exatos de backend e frontend usados na composição.

## Imagens Docker

O workflow `deploy-homolog.yml` constrói os dois serviços a partir dos submódulos no estado fixado pela branch `homolog`. A configuração equivalente de produção está preparada, mas desabilitada em `deploy-prod.yml.disabled`.

- `joaoprdo/controle-visitantes-backend:homolog|latest`
- `joaoprdo/controle-visitantes-frontend:homolog|latest`
- tags imutáveis `sha-<commit>` para identificar e recuperar uma publicação específica.

Crie os repositórios no Docker Hub e configure nos **Secrets** do repositório central:

- `DOCKERHUB_USERNAME`: usuário Docker Hub (`joaoprdo` para o namespace configurado nas imagens).
- `DOCKERHUB_TOKEN`: access token do Docker Hub com permissão de escrita. O token substitui o uso de senha da conta no login automatizado.

As imagens podem ser privadas; o job de deploy autentica no Docker Hub antes de fazer pull.

## Runner e ambiente de homologação

O runner self-hosted de homologação já está instalado diretamente no servidor e o workflow seleciona os labels customizados `selfhosted-homolog-gda` e `homolog-gda`. O usuário do serviço do runner precisa executar `docker`/`docker compose` sem sudo e ter permissão para gravar no diretório de deploy. Instruções operacionais estão em [`runners/README.md`](../runners/README.md).

Crie o GitHub Environment `homolog` com a variável `DEPLOY_PATH` apontando para o diretório persistente correspondente, por exemplo `/opt/controle-visitantes`. O workflow espera que esse diretório já tenha:

- `.env` com configuração própria daquele ambiente;
- `ssl/server.crt` e `ssl/server.key` para o Nginx;
- espaço persistente para os volumes Docker de PostgreSQL, imagens de visitantes e imagens customizadas do sistema.

O deploy de homologação usa o projeto Compose `controle-homolog`. Os diretórios `.env` e `ssl/` não são copiados do checkout. A workflow de produção está no arquivo `.github/workflows/deploy-prod.yml.disabled`, que o GitHub não executa; ela só deve ser habilitada quando o runner e as configurações de produção forem preparados.

**Migração do stack já existente:** antes do primeiro deploy em qualquer servidor, faça backup do banco atual e identifique explicitamente o volume/serviço PostgreSQL que contém os dados. O Compose novo cria volumes vinculados aos nomes `controle-homolog`/`controle-prod`; não aponte produção para um volume novo sem restaurar/conectar os dados existentes e validar a aplicação. Preserve também os volumes de uploads/imagens e configure permissões/caminhos antes de ativar o workflow.

## Estrutura Compose

- `docker/compose.yaml`: serviços comuns e imagens; não inicia Nginx.
- `docker/compose.dev.yaml`: build local e portas diretas de desenvolvimento.
- `docker/compose.nginx.yaml`: overlay opcional para Nginx e TLS.

Desenvolvimento sem Nginx:

```sh
docker compose --env-file .env -f docker/compose.yaml -f docker/compose.dev.yaml up --build
```

Teste local com Nginx:

```sh
docker compose --env-file .env -f docker/compose.yaml -f docker/compose.dev.yaml -f docker/compose.nginx.yaml up --build
```

Se o npm local estiver atrás de um proxy que assina TLS com uma CA própria, exporte essa CA como PEM, mantenha o arquivo fora do Git (por exemplo `ssl/npm-ca.pem`), defina `NPM_CA_FILE=../ssl/npm-ca.pem` no `.env` e adicione `-f docker/compose.dev-ca.yaml` aos comandos locais. O certificado é passado como BuildKit secret apenas durante `npm ci`; a validação TLS permanece habilitada.

Os builds React em CI usam `CI=false` durante `npm run build` para que warnings de ESLint legados sejam reportados sem derrubar a compilação. Eles continuam visíveis para limpeza posterior.

Deploy usa `docker/compose.yaml` com `docker/compose.nginx.yaml`; a tag é definida pelo branch. O pipeline puxa as imagens e usa `docker compose up -d --wait`. A migração Prisma continua sendo executada pelo CMD da imagem do backend.

## Configuração manual necessária no GitHub

1. Adicionar `DOCKERHUB_USERNAME` e `DOCKERHUB_TOKEN` como repository secrets.
2. Criar o GitHub Environment `homolog` com `DEPLOY_PATH`.
3. Confirmar que o runner instalado no servidor tem labels `selfhosted-homolog-gda`, `homolog-gda`, acesso ao Docker e permissão de escrita no deploy path.
4. Criar/atualizar a branch `homolog` por merge de `develop` quando quiser testar no servidor.
5. Preencher `.env` e instalar certificados TLS no servidor antes do primeiro deploy.

Para rollback, selecione a tag SHA desejada em `IMAGE_TAG` e recrie os serviços com os mesmos Compose files. A tag SHA publicada é a do commit do repositório central que fixa os dois submódulos.

Como o socket Docker dá ao runner controle do daemon do servidor, mantenha os runners privados do repositório e não execute workflows de branches/contribuidores não confiáveis neles.
