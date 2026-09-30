# controle-visitantes-estacionamento

_Status do Projeto: Em desenvolvimento 🚧_

Sistema de controle de acesso desenvolvido para modernizar a gestão de segurança em organizações militares. A solução substitui processos manuais baseados em papel por uma plataforma digital ágil e confiável, focada na operação em rede local (intranet) para garantir máxima segurança e disponibilidade.

## 🚀 Iniciando o Projeto

Esse projeto usa o repositorio do github com submodulos, para clonar o projeto corretamente use o comando:

```code
git clone --recurse-submodules https://github.com/joao-pedro-rdo/controle-visitantes-estacionamento.git
```

Copie `.env.example` para `.env` e ajuste os valores locais. O modo padrão de desenvolvimento não sobe o Nginx; assim é possível desenvolver e testar frontend/API sem TLS:

```bash
docker compose --env-file .env -f docker/compose.yaml -f docker/compose.dev.yaml up --build
```

Frontend: `http://localhost:3000`; API: `http://localhost:5000`. Para testar também o proxy HTTPS do Nginx, gere certificados locais e inclua o override:

**Linux/macOS (Bash):**
```bash
mkdir -p ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
 -keyout ssl/server.key \
 -out ssl/server.crt \
 -subj "/C=BR/ST=State/L=City/O=Organization/CN=localhost"
```

```bash
docker compose --env-file .env -f docker/compose.yaml -f docker/compose.dev.yaml -f docker/compose.nginx.yaml up --build
```

Mais detalhes de branches, imagens, runners e configuração dos ambientes estão em [`docs/deploy.md`](docs/deploy.md).

**Windows (PowerShell):**

> ⚠️ Pré-requisito: instale o [Chocolatey](https://dev.to/kevinkirsten/como-instalar-e-utilizar-o-chocolatey-guia-para-iniciantes-1i98) e depois o OpenSSL. Abra o PowerShell como **Administrador** e execute:
> ```powershell
> choco install openssl -y
> ```
> Feche e reabra o PowerShell (não precisa ser admin) para usar o `openssl`.

```powershell
New-Item -ItemType Directory -Force -Path ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 `
 -keyout ssl/server.key `
 -out ssl/server.crt `
 -subj "/C=BR/ST=State/L=City/O=Organization/CN=localhost"
```

## O Problema que Resolvemos

Em muitas unidades, o controle de entrada e saída de veículos e visitantes ainda depende de fichas de papel. Esse método é lento, propenso a erros de preenchimento, dificulta consultas rápidas.

## ✨ Principais Funcionalidades

- **Validação Rápida por QR Code:** Agilize a entrada e saída de militares do estacionameto e funcionários com um sistema de leitura de QR Code.
- **Cadastro de Visitantes Simplificado:** Registre novos visitantes em segundos, com captura de foto via webcam diretamente no sistema.
- **Monitoramento em Tempo Real:** Tenha uma visão clara de todos os visitantes que estão atualmente dentro da unidade em um painel de controle intuitivo.
- **Auditoria e Relatórios:** Gere relatórios detalhados de acesso por data ou pessoa, com exportação para formatos abertos (ODF), garantindo rastreabilidade total.
- **Operação 100% Offline:** Projetado para rodar em uma rede interna sem depender de conexão com a internet, garantindo que o sistema esteja sempre operacional.

## 💻 Tech Stack & Arquitetura

O sistema é construído com uma arquitetura moderna de cliente-servidor para garantir manutenibilidade e escalabilidade.

- **Frontend:** Desenvolvido com **React.js**.
- **Backend:** Uma **API REST**
- **Banco de Dados:** **PostgreSQL** e **Prisma**, para persistência de dados localmente.
- **Arquitetura:** Operação exclusiva em rede local (intranet).
