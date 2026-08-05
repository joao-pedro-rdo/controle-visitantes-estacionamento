# Ubiquitous Language

## Pessoas e acesso

| Termo | Definicao | Alias a evitar |
| ----- | --------- | -------------- |
| **Usuario** | Uma identidade autenticada que usa o sistema com um perfil de permissao. | Login, conta |
| **Visitante** | Uma pessoa externa que entra na OM para uma visita pontual. | Pessoa, convidado, civil |
| **Permissionario** | Um funcionario civil, terceirizado ou prestador de servicos com acesso recorrente a OM. Autorizado enquanto permanecer cadastrado. | Visitante fixo, trabalhador, visitante recorrente |
| **Militar** | Uma pessoa vinculada a OM cujo uso do estacionamento depende de veiculo cadastrado. | Usuario, proprietario |
| **Pessoa Nao Autorizada** | Uma pessoa registrada como impedida de entrar na OM inteira. | Bloqueado, impedido |
| **Proprietario** | A pessoa associada a um veiculo cadastrado. | Dono, titular |

## Perfis do sistema

| Termo | Definicao | Alias a evitar |
| ----- | --------- | -------------- |
| **S2** | O perfil administrativo maximo responsavel por cadastros e configuracoes; tambem nomeia uma Secao real da OM. | Admin, administrador, chefe |
| **Guarda** | O perfil operacional responsavel apenas por registrar Entradas, Saidas e consultas na rotina da guarda. | Porteiro, vigilante, operador |

## Identificacao pessoal

| Termo | Definicao | Alias a evitar |
| ----- | --------- | -------------- |
| **CPF** | O identificador civil nacional usado para reconhecer visitantes, permissionarios e proprietarios. | IDT, identidade, documento |
| **CNH** | O documento de habilitacao de motorista usado quando a pessoa dirige um veiculo. | Habilitacao, carteira de motorista |
| **PG** | A patente ou graduacao militar de uma pessoa militar. | Posto, graduacao quando usado separado |
| **Nome de Guerra** | O nome operacional curto pelo qual um militar e identificado na rotina da OM. | Nome, nome abreviado |

## Veiculos

| Termo | Definicao | Alias a evitar |
| ----- | --------- | -------------- |
| **Veiculo** | Um automovel cadastrado ou informado para controle de entrada e saida. | Carro, automovel |
| **Placa** | O identificador veicular oficial usado para buscar e controlar um veiculo. | Identificador do veiculo |
| **Cor do Veiculo** | A cor padronizada do veiculo selecionada em uma lista controlada. | Cor, pintura |
| **Foto do Veiculo** | A imagem do veiculo associada ao registro de entrada ou cadastro. | Imagem do carro, foto |

## Organizacao militar e destino

| Termo | Definicao | Alias a evitar |
| ----- | --------- | -------------- |
| **OM** | A organizacao militar onde a instalacao do sistema controla acessos e estacionamento; cada OM roda sua propria instancia. | Unidade, organizacao |
| **Esquadrao** | Uma subdivisao organizacional da OM associada a militares ou proprietarios. | Secao, destino |
| **Secao de Lotacao** | A secao interna da OM onde um militar ou proprietario esta lotado. | Secao, origem, setor |
| **Secao de Destino** | A secao da OM que um visitante ou permissionario pretende acessar; lista configuravel por OM. | Destino, secao, local de visita |

## Ciclo de acesso

| Termo | Definicao | Alias a evitar |
| ----- | --------- | -------------- |
| **Entrada** | O registro de ingresso de uma pessoa ou veiculo na OM. | Check-in, acesso |
| **Saida** | O registro de retirada de uma pessoa ou veiculo da OM. | Check-out, baixa |
| **Visita** | Um evento pontual em que um visitante acessa a OM com destino definido; na chegada os dados pre-cadastrados sao aproveitados, a Guarda tira foto e o horario de entrada e marcado. | Entrada de visitante, atendimento |
| **Agendamento** | O registro antecipado de uma Visita com dados pessoais do Visitante e destino definido. | Reserva, marcacao |
| **Sessao** | O periodo em que um usuario permanece autenticado no sistema. | Token, JWT, login ativo |

## Imagens do sistema

| Termo | Definicao | Alias a evitar |
| ----- | --------- | -------------- |
| **Logo do Sistema** | A imagem institucional exibida como marca do sistema. | Logo, imagem da OM |
| **Imagem de Fundo** | A imagem visual usada como plano de fundo da interface. | Background, fundo |
| **Imagem Padrao** | A imagem usada quando nao ha personalizacao configurada para o ambiente. | Fallback, imagem default |
| **Imagem Customizada** | A imagem configurada para um ambiente especifico de instalacao. | Upload, imagem personalizada |

## Relationships

