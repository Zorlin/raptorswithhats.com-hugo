---
title: "Hacking My Life: Day 2"
date: 2014-12-25T23:55:00-08:00
Categories: [hml]
Tags: [khanacademy, sql, programming]
---

Really, this post should be titled *Hacking My Life: Day 2 - In Which Benjamin Does a Thing*.

Had an excellent Christmas with friends and family, so I've been busy most of the day. I did get to lug my Chromebook to and from a lunch/dinner party and was thus able to work on learning SQL - I did the "Hour of Databases" on Khan Academy.

<!-- more -->
This is what I ended up submitting for the project at the end. I'm fairly happy with it - I was amazed at the stuff I forgot when I did a first pass. It'll be interesting to see how it's evaluated. KA appears to use students to eval other students, which helps scalability but probably isn't even close to 1:1.

Anyways, without further ado:

{{< highlight sql >}}

CREATE TABLE customers (id INTEGER primary key, name TEXT, age INTEGER, email TEXT, signup_date TEXT, signup_time TEXT, membership_level INTEGER);


INSERT INTO customers VALUES(1, "Bob", 24, "bob@thebobs.com", "2014-12-10", "04:04:24", "0");
INSERT INTO customers VALUES(2, "Jane", 27, "jane@krakowski.com", "2014-04-10", "14:21:01", "2");
INSERT INTO customers VALUES(3, "Jay", 16, "jay@silentbob.com", "2014-08-11", "01:04:14", "1");

SELECT * FROM customers;

CREATE TABLE inventory (id INTEGER primary key, quantity INTEGER NOT NULL, name TEXT, price REAL, manufacturer TEXT, warranty TEXT, shipping_cost REAL, description TEXT);

INSERT INTO inventory VALUES (1, 10, "Nexus 5 32GB", 349.99, "LG", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome phone");
INSERT INTO inventory VALUES (2, 2, "Nexus 10 16GB", 499.99, "Samsung", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome tablet");
INSERT INTO inventory VALUES (3, 23, "Nexus 7 16GB (2012)", 199.99, "ASUS", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome tablet");
INSERT INTO inventory VALUES (4, 36, "Nexus 7 16GB (2013)", 229.99, "ASUS", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome tablet");
INSERT INTO inventory VALUES (5, 12, "Nexus 7 32GB (2012)", 249.99, "ASUS", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome tablet");
INSERT INTO inventory VALUES (6, 14, "Nexus 7 32GB (2013)", 269.99, "LG", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome tablet");
INSERT INTO inventory VALUES (7, 10, "Nexus 4 16GB", 199.99, "LG", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome phone");
INSERT INTO inventory VALUES (8, 10, "Nexus 4 32GB", 249.99, "LG", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome phone");
INSERT INTO inventory VALUES (9, 10, "Galaxy Nexus 16GB", 349.99, "Samsung", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome phone");
INSERT INTO inventory VALUES (10, 10, "Galaxy Nexus 32GB", 399.99, "Samsung", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome phone");
INSERT INTO inventory VALUES (11, 10, "Nexus S 16GB", 529.99, "Samsung", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome phone");
INSERT INTO inventory VALUES (12, 10, "Nexus One 512MB", 629.99, "HTC", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesomeish phone");
INSERT INTO inventory VALUES (13, 10, "TouchPad 16GB", 199.99, "HP", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome tablet on a firesale");
INSERT INTO inventory VALUES (14, 10, "TouchPad 32GB", 249.99, "HP", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome tablet on a firesale");
INSERT INTO inventory VALUES (15, 10, "iPhone 6 16GB", 599.99, "Apple", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome phone with a restrictive OS");
INSERT INTO inventory VALUES (16, 10, "iPhone 6 64GB", 649.99, "Apple", "1 year parts, 1 year labour, with cross-shipping", 0.00, "An awesome phone with a restrictive OS");

SELECT * FROM inventory;

CREATE TABLE customer_orders (id INTEGER primary key, items TEXT, quantities TEXT, order_date TEXT, order_time TEXT, status TEXT NOT NULL, ship_date TEXT, ship_time TEXT, tracking_number TEXT, FOREIGN KEY (id) REFERENCES customers(id));

INSERT INTO customer_orders VALUES (1, "1", "2", "2014-11-24", "14:10:49", "shipped", "2014-11-25", "16:04:10", null);
INSERT INTO customer_orders VALUES (2, "3,4", "1,4", "199.99, 229.99", "2014-11-21", "14:10:49", "pending", null, null, null);

SELECT * FROM customer_orders;

/* Now use select statements to order our items by price */
SELECT price FROM inventory ORDER BY price asc;

/* Gather stats on at least one aspect of our data */
SELECT id FROM customer_orders ORDER BY id DESC LIMIT 1; /* How many orders? */
SELECT id FROM customers ORDER BY id DESC LIMIT 1; /* How many customers? */
SELECT id FROM inventory ORDER BY id DESC LIMIT 1; /* How many items in our inventory? */

{{< / highlight >}}

*This post is part of a series. If you're interested in reading the rest of it, click [here](/categories/hml/).*
