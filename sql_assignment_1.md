![image](https://github.com/user-attachments/assets/8fc28667-d6e6-48fb-b9fc-d2b71516d0b6)**https://github.com/saastechacademy/foundation/blob/main/udm/intermediate/sql-assignment/sql-assignment-1.md**

**Query 1:**

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

**Output:**

![image](https://github.com/user-attachments/assets/f9f8d1b1-2014-43ac-b047-34d440cde592)


-----------------------------------------------------------------------

**Query 2:**
**a:**

select distinct
	p.product_id,
	p.product_type_id,
	p.internal_name
from inventory_item i
left join product p
on i.product_id=p.product_id;

**Output:**

![image](https://github.com/user-attachments/assets/91d3727a-e394-4982-aa85-d3c8aa530934)


**b:**

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

**Output:**

![image](https://github.com/user-attachments/assets/ba64e646-1b95-46f6-bb63-d98431dfef29)


-----------------------------------------------------------------------

**Query 3:**

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

**Output:**

![image](https://github.com/user-attachments/assets/11c87627-f311-4144-bde2-3d11c2d51d4b)


-----------------------------------------------------------------------

**Query4:**

SELECT 
    gi.product_id, 
    (CASE WHEN gi.GOOD_IDENTIFICATION_TYPE_ID = 'SHOPIFY_PROD_ID' THEN gi.id_value END) AS shopify_id, 
    (CASE WHEN gi.GOOD_IDENTIFICATION_TYPE_ID = 'HC_GOOD_ID_TYPE' THEN gi.id_value END) AS hotwax_id, 
    (CASE WHEN gi.GOOD_IDENTIFICATION_TYPE_ID = 'ERP_ID' THEN gi.id_value END) AS erp_id
FROM good_identification gi 
    WHERE gi.GOOD_IDENTIFICATION_TYPE_ID IN ('SHOPIFY_PROD_ID', 'HC_GOOD_ID_TYPE', 'ERP_ID')
GROUP BY gi.product_id;

**Output:**

![image](https://github.com/user-attachments/assets/83138b6f-78f3-4854-966f-dcdc35947fff)


----------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------

Query5:
//tbd

-----------------------------------------------------------------------

Query6:
//does not exist

-----------------------------------------------------------------------

Query7:

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

-----------------------------------------------------------------------

Query8:

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

-----------------------------------------------------------------------

Query9:

SELECT
	count(o.order_id),
	hour(o.order_date)
from order_header o
where o.order_date between '2024-10-28 00:00:01' and '2024-10-28 23:59:59'
and o.status_id = 'ORDER_COMPLETED'
group by hour(o.order_date);

-----------------------------------------------------------------------

Query10:
//tbd

-----------------------------------------------------------------------

Query11:

select
	count(o.order_id),
    os.CHANGE_REASON_ENUM_ID
from order_header o
inner join order_status os
on o.order_id=os.order_id
where o.order_date between '2024-12-01 00:00:01' and '2024-12-31 23:59:59'
and o.status_id = 'ORDER_CANCELLED' and CHANGE_REASON_ENUM_ID is not null;

-----------------------------------------------------------------------

Query12:
//tbd

-----------------------------------------------------------------------








