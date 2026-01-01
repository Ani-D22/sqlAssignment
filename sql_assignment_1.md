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

**Query Cost:** 16,276.42


```
SELECT 
    PR.PARTY_ID,
	PR.FIRST_NAME,
	PR.LAST_NAME, 
    CM.INFO_STRING AS EMAIL,
	TN.CONTACT_NUMBER AS PHONE,
    PR.CREATED_STAMP AS ENTRY_DATE
FROM PERSON PR
	JOIN PARTY_ROLE PRL
		ON PR.PARTY_ID = PRL.PARTY_ID
	JOIN PARTY_CONTACT_MECH PCM
		ON PR.PARTY_ID = PCM.PARTY_ID
		AND PRL.ROLE_TYPE_ID = 'CUSTOMER'
	JOIN CONTACT_MECH CM
		ON PCM.CONTACT_MECH_ID = CM.CONTACT_MECH_ID
	LEFT JOIN TELECOM_NUMBER TN
		ON PCM.CONTACT_MECH_ID = TN.CONTACT_MECH_ID
WHERE (CM.CONTACT_MECH_TYPE_ID = 'EMAIL_ADDRESS' OR TN.CONTACT_NUMBER IS NOT NULL)
    AND PR.CREATED_STAMP BETWEEN '2023-06-01 00:00:00' AND '2023-06-31 23:59:59';
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

**Query Cost:** 82,090.04


```
SELECT P.PRODUCT_ID,
	P.PRODUCT_TYPE_ID,
	P.INTERNAL_NAME
FROM PRODUCT P
	JOIN PRODUCT_TYPE PT
		ON P.PRODUCT_TYPE_ID = PT.PRODUCT_TYPE_ID
WHERE PT.IS_PHYSICAL='Y'
	AND P.IS_VARIANT ='Y'
	AND P.SALES_DISCONTINUATION_DATE IS NULL;
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

**Query Cost:** 835,893.79


```
SELECT P.PRODUCT_ID,
	P.INTERNAL_NAME,
	P.PRODUCT_TYPE_ID,
	GI.ID_VALUE AS NETSUITE_ID
FROM PRODUCT P
	LEFT JOIN GOOD_IDENTIFICATION GI
		ON GI.PRODUCT_ID=P.PRODUCT_ID
		AND GI.GOOD_IDENTIFICATION_TYPE_ID='ERP_ID'
WHERE GI.ID_VALUE = '' OR GI.ID_VALUE IS NULL;
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

**Query Cost:** 115,248.67


```
SELECT 
    P.PRODUCT_ID AS PRODUCT_ID,
    SP.SHOPIFY_PRODUCT_ID AS SHOPIFY_ID,
    P.PRODUCT_ID AS HOTWAX_ID,
    GI.ID_VALUE AS ERP_ID
FROM PRODUCT P
	JOIN GOOD_IDENTIFICATION GI
		ON P.PRODUCT_ID = GI.PRODUCT_ID
		AND GI.GOOD_IDENTIFICATION_TYPE_ID = 'ERP_ID'
	JOIN SHOPIFY_PRODUCT SP
		ON SP.PRODUCT_ID = P.PRODUCT_ID
WHERE GI.ID_VALUE IS NOT NULL;
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


```
SELECT DISTINCT
    OH.ORDER_ID,
    OH.STATUS_ID,
    OH.EXTERNAL_ID,
    OHI.ORDER_HISTORY_ID,
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
    ON OS.ORDER_ID = OH.ORDER_ID
   AND OS.STATUS_ID = 'ORDER_COMPLETED'
   AND OS.STATUS_DATETIME >= '2023-08-01 00:00:00'
   AND OS.STATUS_DATETIME <=  '2023-08-31 59:59:59'
JOIN ORDER_HISTORY OHI
    ON OHI.ORDER_ID = OH.ORDER_ID
JOIN ORDER_ITEM OI
    ON OI.ORDER_ID = OHI.ORDER_ID
   AND OI.ORDER_ITEM_SEQ_ID = OHI.ORDER_ITEM_SEQ_ID
JOIN PRODUCT P
    ON P.PRODUCT_ID = OI.PRODUCT_ID
JOIN ORDER_ITEM_SHIP_GROUP OSG
    ON OSG.ORDER_ID = OHI.ORDER_ID
   AND OSG.SHIP_GROUP_SEQ_ID = OHI.SHIP_GROUP_SEQ_ID
JOIN FACILITY F
    ON F.FACILITY_ID = OSG.FACILITY_ID;
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
SELECT DISTINCT
    O.ORDER_ID,
    O.GRAND_TOTAL AS TOTAL_AMOUNT,
    OP.PAYMENT_METHOD_TYPE_ID AS PAYMENT_METHOD,
    O.EXTERNAL_ID AS SHOPIFY_ORDER_ID
FROM ORDER_HEADER O
LEFT JOIN ORDER_PAYMENT_PREFERENCE OP
    ON OP.ORDER_ID = O.ORDER_ID
WHERE O.STATUS_ID = 'ORDER_CREATED'
  AND O.ORDER_TYPE_ID = 'SALES_ORDER'
ORDER BY O.ORDER_DATE DESC;
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
    O.ORDER_ID,
    O.STATUS_ID AS ORDER_STATUS,
    P.STATUS_ID AS PAYMENT_STATUS,
    S.STATUS_ID AS SHIPMENT_STATUS
FROM ORDER_HEADER O
JOIN ORDER_PAYMENT_PREFERENCE P
    ON P.ORDER_ID = O.ORDER_ID
LEFT JOIN SHIPMENT S
    ON S.PRIMARY_ORDER_ID = O.ORDER_ID
WHERE P.STATUS_ID NOT IN ('PAYMENT_RECEIVED', 'PAYMENT_SETTLED')
  AND (S.STATUS_ID IS NULL OR S.STATUS_ID <> 'SHIPMENT_SHIPPED');
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
	COUNT(O.ORDER_ID) AS TOTAL_ORDER,
	HOUR(O.STATUS_DATETIME) AS HOUR
FROM
	ORDER_STATUS O
WHERE
	O.STATUS_DATETIME
		BETWEEN '2024-10-28 00:00:01'
			AND '2024-10-28 23:59:59'
	AND O.STATUS_ID = 'ORDER_COMPLETED'
GROUP BY HOUR
ORDER BY HOUR;
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
