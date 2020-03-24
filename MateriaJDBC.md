# CURSO DE JDBC
###### concluído em 23/01/2018

## AULA 1
É muito comum que uma aplicação queira acessar um banco de dados para armazenar suas informações. Poderíamos escrever diretamente todo o código de conexão com um banco, por exemplo, MySQL. Mas se desejássemos mudar dele para um SQLServer, teríamos que reimplementar todo o protocolo de comunicação do servidor. Cada banco possui seu próprio protocolo, através do qual enviamos e recebemos requisições SQL, mas não queremos implementar algo tão baixo nível quanto o protocolo do banco.
Portanto precisamos de alguém que faça uma ponte entre nós e o banco. Algum tipo de interface comum a todos os bancos relacionais. Imaginemos uma interface de conexão com o banco:

```java
    public interface Connection {
      Result executeQuery(String sql);
    }
```
Cada um dos bancos nos forneceria uma API que devolve uma instância de Connection diferente. 
Se queremos um Mysql, fazemos:
```java
Connection mysql = new MysqlConnection("localhost", "usuario", "senha");
```
Ou se desejamos um SQLServer:
```java
Connection mysql = new SQLServerConnection("localhost", "usuario", "senha");
```
Tudo isso poderia funcionar, mas ainda estaríamos colocando a escolha do banco em nosso código, escolhendo isto na mão através do código compilado. Seria possível fazer essa escolha do banco através de uma String? Assim poderíamos posteriormente ler essa String de um arquivo de configuração e a escolha do banco ser feita automaticamente:
```java
Connection conexao = GerenciadorDeConexoes.getConnection("mysql", "localhost", "nome_do_banco", "usuario", "senha");
```
O código do gerenciador de conexões seria o de alguém que, de acordo com a String passada escolhe o componente adequado a ser instanciado e utilizado:
```java
public class GerenciadorDeConexoes {

  public static Connection getConnection(String tipo, String ip, String nome, String usuario, String senha) {

    if(tipo.equals("mysql")) {
      return new MySQLConnection(ip, nome, usuario, senha);
    } else if(tipo.equals("sqlserver")) {
      return new SqlServerConnection(ip, nome, usuario, senha);
    }

    throw new IllegalArgumentException("Tipo de banco não suportado");
  }
}
```
**Note que o papel do gerenciador de conexões é o de escolher a estratégia adequada de comunicação com o banco.** Ele escolhe alguém que sabe falar a língua do banco. **Ele escolhe o Driver que fala a língua do banco, e nos devolve uma conexão**. A estratégia escolhida depende somente da String que passamos. Seria interessante criarmos esse GerenciadorDeConexoes, ou GerenciadorDeDrivers, mas na verdade **ele já existe**:
```java
Connection connection = DriverManager.getConnection("jdbc:mysql://ip-do-banco/nome-do-banco", "usuario", "senha");
```
A interface **java.sql.Connection** também já existe e toda conexão que é aberta deve ser fechada:
```java
connection.close();
```
Neste curso utilizaremos o MySQL. 

- Criamos um novo projeto java no Eclipse chamado **loja-virtual**. 
- subimos o servidor do MySQL
- clicamos em new schema 
- damo-lhes o nome de loja-virtual

```sql
CREATE SCHEMA `loja-virtual` DEFAULT CHARACTER SET latin1 ;
```

- Agora conectamos ao banco via Java. Devemos** importar o driver** para o projeto. Clicamos no arquivo ‘***mysql-connector-java-5.1.45-bin.jar***’ e arrastamos ele para o projeto. Escolhemos "*copia*" e depois, no eclipse, clicamos com o botao direito em cima do arquivo e em *build path* e *add to build path*. 

- Criamos uma classe chamada **TestaConexao** e colocamos o código *main* que usa a string de conexão que conecta em **localhost** no banco chamado *loja-virtual*. Note que o driver do mysql especifica em sua documentação que a String de conexão deve começar:

```java
"jdbc:mysql://localhost:3306/loja-virtual", "root", "root" 
```

-  o método *main* deve estar desta forma:
```java
public static void main(String[] args) throws SQLException {
		Connection c = DriverManager.getConnection("jdbc:mysql://localhost:3306/loja-virtual", "root", "root");
		System.out.println("Conexao Aberta!");
 		c.close();
		System.out.println("Conexao Fechada!");
 }
```

-  Vamos agora criar uma tabela em nosso banco. 

```sql
CREATE TABLE Produto (id INTEGER PRIMARY KEY, nome VARCHAR(255), descricao VARCHAR(255));
```
Vamos inserir 2 novos produtos:

```sql
insert into Produto values (1, 'Geladeira', 'Geladeira duas portas');
```
```sql
insert into Produto values (2, 'Ferro de passar', 'Ferro de passar roupa com vaporizador');
```
E vamos conferir que os dois estão presentes:
```sql
select * from Produto;
```
Voltemos agora ao nosso código Java. Desejamos selecionar todos os produtos. Sabemos o código SQL, mas como executar este statement a partir de uma conexão com o banco (Connection)?
```java
Connection c = DriverManager.getConnection("jdbc:mysql://localhost:3306/loja-virtual", "root", "root");
connection.statement("insert ..."); //???
connection.close();
```
Pesquisando no Javadoc do JDBC encontramos o método createStatement, que possui o método execute:
```java
Connection c = DriverManager.getConnection("jdbc:mysql://localhost:3306/loja-virtual", "root", "root");
Statement stmt = connection.createStatement();
boolean resultado = stmt.execute("select * from Produto");
System.out.println("O resultado foi: " + resultado);
connection.close();
```
O resultado é **true** se obtemos uma lista de resultados como quando fazemos um select. Ao rodarmos nosso programa, o resultado é exatamente este, true. Mas como acessar o conjunto de dados que é o resultado da query? Queremos acessar o conjunto de resultados, o **ResultSet**. Para isso o **Statement** possui o método **getResultSet**:
```java
ResultSet resultSet = statement.getResultSet();
```
Após executar a query, o **cursor de busca é posicionado antes do primeiro resultado**. **Precisamos andar para o primeiro elemento**, então **resultSet.next()**. 

