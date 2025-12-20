# Normalización de los datos
Se tomo una base de datos sobre ventas de productos con este formato de tabla

![Gráfico de Suscripciones](https://raw.githubusercontent.com/juanmcastineira/proyecto_1/main/images/raw.png)

se procedio a hacer las tablas de distintas columnas para ponerle un ID a cada atributo unico.
Las colmunas que se tomaron son:
  - **Category**
  - **Item**
  - **Color**
  - **Size**
  - **Season**
  - **Payment**
  - **Location**
  - **Shipping**
  - **Frecuency**

Estas tablas (salvo la tabla de Item) comparten el siguiente querry:

```sql CREATE TABLE Categories(
Category_id INT IDENTITTY(1,1) PRIMARY KEY,
Category VARCHAR(50));

INSERT INTO Categories(Category)
(SELECT DISTINCT[Category]
FROM shopping)
```
Se estableció una relación entre la tabla *Item* y la tabla Category a través del campo Category_ID, permitiendo vincular cada ítem con su categoría correspondiente.

```sql CREATE TABLE items(
Item_id  INT IDENTITY(1,1) PRIMARY KEY,
Item VARCHAR(200),
Category_id INT, FOREIGN KEY (Category_id ) REFERENCES categories(Category_id));

INSERT INTO items (Item, Category_id)
SELECT DISTINCT 
    s.Item_Purchased,
    c.Category_id
FROM shopping s
JOIN categories c ON s.Category = c.Category;
```
Luego se hacen tablas para unificar los distintos IDs asi como una tabla con aquellas columnas que no se atomizaron.
Para eso creo una vista que incluya todos los IDs

```sql CREATE VIEW shopping_view AS
SELECT Customer_ID,Age,Gender,Item_id,Item_Purchased,Category_id,
  Category,Purchase_Amount_USD,Location_id, sh.Location, size_id,
  sh.Size,color_id, sh.Color, season_id,sh.Season,
  ROUND(Review_Rating,1)AS Rating,Subscription_Status,shipping_id,sh.Shipping_Type,Discount_Applied,
  Promo_Code_Used,Previous_Purchases,payment_id,Payment_Method,Frequency_of_Purchases
 FROM shopping sh
JOIN locations l ON l.Location = sh.Location
JOIN season se ON se.season= sh.Season
JOIN shipping shi ON shi.shipping= sh.Shipping_Type
JOIN payment p ON p.payment=sh.Payment_Method
JOIN items i ON i.Item = sh.Item_Purchased
JOIN color c ON c.color= sh.Color
JOIN size s ON s.size = sh.Size;
```
armo las siguientes tablas:
| Tabla | Atributos / IDs |
| :--- | :--- |
| **Item_Purchase** | Item_purchase_ID, Item_ID, Color_ID, Size_ID |
| **Purchase_Condition** | Purchase_Condition_ID, Location_ID, Season_ID |
| **Customer_Info** | Customer_ID, age, Gender, Subscription_Status |


Por ultimo hago una tabla que abarque los ID de las tablas anteriores

```sql WITH ranked AS(
SELECT sh.customer_id,
    i.itempurchaseid,
    p.purchaseconditionid,
    sh.Discount_Applied,
    sh.PurchaseAmountUSD,
    sh.Rating,
    ROW_NUMBER() OVER (PARTITION by sh.customerid order by sh.customerID,p.purchasecondition_id) AS rn
FROM shopping_view sh
  JOIN itempurchase i on i.itemid = sh.Itemid AND i.colorid= sh.colorid AND i.sizeid = sh.size_id
  JOIN purchasecondition p on p.locationid= sh.Locationid and p.seasonid = sh.seasonid and p.shippingid = sh.shippingid and p.paymentid = sh.payment_id
)
 
INSERT INTO purchaseinfo(customerid,
  itempurchaseid,
 purchaseconditionid,
  discount_applied,
  purchaseamountusd,
  review_rating
)
SELECT
  Customer_ID, Item_Purchase_ID, Purchase_Condition_ID, Discount_Applied, Purchase_Amountusd, rating
FROM Ranked
WHERE rn = 1;
```

Aclaro que es necesario hacerla de esta manera ya que si no hacen todas las permutaciones de los IDs.

Queda entonces la siguiente disposicion de tablas normalizadas

![Gráfico de Suscripciones](https://raw.githubusercontent.com/juanmcastineira/proyecto_1/main/images/finish.png)
