# ER Model

Consider replacing the following piece of trash with your beautiful sketch:

|**Collision**| --- \<*Involves*\> <-- |**Party**| --- \<*Associated with*\> <-- |**Victim**|

# Data constraints

## Participation constraints

- Partial participation from collision in the 'involves' relationship:
        case_id 10 has no involved party.
- 'Exactly one' participation from party in the 'involves' relationship:
        Every party has an associated case_id that is unique.
- Partial participation from party in the 'associated with' relationship:
        Some case_id have no victim e.g. case_id 1,2.
- 'Exactly one' participation from victim in the 'associated with' relationship:
        Given a case_id, if there is a victim, there is a party (unique).

## Additional constraints

- Key/participation constraints already covered. (Are those two different?)
- No ISA relationship hence no covering/overlap constraint.
- We need to translate the key/participation constraints into SQL constraints. 
    (e.g. nullable)
- Now, the other constraints we cannot express using ER model are important 
    to mention in the report.
- In the project description, each attribute that is nullable is clearly stated: 
    > Blank or - - Not Stated

    Hence we can mark as `NOT NULL` every other attribute, except for `PRIMARY KEY`s, which are implicitely so.

# Design choices

## Star schema

We decided to cluster attributes into separate entities following a star schema. Some groups are obvious, others are debatable.

The obvious groups are:

 * For the `Collision` entity, some attributes form the logical groups `Pcf`, `Location`, `Factor`, `Case`.
 * Similarly, for the `Victim` entity, the only logical group is `Vehicle`.

 We also decided to add two less obvious groups:
 * For the `Party` entity, attributes which are orthogonal to the collision are not stored in a separate entity (age, sex, ...)

    Attributes which are about the context of the collision are stored in a `PartyContext` entity.

 * Similarly for the `Victim` entity, we have a `VictimContext` entity.

## Attribute types in the SQL code

 * In the project description, some attributes are enums: they can only take on specific pre-defined values. 
 Therefore, we can let them be `INTEGER` and have lookup tables when we dump the `.csv`s into a `SQL` database. 
 
    The alternative would be to let them be `VARCHAR`. The problem with this approach is that determining the max length means looking up the max number of characters for each attribute. 
    
    Granted: creating lookup tables would require the same amount of work; however it leads to  substantial data compression.
 
    Note that these attributes are the same that are `nullable`.

 * Similarly, `tow_away` from `Collisions` can be translated from a `float` (0.0 or 1.0) to a `BIT`.

 * The rest of the attributes are clearly `INTEGER` from the project description as well as upon inspection of the values in the `.csv` files.

# SQL code

TODO:
- Review this shit.
- Add NOT NULL constraints.

Do we refine data types?
- For example, entries might be enums, hence we need only 1 character to encode them.
    Do we reduce such entries like "not hit and run" into "A"?
- If we don't reduce enum what argument do we give VARCHAR()?
    A good upper-bound? The exact max length of an entry?

## Collision

```SQL
CREATE TABLE Collisions(case_id INTEGER, 
                        collision_date DATE,
                        collision_time TIME,
                        type_of_collision VARCHAR(15),
                        collision_severity VARCHAR(30),
                        hit_and_run VARCHAR(20),
                        tow_away BIT NOT NULL,                        
                        PRIMARY KEY(case_id))
```

```SQL
CREATE TABLE Pcfs(case_id INTEGER,
                    pcf_violation INTEGER,
                    pcf_violation_category VARCHAR(30),
                    pcf_violation_subsection CHAR(1),
                    FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

```SQL
CREATE TABLE Locations(case_id INTEGER,
                        population INTEGER,
                        county_city_location INTEGER,
                        FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

```SQL
CREATE TABLE Factors(case_id INTEGER,
                        location_type VARCHAR(20),
                        lighting VARCHAR(20),
                        road_condition_1 VARCHAR(30),
                        road_condition_2 VARCHAR(30),
                        road_surface VARCHAR(20),
                        weather_1 VARCHAR(10),
                        weather_2 VARCHAR(10),
                        FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

```SQL
CREATE TABLE Cases(case_id INTEGER,
                    process_date DATE,
                    officer_id INTEGER,
                    jurisdiction INTEGER,
                    FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

## Party

Only permanent attributes of a `Party` are stored in the `Parties` table.
Attributes from the context of the collision are stored in the `PartyContexts` table.

```SQL
CREATE TABLE Parties(id INTEGER,
                        case_id INTEGER,
                        party_number INTEGER,
                        finanicial_responsibility ???,
                        party_age INTEGER,
                        party_sex ???,
                        PRIMARY KEY(id),
                        FOREIGN KEY(party_id) REFERENCES Parties(party_id))
```

```SQL
CREATE TABLE PartyContexts(party_id INTEGER,
                        at_fault ???,
                        cellphone_use ???,
                        hazardous_materials ???,
                        movement_preceding_collision ???,
                        other_associate_factor_1 ???,
                        other_associate_factor_2 ???,
                        party_drug_physical ???,
                        party_safety_equipment_1 ???,
                        party_safety_equipment_2 ???,
                        party_sobriety  ???,
                        FOREIGN KEY(party_id) REFERENCES Parties(party_id))
```

```SQL
CREATE TABLE Vehicles(party_id INTEGER,
                        school_bus_related ???,
                        statewide_vehicle_type ???,
                        vehicle_make ???,
                        vehicle_year ???,
                        FOREIGN KEY(party_id) REFERENCES Parties(party_id))
```

## Victim

All collision context specific attributes of a `Victim` are stored
in a separate table. 

```SQL
CREATE TABLE Victims(id INTEGER,
                        case_id INTEGER,
                        party_number INTEGER,
                        victim_age INTEGER,
                        victim_sex ???,
                        PRIMARY KEY(id),
                        FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

```SQL
CREATE TABLE VictimContexts(victim_id INTEGER,
                            victim_degree_of_injury ???,
                            victim_ejected ???,
                            victim_role ???,
                            victim_safety_equipment_1 ???,
                            victim_safety_equipment_2 ???,
                            victim_seating_position ???,
                            FOREIGN KEY(victim_id) REFERENCES Victims(victim_id))
```