Agora que estamos no primeiro resultado, podemos usar o método **getString** (e similares) para **extrair o valor dos campos que queremos**:
```java
int id = resultSet.getInt("id");
String nome = resultSet.getString("nome");
String descricao = resultSet.getString("descricao");
```
Após terminarmos de analisar este primeiro elemento queremos entrar em um loop. Analisar o segundo, o terceiro etc. Então colocamos todo este código em loop:
```java
while(resultSet.next()) {
    int id = resultSet.getInt("id");
    String nome = resultSet.getString("nome");
    String descricao = resultSet.getString("descricao");
}
```
O que acontece é que quando executamos a query **o cursor fica posicionado antes do primeiro resultado**. Caso **não tenha nenhum resultado, o método next retorna false** e o loop não é executado. **Caso tenha um próximo resultado, entramos no loop** e processamos esta linha. A cada nova linha o loop é executado, até que não existem mais dados e o loop acaba. 

Em outras palavras, **o loop é executado para cada linha da tabela que é retornada**.

**Lembre-se que tudo que abrimos tem que ser fechado.** Portanto tanto o **ResultSet** quanto o **Statement** **devem ser fechados**.

```java
rs.close();
stmt.close();
```

Ficamos com um código final que executa um select e imprime os ids, nomes e descrições de tudo o que for retornado:

```java
	public static void main(String[] args) throws SQLException {
		Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/loja-virtual", "root", "root");
 		System.out.println("Conexao Aberta!");
 		
 		Statement stmt = connection.createStatement();
 		boolean resultado = stmt.execute("select * from Produto");
 		System.out.println("O resultado foi: " + resultado);

 		ResultSet resultSet = stmt.getResultSet();
 		
 		while(resultSet.next()) {
 		    int id = resultSet.getInt("id");
 		    String nome = resultSet.getString("nome");
 		    String descricao = resultSet.getString("descricao");
 		    
 		    System.out.println(id+"-"+nome+"-"+descricao);
 		}

 		resultSet.close();
 		stmt.close();
 		connection.close();
 		System.out.println("Conexao Fechada!");
	}
```


## AULA 2
Desejamos inserir um novo produto, um notebook, e para isso executamos o mesmo processo de antes.

criamos uma classe chamada **TestaInserçao** e colocamos um método *main*.
no método main:

```java
public static void main(String[] args) throws SQLException {
		Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/loja-virtual", "root", "root");
 		System.out.println("Conexao Aberta!");
 		
 		Statement stmt = connection.createStatement();
 		boolean resultado = stmt.execute("insert into Produto (nome, descricao) values ('Notebook', 'Notebook i5')");
 		System.out.println("O resultado foi: " + resultado);

 		stmt.close();
 		connection.close();
 		System.out.println("Conexao Fechada!");
	}
```

Executando o código acima obtemos o resultado esperado: **false**. E se fizermos um select no banco de dados temos que o notebook foi inserido com sucesso:

-  Apesar de ter funcionado, é interessante extrairmos o código que abre a conexão para um método separado. Criamos uma classe chamada **Database** com o método *getConnection*:
  ```java
 public static Connection getConnection() throws SQLException {
		Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/loja-virtual", "root", "root");
 		System.out.println("Conexao Aberta!");
		return connection;
	}
```

Apesar do código funcionar, **repare que não ficamos sabendo o ID que foi gerado dentro do código Java**. 

**Somente através de um select posterior seríamos capazes - de maneira complicada - de descobrir o id gerado. **

###### Na verdade a **API do Statement** nos entrega todas as chaves que foram geradas automaticamente caso utilizemos a configuração **Statement.RETURN_GENERATED_KEYS**.

Ao executar o statement passamos a opção mencionada como segundo argumento:

```java
boolean resultado = statement.execute("insert into Produto (nome, descricao) values ('Notebook', 'Notebook i5')", Statement.RETURN_GENERATED_KEYS);
```

Agora o método getGeneratedKeys nos retorna um ResultSet com todas as chaves geradas:

       

    ResultSet resultSet = statement.getGeneratedKeys();

Já sabemos iterar por um resultSet, portanto podemos imprimir todas as chaves que foram geradas. No nosso caso só será gerada a chave para o campo id, portanto fazemos um laço e imprimos o resultado do campo gerado ID:
     

      while (resultSet.next()) {
                String id = resultSet.getString("id");
                System.out.println(id + " gerado");
            }

Rodamos a aplicação mais algumas vezes até obter o resultado para o notebook:
5 gerado

Isso ocorreu pois temos agora três notebooks no banco. Executando o select no banco conseguimos notar esta repetição.

Queremos executar um delete para remover dois deles, para isto criamos uma nova classe **TestaRemocao**:
```java
public static void main(String[] args) throws SQLException {
		Connection connection = Database.getConnection();
 		
 		Statement stmt = connection.createStatement();
 		int resultado = stmt.executeUpdate("delete from Produto where id>3");
 		System.out.println("Foram atualizadas: " + resultado+ " linhas.");
 		stmt.close();
 		connection.close();
 		System.out.println("Conexao Fechada!");
	}
```

Mas antes de executar o delete, como descobrir quantas linhas foram removidas? Quero ter certeza que os dois notebooks extras serão removidos. 

Lembre-se que uma remoção também é uma atualização do banco. 

O método *getUpdateCount* diz o número de linhas atualizadas:
      

     int count = stmt.getUpdateCount();
     System.out.println(count + " linhas atualizadas");
	 
Pronto! Executamos o programa e temos sucesso: somente 2 linhas foram removidas. 

###### Como vimos, o JDBC é uma especificação que generaliza o acesso aos banco de dados relacionais, permitindo focarmos no SQL e trabalhar de maneira uniforme com a interface de Conexão, execução de Statement e resultados (ResultSet).


## AULA 3
Somos capazes de inserir novos produtos através de um Statement. Mas o que acontece se o nome do nosso produto é **Notebook'i5 2013**? Isso mesmo, com uma aspas simples no meio do nome?

