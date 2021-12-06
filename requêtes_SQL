# Requête sales

# Création view n-1
CREATE VIEW test_2020 AS (SELECT YEAR(orders.orderDate) as annee_2020, MONTH(orders.orderDate) as mois_2020, productlines.productLine as productline_2020, SUM(quantityOrdered) as QtyOrder_2020
FROM orderdetails
JOIN orders ON orderdetails.orderNumber = orders.orderNumber
JOIN products ON products.productCode = orderdetails.productCode
JOIN productlines ON productlines.productLine = products.productLine
WHERE YEAR(orders.orderDate) = YEAR(CURDATE() - INTERVAL 1 YEAR)
GROUP BY annee_2020, mois_2020, productline_2020
ORDER BY mois_2020);

#Création view n
CREATE VIEW test_2021 AS (SELECT YEAR(orders.orderDate) as annee_2021, MONTH(orders.orderDate) as mois_2021, productlines.productLine as productline_2021, SUM(quantityOrdered) as QtyOrder_2021
FROM orderdetails
JOIN orders ON orderdetails.orderNumber = orders.orderNumber
JOIN products ON products.productCode = orderdetails.productCode
JOIN productlines ON productlines.productLine = products.productLine
WHERE YEAR(orders.orderDate) = YEAR(CURDATE())
GROUP BY annee_2021, mois_2021, productline_2021
ORDER BY mois_2021);

#Calcul de la différence
SELECT annee_2021, mois_2021, productline_2021, QtyOrder_2021 AS '2021', QtyOrder_2020 AS '2020', (QtyOrder_2021 - QtyOrder_2020) AS Difference, (QtyOrder_2021 - QtyOrder_2020) / QtyOrder_2020 AS Variation
FROM test_2021
RIGHT OUTER JOIN test_2020 ON test_2021.productline_2021 = test_2020.productline_2020
WHERE test_2021.mois_2021 = test_2020.mois_2020
GROUP BY annee_2021, mois_2021, productline_2021, productline_2020, QtyOrder_2021, QtyOrder_2020, Difference;

# Sales requête additionnelle

SELECT YEAR(orders.orderDate) AS annee, MONTH(orders.orderDate) AS mois, offices.country, employees.lastName, SUM(orderdetails.quantityOrdered * orderdetails.priceEach)
FROM offices
JOIN employees ON offices.officeCode = employees.officeCode
JOIN customers ON employees.employeeNumber = customers.salesRepEmployeeNumber
JOIN orders ON orders.customerNumber = customers.customerNumber
JOIN orderdetails ON orderdetails.orderNumber = orders.orderNumber
WHERE orders.status != 'cancelled'
GROUP BY annee, mois, offices.country, employees.lastName
ORDER BY annee, mois;

# 1ere requete Finance : chiffre d'affaire sur les 2 derniers mois par pays

SELECT offices.country AS country, MONTH(orderDate) AS mois, SUM(quantityOrdered * priceEach) AS turnover
FROM orders
JOIN customers ON orders.customerNumber = customers.customerNumber
JOIN employees ON customers.salesRepEmployeeNumber = employees.EmployeeNumber
JOIN offices ON employees.officeCode = offices. officeCode
JOIN orderdetails ON orders.orderNumber = orderdetails.orderNumber
WHERE YEAR(orders.orderDate) = YEAR(NOW()) AND MONTH(orders.orderDate) >= MONTH(CURDATE() - INTERVAL 2 MONTH) AND orders.status != 'cancelled'
GROUP BY country, mois;

# Check 1ere requete finance 

SELECT offices.country AS country, YEAR(orderDate) AS annee, MONTH(orderDate) AS mois, SUM(quantityOrdered * priceEach) AS turnover
FROM orders
JOIN customers ON orders.customerNumber = customers.customerNumber
JOIN employees ON customers.salesRepEmployeeNumber = employees.EmployeeNumber
JOIN offices ON employees.officeCode = offices. officeCode
JOIN orderdetails ON orders.orderNumber = orderdetails.orderNumber
GROUP BY country, annee, mois;

