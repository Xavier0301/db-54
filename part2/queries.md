### 1st Query of the number of collisions per year.
```SQL
SELECT EXTRACT(YEAR FROM c.collision_date) as year, COUNT(*) as number_of_collisions
FROM Collisions c 
GROUP BY EXTRACT(YEAR FROM c.collision_date) 
```

### 2nd Query of Most Popular Vehicle Make
```SQL
CREATE VIEW MostPopularVehicle AS
(SELECT COUNT(p.id) as number_of_vehicles, v.vehicle_make as brand_id
			     FROM Parties p, Vehicles v
			     WHERE p.id = v.party_id GROUP BY v.vehicle_make
			     ORDER BY number_of_vehicles desc
			     limit 1)
			     
SELECT vm.description as most_popular , vp.number_of_vehicles 
FROM   VehiculeMake vm , MostPopularVehicle vp 
WHERE  vm.id = vp.brand_id
```

### 3rd Query of the fraction of total collisions that happened under dark lighting conditions
```SQL
SELECT
  (
    SELECT
      COUNT(*)
    FROM
      (
        SELECT
          c.case_id
        FROM
          Collisions c,
          Factors f,
          Lighting l
        WHERE
          c.case_id = f.case_id
          AND f.lighting = l.id
          AND l.description LIKE 'dark%'
      ) AS DarkCols
  ) / (
    SELECT
      COUNT(*)
    FROM
      Collisions
  )
```

### 4rth Query of the number of collisions that have occurred under snowy weather conditions
```SQL
SELECT COUNT(snowy.case_id)
FROM Collisions snowy , Factors f , Weather w 
WHERE snowy.case_id = f.case_id AND
      (f.weather_1 = w.id OR
      f.weather_2 = w.id) AND
      w.description LIKE 'snowing' 
```

### 5th Query the number of collisions per day of the week, and find the day that witnessed the highest number of collisions. List the day along with the number of collisions.

```SQL
SELECT DAYOFWEEK(collision_date) as day , COUNT(*) as counts 
FROM   Collisions
GROUP BY day
ORDER BY counts desc
LIMIT 1
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
SELECT COUNT(*) as count 
FROM Parties p, Factors f
WHERE 
	p.at_fault = 1 
	AND 
	p.financial_responsibility IN 
		(SELECT id FROM FinancialResponsibility WHERE description='Y')
	AND 
	f.case_id = p.case_id
	AND 
		(f.road_condition_1 IN
			(SELECT id FROM RoadCondition WHERE description='loose material') 
		OR
		f.road_condition_2 IN 
			(SELECT id FROM RoadCondition WHERE description='loose material') 
		)
```

### 8th Query
```SQL
SET
  @rowindex: = -1;
SELECT
  AVG(t.age)
FROM
  (
    SELECT
      @rowindex: = @rowindex + 1 AS rowindex,
      V.victim_age AS age
    FROM
      Victims V
    ORDER BY
      V.victim_age
  ) AS t
WHERE
  t.rowindex IN (FLOOR(@rowindex / 2), CEIL(@rowindex / 2));

SELECT
  victim_seating_position
FROM
  Victims
GROUP BY
  victim_seating_position
ORDER BY
  COUNT(*) DESC
limit
  1
```

### 9th Query
```SQL
SELECT
  (
    (
      SELECT
        COUNT(V.case_id)
      FROM
        Victims V,
        VictimSafetyEquipment S
      WHERE
        (
          V.victim_safety_equipment_1 = S.id
          OR V.victim_safety_equipment_2 = S.id
        )
        AND S.description IN ('C', 'E', 'G')
    ) / (
      SELECT
        COUNT(*)
      FROM
        Victims
    )
  )
```

### 10th Query
```SQL
SELECT
  counts /(
    SELECT
      DISTINCT C.case_id,
      COUNT(*)
    FROM
      Collisions C
  )
FROM(
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
      count(1) as counts
    from
      Collisions
    group by
      `Range`
  )
```