O SQL resultante seria:


    insert into Produto (nome, descricao) values ('Notebook 'i5 2013', 'Notebook i5');

Nesse caso teremos uma **exception** pois o **SQL é inválido**. 

###### Acontece que toda vez que vamos concatenar dados de fora, como os dados que um usuário fornece, com uma query precisamos lembrar de fazer o "escape" das aspas e caracteres especiais. Precisamos tomar esse cuidado para evitar que o usuário final digite um valor que quebre nossa aplicação, ou ainda um código malicioso que altere os dados do banco de maneira indesejada.

Imagine uma query de login que faz a busca do usuário por login e senha:


    String sql = "select * from Usuario where login='" + login + "' and senha='" + senha + "'";
	
Caso o usuário forneça o login "xpto" e a senha "' or id=1" ele consegue o sql a seguir:


    select * from Usuario where login='xpto' and senha='' or id=1
	
**E acaba se logando como o usuário 1**. 

Esse tipo de código malicioso é chamado de **SQL Injection** e temos que evitá-lo tratando os caracteres especiais. Felizmente o JDBC já faz isso para nós através de um **PreparedStatement**. 

- Começamos passando **?** (pontos de interrogação) **no lugar dos valores** que queremos preencher:
       

    String sql = "insert into Produto (nome, descricao) values(?, ?)";
	
- Criamos ele como um Statement normal, mas já passando o SQL:
      

     String sql = "insert into Produto (nome, descricao) values(?, ?)";
            PreparedStatement statement = connection.prepareStatement(sql,
                    Statement.RETURN_GENERATED_KEYS);
                    statement.close();

Agora falta colocar os valores nas posições adequadas. 

Ao invés de serem as posições 0 e 1,** usamos os números 1 e 2** para marcar o primeiro e o segundo ponto de interrogação:
      

     statement.setString(1, nome);
            statement.setString(2, descricao);
			
Pronto, agora podemos executar nosso statement:

       

    boolean resultado = statement.execute();
	
A partir daqui podemos trabalhar com os métodos do statement como já estávamos acostumados. Note que com a simples adoção de um **PreparedStatement** com os **?** passamos a ter acesso a duas vantagens:
1. o JDBC passa a fazer o escaping de caracteres especiais e não nos preocupamos com SQL Injection.
1. o JDBC dá a chance ao banco de dados de escolher a melhor maneira de executar uma query (definir o plano de ação), e podemos executar a mesma query com parâmetros diferentes bastando chamar o método *setter* e o *execute* novamente.

###### Na prática, sempre que utilizamos JDBC puro acessamos um PreparedStatement ao invés de Statements normais.

Recapitulando o Prepared Statement:
- Altere sua classe **TestaInsercao**. 
- Crie as variáveis **nome** e **descrição**, colocando um nome de produto que contenha aspas simples. 
- Prepare o sql a seguir:


    String sql = "insert into Produto (nome, descricao) values(?, ?)";
	
- Use o prepared statement:


    PreparedStatement statement = connection.prepareStatement(sql,
                    Statement.RETURN_GENERATED_KEYS);
					
- Configure o nome e a descrição do seu produto:



    statement.setString(1, nome);        
    statement.setString(2, descricao);
	
- Execute o statement com o método *execute* sem nenhum argumento.



    boolean resultado = statement.execute();
	
- Execute o método *main* e veja como o produto é inserido com sucesso, mesmo que as aspas simples estejam dentro do nome do produto.

Pergunta: ***Quais os riscos de utilizar um Statement ao invés de um PreparedStatement?***

###### Os riscos de se utilizar um statement ao invés de um prepared statement são de erros com caracteres que não são escapados automaticamente, quebrando a query e de código malicioso que pode ser inserido no input do usuário (como um delete) que será injetado na query a ser executada e rodará sem erros.


## AULA 4
Vamos retomar o código de nossa inserção. Leve em consideração que ele inclui somente um produto:
       

    PreparedStatement stmt = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
    		stmt.setString(1, nome);
    		stmt.setString(2, descricao);
    		boolean resultado = stmt.execute();
     		System.out.println("O resultado foi: " + resultado);
    	 	ResultSet resultSet = stmt.getGeneratedKeys(); 
     		while (resultSet.next()) {
                int id = resultSet.getInt(1);
                System.out.println(id + " gerado");
            }
     		resultSet.close();
     		stmt.close();

Mas e se desejamos incluir dois produtos? Podemos extrair um método no momento em que chamamos o método *statement.setString*. Extraímos o método *adiciona*:


     public static void adiciona(Connection connection, String sql, String nome, String descricao) throws SQLException {
    		PreparedStatement stmt = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
    		stmt.setString(1, nome);
    		stmt.setString(2, descricao);
    
    		boolean resultado = stmt.execute();
    		
     		System.out.println("O resultado foi: " + resultado);
     		
     		ResultSet resultSet = stmt.getGeneratedKeys();
     		while (resultSet.next()) {
                int id = resultSet.getInt(1);
                System.out.println(id + " gerado");
            }
    
     		resultSet.close();
     		stmt.close();
    	}

E nosso método *main* passa a chamar o método *adiciona*:
              

     adiciona(connection, sql, "TV LCD", "TV de 32 polegadas");
	 
Da mesma maneira, podemos logo em seguida adicionar um segundo produto:
       

     adiciona(connection, sql, "TV LCD", "TV de 32 polegadas");
     adiciona(connection, sql, "Blueray", "Blueray azul");

###### Note que o statement será executado, terá seus parâmetros alterados e será executado novamente. 

Sem problemas, rodamos o programa e dois produtos são adicionados no banco! Fazemos o select no manager e temos os produtos lá.

Mas e se algo acontece de errado entre o primeiro e o segundo insert? 

O programa já adicionou o primeiro produto. Vamos testar? Removemos os produtos novamente:


    delete from Produto where id>3
	
E adicionamos um if dentro de nosso método adiciona para jogar uma Exception no caso do Blueray:
 

     public static void adiciona(Connection connection, String sql, String nome, String descricao) throws SQLException {
    		
    		//erro proposital
    		if (nome.equals("Blueray")) {
                throw new IllegalArgumentException("Problema ocorrido");
            }		
    		PreparedStatement stmt = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
    
            // resto do método adiciona 
    
        }
Executamos nosso programa e note que a TV é adicionada, mesmo que o Blueray gere um erro. Conferimos no banco de dados.

Isto é, a cada novo statement do tipo insert (update, delete e similares), **o comando é comitado automaticamente para o servidor**, não dá para voltar atrás. 

###### Este é o comportamento padrão de uma Connection segundo a especificação do JDBC. 

Mas e se eu quiser executar **os dois inserts ou nada**? 

Isto é, tudo ou nada. Ou executa os dois com sucesso ou nenhum. **Nesse caso precisamos desativar o auto commit**. Para isso fazemos logo após receber a conexão:
          

     connection.setAutoCommit(false);
	 
Voltamos no nosso banco de dados, removemos os produtos extras para ficar com 3 produtos novamente:


    delete from Produto where id>3
Se rodarmos novamente o programa e efetuarmos o select notamos que nada foi inserido no banco, nem a TV, nem o Blueray!

Quando o auto commit está configurado como false precisamos indicar o momento de executar o commit, e não fizemos isso ainda. Vamos testar colocar o commit logo após adicionar o primeiro e o segundo produto?
        

    adiciona(connection, sql, "TV LCD", "TV de 32 polegadas");
    connection.commit();
    adiciona(connection, sql, "Blueray", "Blueray azul");
    connection.commit();

Rodamos novamente e agora sim a TV é gravada mas o blueray não.

Vamos apagar novamente os produtos e ficar só com os 3 primeiros:


    delete from Produto where id>3
Queremos agora deixar explícito a maneira de commit e de voltar atrás em nossa decisão, de fazer um **rollback**. Isto é, deixamos explícito o commit no caso de sucesso dos dois itens adicionados:
          

    		try {
    			PreparedStatement stmt = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
    			
    			stmt.setString(1, nome);
    			stmt.setString(2, descricao);
    			boolean resultado = stmt.execute();
    			connection.commit();		
    	 		System.out.println("O resultado foi: " + resultado);
    	 		ResultSet resultSet = stmt.getGeneratedKeys();
    	 		while (resultSet.next()) {
    	            int id = resultSet.getInt(1);
    	            System.out.println(id + " gerado");
    	        }
    	 		resultSet.close();
    	 		stmt.close();
    		}catch (Exception e) {
    			e.printStackTrace();
    		}
E dentro do nosso catch desejamos voltar atrás, executar um rollback em nossa operação, connection.rollback:
        

     		}catch (Exception e) {
    			e.printStackTrace();
    			connection.rollback();
                System.out.println("Rollback efetuado");
    		}
