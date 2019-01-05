-- Querying york-river bookseller's database project

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Query 1 : List customers by name along with category and language such that the customer has bought all the books offerred in that category / language group and there is more than one book in that category / language group. Do not
have any duplicates. Order by name + category + language.

Show the (distinct) name, category, and language.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

WITH books_in_all_cat (title,year,cat,language) as (SELECT b1.title,b1.year,b1.cat, b1.language
FROM yrb_book as b1, yrb_book as b2 
WHERE b1.cat=b2.cat),
more_than_one_book_in_that_cat (cat) as (SELECT cat FROM books_in_all_cat GROUP BY cat HAVING count(cat) > 1),
more_than_one_book_in_that_lan (language) as (SELECT language FROM books_in_all_cat group by language having count(language) > 1),
more_than_one_book_and_language_cat (title,year,language,cat) as (SELECT distinct title,year,language ,cat FROM books_in_all_cat as a WHERE cat IN (SELECT cat FROM more_than_one_book_in_that_cat) and language IN (SELECT language
FROM more_than_one_book_in_that_lan))
SELECT distinct c.name, b.cat as category, b.language 
FROM yrb_customer as c, yrb_purchase as p, yrb_book as b
WHERE c.name = 'Ekksdwl Qjksynn' and b.cat = 'science' and language = 'Plutonian';

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Query 2 : For each customer, find the total cost he or she has paid for books in each category. You don't need to consider qnty for this query, you can assume that qnty for every purchase is 1

Show the customers's name, the category and the cost.   
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

WITH F(cid, cat, sum_of_price) as (SELECT p.cid, b.cat, sum(o.price)
FROM yrb_book as b, yrb_offer as o, yrb_purchase as p
WHERE b.title=o.title and o.title=p.title and b.year=o.year and o.year=p.year and b.year=p.year and b.title=p.title and o.club=p.club
GROUP BY p.cid, b.cat),
S(cid, name,cat,cost) as (SELECT c.cid,c.name,s.cat, s.sum_of_price
FROM yrb_customer as c, F as s
WHERE c.cid=s.cid)
SELECT name, cat, cost
FROM S;

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Query 3 : Suppose all books shipped to the customer in purchases made on the same time (when) are shipped as one package. Remember, that customers are billed for the books and the postage.

For each customer, how much do they spend on each package? If the weight of one package is X grams, the entry just higher than (or equal to) X is found in the shipping table and the associated shipping price is the postage for
this package.
Show the customer's name, the day of the package and the cost.  
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

WITH First_table(cid, club, title, year, when, fullprice, fullweight) AS
(SELECT yrb_purchase.cid, yrb_purchase.club, yrb_purchase.title, yrb_purchase.year, yrb_purchase.when, qnty*price as fullprice, qnty*weight as fullweight
FROM yrb_offer, yrb_purchase, yrb_book
WHERE yrb_purchase.club = yrb_offer.club
AND yrb_purchase.title = yrb_offer.title
AND yrb_purchase.year = yrb_offer.year
AND yrb_purchase.title = yrb_book.title
AND yrb_purchase.year = yrb_book.year
AND yrb_book.title = yrb_offer.title
AND yrb_book.year = yrb_offer.year),
Second_table(cid, day, bookcost, bookweight) AS
(SELECT First_table.cid, cast (when as date) as day, SUM(First_table.fullprice) as bookcost, SUM(First_table.fullweight) as bookweight
FROM First_table
GROUP BY cid, when),
Third_table(cid, day, bookcost, bookweight, shipweight) AS
(SELECT cid, day, bookcost, bookweight, MIN(weight) as shipweight
FROM Second_table, yrb_shipping
WHERE weight >= bookweight
GROUP BY cid, day, bookcost, bookweight)
SELECT cid, day, bookcost + cost as cost
FROM Third_table, yrb_shipping
WHERE Third_table.shipweight = yrb_shipping.weight
ORDER BY cid;

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Query 4 : Show the books of which two copies have been bought by one customer over time.

Show the title and year.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

select title,year
from yrb_purchase 
group by title,year,cid
having sum(qnty) = 2; 

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Query 5 : Find the books which offered by exactly eight clubs

Show the title and year.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

SELECT title,year
from (
      SELECT title,year,club
      FROM yrb_offer 
      GROUP BY title,year,club)
group by title,year
having count(club) = 8;      

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Query 6 : Select the books, which are more expensive than the average price (across all clubs) of the most expensive books (for each club) written in English. If a club doesn't have books written in English, do not count.

Show the title, the year and the price of the book.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

WITH max_Price_Of_Each_Club(club, Price) as (select o.club, max(o.price)
from yrb_offer as o, yrb_book as b
where o.title=b.title and o.year=b.year and b.language = 'English'
GROUP BY o.club),
average_Of_Expensive_Books (avgPrice) as (SELECT avg(Price) 
FROM max_Price_Of_Each_Club)
SELECT o.title, o.year, o.price
FROM yrb_offer as o, average_Of_Expensive_Books as a 
WHERE o.price > a.avgPrice;    

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Query 7 : Which club or clubs have the least members?

Show the club(s) and the number.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

WITH First_table(club,one) AS
(SELECT club, count(*) as one
FROM yrb_member
GROUP BY club),
Second_table(min) AS
(SELECT MIN(one) as min
FROM First_table)
SELECT club, min AS count
FROM First_table, Second_table
WHERE min = one;     

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Query 8 : For each customer, show the category he or she spent more on the books of that category than on books of any other category. You do not need to consider qnty for this query, that is, you can assume that qnty for every
purchase is 1.

Show customer name, the category and the cost.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

WITH sort(cid, cat, sum_of_price) as (SELECT p.cid, b.cat, sum(o.price)
FROM yrb_book as b, yrb_offer as o, yrb_purchase as p
WHERE b.title=o.title and o.title=p.title and b.year=o.year and o.year=p.year and b.year=p.year and b.title=p.title and o.club=p.club
GROUP BY p.cid, b.cat),
n_sort(cid,name,cat,cost) as (SELECT s.cid, c.name,s.cat, s.sum_of_price
FROM yrb_customer as c, sort as s
WHERE c.cid=s.cid),
max_p(cid, name,cost) as (SELECT cid, name, max(cost) FROM n_sort GROUP BY cid, name )
SELECT m.name,n.cat, m.cost
FROM max_p as m, n_sort as n
WHERE m.name = n.name and m.cost=n.cost
order by n.cid;

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Query 9 : Show the customers who made a purchase before January 1 1998.

Show customer’s name and the date (not timestamp!) of purchase.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

SELECT c.name as Name, cast(p.when as date) as Date
FROM yrb_purchase as p, yrb_customer as c
WHERE c.cid=p.cid and cast(p.when as date) < cast('1998-01-01' as date);  

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Query 10 : Which pairs of customers purchased at least one book in common?

List distinct pairs of customers (by name). For each pair of customers, show first the one with the larger CID; name the two columns namea and nameb.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


SELECT distinct pc1.name as namea,pc2.name as nameb
FROM (SELECT c.name as name ,p.cid as cid ,p.title as title, p.year as year
      FROM yrb_purchase as p, yrb_customer as c
      WHERE p.cid=c.cid and p.qnty >=1
      ORDER by c.cid desc) as pc1,
     (SELECT c.name as name,p.cid as cid,p.title as title, p.year as year
      FROM yrb_purchase as p, yrb_customer as c
      WHERE p.cid=c.cid and p.qnty >=1
      ORDER by c.cid desc) as pc2
WHERE pc1.year=pc2.year and pc1.title = pc2.title and pc1.cid > pc2.cid;

