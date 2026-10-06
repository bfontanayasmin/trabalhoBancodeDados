# Apache Spark e PySpark

## Apache Spark

O Apache Spark é uma ferramenta de processamento de dados que pode
distribuir o trabalho entre várias máquinas. Também pode funcionar
localmente, utilizando os recursos de um único computador.

Neste projeto, utilizamos o Spark 3.5.3 em modo local para executar
comandos SQL sobre tabelas Delta Lake e Apache Iceberg.

## PySpark

O PySpark é a interface Python do Apache Spark. Ele permite controlar
o processamento usando código Python, trabalhar com DataFrames
e executar consultas SQL.

Nos notebooks, utilizamos o PySpark para criar as sessões e executar
os comandos de criação, inserção, atualização, exclusão e consulta.

## SparkSession

SparkSession é o ponto de entrada para trabalhar com o Spark.

Um exemplo básico de inicialização é:

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("Exemplo de processamento")
    .master("local[*]")
    .getOrCreate()
)
```

A opção local[*] permite utilizar os núcleos disponíveis no computador.

Nos notebooks do projeto, acrescentamos as configurações necessárias
para carregar o suporte ao Delta ou ao Iceberg.

## DataFrames e consultas SQL

Um DataFrame organiza dados em linhas e colunas, com tipos definidos.

O método spark.sql executa um comando SQL. Nas consultas SELECT,
ele retorna um DataFrame. O método show exibe seus registros.

Após criar e preencher as tabelas no notebook Delta, podemos consultar:

```python
spark.sql("""
    SELECT id_produto, nome_produto, preco, estoque
    FROM loja_delta.produtos
    ORDER BY id_produto
""").show()
```

No notebook Iceberg, a consulta equivalente utiliza o catálogo local:

```python
spark.sql("""
    SELECT id_produto, nome_produto, preco, estoque
    FROM local.loja_iceberg.produtos
    ORDER BY id_produto
""").show()
```

## Papel das ferramentas no projeto

| Ferramenta | Papel |
|---|---|
| Spark | Processar os dados e executar os comandos. |
| PySpark | Permitir controlar o Spark usando Python. |
| Delta Lake | Organizar as tabelas com arquivos de dados e registro de transações. |
| Apache Iceberg | Organizar as tabelas com arquivos de dados, metadados e snapshots. |

Os exemplos de INSERT, UPDATE e DELETE estão nas páginas de
[Delta Lake](delta.md) e [Apache Iceberg](iceberg.md).

## Referências

- [Apache Spark](https://spark.apache.org/)
- [Guia de DataFrames do PySpark](https://spark.apache.org/docs/latest/api/python/getting_started/quickstart_df.html)
