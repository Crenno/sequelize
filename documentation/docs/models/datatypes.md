---
sidebar_position: 2
---

# Data Types

Sequelize provides [a lot of built-in data types](https://github.com/sequelize/sequelize/blob/main/src/data-types.js). To access a built-in data type, you must import `DataTypes`:

```js
const { DataTypes } = require('@sequelize/core'); // Import the built-in data types
```

## Strings

```js
DataTypes.STRING             // VARCHAR(255)
DataTypes.STRING(1234)       // VARCHAR(1234)
DataTypes.STRING.BINARY      // VARCHAR BINARY
DataTypes.TEXT               // TEXT
DataTypes.TEXT('tiny')       // TINYTEXT
DataTypes.CITEXT             // CITEXT          PostgreSQL and SQLite only.
DataTypes.TSVECTOR           // TSVECTOR        PostgreSQL only.
```

## Boolean

```js
DataTypes.BOOLEAN            // TINYINT(1)
```

## Numbers

```js
DataTypes.INTEGER            // INTEGER
DataTypes.BIGINT             // BIGINT
DataTypes.BIGINT(11)         // BIGINT(11)

DataTypes.FLOAT              // FLOAT
DataTypes.FLOAT(11)          // FLOAT(11)
DataTypes.FLOAT(11, 10)      // FLOAT(11,10)

DataTypes.REAL               // REAL            PostgreSQL only.
DataTypes.REAL(11)           // REAL(11)        PostgreSQL only.
DataTypes.REAL(11, 12)       // REAL(11,12)     PostgreSQL only.

DataTypes.DOUBLE             // DOUBLE
DataTypes.DOUBLE(11)         // DOUBLE(11)
DataTypes.DOUBLE(11, 10)     // DOUBLE(11,10)

DataTypes.DECIMAL            // DECIMAL
DataTypes.DECIMAL(10, 2)     // DECIMAL(10,2)
```

### Unsigned & Zerofill integers - MySQL/MariaDB only

In MySQL and MariaDB, the data types `INTEGER`, `BIGINT`, `FLOAT` and `DOUBLE` can be set as unsigned or zerofill (or both), as follows:

```js
DataTypes.INTEGER.UNSIGNED
DataTypes.INTEGER.ZEROFILL
DataTypes.INTEGER.UNSIGNED.ZEROFILL
// You can also specify the size i.e. INTEGER(10) instead of simply INTEGER
// Same for BIGINT, FLOAT and DOUBLE
```

## Dates

```js
DataTypes.DATE       // DATETIME for mysql / sqlite, TIMESTAMP WITH TIME ZONE for postgres
DataTypes.DATE(6)    // DATETIME(6) for mysql 5.6.4+. Fractional seconds support with up to 6 digits of precision
DataTypes.DATEONLY   // DATE without time
```

## UUIDs

For UUIDs, use `DataTypes.UUID`. It becomes the `UUID` data type for PostgreSQL and SQLite, and `CHAR(36)` for MySQL. Sequelize can generate UUIDs automatically for these fields, simply use `DataTypes.UUIDV1` or `DataTypes.UUIDV4` as the default value:

```js
{
  type: DataTypes.UUID,
  defaultValue: DataTypes.UUIDV4 // Or DataTypes.UUIDV1
}
```

## Ranges (PostgreSQL only)

```js
DataTypes.RANGE(DataTypes.INTEGER)    // int4range
DataTypes.RANGE(DataTypes.BIGINT)     // int8range
DataTypes.RANGE(DataTypes.DATE)       // tstzrange
DataTypes.RANGE(DataTypes.DATEONLY)   // daterange
DataTypes.RANGE(DataTypes.DECIMAL)    // numrange
```

Since range types have extra information for their bound inclusion/exclusion it's not very straightforward to just use a tuple to represent them in javascript.

When supplying ranges as values you can choose from the following APIs:

```js
// defaults to inclusive lower bound, exclusive upper bound
const range = [
  new Date(Date.UTC(2016, 0, 1)),
  new Date(Date.UTC(2016, 1, 1))
];
// '["2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00")'

// control inclusion
const range = [
  { value: new Date(Date.UTC(2016, 0, 1)), inclusive: false },
  { value: new Date(Date.UTC(2016, 1, 1)), inclusive: true },
];
// '("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00"]'

// composite form
const range = [
  { value: new Date(Date.UTC(2016, 0, 1)), inclusive: false },
  new Date(Date.UTC(2016, 1, 1)),
];
// '("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00")'

const Timeline = sequelize.define('Timeline', {
  range: DataTypes.RANGE(DataTypes.DATE)
});

await Timeline.create({ range });
```

However, retrieved range values always come in the form of an array of objects. For example, if the stored value is `("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00"]`, after a finder query you will get:

```js
[
  { value: Date, inclusive: false },
  { value: Date, inclusive: true }
]
```

You will need to call `reload()` after updating an instance with a range type or use the `returning: true` option.

### Special Cases

```js
// empty range:
Timeline.create({ range: [] }); // range = 'empty'

// Unbounded range:
Timeline.create({ range: [null, null] }); // range = '[,)'
// range = '[,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [null, new Date(Date.UTC(2016, 0, 1))] });

// Infinite range:
// range = '[-infinity,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [-Infinity, new Date(Date.UTC(2016, 0, 1))] });
```

## BLOBs

```js
DataTypes.BLOB                // BLOB (bytea for PostgreSQL)
DataTypes.BLOB('tiny')        // TINYBLOB (bytea for PostgreSQL)
DataTypes.BLOB('medium')      // MEDIUMBLOB (bytea for PostgreSQL)
DataTypes.BLOB('long')        // LONGBLOB (bytea for PostgreSQL)
```

The blob datatype allows you to insert data both as strings and as buffers. However, when a blob is retrieved from database with Sequelize, it will always be retrieved as a buffer.

## ENUMs

The ENUM is a data type that accepts only a few values, specified as a list.

```js
DataTypes.ENUM('foo', 'bar') // An ENUM with allowed values 'foo' and 'bar'
```

ENUMs can also be specified with the `values` field of the column definition, as follows:

```js
sequelize.define('foo', {
  states: {
    type: DataTypes.ENUM,
    values: ['active', 'pending', 'deleted']
  }
});
```

## JSON (SQLite, MySQL, MariaDB and PostgreSQL only)

The `DataTypes.JSON` data type is only supported for SQLite, MySQL, MariaDB and PostgreSQL. However, there is a minimum support for MSSQL (see below).

### Note for PostgreSQL

The JSON data type in PostgreSQL stores the value as plain text, as opposed to binary representation. If you simply want to store and retrieve a JSON representation, using JSON will take less disk space and less time to build from its input representation. However, if you want to do any operations on the JSON value, you should prefer the JSONB data type described below.

### JSONB (PostgreSQL only)

PostgreSQL also supports a JSONB data type: `DataTypes.JSONB`. It can be queried in three different ways:

```js
// Nested object
await Foo.findOne({
  where: {
    meta: {
      video: {
        url: {
          [Op.ne]: null
        }
      }
    }
  }
});

// Nested key
await Foo.findOne({
  where: {
    "meta.audio.length": {
      [Op.gt]: 20
    }
  }
});

// Containment
await Foo.findOne({
  where: {
    meta: {
      [Op.contains]: {
        site: {
          url: 'http://google.com'
        }
      }
    }
  }
});
```

### MSSQL

MSSQL does not have a JSON data type, however it does provide some support for JSON stored as strings through certain functions since SQL Server 2016. Using these functions, you will be able to query the JSON stored in the string, but any returned values will need to be parsed seperately.

```js
// ISJSON - to test if a string contains valid JSON
await User.findAll({
  where: sequelize.where(sequelize.fn('ISJSON', sequelize.col('userDetails')), 1)
})

// JSON_VALUE - extract a scalar value from a JSON string
await User.findAll({
  attributes: [[ sequelize.fn('JSON_VALUE', sequelize.col('userDetails'), '$.address.Line1'), 'address line 1']]
})

// JSON_VALUE - query a scalar value from a JSON string
await User.findAll({
  where: sequelize.where(sequelize.fn('JSON_VALUE', sequelize.col('userDetails'), '$.address.Line1'), '14, Foo Street')
})

// JSON_QUERY - extract an object or array
await User.findAll({
  attributes: [[ sequelize.fn('JSON_QUERY', sequelize.col('userDetails'), '$.address'), 'full address']]
})
```

## Others

```js
DataTypes.ARRAY(/* DataTypes.SOMETHING */)  // Defines an array of DataTypes.SOMETHING. PostgreSQL only.

DataTypes.CIDR                        // CIDR                  PostgreSQL only
DataTypes.INET                        // INET                  PostgreSQL only
DataTypes.MACADDR                     // MACADDR               PostgreSQL only

DataTypes.GEOMETRY                    // Spatial column. PostgreSQL (with PostGIS) or MySQL only.
DataTypes.GEOMETRY('POINT')           // Spatial column with geometry type. PostgreSQL (with PostGIS) or MySQL only.
DataTypes.GEOMETRY('POINT', 4326)     // Spatial column with geometry type and SRID. PostgreSQL (with PostGIS) or MySQL only.
```
