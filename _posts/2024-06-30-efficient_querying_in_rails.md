# Efficient Querying in Rails: A Deep Dive into `includes`, `preload`, `eager_load`, and `joins`

In Ruby on Rails, `includes(*args)`, `preload(*args)`, `eager_load(*args)` and `joins(*args)` are powerful ActiveRecord query methods. These query methods are often used to optimize data loading and prevent N+1 query problems. However, the way they work under the hood is not the same. Here's a detailed explanation of what happens with each method.

#### _Example Structure:_
We have a User model that can have many ShortPost and LongPost.

## includes

`includes` is dynamic. It is a shorthand for `eager_load` and `preload`. It will either generate a single LEFT OUTTER JOIN like `eager_load` (e.g., if you have a `where` clause that references a relationship) or loop over every association and generate seperate query for each just like `preload`.

<br>

## eager_load
```
users = User.eager_load(:short_posts, :long_posts)
```

This will generate SQL query similar to the following:

```
SELECT DISTINCT "users"."id" FROM "users"
LEFT OUTER JOIN "short_posts" ON "short_posts"."user_id" = "users"."id"
LEFT OUTER JOIN "long_posts" ON "long_posts"."user_id" = "users"."id"

SELECT "users"."id" AS t0_r0, "users"."name" AS t0_r1, "users"."created_at" AS t0_r2, "users"."updated_at" AS t0_r3,
       "short_posts"."id" AS t1_r0, "short_posts"."user_id" AS t1_r1, "short_posts"."created_at" AS t1_r2,
       "short_posts"."updated_at" AS t1_r3, "short_posts"."published" AS t1_r4, "long_posts"."id" AS t2_r0,
       "long_posts"."user_id" AS t2_r1, "long_posts"."created_at" AS t2_r2, "long_posts"."updated_at" AS t2_r3, "long_posts"."published" AS t2_r4 

FROM "users"
LEFT OUTER JOIN "short_posts" ON "short_posts"."user_id" = "users"."id"
LEFT OUTER JOIN "long_posts" ON "long_posts"."user_id" = "users"."id"
WHERE "users"."id" IN (?, ?, ?, ?, ?)
```

> `eager_load` generates single query joining all specified associations at once with left outter join and loads `User`, `ShortPost` and `LongPost` records to the memory. This might cause an effincency problem if the datasets you are querying is huge. 

> Because this query will return all records from the left table, regardless of whether they have a match in the right table, along with any matching records from the right table, it's better to be cautious.

<br>


## preload
```
users = User.preload(:short_posts, :long_posts)
```

This will generate SQL query similar to the following:

```
SELECT "users".* FROM "users"
SELECT "short_posts".* FROM "short_posts" WHERE "short_posts"."user_id" IN (?, ?, ?, ?, ?)
SELECT "long_posts".* FROM "long_posts" WHERE "long_posts"."user_id" IN (?, ?, ?, ?, ?)
```

> `preload` generates seperate queries for each specified association and loads `User`, `ShortPost` and `LongPost` records to the memory.

<br>

## joins

```
users = User.joins(:short_posts, :long_posts)
            .where(short_posts: { published: true }, long_posts: { published: false })
```

This will generate SQL query similar to the following:

```
SELECT "users".*
FROM "users"
INNER JOIN "short_posts" ON "short_posts"."user_id" = "users"."id"
INNER JOIN "long_posts" ON "long_posts"."user_id" = "users"."id"
WHERE "short_posts"."published" = 1 AND "long_posts"."published" = 0;
```

> `joins` uses simple INNER JOIN and does not load associated records to the memory, meaning that it will trigger N+1 if you try to access associated data columns. However, it's handy for building conditional queries for spesific data matching and filtering.

<br>

## TAKEAWAY

Each method serves different purposes:

* `eager_load` fetches all associated records in a _single query_, ideal for reducing N+1 query issues.
* `preload` issues _separate queries_ to preload associations, ideal for reducing N+1 query issues.
* `joins` performs an inner join to filter results based on specified conditions, useful for fetching filtered datasets efficiently, **not ideal** for reducing N+1 query issues.
* `includes` it will decide if it'll use `eager_load` or `preload` under the hood, ideal for reducing N+1 query issues.

<br>
