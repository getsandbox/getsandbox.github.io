---
layout: post
heading: How to manage incremental database changes, Liquibase vs Flyway.
description: Managing incremental database change has always been one of the most annoying and frustrating aspects of dealing with an traditional relational database, we've finally found a nice way to manage database migrations using Flyway.
author: nick
---


Ugh, databases.
----------

Such is the pain of having to deal with databases and the incremental change that comes as applications evolve, that in the past we actively design around having to store state to avoid them, or when storing is unavoidable, do it in a way that makes it as easy as possible to change things over time, potentially sacrificing nice things like indexes and simple querying.

Obviously using modern ORMs like Hibernate and EclipseLink make defining the schema and interacting with the data easier. But that still leaves the painful job of migrating data from an old version to the latest version. In the most basic example this could be making a small schema change from one application version to the next, or in a more difficult scenario you might need to migrate a number of significant schema differences and the associated data between major application versions.

There are obviously a number of different ways to approach this problem and a number of different tools to help, since we are using Postgres and Java, basically all the various tools are compatible (There is a pretty comprehensive comparison matrix [here](http://flywaydb.org/#features) of all the Java tools in this space). 

What I wanted was a tool that helped me firstly find the difference between what the current production schema had, and what the new production code needed. Secondly I wanted a tool that would let me describe how to migrate the schema and data from V1 -> V2 as easily as possible. The second being by far the more important of the two problems.

The two standouts for this job were liquibase and flyway. I intially dismissed flyway as the documentation / getting-started page didn't grab me and someone had already discussed liquibase with me previously. So I started there.

Liquibase, XML ftw?
----------

So I was immediately a bit suss on liquibase for using XML as its definition language, I think we're all a bit sick of XML generally and I didn't really love the idea of having to learn a new DSL to describe how I wanted to manipulate a database, that is really what we have SQL for.

Either way I pushed on, since I am starting down this migration automation adventure many months into an on-going project, we have an existing system with an existing schema that I needed to import. The liquibase CLI supports connecting to an existing database and deriving its initial model from that, so no problems.

The tooling generated me a big XML file that looked like my schema, but when it came to trying to write some changes myself I found it quite tricky, and had a number of problems the first of which was trying to `ALTER TABLE` and add a `non-null` field. Apparently this isn't straightforward because adding a non-null column to a table that already has existing data violates the non-null constraints of the new column, the workaround is apparently to apply the constraint after the `ALTER TABLE` and `UPDATE..` commands have run and filled the new column with data. Bit shit.

The other problem was writing the XML. As I’m using Hibernate, I’m cheating a bit and generating a schema diff by running hibernate in schema `UPDATE` mode against the production-like schema, which tries its best to transform the schema (without data) from the old to the new. This is not recommended for production usage but works quite well to generate a list of SQL statements to do the migration. But I then had to translate the SQL into Liquibase XML for it to be useful.

At this point I gave up on liquibase, too many problems, not enough value.

Flyway
----------

So after a few weeks of sulking about liquibase not solving all my problems I went back to Flyway. Maybe it is just the way I read things or how the getting started page is structured, but it just seemed like too much effort to get going, however motivated by my failure with liquibase I got on with it, and found what I now consider to the be the perfect solution to this problem (we will see how I feel about that in a few months).


It works in basically the same way as liquibase, you give it a definition of your schema and then any changes made to the DB must be done through the tool. So in my case I had the entire production schema as V1 and then small changes in V2, more changes in V3 etc. The major difference to liquibase is that you can write the migration in SQL not XML.

The migrations can also be written in Java if they are complex enough that it can't be done in SQL, or if you are more proficient at writing logic in Java then in SQL, like me =). My first Java migration was to generate a new Billing record for each row in the User table, according to some simple logic about what kind of user it was. Very simple in Java, would have taken me ages in SQL.

A couple of other added bonuses of the SQL migrations:

-	I can use my Hibernate-sourced schema update SQL, a bit cheaty but handy.
-	Vendor specific SQL, Postgres has support for giving default values when adding new columns that liquibase doesn't support.
-	No new DSL, not having to learn something new is generally a positive.


Summary
----------

If you are like me, and have been putting this problem in the too-hard basket, go look at flyway it solved all my problems, without making me bend over backwards to accommodate it. Now I don't worry about database migration anymore. Amazing.

