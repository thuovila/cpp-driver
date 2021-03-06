# User Defined Types (UDTs)

User defined types (UDT) can be used to create arbitrary user types with fields
that can be accessed by name or position. When used with the driver they can be
created from a previously defined type determined from schema or they can be
created from a manually defined data type.

## Creating a UDT from Schema

An [`CassSchema`] instance can be used to construct a new [`CassUserType`]. The
[`CassSchema`] instance returns a [`CassDataType`] object which is used to
describe Cassandra types including UDTs, tuples, collections and all basic types
(`int`, `bigint`, `uuid`, etc.).

```c
CassSchema* schema = cass_session_get_schema(session);

CassDataType* data_type = cass_schema_get_udt(schema, "keyspace", "typename");

CassUserType* user_type = cass_user_type_new_from_data_type(data_type);

/* Bind values to user type fields and bind user type to a statement */
```

## Manually Constructing a UDT Data Type

If schema metadata updates are disabled it is still possible to create UDTs
from a manually construct [`CassDataType`].

```c
CassDataType* data_type = cass_data_type_new_udt(3);

cass_data_type_add_sub_value_type_by_name(data_type, "field1",
CASS_VALUE_TYPE_INT);
cass_data_type_add_sub_value_type_by_name(data_type, "field2",
CASS_VALUE_TYPE_UUID);
cass_data_type_add_sub_value_type_by_name(data_type, "field3",
CASS_VALUE_TYPE_TEXT);

CassUserType* user_type = cass_user_type_new_from_data_type(data_type);

/* Bind values to user type fields */

cass_data_type_free(data_type);
```

The preceding code is equivalent to defining the following schema and using
[`cass_session_get_schema()`] obtain the data type.

```cql
CREATE TYPE IF NOT EXISTS udt (field1 int, field2 uuid, field3 text);
```

## Consuming values from a UDT result

A UDT returned from Cassandra is consumed by iterating over its fields similar
to the way collections or tuples are consumed.

```c
/* Retrieve UDT value from column */
const CassValue* udt_value = cass_row_get_column_by_name(row, "value1");

/* Create an iterator for the UDT value */
CassIterator* udt_iterator = cass_iterator_from_user_type(udt_value);

/* Iterate over the UDT fields */
while (cass_iterator_next(udt_iterator)) {
  const char* field_name;
  size_t field_name_length;
  /* Get UDT field name */
  cass_iterator_get_user_type_field_name(udt_iterator,
                                         &field_name, &field_name_length);

  /* Get UDT field value */
  const CassValue* field_value =
    cass_iterator_get_user_type_field_value(udt_iterator);

  /* ... */
}

/* The UDT iterator needs to be freed */
cass_iterator_free(udt_iterator);
```
[`CassSchema`]: http://datastax.github.io/cpp-driver/api/CassSchema/
[`CassUserType`]: http://datastax.github.io/cpp-driver/api/CassUserType/
[`CassDataType`]: http://datastax.github.io/cpp-driver/api/CassDataType/
[`cass_session_get_schema()`]: http://datastax.github.io/cpp-driver/api/CassSession/#cass-session-get-schema
