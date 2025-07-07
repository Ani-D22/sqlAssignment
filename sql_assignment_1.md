**Assignment Link: https://github.com/saastechacademy/foundation/blob/main/udm/intermediate/sql-assignment/sql-assignment-1.md**

---

1. New Customers Acquired in June 2023
Business Problem:
The marketing team ran a campaign in June 2023 and wants to see how many new customers signed up during that period.

Fields to Retrieve:

PARTY_ID
FIRST_NAME
LAST_NAME
EMAIL
PHONE
ENTRY_DATE

**Query 1:**

**Query Cost:** 15,907.31


```
SELECT 
    pr.party_id, pr.first_name, pr.last_name, 
    cm.info_string AS email, tn.contact_number AS phone,
    pr.created_stamp AS entry_date
FROM person pr INNER JOIN party_role prl ON pr.party_id = prl.party_id
INNER JOIN party_contact_mech pcm ON pr.party_id = pcm.party_id
INNER JOIN contact_mech cm ON pcm.contact_mech_id = cm.contact_mech_id
LEFT JOIN telecom_number tn ON pcm.contact_mech_id = tn.contact_mech_id
WHERE (cm.contact_mech_type_id = 'EMAIL_ADDRESS' OR tn.contact_number IS NOT NULL)
    AND prl.role_type_id = 'CUSTOMER'
    AND pr.created_stamp BETWEEN '2023-06-01 00:00:00' AND '2023-06-30 23:59:59';
```
    
**Output:**

