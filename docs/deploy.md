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

O runner self-hosted de homologação já está instalado diretamente no servidor e o workflow seleciona os labels `self-hosted`, `linux` e `homolog`. O job de deploy faz um preflight e falha com mensagem clara se faltar `DEPLOY_PATH`, `.env`, `ssl/server.crt`, `ssl/server.key` ou acesso ao Docker. O usuário do serviço do runner precisa executar `docker`/`docker compose` sem sudo e ter permissão para gravar no diretório de deploy. Instruções operacionais estão em [`runners/README.md`](../runners/README.md).

Crie o GitHub Environment `homolog` com `DEPLOY_PATH` (como Variable ou Secret; o workflow aceita os dois, preferindo a Variable) apontando para o diretório persistente correspondente, por exemplo `/opt/controle-visitantes`. O workflow espera que esse diretório já tenha:

- `.env` com configuração própria daquele ambiente;
- `ssl/server.crt` e `ssl/server.key` para o Nginx;
- `branding/` com a **Logo do Sistema** e a **Imagem de Fundo** daquele ambiente (opcional; ver abaixo);
- espaço persistente para os volumes Docker de PostgreSQL e imagens de visitantes.

O deploy de homologação usa o projeto Compose `controle-homolog`. Os diretórios `.env` e `ssl/` não são copiados do checkout. A workflow de produção está no arquivo `.github/workflows/deploy-prod.yml.disabled`, que o GitHub não executa; ela só deve ser habilitada quando o runner e as configurações de produção forem preparados.

**Migração do stack já existente:** antes do primeiro deploy em qualquer servidor, faça backup do banco atual e identifique explicitamente o volume/serviço PostgreSQL que contém os dados. O Compose novo cria volumes vinculados aos nomes `controle-homolog`/`controle-prod`; não aponte produção para um volume novo sem restaurar/conectar os dados existentes e validar a aplicação. Preserve também os volumes de uploads/imagens e configure permissões/caminhos antes de ativar o workflow.

## Identidade Visual

Logo, imagem de fundo, título e cores são definidos por ambiente, no deploy, e não mais pela tela de Configurações:

- os arquivos ficam em `branding/` no host (`BRANDING_DIR`, relativo a `docker/`; padrão `../branding`), montada somente leitura no frontend em `/branding/`;
- `APP_LOGO_FILE`, `APP_BACKGROUND_FILE`, `APP_TITLE`, `APP_PRIMARY_COLOR` e `APP_SECONDARY_COLOR` ficam no `.env`;
- na subida, o container do frontend gera `/config.js` (`frontend/docker/40-app-config.sh`). Arquivo ausente ou valor inválido cai no padrão embutido na imagem;
- para aplicar uma troca, rode `docker compose ... up -d`. `config.js` e `/branding/` são servidos com `Cache-Control: no-cache` e as imagens levam `?v=<mtime>`, então todos os dispositivos passam a ver a versão nova.

Assim uma mesma imagem Docker serve qualquer OM ou ambiente, sem rebuild.

## Posto da Guarda sem login

As máquinas da guarita podem usar o sistema sem login, liberadas por IP:

- `GUARDA_TRUSTED_IPS` no `.env` recebe IPs ou faixas CIDR separados por vírgula (ex.: `10.0.5.21, 10.0.5.22` ou `10.0.5.0/28`). Vazio desativa o recurso. Valor inválido impede o backend de subir;
- sem sessão válida, essas máquinas recebem perfil **Guarda** (usuário `posto-guarda`). Quem fizer login nelas usa o próprio perfil; ao sair, a máquina volta ao perfil de posto. O perfil de posto nunca recebe permissões de S2;
- reserve IP fixo para essas máquinas no DHCP. Qualquer computador que receber um IP da lista entra como Guarda;
- o IP do cliente vem do `X-Forwarded-For` somente dos proxies confiáveis, definidos em `TRUST_PROXY`. No Compose o backend fica atrás de exatamente um Nginx (o do frontend ou o overlay `compose.nginx.yaml`, que proxia `/api/` direto para o backend), então o valor é `1`. `TRUST_PROXY=true` é recusado porque permitiria forjar o IP;
- em produção a porta do backend não deve ser publicada (`compose.yaml` usa só `expose`). No `compose.dev.yaml` a porta 5000 é publicada, então lá um acesso direto ao backend consegue forjar o IP: use o recurso só para teste nesse modo.

## Atualizar uma instalação antiga

O banco não muda de estrutura além das migrações Prisma (aplicadas sozinhas quando o backend sobe). O que muda é o formato dos dados: o sistema novo grava **CPF e CNH só com dígitos** e **placa sem hífen e em maiúsculas** (`ABC1234`, `BRA2E19`), e mostra a máscara apenas na tela. As buscas por QR, placa e CPF comparam o valor exato, então registros antigos com máscara precisam ser normalizados uma vez:

1. Backup antes de tudo: `docker compose --env-file .env -f docker/compose.yaml exec db pg_dump -U <POSTGRES_USER> -d <POSTGRES_DB> -Fc > backup-antes-da-atualizacao.dump`;
2. Suba a versão nova normalmente (as migrações rodam no início do backend);
3. Simule: `docker compose --env-file .env -f docker/compose.yaml exec backend npm run normalize:documents`. Nada é gravado; a saída lista o que seria normalizado e o que ficará como está;
4. Revise a lista "Não alterados": valores inválidos (`N/A`, CPF com dígito errado) e valores que ficariam iguais a outro registro (ex.: o mesmo veículo cadastrado como `ABC-1234` e `ABC1234`). Corrija pela tela o que for importante; o resto continua funcionando, só não será achado pela busca exata;
5. Grave: o mesmo comando com `-- --apply`. Pode ser repetido sem efeito colateral;
6. Se houver fotos de permissionário anteriores à criptografia, faça o mesmo com `npm run encrypt:permissionario-images` (simulação e depois `-- --apply`).

O script cobre: Veículos (CPF, CNH, placa), Permissionários (placa), Pessoas Não Autorizadas (CPF) e o histórico de entradas (CPF do visitante e placa). Placas fora do padrão no histórico (`N/A` etc.) são mantidas e só aparecem na contagem.

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
3. Confirmar que o runner instalado no servidor tem os labels `self-hosted`, `linux`, `homolog`, acesso ao Docker e permissão de escrita no deploy path.
4. Criar/atualizar a branch `homolog` por merge de `develop` quando quiser testar no servidor.
5. Preencher `.env` e instalar certificados TLS no servidor antes do primeiro deploy.

Para rollback, selecione a tag SHA desejada em `IMAGE_TAG` e recrie os serviços com os mesmos Compose files. A tag SHA publicada é a do commit do repositório central que fixa os dois submódulos.

Como o socket Docker dá ao runner controle do daemon do servidor, mantenha os runners privados do repositório e não execute workflows de branches/contribuidores não confiáveis neles.
