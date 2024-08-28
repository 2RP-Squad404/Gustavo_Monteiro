# 📖 Tarefa 2: Criação de Script SQL para Análise de Campanhas e Compras
**Nome do Estagiário:** Gustavo Monteiro Gomes Pires  

**Descrição:**

Você possui dois datasets em formato CSV, um contendo informações sobre campanhas (Campaign Dataset) e outro sobre compras (Purchase Dataset). O objetivo desta tarefa é criar um script SQL que integre esses datasets e retorne uma tabela consolidada com as seguintes informações:

1. **client_id**: Identificação do cliente.
2. **total_price**: Total gasto pelo cliente, calculado como `(price * amount * discount_applied)`.
3. **most_purchase_location**: Local mais utilizado pelo cliente para realizar compras (`website`, `app`, `store`).
4. **first_purchase**: Data da primeira compra realizada pelo cliente.
5. **last_purchase**: Data da última compra realizada pelo cliente.
6. **most_campaign**: Campanha mais recebida pelo cliente.
7. **quantity_error**: Quantidade de campanhas que retornaram o status "error" para o cliente.
8. **date_today**: Data atual formatada como `YYYY-MM-DD`.
9. **anomes_today**: Data atual formatada como `MMYYYY` (tipo `int`).

## Extração de Dados:

Primeiramente separei cada coluna e em qual tabela faria a extração dessas informações:

```
client_id - purchases
total_price - purchases
most_purchase_location - purchases
first_purchase - purchases
last_purchase - purchases
most_campaign - campaign_data
quantity_error - campaign_data
date_today - campaign_data
anomes_today - campaign_data
```

Teste de compras de um único cliente (`client_id`):

```sql
SELECT 
    client_id,
    purchase_datetime
FROM 
    purchases
WHERE 
    client_id = '11065ae0-c035-4a62-bcd2-348afb9be318'
```

Total gasto pelo cliente (`total_price`):

```sql
SELECT 
    purchase_id,
    price,
    amount,
    discount_applied,
    ROUND((price * amount) * discount_applied, 2) AS total_price
FROM 
    purchases
```

Lógica para extração da primeira e última compra (`first_purchase` &
`last_purchase`):

```sql
WITH ranked_purchases AS (
    SELECT
        client_id,
        purchase_datetime,
        -- ROW_NUMBER() numera as linhas dentro da participação de cada cliente depois ORDER BY ordena por compra descendente e crescente
        ROW_NUMBER() OVER (PARTITION BY client_id ORDER BY purchase_datetime ASC) AS primeira_compra,
        ROW_NUMBER() OVER (PARTITION BY client_id ORDER BY purchase_datetime DESC) AS ultima_compra
    FROM
        purchases
)
SELECT
    client_id,
    MIN(CASE WHEN primeira_compra = 1 THEN purchase_datetime END) AS first_purchase,
    MIN(CASE WHEN ultima_compra = 1 THEN purchase_datetime END) AS last_purchase
FROM
    ranked_purchases
GROUP BY
    client_id
```

Local mais utilizado pelo cliente para realizar compras (`most_purchase_location`):

```sql
WITH purchase_counts AS (
    SELECT
        client_id,
        purchase_location,
        COUNT(*) AS location_count,
        ROW_NUMBER() OVER (PARTITION BY client_id ORDER BY COUNT(*) DESC) AS rn
        -- Lógica parecida a anterior, ROW_NUMBER() numera as linhas dentro da participação de cada cliente onde ele mais comprou 
        -- COUNT() soma todos os locais repetidos e ordena do maior para o menor (ORDER BY x DESC)
    FROM
        purchases
    GROUP BY
        client_id,
        purchase_location
)
SELECT
    client_id,
    purchase_location AS most_purchase_location
FROM
    purchase_counts
WHERE
    -- rn = 1 mostra apenas a maior soma, ou seja, o local que mais se repetiu
    rn = 1
```

Criada uma nova tabela com apenas as informações úteis, com auxílio de inteligência artificial para ajustar e concatenar todas as consultas:

```sql
CREATE TABLE tarefa2 AS
WITH total_price_calc AS (
    SELECT
        client_id,
        ROUND(SUM(price * amount * discount_applied), 2) AS total_price
    FROM
        purchases
    GROUP BY
        client_id
),
most_location_calc AS (
    SELECT
        client_id,
        purchase_location AS most_purchase_location
    FROM (
        SELECT
            client_id,
            purchase_location,
            COUNT(*) AS location_count,
            ROW_NUMBER() OVER (PARTITION BY client_id ORDER BY COUNT(*) DESC) AS rn
        FROM
            purchases
        GROUP BY
            client_id,
            purchase_location
    ) t
    WHERE
        rn = 1
),
purchase_dates AS (
    SELECT
        client_id,
        MIN(purchase_datetime) AS first_purchase,
        MAX(purchase_datetime) AS last_purchase
    FROM
        purchases
    GROUP BY
        client_id
)
SELECT
    tp.client_id,
    tp.total_price,
    ml.most_purchase_location,
    pd.first_purchase,
    pd.last_purchase
FROM
    total_price_calc tp
JOIN
    most_location_calc ml
ON
    tp.client_id = ml.client_id
JOIN
    purchase_dates pd
ON
    tp.client_id = pd.client_id
```

Utilizando as mesmas lógicas nas consultas anteriores, agora em outro dataset o `campaign_data`:

Campanha mais recebida pelo cliente(`most_campaign`):

```sql
WITH most_campaign AS (
    SELECT
        client_id,
        id_campaign,
        ROW_NUMBER() OVER (PARTITION BY client_id ORDER BY COUNT(*) DESC) AS rn
    FROM
        campaign_data
    GROUP BY
        client_id,
        id_campaign
)
```

