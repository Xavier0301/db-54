# ER Model

<img src="https://user-images.githubusercontent.com/35627867/112846274-27b3d580-90a6-11eb-8029-fe0aa248594e.jpg" alt="drawing"/>

# Data constraints

## Participation constraints

- Partial participation from `Collision` in the `involved in` relationship:
        `case_id` `10` has no involved party.
- 'Exactly one' participation from `Party` in the `involved in` relationship:
        Every party has an associated `case_id` that is unique.
- Partial participation from `Party` in the 'associated with' relationship:
        Some `case_id` have no victim e.g. `case_id` `1,2`.
- 'Exactly one' participation from `Victim` in the `associated with` relationship:
        Given a `case_id`, if there is a victim, there is a party (unique).

## Additional constraints

 * In the project description, each attribute that is nullable is clearly stated: 

        > Blank or - - Not Stated
   Hence we can mark as `NOT NULL` every other attribute, except for `PRIMARY KEY`s, which are implicitely so.
 * Since we are using a star schema, we cannot express the following constraints in the SQL code: every collision has a location, a factor, a pcf and a case. Likewise, every party has a vehicle and a party context. Finally, every victim has a victim context. 
 * As said in the moodle forum, a victim is not a party.

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

 * Similarly, `tow_away` from `Collisions` can be translated from a `float` (`0.0` or `1.0`) to a `BIT`.

 * The rest of the attributes are clearly `INTEGER` from the project description as well as upon inspection of the values in the `.csv` files.

# SQL code

## Collision

### Main Tables

```SQL
CREATE TABLE Collisions(case_id NUMERIC(19), 
                        collision_date DATE NOT NULL,
                        collision_time TIME,
                        type_of_collision INTEGER,
                        collision_severity INTEGER NOT NULL,
                        hit_and_run INTEGER NOT NULL,
                        tow_away BIT,
                        FOREIGN KEY(type_of_collision) REFERENCES TypeOfCollision(id), 
                        FOREIGN KEY(collision_severity) REFERENCES CollisionSeverity(id), 
                        FOREIGN KEY(hit_and_run) REFERENCES HitAndRun(id),                      
                        PRIMARY KEY(case_id))
```

