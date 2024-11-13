# MAKE-MY-TRIP
Hi everyone, I'm sharing my data analysis project on hotel and city data from MakeMyTrip. I analyzed 580 hotel records using SQL, focusing on factors like hotel price, tax, reviews, and revenue. I used MySQL Server and Workbench for analysis and visualization. The goal was to identify key business insights and optimize operations. 

This is the SQL Query i have written for all the business Questions

create database make_my_trip;
use make_my_trip;

 -- 1. Total Number of Hotels in Each City
	SELECT c.City, COUNT(*) AS Number_of_Hotels
	FROM hotels h join city c on h.city_code = c.city_code
    GROUP BY c.City;
    
-- 2. Number of Hotels by Star Rating in each city
SELECT  Star_Rating,  COUNT(*) AS Num_Hotels
FROM hotels h join city c on h.city_code = c.city_code
GROUP BY   Star_Rating;

-- 3. number of hotels by description in each city
SELECT  c.city, h.rating_description,  COUNT(*) AS Num_Hotels
FROM hotels h join city c on h.city_code = c.city_code
GROUP BY   c.city, h.rating_description;

-- 4. Average Price of Hotels in each city
   SELECT  c.city, AVG(h.price) AS Avg_Price
   FROM hotels h join city c on h.city_code = c.city_code
   group by c.city;

-- 5. Top 5 Most Expensive Hotels in big 4 metropolitan city
   with expensive as (SELECT h.Hotel_Name, h.Price, c.city,
   dense_rank() over(partition by city order by h.price desc) as most_expensive
   FROM hotels h join city c on h.city_code = c.city_code
   where c.city regexp "^[cdmk]")
   select Hotel_Name, Price, city, most_expensive
   from expensive
   where most_expensive <=5;
   
   -- 6. Top 5 best Hotels in big 4 metropolitan city
   with best as (SELECT h.Hotel_Name, h.rating, c.city,
   dense_rank() over(partition by city order by h.rating desc) as best_rating
   FROM hotels h join city c on h.city_code = c.city_code
   where c.city regexp "^[cdmk]")
   select Hotel_Name, rating, city, best_rating
   from best
   where best_rating <=5;
   
   -- 7.no_of_Cheapest Hotel in Each City
      with affordable as ( SELECT c.City, h.Hotel_Name, MIN(h.Price) over(partition by c.city) AS Cheapest_Hotel
      FROM hotels h join city c on h.city_code = c.city_code)
      select city,  count(Cheapest_Hotel) as no_of_hotels
      from affordable
	  group by city ;

-- 8. Calculate Total Revenue for Each City (Price + Tax)
     SELECT c.City, SUM(h.Price + h.Tax) AS Total_Revenue
     FROM hotels h join city c on h.city_code = c.city_code
     where h.tax not like "null"
     GROUP BY c.City;
     
     -- 9. ranking based on revenue
	 with positions as ( SELECT c.City, SUM(h.Price + h.Tax) AS Total_Revenue
     FROM hotels h join city c on h.city_code = c.city_code
     where h.tax not like "null"
     GROUP BY c.City)
     select    city, total_revenue, 
	 rank() over(order by total_revenue desc) as position
     from positions;

-- 10. Hotels with Tax Information Missing
   SELECT h.Hotel_Name, c.City, h.Price, h.Tax
   FROM hotels h join city c on h.city_code = c.city_code
   WHERE h.Tax = "NULL";

  -- 11.  Find the hotel with the highest number of reviews in each city
WITH Hotel_Reviews AS (
    SELECT H.Hotel_Name, C.CITY, H.Reviews,
           ROW_NUMBER() OVER (PARTITION BY C.CITY ORDER BY H.Reviews DESC) AS POSITION
    FROM hotels H JOIN CITY C ON H.city_code = C.city_code)
SELECT Hotel_Name, CITY, Reviews
FROM Hotel_Reviews
WHERE POSITION = 1;

-- 12.  Retrieve the top 3 most reviewed hotels in each city.
WITH Hotel_Review_Rank AS (
    SELECT H.Hotel_Name, C.CITY, H.Reviews,
           RANK() OVER (PARTITION BY C.CITY ORDER BY H.Reviews asc) AS Review_Rank
    FROM hotels H JOIN CITY C ON H.city_code = C.city_code)
SELECT Hotel_Name, CITY, Reviews
FROM Hotel_Review_Rank
WHERE Review_Rank <= 3;

-- 13. Hotels with Rating Greater Than 4.0 in Bangalore,Chennai, hyderabad 
   SELECT h.Hotel_Name, h.Rating, c.city
   FROM hotels h join city c on h.city_code = c.city_code
   WHERE Rating > 4.0 AND c.city regexp "^[bch]";

-- 14. List of Hotels with a 5-Star Rating in each city
   with `5-Star` as (SELECT h.Hotel_Name, c.City, h.Star_Rating, h.Price
   FROM hotels h join city c on h.city_code = c.city_code
   WHERE h.Star_Rating = 5.0)
   select city, count(hotel_name) as no_of_hotels
   from `5-Star`
   group by city;

-- 15. Find Hotels with Rating Above the City Average in kolkata
   SELECT h.Hotel_Name, c.City, h.Rating
   FROM hotels h join city c on h.city_code = c.city_code
   WHERE c.city = "kolkata" and Rating > (
     SELECT  AVG(h.Rating) as city_avg
     FROM hotels h join city c on h.city_code = c.city_code
     where c.city = "kolkata");

-- 16. Calculate Price Range (Min and Max Price) for Each City
   SELECT c.City, MIN(h.Price) AS Min_Price, MAX(h.Price) AS Max_Price
   FROM hotels h join city c on h.city_code = c.city_code
   GROUP BY c.City;

-- 17.  List hotels whose prices are between the minimum and maximum price in their city.
SELECT H.Hotel_Name, H.Price, C.CITY
FROM hotels H JOIN CITY C ON H.city_code = C.city_code
WHERE H.Price BETWEEN (
    SELECT MIN(Price) 
    FROM hotels 
    WHERE city_code = H.city_code) AND 
    (SELECT MAX(Price) 
    FROM hotels 
    WHERE city_code = H.city_code);
   
-- 18. Rank Hotels by Price in Each City Using Window Functions
  SELECT h.Hotel_Name, c.City,h. Price,
  RANK() OVER (PARTITION BY c.City ORDER BY h.Price DESC) AS Price_Rank
  FROM hotels h join city c on h.city_code = c.city_code;
  
-- 19. Calculate the Percentage of 5-Star Hotels in Each City
   SELECT c.City, 
          100 * SUM(CASE WHEN h.Star_Rating = 5 THEN 1 ELSE 0 END) / COUNT(*) AS Percentage_5_Star
   FROM hotels h join city c on h.city_code = c.city_code
   GROUP BY c.City;

-- 20. Calculate the Average Price of Hotels Grouped by Rating Description
   SELECT c.city, h.Rating_Description, AVG(h.Price) AS Avg_Price
   FROM hotels h join city c on h.city_code = c.city_code
   GROUP BY c.city, h.Rating_Description;


