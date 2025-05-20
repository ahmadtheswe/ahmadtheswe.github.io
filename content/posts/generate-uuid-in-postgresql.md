+++
date = '2025-05-20T08:43:59+07:00'
draft = false
title = 'Generate UUID in PostgreSQL'
author = 'ahmad'
categories = ['SQL', 'Tutorial']
tags = ['postgresql', 'uuid']
featuredImage = '/images/generate-uuid-in-postgresql/featured.png'
+++

UUID is an important data type that we can utilize in relational database (RDS). There are two common ways to generate UUIDs in PostgreSQL: using the `uuid-ossp` extension or the `pgcrypto` extension. In this article, we will discuss how to generate UUIDs using both `pgcrypto`.

To use this function, make sure that the `pgcrypto` extension is enabled in your database. You can do this by running this command:

```postgresql
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
```

Let's say, you have a table named `candidate` with a UUID column named `id`. You can create the table like this:

```postgresql
CREATE TABLE candidate (
    id UUID PRIMARY KEY,
    name TEXT
);
```

Once the extension is enabled, you can use the `gen_random_uuid()` function in your `INSERT` statement, for example:

```postgresql
INSERT INTO candidate (id, name) VALUES (gen_random_uuid(), 'Ahmad');
```

You can also use the `gen_random_uuid()` function to generate UUIDs multiple data in your table in a single insert query, for example :

```postgresql
INSERT INTO candidate (id, name) VALUES (gen_random_uuid(), 'Ahmad'), (gen_random_uuid(), 'Mujahid');
```

If you want to generate UUIDs for existing rows in your table, you can use the `UPDATE` statement like this:

```postgresql
UPDATE candidate SET id = gen_random_uuid() WHERE id IS NULL;
```

### Pro Tip
You can set the `id` column to be generated automatically by using the `DEFAULT` keyword in your table definition so that you don't need to specify the UUID in your `INSERT` statement.

```postgresql
CREATE TABLE candidate (
    id UUID PRIMARY KEY DEFAULT
        gen_random_uuid(),
    name TEXT
);
```

Then, you can simply insert data without specifying the `id`:

```postgresql
INSERT INTO candidate (name) VALUES ('Ahmad');
```