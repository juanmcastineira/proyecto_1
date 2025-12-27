# Analisis de datos

Se quieren hacer analisis de distintos factores de la tabla

## Analisis de cliente

Se quiere encontrar un perfil de comprador para eso primero hago una view para separar a los clientes por diferentes rangos de edad asi como una agrupacion de localizaciones por region de la siguiente manera  
```sql CREATE VIEW sh_v AS
SELECT * FROM shopping;
ALTER VIEW sh_v AS
SELECT *,
    CASE 
        WHEN Age < 25  THEN '18-24'
        WHEN Age BETWEEN 25 AND 30 THEN '25-30'
        WHEN Age BETWEEN 31 AND 35 THEN '31-35'
        WHEN Age BETWEEN 36 AND 40 THEN '36-40'
        WHEN Age BETWEEN 41 AND 45 THEN '41-45'
        WHEN Age BETWEEN 46 AND 50 THEN '46-50'
        WHEN Age BETWEEN 51 AND 55 THEN '51-55'
        WHEN Age BETWEEN 56 AND 60 THEN '56-60'
        WHEN Age BETWEEN 61 AND 65 THEN '61-65'
        ELSE '65+'  
    END AS Cluster_edad,--- separacion de edad cada 5 años
    CASE 
        WHEN Location IN ('Connecticut', 'Maine', 'Massachusetts', 'New Hampshire', 'New Jersey', 'New York', 'Pennsylvania', 'Rhode Island', 'Vermont') THEN 'Northeast'
        WHEN Location IN ('Illinois', 'Indiana', 'Iowa', 'Kansas', 'Michigan', 'Minnesota', 'Missouri', 'Nebraska', 'North Dakota', 'Ohio', 'South Dakota', 'Wisconsin') THEN 'Midwest'
        WHEN Location IN ('Alabama', 'Arkansas', 'Delaware', 'Florida', 'Georgia', 'Kentucky', 'Louisiana', 'Maryland', 'Mississippi', 'North Carolina', 'Oklahoma', 'South Carolina', 'Tennessee', 'Texas', 'Virginia', 'West Virginia') THEN 'South'
        WHEN Location IN ('Alaska', 'Arizona', 'California', 'Colorado', 'Hawaii', 'Idaho', 'Montana', 'Nevada', 'New Mexico', 'Oregon', 'Utah', 'Washington', 'Wyoming') THEN 'West'
        ELSE 'Other' 
    END AS Region,--- separacion por region
    CAST(CASE WHEN `Subscription Status` = 'Yes' THEN 1 ELSE 0 END AS FLOAT) AS SS
FROM shopping;   
```
Separacion por genero

```sql
SELECT 
    gender,
    COUNT(*) AS cantidad,
    CONCAT(ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM shopping), 2), '%')  AS percent_tot,
---PORCENTAJE DE SUBSCRIPTORES
    CONCAT(ROUND(sum(ss) * 100.0 / (SELECT COUNT(*) FROM shopping), 2), '%')  AS percent_sub
FROM sh_v
GROUP BY gender;
```

| Gender | cantidad | percent_tot | percent_sub |
|--------|----------|-------------|-------------|
| Male   | 2652     | 68.00%      | 27%         |
| Female | 1248     | 32.00%      | 0%          |

Separacion por edad

```sql
    SELECT 
        Cluster_edad,
        COUNT(*) AS cantidad,
        CONCAT(ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM shopping), 2), '%') AS porcentaje
    FROM sh_v
    GROUP BY Cluster_edad
ORDER BY MIN(Age);
```
| Cluster_edad | cantidad | porcentaje |
|---|---|---|
| 18-24 | 486 | 12.46% |
| 25-30 | 463 | 11.87% |
| 31-35 | 364 | 9.33% |
| 36-40 | 361 | 9.26% |
| 41-45 | 368 | 9.44% |
| 46-50 | 382 | 9.79% |
| 51-55 | 371 | 9.51% |
| 56-60 | 382 | 9.79% |
| 61-65 | 368 | 9.44% |
| 65+ | 355 | 9.10% |          

Separacion por genero y edad

