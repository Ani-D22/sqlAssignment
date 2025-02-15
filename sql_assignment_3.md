**Assignment Link: https://github.com/saastechacademy/foundation/blob/main/udm/intermediate/sql-assignment/sql-assignment-3.md**

1 Completed Sales Orders (Physical Items)
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

Query:

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
from order_status os
right join order_header o
on os.ORDER_ID=o.ORDER_ID
inner join order_item oi
on o.ORDER_ID=oi.ORDER_ID
inner join product p
on oi.PRODUCT_ID=p.PRODUCT_ID;
```

------------------------------------------------------------------------------

2 Completed Return Items
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

Query:

```
select
	ri.RETURN_ID,
	ri.ORDER_ID,
	oh.PRODUCT_STORE_ID,
	ri.status_id as STATUS_DATETIME,
	oh.ORDER_NAME,
	rh.FROM_PARTY_ID,
	rh.RETURN_DATE,
	rh.ENTRY_DATE,
	rh.RETURN_CHANNEL_ENUM_ID
from return_header rh
inner join return_item ri
on rh.return_id=ri.return_id
inner join order_header oh
on ri.ORDER_ID=oh.ORDER_ID;

```

------------------------------------------------------------------------------

3 Single-Return Orders (Last Month)
Business Problem:
The mechandising team needs a list of orders that only have one return.

Fields to Retrieve:

PARTY_ID
FIRST_NAME

Query:

```
select 
	rh.from_party_id as party_id,
	per.first_name
from return_header rh 
join person per on rh.from_party_id=per.party_id
join return_item ri on ri.return_id=rh.return_id
where return_date between "2024-12-01" AND "2024-12-31"
GROUP by ri.order_id,ri.RETURN_ID,rh.FROM_PARTY_ID
HAVING COUNT(rh.return_id) = 1;
```

------------------------------------------------------------------------------

4 Returns and Appeasements
Business Problem:
The retailer needs the total amount of items, were returned as well as how many appeasements were issued.

Fields to Retrieve:

TOTAL RETURNS
RETURN $ TOTAL
TOTAL APPEASEMENTS
APPEASEMENTS $ TOTAL

Query:

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

------------------------------------------------------------------------------

5 Detailed Return Information
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

Query:

```
select 
      ra.RETURN_ID, rh.ENTRY_DATE,
      ra.RETURN_ADJUSTMENT_TYPE_ID,
      ra.amount, ra.COMMENTS,
      oh.order_id, oh.order_date,
      rh.return_date, oh.PRODUCT_STORE_ID
from return_header rh 
left join return_adjustment ra
on ra.RETURN_ID=rh.RETURN_ID
left join return_item ri
on ra.ORDER_ID=ri.ORDER_ID 
join order_header oh
on oh.ORDER_ID=ri.order_id;
```

------------------------------------------------------------------------------

6 Orders with Multiple Returns
Business Problem:
Analyzing orders with multiple returns can identify potential fraud, chronic issues with certain items, or inconsistent shipping processes.

Fields to Retrieve:

ORDER_ID
RETURN_ID
RETURN_DATE
RETURN_REASON
RETURN_QUANTITY

Query:

```
SELECT 
    ri.order_id, rh.return_id,
    rh.return_date,
    ri.reason AS return_reason,
    ri.return_quantity
FROM return_header rh
JOIN return_item ri
ON ri.return_id = rh.return_id
WHERE ri.order_id IN (
    SELECT order_id
    FROM return_item
    GROUP BY order_id
    HAVING COUNT(DISTINCT return_id) > 1
);
```

------------------------------------------------------------------------------

8 List of Warehouse Pickers
Business Problem:
Warehouse managers need a list of employees responsible for picking and packing orders to manage shifts, productivity, and training needs.

Fields to Retrieve:

PARTY_ID (or Employee ID)
NAME (First/Last)
ROLE_TYPE_ID (e.g., “WAREHOUSE_PICKER”)
FACILITY_ID (assigned warehouse)
STATUS (active or inactive employee)

Query:

```
select
	pr.PARTY_ID,
	p.first_name as NAME,
	pr.ROLE_TYPE_ID,
	pl.FACILITY_ID,
	(CASE WHEN pr.thru_date IS NULL THEN 'ACTIVE' ELSE 'INACTIVE' END AS status)
from picklist pl
inner join picklist_role pr
on pl.picklist_id=pr.picklist_id
inner join person p
on pr.PARTY_ID=p.PARTY_ID;
```

------------------------------------------------------------------------------
