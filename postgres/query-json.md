# How to query JSON data in Postgres

Postgres allows to query directly for a value of a specific attribute in a JSON data structure.

`SELECT * FROM table WHERE json_top_level_attr -> 'nested_hash' -> 'nested_array' ->> 0 = value`
