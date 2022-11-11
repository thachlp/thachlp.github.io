Although **SQL** and **Elasticsearch** have different terms for the way the data is organized (and different semantics), but their purpose is the same, so here is the mapping:
## Terminologies and data types
| SQL       | Elasticsearch | Description                                                                                                                                                                              |
|-----------|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| database  | cluster       | In MySQL, database is used to represent a set of schema, a number of tables. <br/>In Elasticsearch a set of indices available are grouped in a cluster                                   |
| schema    | *implicit*    |                                                                                                                                                                                          |
| table     | index         |                                                                                                                                                                                          |
| row       | document      |                                                                                                                                                                                          |
| column    | field         |                                                                                                                                                                                          |
| AND       | must          |                                                                                                                                                                                          |
| OR        | should        |                                                                                                                                                                                          |
| NOT       | must_not      |                                                                                                                                                                                          |
| TINYINT   | byte          | In Elasticsearch, byte is 8-bit                                                                                                                                                          |
| SMALLINT  | short         | In Elasticsearch, short is 16-bit                                                                                                                                                        |
| BIGINT    | long          | In Elasticsearch, long is 64-bit integer                                                                                                                                                 |
| REAL      | float         | In Elasticsearch, float is 32-bit floating point                                                                                                                                         |
| FLOAT     | half_float    | In Elasticsearch, half_float is 32-bit floating point                                                                                                                                    |
| DOUBLE    | scaled_float  | In Elastic search, scaled_float is backed by a long, scaled by a fixed double                                                                                                            |
| VARCHAR   | keyword       | In Elasticsearch, keyword which is used for structured content such as IDs, email addresses, hostnames, status codes, zipcodes .... <br/>Avoid using keyword fields for full-text search |
| VARCHAR   | text          | In Elasticsearch, text for full-text content such as the body of an email or description of product                                                                                      |
| VARCHAR   | ip            | In Elasticsearch, ip field can index/store either IPv4 or IPv6                                                                                                                           |
| VARCHAR   | version       | In Elasticsearch, version is a specialization of keyword for handling software version                                                                                                   |
| TIMESTAMP | date          | In Elasticsearch, date can be strings containing formatted dates, a number representing milliseconds-since-the-epoch, seconds-since-the-epoch                                            |
| STRUCT    | object        | A JSON object                                                                                                                                                                            |
| STRUCT    | nested        | A specialised version of the object that allows arrays of object                                                                                                                         |