Quantidade de campanhas que retornaram o status "error" para o cliente (`quantity_error`):

```sql
quantity_error AS (
    SELECT
        client_id,
        COUNT(*) AS quantity_error
    FROM
        campaign_data
    WHERE
        return_status = 'error'
    GROUP BY
        client_id
)
```

Data atual formatada como `YYYY-MM-DD` e `MMYYYY` (`date_today` & `anomes_today`):

```sql
SELECT
    client_id,
    id_campaign AS most_campaign,
    CURRENT_DATE() AS date_today,
    CAST(CONCAT(LPAD(MONTH(CURRENT_DATE()), 2, '0'), YEAR(CURRENT_DATE())) AS INT) AS anomes_today
    -- Concatena o mês (MM) e o ano (YYYY) atuais em uma string no formato MMYYYY.
    -- Usa LPAD para garantir que o mês tenha sempre 2 dígitos e CAST para converter a string em um número inteiro.
    -- O resultado é uma representação numérica da data atual no formato MMYYYY.
FROM
    campaign_data
```

Resultando na consulta concatenada da segunda tabela:

```sql
CREATE TABLE tarefa2part2 AS
WITH most_campaign AS (
    SELECT
        client_id,
        id_campaign,
        ROW_NUMBER() OVER (PARTITION BY client_id ORDER BY COUNT(*) DESC) AS rn
    FROM
        campaign_data
    GROUP BY
        client_id,
        id_campaign
),
quantity_error AS (
    SELECT
        client_id,
        COUNT(*) AS quantity_error
    FROM
        campaign_data
    WHERE
        return_status = 'error'
    GROUP BY
        client_id
)
SELECT
    mc.client_id,
    mc.id_campaign AS most_campaign,
    COALESCE(qe.quantity_error, 0) AS quantity_error,
    CURRENT_DATE() AS date_today,
    CAST(CONCAT(LPAD(MONTH(CURRENT_DATE()), 2, '0'), YEAR(CURRENT_DATE())) AS INT) AS anomes_today
FROM
    most_campaign mc
LEFT JOIN
    quantity_error qe
ON
    mc.client_id = qe.client_id
WHERE
    mc.rn = 1
```

Unindo as duas tabelas a partir do `client_id`:

```sql
CREATE TABLE tarefa2_final AS
SELECT
    t2.client_id,
    t2.total_price,
    t2.most_purchase_location,
    t2.first_purchase,
    t2.last_purchase,
    t2p2.most_campaign,
    t2p2.quantity_error,
    t2p2.date_today,
    t2p2.anomes_today
FROM
    tarefa2 t2
LEFT JOIN
    tarefa2part2 t2p2
ON
    t2.client_id = t2p2.client_id
```

## Conclusão

Recordando todo passo a passo, certamente a melhorias a serem realizadas, como consultas mais simples, unir a segunda consulta com a tabela `tarefa2` em vez de criar outra para união, ou talvez, utilização de CTE simplificar consultas complexas, dividindo a lógica em partes menores.

Por exemplo:

```sql
CREATE TABLE tarefa2 AS
WITH purchase_data AS (
    SELECT
        client_id,
        ROUND(SUM(price * amount * (1 - discount_applied)), 2) AS total_price, -- Preço total gasto pelo cliente
        purchase_datetime,
        purchase_location
    FROM
        purchases
    GROUP BY
        client_id, purchase_datetime, purchase_location
),
first_last_purchase AS (
    SELECT
        client_id,
        MIN(purchase_datetime) AS first_purchase, -- primeira compra
        MAX(purchase_datetime) AS last_purchase   -- última compra
    FROM
        purchase_data
    GROUP BY
        client_id
),
most_purchase_location AS (
    SELECT
        client_id,
        purchase_location AS most_purchase_location
    FROM (
        SELECT
            client_id,
            purchase_location,
            ROW_NUMBER() OVER (PARTITION BY client_id ORDER BY COUNT(*) DESC) AS rn -- Ordena por contagem decrescente
        FROM
            purchase_data
        GROUP BY
            client_id, purchase_location
    ) ranked
    WHERE rn = 1 -- mostra apenas o local mais frequente
),
campaign_summary AS (
    SELECT
        client_id,
        COUNT(CASE WHEN return_status != 'error' THEN 1 END) AS most_campaign, -- soma tudo menos 'error'
        COUNT(CASE WHEN return_status = 'error' THEN 1 END) AS quantity_error  -- soma apenas 'error'
    FROM
        campaign_data
    GROUP BY
        client_id
)
-- Consulta final
SELECT
    pd.client_id,
    pd.total_price,
    mpl.most_purchase_location,
    flp.first_purchase,
    flp.last_purchase,
    COALESCE(cs.most_campaign, 0) AS most_campaign,
    COALESCE(cs.quantity_error, 0) AS quantity_error,
    CURRENT_DATE() AS date_today,
    CAST(CONCAT(LPAD(MONTH(CURRENT_DATE()), 2, '0'), YEAR(CURRENT_DATE())) AS INT) AS anomes_today
FROM
    purchase_data pd
JOIN
    first_last_purchase flp ON pd.client_id = flp.client_id      --
JOIN                                                             -- soma os select da primeira/ultima compra e local mais usado
    most_purchase_location mpl ON pd.client_id = mpl.client_id   --
LEFT JOIN                                                       
    campaign_summary cs ON pd.client_id = cs.client_id           -- une o client_id(purchases e campaign_data) na parte esquerda da tabela
GROUP BY
    pd.client_id, pd.total_price, mpl.most_purchase_location, flp.first_purchase, flp.last_purchase, cs.most_campaign, cs.quantity_error
```