# Analisis exploratorio de datos (EDA)

El objetivo de este análisis es **comprender el comportamiento de los clientes, el desempeño de los productos y los factores que influyen en la calificación (rating)** a partir de una base de datos de ventas previamente normalizada.

El análisis se divide en tres ejes principales:

1. Perfil de clientes

2. Análisis de productos

3. Análisis del rating
   
## 1️⃣ Analisis de cliente

### Objetivo

Identificar patrones demográficos y geográficos del comprador, así como su relación con el estado de suscripción.

### Preparación de los datos

Se creó una vista para:

- Agrupar edades en rangos (clusters).

- Agrupar estados en regiones.

- Transformar el estado de suscripción en una variable numérica (`SS`) para facilitar cálculos.  

```sql CREATE VIEW sh_v AS
SELECT *,
    CASE 
        WHEN Age < 25 THEN '18-24'
        WHEN Age BETWEEN 25 AND 30 THEN '25-30'
        WHEN Age BETWEEN 31 AND 35 THEN '31-35'
        WHEN Age BETWEEN 36 AND 40 THEN '36-40'
        WHEN Age BETWEEN 41 AND 45 THEN '41-45'
        WHEN Age BETWEEN 46 AND 50 THEN '46-50'
        WHEN Age BETWEEN 51 AND 55 THEN '51-55'
        WHEN Age BETWEEN 56 AND 60 THEN '56-60'
        WHEN Age BETWEEN 61 AND 65 THEN '61-65'
        ELSE '65+'
    END AS cluster_edad,
    CASE 
        WHEN Location IN ('Connecticut','Maine','Massachusetts','New Hampshire',
                          'New Jersey','New York','Pennsylvania','Rhode Island','Vermont') THEN 'Northeast'
        WHEN Location IN ('Illinois','Indiana','Iowa','Kansas','Michigan','Minnesota',
                          'Missouri','Nebraska','North Dakota','Ohio','South Dakota','Wisconsin') THEN 'Midwest'
        WHEN Location IN ('Alabama','Arkansas','Delaware','Florida','Georgia','Kentucky',
                          'Louisiana','Maryland','Mississippi','North Carolina','Oklahoma',
                          'South Carolina','Tennessee','Texas','Virginia','West Virginia') THEN 'South'
        WHEN Location IN ('Alaska','Arizona','California','Colorado','Hawaii','Idaho',
                          'Montana','Nevada','New Mexico','Oregon','Utah','Washington','Wyoming') THEN 'West'
        ELSE 'Other'
    END AS region,
    CASE WHEN Subscription_Status = 'Yes' THEN 1 ELSE 0 END AS ss
FROM shopping;
```
### distribución por genero

La distribución por rangos etarios es relativamente homogénea, con una ligera mayor concentración entre 18–30 y 46–60 años.

### Insight:
La edad influye en el volumen de usuarios, pero no muestra una relación directa con la tasa de suscripción.

| Gender | cantidad | percent_tot | percent_sub |
|--------|----------|-------------|-------------|
| Male   | 2652     | 68.00%      | 27%         |
| Female | 1248     | 32.00%      | 0%          |


### Insight:
Aunque solo dos tercios de los usuarios son hombres, **el 100% de los suscriptores pertenece a este grupo**, lo que sugiere una fuerte segmentación por género.
Separacion por edad

###     Distribución por edad

La distribución por rangos etarios es relativamente homogénea, con una ligera mayor concentración entre **18–30** y **46–60 años**.

### Insight:
La edad influye en el volumen de usuarios, pero **no muestra una relación directa con la tasa de suscripción**.

### Análisis combinado: región, género y edad

Se utilizó un ranking acumulado para identificar los segmentos con mayor peso relativo.

 ### Hallazgo clave:
Los **primeros 24 segmentos explican aproximadamente el 52% del total de los registros**, lo que indica una **alta concentración de clientes en combinaciones específicas de región, género y edad**.

