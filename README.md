# PostgresQL Specialization - Coursera

<h2> Index </h2>  

[PostgreSQL for Everybody Specialization link](https://www.coursera.org/specializations/postgresql-for-everybody)

- [Links](#links)
- [1. Database Design and Basic SQL in PostgreSQL](#1-Database Design and Basic SQL in PostgreSQL)
- [2. Intermediate PostgreSQL](#2-Intermediate PostgreSQL)

## 2. Intermediate PostgresQL

### Text in databases.
 
Generating text. 

```

CREATE TABLE bigtext(content TEXT);

iINSERT INTO bigtext(content) SELECT 'This is record number ' || i::text ||' of quite a few text records.' FROM generate_series(100000, 200000) AS t(i);
```