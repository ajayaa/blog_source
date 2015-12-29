---
date: 2015-01-14
title: "Difference between cassandra insert and update statement"
excerpt: "cassandra-insert-update-ttl"
tags: [cassandra, update, insert, ttl, inconsistent, time_to_live]
type: post
---

This post is about explaining one huge difference between cassandra update and insert and how things get hairy when you use ttl with a column.

### Table schema
Here is the cql statement to create the table.

<figure>
<a href="/img/create_table.png"><img src="/img/create_table.png"></a>
</figure>

### Normal insert

<figure>
<a href="/img/insert_normal.png"><img src="/img/insert_normal.png"></a>
</figure>

so far so good.

### Normal update

<figure>
<a href="/img/update_normal.png"><img src="/img/update_normal.png"></a>
</figure>

This is an update which behaves like normal insert.

### insert and update with a null

<figure>
<a href="/img/insert_null.png"><img src="/img/insert_null.png"></a>
</figure>

<figure>
<a href="/img/update_null.png"><img src="/img/update_null.png"></a>
</figure>


In the insert statement as we have mentioned a primary key but set a non-primary-key column to null a row should be inserted into the table. We did the similar thing with update statement. We should be able to verify this with a select statement.

<figure>
<a href="/img/select_after_update_null.png"><img src="/img/select_after_update_null.png"></a>
</figure>

What happended here. There is no row corresponding to primary key 'd' which we used in update statement but there is one row corresponding to primary key 'c' which we used in insert statement. That is weird, right? Yes, it is. Apparently cassandra deletes a row if all of its non primary key fields are set to null if that row was inserted with a update statement.

#### TTL

When you insert ttl into picture things really get hairy and you would run into situations where you would think it's a cassandra bug. But in reality it's not. Many a times it's been reported as a bug in cassandra but had to be explained to naive souls :). I was such a naive soul sometime back. See [8430](https://issues.apache.org/jira/browse/CASSANDRA-8430) and [6668](https://issues.apache.org/jira/browse/CASSANDRA-6668). TTLs are on per column basis in cassandra and you need to be careful when using ttls with update statements. The lifetime of a row would also depend on whether the row was inserted with an insert or update statement. 
