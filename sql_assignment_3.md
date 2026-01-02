**Assignment Link: https://github.com/saastechacademy/foundation/blob/main/udm/intermediate/sql-assignment/sql-assignment-3.md**

---

1. Completed Sales Orders (Physical Items)

Business Problem:
Merchants need to track only physical items (requiring shipping and fulfillment) for logistics and shipping-cost analysis.

Fields to Retrieve:

ORDER_ID
ORDER_ITEM_SEQ_ID
PRODUCT_ID
PRODUCT_TYPE_ID
SALES_CHANNEL_ENUM_ID
ORDER_DATE
ENTRY_DATE
STATUS_ID
STATUS_DATETIME
ORDER_TYPE_ID
PRODUCT_STORE_ID

**Query:**

**Query cost:** 131,586.55

```
select
	oi.ORDER_ID,
	oi.ORDER_ITEM_SEQ_ID,
	oi.PRODUCT_ID,
	p.PRODUCT_TYPE_ID,
	o.SALES_CHANNEL_ENUM_ID,
	o.ORDER_DATE,
	o.ENTRY_DATE,
	oi.STATUS_ID,
	os.STATUS_DATETIME,
	o.ORDER_TYPE_ID,
	o.PRODUCT_STORE_ID
from order_header o
join order_status os on os.ORDER_ID=o.ORDER_ID
join order_item oi on o.ORDER_ID=oi.ORDER_ID
join product p on oi.PRODUCT_ID=p.PRODUCT_ID
JOIN product_type pt ON p.product_type_id = pt.product_type_id
WHERE pt.is_physical = 'Y' AND p.is_variant = 'Y'
AND o.status_id = 'ORDER_COMPLETED' AND o.order_type_id = 'SALES_ORDER';
```

**Output:**