SELECT SUM(quantityOrdered * priceEach) AS turnover
FROM orders
JOIN customers ON orders.customerNumber = customers.customerNumber
JOIN employees ON customers.salesRepEmployeeNumber = employees.EmployeeNumber
JOIN offices ON employees.officeCode = offices. officeCode
JOIN orderdetails ON orders.orderNumber = orderdetails.orderNumber
WHERE YEAR(orders.orderDate) = YEAR(NOW()) AND MONTH(orders.orderDate) >= MONTH(CURDATE() - INTERVAL 2 MONTH);

# Requête additionnelle Finances 1

SELECT YEAR(orders.orderDate), offices.country, SUM(orderdetails.quantityOrdered * orderdetails.priceEach) AS turnover_2021
FROM customers 
JOIN orders ON customers.customerNumber = orders.customerNumber
JOIN employees ON employees.employeeNumber = customers.salesRepEmployeeNumber
JOIN offices ON offices.officeCode = employees.officeCode
JOIN orderdetails ON orders.orderNumber = orderdetails.orderNumber
WHERE orders.status != 'cancelled'
GROUP BY YEAR(orders.orderDate), country;

# 2eme requête probleme payement

SELECT orders.orderNumber, payments.customerNumber, SUM(quantityOrdered * priceEach) AS mt_order, payments.amount
FROM orders
RIGHT JOIN customers ON orders.customerNumber = customers.customerNumber
RIGHT JOIN payments ON customers.customerNumber = payments.customerNumber
RIGHT JOIN orderdetails ON orders.orderNumber = orderdetails.orderNumber
GROUP BY orders.orderNumber, orders.customerNumber, payments.amount;

CREATE VIEW total_payment AS (SELECT unpaid.orderDate, unpaid.orderNumber, unpaid.customerNumber, unpaid.mt_order, unpaid.amount
FROM (
SELECT YEAR(orders.orderDate) AS orderDate, orders.orderNumber AS orderNumber,orders.status AS status, payments.customerNumber AS customerNumber, SUM(quantityOrdered * priceEach) AS mt_order, payments.amount AS amount
FROM orders
RIGHT JOIN customers ON orders.customerNumber = customers.customerNumber
RIGHT JOIN payments ON customers.customerNumber = payments.customerNumber
RIGHT JOIN orderdetails ON orders.orderNumber = orderdetails.orderNumber
GROUP BY orderDate, orders.orderNumber, orders.customerNumber, payments.amount) AS unpaid
WHERE unpaid.amount = unpaid.mt_order AND unpaid.status != 'cancelled');

CREATE VIEW total_order AS (SELECT YEAR(orders.orderDate) AS orderDate, orders.orderNumber, SUM(orderdetails.quantityOrdered * orderdetails.priceEach) AS mt_order
FROM orders
JOIN orderdetails ON orderdetails.orderNumber = orders.orderNumber
WHERE orders.status != 'cancelled'
GROUP BY orderDate, orders.orderNumber);

SELECT total_order.orderDate, total_order.orderNumber, total_order.mt_order, IFNULL(total_payment.amount, 0), total_order.mt_order - IFNULL(total_payment.amount, 0)
FROM total_payment
RIGHT JOIN total_order 
ON total_payment.orderNumber = total_order.orderNumber;

# Credit limit non respecté 

SELECT orders.customerNumber, orders.orderNumber, YEAR(orderDate) AS annee, SUM(quantityOrdered * priceEach) AS mt_order, creditLimit, (creditLimit - SUM(quantityOrdered * priceEach)) AS out_creditLimit
FROM orders
RIGHT JOIN customers ON orders.customerNumber = customers.customerNumber
RIGHT JOIN payments ON customers.customerNumber = payments.customerNumber
RIGHT JOIN orderdetails ON orders.orderNumber = orderdetails.orderNumber
WHERE orders.status != 'cancelled'
GROUP BY orders.customerNumber, orders.orderNumber, annee;

# Requête logistics

