# Primeira migração de produção (controle-gda → sistema novo)

Roteiro para trocar a instalação antiga (repo `controle-gda`, imagens `joaoprdo/controle-gda-back|front`, containers `db-roberti`, `back-roberti`, `front-roberti`, `nginx-roberti`) pelo sistema novo (projeto Compose `controle-prod`), sem perder dados.

## O que já foi verificado

- **Migrações compatíveis.** As 26 migrações do `controle-gda` (`main` e `develop`) são idênticas, em nome e conteúdo, às 26 primeiras de `backend/prisma/migrations`. O sistema novo só acrescenta duas, ambas sem perda de dados:
  - `20251217173955_add_system_settings_and_destinations`: cria as tabelas `system_settings` e `destinations`;
  - `20261126120000_permissionario_identidade_opcional`: torna `Permissionario.idNumber` opcional.
  As tabelas de módulos removidos (Viaturas, Missões, Motoristas) continuam no banco, sem uso.
- **Fotos compatíveis.** Mesmo diretório (`/app/uploads/visitors` e `/app/uploads/permissionarios`), mesmo algoritmo (AES-256-CBC) e mesma derivação de chave. **As fotos só abrem com o mesmo `IMAGE_ENCRYPTION_KEY`** do `.env` antigo. Se o `.env` antigo não tinha essa variável ou ela tinha menos de 32 caracteres, a chave vinha do **`JWT_SECRET`**, que então também precisa ser copiado.
- **Formato dos dados muda.** CPF e CNH passam a ser gravados só com dígitos, e a placa sem hífen e em maiúsculas. Os scripts de normalização (passo 6) cuidam disso.
- **Perfis.** O sistema novo só aceita os perfis `S2`, `Guarda`, `Scmt` e `SFPC`. Usuários com outro `role` (por exemplo, motorista) deixam de conseguir entrar.

## Estratégia

1. Cópia completa do servidor de produção (clone ou snapshot da VM) e backup lógico do banco.
2. **Ensaio em homologação** com os dados reais de produção: tudo o que der errado aparece aqui, sem risco.
3. Corte em produção: o stack antigo é parado, mas **não é apagado**. O sistema novo usa volumes próprios (`controle-prod_*`), então voltar atrás é só subir o antigo de novo.

## 1. Levantamento e backup no servidor de produção

No diretório do stack antigo:

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}'
docker volume ls | grep -E 'db-roberti|visitor-images'   # anote os nomes completos (com prefixo do projeto)
cp .env ~/env-producao-antiga.backup                       # guardar IMAGE_ENCRYPTION_KEY e JWT_SECRET
```

Backup do banco (formato custom, para `pg_restore`):

```bash
docker exec db-roberti sh -c 'pg_dump -U "$POSTGRES_USER" -d "$POSTGRES_DB" -Fc' > prod-antes-migracao.dump
ls -lh prod-antes-migracao.dump
```

Backup das fotos (troque `<VOLUME_UPLOADS>` pelo nome completo anotado acima):

```bash
docker run --rm -v <VOLUME_UPLOADS>:/data:ro -v "$PWD":/out alpine tar czf /out/prod-uploads.tar.gz -C /data .
```

Guarde os três arquivos (`.dump`, `.tar.gz` e o `.env` antigo) fora do servidor.

## 2. Conferir o histórico de migrações do banco antigo

```bash
docker exec -it db-roberti sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"'
```

```sql
SELECT migration_name, finished_at IS NOT NULL AS ok, rolled_back_at
FROM _prisma_migrations ORDER BY started_at;

