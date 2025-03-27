# Music Store Analysis - SQL Project

# Project Overview

The Music Store Analysis project explores key business insights from a music store database using SQL. The analysis covers employee hierarchy, sales trends, customer behavior, and revenue distribution to drive strategic decisions for the business.

# Objectives

Identify the senior-most employee based on job title.

Determine which countries have the most invoices.

Find the top three invoice values.

Locate the city with the highest revenue for potential promotional events.

Identify the highest-spending customer.

# Database and Tools

Postgre SQL

PgAdmin4

# Dataset

The project uses a relational database that contains multiple tables, including:

Customers: Customer details such as name, location, and email.

Invoices: Sales transactions with invoice dates and total amounts.

Invoice_Items: Detailed breakdown of items purchased in each invoice.

Tracks: Information on music tracks including genre, album, and price.

Albums: Album details linked to tracks.

Artists: Information on music artists.

Genres: Genre classification for each track.

# SQL Queries and Analysis

/* Q1: Who is the senior most employee based on job title? */

select * from employee
order by levels desc
limit 1;

/* Q2: Which countries have the most Invoices? */

select count(*) as total_count,billing_country  from invoice
group by billing_country order by total_count desc; 

/* Q3: What are top 3 values of total invoice? */

select ROUND(total) from invoice 
order by total desc limit 3;

select total from invoice 
order by total desc limit 3

/* Q4: Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. 
Write a query that returns one city that has the highest sum of invoice totals. 
Return both the city name & sum of all invoice totals */


select billing_city,sum(total) as invoice_total 
from invoice group by billing_city order by invoice_total desc limit 1 


/* Q5: Who is the best customer? The customer who has spent the most money will be declared the best customer. 
Write a query that returns the person who has spent the most money.*/


select c.customer_id,c.first_name,c.last_name, sum(i.total) as total 
from customer as c
JOIN
Invoice as i on c.customer_id = i.customer_id 
group by c.customer_id order by total desc limit 1 


/*Q1: Write query to return the email, first name, last name, & Genre of all Rock Music listeners. 
Return your list ordered alphabetically by email starting with A*/

SELECT DISTINCT email,first_name, last_name
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
JOIN invoiceline ON invoice.invoice_id = invoiceline.invoice_id
WHERE track_id IN(
	SELECT track_id FROM track
	JOIN genre ON track.genre_id = genre.genre_id
	WHERE genre.name LIKE 'Rock'
)
ORDER BY email;



SELECT DISTINCT email AS Email,first_name AS FirstName, last_name AS LastName, genre.name AS Name
FROM customer
JOIN invoice ON invoice.customer_id = customer.customer_id
JOIN invoiceline ON invoiceline.invoice_id = invoice.invoice_id
JOIN track ON track.track_id = invoiceline.track_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
ORDER BY email;


/* Q2: Let's invite the artists who have written the most rock music in our dataset. 
Write a query that returns the Artist name and total track count of the top 10 rock bands. */

SELECT artist.artist_id, artist.name,COUNT(artist.artist_id) AS number_of_songs
FROM track
JOIN album ON album.album_id = track.album_id
JOIN artist ON artist.artist_id = album.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id
ORDER BY number_of_songs DESC
LIMIT 10;


/* Q3: Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first. */

SELECT name,miliseconds
FROM track
WHERE miliseconds > (
	SELECT AVG(miliseconds) AS avg_track_length
	FROM track )
ORDER BY miliseconds DESC;


/* Q2: We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre 
with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where 
the maximum number of purchases is shared return all Genres. */


/*Using CTE */

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
SELECT * FROM popular_genre WHERE RowNo <= 1


/* Q3: Write a query that determines the customer that has spent the most on music for each country. 
Write a query that returns the country along with the top customer and how much they spent. 
For countries where the top amount spent is shared, provide all customers who spent this amount. */


/* using CTE */

WITH Customter_with_country AS (
		SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending,
	    ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
		FROM invoice
		JOIN customer ON customer.customer_id = invoice.customer_id
		GROUP BY 1,2,3,4
		ORDER BY 4 ASC,5 DESC)
SELECT * FROM Customter_with_country WHERE RowNo <= 1

# Key Insights & Recommendations

Top Performing Locations: The city generating the highest revenue should be considered for promotional events or special discounts.

Customer Loyalty Program: High-spending customers can be rewarded with exclusive offers to boost retention.

Market Expansion Strategy: Countries with lower invoice counts may benefit from targeted marketing efforts.

Optimized Pricing Strategy: Analyzing top invoice values can help structure pricing models for maximum revenue.


