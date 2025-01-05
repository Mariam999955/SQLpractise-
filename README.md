# SQLpractise-
this will help you with making CTE ,view and how to find answers in sql 
SELECT product.productid , product.NAME , AVG(productreview.rating) as avg_rating
,COUNT(productreview.rating) as review_count
FROM product
join productreview 
    on product.productid = productreview.productid 
group by product.productid ,product.name ;


SELECT dc.productmodelid , d.description
from productmodelproductdescriptionculture  as dc
join productdescription as d
     on dc.productdescriptionID = d.productdescriptionID 
where dc.cultureID  like '%en%'
GROUP by dc.productmodelid
ORDER by dc.productmodelid ASC ;


CREATE VIEW english_description AS 
SELECT 
    dc.productmodelid, 
    d.description
FROM productmodelproductdescriptionculture AS dc
JOIN productdescription AS d
    ON dc.productdescriptionID = d.productdescriptionID
where dc.cultureID  like '%en%'
GROUP BY dc.productmodelid, d.description;


SELECT 
    english_description.productmodelid, 
    english_description.description,
    product.NAME, 
   COUNT(s.orderqty) AS total_orders
FROM english_description
JOIN product
    ON english_description.productmodelid = product.productmodelid
JOIN salesorderdetail AS s
    ON product.productid = s.productid
GROUP BY english_description.productmodelid;


Select productid , sum(orderqty)  as quantity
from salesorderdetail 
group by productid ;



SELECT p.productid ,pc.name as category ,psc.name as subcategory ,p.listprice 
from product p 
join productsubcategory psc
     on p.productsubcategoryid =psc.productsubcategoryid 
join productcategory pc
     on psc.productcategoryid =pc.productcategoryid 
GROUP by productid ;
     
WITH CTE1 AS (
    SELECT 
        productid, 
        SUM(orderqty) AS quantity
    FROM salesorderdetail
    GROUP BY productid
),
CTE2 AS (
    SELECT 
        p.productid, 
        pc.name AS category, 
        psc.name AS subcategory, 
        p.listprice
    FROM product AS p
    JOIN productsubcategory AS psc
        ON p.productsubcategoryid = psc.productsubcategoryid
    JOIN productcategory AS pc
        ON psc.productcategoryid = pc.productcategoryid
)
SELECT 
    CTE2.category AS category,
    CTE2.subcategory as subcategory,
    AVG(CTE2.listprice) as average_price_in_subcategory,
    SUM(CTE1.quantity)as total_items_sold_in_subcategory
FROM CTE1
JOIN CTE2
    ON CTE1.productid = CTE2.productid
group by CTE1.productid
order by CTE2.category ASC , CTE2.subcategory ASC;


SELECT DISTINCT businessentityid ,salesytd 
from salesperson 
group by businessentityid 
order by salesytd DESC 
LIMIT 5;

SELECT 
    salespersonid, 
    SUM(subtotal) AS total_sales
FROM salesorderheader
WHERE salespersonid IS NOT NULL
  AND TRIM(COALESCE(salespersonid, '')) != ''
  AND orderdate >= '2014-01-01' 
  AND orderdate < '2015-01-01'
GROUP BY salespersonid
ORDER BY total_sales DESC 
LIMIT 5;


SELECT salesorderid , SUM((unitprice * (1 - unitpricediscount)) * orderqty) as ordertotal
from salesorderdetail 
group by salesorderid 
limit 10;


WITH CTE1 AS (
    SELECT 
        salesorderid, 
        SUM((unitprice * (1 - unitpricediscount)) * orderqty) AS ordertotal
    FROM salesorderdetail
    GROUP BY salesorderid
),
CTE2 AS (
    SELECT 
        salesorderid, 
        salespersonid
    FROM salesorderheader
    WHERE salespersonid IS NOT NULL
      AND TRIM(COALESCE(salespersonid, '')) != ''
      AND orderdate >= '2014-01-01' 
      AND orderdate < '2015-01-01'
)
, CTE3 AS (SELECT 
    CTE2.salespersonid, 
    SUM(CTE1.ordertotal) AS ordertotalsum
FROM CTE1
JOIN CTE2 
    ON CTE1.salesorderid = CTE2.salesorderid
GROUP BY CTE2.salespersonid
ORDER BY ordertotalsum DESC  )

SELECT CTE3.salespersonid ,CTE3.ordertotalsum,s.commissionpct

FROM CTE3
    Join salesperson s 
         ON CTE3.salespersonid = s.businessentityid 
GROUP BY CTE3.salespersonid
ORDER BY CTE3.salespersonid ASC ;


SELECT 
    s.salespersonid,
    s.salesorderid,
    CASE 
        WHEN c.currencyrateid IS NULL THEN 'None'
    ELSE c.currencyrateid END as currencyrateid ,
    CASE 
        WHEN c.tocurrencycode IS NULL THEN 'None'
    ELSE c.tocurrencycode END AS tocurrencycode
FROM salesorderheader s
LEFT JOIN currencyrate c 
    ON s.currencyrateid = c.currencyrateid
WHERE s.salespersonid IS NOT NULL
  AND orderdate >= '2014-01-01' 
  AND orderdate < '2015-01-01'
GROUP BY 
    s.salespersonid, 
    s.salesorderid, 
    c.currencyrateid, 
    c.tocurrencycode
ORDER BY s.salespersonid ASC;


SELECT 
    s.salespersonid,
    s.salesorderid,
    CASE 
        WHEN c.tocurrencycode IS NULL THEN 'USD'
    ELSE c.tocurrencycode END AS tocurrencycode
FROM salesorderheader s
LEFT JOIN currencyrate c 
    ON s.currencyrateid = c.currencyrateid
WHERE s.salespersonid IS NOT NULL
  AND orderdate >= '2014-01-01' 
  AND orderdate < '2015-01-01'
GROUP BY 
    s.salesorderid 
ORDER BY s.salespersonid ASC
LIMIT 10;
WITH CTE1 AS (
    SELECT 
        salesorderid, 
        SUM((unitprice * (1 - unitpricediscount)) * orderqty) AS ordertotal
    FROM salesorderdetail
    GROUP BY salesorderid
),
CTE2 AS (
    SELECT 
        salesorderid, 
        salespersonid,
        currencyrateid 
    FROM salesorderheader
    WHERE salespersonid IS NOT NULL
      AND orderdate >= '2014-01-01' 
      AND orderdate < '2015-01-01'
),
CTE3 AS (
    SELECT 
        CTE2.salespersonid, 
        SUM(CTE1.ordertotal) AS ordertotalsum,
        CTE2.currencyrateid 
    FROM CTE1
    JOIN CTE2 
        ON CTE1.salesorderid = CTE2.salesorderid
    GROUP BY CTE2.salespersonid
)

SELECT 
    CTE3.salespersonid,
    CASE 
        WHEN c.tocurrencycode IS NULL THEN 'USD'
        ELSE c.tocurrencycode 
    END AS tocurrencycode ,
    CTE3.ordertotalsum,
    s.commissionpct
FROM CTE3
JOIN currencyrate c 
    ON CTE3.currencyrateid = c.currencyrateid
JOIN salesperson s 
    ON CTE3.salespersonid = s.businessentityid
GROUP BY CTE3.Salespersonid 
ORDER BY c.tocurrencycode ASC , 
CTE3.ordertotalsum DESC;