SELECT role, count(*) FROM "User" GROUP BY role;
```

- **Esperado:** até 26 linhas, todas com `ok = t`, com nomes que existem em `backend/prisma/migrations`. Nesse caso o backend novo aplica sozinho só o que falta.
- **Linha com `ok = f`** (migração que falhou): pare e resolva antes. Veja se a mudança foi aplicada e marque com `prisma migrate resolve --applied <nome>` ou `--rolled-back <nome>`.
- **Tabela `_prisma_migrations` não existe** (banco criado pelo `schema.sql` antigo): o `migrate deploy` falha com `P3005`. Antes de subir, marque como aplicadas as migrações que o banco já tem, com `prisma migrate resolve --applied <nome>` para cada uma, e deixe as que faltam para o `migrate deploy`.
- **Perfis:** usuários com `role` fora de `S2`, `Guarda`, `Scmt` e `SFPC` precisam ser ajustados ou recriados.

## 3. Ensaio em homologação com os dados de produção

Isto **substitui o banco e as fotos de homologação** pelos de produção.

No servidor de homolog (`/opt/controle-visitantes`), com os arquivos do passo 1 copiados para lá:

1. No `.env` de homolog, use o **mesmo `IMAGE_ENCRYPTION_KEY`** (e o `JWT_SECRET`, se for o caso) do `.env` antigo de produção.
2. Pare o backend, recrie o banco e restaure:

```bash
cd /opt/controle-visitantes
DC="docker compose -p controle-homolog --env-file .env -f docker/compose.yaml -f docker/compose.nginx.yaml"
$DC stop nginx frontend backend
$DC exec -T db sh -c 'dropdb -U "$POSTGRES_USER" --if-exists "$POSTGRES_DB" && createdb -U "$POSTGRES_USER" "$POSTGRES_DB"'
$DC exec -T db sh -c 'pg_restore -U "$POSTGRES_USER" -d "$POSTGRES_DB" --no-owner --no-acl' < prod-antes-migracao.dump
docker run --rm -v controle-homolog_visitor-images:/data -v "$PWD":/in alpine sh -c 'rm -rf /data/* && tar xzf /in/prod-uploads.tar.gz -C /data'
$DC up -d --wait
$DC logs backend | grep -iE 'migrat|error' | head -20
```

3. Normalização, primeiro em simulação:

```bash
$DC exec backend npm run normalize:documents
```

Revise a lista "Não alterados" e depois grave com `-- --apply`. Faça o mesmo com `npm run encrypt:permissionario-images`.

4. Teste na tela:
   - login com usuários antigos;
   - fotos de visitantes e permissionários antigos abrindo;
   - busca por placa e por CPF;
   - QR de permissionário;
   - relatório com período antigo.

Anote tudo o que precisar de correção manual. Se for preciso repetir o ensaio, volte ao passo 2 desta seção.

## 4. Preparar o servidor e o pipeline de produção

No GitHub:

- runner self-hosted no servidor de produção com os labels `self-hosted`, `linux` e `prod`, cujo usuário rode `docker` sem `sudo`;
- Environment `production` com `DEPLOY_PATH` (Variable ou Secret). Recomendado: em **Required reviewers**, colocar você mesmo, para cada deploy pedir aprovação;
- variável `PROD_DEPLOY_ENABLED`: **deixe sem definir** durante a migração. Sem ela, o merge em `main` só valida e publica as imagens `:latest` e `:sha-<commit>`. O primeiro deploy é manual (passo 5). Depois que tudo estiver validado, defina `PROD_DEPLOY_ENABLED = true` para os próximos merges em `main` serem implantados pelo runner (com backup automático antes).

No servidor, em `DEPLOY_PATH`:

- `.env` novo a partir do `.env.example`, com **`IMAGE_ENCRYPTION_KEY` (e `JWT_SECRET`, se for o caso) copiados do `.env` antigo**. Os demais segredos podem ser novos (`openssl rand -hex 32`);
- `ssl/server.crt` e `ssl/server.key` (podem ser os mesmos do stack antigo);
- `branding/` com logo e fundo, se quiser.

## 5. Corte em produção

Escolha um horário sem movimento na guarda.

1. Faça um backup novo do banco (passo 1): os dados mudaram desde o ensaio.
2. Pare o stack antigo **sem apagar volumes** (sem `-v`), no diretório dele:

```bash
docker compose -f compose-prod.yml down
```

3. Suba só o banco novo, restaure o dump e as fotos:

```bash
cd "$DEPLOY_PATH"
mkdir -p docker && cp <checkout-de-main>/docker/{compose.yaml,compose.nginx.yaml,nginx.conf} docker/
DC="docker compose -p controle-prod --env-file .env -f docker/compose.yaml -f docker/compose.nginx.yaml"
$DC up -d --wait db
$DC exec -T db sh -c 'pg_restore -U "$POSTGRES_USER" -d "$POSTGRES_DB" --no-owner --no-acl' < prod-antes-migracao.dump
docker volume create controle-prod_visitor-images   # o Compose avisa que o volume "não foi criado por ele"; é só aviso
docker run --rm -v controle-prod_visitor-images:/data -v "$PWD":/in alpine tar xzf /in/prod-uploads.tar.gz -C /data
```

4. Suba o sistema novo **manualmente** (o primeiro deploy não usa o runner). Use a tag `sha-<commit de main>` que o workflow publicou; ela aparece no resumo do run, ou use `latest`:

```bash
export IMAGE_TAG=sha-<commit-de-main>   # ou latest
docker login                            # se as imagens forem privadas
$DC pull backend frontend
$DC up -d --wait --wait-timeout 300
$DC logs backend | grep -iE 'migrat|error' | head -20
```

   As migrações rodam no início do backend. Se o `--wait` falhar, veja `$DC ps -a` e `$DC logs --tail 200 backend`.
5. Rode a normalização (simulação e depois `-- --apply`), como no ensaio, com `-p controle-prod`.
6. Repita os testes de tela do ensaio.

## 6. Voltar atrás

O banco e as fotos antigos não foram tocados:

```bash
cd "$DEPLOY_PATH" && docker compose -p controle-prod --env-file .env -f docker/compose.yaml -f docker/compose.nginx.yaml down
cd <diretório do stack antigo> && docker compose -f compose-prod.yml up -d
```

Para voltar a uma versão anterior do sistema novo, re-execute o run do workflow de produção daquele commit: o deploy usa a tag imutável `sha-<commit>`. Cada deploy deixa um `pg_dump` em `$DEPLOY_PATH/backups/` (são mantidos os 15 mais recentes).
