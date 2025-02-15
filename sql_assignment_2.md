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

```
select 
	o.order_id,
	p.party_id,
	concat(p.first_name,' ',p.last_name) as customer_name,
	pa.address1 as street_address,
	pa.city,
	pa.state_province_geo_id as state_province,
	pa.postal_code,
	pa.country_geo_id as country_code,
	o.status_id as order_status,
	o.order_date
from order_header o
inner join order_contact_mech ocm
on o.order_id=ocm.order_id
inner join party_contact_mech pcm
on ocm.contact_mech_id=pcm.contact_mech_id
inner join postal_address pa
on pcm.contact_mech_id=pa.contact_mech_id
inner join person p
on pcm.party_id=p.party_id
where o.order_date between
'2023-10-01 00:00:01' and '2023-10-31 23:59:59';
```

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

```
select 
	o.order_id,
	concat(p.first_name,' ',p.last_name) as customer_name,
	pa.address1 as street_address,
	pa.city,
	pa.state_province_geo_id as state_province,
	pa.postal_code,
	o.GRAND_TOTAL as total_amount,
	o.order_date,
	o.status_id as order_status
from order_header o
inner join order_contact_mech ocm
on o.order_id=ocm.order_id
inner join party_contact_mech pcm
on ocm.contact_mech_id=pcm.contact_mech_id
inner join postal_address pa
on pcm.contact_mech_id=pa.contact_mech_id
inner join person p
on pcm.party_id=p.party_id
where pa.state_province_geo_id='NY'
and pa.city='New York';
```

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

```
select
	oi.product_id,
	p.internal_name,
	count(oi.product_id) as TOTAL_QUANTITY_SOLD,
	concat(pa.city,' (',pa.STATE_PROVINCE_GEO_ID,')') as CITY_OR_STATE,
	sum(oi.UNIT_PRICE) as REVENUE
from order_item oi
left join product p
on oi.product_id=p.product_id
inner join order_contact_mech ocm
on oi.order_id=ocm.order_id
inner join postal_address pa
on ocm.contact_mech_id=pa.contact_mech_id
group by oi.product_id, 
    p.internal_name, 
    pa.city, 
    pa.STATE_PROVINCE_GEO_ID
order by revenue desc limit 1;
```

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

```
SELECT 
	f.FACILITY_ID, 
    f.FACILITY_NAME, 
    sum(oi.QUANTITY), 
	sum(o.grand_Total) as REVENUE, 
	MIN(o.ORDER_DATE) AS START_DATE, 
    MAX(o.ORDER_DATE) AS END_DATE
from order_item oi 
join order_item_ship_group oisg on oi.ORDER_ID=oisg.ORDER_ID 
join facility f on f.FACILITY_ID=oisg.FACILITY_ID
join order_header o on o.ORDER_ID=oi.ORDER_ID 
GROUP BY f.FACILITY_ID ;
```

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

```
SELECT
    id.inventory_item_id,
    i.PRODUCT_ID,
    i.facility_id,
    id.quantity_on_hand_diff AS quantity_lost_or_damaged,
    id.reason_enum_id AS REASON_CODE,
    id.effective_date AS TRANSACTION_DATE
from Inventory_Item_Detail id
join Inventory_Item i ON id.inventory_item_id = i.inventory_item_id
where id.reason_enum_id IN ('VAR_LOST','VAR_DAMAGED');
```

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

```
```


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

---------------------------------------------------------------------------------------------