```sql
SELECT 
    gender,
    COUNT(*) AS cantidad,
    CONCAT(ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM shopping),2), '%')  AS percent_tot, 
    CONCAT(ROUND(sum(ss) * 100.0 / (SELECT COUNT(*) FROM shopping),2), '%')  AS percent_sub
FROM sh_v
GROUP BY gender;
```
| Gender | Cluster_edad | cantidad | Cant_Sub | Sub_rate |
|---|---|---|---|---|
| Female | 18-24 | 152 | 0 | 0% |
| Female | 25-30 | 143 | 0 | 0% |
| Female | 31-35 | 120 | 0 | 0% |
| Female | 36-40 | 117 | 0 | 0% |
| Female | 41-45 | 126 | 0 | 0% |
| Female | 46-50 | 132 | 0 | 0% |
| Female | 51-55 | 114 | 0 | 0% |
| Female | 56-60 | 116 | 0 | 0% |
| Female | 61-65 | 124 | 0 | 0% |
| Female | 65+ | 104 | 0 | 0% |
| Male | 18-24 | 334 | 125 | 37.43% |
| Male | 25-30 | 320 | 124 | 38.75% |
| Male | 31-35 | 244 | 97 | 39.75% |
| Male | 36-40 | 244 | 95 | 38.93% |
| Male | 41-45 | 242 | 97 | 40.08% |
| Male | 46-50 | 250 | 112 | 44.8% |
| Male | 51-55 | 257 | 117 | 45.53% |
| Male | 56-60 | 266 | 91 | 34.21% |
| Male | 61-65 | 244 | 107 | 43.85% |
| Male | 65+ | 251 | 88 | 35.06% |

```sql
SELECT 
    region, 
    gender,
    COUNT(*) AS cantidad, 
    -- Porcentaje respecto al TOTAL de la tabla
    CONCAT(ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2), '%') AS porcentaje_total,
    -- Porcentaje respecto al total de su GÉNERO 
    CONCAT(ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(PARTITION BY gender), 2), '%') AS porcentaje_por_genero
FROM sh_v
GROUP BY region, gender
ORDER BY gender, cantidad DESC;
```

```sql
WITH cte AS (
    SELECT 
        region, 
        gender, 
        Cluster_edad, 
        COUNT(*) AS cantidad,
        -- Calculamos el porcentaje decimal
        (COUNT(*) * 100.0 / (SELECT COUNT(*) FROM sh_v)) AS porcentaje_decimal
    FROM sh_v
    GROUP BY region, gender, Cluster_edad
)
SELECT 
    region, 
    gender, 
    Cluster_edad, 
    cantidad,
    -- Porcentaje individual formateado
    CONCAT(ROUND(porcentaje_decimal, 2), '%') AS porcentaje,
    -- PORCENTAJE CORRIENTE 
    CONCAT(ROUND(SUM(porcentaje_decimal) OVER(ORDER BY cantidad DESC), 2), '%') AS porcentaje_corriente,
    -- Ranking para control
    RANK() OVER(ORDER BY cantidad DESC) AS rn
FROM cte
ORDER BY cantidad DESC;
```
| region | gender | Cluster_edad | cantidad | porcentaje | porcentaje_corriente | rn |
|---|---|---|---|---|---|---|
| South | Male | 18-24 | 111 | 2.85% | 2.85% | 1 |
| South | Male | 25-30 | 102 | 2.62% | 5.46% | 2 |
| South | Male | 61-65 | 99 | 2.54% | 8.00% | 3 |
| ... | ... | ... | ... | ... | ... | ...|
| Midwest | Male | 46-50 | 61 | 1.56% | 46.64% | 23 |
| West | Male | 41-45 | 58 | 1.49% | 51.10% | 24 |
>[!important]
>Se analiza mas los primeros 24 registros ya que estos corresponden al 51,87% de la cantidad de datos

<details>
<summary>📂tabla completa</summary>
    
