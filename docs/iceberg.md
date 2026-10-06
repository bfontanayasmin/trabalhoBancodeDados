# Apache Iceberg

## O que é

Apache Iceberg é um formato aberto de tabela para organizar conjuntos
de dados e controlar suas alterações.

Ele utiliza metadados para descrever a estrutura da tabela e os
arquivos que a compõem. Os snapshots representam estados da tabela
ao longo do tempo.

As alterações são publicadas de forma atômica: os leitores passam
a enxergar o novo estado após a conclusão da operação.

## Uso no projeto

Utilizamos Apache Iceberg 1.6.1 com PySpark 3.5.3.

O notebook iceberg.ipynb configura a extensão
IcebergSparkSessionExtensions e um catálogo chamado local,
implementado por SparkCatalog e configurado com o tipo hadoop.

A propriedade warehouse aponta para a pasta data/iceberg,
onde ficam os arquivos das tabelas.

## Catálogo e namespace

As tabelas são acessadas por nomes com três partes:

```text
local.loja_iceberg.produtos
```

| Parte | Significado |
|---|---|
| local | Catálogo configurado na sessão Spark. |
| loja_iceberg | Namespace que agrupa as tabelas do exemplo. |
| produtos | Nome da tabela. |

## Criação das tabelas — DDL

```sql
CREATE NAMESPACE IF NOT EXISTS local.loja_iceberg;

CREATE TABLE IF NOT EXISTS local.loja_iceberg.categorias (
    id_categoria INT,
    nome_categoria STRING
)
USING ICEBERG
TBLPROPERTIES ('format-version' = '2');

CREATE TABLE IF NOT EXISTS local.loja_iceberg.produtos (
    id_produto INT,
    nome_produto STRING,
    preco DECIMAL(10,2),
    estoque INT,
    id_categoria INT
)
USING ICEBERG
TBLPROPERTIES ('format-version' = '2');
```

USING ICEBERG seleciona o formato da tabela.

A propriedade format-version = 2 indica a versão do formato
de armazenamento. Ela é diferente da versão 1.6.1 da biblioteca
Iceberg utilizada no projeto.

## Inserção — INSERT

INSERT INTO adiciona registros às tabelas. Utilizamos os mesmos dados
fictícios do notebook Delta:

```sql
INSERT INTO local.loja_iceberg.categorias
    (id_categoria, nome_categoria)
VALUES
    (1, 'Informática'),
    (2, 'Papelaria');

INSERT INTO local.loja_iceberg.produtos
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

Após a inserção, existem duas categorias e dez produtos.

## Atualização — UPDATE

UPDATE modifica registros existentes. Neste exemplo, alteramos
o preço e o estoque do Mouse:

```sql
UPDATE local.loja_iceberg.produtos
SET preco = 59.90,
    estoque = 15
WHERE id_produto = 1;
```

SET define os novos valores. WHERE limita a alteração ao produto
identificado pelo número 1.

O Mouse passa a ter preço de R$ 59,90 e estoque de 15 unidades.
Os demais produtos mantêm seus valores.

## Exclusão — DELETE

DELETE FROM remove registros do estado atual da tabela:

```sql
DELETE FROM local.loja_iceberg.produtos
WHERE id_produto = 4;
```

A condição seleciona a Caneta. Após a exclusão, permanecem nove produtos.

As versões anteriores podem continuar referenciando arquivos antigos.
A exclusão de um registro não implica a remoção imediata de todo
o histórico físico da tabela.

## Consulta dos resultados

No notebook, usamos spark.sql para executar os comandos e show
para apresentar os resultados das consultas:

```python
spark.sql("""
    SELECT * FROM local.loja_iceberg.produtos
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

A célula de limpeza executa DROP TABLE IF EXISTS com a opção PURGE
para remover as tabelas e seu conteúdo.

Em seguida, remove e recria a pasta data/iceberg, preparando o ambiente
para executar novamente as células de criação e manipulação.

Para repetir a demonstração, usamos Restart Kernel and Run All Cells
no notebook.

## Referências

- [Configuração do Iceberg com Spark](https://iceberg.apache.org/docs/latest/spark-configuration/)
- [Criação e exclusão de tabelas](https://iceberg.apache.org/docs/latest/spark-ddl/)
- [Inserção, atualização e exclusão de registros](https://iceberg.apache.org/docs/latest/spark-writes/)
- [Snapshots e confiabilidade](https://iceberg.apache.org/docs/latest/reliability/)
