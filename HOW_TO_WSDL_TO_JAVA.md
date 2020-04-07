## Mãos a obra: Criação do projeto e geração das classes
Como mostrado no vídeo do capítulo, é preciso criar o projeto e gerar as classes a partir do WSDL. 
1) No Eclipse crie um novo projeto do tipo *Dynamic Web Project*. Chame o projeto estoque-web.

2) Associe o projeto com o Wildfly (arrastando o projeto encima do Wildfly).

3) Baixe o WSDL utilizado nesse treinamento disponível nesse link. Coloque o WSDL na pasta *src* do projeto estoque-web.

4) Agora vamos gerar as classes a partir desse WSDL. Abra um terminal e entre na pasta da instalação do wildfly: wildfly-8.2.0/bin. Nessa pasta deve ter o script *wsconsume*. Se você usa Windows utiliza-se o *wsconsume.bat*, no Linux ou Mac usa-se wsconsume.sh.
No Windows, assumindo que a pasta workspace está no C:\ execute:
**wsconsume.bat -k -n -o C:\workspace\estoque-web\src C:\workspace\estoque-web\src\EstoqueWSServiceCap5.wsdl**
Ajuste o comando se for necessário e acompanhe a saída para ver eventuais problemas:


    Loading FrontEnd jaxws ...
    Loading DataBinding jaxb ...
    wsdl2java -exsh false -d C:\workspac\eestoque-web\src -verbose -allowElementReferences file:C:\workspace\estoque-web/src/EstoqueWSServiceCap5.wsdl
    wsdl2java - Apache CXF 2.7.11
	
5) Por fim, volte no Eclipse e atualize o projeto estoque-web. No projeto deve aparecer o código fonte das classes geradas.

Usaremos as classes no próximo exercício.

###### O script wsconsume é apenas um wrapper para simplificar o uso da ferramenta wsdl2java que faz parte da implementação CXF que vem embutida no Wildfly (CXF é uma implementação da especificação JAX-WS). As classes geradas podem ser utilizadas para publicar o serviço ou criar o cliente!
