# Lab 5

## Завдання 1
> Напишіть SQL запит, який буде відображати таблицю `order_details`
> та поле `customer_id` з таблиці orders відповідно для кожного поля запису з таблиці `order_details`.
> Це має бути зроблено за допомогою вкладеного запиту в операторі `SELECT`.

### variant 1
```sql
    select
    (select customer_id from orders where orders.id = od.order_id) as customer_id,
    od.*
    from order_details as od;
```

### variant 2
```sql
select o.customer_id, od.*
from order_details as od
join orders o on od.order_id = o.id;
```

### variant 3
```sql
DROP FUNCTION IF EXISTS GetCustomer;

DELIMITER //
CREATE FUNCTION GetCustomer(order_id INT)
    RETURNS INT
    READS SQL DATA
    NO SQL
BEGIN
    DECLARE customerId INT;
    select orders.customer_id
    into customerId
    from orders
    where orders.id = order_id
    limit 1;
    RETURN customerId;
END //
DELIMITER ;

select GetCustomer(od.order_id) as customer_id, od.*
from order_details as od;
```

## Завдання 2
> Напишіть SQL запит, який буде відображати таблицю `order_details`.
> Відфільтруйте результати так, щоб відповідний запис із таблиці `orders` виконував умову `shipper_id = 3`.
> Це має бути зроблено за допомогою вкладеного запиту в операторі `WHERE`.

### variant 1
```sql
select * from order_details
where (select id
       from orders
       where
           shipper_id = 3 and orders.id = order_details.order_id
       limit 1);
```

### variant 2
```sql
select * from order_details
where order_details.order_id in (select id
                                 from orders
                                 where shipper_id = 3);
```

### variant 3
```sql
DROP FUNCTION IF EXISTS HasShipper;

DELIMITER //
CREATE FUNCTION HasShipper(shipper_id INT, order_id INT )
    RETURNS BOOLEAN
    READS SQL DATA
    NO SQL
BEGIN
    RETURN (select count(order_id)
            from orders as o
            where
                o.shipper_id = shipper_id and o.id = order_id
            limit 1);
END //
DELIMITER ;

select * from order_details
where HasShipper(3, order_details.order_id);
```

## Завдання 3
> Напишіть SQL запит, вкладений в операторі FROM, який буде обирати рядки з умовою quantity>10 з таблиці order_details.
> Для отриманих  даних знайдіть середнє значення поля quantity — групувати слід за order_id.

```sql
select avg(quantity) as 'середнє значення quantity', order_id
from (select quantity, order_id
       from order_details
       where quantity > 10) as temp_tabl
group by order_id;
```

## Завдання 4
> Розв’яжіть завдання 3, використовуючи оператор WITH для створення тимчасової таблиці temp.
> 
```sql
with temp_tabl as (select quantity, order_id
    from order_details
    where quantity > 10)
select avg(quantity) as 'середнє значення quantity', order_id
from temp_tabl
group by order_id;
```

## Завдання 5
> Створіть функцію з двома параметрами, яка буде ділити перший параметр на другий.
> Обидва параметри та значення, що повертається, повинні мати тип FLOAT.
> Використайте конструкцію DROP FUNCTION IF EXISTS.
> Застосуйте функцію до атрибута quantity таблиці order_details.

```sql
DROP FUNCTION IF EXISTS Divide;

DELIMITER //
CREATE FUNCTION Divide(arg1 FLOAT, arg2 FLOAT)
    RETURNS FLOAT
    READS SQL DATA
    NO SQL
BEGIN
    RETURN arg1 / arg2;
END //
DELIMITER ;

select Divide(quantity, 3) as 'в подарунок', quantity
from order_details;
```