```SQL
CREATE TABLE Pcfs(case_id NUMERIC(19) NOT NULL,
                    pcf_violation INTEGER,
                    pcf_violation_category INTEGER,
                    pcf_violation_subsection CHAR(1),
                    FOREIGN KEY(pcf_violation_category) REFERENCES PcfViolationCategory(id),
                    FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

```SQL
CREATE TABLE Locations(case_id NUMERIC(19) NOT NULL,
                        population INTEGER,
                        county_city_location INTEGER NOT NULL,
                        FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

```SQL
CREATE TABLE Factors(case_id NUMERIC(19) NOT NULL,
                        location_type INTEGER,
                        lighting INTEGER,
                        road_condition_1 INTEGER,
                        road_condition_2 INTEGER,
                        road_surface INTEGER,
                        weather_1 INTEGER,
                        weather_2 INTEGER,
                        FOREIGN KEY(location_type) REFERENCES LocationType(id),
                        FOREIGN KEY(lighting) REFERENCES Lighting(id),
                        FOREIGN KEY(road_condition_1) REFERENCES RoadCondition(id),
                        FOREIGN KEY(road_condition_2) REFERENCES RoadCondition(id),
                        FOREIGN KEY(road_surface) REFERENCES RoadSurface(id),
                        FOREIGN KEY(weather_1) REFERENCES Weather(id),
                        FOREIGN KEY(weather_2) REFERENCES Weather(id),
                        FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

```SQL
CREATE TABLE Cases(case_id NUMERIC(19) NOT NULL,
                    process_date DATE NOT NULL,
                    officer_id VARCHAR(8),
                    jurisdiction INTEGER,
                    FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

### Satelite Enum Tables

```SQL
CREATE TABLE CollisionSeverity(id INTEGER AUTO_INCREMENT,
			       description VARCHAR(25) NOT NULL UNIQUE,
			       PRIMARY KEY(id))
```

```SQL
CREATE TABLE HitAndRun(id INTEGER AUTO_INCREMENT,
			description VARCHAR(20) NOT NULL UNIQUE,
			PRIMARY KEY(id))
```

```SQL
CREATE TABLE Lighting(id INTEGER AUTO_INCREMENT,
		       description VARCHAR(39) NOT NULL UNIQUE,
		       PRIMARY KEY(id))
```

```SQL
CREATE TABLE LocationType(id INTEGER AUTO_INCREMENT,
			   desc VARCHAR(20) NOT NULL UNIQUE,
			   PRIMARY KEY(id))
```

```SQL
CREATE TABLE PcfViolationCategory(id INTEGER AUTO_INCREMENT,
				   description VARCHAR(70) NOT NULL UNIQUE,
				   PRIMARY KEY(id))
```

```SQL
CREATE TABLE PrimaryCollisionFactor(id INTEGER AUTO_INCREMENT,
				     description VARCHAR(25) NOT NULL UNIQUE,
				     PRIMARY KEY(id))
```

```SQL
CREATE TABLE RoadCondition(id INTEGER AUTO_INCREMENT,
			    description VARCHAR(20) NOT NULL UNIQUE,
			    PRIMARY KEY(id))
```

```SQL
CREATE TABLE RoadSurface(id INTEGER AUTO_INCREMENT,
			  description VARCHAR(15) NOT NULL UNIQUE,
			  PRIMARY KEY(id))
```


```SQL
CREATE TABLE TypeOfCollision(id INTEGER AUTO_INCREMENT,
			      description VARCHAR(15) NOT NULL UNIQUE,
			      PRIMARY KEY(id))
```

```SQL
CREATE TABLE Weather(id INTEGER AUTO_INCREMENT,
		      desc VARCHAR(10) NOT NULL UNIQUE,
		      PRIMARY KEY(id))
```

## Party

### Main Tables

```SQL
CREATE TABLE Parties(id INTEGER,
                        case_id NUMERIC(19) NOT NULL,
                        party_number INTEGER NOT NULL,
                        finanicial_responsibility INTEGER,
                        party_age INTEGER,
                        party_sex INTEGER,
                        at_fault BIT NOT NULL,
                        cellphone_use INTEGER,
                        hazardous_materials CHAR(1),
                        movement_preceding_collision INTEGER,
                        other_associate_factor_1 INTEGER,
                        other_associate_factor_2 INTEGER,
                        party_drug_physical INTEGER,
                        party_safety_equipment_1 INTEGER,
                        party_safety_equipment_2 INTEGER,
                        party_sobriety INTEGER,
                        PRIMARY KEY(id),
                        FOREIGN KEY(finanicial_responsibility) REFERENCES FinancialResponsability(id),
                        FOREIGN KEY(party_sex) REFERENCES PersonSex(id),
                        FOREIGN KEY(cellphone_use) REFERENCES CellphoneUse(id),
                        FOREIGN KEY(movement_preceding_collision) REFERENCES MovementPrecedingCollision(id),
                        FOREIGN KEY(other_associate_factor_1) REFERENCES OtherAssociatedFactors(id),
                        FOREIGN KEY(other_associate_factor_2) REFERENCES OtherAssociatedFactors(id),
                        FOREIGN KEY(party_drug_physical) REFERENCES PartyDrugPhysical(id),
                        FOREIGN KEY(party_safety_equipment_1) REFERENCES PartySafetyEquipement(id),
                        FOREIGN KEY(party_safety_equipment_2) REFERENCES PartySafetyEquipement(id),
                        FOREIGN KEY(party_sobriety) REFERENCES PartySobriety(id),
                        FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

```SQL
CREATE TABLE Vehicles(party_id INTEGER NOT NULL,
                        school_bus_related BIT,
                        statewide_vehicle_type INTEGER,
                        vehicle_make INTEGER,
                        vehicle_year INTEGER,
                        FOREIGN KEY(statewide_vehicle_type) REFERENCES StatewideVehiculeType(id),
                        FOREIGN KEY(vehicle_make) REFERENCES VehiculeMake(id),
                        FOREIGN KEY(party_id) REFERENCES Parties(id))
```

### Satelite Enum Tables

```SQL
CREATE TABLE CellphoneUse(id INTEGER AUTO_INCREMENT,
			   description CHAR(1) NOT NULL UNIQUE,
                CHECK (description BETWEEN 'B' AND 'D'),	
			   PRIMARY KEY(id))
```

```SQL
CREATE TABLE FinancialResponsibility(id INTEGER AUTO_INCREMENT,
			   	      description CHAR(1) NOT NULL UNIQUE,
                                      CHECK (description IN ('N', 'Y', 'O', 'E', '-')),	
			              PRIMARY KEY(id))
```

```SQL
CREATE TABLE MovementPrecedingCollision(id INTEGER AUTO_INCREMENT,
					 description VARCHAR(100) NOT NULL UNIQUE,
					 PRIMARY KEY(id))
```

```SQL
CREATE TABLE OtherAssociatedFactors (id INTEGER AUTO_INCREMENT,
    description CHAR(1) NOT NULL UNIQUE,
    CHECK (description BETWEEN 'A' AND 'Z'),
    PRIMARY KEY(id))				     
```

```SQL
CREATE TABLE PartyDrugPhysical(id INTEGER AUTO_INCREMENT,
			       description CHAR(1) NOT NULL UNIQUE,
                            CHECK (description IN ('E', 'F', 'H', 'I')),
			       PRIMARY KEY(id))
```

```SQL
CREATE TABLE PartySafetyEquipement(id INTEGER AUTO_INCREMENT,
				   description CHAR(1) NOT NULL UNIQUE,
                                   CHECK (description BETWEEN 'A' AND 'Y'),
				   PRIMARY KEY(id))
```

The following can store `victim_sex` and `party_sex`
```SQL 
CREATE TABLE PersonSex(id INTEGER AUTO_INCREMENT,
		       description VARCHAR(6) NOT NULL UNIQUE,
                       CHECK (description in ('male', 'female')),
		       PRIMARY KEY(id))
```

```SQL
CREATE TABLE PartySobriety(id INTEGER AUTO_INCREMENT,
			    desc CHAR(1) NOT NULL UNIQUE,
                            CONSTRAINT sobriety_check CHECK (desc IN ('A', 'B', 'C', 'D', 'G', 'H')),
			    PRIMARY KEY(id))
```

```SQL
CREATE TABLE PartyType(id INTEGER AUTO_INCREMENT,
			description VARCHAR(14) NOT NULL UNIQUE,
			PRIMARY KEY(id))
```


```SQL
CREATE TABLE StatewideVehiculeType(id INTEGER AUTO_INCREMENT,
				    description VARCHAR(35) NOT NULL UNIQUE,
				    PRIMARY KEY(id))
```

```SQL
CREATE TABLE VehiculeMake(id INTEGER AUTO_INCREMENT,
			   description VARCHAR(28) NOT NULL,
			   PRIMARY KEY(id))
```

## Victim

### Main Table

```SQL
CREATE TABLE Victims(id INTEGER,
                        case_id NUMERIC(19) NOT NULL,
                        party_number INTEGER NOT NULL,
                        victim_age INTEGER,
                        victim_sex INTEGER,
                        victim_degree_of_injury INTEGER,
                        victim_ejected INTEGER,
                        victim_role INTEGER NOT NULL,
                        victim_safety_equipment_1 INTEGER,
                        victim_safety_equipment_2 INTEGER,
                        victim_seating_position INTEGER,
                        PRIMARY KEY(id),
                        FOREIGN KEY(victim_sex) REFERENCES PersonSex(id),
                        FOREIGN KEY(victim_safety_equipment_1) REFERENCES VictimSafetyEquipment(id),
                        FOREIGN KEY(victim_safety_equipment_2) REFERENCES VictimSafetyEquipment(id),
                        FOREIGN KEY(victim_degree_of_injury) REFERENCES VictimDegreeOfInjury(id),
                        FOREIGN KEY(case_id) REFERENCES Collisions(case_id))
```

### Satelite Enum Tables

```SQL
CREATE TABLE VictimSafetyEquipment(id INTEGER AUTO_INCREMENT,
				   description CHAR(1) NOT NULL UNIQUE,
                                   CHECK(description BETWEEN 'A' AND 'Z'),
				   PRIMARY KEY(id))
```

```SQL
CREATE TABLE VictimDegreeOfInjury(id INTEGER AUTO_INCREMENT,
				   description VARCHAR(30) NOT NULL UNIQUE,
				   PRIMARY KEY(id))
```

The following can store `victim_sex` and `party_sex`.

```SQL 
CREATE TABLE PersonSex(id INTEGER AUTO_INCREMENT,
		       description VARCHAR(6) NOT NULL UNIQUE,
                       CHECK (description in ('male', 'female')),
		       PRIMARY KEY(id))
```