| region | gender | Cluster_edad | cantidad | porcentaje | porcentaje_corriente | rn |
|---|---|---|---|---|---|---|
| South | Male | 18-24 | 111 | 2.85% | 2.85% | 1 |
| South | Male | 25-30 | 102 | 2.62% | 5.46% | 2 |
| South | Male | 61-65 | 99 | 2.54% | 8.00% | 3 |
| West | Male | 18-24 | 90 | 2.31% | 10.31% | 4 |
| South | Male | 41-45 | 88 | 2.26% | 12.56% | 5 |
| South | Male | 46-50 | 85 | 2.18% | 16.92% | 6 |
| West | Male | 25-30 | 85 | 2.18% | 16.92% | 7 |
| South | Male | 31-35 | 81 | 2.08% | 19.00% | 8 |
| Midwest | Male | 18-24 | 80 | 2.05% | 25.15% | 9 |
| West | Male | 56-60 | 80 | 2.05% | 25.15% | 10 |
| Midwest | Male | 25-30 | 80 | 2.05% | 25.15% | 11 |
| South | Male | 65+ | 77 | 1.97% | 31.08% | 12 |
| West | Male | 36-40 | 77 | 1.97% | 31.08% | 13 |
| South | Male | 51-55 | 77 | 1.97% | 31.08% | 14 |
| South | Male | 56-60 | 75 | 1.92% | 33.00% | 15 |
| West | Male | 51-55 | 73 | 1.87% | 34.87% | 16 |
| Midwest | Male | 65+ | 72 | 1.85% | 36.72% | 17 |
| West | Male | 31-35 | 69 | 1.77% | 38.49% | 18 |
| Midwest | Male | 36-40 | 66 | 1.69% | 40.18% | 19 |
| South | Male | 36-40 | 65 | 1.67% | 43.51% | 20 |
| Midwest | Male | 56-60 | 65 | 1.67% | 43.51% | 21 |
| West | Male | 46-50 | 61 | 1.56% | 46.64% | 22 |
| Midwest | Male | 46-50 | 61 | 1.56% | 46.64% | 23 |
| West | Male | 41-45 | 58 | 1.49% | 51.10% | 24 |
| Midwest | Male | 51-55 | 58 | 1.49% | 51.10% | 25 |
| Midwest | Male | 61-65 | 58 | 1.49% | 51.10% | 26 |
| Midwest | Male | 31-35 | 55 | 1.41% | 53.92% | 27 |
| South | Female | 18-24 | 55 | 1.41% | 53.92% | 28 |
| Northeast | Male | 18-24 | 53 | 1.36% | 56.64% | 29 |
| Northeast | Male | 25-30 | 53 | 1.36% | 56.64% | 30 |
| Northeast | Male | 65+ | 52 | 1.33% | 57.97% | 31 |
| West | Male | 65+ | 50 | 1.28% | 60.54% | 32 |
| Midwest | Male | 41-45 | 50 | 1.28% | 60.54% | 33 |
| Northeast | Male | 51-55 | 49 | 1.26% | 61.79% | 34 |
| South | Female | 61-65 | 47 | 1.21% | 63.00% | 35 |
| Northeast | Male | 41-45 | 46 | 1.18% | 66.54% | 36 |
| Northeast | Male | 56-60 | 46 | 1.18% | 66.54% | 37 |
| South | Female | 25-30 | 46 | 1.18% | 66.54% | 38 |
| Northeast | Male | 61-65 | 44 | 1.13% | 68.79% | 39 |
| South | Female | 36-40 | 44 | 1.13% | 68.79% | 40 |
| Northeast | Male | 46-50 | 43 | 1.10% | 74.31% | 41 |
| West | Male | 61-65 | 43 | 1.10% | 74.31% | 42 |
| South | Female | 41-45 | 43 | 1.10% | 74.31% | 43 |
| West | Female | 46-50 | 43 | 1.10% | 74.31% | 44 |
| South | Female | 56-60 | 43 | 1.10% | 74.31% | 45 |
| Northeast | Male | 31-35 | 39 | 1.00% | 75.31% | 46 |
| West | Female | 25-30 | 38 | 0.97% | 76.28% | 47 |
| South | Female | 46-50 | 37 | 0.95% | 77.23% | 48 |
| Northeast | Male | 36-40 | 36 | 0.92% | 78.15% | 49 |
| Midwest | Female | 25-30 | 34 | 0.87% | 79.90% | 50 |
| Midwest | Female | 18-24 | 34 | 0.87% | 79.90% | 51 |
| Midwest | Female | 31-35 | 33 | 0.85% | 84.13% | 52 |
| Midwest | Female | 41-45 | 33 | 0.85% | 84.13% | 53 |
| West | Female | 36-40 | 33 | 0.85% | 84.13% | 54 |
| South | Female | 51-55 | 33 | 0.85% | 84.13% | 55 |
| West | Female | 31-35 | 33 | 0.85% | 84.13% | 56 |
| South | Female | 65+ | 32 | 0.82% | 86.59% | 57 |
| Northeast | Female | 18-24 | 32 | 0.82% | 86.59% | 58 |
| West | Female | 51-55 | 32 | 0.82% | 86.59% | 59 |
| West | Female | 41-45 | 31 | 0.79% | 90.56% | 60 |
| West | Female | 65+ | 31 | 0.79% | 90.56% | 61 |
| West | Female | 18-24 | 31 | 0.79% | 90.56% | 62 |
| West | Female | 61-65 | 31 | 0.79% | 90.56% | 63 |
| South | Female | 31-35 | 31 | 0.79% | 90.56% | 64 |
| Midwest | Female | 65+ | 29 | 0.74% | 92.79% | 65 |
| Midwest | Female | 51-55 | 29 | 0.74% | 92.79% | 66 |
| West | Female | 56-60 | 29 | 0.74% | 92.79% | 67 |
| Midwest | Female | 46-50 | 28 | 0.72% | 93.51% | 68 |
| Midwest | Female | 56-60 | 27 | 0.69% | 94.21% | 69 |
| Northeast | Female | 25-30 | 25 | 0.64% | 95.49% | 70 |
| Northeast | Female | 61-65 | 25 | 0.64% | 95.49% | 71 |
| Northeast | Female | 46-50 | 24 | 0.62% | 96.72% | 72 |
| Midwest | Female | 36-40 | 24 | 0.62% | 96.72% | 73 |
| Northeast | Female | 31-35 | 23 | 0.59% | 97.31% | 74 |
| Midwest | Female | 61-65 | 21 | 0.54% | 97.85% | 75 |
| Northeast | Female | 51-55 | 20 | 0.51% | 98.36% | 76 |
| Northeast | Female | 41-45 | 19 | 0.49% | 98.85% | 77 |
| Northeast | Female | 56-60 | 17 | 0.44% | 99.28% | 78 |
| Northeast | Female | 36-40 | 16 | 0.41% | 99.69% | 79 |
| Northeast | Female | 65+ | 12 | 0.31% | 100.00% | 80 |

