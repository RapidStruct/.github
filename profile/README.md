# RapidStruct

RapidStruct is a bare-bones, schema-based binary serialization format.
It is desinged to be straightforward, fast, and easy to implement.

## Format

A 'RapidStruct' natively supports the following data types:
- Boolean
- 1-byte unsigned integer
- 2-byte unsigned integer (short)
- 4-byte unsigned integer
- 8-byte unsigned integer (long)
- 4-byte floating point
- 8-byte floating point
- Strings of single-byte characters
- Raw byte arrays
- Nested structs

A RapidStruct schema can support 256 unique field definitions.
A field definition is a textual tag that represents a data type.

So let's take the example of representing a birthday:
You would need year, month, and data fields, and we'll also list a name as well.
That can be represented with four fields.
In psuedo-code, that schema definition would like something like the following:
~~~
Schema birthdaySchema
birthdaySchema.addFieldToSchema("Year", FieldType.SHORT)
birthdaySchema.addFieldToSchema("Month", FieldType.BYTE)
birthdaySchema.addFieldToSchema("Day", FieldType.BYTE)
birthdaySchema.addFieldToSchema("Name", FieldType.STRING)
~~~
Behind the scenes, that schema is assigning 1-byte keys to each field definition.
I.e., Year is now field 0, Month is field 1, Day is 2, and Name is 3.
And since the schema is available on both the serialization and deserialization, both sides know what data type is associate with field "Year" (unsigned short).
This schema definition only represents the *possible* fields that it can contain.
A RapidStruct does not have to inlude all of the defined fields, and it may have multiple of the same field.
In the birthday example above, you may want to list everybody in a group that shares a birthday.
You could set the year, month, and day fields, and then have multiple name fields, which is perfectly valid and allows things arrays to be represented.

## Bytes on the Wire
All data is represented in network order/big endian.
When serializing, all fixed-length data types are prepended with their schema-key/tag number, and then their value.
With variable length fields (strings, raw bytes, and nested structs) the schema-key/tag number comes first, then a 2-byte length, and finally the data.
This is about as simple as you can get using the tag-length-value approach.
Referring to the birthday example again, here is how that data would be serialized to a byte array.

For reference:  
Year, key 0, value: 1987  
**Byte Representation:** `[0][7][95]`  

Month, key 1, value 11  
**Byte Representation:** `[1][11]`  

Day, key 2, value 21  
**Byte Representation:** `[2][21]`  

Name, key 3, value "John Smith"  
**Byte Representation:** `[3][0][10][74][111][104][110][32][83][109][105][116][104]`

Combined byte-array below:
~~~
   Year     Month    Day                             Name
[0][7][95]-[1][11]-[2][21]-[3][0][10][74][111][104][110][32][83][109][105][116][104]
             ASCII Name reference---> [J] [o]  [h]  [n]  [ ] [S] [m]  [i]  [t]  [h]
~~~

## Quick Facts

- Because the fields are represented using 1-byte keys, you can have a maximum of 256 fields definitions, but you can have multiple of those fields in one RapidStruct
- Because the length of fields that a variable in size (strings, raw bytes, nested structs) are represented using a 2-byte integer, the maximum length of a single field is 65535 bytes
- The total length of a RapidStruct is not present in it's serialized form: it is up to you to determine how many bytes to deserialize

Copyright (c) 2026, Noah McLean
