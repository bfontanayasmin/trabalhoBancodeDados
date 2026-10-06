# Delta Lake

## O que é

Delta Lake é um formato de tabela que acrescenta controle de transações
ao armazenamento de dados.

Os registros ficam em arquivos Parquet. Um registro de transações,
armazenado na pasta _delta_log, controla quais arquivos fazem parte
da versão atual da tabela.

Esse controle permite realizar alterações de forma consistente
e manter o histórico das versões da tabela.

## Uso no projeto

Utilizamos Delta Lake 3.2.0 com PySpark 3.5.3.

O notebook delta_lake.ipynb configura a extensão DeltaSparkSessionExtension
e o catálogo DeltaCatalog. O processamento é executado localmente.

As tabelas são categorias e produtos, agrupadas no namespace loja_delta.
Seus arquivos ficam na pasta data/delta.

## Criação das tabelas — DDL

O notebook prepara a variável pasta_delta com o caminho absoluto
da pasta data/delta. Em seguida, executa:

```python
spark.sql(f"""
    CREATE DATABASE IF NOT EXISTS loja_delta
    LOCATION '{pasta_delta.as_posix()}'
""")

spark.sql(f"""
    CREATE OR REPLACE TABLE loja_delta.categorias (
        id_categoria INT,
        nome_categoria STRING
    )
    USING DELTA
    LOCATION '{(pasta_delta / "categorias").as_posix()}'
""")

spark.sql(f"""
    CREATE OR REPLACE TABLE loja_delta.produtos (
        id_produto INT,
        nome_produto STRING,
        preco DECIMAL(10,2),
        estoque INT,
        id_categoria INT
    )
    USING DELTA
    LOCATION '{(pasta_delta / "produtos").as_posix()}'
""")
```

USING DELTA define o formato das tabelas. LOCATION indica onde seus
arquivos serão armazenados.

CREATE OR REPLACE TABLE permite criar ou substituir a tabela.
Neste exemplo, começamos com tabelas vazias.

## Inserção — INSERT

INSERT INTO adiciona registros. Após criar as tabelas, cadastramos
duas categorias e dez produtos fictícios:

```sql
INSERT INTO loja_delta.categorias
    (id_categoria, nome_categoria)
VALUES
    (1, 'Informática'),
    (2, 'Papelaria');

INSERT INTO loja_delta.produtos
    (id_produto, nome_produto, preco, estoque, id_categoria)
VALUES
    (1, 'Mouse', 49.90, 20, 1),
    (2, 'Teclado', 120.00, 10, 1),
    (3, 'Caderno', 19.90, 30, 2),
    (4, 'Caneta', 3.50, 100, 2),
    (5, 'Headset', 199.90, 12, 1),
    (6, 'Webcam', 149.90, 8, 1),
    (7, 'Pendrive', 39.90, 25, 1),
    (8, 'Lapiseira', 9.90, 40, 2),
    (9, 'Borracha', 2.50, 60, 2),
    (10, 'Marcador', 6.90, 35, 2);
```

A lista de campos indica a posição de cada valor. O campo id_categoria
associa cada produto à sua categoria.

## Atualização — UPDATE

UPDATE modifica registros existentes. SET define os novos valores,
e WHERE seleciona os registros que serão alterados:

```sql
UPDATE loja_delta.produtos
SET preco = 59.90,
    estoque = 15
WHERE id_produto = 1;
```

O Mouse passa a ter preço de R$ 59,90 e estoque de 15 unidades.
Os demais produtos mantêm seus valores.

## Exclusão — DELETE

DELETE FROM remove registros da versão atual da tabela:

```sql
DELETE FROM loja_delta.produtos
WHERE id_produto = 4;
```

A condição seleciona a Caneta. Após a operação, permanecem nove produtos.

Essa exclusão lógica não significa a remoção imediata de todos os
arquivos físicos relacionados às versões anteriores da tabela.

## Consulta dos resultados

No notebook, os comandos SQL são executados usando spark.sql.
As consultas antes e depois permitem conferir cada alteração:

```python
spark.sql("""
    SELECT * FROM loja_delta.produtos
    ORDER BY id_produto
""").show()
```

| Etapa | Resultado esperado |
|---|---|
| Antes do INSERT | Tabelas vazias. |
| Depois do INSERT | Duas categorias e dez produtos. |
| Depois do UPDATE | Dez produtos; Mouse com preço 59.90 e estoque 15. |
| Depois do DELETE | Nove produtos; Caneta removida. |

## Repetição do exemplo

A célula de limpeza do notebook remove o banco loja_delta e os arquivos
em data/delta, incluindo o histórico do exemplo anterior.

Para repetir a demonstração, executamos o notebook inteiro desde
o início, usando Restart Kernel and Run All Cells.

## Referências

- [Introdução ao Delta Lake](https://docs.delta.io/quick-start/)
- [Criação e manipulação de tabelas](https://docs.delta.io/delta-batch/)
- [Atualização e exclusão de registros](https://docs.delta.io/delta-update/)