- Um **Usuario** possui exatamente uma **Sessao** ativa por contexto de autenticacao definido.
- Um **Usuario** possui exatamente um perfil principal, como **S2** ou **Guarda**; o **S2** realiza cadastros e o **Guarda** somente registra **Entrada** e **Saida**.
- Um **Visitante** deve ter exatamente um **CPF** quando a regra de identificacao civil exigir cadastro completo.
- Um **Permissionario** deve ter exatamente um **CPF**, pode ter um ou mais **Veiculos** associados e nao possui data de validade enquanto permanecer cadastrado.
- Um **Permissionario** deve ter **Entrada** e **Saida** controladas separadamente de **Visitante** e **Militar**.
- Um **Militar** pode estar associado a zero ou mais **Veiculos**, mas so pode usar o estacionamento quando houver **Veiculo** cadastrado; o sistema nao controla a entrada dele na OM.
- Um **Veiculo** pertence a exatamente um **Proprietario** no cadastro de veiculos.
- Um **Proprietario** pode ser identificado por **CPF** e pode ter **PG** e **Nome de Guerra** quando for militar.
- Uma **Visita** possui uma **Secao de Destino** e pode representar entregas de material, visitas administrativas ou outros acessos pontuais a OM.
- Uma **Visita** possui um **Agendamento** antecipado opcional que pre-cadastra os dados do **Visitante**.
- Uma **Entrada** pode estar relacionada a um **Visitante**, **Permissionario** ou **Militar**.
- Uma **Entrada** deve ter no maximo uma **Saida** correspondente.
- Uma **Pessoa Nao Autorizada** esta impedida de entrar na OM inteira, independentemente de possuir veiculo.
- Uma **Imagem Customizada** substitui uma **Imagem Padrao** para **Logo do Sistema** ou **Imagem de Fundo**.

## Example dialogue

> **Dev:** "Quando digitamos o **CPF** de um **Visitante** ja cadastrado, devemos criar outro visitante?"

> **Domain expert:** "Nao. O **CPF** identifica o mesmo **Visitante**; o sistema deve reaproveitar os dados pessoais e criar apenas uma nova **Visita** com a **Secao de Destino** informada."

> **Dev:** "E se o visitante vier de carro?"

> **Domain expert:** "A **Visita** pode receber dados do **Veiculo** manualmente, mas isso nao transforma o **Visitante** em **Proprietario** de um veiculo cadastrado."

> **Dev:** "E o **Permissionario** entra como visitante?"

> **Domain expert:** "Nao. O **Permissionario** e sempre funcionario civil, terceirizado ou prestador com acesso recorrente, entao a **Entrada** e a **Saida** dele devem ser controladas separadamente e ele nao tem data de validade enquanto estiver cadastrado."

> **Dev:** "Quem pode cadastrar um **Veiculo** ou um **Visitante**?"

> **Domain expert:** "Apenas o **S2**. A **Guarda** somente registra **Entrada** e **Saida** e faz consultas na guarda."

> **Dev:** "Se um **Militar** nao tiver **Veiculo** cadastrado, ele esta bloqueado?"

> **Domain expert:** "Nao. Ele continua podendo entrar na OM, mas o sistema so controla a permissao de estacionamento. So a **Pessoa Nao Autorizada** e impedida de entrar na OM inteira."

> **Dev:** "Na tela de veiculos, devo chamar o documento do proprietario de **IDT** ou **CPF**?"

> **Domain expert:** "Use **CPF**. **IDT** e identidade devem ser evitados nessa regra porque o cadastro e a validacao esperada sao de CPF."

> **Dev:** "A secao do proprietario e a secao que o visitante vai acessar sao a mesma coisa?"

> **Domain expert:** "Nao. Para proprietarios militares, use **Secao de Lotacao**; para visitantes e permissionarios, use **Secao de Destino**."

## Flagged ambiguities

- "IDT" foi usado na lista de veiculos para representar o documento do proprietario, enquanto cadastro e edicao usam "CPF"; recomendacao: padronizar como **CPF** quando a regra exige CPF validado.
- "habilitacao" foi usada como nome de campo, mas o termo documental mais preciso e **CNH**; recomendacao: usar **CNH** em rotulos, validacoes e documentacao.
- "secao" foi usada tanto para origem interna do proprietario na OM quanto para destino do visitante; recomendacao: separar em **Secao de Lotacao** e **Secao de Destino**.
- "proprietario", "militar" e "usuario" podem se confundir; recomendacao: **Usuario** e quem autentica, **Militar** e o vinculo institucional, e **Proprietario** e a pessoa associada a um **Veiculo**.
- "registro de visitante" pode significar a pessoa visitante ou uma visita especifica; recomendacao: usar **Visitante** para a pessoa e **Visita** para o evento de acesso.
- "admin" e "S2" podem se confundir; recomendacao: usar **S2** como nome do perfil de dominio; no contexto de secao, usar **Secao S2** para a secao real da OM.
- "guarda", "operador" e "vigilante" podem se confundir; recomendacao: usar **Guarda** para o perfil que registra somente a rotina de entrada e saida.
- "permissionario" e "pessoa nao autorizada" podem parecer inversos de status; recomendacao: **Permissionario** e autorizado por cadastro enquanto vigente, e **Pessoa Nao Autorizada** e o registro que impede a entrada na OM inteira.
- "token", "JWT" e "sessao" foram misturados; recomendacao: no dominio falar em **Sessao**, deixando JWT como detalhe tecnico de implementacao.
- "imagem", "logo", "background" e "fallback" aparecem misturados nas melhorias cosmeticas; recomendacao: usar **Logo do Sistema**, **Imagem de Fundo**, **Imagem Padrao** e **Imagem Customizada** conforme o caso.
