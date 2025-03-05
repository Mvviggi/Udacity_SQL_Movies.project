# Udacity_SQL_Movies.project
Udacity Nanodegree: Programming for Data Science

## Project: Investigate a Relational Database
### The **Sakila Movie database** is a SQL database of online DVD rentals. This project purpose is to answer business questions about business decisions.

Questions- 
1. What movies families are watching?
2. What are the percentile by Categories of family movies?
3. What is the range of paying customers per store?
4. Find the total rentals by month for 2005 for each store.

**Question 1** - query shown in graph for slide 1*/

```
SELECT 
  Category_name, 
  SUM(count_rentals) total_category 
FROM 
  (
    SELECT 
      f.title Film_title, 
      c.name Category_name, 
      COUNT(r.rental_id) Count_rentals 
    FROM 
      film f 
      JOIN film_category t ON f.film_id = t.film_id 
      JOIN category c ON c.category_id = t.category_id 
      JOIN inventory i ON i.film_id = f.film_id 
      JOIN rental r ON r.inventory_id = i.inventory_id 
    WHERE 
      c.name IN (
        'Animation', 'Children', 'Classics', 
        'Comedy', 'Family', 'Music'
      ) 
    GROUP BY 
      Film_title, 
    Category_name
  ) t1 
GROUP BY 
  Category_name;
```
  
**Question 2 **- query for slide 2*/

```
SELECT 
  Category_name, 
  standard_quartile, 
  COUNT (*) 
FROM 
  (
    SELECT 
      f.title film_title, 
      c.name Category_name, 
      rental_duration, 
      NTILE(4) OVER(
        ORDER BY 
          rental_duration
      ) AS standard_quartile 
    FROM 
      film f 
      JOIN film_category fc ON f.film_id = fc.film_id 
      JOIN category c ON fc.category_id = c.category_id 
    WHERE 
      c.name IN (
        'Animation', 'Children', 'Classics', 
        'Comedy', 'Family', 'Music'
      )
  ) sub 
GROUP BY 
  Category_name, 
  standard_quartile 
ORDER BY 
  1, 
  2;
```

**Question 3** - query for slide 3*/ 


```
SELECT 
  COUNT(t2.customer_level), 
  t2.store_id, 
  t2.customer_level 
FROM 
  (
    SELECT 
      t1.store_id, 
      t1.customer_id, 
      t1.full_name, 
      SUM(p.amount) AS total_by_customer, 
      CASE WHEN SUM(p.amount) >= 100 THEN 'top' WHEN SUM(p.amount) >= 50 
      AND SUM(p.amount) < 100 THEN 'middle' ELSE 'low' END AS customer_level 
    FROM 
      (
        SELECT 
          store_id, 
          customer_id, 
          first_name, 
          last_name, 
          first_name || ' ' || last_name AS full_name 
        FROM 
          customer 
        WHERE 
          active = 1
      ) t1 
      JOIN payment p ON t1.customer_id = p.customer_id 
    GROUP BY 
      t1.store_id, 
      t1.customer_id, 
      t1.full_name 
    ORDER BY 
      4 DESC
  ) t2 
GROUP BY 
  t2.customer_level, 
  t2.store_id;
```

**Question 4** - query for slide 4*/

```
SELECT 
  i.store_id, 
  DATE_PART ('year', r.rental_date) AS rental_year, 
  DATE_PART ('month', r.rental_date) AS rental_month, 
  COUNT(r.rental_id) AS count_rentals 
FROM 
  inventory i 
  JOIN rental r ON i.inventory_id = r.inventory_id 
WHERE 
  DATE_PART ('year', r.rental_date) IN ('2005') 
GROUP BY 
  i.store_id, 
  DATE_PART ('year', r.rental_date), 
  DATE_PART ('month', r.rental_date) 
ORDER BY 
  4 DESC;
```