![image](https://github.com/user-attachments/assets/2d6e6be0-fbc4-42f6-b04f-effcae92a820)



------------------------------------------------------------------------------

2. Completed Return Items

Business Problem:
Customer service and finance often need insights into returned items to manage refunds, replacements, and inventory restocking.

Fields to Retrieve:

RETURN_ID
ORDER_ID
PRODUCT_STORE_ID
STATUS_DATETIME
ORDER_NAME
FROM_PARTY_ID
RETURN_DATE
ENTRY_DATE
RETURN_CHANNEL_ENUM_ID

**Query:**

**Query cost:** 5,726.11

```
select
	ri.RETURN_ID,
	ri.ORDER_ID,
	oh.PRODUCT_STORE_ID,
	(SELECT os.status_datetime FROM order_status os WHERE os.order_id=ri.order_id
       ORDER BY os.status_datetime desc limit 1) as STATUS_DATETIME,
	oh.ORDER_NAME,
	rh.FROM_PARTY_ID,
	rh.RETURN_DATE,
	rh.ENTRY_DATE,
	rh.RETURN_CHANNEL_ENUM_ID
from return_header rh
join return_item ri on rh.return_id=ri.return_id
join order_header oh on ri.ORDER_ID=oh.ORDER_ID
where ri.status_id='RETURN_COMPLETED';
```

**Output:**

![image](https://github.com/user-attachments/assets/23867d21-eb8e-41c7-b015-e2c5e3c7e083)



------------------------------------------------------------------------------

3. Single-Return Orders (Last Month)

Business Problem:
The mechandising team needs a list of orders that only have one return.

Fields to Retrieve:

PARTY_ID
FIRST_NAME

**Query:**

**Query cost:** 1,722.88

```
select distinct
	rh.from_party_id as party_id,
	per.first_name
from return_header rh 
join person per on rh.from_party_id=per.party_id
join return_item ri on ri.return_id=rh.return_id
where return_date between "2024-12-01 00:00:00" AND "2024-12-31 11:59:59"
GROUP by ri.order_id,ri.RETURN_ID,rh.FROM_PARTY_ID
HAVING COUNT(rh.return_id) = 1;
```

**Output:**

![image](https://github.com/user-attachments/assets/504e18d4-2c84-4769-855e-7b852bd69dec)



------------------------------------------------------------------------------

4. Returns and Appeasements

Business Problem:
The retailer needs the total amount of items, were returned as well as how many appeasements were issued.

Fields to Retrieve:

TOTAL RETURNS
RETURN $ TOTAL
TOTAL APPEASEMENTS
APPEASEMENTS $ TOTAL

**Query:**

**Query cost:** 462.16

```
select 
    count(ri.return_id) as total_returns,
	SUM(ri.return_price) AS return_total,
    COUNT(ra.return_adjustment_id) AS total_appeasements,
	SUM(ra.amount) AS appeasement_total
from return_item ri 
left join return_adjustment ra
on ri.return_id=ra.return_id
where RETURN_ADJUSTMENT_TYPE_ID="APPEASEMENT";
```

**Output:**

![image](https://github.com/user-attachments/assets/2a700fb2-ed2c-4f8d-99d0-e13d62764239)



------------------------------------------------------------------------------

5. Detailed Return Information

Business Problem:
Certain teams need granular return data (reason, date, refund amount) for analyzing return rates, identifying recurring issues, or updating policies.

Fields to Retrieve:

RETURN_ID
ENTRY_DATE
RETURN_ADJUSTMENT_TYPE_ID (refund type, store credit, etc.)
AMOUNT
COMMENTS
ORDER_ID
ORDER_DATE
RETURN_DATE
PRODUCT_STORE_ID

**Query:**

**Query cost:** 7,021.7

```
select distinct
      rh.RETURN_ID, rh.ENTRY_DATE,
      ra.RETURN_ADJUSTMENT_TYPE_ID,
      ra.amount, ra.COMMENTS,
      ri.order_id, oh.order_date,
      rh.return_date, oh.PRODUCT_STORE_ID
from return_header rh
join return_item ri on ri.return_id = rh.return_id
join return_adjustment ra on ra.return_id = rh.return_id
join order_header oh on oh.order_id=ri.order_id;
```

**Output:**

![image](https://github.com/user-attachments/assets/943afd25-ad13-4413-9335-ad9110bf4738)



------------------------------------------------------------------------------

6. Orders with Multiple Returns

Business Problem:
Analyzing orders with multiple returns can identify potential fraud, chronic issues with certain items, or inconsistent shipping processes.

Fields to Retrieve:

ORDER_ID
RETURN_ID
RETURN_DATE
RETURN_REASON
RETURN_QUANTITY

**Query:**

**Query cost:** 1,956.88

```
SELECT 
    ri.order_id, rh.return_id,
    rh.return_date,
    ri.reason AS return_reason,
    ri.return_quantity
FROM return_header rh
JOIN return_item ri ON ri.return_id = rh.return_id
WHERE ri.order_id IN (SELECT order_id FROM return_item
GROUP BY order_id HAVING COUNT(DISTINCT return_id) > 1);
```

**Output:**

![image](https://github.com/user-attachments/assets/c13e80aa-8ac7-40e4-9c14-ab7fdcd4b174)



------------------------------------------------------------------------------

7. Store with Most One-Day Shipped Orders (Last Month)

Business Problem:
Identify which facility (store) handled the highest volume of “one-day shipping” orders in the previous month, useful for operational benchmarking.

Fields to Retrieve:

FACILITY_ID
FACILITY_NAME
TOTAL_ONE_DAY_SHIP_ORDERS
REPORTING_PERIOD

**Query:**

**Query cost:** 76,336.36

```
SELECT oisg.facility_id, f.facility_name,
      count(oisg.order_id) as TOTAL_ONE_DAY_SHIP_ORDERS,
      concat(min(oh.entry_date), ' to ', max(oh.entry_date)) as date_range
FROM order_header oh
JOIN order_item_ship_group oisg ON oh.order_id=oisg.order_id 
JOIN facility f ON f.facility_id=oisg.facility_id
JOIN order_shipment os ON os.order_id=oisg.order_id
JOIN shipment s ON s.SHIPMENT_ID=os.SHIPMENT_ID
WHERE month(oh.entry_date)=month('2025-05-01')-1
AND year(oh.entry_date)=year('2025-05-01')-1
AND oisg.shipment_method_type_id='NEXT_DAY'
AND s.STATUS_ID='SHIPMENT_SHIPPED'
Group by oisg.facility_id
order by TOTAL_ONE_DAY_SHIP_ORDERS desc;
```

**Result:**

![image](https://github.com/user-attachments/assets/73238c53-ef51-43b1-a0b5-c700ba3a68c0)



------------------------------------------------------------------------------

8. List of Warehouse Pickers

Business Problem:
Warehouse managers need a list of employees responsible for picking and packing orders to manage shifts, productivity, and training needs.

Fields to Retrieve:

PARTY_ID (or Employee ID)
NAME (First/Last)
ROLE_TYPE_ID (e.g., “WAREHOUSE_PICKER”)
FACILITY_ID (assigned warehouse)
STATUS (active or inactive employee)

**Query:**

**Query cost:** 7,808.33

```
select distinct
	p.PARTY_ID, per.first_name as NAME,
	plr.ROLE_TYPE_ID, pl.FACILITY_ID,
	p.status_id as Status
from picklist pl
join picklist_role plr on pl.picklist_id=plr.picklist_id
join person per on plr.PARTY_ID=per.PARTY_ID
join party p on per.PARTY_ID = p.PARTY_ID
where p.STATUS_ID is not null;
```

**Output:**

![image](https://github.com/user-attachments/assets/b4502ee9-886b-4152-950d-f03d4a0f11c2)



------------------------------------------------------------------------------

9. Total Facilities That Sell the Product

Business Problem:
Retailers want to see how many (and which) facilities (stores, warehouses, virtual sites) currently offer a product for sale.

Fields to Retrieve:

PRODUCT_ID
PRODUCT_NAME (or INTERNAL_NAME)
FACILITY_COUNT (number of facilities selling the product)
(Optionally) a list of FACILITY_IDs if more detail is needed


**Query:**

**Query cost:** 18.82

```
SELECT distinct p.product_id, p.internal_name,
       count(pf.facility_id) as facility_count
FROM product p
JOIN product_facility pf ON pf.product_id = p.product_id
AND p.PRODUCT_ID = '21127' group by p.product_id;
```

**Output:**

![image](https://github.com/user-attachments/assets/77a53455-80b2-447d-be14-6529c6ac440d)



------------------------------------------------------------------------------

10. Total Items in Various Virtual Facilities

Business Problem:
Retailers need to study the relation of inventory levels of products to the type of facility it's stored at. Retrieve all inventory levels for products at locations and include the facility type Id. Do not retrieve facilities that are of type Virtual.

Fields to Retrieve:

PRODUCT_ID
FACILITY_ID
FACILITY_TYPE_ID
QOH (Quantity on Hand)
ATP (Available to Promise)


**Query:**

**Query cost:** 367,088.11

```
select
    ii.product_id, ii.facility_id, f.facility_type_id,
    ii.quantity_on_hand_total AS QOH,
    ii.available_to_promise_total AS ATP
FROM inventory_item ii
JOIN facility f on f.facility_id = ii.facility_id
JOIN facility_type ft on ft.facility_type_id = f. facility_type_id
WHERE ft.parent_type_id <> 'VIRTUAL_FACILITY'
AND ii.quantity_on_hand_total > 0 AND ii.available_to_promise_total > 0;
```

**Output:**

![image](https://github.com/user-attachments/assets/2cf931be-fba7-419e-9532-71e55df61dd8)



------------------------------------------------------------------------------

11. Transfer Orders Without Inventory Reservation

Business Problem:
When transferring stock between facilities, the system should reserve inventory. If it isn’t reserved, the transfer may fail or oversell.

Fields to Retrieve:

TRANSFER_ORDER_ID
FROM_FACILITY_ID
TO_FACILITY_ID
PRODUCT_ID
REQUESTED_QUANTITY
RESERVED_QUANTITY
TRANSFER_DATE
STATUS


**Query:**

**Query cost:** 3.09

```
SELECT 
    it.inventory_transfer_id AS transfer_order_id,
    it.facility_id AS from_facility_id,
    it.facility_id_to AS to_facility_id,
    it.product_id,
    it.quantity AS requested_quantity,
    0 AS reserved_quantity,
    it.send_date AS transfer_date,
    it.status_id AS status
FROM inventory_transfer it
LEFT JOIN order_item_ship_grp_inv_res oisgir 
    ON oisgir.inventory_item_id = it.inventory_item_id
WHERE oisgir.inventory_item_id IS NULL;
```

**Output:**

![image](https://github.com/user-attachments/assets/b774e7d8-9d4a-43a2-8d57-5496462e4061)



------------------------------------------------------------------------------

12 Orders Without Picklist

Business Problem:
A picklist is necessary for warehouse staff to gather items. Orders missing a picklist might be delayed and need attention.

Fields to Retrieve:

ORDER_ID
ORDER_DATE
ORDER_STATUS
FACILITY_ID
DURATION (How long has the order been assigned at the facility)


**Query:**

**Query cost:** 105,538.65

```
SELECT 
    oh.order_id,
    oh.order_date,
    oh.status_id AS order_status,
    oh.origin_facility_id AS facility_id,
    DATEDIFF(CURDATE(), oh.entry_date) AS duration
FROM order_header oh
LEFT JOIN picklist_item pli 
    ON pli.order_id = oh.order_id
WHERE pli.order_id IS NULL;
```

**Output:**

![image](https://github.com/user-attachments/assets/874b5be9-9fa2-42aa-b4d1-d0bcfd18aa5b)



------------------------------------------------------------------------------
