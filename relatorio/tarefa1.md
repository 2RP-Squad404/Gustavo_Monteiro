# 📖 Tarefa 1: Rotina ETL com Airflow e LivyOperator
**Nome do Estagiário:** Gustavo Monteiro Gomes Pires  

Objetivo:
Desenvolver uma rotina ETL que seja capaz de extrair, transformar e carregar dados de um arquivo CSV (purchases_2023.csv). A rotina deverá filtrar os dados por uma data específica dentro de um intervalo de uma hora e incluir apenas as colunas consideradas realmente úteis pelos desenvolvedores. Além disso, as colunas que forem julgadas como incorretas ou inadequadas devem ser tratadas ou descartadas.

Requisitos:

- Extração de Dados:

```python
%livy.pyspark

df = spark.read.csv("s3a://warehouse/purchases_2023.csv", header=True)
df.show()
```

- Implementar filtros:

```python
start_datetime = '2023-09-01 00:00:00'
end_datetime = '2023-09-30 23:59:59'

df_filtrado = df.filter((df['purchase_datetime'] >= start_datetime) & (df['purchase_datetime'] <= end_datetime))

df_filtrado.show()
```

- Seleção de Colunas:

Incluir no output apenas as colunas que forem consideradas úteis pelos desenvolvedores. Justificar as colunas escolhidas.

```python
df.select("product_name", "amount", "price", "discount_applied", "payment_method", "purchase_datetime", "purchase_location").show()
```

- Tratamento de Dados:

Identificar e tratar quaisquer valores incorretos ou inconsistentes nas colunas escolhidas. Explicar as decisões de tratamento aplicadas.

Testes para verificação de dados duplicados ou ausentes:

```python
df.IsNaN().duplicated().sum()
```

Melhor visualização de preço:

```python
df = df.withColumn('price', round(col('price'), 2))
df.show()
```