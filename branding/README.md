# branding/

Pasta do host com a identidade visual do sistema. Ela é montada (somente leitura)
no container do frontend em `/branding/`.

- `logo.png`: logo do cabeçalho, da tela de login e do favicon (nome configurável em `APP_LOGO_FILE`)
- `bg-cover.png`: imagem de fundo (nome configurável em `APP_BACKGROUND_FILE`)

Título e cores ficam no `.env` (`APP_TITLE`, `APP_PRIMARY_COLOR`, `APP_SECONDARY_COLOR`).
Arquivo ausente faz o sistema usar o padrão embutido na imagem.

Para aplicar mudanças: `docker compose ... up -d` (recria o frontend e regenera `/config.js`).
Os navegadores buscam as imagens novas automaticamente.

Nos servidores, esta pasta é mantida no host (`$DEPLOY_PATH/branding`), como o `.env`.
Somente este README é versionado.
