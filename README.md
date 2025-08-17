USE music;
select * from employee;

#Q1. Who is the senior most employee based on job title ?
select * from employee
order by levels desc
limit 1;

#Adam is the senior most employee based on the job title. 

select * from invoice; 
#Q2. which country have the most invoices?
select count(*) as c , billing_country
from invoice
group by billing_country
order by c desc;

# which city had the best cutomers? we would like to throw a promotional music festival in the city we made the most money. write a query that returns one city that has the highest sum of invoice totals. return both the city name & sum of all invoice total.

select SUM(total) as invoice_total, billing_city
from invoice 
group by billing_city
order by invoice_total desc
Limit 3;


#Q3. what are top 3 values of total invoices?
select total from invoice
order by total desc 
limit 3;

select * from invoice; 
select * from customer; 
#Who is the best customer? the customer who has spent the most money will be declared the best cutomer . write a query that returns the person who has spent the most money. 
select customer.customer_id, customer.first_name, customer.last_name, 
SUM(invoice.total) as total 
from customer
JOIN invoice ON customer.customer_id = invoice.customer_id
group by customer.customer_id
order by total desc
limit 1;

SELECT 
    customer.customer_id, 
    customer.first_name, 
    customer.last_name, 
    SUM(invoice.total) AS total 
FROM 
    customer
JOIN 
    invoice ON customer.customer_id = invoice.customer_id
GROUP BY 
    customer.customer_id
ORDER BY 
    invoice.total DESC
LIMIT 1;


#Write query to return the Email , first name, last name & genre of all rock music listeners . return your list ordered alphabetically by email strating with A. 
select distinct email,first_name, last_name
from  customer
join invoice on customer.customer_id= invoice.customer_id
join invoice_line on invoice.invoice_id = invoice_line.invoice_id
where track_id in (
	select track_id from track
	join genre on track.genre_id = genre.genre_id
	where genre.name like 'ROCK'
)
order by email;

#Lets invite the artists who have written the most rock music in our dataset.write a query that returns the artist name and total track count of the top 10 rock bands. 
use music;
SELECT 
    artist.artist_id, 
    artist.name, 
    COUNT(track.track_id) AS number_of_songs
FROM track
JOIN album ON album.album_id = track.album_id
JOIN artist ON artist.artist_id = album.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id, artist.name
ORDER BY number_of_songs DESC
LIMIT 10;

#3 Return all the track names that have a song length than the average song length,return the name and milliseconds for each track, order by the song length with the longest songs listed first.

select name,milliseconds 
from track
where milliseconds > (
	select avg(milliseconds) as avg_track_length
    from track)
order by milliseconds desc;

#Advance
#1 . Find how much amount spent by each customer on artist? write a query to return customer name, artist name and total spent.

WITH best_selling_artist AS (
	SELECT artist.artist_id AS artist_id, artist.name AS artist_name, SUM(invoice_line.unit_price*invoice_line.quantity) AS total_sales
	FROM invoice_line
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN album ON album.album_id = track.album_id
	JOIN artist ON artist.artist_id = album.artist_id
	GROUP BY 1
	ORDER BY 3 DESC
	LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price*il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY 1,2,3,4
ORDER BY 5 DESC;




#2 we want to find out the most popular music genre for the each country. we determine the most popular genre as the genre with the highest amount of purchase . write a query that return each country with top genre. for countries where the maximum number of purchase return is shared return all genres. 
#WITH popular_genre AS
USE music;
WITH popular_genre AS 
(
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
	JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN customer ON customer.customer_id = invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY 2,3,4
	ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1;

#3. Write a query that determines the customer that has spent the most on music for each country. Write a query that return the country along with the top customer and how much they spent. for countries where the top amount spent is shared, provide all customer who spent this amount.

WITH RESCURSIVE 
	Customer_with_country AS(
    SELECT customer.customer_id, first_name, last_name, billing_country, SUM (total) AS total_spending
    FROM invoice
    JOIN customer ON customer.customer_id = invoice.customer_id
    GROUP BY 1,2,3,4
    ORDER BY 2,3 DESC),
    
    country_max_spending AS(
    SELECT billing_country, MAX(total_spending) AS max_spending
    FROM customer_with_country
    GROUP BY billing_country)
    
SELECT cc.billing_country, cc.total_spending, cc.first_name, cc.last_name, cc.customer_id
FROM customer_with_country cc
JOIN country_max_spending ms
ON cc.billing_country = ms.billing_country
WHERE cc.total_spending = ms.max_spending
ORDER  BY 1;
    

