# Normalización de los datos
Se tomo una base de datos sobre ventas de productos con este formato de tabla

[image alt](https://github.com/juanmcastineira/proyecto_1/blob/e8dbd224ec7f385bfb81b850faa52d43583a71ee/raw.png)

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

>create table Categories(
>Category_id int identity(1,1) primary key,
>Category varchar(50));
>
>insert into Categories(Category)
>(select distinct[Category]
>from shopping)

Se estableció una relación entre la tabla *Item* y la tabla Category a través del campo Category_ID, permitiendo vincular cada ítem con su categoría correspondiente.

>create table items(
>Item_id int identity(1,1) primary key,
>Item varchar (200),
>Category_id int, FOREIGN KEY (Category_id ) REFERENCES categories(Category_id));
>
>INSERT INTO items (Item, Category_id)
>SELECT DISTINCT 
>    s.Item_Purchased,
>    c.Category_id
>FROM shopping s
>JOIN categories c ON s.Category = c.Category;

Luego se hacen tablas para unificar los distintos IDs asi como una tabla con aquellas columnas que no se atomizaron.
Para eso creo una vista que incluya todos los IDs

>create view shopping_view as
>select Customer_ID,Age,Gender,Item_id,Item_Purchased,Category_id,Category,Purchase_Amount_USD,Location_id, sh.Location, size_id, sh.Size,color_id, sh.Color, season_id,sh.Season,round(Review_Rating,1)as Rating,Subscription_Status,shipping_id,sh.Shipping_Type,Discount_Applied, Promo_Code_Used,Previous_Purchases,payment_id,Payment_Method,Frequency_of_Purchases
>from shopping sh
>join locations l on l.Location = sh.Location
>join season se on se.season= sh.Season
>join shipping shi on shi.shipping= sh.Shipping_Type
>join payment p on p.payment=sh.Payment_Method
>join items i on i.Item = sh.Item_Purchased
>join color c on c.color= sh.Color
>join size s on s.size = sh.Size;

armo las siguientes tablas:
 - **Item_Purchase**
    -Item_purchase_ID
    -Item_ID
    -Color_ID
    -Size_ID

 - **Purchase_Condition**
    -Purchase_Condition_ID
    -Location_ID
    -Season_ID
    -Shipping_ID
    -Payment_ID

- **Customer_Info**
    -Customer_ID
    -age
    -Gender
    -Subscription_Status
    -Previous_Purchaces
    -Frecuency_ID

Por ultimo hago una tabla que abarque los ID de las tablas anteriores

>with ranked as(
>select sh.customer_id,
>    i.itempurchaseid,
>    p.purchaseconditionid,
>    sh.Discount_Applied,
>    sh.PurchaseAmountUSD,
>    sh.Rating,
>        rownumber() over (partition by sh.customerid order by sh.customerID,p.purchasecondition_id) as rn
>    from shopping_view sh
>    join itempurchase i on i.itemid = sh.Itemid and i.colorid= sh.colorid and i.sizeid = sh.size_id
>join purchasecondition p on p.locationid= sh.Locationid and p.seasonid = sh.seasonid and p.shippingid = sh.shippingid and p.paymentid = sh.payment_id
>)
> 
>insertinto purchaseinfo(  customerid,
>  itempurchaseid,
>  purchaseconditionid,
>  discount_applied,
>  purchaseamountusd,
>  review_rating
>)
>SELECT
>  Customer_ID, Item_Purchase_ID, Purchase_Condition_ID, Discount_Applied, Purchase_Amountusd, rating
>FROM Ranked
>WHERE rn = 1;

Aclaro que es necesario hacerla de esta manera ya que si no hacen todas las permutaciones de los IDs.

Queda entonces la siguiente disposicion de tablas normalizadas

[image alt](https://github.com/juanmcastineira/proyecto_1/blob/e8dbd224ec7f385bfb81b850faa52d43)83a71ee/finish.png
