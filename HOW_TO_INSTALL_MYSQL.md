### Instalando o MySQL no windows
- fazer o download do mysql

- para instalar o mysql, executar o setup do instalador.

- configuração normal.

- inicializar o mysql como serviço manualmente :
painel de controle
administrative tools
services
na lista, encontrar o mysql e clicar em start


### Instalando o MySQL no Linux Ubuntu
A demonstração da instalação do MySQL abaixo foi realizada no Ubuntu 12.04 32-bit.

- Abra um terminal e execute o comando: **sudo apt-get install mysql-server-5.5 mysql-workbench**
- Digite **y** para as próximas confirmações e o início do download do repositório 
###### IMPORTANTE: Você precisa estar conectado a internet.
- Para iniciar o servidor, execute o comando: **sudo service mysql start**
- Para abrir o workbench você pode na linha de comando digitar: **mysql-workbench** ou utilizar o atalho criado.
Você então irá ver a tela inicial do workbench 
- Clicando em "**localhost**" na caixa "**Open Connection to Start Querying**", você irá para a o workbench e poderá fazer todas as consultas.
