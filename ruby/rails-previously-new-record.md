# How to know whether a save created a record

After a save, `new_record?` is always `false`, so it cannot tell whether the save inserted or updated
the record.

`previously_new_record?` (Rails 6.1+) returns `true` if the last save created the record:

```
record = Model.find_or_initialize_by(id: id)
record.update!(name: name)

record.previously_new_record? # true if update! inserted the record, false if it updated an existing one
```

The value only describes the last save: saving the record again resets it to `false`.

Similar: `previously_persisted?` returns `true` for a record that was persisted and then destroyed.
