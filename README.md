# DVWA - SQLI Manual

### **In-Band SQL Injection**

- **O que é SQL Injection:** Antes de entender o **In-Band SQL Injection**, é importante saber o que é uma injeção de SQL. Ela é uma vulnerabilidade de segurança que permite que um invasor insira comandos SQL maliciosos em uma consulta de banco de dados através de um formulário da web ou outro ponto de entrada, geralmente em campos de entrada de texto, mas não só em campos de texto.
- **O que é In-Band SQL Injection:** In-Band SQL Injection é o tipo mais comum de injeção de SQL. Pois nesse tipo de injeção, as informações são mostradas diretamente, diferente do Blind SQLi, que não mostra informações diretamente.

---

### Preparação

- Primeiro precisamos saber qual a abertura de SQLi que existe.
- Primeiro vamos testar se ele aceita sem caracteres especiais.
    
    ![Untitled](DVWA%20-%20SQLI%20Manual/Untitled.png)
    
- Aparentemente não. Então vamos testar com caracteres especiais.
- Vamos colocar um ‘ para ver o que acontece.
    
    ![Untitled](DVWA%20-%20SQLI%20Manual/Untitled%201.png)
    
    ![Untitled](DVWA%20-%20SQLI%20Manual/Untitled%202.png)
    

---

- Então a ' não está sendo filtrada e podemos usá-la daqui para frente.
- Mas para evitar erros, temos que comentar o resto da linha, assim ele executará nossa linha de comando e ignorará o restante dos comandos subsequentes da mesma linha.
- Para isso, podemos usar `--`  e `#` .
- É importante lembrar que deve-se usar um espaço após esses caracteres, caso contrário, eles ficarão colados no próximo comando e causarão erro.
- Então nosso comando ficará entre ' e -- ou #, resultando em `' <comando> --`  ou `' <comando> #` .

---

### Números de colunas

- Como esse método usa o UNION, precisamos saber quantas colunas existem para evitar erros e garantir que as informações venham corretamente.
- Então existem dois modos de determinar quantas colunas existem, usando o UNION ou o ORDER BY
- Isso será tentativa e erro, então é nessa etapa que muitas pessoas acabam recorrendo a ferramentas como:
    - **SQLMap:** Uma ferramenta poderosa e versátil que automatiza a maior parte do processo de teste de SQIi.
    - **jSQL Injection:** Uma ferramenta Java que automatiza a detecção de vulnerabilidades de SQIi em aplicações web.
    - **BBQSQL:** Uma ferramenta especializada em explorar vulnerabilidades de SQIi "blind".
    - **NoSQLMap:** Uma ferramenta similar ao SQLMap, mas focada em bancos de dados NoSQL.
    - **Whitewidow:** Um scanner de vulnerabilidades que inclui testes de SQIi.
    - **DSSS (Damn Small SQLi Scanner):** Um scanner leve e rápido para detectar vulnerabilidades de SQIi.

---

- Então vamos aos testes.
    - `' order by 1 --`
        
        ![Untitled](DVWA%20-%20SQLI%20Manual%203c4d8ceba5314134b2a0a590b3e64ad2/Untitled%203.png)
        
        - Perceba que nada aconteceu, então o número de colunas está correto ou é menor do que o esperado.
    
    ---
    
    - `' union select 1 --`
        
        ![Untitled](DVWA%20-%20SQLI%20Manual%203c4d8ceba5314134b2a0a590b3e64ad2/Untitled%204.png)
        
        - Já com o UNION é diferente, pois ele requer uma quantidade exata.

---

- Continuando com os testes
    - `' order by 2 --`
        - Perceba que nada aconteceu novamente, então o número de colunas está correto ou é menor do que o esperado.
    
    ---
    
    - `' union select 1,2 --`
        
        ![Untitled](DVWA%20-%20SQLI%20Manual%203c4d8ceba5314134b2a0a590b3e64ad2/Untitled%205.png)
        
        - Agora sim, perceba que funcionou
    
    ---
    
- Então, basta aumentar de 1 em 1 a quantidade, ou seja, se for com ORDER BY, o próximo valor será 3; caso seja com UNION SELECT, serão 1, 2, 3 e assim por diante.
- A quantidade de colunas acaba sendo visível.
- Sendo o número 1 a primeira coluna e 2 a segunda coluna.
- Então é só procurar onde estão essas informações.

---

### Pegando nome das tabelas

- Agora podemos substituir os valores dos parâmetros do SELECT por funções do SQL.
- A primeira função que vamos usar é o database().
    - Ela retorna o nome do banco de dados atual.
    - `' UNION SELECT null,database() --`
    
    ![Untitled](DVWA%20-%20SQLI%20Manual/Untitled%206.png)
    
    - Retornou `dvwa`

---

- Agora vamos consultar o nome das tabelas dentro do banco de dados.
    - Agora, no lugar do database(), vamos colocar outra função o `group_concat(table_name)`
        - Essa função, junto com o parâmetro `table_name`, concatena os nomes das tabelas em uma única string.
    
    ---
    
    - Então, para continuar, precisamos do FROM.
        - `FROM information_schema.tables`
            - Quando você usa `FROM information_schema.tables`, está basicamente dizendo ao banco de dados para buscar informações sobre as tabelas do banco de dados no qual você está conectado.
    
    ---
    
    - Ultimo filtro
        - `WHERE table_schema = 'dvwa'`
            - Filtra as tabelas apenas para aquelas que estão no esquema 'dvwa'.
    
    ---
    
    - Comando completo
        - `' UNION SELECT null,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'dvwa' --`
    
    ---
    
    - Resultado
        
        ![Untitled](DVWA%20-%20SQLI%20Manual%203c4d8ceba5314134b2a0a590b3e64ad2/Untitled%207.png)
        

---

### Pegando nome das colunas

- Agora, em vez de solicitar os nomes das tabelas, vamos pedir os nomes das colunas.
- Então, no lugar de `table_name`, será `column_name`.
    - Concatena os nomes das colunas em uma única string.

---

- No lugar de `information_schema.tables`, será `information_schema.columns`.
    - A tabela `information_schema.columns` é uma tabela de metadados que contém informações sobre as colunas presentes no banco de dados

---

- E no lugar de `WHERE table_schema = 'dvwa' --`  será `WHERE table_name = 'users' --` .
    - Filtra as tabelas apenas para aquelas que estão '`users`'.

---

- Comando completo
    - `' UNION SELECT null,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'users' --`

---

- Resultado
    
    ![Untitled](DVWA%20-%20SQLI%20Manual/Untitled%208.png)
    

---

### Pegando dados

- Agora basta fazer uma query pedindo o que queremos saber.
- Dentro do group_concat, vamos pegar o valor de user e password, e vamos aproveitar para formatar a saída de dados.
    - `' UNION SELECT null,group_concat(user,': ',password SEPARATOR '<br>') FROM users --`
    
    ![Untitled](DVWA%20-%20SQLI%20Manual/Untitled%209.png)
    

---

### Descriptografando senhas

- Um jeito simples de descriptografar algo é com o [CrackStation](https://crackstation.net/).
- Basta colocar as senhas nele e ver o resultado.

![Untitled](DVWA%20-%20SQLI%20Manual/Untitled%2010.png)
