# SQL Anti Padrões 🧙‍♂️

Este repositório destaca práticas comuns, **mas ineficientes**, em SQL e como corrigi-las. É destinado a desenvolvedores que estão aprendendo a otimizar suas consultas.

- Apresentação Pessoal: 

    - Nome: Tiago Linhares;
    - Cargo: Analytics Engineer;
    - [Blog](https://linhares015.github.io/);
    - [Curso Sql Master Gratuito](https://github.com/Linhares015/curso_sql);
    - [30 Dias de Desafio SQL](https://github.com/Linhares015/desafio_sql_30_dias);
    - [Linkedin](https://www.linkedin.com/in/tiago-linhares/);
    - [Github](https://github.com/Linhares015);
    - [Livro - Guia para se Tornar um Analista de Dados](https://www.amazon.com.br/dp/B0CDDFZMLD?ref_=cm_sw_r_mwn_dp_VT4QMG06XS904M6EEQ3A).

## Menu

- [1. SELECT](#1-select)
- [2. Não usar índices](#2-não-usar-índices)
- [3. Uso excessivo de JOINs](#3-uso-excessivo-de-joins)
- [4. Uso de Funções em Colunas em Cláusulas WHERE](#4-uso-de-funções-em-colunas-em-cláusulas-where)
- [5. Não usar cláusula LIMIT/OFFSET](#5-não-usar-cláusula-limitoffset)
- [6. Uso de Subconsultas Não Correlacionadas](#6-uso-de-subconsultas-não-correlacionadas)
- [7. Consultas com Lógica Complexa](#7-consultas-com-lógica-complexa)
- [8. Não usar Normalização](#8-não-usar-normalização)
- [9. Hardcoding de Valores](#9-hardcoding-de-valores)
- [10. Tabelas sem Chave Primária](#10-tabelas-sem-chave-primária)
- [11. Uso de Campos VARCHAR para Datas](#11-uso-de-campos-varchar-para-datas)
- [12. Não usar Parâmetros em Consultas Dinâmicas](#12-não-usar-parâmetros-em-consultas-dinâmicas)
- [13. Uso de Cursores](#13-uso-de-cursores)
- [14. Uso de Variáveis Globais](#14-uso-de-variáveis-globais)
- [15. Não usar Constraints](#15-não-usar-constraints)


## 1-select

- Descrição do anti-padrão: 

Usar `SELECT *` em consultas, o que significa selecionar todas as colunas de uma tabela.

- Por que é considerado um anti-padrão:
    - Pode levar a um consumo desnecessário de recursos, especialmente se a tabela tiver muitas colunas.
    - Pode causar problemas se a estrutura da tabela mudar.

Exemplo de código:

```sql
SELECT 
    * 
FROM usuarios;
```

- Como corrigi-lo:

Especifique as colunas que você realmente precisa em sua consulta.

Exemplo de código corrigido:

```sql
SELECT 
    id
    , nome
    , email 
FROM usuarios;
```

## 2-não-usar-índices 

- Descrição do anti-padrão:

Não usar `índices` em colunas frequentemente consultadas.

- Por que é considerado um anti-padrão:
    - Pode levar a tempos de consulta muito mais longos.
    - Aumenta a carga no servidor de banco de dados.

Exemplo de código:

```sql
SELECT 
    nome 
FROM usuarios 
WHERE email = 'exemplo@email.com';
-- (Considerando que a coluna email não está indexada)
```

- Como corrigi-lo:

Adicione um índice à coluna frequentemente consultada.

Exemplo de código corrigido:

```sql
CREATE INDEX idx_email ON usuarios(email);
```

## 3-uso-excessivo-de-joins

- Descrição do anti-padrão:

Usar um número excessivo de `JOINs` em uma única consulta.

- Por que é considerado um anti-padrão:
    - Pode tornar a consulta extremamente lenta.
    - Torna a consulta difícil de ler e manter.

Exemplo de código:

```sql
SELECT 
    u.nome
    , o.descricao
    , p.nome 
FROM usuarios u 
JOIN pedidos o ON u.id = o.usuario_id 
JOIN produtos p ON o.produto_id = p.id 
JOIN categorias c ON p.categoria_id = c.id 
--(e assim por diante com mais JOINs)
```

- Como corrigi-lo:

Reavalie a necessidade de todos os `JOINs`. Considere dividir a consulta ou usar `subconsultas/CTEs` quando apropriado.

Exemplo de código corrigido:

```sql
WITH UserOrders AS (
    SELECT 
        u.nome
        , o.descricao 
    FROM usuarios u 
    JOIN pedidos o ON u.id = o.usuario_id
)

SELECT 
    uo.nome
    , p.nome 
FROM UserOrders uo 
JOIN produtos p ON uo.produto_id = p.id;
```

## 4-uso-de-funções-em-colunas-em-cláusulas-where

- Descrição do anti-padrão:

Aplicar funções diretamente às colunas dentro da cláusula `WHERE`.

- Por que é considerado um anti-padrão:
    - Impede que os índices da coluna sejam usados, levando a  varreduras completas da tabela.
    - Pode tornar a consulta significativamente mais lenta.

Exemplo de código:

```sql
SELECT 
    * 
FROM usuarios 
WHERE UPPER(sobrenome) = 'SILVA';
```

- Como corrigi-lo:

Evite usar funções nas colunas. Se necessário, considere criar uma coluna adicional ou um índice computado.

Exemplo de código corrigido:

```sql
SELECT 
    * 
FROM usuarios 
WHERE sobrenome = 'Silva';
```

## 5-não-usar-cláusula-limitoffset

- Descrição do anti-padrão:

Retornar todos os registros de uma tabela sem usar a cláusula `LIMIT`, `TOP` ou `OFFSET`.

- Por que é considerado um anti-padrão:
    - Pode retornar um número muito grande de registros, consumindo recursos e tempo.
    - Pode causar problemas de desempenho, especialmente em tabelas grandes.

Exemplo de código:

```sql
SELECT 
    * 
FROM logs;
```

- Como corrigi-lo:

Sempre use `LIMIT`, `TOP` ou `OFFSET` para restringir o número de registros retornados, especialmente se você não tiver certeza do tamanho da tabela.

Exemplo de código corrigido:

```sql
SELECT 
    * 
FROM logs 
LIMIT 100;
```

## 6-uso-de-subconsultas-não-correlacionadas

- Descrição do anti-padrão:

Usar `subconsultas` que são executadas repetidamente, mas que retornam o mesmo resultado.

- Por que é considerado um anti-padrão:
    - Pode causar consultas a serem executadas mais vezes do que o necessário.
    - Pode tornar a consulta muito mais lenta do que precisa ser.

Exemplo de código:

```sql
SELECT 
    nome
    , (SELECT MAX(data_criacao) FROM pedidos) AS ultima_data_pedido 
FROM usuarios;
```

- Como corrigi-lo:

Use uma `subconsulta` correlacionada ou, se a `subconsulta` não estiver relacionada, mova-a para uma `CTE` ou subconsulta no `FROM`.

Exemplo de código corrigido:

```sql
WITH UltimaDataPedido AS (
    SELECT 
        MAX(data_criacao) AS data 
    FROM pedidos
)
SELECT 
    nome
    , udp.data AS ultima_data_pedido 
FROM usuarios, UltimaDataPedido udp;
```

## 7-consultas-com-lógica-complexa

- Descrição do anti-padrão:

Incorporar lógica de programação complexa diretamente nas consultas SQL.

- Por que é considerado um anti-padrão:
    - Torna a consulta difícil de ler e manter.
    - Pode levar a problemas de desempenho.
    - Mistura lógica de negócios com lógica de banco de dados.

Exemplo de código:

```sql
SELECT 
    CASE 
        WHEN (
                SELECT 
                    COUNT(*) 
                FROM pedidos 
                WHERE cliente_id = clientes.id
            ) > 10 THEN 'VIP'
        ELSE 'Regular'
    END AS status_cliente
FROM clientes;
```

- Como corrigi-lo:

Mova a lógica complexa para o código da aplicação ou use `funções/procedures` para encapsular a lógica.

Exemplo de código corrigido:

```sql
CREATE FUNCTION fn_status_cliente(@cliente_id INT) RETURNS VARCHAR(10)

AS

BEGIN
    IF (
        SELECT 
            COUNT(*) 
        FROM pedidos 
        WHERE cliente_id = @cliente_id
        ) > 10
        RETURN 'VIP'
    ELSE
        RETURN 'Regular'
END;

SELECT 
    fn_status_cliente(id) AS status_cliente 
FROM clientes;
```

## 8-não-usar-normalização

- Descrição do anti-padrão:

Armazenar dados em um formato não normalizado, levando a redundâncias.

- Por que é considerado um anti-padrão:
    - Pode levar a inconsistências de dados.
    - Aumenta o tamanho do banco de dados.
    - Torna as atualizações mais complexas.

Exemplo de código:

Uma tabela pedidos que contém colunas para `nome_cliente`, `endereco_cliente`, etc., em vez de uma chave estrangeira para uma tabela clientes.

- Como corrigi-lo:

Use técnicas de normalização para organizar as tabelas e relações de forma eficiente.

Exemplo de código corrigido:

Divida a tabela pedidos em duas tabelas: `clientes` e `pedidos`, e use uma chave estrangeira em pedidos para referenciar clientes.

## 9-hardcoding-de-valores

- Descrição do anti-padrão:

Inserir valores fixos ou `hardcoded` diretamente nas consultas.

- Por que é considerado um anti-padrão:
    - Torna a consulta inflexível.
    - Pode levar a erros se os valores precisarem ser alterados.
    - Dificulta a manutenção.

Exemplo de código:

```sql
SELECT 
    * 
FROM produtos 
WHERE categoria = 'Eletrônicos' AND preco < 100;
```

- Como corrigi-lo:

Use variáveis ou parâmetros em vez de valores hardcoded.

Exemplo de código corrigido:

```sql
DECLARE @categoria NVARCHAR(50) = 'Eletrônicos';
DECLARE @precoMax DECIMAL(10,2) = 100;
SELECT 
    * 
FROM produtos 
WHERE categoria = @categoria AND preco < @precoMax;
```

## 10-tabelas-sem-chave-primária

- Descrição do anti-padrão:

Criar tabelas sem uma chave primária definida.

- Por que é considerado um anti-padrão:
    - Dificulta a identificação única de registros.
    - Pode levar a dados duplicados.
    - Reduz a integridade referencial.

Exemplo de código:

```sql
CREATE TABLE clientes (
    nome VARCHAR(100),
    endereco VARCHAR(255)
);
```

- Como corrigi-lo:

Sempre defina uma chave primária para suas tabelas.

Exemplo de código corrigido:

```sql
CREATE TABLE clientes (
    id INT PRIMARY KEY,
    nome VARCHAR(100),
    endereco VARCHAR(255)
);
```

## 11-uso-de-campos-varchar-para-datas

- Descrição do anti-padrão:

Usar campos `VARCHAR` ou `TEXT` para armazenar datas.

- Por que é considerado um anti-padrão:
    - Dificulta operações relacionadas a datas.
    - Pode levar a inconsistências de dados.
    - Reduz a eficiência das consultas.

Exemplo de código:

```sql
CREATE TABLE eventos (
    nome_evento VARCHAR(100),
    data_evento VARCHAR(10)
);
```

- Como corrigi-lo:

Use o tipo de dado apropriado para datas.

Exemplo de código corrigido:

```sql
CREATE TABLE eventos (
    nome_evento VARCHAR(100),
    data_evento DATE
);
```

## 12-não-usar-parâmetros-em-consultas-dinâmicas

- Descrição do anti-padrão:

Construir consultas SQL dinâmicas concatenando strings sem usar parâmetros.

- Por que é considerado um anti-padrão:
    - Vulnerável a ataques de injeção SQL.
    - Dificulta a leitura e manutenção do código.

Exemplo de código:

```sql
DECLARE @nome_cliente NVARCHAR(50) = 'John';
EXEC('SELECT * FROM clientes WHERE nome = ' + @nome_cliente);
```

- Como corrigi-lo:

Use parâmetros em consultas dinâmicas.

Exemplo de código corrigido:

```sql
DECLARE @nome_cliente NVARCHAR(50) = 'John';
EXEC sp_executesql N'SELECT * FROM clientes WHERE nome = @nome', N'@nome NVARCHAR(50)', @nome = @nome_cliente;
```

## 13-uso-de-cursores

- Descrição do anti-padrão:

Depender excessivamente de cursores para processar linhas individualmente.

- Por que é considerado um anti-padrão:
    - Os cursores são lentos em comparação com operações definidas em conjuntos.
    - Consumo excessivo de recursos do servidor.

Exemplo de código:

```sql
DECLARE cursor_example CURSOR FOR 
SELECT 
    nome 
FROM clientes;
OPEN cursor_example;
-- ... processamento linha a linha ...
CLOSE cursor_example;
DEALLOCATE cursor_example;
```

- Como corrigi-lo:

Sempre que possível, opte por operações definidas em conjuntos.

Exemplo de código corrigido:

```sql
SELECT 
    nome 
FROM clientes 
WHERE condicao = 'valor';
```

## 14-uso-de-variáveis-globais

- Descrição do anti-padrão:

Depender de variáveis globais para armazenar informações temporárias.

- Por que é considerado um anti-padrão:
    - Pode levar a problemas de concorrência.
    - Reduz a modularidade e reutilização do código.

Exemplo de código:

```sql
SET @GlobalVar = 'valor';
```

- Como corrigi-lo:

Use variáveis locais ou parâmetros.

Exemplo de código corrigido:

```sql
DECLARE @LocalVar VARCHAR(50) = 'valor';
```

## 15-não-usar-constraints

- Descrição do anti-padrão:

Criar tabelas sem constraints adequadas, como `FOREIGN KEY`, `CHECK`, etc.

- Por que é considerado um anti-padrão:
    - Pode levar a dados inconsistentes.
    - Reduz a integridade dos dados.

Exemplo de código:

```sql
CREATE TABLE pedidos (
    id INT,
    cliente_id INT,
    valor DECIMAL(10,2)
);
```

- Como corrigi-lo:

Adicione constraints apropriadas para garantir a integridade dos dados.

Exemplo de código corrigido:

```sql
CREATE TABLE pedidos (
    id INT PRIMARY KEY,
    cliente_id INT REFERENCES clientes(id),
    valor DECIMAL(10,2) CHECK (valor > 0)
);
```

Obrigado por ler até aqui! 🙌

Selo:

[<img src="logo.png" width="100" height="100">](https://github.com/Linhares015)