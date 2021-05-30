# Query 2 

The unoptimised version of this query leads the database to only make full scans and hash joins.
* The cumulative cost for the hash joins is 4.5e12
* The cumulative cost for the full scans is 92000

We optimise the query by adding multiple indexes:
* On `case_id` of `Collisions`
* On `case_id` of `Parties`
* On `party_id` of `Victims`
* On `id` of `VehicleMake`

* The cumulative cost for the joins is now 300000, much lower as the database uses nested loops that take advantages of the indexes.
* The cumulative cost the the scans is 19000: using an index makes a difference compared to a full scan.

The runtime of the query is:
* Unoptimised: 14s700ms
* Optimised: 1s337ms

```SQL
SELECT vm.description , COUNT(vm.id)
FROM(
    SELECT p.id
    FROM Collisions c USE INDEX (), Factors f USE INDEX (), RoadCondition r USE INDEX (), Parties p USE INDEX ()
    WHERE c.case_id = f.case_id AND(
        r.id = f.road_condition_1 OR
        r.id = f.road_condition_2) AND
        r.description = 'holes' AND
        c.case_id = p.case_id ) AS parties_holes , Vehicles v USE INDEX (), VehiculeMake vm USE INDEX ()
WHERE parties_holes.id = v.party_id AND
      vm.id = v.vehicle_make
GROUP BY vm.description
ORDER BY COUNT(vm.id) DESC
LIMIT 5
```

# Query 3

The unoptimised version of this query leads the database to only make full scans and hash joins.
* The cumulative cost for the hash joins is 7e12
* The cumulative cost for the full scans is 52000

We optimise the query by adding multiple indexes:
* On `collision_severity` of `Collisions`
* On `case_id` of `Parties`
* On `party_id` of `Victims`
* On `id` of `VehicleMake`

* The cumulative cost for the joins is now 1.1e7, much lower as the database uses nested loops that take advantages of the indexes.
* The cumulative cost the the scans is 50500: using an index makes a slight difference compared to a full scan.

The runtime of the query is:
* Unoptimised: 10s600ms
* Optimised: 794ms

```SQL
SELECT vm.description , COUNT(vm.description)
FROM(
    SELECT p.id
    FROM Collisions c USE INDEX (), CollisionSeverity cs USE INDEX (), Parties p USE INDEX ()
    WHERE c.collision_severity = cs.id AND (
          cs.description = 'Fatal' OR
          cs.description = 'Severe') AND
          p.case_id = c.case_id) AS hard_collisions, Vehicles v USE INDEX (), VehiculeMake vm USE INDEX ()
WHERE v.party_id = hard_collisions.id AND
      vm.id = v.vehicle_make
GROUP BY (vm.description)
ORDER BY COUNT(vm.description) DESC
LIMIT 10;
```