Mas temos também que fechar o statement que abrimos, onde fechamos ele? Dentro do try? Mas e se acontecer uma exception, não fecharemos: precisamos do close dentro de um bloco **finally**. E precisamos conferir que ela foi aberto com sucesso. São muitos detalhes pequenos que precisamos cuidar para fechar tudo o que abrimos em blocos do tipo **try/catch/finally**! 

###### Para evitar ter que se preocupar com o caso de fechar tais recursos, o Java 7 introduziu a construção try(recurso). Se abrirmos uma conexão, ou qualquer coisa "fechável", ela será automaticamente fechada quando o bloco try terminar, seja através de um sucesso ou de uma exception/error. Para evitar ter que se preocupar com o caso de fechar tais recursos, o Java 7 introduziu a construção try(recurso). Se abrirmos uma conexão, ou qualquer coisa "fechável", ela será automaticamente fechada quando o bloco try terminar, seja através de um sucesso ou de uma exception/error. 

Portanto podemos abrir a conexão com:
      

    		try(Connection connection = Database.getConnection()) {
    	 		String sql = "insert into Produto (nome, descricao) values (?, ?)";
    	 		adiciona(connection, sql, "TV LCD", "TV de 32 polegadas");	
    	 		adiciona(connection, sql, "Blueray", "Blueray azul");
    		}
E podemos fazer a mesma coisa para o prepared statement:
        

    		try(PreparedStatement stmt = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
    			stmt.setString(1, nome);
    			stmt.setString(2, descricao);
    			boolean resultado = stmt.execute();
    					
    	 		System.out.println("O resultado foi: " + resultado);
    	 		ResultSet resultSet = stmt.getGeneratedKeys();
    	 		while (resultSet.next()) {
    	            int id = resultSet.getInt(1);
    	            System.out.println(id + " gerado");
    	        }
    	 		resultSet.close();
    		}
###### Note que agora não nos preocupamos mais em fechar os recursos que abrimos dentro do try. O compilador garante que eles serão fechados!

Se executarmos nosso sistema agora, podemos acessar o banco e verificamos que nada foi inserido. Inclusive o console mostra que o rollback foi efetuado. Isto é, enviamos os dois insert para o banco, mas como deu uma exception pedimos para ele voltar atrás em tudo, efetuar o rollback, portanto nada foi inserido.

Entao a classe **TestaInsercao** fica assim:


    public static void main(String[] args) throws SQLException {
    		try(Connection connection = Database.getConnection()) {
    	 		String sql = "insert into Produto (nome, descricao) values (?, ?)";
    	 		adiciona(connection, sql, "TV LCD", "TV de 32 polegadas");	
    	 		adiciona(connection, sql, "Blueray", "Blueray azul");
    	 		connection.commit();
    		}catch (Exception e) {
    			e.printStackTrace();
                System.out.println("Rollback efetuado");
    		}
     		System.out.println("Conexao Fechada!");
    }
    
    public static void adiciona(Connection connection, String sql, String nome, String descricao) throws SQLException {
    		//erro proposital
    		if (nome.equals("Blueray")) {
                throw new IllegalArgumentException("Problema ocorrido");
            }
    		try(PreparedStatement stmt = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
    			
    			stmt.setString(1, nome);
    			stmt.setString(2, descricao);
    			boolean resultado = stmt.execute();
    					
    	 		System.out.println("O resultado foi: " + resultado);
    	 		ResultSet resultSet = stmt.getGeneratedKeys();
    	 		while (resultSet.next()) {
    	            int id = resultSet.getInt(1);
    	            System.out.println(id + " gerado");
    	        }
    	 		resultSet.close();
    		}
    	}


## AULA 5
Até esse instante trabalhamos com aplicações que fazem uma única conexão ao banco de dados. Mas o que acontece em aplicações web ou cliente servidor onde temos dezenas, centenas ou até mesmo milhares de usuários acessando o serviço simultaneamente?

Se mantemos uma única conexão o tempo todo, assim que o primeiro usuário acessa e enquanto ele executa suas queries, o segundo usuário espera. Se um terceiro usuário fizer uma requisição ele terá que esperar o término das requisições do primeiro e do segundo usuário. Dessa maneira vamos enfileirando todas as requisições com uma única conexão para atendê-las. É similar a uma fila do banco: diversas pessoas vão entrando na fila e só existe um único caixa para atender, uma única conexão para atender. Enquanto o primeiro não termina, todos esperam.

