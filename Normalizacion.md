# Normalización de datos de venta
Se partió de una base de datos de ventas con una estructura desnormalizada, donde múltiples atributos se encontraban repetidos en una única tabla.

![Gráfico de Suscripciones](https://raw.githubusercontent.com/juanmcastineira/proyecto_1/main/images/raw.png)

El objetivo del proyecto fue normalizar el dataset mediante un modelo relacional, reduciendo redundancia, mejorando integridad de datos y facilitando futuros análisis.

### Identificacion de entidades 

Se identificaron los atributos con valores únicos que podían ser normalizados en tablas independientes:
  - **Category**
  - **Item**
  - **Color**
  - **Size**
  - **Season**
  - **Payment**
  - **Location**
  - **Shipping**
  - **Frecuency**

Cada una de estas entidades fue convertida en una tabla con un identificador único (ID) para asegurar consistencia y escalabilidad.

## Creacion de tablas de dimensiones
Exceptuando la tabla Item, todas las tablas de dimensiones comparten la misma lógica de creación:

```sql CREATE TABLE Categories (
    Category_id INT IDENTITY(1,1) PRIMARY KEY,
    Category VARCHAR(50)
);

INSERT INTO Categories (Category)
SELECT DISTINCT Category
FROM shopping;
```
### relacion item-category
La tabla Item se relaciona con Category mediante una clave foránea (`Category_id`), permitiendo asociar cada producto a su categoría correspondiente.

```sql CREATE TABLE Items (
    Item_id INT IDENTITY(1,1) PRIMARY KEY,
    Item VARCHAR(200),
    Category_id INT,
    FOREIGN KEY (Category_id) REFERENCES Categories(Category_id)
);

INSERT INTO Items (Item, Category_id)
SELECT DISTINCT 
    s.Item_Purchased,
    c.Category_id
FROM shopping s
JOIN Categories c ON s.Category = c.Category;
```


Para facilitar la construcción de tablas intermedias y la integración de IDs, se creó una vista que centraliza todas las claves foráneas junto con los atributos relevantes.

>Esta vista no forma parte del modelo final, sino que se utiliza como capa intermedia de transformación.

```sql CREATE VIEW shopping_view AS
SELECT 
    sh.Customer_ID,
    sh.Age,
    sh.Gender,
    i.Item_id,
    sh.Item_Purchased,
    c.Category_id,
    sh.Purchase_Amount_USD,
    l.Location_id,
    sh.Location,
    s.Size_id,
    sh.Size,
    co.Color_id,
    sh.Color,
    se.Season_id,
    sh.Season,
    ROUND(sh.Review_Rating, 1) AS Rating,
    sh.Subscription_Status,
    shi.Shipping_id,
    sh.Shipping_Type,
    sh.Discount_Applied,
    sh.Promo_Code_Used,
    sh.Previous_Purchases,
    p.Payment_id,
    sh.Payment_Method,
    sh.Frequency_of_Purchases
FROM shopping sh
JOIN locations l ON l.Location = sh.Location
JOIN season se ON se.Season = sh.Season
JOIN shipping shi ON shi.Shipping = sh.Shipping_Type
JOIN payment p ON p.Payment = sh.Payment_Method
JOIN items i ON i.Item = sh.Item_Purchased
JOIN color co ON co.Color = sh.Color
JOIN size s ON s.Size = sh.Size;
```
### Tablas intermedias

A partir de la vista se construyeron las siguientes tablas:
| Tabla | Atributos / IDs |
| :--- | :--- |
| **Item_Purchase** | Item_purchase_ID, Item_ID, Color_ID, Size_ID |
| **Purchase_Condition** | Purchase_Condition_ID, Location_ID, Season_ID |
| **Customer_Info** | Customer_ID, age, Gender, Subscription_Status |

### Tabla Purchaseinfo
Finalmente, se creó una tabla de hechos que consolida las claves foráneas y métricas principales de cada compra.

Para evitar duplicaciones producto de combinaciones múltiples de IDs, se utilizó `ROW_NUMBER()`.
```sql WITH ranked AS (
    SELECT 
        sh.Customer_ID,
        i.ItemPurchaseID,
        p.PurchaseConditionID,
        sh.Discount_Applied,
        sh.PurchaseAmountUSD,
        sh.Rating,
        ROW_NUMBER() OVER (
            PARTITION BY sh.Customer_ID 
            ORDER BY sh.Customer_ID, p.PurchaseConditionID
        ) AS rn
    FROM shopping_view sh
    JOIN ItemPurchase i 
        ON i.ItemID = sh.Item_id 
       AND i.ColorID = sh.Color_id 
       AND i.SizeID = sh.Size_id
    JOIN PurchaseCondition p 
        ON p.LocationID = sh.Location_id
       AND p.SeasonID = sh.Season_id
       AND p.ShippingID = sh.Shipping_id
       AND p.PaymentID = sh.Payment_id
)

INSERT INTO PurchaseInfo (
    Customer_ID,
    ItemPurchaseID,
    PurchaseConditionID,
    Discount_Applied,
    PurchaseAmountUSD,
    Review_Rating
)
SELECT
    Customer_ID,
    ItemPurchaseID,
    PurchaseConditionID,
    Discount_Applied,
    PurchaseAmountUSD,
    Rating
FROM ranked
WHERE rn = 1;
```

>Este enfoque es necesario para evitar la generación de todas las permutaciones posibles entre los distintos IDs.

### Modelo final normalizado
El resultado es un modelo relacional con separación clara entre dimensiones y tabla de hechos:

![Gráfico de Suscripciones](https://raw.githubusercontent.com/juanmcastineira/proyecto_1/main/images/finish.png)

## Conclusión
La normalización permitió:

- Reducir redundancia.

- Mejorar integridad referencial.

- Facilitar análisis posteriores mediante SQL analítico.

Este modelo sirve como base para análisis exploratorios, segmentación de clientes y evaluación de desempeño de productos.