### Conclusiones del análisis de clientes

- El perfil dominante es **hombre**, principalmente en regiones **South y West**.

- La edad no determina la suscripción, pero **los rangos 46–55 muestran las tasas más altas de suscripción** (hasta ~45%).

- Existen segmentos bien definidos que concentran gran parte del negocio, lo que abre oportunidades de segmentación comercial.

## 2️⃣ Análisis de Productos
### Objetivo

Determinar, por categoría:

- El producto que más ingresos genera.

- El producto más vendido en cantidad.

- El producto con menor desempeño.

### Agregación de ventas por producto

```sql WITH ventas_agrupadas AS (
    SELECT
        Category,
        Item_Purchased,
        SUM(Purchase_Amount_USD) AS total_ingresos,
        COUNT(*) AS total_ventas
    FROM shopping
    GROUP BY Category, Item_Purchased
)
SELECT DISTINCT
    Category,
    FIRST_VALUE(Item_Purchased) OVER (PARTITION BY Category ORDER BY total_ingresos DESC) AS producto_mayor_ingreso,
    FIRST_VALUE(total_ingresos) OVER (PARTITION BY Category ORDER BY total_ingresos DESC) AS ingreso_max,
    FIRST_VALUE(Item_Purchased) OVER (PARTITION BY Category ORDER BY total_ventas DESC) AS producto_mas_vendido,
    FIRST_VALUE(total_ventas) OVER (PARTITION BY Category ORDER BY total_ventas DESC) AS ventas_max
FROM ventas_agrupadas;
```
### Insights por categoría

| Categoría |Insight principal | 
| :--- | :---: |
| **Clothing** | Alta disparidad: **Jeans** actúa como outlier en ingresos |
| **Footwear** | Relacion inversa clara entre el precio y volumen |
| **Accessory** | Precios homogéneos → menor brecha |
| **Outwear** | Solo dos productos → análisis limitado |

### Conclusión general:
Existe una **relación inversa entre precio y volumen de ventas**, más marcada en Clothing y Footwear.



## 3️⃣ Análisis del rating

### Objetivo

Identificar si existen variables asociadas a **ratings altos**, comparando el **Top 10%** contra el promedio general.

```sql WITH segmentacion AS(
SELECT  *, CASE WHEN PERCENT_RANK() OVER( PARTITION BY Review_Rating) >= 0.9 AS 'top' else 'general' END AS segmentacion_cliente)
FROM sh_v
```
Luego aplico el siguiente querry para los parametros de Subscription_Status,Category, Discount_Applied, Payment_Method, Cluster_Edad y Shipping_type para ver si hay desviaciones.

### Segmentación por percentil

```sql SELECT  
    subscription_Status,
    COUNT(CASE WHEN segmentacion_cliente ='top' THEN 1 END)*1.0)/COUNT(*) FILTER ( WHEN segmentacion_cliente = 'top') as percent_top,
    COUNT(CASE WHEN segmentacion_cliente ='general' THEN 1 END)*1.0)/COUNT(*) FILTER ( WHEN segmentacion_cliente = 'general') as percent_gral
FROM segmentacion 
GROUP BY subscription_Status
```
### Comparación por estado de suscripción
| Subscription Status | percent_top | percent_gral |
|---|---|---|
| No | 73.54% | 72.94% |
| Yes | 26.46% | 27.06% |

### Insight

No se observan diferencias significativas entre clientes con y sin suscripción respecto a ratings altos, lo que sugiere que **la suscripción no es un factor determinante del rating**.


## Conclusión General

- El negocio presenta **segmentos de clientes claramente concentrados**.

- El desempeño de productos está fuertemente influenciado por el precio.

- El rating no muestra correlaciones fuertes con variables operativas analizadas.

- El análisis demuestra el uso de EDA, funciones de ventana, CTEs y razonamiento analítico orientado a negocio.