Claramente, em geral, essa não é uma boa solução. Mas e se abrirmos e fecharmos uma nova conexão para cada novo usuário/requisição? O custo de abrir e fechar uma conexão é alto: é necessário enviar e receber dados de autenticação via TCP para o banco que é remoto. Além disso, se o número de usuários cresce muito, não teremos conexões suficientes no banco para responder pelo desejo dos usuários. O mesmo acontece na fila do banco: criar novos caixas e fechá-los a todo instante que chega um novo cliente é muito custoso, mas seria ótimo, claro.

Que tal o meio termo? Mantemos 5 conexões abertas o tempo todo e a medida que chegam usuários, eles acessam essas 5 conexões. Quando eles vão liberando, o próximo da fila entra e usa a conexão disponível. Desta maneira somos justos (todos serão atendidos na ordem que chegaram), disponibilizamos um número maior de recursos/conexões (não somente uma) mas também não sobrecarregamos com o número de conexões.

Essa abordagem, onde disponibilizamos um número limitado de objetos (conexões), é chamado de **pool**. Temos um **pool de conexões**, de onde virão os objetos para atender os requerimentos de cada cliente. A medida que os clientes terminam sua requisição, os objetos são devolvidos para esse pool.

###### Poderíamos implementar nosso pool de conexões mas os drivers do JDBC já implementam isso para nós (além de diversas bibliotecas Java), através da interface DataSource.

Vamos ao código? 

Altere o nome da Classe **Database** para **ConnectionPool**.

Vamos agora deixar de criar uma **Connection** nova a cada requisição e passar a usar um **pool**. 
Quando executarmos um new **Database** vamos criar nosso pool. 
No caso do **MySQL** usamos a classe **ComboPooledDataSource** do **C3PO**. 

O **MySQL** não tem uma classe dessas por padrão em suas bibliotecas, mas há outras duas bibliotecas muito famosas que são utilizadas para isso: **Apache DBCP** e **C3P0**.

###### Tem várias outras bibliotecas para isso, mas essas duas são bastante confiáveis. Inclusive, a C3P0 é muito utilizada junto com Hibernate.

Para utilizá-las, é bastante simples o processo. Você deve fazer o download do(s) jar(s) necessários no site deles e colocar no seu projeto.

Segue um código de exemplo de cada:
#### Apache DBCP


    import java.sql.*;
    import org.apache.commons.dbcp.*;
    
    public class C3POPConnectionExample {
        public static void main(String args[]) throws Exception {
            BasicDataSource dataSource = new BasicDataSource();
            dataSource.setDriverClassName("com.mysql.jdbc.Driver");
            dataSource.setUrl("jdbc:mysql://localhost:3306/mydatabase");
            dataSource.setUsername("root");
            dataSource.setPassword("");
            dataSource.setInitialSize(1);
            Connection con = dataSource.getConnection();
            System.out.println("Connection Object information : " + con);
        }
    }
	
#### C3P0


    import java.sql.*;
    import com.mchange.v2.c3p0.ComboPooledDataSource;
    
    public class DBCPConnectionExample {
        public static void main(String args[]) throws Exception {
            ComboPooledDataSource connectionPoolDatasource = new ComboPooledDataSource();
            connectionPoolDatasource.setDriverClass("com.mysql.jdbc.Driver");
            connectionPoolDatasource.setJdbcUrl("jdbc:mysql://localhost:3306/mydatabase");
            connectionPoolDatasource.setUser("root");
            connectionPoolDatasource.setPassword("");
            connectionPoolDatasource.setMinPoolSize(1);
            connectionPoolDatasource.setAcquireIncrement(5);
            connectionPoolDatasource.setMaxPoolSize(20);
            Connection con = connectionPoolDatasource.getConnection();
            System.out.println("Connection Object information : " + con);
        }
    }

  a classe final fica assim:


    public class ConnectionPool {
    	
    	//C3PO pool of connections
    	ComboPooledDataSource pool;
    	
    	ConnectionPool() throws Exception {
    		pool = new ComboPooledDataSource();
    		pool.setDriverClass("com.mysql.jdbc.Driver");
    		pool.setJdbcUrl("jdbc:mysql://localhost:3306/loja-virtual");
    		pool.setUser("root");
    		pool.setPassword("root");
    		pool.setMinPoolSize(1);
    		pool.setAcquireIncrement(5);
    		pool.setMaxPoolSize(20);
    	}
    	
    	Connection getConnection() throws SQLException {
     		System.out.println("Conexao Aberta!");
    		return pool.getConnection();
    	}
    }


Pronto! O que falta é armazenarmos esse pool como uma variável membro do nosso **Database**, além de a cada chamada do método *getConnection* invocarmos o método *pool.getConnection()*:

Repare que utilizamos a interface **DataSource** pois ela só disponibiliza os getters, não os setters. Não desejamos alterar os setters após a construção de nosso pool, portanto usamos a interface. 

Tiraremos também a característica static de nosso método: é importante criar um Database (e consequentemente o pool) antes de invocar o método.

Agora o nosso **TestaListagem** deixa de chamar o método diretamente (estaticamente) e passa a instanciar o **Database**:
       

    Database database = new Database();
    Connection connection = database.getConnection();
		   
Executamos a listagem e o resultado é o esperado, afinal estamos abrindo um pool de conexões e pegando uma única conexão.

Agora vamos executar o mesmo trabalho 100 vezes? 

Executamos mais uma vez o programa e abrimos o arquivo de log, vemos que somente as 4 conexoes do mysql estão lá.