![image](https://github.com/user-attachments/assets/102492f4-07a8-4faf-8a98-fec90e41daa7)


-----------------------------------------------------------------------

2. List All Active Physical Products
Business Problem:
Merchandising teams often need a list of all physical products to manage logistics, warehousing, and shipping.

Fields to Retrieve:

PRODUCT_ID
PRODUCT_TYPE_ID
INTERNAL_NAME

**Query 2:**

**a:**

**Query Cost:** 81,900.39


```
select distinct
	p.product_id,
	p.product_type_id,
	p.internal_name
from product p
JOIN PRODUCT_TYPE pt ON p.product_type_id = pt.product_type_id
where pt.is_physical='Y' and p.is_variant ='Y'
and p.sales_discontinuation_date is null;
```

**Output:**

![image](https://github.com/user-attachments/assets/a55def8f-d09c-4007-b91e-05f633dcb11e)



-----------------------------------------------------------------------

3. Products Missing NetSuite ID
Business Problem:
A product cannot sync to NetSuite unless it has a valid NetSuite ID. The OMS needs a list of all products that still need to be created or updated in NetSuite.

Fields to Retrieve:

PRODUCT_ID
INTERNAL_NAME
PRODUCT_TYPE_ID
NETSUITE_ID (or similar field indicating the NetSuite ID; may be NULL or empty if missing)


**Query 3:**

**Query Cost:** 747,058.6


```
select
	p.product_id,
	p.internal_name,
	p.product_type_id,
	gi.id_value as NETSUITE_ID
from product p
left join good_identification gi
on gi.product_id=p.product_id
and gi.GOOD_IDENTIFICATION_TYPE_ID='ERP_ID'
where gi.ID_VALUE = '' or gi.ID_VALUE is null;
```

**Output:**

![image](https://github.com/user-attachments/assets/88ee3295-fc4d-4759-baa8-dff3a50c0297)



-----------------------------------------------------------------------

4. Product IDs Across Systems
Business Problem:
To sync an order or product across multiple systems (e.g., Shopify, HotWax, ERP/NetSuite), the OMS needs to know each system’s unique identifier for that product. This query retrieves the Shopify ID, HotWax ID, and ERP ID (NetSuite ID) for all products.

Fields to Retrieve:

PRODUCT_ID (internal OMS ID)
SHOPIFY_ID
HOTWAX_ID
ERP_ID or NETSUITE_ID (depending on naming)

**Query 4:**

**Query Cost:** 120,632.42


```
SELECT 
    p.PRODUCT_ID as PRODUCT_ID,
    sp.shopify_product_id AS SHOPIFY_ID,
    p.PRODUCT_ID as Hotwax_ID,
    gi.ID_VALUE AS ERP_ID
FROM product p
LEFT JOIN good_identification gi ON p.PRODUCT_ID = gi.PRODUCT_ID
AND gi.GOOD_IDENTIFICATION_TYPE_ID = 'ERP_ID'
JOIN shopify_product sp ON sp.product_id = p.product_id
where gi.id_value is not null;
```

**Output:**

![image](https://github.com/user-attachments/assets/1e3ab939-2c28-419a-af58-a1b5e55e14fd)


----------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------

5. Completed Orders in August 2023
Business Problem:
After running similar reports for a previous month, you now need all completed orders in August 2023 for analysis.

Fields to Retrieve:

PRODUCT_ID
PRODUCT_TYPE_ID
PRODUCT_STORE_ID
TOTAL_QUANTITY
INTERNAL_NAME
FACILITY_ID
EXTERNAL_ID
FACILITY_TYPE_ID
ORDER_HISTORY_ID
ORDER_ID
ORDER_ITEM_SEQ_ID
SHIP_GROUP_SEQ_ID

**Query 5:**

**Query Cost:** 287,040.39


```SELECT 
    OH.ORDER_ID,
    OH.STATUS_ID,
    OH.EXTERNAL_ID,
    OH2.ORDER_HISTORY_ID,
    OI.ORDER_ITEM_SEQ_ID,
    OI.QUANTITY,
    P.PRODUCT_ID,
    P.PRODUCT_TYPE_ID,
    P.INTERNAL_NAME,
    OSG.SHIP_GROUP_SEQ_ID,
    F.FACILITY_ID,
    F.FACILITY_TYPE_ID,
    F.PRODUCT_STORE_ID
FROM ORDER_HEADER OH JOIN ORDER_STATUS OS
	ON OS.STATUS_ID = 'ORDER_COMPLETED'
	AND OS.ORDER_ID = OH.ORDER_ID
	AND OS.STATUS_DATETIME BETWEEN '2023-08-01' AND '2023-09-01'
JOIN ORDER_HISTORY OH2
	ON OH.ORDER_ID = OH2.ORDER_ID
JOIN ORDER_ITEM OI
	ON OI.ORDER_ITEM_SEQ_ID = OH2.ORDER_ITEM_SEQ_ID
	AND OI.ORDER_ID = OH2.ORDER_ID
JOIN PRODUCT P
	ON OI.PRODUCT_ID = P.PRODUCT_ID
JOIN ORDER_ITEM_SHIP_GROUP OSG
	ON OH2.SHIP_GROUP_SEQ_ID = OSG.SHIP_GROUP_SEQ_ID
	AND OH2.ORDER_ID = OSG.ORDER_ID
JOIN FACILITY F
	ON OSG.FACILITY_ID = F.FACILITY_ID
GROUP BY
    OH.ORDER_ID, OH.STATUS_ID, OH.EXTERNAL_ID, OH2.ORDER_HISTORY_ID,
    OI.ORDER_ITEM_SEQ_ID, OI.QUANTITY, P.PRODUCT_ID, P.PRODUCT_TYPE_ID, P.INTERNAL_NAME,
    OSG.SHIP_GROUP_SEQ_ID, F.FACILITY_ID, F.FACILITY_TYPE_ID, F.PRODUCT_STORE_ID;
```

**Output:**

![image](https://github.com/user-attachments/assets/3acbf8c2-6d3c-46c2-b5cc-55ae8cfdf8fb)



----------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------

7. Newly Created Sales Orders and Payment Methods
Business Problem:
Finance teams need to see new orders and their payment methods for reconciliation and fraud checks.

Fields to Retrieve:

ORDER_ID
TOTAL_AMOUNT
PAYMENT_METHOD
Shopify Order ID (if applicable)

**Query 7:**

**Query Cost:** 2,518.98


```
SELECT
	o.order_id,
    o.grand_total as Total_amount,
    op.payment_method_type_id as payment_method,
    o.external_id as shopify_order_id
FROM order_header o
left join order_payment_preference op
on o.order_id=op.order_id
where o.status_id = 'ORDER_CREATED' and o.order_type_id='SALES_ORDER'
order by o.order_date DESC;
```

**Output:**

![image](https://github.com/user-attachments/assets/a5409665-4e0a-4472-953a-0147e045ff6a)



-----------------------------------------------------------------------

8. Payment Captured but Not Shipped
Business Problem:
Finance teams want to ensure revenue is recognized properly. If payment is captured but no shipment has occurred, it warrants further review.

Fields to Retrieve:

ORDER_ID
ORDER_STATUS
PAYMENT_STATUS
SHIPMENT_STATUS

**Query 8:**

**Query Cost:** 19,522.58


```
SELECT 
    o.order_id, 
    o.status_id as order_status, 
    p.status_id as payment_status, 
    s.status_id as shipment_status
FROM order_header o
INNER JOIN order_payment_preference p ON o.order_id = p.order_id
LEFT JOIN shipment s ON o.order_id = s.primary_order_id
WHERE p.status_id NOT IN ('PAYMENT_RECEIVED','PAYMENT_SETTLED')
    AND s.status_id <> 'SHIPMENT_SHIPPED';
```

**Output:**

![image](https://github.com/user-attachments/assets/408be6fc-97e4-45c7-8edc-7686298a7a0f)



-----------------------------------------------------------------------

9 Orders Completed Hourly
Business Problem:
Operations teams may want to see how orders complete across the day to schedule staffing.

Fields to Retrieve:

TOTAL ORDERS
HOUR

**Query 9:**

**Query Cost:** 4,967.15


```
SELECT
	count(o.order_id) as total_order,
	hour(o.STATUS_DATETIME) as hour
from order_status o
where o.STATUS_DATETIME  between '2024-10-28 00:00:01' and '2024-10-28 23:59:59'
and o.status_id = 'ORDER_COMPLETED'
group by hour order by hour;
```

**Output:**

![image](https://github.com/user-attachments/assets/b4f944d8-8056-4887-bad3-de174eca8604)



-----------------------------------------------------------------------

10. BOPIS Orders Revenue (Last Year)
Business Problem:
BOPIS (Buy Online, Pickup In Store) is a key retail strategy. Finance wants to know the revenue from BOPIS orders for the previous year.

Fields to Retrieve:

TOTAL ORDERS
TOTAL REVENUE

Query10:

**Query Cost:** 1,620.09


```
SELECT
    COUNT(o.order_id) as total_orders, 
    SUM(o.grand_Total) as total_revenue
from order_header o
join order_item_ship_group oisg
on oisg.order_id = o.order_id
where oisg.shipment_method_type_id = 'STOREPICKUP' 
and o.order_date BETWEEN '2024-01-01' AND '2025-01-01';
```

**Output:**

![image](https://github.com/user-attachments/assets/74a5e56e-0b04-42c3-b54e-3a0a79d0837d)



-----------------------------------------------------------------------

11. Canceled Orders (Last Month)
Business Problem:
The merchandising team needs to know how many orders were canceled in the previous month and their reasons.

Fields to Retrieve:

TOTAL ORDERS
CANCELATION REASON

**Query 11:**

**Query Cost:** 93.92


```
select
	count(o.order_id) as total_orders,
    os.CHANGE_REASON as cancellation_reason
from order_header o
inner join order_status os
on o.order_id=os.order_id
where o.order_date between '2024-12-01 00:00:01' and '2024-12-31 23:59:59'
and o.status_id = 'ORDER_CANCELLED' and CHANGE_REASON is not null
group by os.CHANGE_REASON;
```

**Output:**

![image](https://github.com/user-attachments/assets/cc152403-4be8-47c4-aa33-4888bc6878be)



-----------------------------------------------------------------------

12. Product Threshold Value
Business Problem The retailer has set a threshild value for products that are sold online, in order to avoid over selling.

Fields to Retrieve:

PRODUCT ID
THRESHOLD

**Query 12:**

**Query Cost:** 60,275.4

```
SELECT 
    pf.PRODUCT_ID, 
    pf.MINIMUM_STOCK AS THRESHOLD
FROM PRODUCT_FACILITY pf
JOIN FACILITY f
ON f.FACILITY_ID = pf.FACILITY_ID
where f.FACILITY_TYPE_ID = 'CONFIGURATION'
AND pf.MINIMUM_STOCK > 0 OR NOT NULL
ORDER BY THRESHOLD DESC;
```

**Output:**

![image](https://github.com/user-attachments/assets/2d8ff762-b138-4167-8f4c-552adc1cf80a)



-----------------------------------------------------------------------
