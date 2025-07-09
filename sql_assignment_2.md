**Assignment Link: https://github.com/saastechacademy/foundation/blob/main/udm/intermediate/sql-assignment/sql-assignment-2.md**

---

5.1. Shipping Addresses for October 2023 Orders
Business Problem:
Customer Service might need to verify addresses for orders placed or completed in October 2023. This helps ensure shipments are delivered correctly and prevents address-related issues.

Fields to Retrieve:

ORDER_ID
PARTY_ID (Customer ID)
CUSTOMER_NAME (or FIRST_NAME / LAST_NAME)
STREET_ADDRESS
CITY
STATE_PROVINCE
POSTAL_CODE
COUNTRY_CODE
ORDER_STATUS
ORDER_DATE

**Query:**

**Query Cost:** 36,888.31

```select 
	o.order_id, p.party_id,
	concat(p.first_name,' ',p.last_name) as customer_name,
	pa.address1 as street_address, pa.city,
	pa.state_province_geo_id as state_province,
	pa.postal_code, pa.country_geo_id as country_code,
	o.status_id as order_status, o.order_date
from order_header o
join order_status os on o.order_id = os.order_id
join order_contact_mech ocm on o.order_id=ocm.order_id
join postal_address pa on ocm.contact_mech_id=pa.contact_mech_id
join order_role orl on orl.order_id = o.order_id
join person p on orl.party_id=p.party_id
where os.STATUS_DATETIME between '2020-10-01 00:00:00' and '2023-10-31 23:59:59'
and os.STATUS_ID = 'ORDER_COMPLETED'
and orl.ROLE_TYPE_ID = 'SHIP_TO_CUSTOMER'
and ocm.CONTACT_MECH_PURPOSE_TYPE_ID='SHIPPING_LOCATION';
```

**Output:**