Vamos colocar um for de 0 até 100 após abrir o connection pool e antes de abrir a conexão:



    public static void main(String[] args) throws SQLException {
    		ConnectionPool connectionPool;
    		
    		try{
    			
    			//a connection aqui abre somente uma connection pool
    			connectionPool = (ConnectionPool) new ConnectionPool();
    			
    			for(int i = 0; i < 100; i++) {
    				//a connection aqui abre várias conexões o que é oneroso
    //				connection = (ConnectionPool) new ConnectionPool();
    				
    				Connection c = connectionPool.getConnection();
    				
    				Statement statement = c.createStatement();
    				System.out.println("Statement Aberta!");
    				boolean resultado = statement.execute("select * from Produto");
    				ResultSet resultSet = statement.getResultSet();
    				System.out.println("ResultSet Aberto!");
    				
    				while(resultSet.next()) {
    					int id = resultSet.getInt("id");
    					String nome = resultSet.getString("nome");
    					String descricao = resultSet.getString("descricao");
    					System.out.println(id + ":" + nome +":"+ descricao);
    					
    				}
    				resultSet.close();
    				System.out.println("ResultSet Fechado!");
    				statement.close();
    				System.out.println("Statement Fechada!");
    				c.close();
    				System.out.println("Conexao Fechada!");
    			}
    			
    			
    			
    		}catch (Exception e ) {
    			e.printStackTrace();
    		}
    	}

Rodamos novamente e verificamos o log. Apesar de termos invocado o método getConnection e close 100 vezes, temos que somente uma conexão foi aberta e fechada! Perfeito! Quando o close é invocado, a conexão é devolvida para o pool de conexões. O próximo getConnection pede uma conexão do pool e recebe a conexão anterior!

