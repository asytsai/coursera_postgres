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


### Inverted Index

Building an inverted index:

1. Split text columns into space-delimited array

2. Turn them into separate rows

3. Create mapping { keyword: document id }

```
CREATE TABLE docs01 (id SERIAL, doc TEXT, PRIMARY KEY(id));

CREATE TABLE invert01 (
  keyword TEXT,
  doc_id INTEGER REFERENCES docs01(id) ON DELETE CASCADE
);
```

```
INSERT INTO docs01 (doc) VALUES
('write in highlevel languages like Python or JavaScript and these'),
('translators convert the programs to machine language for actual'),
('execution by the CPU'),
('Since machine language is tied to the computer hardware machine'),
('language is not portable across different types of'),
('hardware Programs written in highlevel languages can be moved between'),
('different computers by using a different interpreter on the new machine'),
('or recompiling the code to create a machine language version of the'),
('program for the new machine'),
('These programming language translators fall into two general categories');
```

This is a way to map each word to a document id

```
SELECT id, s.kwd AS kwd FROM docs01 as D, unnest(string_to_array(lower(D.doc), ' ')) s(kwd) ORDER BY id;
```

Insert these values into table holding inverted index:

```
INSERT INTO invert01 (keyword, doc_id) SELECT DISTINCT s.kwd as kwd, id  FROM docs01 as D, unnest(string_to_array(lower(D.doc), ' ')) s(kwd) ORDER BY id;
```


### Building an inverted index with Postgres functions

Index types:

1. **GIN**: generalized inverted index (insert is more expensive)

2. **GIST**: generalized search tree (query is more expensive)

```
CREATE INDEX array03 ON docs03 USING gin(string_to_array(lower(docs03.doc), ' ') array_ops);

EXPLAIN SELECT id, doc FROM docs03 WHERE '{interpreter}' <@ string_to_array(lower(doc), ' ');
```

If EXPLAIN SELECT displays ```seq scan```, index is not working. If it displays 
```Bitmap Heap Scan```, the index is being used.

#### Build GIN index using **to_tsvector**:

```
CREATE INDEX fulltext03 ON docs03 USING gin(to_tsvector('english', doc));

pg4e_146c059e69=> EXPLAIN SELECT id, doc FROM docs03 WHERE to_tsquery('english', 'instructions') @@ to_tsvector('english', doc);
```