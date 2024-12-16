insert into orders(id ,merchant_id,customer_id,currency_code,order_amount,order_ref_number,sbi_order_ref_number,status,other_details,expiry,multi_accounts,payment_mode,order_hash,created_by,updated_bycreated_date,updated_date) values(5D0B65DD45AA459F891A55E0AAAB8A33,
1122323,INR,2,TESTNEERAJ123,103096577281661wDYJJ,CREATED,600000,wv37n7MEQjeNDymU5oOGMG0RDmz1BMzxhfSw6U9x2rr1v8NwWsSPcmNVMFh4mvSNSNVMLce5rCXnQAg,1734355740148,1734355740148,https://dev.epay.sbi/2.0/#);



INSERT INTO orders (
    id,
    merchant_id,
    customer_id,
    currency_code,
    order_amount,
    order_ref_number,
    sbi_order_ref_number,
    status,
    other_details,
    expiry,
    multi_accounts,
    payment_mode,
    order_hash,
    created_by,
    updated_by,
    created_date,
    updated_date
) 
VALUES (
    '5D0B65DD45AA459F891A55E0AAAB8A33', 
    1122323, 
    NULL, 
    'INR', 
    2, 
    'TESTNEERAJ123', 
    '103096577281661wDYJJ', 
    'CREATED', 
    '600000', 
    TO_DATE('1734355740148', 'YYYYMMDDHH24MISS'), 
    TO_DATE('1734355740148', 'YYYYMMDDHH24MISS'), 
    'https://dev.epay.sbi/2.0/#',
    'wv37n7MEQjeNDymU5oOGMG0RDmz1BMzxhfSw6U9x2rr1v8NwWsSPcmNVMFh4mvSNSNVMLce5rCXnQAg',
    NULL,
    NULL
);
