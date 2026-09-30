# Controle de Visitantes — CI/CD e Deploy

Decisões e contexto atuais estão em [`docs/deploy.md`](docs/deploy.md). Este repositório central fixa os commits dos submódulos e controla build, publicação e deploy por branch.

## Modelo vigente

- `develop` é a branch de desenvolvimento e executa CI sem deploy.
- Merge de `develop` para `homolog` valida, publica tags `homolog` e `sha-<commit>`, e implanta no servidor de homologação.
- Merge de `homolog` para `main` valida, publica tags `latest` e `sha-<commit>`, e implanta no servidor de produção.
- As imagens são `joaoprdo/controle-visitantes-backend` e `joaoprdo/controle-visitantes-frontend`, construídas a partir dos gitlinks dos submódulos no repositório central.
- Homologação usa o runner self-hosted instalado diretamente no servidor, com labels `selfhosted-homolog-gda` e `homolog-gda`. O workflow de produção está preparado separadamente, mas desabilitado até o runner e o ambiente de produção serem configurados.
- Os Compose ficam em `docker/`. `compose.yaml` contém serviços comuns; `compose.dev.yaml` habilita builds e portas locais; `compose.nginx.yaml` é um overlay TLS opcional.
- Configuração `.env`, certificados TLS e dados persistentes são mantidos no host de cada ambiente. Nunca são obtidos de branches do Git.
- Backend e frontend são publicados em conjunto para manter cada release consistente; o deploy só ocorre depois de testes e builds passarem.

## Primeira migração de ambiente

Antes de habilitar o primeiro deploy nos servidores atuais, identificar a origem dos dados PostgreSQL em uso e preservar/restaurar o volume correto. O projeto Compose novo cria volumes nomeados sob os projetos `controle-homolog` e `controle-prod`; não se deve assumir que esses volumes contêm os dados do Compose legado. Fazer backup e validar a migração antes de apontar produção para o novo stack.
