# Feedback from the teaching assistant

Group 54, Here is some feedback on your recent Project 1 Milestone. 

## ER Diagram 
- Modeling choices: Think about the repeated information stored in rows of your table with your current entity modeling. As a thought experiment, if 1000 accidents took place in the same county_city_location "Lausanne", it would not be efficient to repeatedly store "Lausanne" 1000 times in the rows of our collision table. We'd prefer to store it only once as a single row in a separate location entity, and reference it. Apply this logic to your modeling structure, starting with the feedback below. 

- Consider modeling "other_associated_factors" as a separate entity. 

- There is no strong benefit to creating "Party Contexts" and "Victim Contexts" entities separate from your Party and Victim entities, because all this requires is more joining at query time. 

- Please justify your decision to model safety equipment-related information as fields of victims and party instead of migrating them to a separate 'Safety Equipment' entity. 

- Consider modeling road conditions and weather as separate entities. 

- Note that once you load and start querying the data, you may have to further calibrate your assumptions and your participation and key constraints. 

- Participation constraints are correct. 

## DDL Commands + Relational Schema 

- Any adjustments needed after applying the above feedback to the ER model. 

- Use "CHECK" to enforce constraints in your DDL commands. For example, hit_and_run CHAR NOT NULL CHECK(hit_and_run IN ('F', 'M', 'N')). 

- Strong vs. weak entities: consider converting currently-modeled-as-strong, non-uniquely identifiable entities to weak ones. For example, parties can be modeled as a weak entity, as they only exist in the context of a collision. 

## Other Notes

- Very neat handwriting :) I like how you modeled the ER diagram with colors. Great work! If you have questions, feel free to attend my office hours next week on Friday or make a separate appointment. 

Cheers, Vinitra

# Meaning

## ER Diagram

- From [this post](https://moodle.epfl.ch/mod/forum/discuss.php?d=58907) as well as the above "thought experiment", we can assume that the TA wants us to store each enum in a separate table. This would allow to have data interpetability as well as data compression.

## DDL Commands + Relational Schema

- We already model Party as a weak entity via `FOREIGN KEY`. Is there something I am missing?