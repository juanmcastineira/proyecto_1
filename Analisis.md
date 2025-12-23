# Analisis de datos

Se quieren hacer analisis de distintos factores de la tabla

## Analisis de cliente

Se quiere encontrar un perfil de comprador para eso primero hago una view para separar a los clientes en clusteers de edad de la siguiente manera  
```sql CREATE VIEW sh_v AS
SELECT * FROM shopping;
ALTER VIEW sh_v AS
SELECT
    sh.*,
    CASE 
        WHEN sh.age <30  THEN '18-30'
        WHEN Age BETWEEN 30 AND 40 THEN '30-40'
        WHEN Age BETWEEN 40 AND 50 THEN '40-50'
        WHEN Age BETWEEN 50 AND 60 THEN '50-60'
        ELSE '60-70' END AS Cluster_edad,
   CAST( CASE WHEN subscription_Status = 'yes' THEN 1 ELSE 0 END) AS FLOAT) AS SS
   FROM shopping.shopping sh;
```
Luego veo cuantos clientes hay por las distintas permutaciones asi como la cantidad de subscripto y el porcentaje de los mismos 

```sql SELECT
Gender, Cluster_edad,COUNT(*) AS cantidad, SUM(ss) AS Sub_cant, ROUND(SUM(SS)/COUNT(*),2) AS Sub_rate
FROM sh_v
GROUP BY `Gender`,`Cluster_edad`
ORDER BY `Gender`, `Cluster_edad`
```
dandome los siguientes resultados 

![Gráfico de Suscripciones](https://raw.githubusercontent.com/juanmcastineira/proyecto_1/main/images/SUBS.png)
### resultados
Se puede apreciar que el perfil de consumidor es en su mayoria hombres  y que estos son los unicos que se subscriben.
Se  puede apreciar tambien que el cluster con mayor cantidad de compradores son los hombres entre 18 a 30 pero que el porcentaje de gente subscripta no lo refleja ya que es bastante homogenos en todos los clusters de edad

## Analisis de productos

Quiero saber que por cada categoria cual es el producto que mas ganancias dio y cual es el que mas vendio asi como la que menos se vendio

mediante:

```sql WITH VentasAgrupadas AS (
    SELECT >
        `Category`, 
        `Item Purchased`, 
        SUM(`Purchase Amount (USD)`) AS TotalDinero,
        COUNT(*) AS TotalCantidad -- Usamos COUNT para la cantidad de ventas
    FROM shopping
    GROUP BY `Category`, `Item Purchased`
)
SELECT DISTINCT
    `Category`,
    -- 1. PRODUCTO QUE GENERÓ MÁS INGRESOS
    FIRST_VALUE(`Item Purchased`) OVER(
        PARTITION BY `Category` 
        ORDER BY TotalDinero DESC
    ) AS prod_max_ingresos,
    
    FIRST_VALUE(TotalDinero) OVER(
        PARTITION BY Category` 
        ORDER BY TotalDinero DESC
    ) AS monto_max,

    -- 2. PRODUCTO MÁS VENDIDO (CANTIDAD)
    FIRST_VALUE(Item Purchased) OVER(
        PARTITION BY Category 
        ORDER BY TotalCantidad DESC
    ) AS prod_mas_frecuente,

    FIRST_VALUE(TotalCantidad) OVER(
        PARTITION BY Category 
        ORDER BY TotalCantidad DESC
    ) AS cantidad_max
FROM VentasAgrupadas;
-- PARA LOS MENOS VENDIDOS CAMBIO DESC POR ASC 
```

dando

![Gráfico de Suscripciones](https://raw.githubusercontent.com/juanmcastineira/proyecto_1/main/images/categorias.png)
![Gráfico de Suscripciones](https://raw.githubusercontent.com/juanmcastineira/proyecto_1/main/images/min.png)
### resultados
Se puede apreciar lo siguiente:
-Accesory: 81,8||18 
-Clothing: 72,5||37,9
-Footwear: 76,6|| 7
-outerwear al ser dos productos solo se puede hacer comparacion directa y son basicamente lo mismo para este analisis
-conclucion general:
## Analisis del rating

El objetivo es identificar variables determinantes en el Rating. Para ello, se segmentó a los clientes en el percentil superior (Top 10%) y se contrastó la distribución de sus atributos frente al promedio poblacional. El objetivo es identificar desviaciones significativas que sugieran una correlación entre ciertos factores y un Rating elevado.

```sql WITH segmentacion AS(
SELECT  *, CASE WHEN PERCENT_RANK() OVER( PARTITION BY Review_Rating) >= 0.9 AS 'top' else 'general' END AS segmentacion_cliente)
FROM sh_v
```
Luego aplico el siguiente querry para los parametros de Subscription_Status,Category, Discount_Applied, Payment_Method, Cluster_Edad y Shipping_type para ver si hay desviaciones.

```sql SELECT 
subscription_Status, COUNT(CASE WHEN segmentacion_cliente ='top' THEN 1 END)*1.0)/COUNT(*) FILTER ( WHEN segmentacion_cliente = 'top') as percent_top, COUNT(CASE WHEN segmentacion_cliente ='general' THEN 1 END)*1.0)/COUNT(*) FILTER ( WHEN segmentacion_cliente = 'general') as percent_gral
FROM segmentacion
GROUP BY subscription_Status
```
Los resultados son los siguientes

![Gráfico de Suscripciones](https://raw.githubusercontent.com/juanmcastineira/proyecto_1/main/images/subsctription.png)

### resultados

