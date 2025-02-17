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

```
SELECT 
    pr.party_id, 
    pr.first_name, 
    pr.last_name, 
    cm.info_string AS email, 
    tn.contact_number AS phone, 
    p.created_date AS entry_date
FROM party p
INNER JOIN person pr on p.party_id=pr.party_id
INNER JOIN party_role prl ON prl.party_id = pr.party_id
INNER JOIN party_contact_mech pcm ON pr.party_id = pcm.party_id
INNER JOIN contact_mech cm ON pcm.contact_mech_id = cm.contact_mech_id
INNER JOIN telecom_number tn ON pcm.contact_mech_id = tn.contact_mech_id
WHERE
    (cm.contact_mech_type_id = 'EMAIL_ADDRESS' OR tn.contact_number IS NOT NULL)
    AND prl.role_type_id = 'CUSTOMER'
    AND p.created_date BETWEEN '2023-06-01 00:00:00' AND '2023-06-30 23:59:59';
```
    
**Output:**

![image](https://github.com/user-attachments/assets/f9f8d1b1-2014-43ac-b047-34d440cde592)


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
```
select distinct
	p.product_id,
	p.product_type_id,
	p.internal_name
from inventory_item i
left join product p
on i.product_id=p.product_id;
```

**Output:**

![image](https://github.com/user-attachments/assets/91d3727a-e394-4982-aa85-d3c8aa530934)


**b:**

```
select distinct
	p.product_id,
	p.product_type_id,
	p.internal_name
from inventory_item i
left join product p
on i.product_id=p.product_id
inner join product_type pt
on p.product_type_id=pt.product_type_id
where pt.is_physical='Y';
```

**Output:**

![image](https://github.com/user-attachments/assets/ba64e646-1b95-46f6-bb63-d98431dfef29)


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

```
select distinct
	p.product_id,
	p.internal_name,
	p.product_type_id,
	gi.id_value as NETSUITE_ID
from good_identification gi
left join product p
on gi.product_id=p.product_id
where gi.GOOD_IDENTIFICATION_TYPE_ID='ERP_ID'
and gi.ID_VALUE is null;
```

**Output:**

![image](https://github.com/user-attachments/assets/11c87627-f311-4144-bde2-3d11c2d51d4b)


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

```
SELECT 
    gi.product_id, 
    (CASE WHEN gi.GOOD_IDENTIFICATION_TYPE_ID = 'SHOPIFY_PROD_ID' THEN gi.id_value END) AS shopify_id, 
    (CASE WHEN gi.GOOD_IDENTIFICATION_TYPE_ID = 'HC_GOOD_ID_TYPE' THEN gi.id_value END) AS hotwax_id, 
    (CASE WHEN gi.GOOD_IDENTIFICATION_TYPE_ID = 'ERP_ID' THEN gi.id_value END) AS erp_id
FROM good_identification gi 
    WHERE gi.GOOD_IDENTIFICATION_TYPE_ID IN ('SHOPIFY_PROD_ID', 'HC_GOOD_ID_TYPE', 'ERP_ID')
GROUP BY gi.product_id;
```

**Output:**

![image](https://github.com/user-attachments/assets/83138b6f-78f3-4854-966f-dcdc35947fff)


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

```
SELECT 
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
FROM ORDER_HEADER OH
JOIN ORDER_STATUS OS
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
    F.PRODUCT_STORE_ID;
```

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

```
SELECT
	o.order_id,
    o.grand_total as Total_amount,
    op.payment_method_type_id as payment_method,
    o.external_id as shopify_order_id,
  --o.order_date
FROM order_header o
left join order_payment_preference op
on o.order_id=op.order_id
order by o.order_date DESC
limit 500;
```

**Output:**

![image](https://github.com/user-attachments/assets/df111c19-eab0-43d6-a356-cee7a0e2be3c)


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

```
SELECT 
    o.order_id, 
    o.status_id as order_status, 
    p.status_id as payment_status, 
    s.status_id as shipment_status
FROM order_header o
INNER JOIN order_payment_preference p ON o.order_id = p.order_id
LEFT JOIN shipment s ON o.order_id = s.primary_order_id
WHERE 
    p.status_id <> 'PAYMENT_RECEIVED'
    AND s.status_id <> 'SHIPPED';
```

**Output:**

![image](https://github.com/user-attachments/assets/18dcf1a4-98de-4fc6-914a-46ab34e9e7ab)


-----------------------------------------------------------------------

9 Orders Completed Hourly
Business Problem:
Operations teams may want to see how orders complete across the day to schedule staffing.

Fields to Retrieve:

TOTAL ORDERS
HOUR

**Query 9:**

```
SELECT
	count(o.order_id),
	hour(o.order_date)
from order_header o
where o.order_date between '2024-10-28 00:00:01' and '2024-10-28 23:59:59'
and o.status_id = 'ORDER_COMPLETED'
group by hour(o.order_date);
```

**Output:**

![image](https://github.com/user-attachments/assets/ba76ed67-0f18-41ca-9ef1-e278566e9c8c)


-----------------------------------------------------------------------

10. BOPIS Orders Revenue (Last Year)
Business Problem:
BOPIS (Buy Online, Pickup In Store) is a key retail strategy. Finance wants to know the revenue from BOPIS orders for the previous year.

Fields to Retrieve:

TOTAL ORDERS
TOTAL REVENUE

Query10:
//tbd

-----------------------------------------------------------------------

11. Canceled Orders (Last Month)
Business Problem:
The merchandising team needs to know how many orders were canceled in the previous month and their reasons.

Fields to Retrieve:

TOTAL ORDERS
CANCELATION REASON

**Query 11:**

```
select
	count(o.order_id),
    os.CHANGE_REASON_ENUM_ID
from order_header o
inner join order_status os
on o.order_id=os.order_id
where o.order_date between '2024-12-01 00:00:01' and '2024-12-31 23:59:59'
and o.status_id = 'ORDER_CANCELLED' and CHANGE_REASON_ENUM_ID is not null;
```

**Output:**

![image](https://github.com/user-attachments/assets/7cd8dea3-44e7-4efa-aa5b-51bfcb6ebae2)


-----------------------------------------------------------------------

12. Product Threshold Value
Business Problem The retailer has set a threshild value for products that are sold online, in order to avoid over selling.

Fields to Retrieve:

PRODUCT ID
THRESHOLD

**Query 12:**
//tbd

-----------------------------------------------------------------------