Mas qual a grande diferença de um pool? Vamos ver o custo de abrir e fechar a conexão 100 vezes, basta mover a linha de criação do database para dentro do pool (conforme marcado em vermelho.
        
Rodamos a aplicação e agora a execução demora bastante tempo. Além disso, o arquivo de log fica cheio de conexões sendo fechadas. Com esse exemplo fica claro que não usar um connection pool pode ser custoso para uma aplicação cliente servidor como as aplicações web tradicionais.

Alguns servidores na cloud pedem para não usarmos connection pool pois eles mesmos já trazem seu connection pool ou lidam com tais situações. Os servidores Java EE também já fornecem um connection pool de maneira declarativa (configurando em algum arquivo externo a aplicação) e basta recebermos o **DataSource** dentro de nossa aplicação.

#### Pool

Em um pool simples com 9 conexões, o que acontece quando o 10º usuário se conecta e todas estão ocupadas? O que acontece quando, posteriormente, o terceiro usuário termina sua tarefa? Que variações você sugeriria fazer na implementação de um pool de conexões para evitar que um usuário espere muito tempo? E o que fazer para evitar que em horários de pico o servidor não fique com poucas conexões para um número grande de usuários?

###### Quando o 10º usuário se conecta, ele fica aguardando a liberação de uma conexão.

###### Quando o 3º usuário termina a sua tarefa, a conexão que foi liberada vai para o próximo usuário aguardando, neste caso, o 10º.Quando o 3º usuário termina a sua tarefa, a conexão que foi liberada vai para o próximo usuário aguardando, neste caso, o 10º.

acho que a melhor implementação de um pool de conexões é a quantidade de conexões ser dinâmica, tendo um mínimo que fica aberto e quando a quantidade de usuários aumenta e chega perto do máximo, este é acrescido de mais quantidade de conexões abertas.  quando o pico de utilização desce, as conexões abertas desnecessariamente vão sendo fechadas dinamicamente.

mas, uma abordagem estática, uma quantidade razoável de conexões deve ser aberta para possibilitar que mais usuários usem as conexões e na fiquem na lista de espera.


## AULA 6
Vamos criar a classe **Produto** e colocá-la no pacote **modelo**:


    package br.com.caelum.jdbc.modelo;
    
    public class Produto {
    
        private Integer id;
        private String nome;
        private String descricao;
		
Vamos criar agora um novo exemplo que tenta inserir um **Produto** no banco. 

Para isso adicionamos uma nova classe de teste e instanciamos um **Produto**. 

###### Note que passaremos no construtor tudo o que um produto realmente precisa:Note que passaremos no construtor tudo o que um produto realmente precisa:
  

     public static void main(String[] args) throws SQLException {
            Produto mesa = new Produto("Mesa Azul", "Mesa com 4 pés");
    
            try (Connection con = new ConnectionPool().getConnection()) {
                // desejamos salvar um produto!
            }
    
        }
Crie o construtor.

Vamos voltar ao nosso teste e criar tudo o que precisamos para salvar o produto:


    public class InsereProduto {
    	static ConnectionPool connectionPool;
    
    	public static void main(String[] args) throws SQLException {		
    		Produto produto = new Produto("Pneu de carro", "Pneu de carro aro 18");
    		
    		try{	
    			connectionPool = (ConnectionPool) new ConnectionPool();
    			Connection connection = connectionPool.getConnection();
    			connection.setAutoCommit(false);
    			String sql = "insert into Produto (nome, descricao) values (?, ?)";
    			adiciona(connection, sql, produto);
    	 		connection.commit();
    		}catch (Exception e) {
    			e.printStackTrace();
                System.out.println("Rollback efetuado");
    		}
    	}
    
    	private static Produto adiciona(Connection connection, String sql, Produto produto) throws SQLException {
    			try(PreparedStatement stmt = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
    				stmt.setString(1, produto.getNome());
    				stmt.setString(2, produto.getDescricao());
    				boolean resultado = stmt.execute();
    						
    		 		System.out.println("O resultado foi: " + resultado);
    		 		ResultSet resultSet = stmt.getGeneratedKeys();
    		 		while (resultSet.next()) {
    		 			produto.setId(resultSet.getInt(1));
    		            System.out.println(produto.getId() + " gerado");
    		        }
    		 		resultSet.close();
    		 		return produto;
    			}	
    	}
    }
	
Executamos nosso programa e agora podemos encontrar a produto no banco:

Vou sobrescrever agora o método *toString* de nosso **Produto**:
   

    @Override
        public String toString() {
            return String.format("[produto: %d %s %s]", id, nome, descricao);
        }
Rodamos diversas vezes e temos a saída que mostra qual o produto que foi inserido:

Para deixar o banco limpo, voltamos a ter somente três produtos:



    delete from Produto where id>3
	
Mas toda vez que eu faço um select, um insert, um delete, eu tenho que fazer os try de Connection, try PreparedStatement, try ResultSet etc. Tudo que acessa o banco de dados ficará espalhado na minha aplicação. Não parece ser uma boa alternativa, uma vez que quem mantém o código terá que conhecer todos os pontos que a aplicação acessa o banco, o que costuma ser um ponto comum de falha. 

Precisamos criar um **DAO** e colocar toda a parte de abrir conexoes nele:

    package br.jdbc.dao;
    
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    import java.sql.Statement;
    
    import br.jdbc.Connection.ConnectionPool;
    import br.jdbc.modelo.Produto;
    
    public class ProdutoDao {
    	private ConnectionPool connectionPool;
    	private Connection connection;
    	
    	public ProdutoDao() throws SQLException, Exception {
    		super();
    		getConexao();
    	}
    	
    	public Produto salva(Produto produto) {
    		try{	
    			String sql = "insert into Produto (nome, descricao) values (?, ?)";
    			produto = adiciona(sql, produto);
    	 		
    		}catch (Exception e) {
    			e.printStackTrace();
                System.out.println("Rollback efetuado");
    		}
    		return produto;
    	}
    
    	private Produto adiciona(String sql, Produto produto) throws SQLException {
    		try(PreparedStatement stmt = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
    			stmt.setString(1, produto.getNome());
    			stmt.setString(2, produto.getDescricao());
    			boolean resultado = stmt.execute();
    					
    	 		System.out.println("O resultado foi: " + resultado);
    	 		ResultSet resultSet = stmt.getGeneratedKeys();
    	 		while (resultSet.next()) {
    	 			produto.setId(resultSet.getInt(1));
    	            System.out.println(produto + " gerado");
    	        }
    	 		resultSet.close();
    	 		
    		}	
    		connection.commit();
    		return produto;
    	}
    
    	public void getConexao() throws Exception, SQLException {
    		connectionPool = (ConnectionPool) new ConnectionPool();
    		connection = connectionPool.getConnection();
    		connection.setAutoCommit(false);
    	}
    
    }
    
Já no nosso **InsereProduto**  ficará assim:


    public class InsereProduto {
    	public static void main(String[] args) throws Exception {		
    		Produto produto = new Produto("Pneu de carro", "Pneu de carro aro 18");
    		
    		new ProdutoDao().salva(produto);
    	}
    }


Sem nenhuma parte relativa a conexão. Mais limpa e com a informação em um só lugar. Mais facil de reaproveitar o codigo.

###### O **DAO** é o objeto que centraliza o acesso aos dados. Como temos em geral diversas coisas a serem feitas com o acesso ao repositório de dados, é comum que seja criado pelo menos um DAO por cada modelo que trabalhamos em nosso projeto.

O **DAO** é um **padrão de design** que utilizamos para **isolar o código SQL** (ou qualquer outro código de acesso à um repositório de dados). Ao adotá-lo, sabemos que existe um único grupo de classes que trabalha com um sistema externo de dados, e podemos nos preocupar somente com essas classes quando trabalharmos nessa área.


## AULA 7
Vamos criar agora uma nova tabela para trabalharmos com relacionamentos entre ela e a tabela **Produto**. O nome dela será **Categoria**, contendo um id gerado automaticamente (chave primária) e o nome (string tradicional):



    create table Categoria (id  INTEGER PRIMARY KEY auto_increment, nome varchar(255));
Ao atualizarmos a view do mysql conseguimos visualizar tanto a tabela **Categoria** quanto a **Produto**:

Agora vamos atualizar nossos produtos existentes para colocá-los em determinadas categorias. Para isso faremos primeiro um select para verificar quais produtos existem no banco:


    select * from Produto
Temos uma geladeira, um ferro, um notebook e duas mesas. Criaremos três categorias para agrupar estes produtos:

- eletrodomésticos para a geladeira e o ferro;
- eletrônicos para o notebook e móveis para as mesas. 

Executaremos cada um dos sql a seguir sem os ponto e vírgula do fim da linha e um por vez:


    insert into Categoria (nome) values ('Eletrodoméstico');


    insert into Categoria (nome) values ('Eletrônico');


    insert into Categoria (nome) values ('Móvel');

Agora podemos conferir nossas categorias:


    select * from Categoria;

Precisamos encontrar alguma maneira de dizer que o produto 1 está na categoria 1, o produto 2 na categoria 1, o produto 3 na categoria 2 etc. Isto é, precisamos dizer o id da categoria para cada um dos nossos produtos, portanto adicionaremos o campo categoria_id dentro da tabela Produto:



    alter table Produto add column categoria_id integer;
	
E conferimos que a coluna foi criada com sucesso. Note que as categorias estão como null por padrão: no nosso caso não obrigamos nenhum produto a ter uma categoria.


    select * from Produto
Marcaremos agora o produto 1 e 2 com a categoria de Eletrodoméstico (categoria 1):


    update Produto set categoria_id=1 where id in (1,2);
	
Marcamos o produto 3 com a categoria Eletrônico:



    update Produto set categoria_id=2 where id in (3);
	
E marcamos os outros produtos com a categoria Móvel:


    update Produto set categoria_id=3 where id > 3;
	
Conferimos novamente nossos produtos usando o select:

Agora atacaremos o código do modelo, nosso código Java. Analogamente ao que fizemos com a tabela **Produto**, criaremos agora uma classe chamada **Categoria**, com id e nome. Adicionamos o construtor com os dois argumentos:


    public class Categoria {
     
     	private Integer id;
     	private String nome;
     	private List<Produto> produtos;
     
     	public Categoria() {
      		super();
    	}
     
     public Categoria(Integer id, String descricao) {
      	super();
      	this.id = id;
      	this.nome = descricao;
     }
    //getters e setters
     @Override
     public String toString() {
      	return "["+id+"]"+" " +nome;
     }
    }

Criamos agora nosso teste **TestaCategorias** com o método *main* 
 

    public static void main(String[] args) throws Exception {   
      List<Categoria> categorias = new CategoriaDao().lista();
      
      for (Categoria cat : categorias) {
       for (Produto prod : new ProdutoDao().lista(cat)) {
        System.out.println("Categoria: "+ cat + " - Produto: "+ prod);
        
       }
      }
     }
 
Agora que sabemos o que precisamos no nosso DAO, vamos criá-lo:


    public List<Categoria> lista() {
      	List<Categoria> resultado = new ArrayList<>();
      
		  try {
			Statement statement = connection.createStatement();
			statement.execute("select * from Categoria");
			ResultSet resultSet  = statement.getResultSet();
			while(resultSet.next()) {
				Categoria cat = new Categoria();
				cat.setId(resultSet.getInt("id"));
				cat.setNome(resultSet.getString("nome"));
				resultado.add(cat);
			}
			resultSet.close();
			statement.close();
			connection.commit();
		  } catch (SQLException e) {
			e.printStackTrace();
		  }
		  return resultado;
     }
	 
Pronto. Criamos uma lista de categorias, executamos a query, para cada resultado adicionamos a categoria à lista e finalmente retornamos ela completa.

Temos que adicionar o getter do nome da categoria, claro. 

Rodamos o programa e temos na saída a lista das categorias.

Agora criaremos nosso outro método de listagem, algo como busca e recebe a categoria:
   

    public List<Produto> lista(Categoria categoria) {
      List<Produto> resultado = new ArrayList<>();
      
      try {
       	String sql = "select * from Produto where categoria_id = ?";
       	PreparedStatement statement = connection.prepareStatement(sql);
       	statement.setInt(1, categoria.getId());
       	statement.execute();
      	 ResultSet resultSet = statement.getResultSet();
       	while(resultSet.next()) {
        	Produto produto = transformaResultadosEmProdutos(resultSet);
        	resultado.add(produto);
       	}
       	resultSet.close();
       	statement.close();
       	connection.commit();
      } catch (SQLException e) {
       	e.printStackTrace();
      }
      return resultado;
     }
	 
Adicionamos também o getter do id da categoria. Pronto. 

Voltamos à nossa classe de teste e adicionamos a invocação ao método de busca do **ProdutosDAO**:
 

    public static void main(String[] args) throws Exception {   
      	List<Categoria> categorias = new CategoriaDao().listaComProdutos();
      
      	for (Categoria cat : categorias) {
       		for (Produto prod : cat.getProdutos()) {
        		System.out.println("Categoria: "+ cat + " - Produto: "+ prod);
        
       		}
      }
 

Agora basta executarmos nosso programa para ter o resultado que desejávamos.

Mas quantas queries executamos? Uma? Duas? Quatro. 

Note que executamos uma primeira query para trazer todas as categorias. Depois disso executamos uma query para cada categoria, isto é: mais 3 queries. Se nosso sistema possui 1000 categorias, executaríamos 1001 queries. Estamos executando **N+1** queries, onde **N é o número de elementos retornados pela primeira pesquisa**. Isto é muito ruim, uma vez que cada pesquisa tem que ir e voltar de um sistema remoto, serializando dados etc.

Como podemos evitar executar uma query para cada categoria? 

Poderíamos trazer todos os produtos de uma única vez, através de um join:


    select c.id as id_cat, c.nome as nome_cat, p.id as id_p, p.nome as nome_prod, p.descricao as desc_prod, p.categoria_id as cat_prod from Categoria as c
    join Produto as p on p.categoria_id = c.id
	
Pronto. Com um único join trouxemos todos os dados da categoria e dos produtos que desejamos. Executamos essa query no banco de dados.

###### O join traz os valores da categoria uma vez para cada produto que ela possui.

O join traz os valores da categoria uma vez para cada produto que ela possui. Isto é bom para os produtos, mas não tão bom para as categorias. 

Executando novamente temos que cada campo tem um nome distinto: o prefixo "p" é usado para os campos do produto e o prefixo "c" para os campos da categoria.

Agora precisamos implementar um novo método de listagem que já traga as categorias e os produtos de uma só vez. Chamemos este método de *listaComProdutos* em nosso **CategoriasDAO**:


    public List<Categoria> listaComProdutos() {
      List<Categoria> resultado = new ArrayList<>();
      Categoria ultima = null;
      
      try {
       Statement statement = connection.createStatement();
       statement.execute("select c.id as id_cat, c.nome as nome_cat, p.id as id_p, p.nome as nome_prod, p.descricao as desc_prod, "
         + "p.categoria_id as cat_prod from Categoria as c\r\n" +
         "join Produto as p on p.categoria_id = c.id "
         + "order by id_cat");
       ResultSet resultSet  = statement.getResultSet();
       while(resultSet.next()) {
        if(ultima==null || !ultima.getNome().equals(resultSet.getString("nome_cat"))) {
                    	Categoria categoria = new Categoria(resultSet.getInt("id_cat"), resultSet.getString("nome_cat"));
                    	resultado.add(categoria);
                    	ultima = categoria;
                	}
        
        Produto prod = new Produto();
        prod.setId(resultSet.getInt("id_p"));
        prod.setNome(resultSet.getString("nome_prod"));
        prod.setDescricao(resultSet.getString("desc_prod"));
        prod.setCategoria(ultima.getId());
        
        if(ultima.getProdutos() == null) {
         ultima.setProdutos(new ArrayList<>());
        }
        
        ultima.getProdutos().add(prod);
       }
       resultSet.close();
       statement.close();
       connection.commit();
      } catch (SQLException e) {
       e.printStackTrace();
      }
      return resultado;
     }
	 
Alteramos nosso método de teste para utilizar o novo método listaComProdutos:

    

    public static void main(String[] args) throws Exception {   
      List<Categoria> categorias = new CategoriaDao().listaComProdutos();
      
      for (Categoria cat : categorias) {
       for (Produto prod : cat.getProdutos()) {
        System.out.println("Categoria: "+ cat + " - Produto: "+ prod);
        
       }
      }
     }

Nosso código dentro do laço se resume então a carregar o id e o nome da categoria, verificar se a categoria mudou e em caso positivo trocá-la, criar um produto e adicionar o produto na categoria.

Pronto, rodamos nossa aplicação e temos que com uma única query e um join trouxemos todas as categorias com todos os produtos, evitando o problema do N+1.

Mesmo assim, o **problema do N+1** é muito famoso e até mesmo o mal uso de tais bibliotecas pode causar efeitos negativos em uma aplicação.

Faça a escolha da biblioteca que deseja utilizar (JDBC, Hibernate, JPA etc) de acordo com as necessidades de seu projeto e otimize as queries também de acordo com aquilo que você e sua equipe precisam.
 
 
mais informações úteis:
http://blog.caelum.com.br/os-7-habitos-dos-desenvolvedores-hibernate-e-jpa-altamente-eficazes/


