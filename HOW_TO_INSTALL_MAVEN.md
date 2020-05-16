##Introdução ao Apache Maven
para baixar o maven, vá em maven.apache.org.

para o** eclipse juno e versões anteriores**, é necessário baixar o plugin do maven. isso é possivel no endereço http://download.jboss.org/jbosstools/updates/m2eclipse-wtp/

para o **eclipse Kepler e posteriores**, o plugin já vem instalado, mas podemos baixá-lo em:
http://download.eclipse.org/m2-wtp/releases/kepler/

mas vamos utilizar a versão externa do maven.

no eclipse, para instalar o plugin (nao necessario a partir da versão kepler), vá em ‘**help**’ e ‘**install new software**’

no campo ‘**work with**’, cole o link do plugin.

selecione os pacotes: **m2e maven archiver connector**, **maven integration for eclipse** e **maven integration for WTP**.

clique em next e aceite os termos.


após extrair o zip do maven, va no eclipse> **window** > **preferences** > **maven** > **installations**
e inclua no eclipse a versão externa do maven.

na pasta do maven, vá em ‘**conf**’ e edite o arquivo **settings.xml**, adicionando o local do repositório local:

**C:\luiz-estudo\MAVEN REPOSITORIES**

no eclipse> **window** > **preferences** > **maven** > **user settings**, coloque o caminho para o arquivo **settings.xml**
