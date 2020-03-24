# Instalação do JBoss Wildfly 8

### configurar o servidor Wildfly no Eclipse

1) Para baixar o servidor de aplicação JBoss Wildfly acesse a página http://wildfly.org/downloads/. Baixe a versão 8 e extraia o arquivo.

2) Com Eclipse (Java EE) aberto instale o Server Adapter (**JBOSS TOOLS**)  para rodar o Wildfly dentro do Eclipse. 

3) Entre na View Servers e clique em New Server. 
Na janela procure o link Download additional server adapters. Na lista escolha JBoss AS Tools e confirme as próximas telas. Após a instalação, reinicie o Eclipse.

4) Com o Server Adapter instalado, configure o Wildfly como Servidor no Eclipse. Novamente, abra a View Servers e crie um New Server. Na lista escolha o Wildfly na versão 8 e configure o diretório de instalação do Wildfly. Após a configuração, o Wildfly deve aparecer na lista de servers.

5) Ainda na View Servers inicie o Wildfly. Fique atento ao console.

Você pode testar a instalação ao acessar http://localhost:8080/

Deve aparecer a página inicial do Wildfly.