![image](https://github.com/user-attachments/assets/2279ada3-eb58-4537-b934-ca66aafb5e92)



-------------------------------------------------------------------------------------------

5.2. Orders from New York
Business Problem:
Companies often want region-specific analysis to plan local marketing, staffing, or promotions in certain areas—here, specifically, New York.

Fields to Retrieve:

ORDER_ID
CUSTOMER_NAME
STREET_ADDRESS (or shipping address detail)
CITY
STATE_PROVINCE
POSTAL_CODE
TOTAL_AMOUNT
ORDER_DATE
ORDER_STATUS

**Query:**

**Query cost:** 8,029.26

```
select 
	o.order_id,
	concat(p.first_name,' ',p.last_name) as customer_name,
	pa.address1 as street_address, pa.city,
	pa.state_province_geo_id as state_province,
	pa.postal_code, o.GRAND_TOTAL as total_amount,
	o.order_date, o.status_id as order_status
from order_header o
join order_contact_mech ocm on o.order_id=ocm.order_id
join party_contact_mech pcm on ocm.contact_mech_id=pcm.contact_mech_id
and ocm.CONTACT_MECH_PURPOSE_TYPE_ID='SHIPPING_LOCATION'
join postal_address pa on pcm.contact_mech_id=pa.contact_mech_id
join person p on pcm.party_id=p.party_id
where pa.state_province_geo_id='NY' and pa.city='New York';
```

**Output:**

![image](https://github.com/user-attachments/assets/bebe9b78-d6de-4541-a83c-10c4f90716ea)



-------------------------------------------------------------------------------------------


5.3. Top-Selling Product in New York
Business Problem:
Merchandising teams need to identify the best-selling product(s) in a specific region (New York) for targeted restocking or promotions.

Fields to Retrieve:

PRODUCT_ID
INTERNAL_NAME
TOTAL_QUANTITY_SOLD
CITY / STATE (within New York region)
REVENUE (optionally, total sales amount)

**Query:**

**Query cost:** 13,319.32

```
select
	oi.product_id, p.internal_name,
	count(oi.product_id) as TOTAL_QUANTITY_SOLD,
	concat(pa.city,' (',pa.STATE_PROVINCE_GEO_ID,')') as CITY_OR_STATE,
	sum(oi.UNIT_PRICE * oi.QUANTITY) as REVENUE
from order_item oi
left join product p on oi.product_id=p.product_id
join order_contact_mech ocm on oi.order_id=ocm.order_id
inner join postal_address pa on ocm.contact_mech_id=pa.contact_mech_id
where pa.CITY = 'New York' and pa.STATE_PROVINCE_GEO_ID = 'NY' 
AND oi.status_id = 'ITEM_COMPLETED'
group by oi.product_id, p.internal_name, pa.city, pa.STATE_PROVINCE_GEO_ID
order by revenue desc;
```

**Output:**

![image](https://github.com/user-attachments/assets/18032e9e-bd70-407a-8317-1e0d68551823)



-------------------------------------------------------------------------------------------

7.3. Store-Specific (Facility-Wise) Revenue
Business Problem:
Different physical or online stores (facilities) may have varying levels of performance. The business wants to compare revenue across facilities for sales planning and budgeting.

Fields to Retrieve:

FACILITY_ID
FACILITY_NAME
TOTAL_ORDERS
TOTAL_REVENUE
DATE_RANGE

**Query:**

**Query cost:** 93,370.76

```
SELECT 
	f.FACILITY_ID, 
    f.FACILITY_NAME,
	COUNT(o.order_id) as total_orders,
	SUM(o.grand_Total) as total_revenue,
    CONCAT(MIN(o.order_date), ' to ', MAX(o.order_date)) as date_range
from order_header o
join order_item_ship_group oisg on o.ORDER_ID = oisg.ORDER_ID 
join facility f on f.FACILITY_ID = oisg.FACILITY_ID
WHERE o.status_id = 'ORDER_COMPLETED'
GROUP BY f.FACILITY_ID
order by total_revenue desc;
```

**Output:**

![image](https://github.com/user-attachments/assets/fcab50f2-7f16-4f7f-a0d4-c0d30f7d2ac4)



-------------------------------------------------------------------------------------------


8.1. Lost and Damaged Inventory
Business Problem:
Warehouse managers need to track “shrinkage” such as lost or damaged inventory to reconcile physical vs. system counts.

Fields to Retrieve:

INVENTORY_ITEM_ID
PRODUCT_ID
FACILITY_ID
QUANTITY_LOST_OR_DAMAGED
REASON_CODE (Lost, Damaged, Expired, etc.)
TRANSACTION_DATE

**Query:** 

**Query cost:** 593,754.04


```
SELECT
    ii.inventory_item_id, ii.PRODUCT_ID, ii.facility_id,
    ABS(iiv.QUANTITY_ON_HAND_VAR) AS quantity_lost_or_damaged,
    iiv.variance_reason_id AS REASON_CODE,
    iiv.created_tx_stamp AS TRANSACTION_DATE
from inventory_item_variance iiv
join inventory_item ii ON iiv.inventory_item_id = ii.inventory_item_id
where iiv.variance_reason_id IN ('DAMAGED', 'VAR_LOST', 'VAR_DAMAGED');
```

**Output:**

![image](https://github.com/user-attachments/assets/d470a83f-3e28-4f6d-8142-1dc4c541402c)



-------------------------------------------------------------------------------------------

8.2 Low Stock or Out of Stock Items Report
Business Problem:
Avoiding out-of-stock situations is critical. This report flags items that have fallen below a certain reorder threshold or have zero available stock.

Fields to Retrieve:

PRODUCT_ID
PRODUCT_NAME
FACILITY_ID
QOH (Quantity on Hand)
ATP (Available to Promise)
REORDER_THRESHOLD
DATE_CHECKED

**Query:** 

**Query cost:** 593,754.04

```
SELECT 
    p.PRODUCT_ID,
    p.PRODUCT_NAME,
    ii.FACILITY_ID,
    ii.QUANTITY_ON_HAND_TOTAL AS QOH,
    ii.AVAILABLE_TO_PROMISE_TOTAL AS ATP,
    pf.REORDER_QUANTITY AS REORDER_THRESHOLD,
    pf.LAST_UPDATED_STAMP AS DATE_CHECKED
FROM PRODUCT p 
JOIN INVENTORY_ITEM ii ON p.PRODUCT_ID = ii.PRODUCT_ID
LEFT JOIN product_facility pf ON pf.PRODUCT_ID = ii.PRODUCT_ID
AND pf.FACILITY_ID = ii.FACILITY_ID
WHERE pf.MINIMUM_STOCK > ii.QUANTITY_ON_HAND_TOTAL
OR ii.AVAILABLE_TO_PROMISE_TOTAL = 0;
```

**Output:**

![image](https://github.com/user-attachments/assets/029479b3-4e90-4abb-abef-c7f888db77b2)



-------------------------------------------------------------------------------------------

8.3. Retrieve the Current Facility (Physical or Virtual) of Open Orders
Business Problem:
The business wants to know where open orders are currently assigned, whether in a physical store or a virtual facility (e.g., a distribution center or online fulfillment location).

Fields to Retrieve:

ORDER_ID
ORDER_STATUS
FACILITY_ID
FACILITY_NAME
FACILITY_TYPE_ID

**Query:**

**Query cost:** 23,858.46

```
SELECT
    o.order_id,
    o.status_id AS order_status,
    f.facility_id,
    f.facility_name,
    f.facility_type_id
from order_header o
join facility f ON o.origin_facility_id = f.facility_id 
where o.status_id IN ('ORDER_APPROVED','ORDER_CREATED');
```

**Output:**

![image](https://github.com/user-attachments/assets/8c0a15df-6640-49a5-8fb9-f3560e048cc9)



-------------------------------------------------------------------------------------------

8.4. Items Where QOH and ATP Differ
Business Problem:
Sometimes the Quantity on Hand (QOH) doesn’t match the Available to Promise (ATP) due to pending orders, reservations, or data discrepancies. This needs review for accurate fulfillment planning.

Fields to Retrieve:

PRODUCT_ID
FACILITY_ID
QOH (Quantity on Hand)
ATP (Available to Promise)
DIFFERENCE (QOH - ATP)

**Query:**

**Query cost:** 2,915,645.7

```
select ii.product_id,
       ii.facility_id,
       ii.quantity_on_hand_total as QOH,
       ii.available_to_promise_total as ATP,
       iid.AVAILABLE_TO_PROMISE_DIFF as Difference
from Inventory_Item ii
join inventory_item_detail iid on ii.INVENTORY_ITEM_ID = iid.INVENTORY_ITEM_ID
WHERE QUANTITY_ON_HAND_TOTAL <> AVAILABLE_TO_PROMISE_TOTAL;
```

**Output:**

![image](https://github.com/user-attachments/assets/9c8942f5-0269-41c5-8454-a955da28b81b)



-------------------------------------------------------------------------------------------

8.5. Order Item Current Status Changed Date-Time
Business Problem:
Operations teams need to audit when an order item’s status (e.g., from “Pending” to “Shipped”) was last changed, for shipment tracking or dispute resolution.

Fields to Retrieve:

ORDER_ID
ORDER_ITEM_SEQ_ID
CURRENT_STATUS_ID
STATUS_CHANGE_DATETIME
CHANGED_BY

**Query:**

**Query cost:** 1,707,343.5

```
SELECT 
    os.ORDER_ID,
    os.ORDER_ITEM_SEQ_ID,
    os.STATUS_ID AS CURRENT_STATUS_ID,
    os.STATUS_DATETIME AS STATUS_CHANGE_DATETIME,
    os.STATUS_USER_LOGIN AS CHANGED_BY
FROM order_status os
JOIN order_status os2 ON os.ORDER_ID = os2.ORDER_ID
AND os.ORDER_ITEM_SEQ_ID = os2.ORDER_ITEM_SEQ_ID
WHERE os.ORDER_ITEM_SEQ_ID IS NOT NULL
AND os.STATUS_DATETIME < os2.STATUS_DATETIME;
```

**Output:**

![image](https://github.com/user-attachments/assets/fbc0292b-aebb-4347-8a34-d8022173492e)



-------------------------------------------------------------------------------------------


8.6. Total Orders by Sales Channel
Business Problem:
Marketing and sales teams want to see how many orders come from each channel (e.g., web, mobile app, in-store POS, marketplace) to allocate resources effectively.

Fields to Retrieve:

SALES_CHANNEL
TOTAL_ORDERS
TOTAL_REVENUE
REPORTING_PERIOD


**Query:**

**Query cost:** 8,450.55

```
select
	sales_channel_enum_id as SALES_CHANNEL,
	count(order_id) as TOTAL_ORDERS,
	sum(grand_total) as TOTAL_REVENUE,
	DATE_FORMAT(order_date, '%Y-%m') as REPORTING_PERIOD
from order_header oh
GROUP BY SALES_CHANNEL, REPORTING_PERIOD
ORDER BY REPORTING_PERIOD DESC;
```

**Output:**

![image](https://github.com/user-attachments/assets/0277e3e5-ce42-4a4e-af0b-48d125ef4fa8)



---------------------------------------------------------------------------------------------
