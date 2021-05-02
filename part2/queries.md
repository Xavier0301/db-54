### 1st Query of the number of collisions per year.
```SQL
SELECT EXTRACT(YEAR FROM c.collision_date) as year, COUNT(*) as number_of_collisions
FROM Collisions c 
GROUP BY EXTRACT(YEAR FROM c.collision_date) 
```

### 2nd Query of Most Popular Vehicle Make
```SQL
WITH MostPopularVehicle(number_of_vehicles, brand_id) as SELECT COUNT(p.id) as number_of_vehicles, v.vehicle_make as brand_id
			     FROM Parties p, Vehicles v
			     WHERE p.id = v.party_id GROUP BY v.vehicle_make
			     ORDER BY number_of_vehicles desc
			     limit 1;
			     

			     
SELECT vm.description as most_popular , vp.number_of_vehicles 
FROM   VehiculeMake vm , MostPopularVehicle vp 
WHERE  vm.id = vp.brand_id
```

### 3rd Query of the fraction of total collisions that happened under dark lighting conditions
```SQL
SELECT COUNT(darkc.*)*1.0/COUNT(c.*)
FROM   Collisions c , Collisions darkc , Factors f , Lighting l
WHERE  darkc.case_id = f.case_id AND 
       f.lighting = l.id	  AND
       l.description LIKE '%dark'
```

### 4rth Query of the number of collisions that have occurred under snowy weather conditions
```SQL
SELECT COUNT(snowy.*)
FROM Collisions snowy , Factors f , Weather w 
WHERE snowy.case_id = f.case_id AND
      (f.weather_1 = w.id OR
      f.weather_2 = w.id) AND
      w.description LIKE '%snow' 
```

### 5th Query the number of collisions per day of the week, and find the day that witnessed the highest number of collisions. List the day along with the number of collisions.
```SQL
SELECT FORMAT(c.collision_date,'dddd') as day , COUNT(*) as count1 
FROM   Collisions c 
GROUP BY day
ORDER BY count1 desc
limit 1
```

### 6th Query
```SQL
SELECT weather, SUM(counts) as counts FROM
(SELECT f.weather_1 as weather, COUNT(*) as counts
FROM Factors f
GROUP BY f.weather_1
UNION ALL
SELECT f.weather_2 as weather, COUNT(*) as counts
FROM Factors f
GROUP BY f.weather_2) AS WeatherList
GROUP BY weather
ORDER BY counts DESC
```

### 7th Query
```SQL
SELECT P.at_fault 
FROM Parties P
WHERE P.at_fault = 1 AND P.financial_responsibility = ‘Y’ AND 
```

### 8th Query
```SQL
SELECT AVG(V.victim_age)
FROM Victims V

SELECT
  victim_seating_position
FROM
  VICTIMS
GROUP BY
  victim_seating_position
ORDER BY
  COUNT(*) DESC
limit
  1
```

### 9th Query
```SQL
SELECT ((SELECT V.victim_safety_equipment_1, V.victim_safety_equipment_2, COUNT(*)
FROM Victims V, VictimSafetyEquipment S
WHERE (V.victim_safety_equipment_1 = S.id OR V.victim_safety_equipment_2 = S.id)
AND S.description LIKE '%Lap Belt Used%') / 
(SELECT DISTINCT P.id, COUNT(*)
FROM Parties P))
```

### 10th Query
```SQL
SELECT
  CASE WHEN collision_time BETWEEN '00:00:00'
  AND '00:59:00' THEN '0' WHEN collision_time BETWEEN '01:00:00'
  AND '01:59:00' THEN '1' WHEN collision_time BETWEEN '02:00:00'
  AND '02:59:00' THEN '2' WHEN collision_time BETWEEN '03:00:00'
  AND '03:59:00' THEN '3' WHEN collision_time BETWEEN '04:00:00'
  AND '04:59:00' THEN '4' WHEN collision_time BETWEEN '05:00:00'
  AND '05:59:00' THEN '5' WHEN collision_time BETWEEN '06:00:00'
  AND '06:59:00' THEN '6' WHEN collision_time BETWEEN '07:00:00'
  AND '07:59:00' THEN '7' WHEN collision_time BETWEEN '08:00:00'
  AND '08:59:00' THEN '8' WHEN collision_time BETWEEN '09:00:00'
  AND '09:59:00' THEN '9' WHEN collision_time BETWEEN '10:00:00'
  AND '10:59:00' THEN '10' WHEN collision_time BETWEEN '11:00:00'
  AND '11:59:00' THEN '11' WHEN collision_time BETWEEN '12:00:00'
  AND '12:59:00' THEN '12' WHEN collision_time BETWEEN '13:00:00'
  AND '13:59:00' THEN '13' WHEN collision_time BETWEEN '14:00:00'
  AND '14:59:00' THEN '14' WHEN collision_time BETWEEN '15:00:00'
  AND '15:59:00' THEN '15' WHEN collision_time BETWEEN '16:00:00'
  AND '16:59:00' THEN '16' WHEN collision_time BETWEEN '17:00:00'
  AND '17:59:00' THEN '17' WHEN collision_time BETWEEN '18:00:00'
  AND '18:59:00' THEN '18' WHEN collision_time BETWEEN '19:00:00'
  AND '19:59:00' THEN '19' WHEN collision_time BETWEEN '20:00:00'
  AND '20:59:00' THEN '20' WHEN collision_time BETWEEN '21:00:00'
  AND '21:59:00' THEN '21' WHEN collision_time BETWEEN '22:00:00'
  AND '22:59:00' THEN '22' ELSE '23' END AS `Range`,
  count(1) as `Count`
from
  Collisions
group by
  `Range`;
```