</details>

| Cluster_edad | Cantidad |
|---|---|
| 18-24 | 3 |
| 25-30 | 3 |
| 46-50 | 3 |
| 56-60 | 3 |
| 36-40 | 3 |
| 41-45 | 2 |
| 31-35 | 2 |
| 65+ | 2 |
| 51-55 | 2 |
| 61-65 | 1 |





| region | Cantidad |
|---|---|
| South | 10 |
| West | 8 |
| Midwest | 6 |

    
### resultados
Se puede ver que dos tercios de los usuarios son hombres pero corresponder al 100% de los subscriptores,por otra parte se puede ver que si tomamos los rangos de edad se puede ver que en el rango de 35-40 y 60-70 hay un significativo aumento en la cantidad de usuarios pero que el nivel de subscriptores no influye en la edad.
Por otra parte si se separa por genero se puede ver que tanto en hombres como mujeres el rango de 35-40 sigue siendo el rango de edad con mayor cantidad de usuarios pero a en al desagregar si se puede ver que hay una variedad en el porcentaje de subscriptores incrementando hasta llegar a un 45,53% en el rango de 50-55.
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
Se puede apreciar lo siguientes:
| Categoría | Brecha Cantidad | Brecha Monto | 
| :--- | :---: | :---: | 
| **Clothing** | 27.5% | 37.9% |
| **Footwear** | 23.4% | 7.0% | 
| **Accessory** | 18.2% | 18.0% |

Teniendo en cuenta eso se puede decir que: 

- Clothing exhibe una marcada disparidad en ingresos, principalmente porque jeans actúa como un outlier. Si se excluye este producto, la brecha se reduce a 11,7% en cantidad y 8,4% en monto, quedando más alineada con el patrón observado en Footwear.  
- En Footwear, la relación entre precio y cantidad vendida es más consistente: los productos de mayor precio tienden a venderse en menor volumen, lo que confirma la relación inversa entre precio y cantidad.  
- En Accessory, en cambio, no se observa variación significativa en los precios dentro de la categoría, lo que explica la menor disparidad entre cantidad e ingresos.  
- Finalmente, en Outerwear, al tratarse de solo dos productos, el análisis comparativo es limitado y ambos ítems muestran comportamientos similares.
-outerwear al ser dos productos solo se puede hacer comparacion directa y son basicamente lo mismo para este analisis

Conclusión general: Existe una relación clara entre el precio del producto y la cantidad vendida: a mayor precio, menor volumen de ventas. Sin embargo, la magnitud de esta relación varía según la categoría, siendo más evidente en Clothing y Footwear, mientras que en Accessory los precios homogéneos reducen la brecha.


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

