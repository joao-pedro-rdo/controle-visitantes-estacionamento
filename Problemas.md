1. Botao de sair bugado, nao da refresh na pagina, o usuario precisar dar refresh na pagina
2. Controle do ciclo de vida do JWT
3. Tela de cadastro de veiculo colocar mascar no front para HABILITACAO (procurar qual o padrao de caracteres de uma habilitacao)
4. Tela de cadastro de veiculo em dadados da seçao há esquadrao e secao que seiram o destino é os dados de onde o proprietario é dentro da OM, nessa aba podemos mudar para ser um dropdown igual ao metodo "Seções de Destino" presente nos visitantes onde é possivel gerenciar as seçoes que o visitante vai ir.
5. Tela de cadastro de veiculo, o Campo cor ser um dropdown com as cores mais comuns
6. Verificacao de caracteres especiais na maior parte dos campos de cadastro, para evitar que o usuario coloque caracteres especiais que possam quebrar a aplicacao(deixando ascentos e cedilhas de fora)
7. Em editar CPF, mesmo com a mascara ele aceita caracteres especiais e letras, precisamos validar para aceitar apenas numeros e o tamanho correto do CPF. um DTO resolveria no backend
8. Ao editar um veiculo o CPF do proprietario nao esta sendo puxado para o campo.
9. Na lista de veiculos esta como IDT e ao editar ou tela de cadastro esta como CPF, precisamos padronizar para CPF.

Melhorias:

- Autopreenchimento de campos na parte visitante onde se ja houver o CPF do visitante na base de dados ja preencher os dados de nome, telefone. Deixando a foto e as informacoes do veiculo e onde vai para ser preenchidas manualmente. Vale ressaltar que vamos permancer contendo um unico registro de visitante, precisamos analisar se para facilitar no banco de dados vamos criar um campo de "visitante_id" ou uma nova tabela de visitantes

- PG e Nome de guerra junto como um campo unificado de texto verificar a viabilidade de separar eles, como tem campos que tem dados em producao vamos precisar verificar uma forma de separar os dados sem perder a integridade dos dados, talvez criar um script para separar os dados e depois atualizar o banco de dados com os novos campos separados.

Melhorias Cosmeticas:

- As imagem de fundo do sistema e logo ainda nao atualiza para todos os dispositvos, uma opcao de marcar no docker compose e apontar as imegens no momento de instalar pode ser uma boa alternativa desenvolver isso na aplicao e colocar na ENV essas melhorias cosmeticas para serem feitas a nivel ambiente em vez de na aplicao.

Melhorias

Futuras taks:
Controle de sessao, mais de uma maquina consegue

Prompt: Eu estou sentindo falta de diversos recursos de um framework, com a implementacao facilitada de um DTO entre outros, seria vantagem fazermos uma refatoracao para o backend inicialmente usar um framework como o NestJS, que é baseado em Node.js e TypeScript, e oferece uma estrutura modular, injeção de dependência, suporte a DTOs, validação de dados e muito mais. Isso poderia melhorar a manutenção do código, facilitar a implementação de novas funcionalidades e aumentar a segurança da aplicação. Acho que tem coisas usando fastfy nesse projeto, ou algo assim porem nao sei se o fastfy tem um suporte de coisas como DTOs e validação de dados tão robusto quanto o NestJS, ou é mais uma LIB preciso fazer um brainstorming sobre se vale a pena ou nao ou apenas adquear esse projeto com fastfy para deixar bom. Me ajude a decidir inicialemente se vale a pena migrar para o NestJS ou continuar com o Fastify