SELECT  orderdetails.productCode, products.productName , SUM(orderdetails.quantityOrdered) AS SommeOrders, products.quantityInStock
FROM orderdetails
JOIN products
ON orderdetails.productCode = products.productCode
JOIN orders
ON orderdetails.orderNumber = orders.orderNumber
WHERE orders.status != 'cancelled'
GROUP BY productCode
ORDER BY sum(orderdetails.quantityOrdered) desc limit 5;

# Requête additionnelle logistics

CREATE VIEW compstock AS (
SELECT  YEAR(orders.orderDate), orderdetails.productCode, products.productName , SUM(orderdetails.quantityOrdered) AS Somme_Oders, products.quantityInStock
FROM orderdetails
JOIN products
ON orderdetails.productCode = products.productCode
JOIN orders
ON orders.orderNumber = orderdetails.orderNumber
WHERE YEAR(orders.orderDate) = "2021" AND orders.status != 'cancelled'
GROUP BY YEAR(orders.orderDate), productCode
ORDER BY sum(orderdetails.quantityOrdered) desc);


SELECT * FROM compstock
WHERE quantityInStock<(Somme_Oders*3) LIMIT 10;

SELECT * FROM compstock
WHERE quantityInStock>(Somme_Oders*3) LIMIT 10;

# Requête human resources

SELECT rank_turnover.annee, rank_turnover.mois, rank_turnover.employer, rank_turnover.nom, rank_turnover.prenom, rank_turnover.poste, rank_turnover.turnover
FROM (
SELECT YEAR(orders.orderDate) AS annee, MONTH(orders.orderDate) AS mois, employees.employeeNumber AS employer, employees.lastName AS nom, employees.firstName AS prenom, employees.jobTitle AS poste, SUM(orderdetails.quantityOrdered * orderdetails.priceEach) as turnover, row_number() OVER (PARTITION BY MONTH(orders.orderDate)
ORDER BY  SUM(orderdetails.quantityOrdered * orderdetails.priceEach) DESC) as monthly_rank_turnover
FROM employees
JOIN customers ON employees.employeeNumber = customers.salesRepEmployeeNumber
JOIN orders ON orders.customerNumber = customers.customerNumber
JOIN orderdetails ON orderdetails.orderNumber = orders.orderNumber
WHERE YEAR(orders.orderDate) = YEAR(CURDATE()) AND orders.status != 'cancelled'
GROUP BY YEAR(orders.orderDate), MONTH(orders.orderDate), employees.employeeNumber, employees.lastName, employees.firstName
ORDER BY MONTH(orders.orderDate), turnover DESC) as rank_turnover
WHERE monthly_rank_turnover <= 2;

# Requête human resources additionnelle

SELECT rank_turnover.annee, rank_turnover.mois, rank_turnover.employer, rank_turnover.nom, rank_turnover.prenom, rank_turnover.poste, rank_turnover.turnover, monthly_rank_turnover
FROM (
SELECT YEAR(orders.orderDate) AS annee, MONTH(orders.orderDate) AS mois, employees.employeeNumber AS employer, employees.lastName AS nom, employees.firstName AS prenom, employees.jobTitle AS poste, SUM(orderdetails.quantityOrdered * orderdetails.priceEach) as turnover, row_number() OVER (PARTITION BY MONTH(orders.orderDate)
ORDER BY  SUM(orderdetails.quantityOrdered * orderdetails.priceEach) ASC) as monthly_rank_turnover
FROM employees
JOIN customers ON employees.employeeNumber = customers.salesRepEmployeeNumber
JOIN orders ON orders.customerNumber = customers.customerNumber
LEFT JOIN orderdetails ON orderdetails.orderNumber = orders.orderNumber
WHERE YEAR(orders.orderDate) = YEAR(CURDATE()) AND orders.status != 'cancelled'
GROUP BY YEAR(orders.orderDate), MONTH(orders.orderDate), employees.employeeNumber, employees.lastName, employees.firstName
ORDER BY MONTH(orders.orderDate), turnover DESC) as rank_turnover
WHERE monthly_rank_turnover <= 2;
