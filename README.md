### O que é SQL Injection?

SQL Injection é uma técnica de ataque que explora vulnerabilidades em sistemas que interagem com bancos de dados SQL. 
O ataque ocorre quando um invasor insere instruções SQL maliciosas em campos de entrada de um formulário web ou parâmetros de URL, manipulando assim a execução das consultas SQL pelo banco de dados.



### Código vulnerável

O código abaixo é de uma aplicação Java capaz de consultar produtos de uma base de dados SQL.

```java
import java.sql.*;

public class SusceptibleToSQLInjection {
    public static void main(String[] args) {
        // Termo de pesquisa fornecido pelo usuário, potencialmente malicioso
        String termoPesquisa = "'; DROP TABLE produtos;--";

        // Dados de conexão com o banco de dados
        String url = "jdbc:mysql://localhost:3306/database";
        String user = "root";
        String pass = "password";

        try {
            // Estabelece conexão com o banco de dados
            Connection conn = DriverManager.getConnection(url, user, pass);
            
            // Criação de uma instrução SQL utilizando concatenação de strings
            Statement stmt = conn.createStatement();
            String query = "SELECT * FROM produtos WHERE nome='" + termoPesquisa + "'";
            
            // Execução da consulta SQL
            ResultSet rs = stmt.executeQuery(query);

            // Exibicação do resultado da pesquisa
            while (rs.next()) {
                String nomeProduto = rs.getString("nome");
                System.out.println("Produto encontrado: " + nomeProduto);
            }

            // Fecha a conexão com o banco de dados
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```



### Solução

Para solucionar o problema, utiliza-se o PreparedStatement, ao invés de concatenar o termo de pesquisa diretamente na consulta SQL, . 
Com o PreparedStatement, o termo de pesquisa é tratado como um valor de parâmetro e é inserido na consulta usando um marcador de posição (?). 
Quando o PreparedStatement é executado, o valor do parâmetro é substituído no local do marcador de posição. 
Isso impede que o termo de pesquisa fornecido pelo usuário seja interpretado como parte da estrutura da consulta SQL, eliminando assim a vulnerabilidade de SQL Injection.

```java
import java.sql.*;

public class SafeFromSQLInjection {
    public static void main(String[] args) {
        // Termo de pesquisa fornecido pelo usuário, potencialmente malicioso
        String termoPesquisa = "'; DROP TABLE produtos;--";

        // Dados de conexão com o banco de dados
        String url = "jdbc:mysql://localhost:3306/database";
        String user = "root";
        String pass = "password";

        try {
            // Estabelece conexão com o banco de dados
            Connection conn = DriverManager.getConnection(url, user, pass);
            
            // Criação de uma consulta parametrizada utilizando PreparedStatement
            String query = "SELECT * FROM produtos WHERE nome=?";
            PreparedStatement pstmt = conn.prepareStatement(query);
            
            // Define o valor do parâmetro usando setString(), o que previne SQL Injection
            pstmt.setString(1, termoPesquisa);
            
            // Execução da consulta SQL
            ResultSet rs = pstmt.executeQuery();

            // Iteração sobre o resultado da consulta
            while (rs.next()) {
                String nomeProduto = rs.getString("nome");
                System.out.println("Produto encontrado: " + nomeProduto);
            }

            // Fecha a conexão com o banco de dados
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
