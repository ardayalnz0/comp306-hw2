# Homework 2

## Question 1
```sh
"C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql.exe" -u root -p < "C:\Users\TR\Downloads\world.sql"
mysql -u root -p
USE world;
SHOW TABLES;
```
![image](https://github.com/user-attachments/assets/5bb57925-4164-4966-806f-93d4a6763059)


## Question 2

### Part A
```sh
SELECT c.Code AS country_code, c.Name AS country_name, ci.Name AS city_name, ci.Population AS city_population
FROM Country c JOIN City ci ON c.Code = ci.CountryCode
WHERE c.LifeExpectancy > 75 AND c.Population < 1000000
ORDER BY ci.Population ASC;
```
![image](https://github.com/user-attachments/assets/25f9acba-4197-4888-9ecc-448a2e5751f3)


### Part B
```sh
SELECT ci.CountryCode, ci.Name AS city_name
FROM City ci JOIN CountryLanguage cl ON ci.CountryCode = cl.CountryCode
WHERE ci.Population < 90000 AND cl.Language = 'German' AND cl.Percentage > 1;
```
![image](https://github.com/user-attachments/assets/0e1f2df8-b184-4dce-8e50-a4015b7e7e42)

## Question 3

### Part A
```sh
SELECT co.Name AS country_name, ci.Name AS capital_city
FROM Country co JOIN City ci ON co.Capital = ci.ID
JOIN 
    (
        SELECT cc.CountryCode, COUNT(*) AS lang_count
        FROM CountryLanguage cc
        GROUP BY CountryCode
    ) cl_count ON co.Code = cl_count.CountryCode
WHERE cl_count.lang_count < 10 AND LEFT(co.Name, 1) IN ('S', 'P', 'A', 'D', 'E');
```
![image](https://github.com/user-attachments/assets/6fd0b975-f520-4c3e-aa75-aa158f6bcf67)
![image](https://github.com/user-attachments/assets/cba5124a-301b-408f-9fcf-05b55e1ec002)

### Part B
```sh
SELECT co.Name AS country_name, COUNT(ci.ID) AS number_of_cities
FROM Country co JOIN City ci ON co.Code = ci.CountryCode
GROUP BY co.Code, co.Name
HAVING COUNT(ci.ID) > 30
ORDER BY number_of_cities DESC;
```
![image](https://github.com/user-attachments/assets/fb262279-6a56-4785-a338-51e43d1653f6)

### Part C

```sh
SELECT co.Name AS country_name, COUNT(cl.Language) AS total_languages
FROM Country co
JOIN CountryLanguage cl ON co.Code = cl.CountryCode
WHERE co.GNP > 80000 AND co.Code IN (
        SELECT CountryCode
        FROM CountryLanguage
        WHERE Language = 'Turkish'
    )
GROUP BY co.Code, co.Name;
```
![image](https://github.com/user-attachments/assets/d12ce5e8-3b29-48cd-8ecb-bddf40e2e22e)

## Question 4

### Part A
```sh
CREATE TABLE restaurant (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    type VARCHAR(50),
    rating INT CHECK (rating BETWEEN 1 AND 5)
);
```
![image](https://github.com/user-attachments/assets/8d7ab047-ac24-49ae-b5d5-baacce81fd80)

### Part B
![image](https://github.com/user-attachments/assets/d1cff261-b7e1-48b3-a586-4ce346d88dc4)

### Part C
```sh
UPDATE restaurant
SET type = 'Vegan'
WHERE name = 'The Vegan Kitchen';
```

![image](https://github.com/user-attachments/assets/d528ff0d-2d13-4cfa-8527-23f7fd9480b1)

### Part D
```sh
DELETE FROM restaurant
WHERE name LIKE '%House%';
```
![image](https://github.com/user-attachments/assets/3feb7211-8316-40a5-a692-4e8cb66eccf2)

## Question 5

### Part A
```sh
CREATE TABLE city_restaurant (
    restaurant_id INT,
    city_id INT,
    PRIMARY KEY (restaurant_id, city_id),
    FOREIGN KEY (restaurant_id) REFERENCES restaurant(id) 
      ON DELETE RESTRICT ON UPDATE CASCADE,
    FOREIGN KEY (city_id) REFERENCES city(ID) 
      ON DELETE RESTRICT ON UPDATE CASCADE
);
```

![image](https://github.com/user-attachments/assets/6faa6442-40f2-4190-a132-6ae87fbeab5a)

### Part B
```sh
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/city_restaurant.txt'
INTO TABLE city_restaurant
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
(restaurant_id, city_id);
```
![image](https://github.com/user-attachments/assets/5e29a427-ba13-4ac0-80d2-5f53926419af)


### Part C
```sh
SELECT cr.city_id, c.Name
FROM city_restaurant cr JOIN city c ON cr.city_id = c.ID
GROUP BY cr.city_id, c.Name
HAVING COUNT(cr.restaurant_id) > 1;
```
![image](https://github.com/user-attachments/assets/500df677-286e-429e-ac65-2b80c738e238)

### Part D
```sh
SELECT r.id, r.name
FROM  restaurant r JOIN city_restaurant cr1 ON r.id = cr1.restaurant_id JOIN city c1 ON cr1.city_id = c1.ID
WHERE c1.Name = 'Manchester'AND r.id NOT IN (
        SELECT cr2.restaurant_id
        FROM city_restaurant cr2 JOIN city c2 ON cr2.city_id = c2.ID
        WHERE c2.Name = 'Liverpool'
    );
```
![image](https://github.com/user-attachments/assets/402978b3-5f8f-478c-b322-c3525dd5450a)

### Part E
```sh
UPDATE restaurant
SET rating = rating - 1
WHERE id IN (
    SELECT cr.restaurant_id
    FROM city_restaurant cr JOIN city c ON cr.city_id = c.ID
    WHERE c.Name = 'Baku'
)
AND rating > 1;

```
![image](https://github.com/user-attachments/assets/a1395c06-7a28-4d11-a775-223b4ff85314)


### Part F
```sh
SELECT r.id, r.name
FROM restaurant r JOIN city_restaurant cr ON r.id = cr.restaurant_id
JOIN city c ON cr.city_id = c.ID
JOIN country co ON c.CountryCode = co.Code
WHERE co.Name = 'Azerbaijan'
GROUP BY r.id, r.name
HAVING COUNT(DISTINCT c.ID) >= 2;
```
![image](https://github.com/user-attachments/assets/4d48c447-0c74-44d9-bb07-f9aca00df493)

### Part G
```sh
SELECT co.Code, co.Name
FROM country co
WHERE co.Continent = 'Asia' AND co.Region != 'Middle East'
    AND co.Code NOT IN (
        SELECT DISTINCT c.CountryCode
        FROM city_restaurant cr
        JOIN city c ON cr.city_id = c.ID
    );
```
![image](https://github.com/user-attachments/assets/f546bc15-3b29-4386-9be5-c69081fd4f47)

## Question 6
```sh
mysqldump -u root -p world > "C:\Users\TR\Desktop\world-export.sql" // On Terminal
```


