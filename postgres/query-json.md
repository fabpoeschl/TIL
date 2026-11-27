# How to query JSON data in Postgres

Postgres allows to query directly for a value of a specific attribute in a JSON data structure.

`SELECT * FROM table WHERE json_top_level_attr -> ARRAY_INDEX ->> 'hash_key' = value`

Same logic may be applied in ActiveRecord where queries.
