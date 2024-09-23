# SQL_Music_Store_Analysis

SQL project to analyze online music store dat

# Table of Contents
1. Project Objective
2. Tools Used
3. Tables Include
4. Project Questions
5. Queries
 
# Project Objective
Unleash the full potential of our online music store's dataset with a dynamic SQL analysis, designed to deliver powerful insights that drive business growth and elevate decision-making. 🚀

* Customer Segmentation: Uncover unique customer groups by diving into their purchasing habits and preferences. 🕵️‍♂️
* Sales Trends: Decode sales patterns over time to reveal seasonal rhythms and long-term trends. 📈
* Artist Effectiveness: Gauge how different artists perform in terms of sales and fan engagement. 🎤
* Artist Growth: Track and forecast artists' rise in popularity and sales momentum. 📊
* Customer Interest: Spotlight trending genres, albums, and tracks to get the pulse of customer preferences. 🎵
* Customer Spending: Pinpoint high-value customers and uncover opportunities to boost revenue. 💰
* Operational Insights: Dive into store operations to pinpoint areas ripe for optimization and improvement. 🔍

# Tools Used
PostgreSQL

# Tables Include
* album
* artist
* customer
* employee
* invoice
* genre
* invoice_line
* media_type
* playlist
* playlist_track
* track
  
# Project Questions
Q1: Who is the senior most employee based on job title?

Q2: Which countries have the most Invoices?

Q3: What are top 3 values of total invoice?

Q4: Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. Write a query that returns one city that has the highest sum of invoice totals. Return both the city name & sum of all invoice totals.

Q5: Who is the best customer? The customer who has spent the most money will be declared the best customer.Write a query that returns the person who has spent the most money.

Q6: Write query to return the email, first name, last name, & Genre of all Rock Music listeners.

Q7: Let's invite the artists who have written the most rock music in our dataset.Write a query that returns the Artist name and total track count of the top 10 rock bands.

Q8: Return all the track names that have a song length longer than the average song length.Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first.

Q9: Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent

Q10: Write a query that returns each country along with the top Genre. For countries where the maximum number of purchases is shared return all Genres.

Q11: Write a query that determines the customer that has spent the most on music for each country.Write a query that returns the country along with the top customer and how much they spent. For countries where the top amount spent is shared, provide all customers who spent this amount.

# Queries

select title, first_name,last_name, levels
from employee
order by levels desc
limit 1;

select count(*) as c, billing_country 
from invoice
group by billing_country
order by c desc;

select total from invoice
order by total desc
limit 3;

select sum(total) as invoice_total, billing_city 
from invoice
group by billing_city
order by invoice_total desc;

select c.customer_id, c.first_name, c.last_name, sum(a.total) as total
from customer as c 
join invoice as a on c.customer_id = a.customer_id
group by c.customer_id
order by total desc
limit 1;

select distinct c.email , c.first_name, c.last_name
from customer as c
join invoice as a on c.customer_id = a.customer_id
join invoice_line as b on a.invoice_id = b.invoice_id
where track_id in(
select track_id from track as t
join genre as g on t.genre_id = g.genre_id
where g.name like 'Rock'
)
order by c.email;

select a.artist_id, a.name, count(a.artist_id) as number_of_songs
from track as t
join album as b on t.album_id = b.album_id
join artist as a on a.artist_id = b.artist_id
join genre as g on g.genre_id = t.genre_id
where g.name like 'Rock'
group by a.artist_id
order by number_of_songs desc
limit 10;

SELECT name,milliseconds
FROM track
WHERE milliseconds > (
	SELECT AVG(milliseconds) AS avg_track_length
	FROM track )
ORDER BY milliseconds DESC;

WITH best_selling_artist AS (
	SELECT a.artist_id AS artist_id, a.name AS artist_name, SUM(il.unit_price*il.quantity) AS total_sales 
	FROM invoice_line as il
	JOIN track as t ON t.track_id = il.track_id
	JOIN album as b ON b.album_id = t.album_id
	JOIN artist as a ON a.artist_id = b.artist_id
	GROUP BY 1
	ORDER BY 3 DESC
	LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price*il.quantity) AS amount_spent
FROM invoice i
JOIN customer as c ON c.customer_id = i.customer_id
JOIN invoice_line as il ON il.invoice_id = i.invoice_id
JOIN track as t ON t.track_id = il.track_id
JOIN album as b ON b.album_id = t.album_id
JOIN best_selling_artist as bsa ON bsa.artist_id = b.artist_id
GROUP BY 1,2,3,4
ORDER BY 5 DESC;

WITH popular_genre AS 
(
    SELECT COUNT(il.quantity) AS purchases, c.country, g.name, g.genre_id, 
	ROW_NUMBER() OVER(PARTITION BY c.country ORDER BY COUNT(il.quantity) DESC) AS RowNo 
    FROM invoice_line as il 
	JOIN invoice as i ON il.invoice_id = i.invoice_id
	JOIN customer as c ON c.customer_id = i.customer_id
	JOIN track as t ON t.track_id = il.track_id
	JOIN genre as g ON g.genre_id = t.genre_id
	GROUP BY 2,3,4
	ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1

WITH Customer_with_country AS (
		SELECT c.customer_id,c.first_name,c.last_name,i.billing_country,SUM(i.total) AS total_spending,
	    ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
		FROM invoice as i
		JOIN customer as c ON c.customer_id = i.customer_id
		GROUP BY 1,2,3,4
		ORDER BY 4 ASC,5 DESC)
SELECT * FROM Customer_with_country WHERE RowNo <= 1






Schema - Music Store Database

<img width="594" alt="schema_diagram" src="https://github.com/user-attachments/assets/65af80fb-fd80-4702-abed-4b03db23d520">
