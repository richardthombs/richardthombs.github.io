---
layout: post
title:  "SQL Azure gotchas"
date:   2014-09-08 10:29:00
categories: azure
---

# 1. Inserts will fail on tables without clustered indexes

When we first starting examining the possibilities of running on SQL Azure, we created an empty copy of our database, pointed our website at it and fired it up. The first few pages worked fine, in fact most of the application did, but some didn't and instead failed with SQL errors.

It turns out that although you can create a table without a clustered index, SQL Azure won't let you insert data into it until you have one defined.

We had a few tables in our database where there wasn't a clustered index defined and any attempt to insert data into them was failing with the error "Tables without a clustered index are not supported in this version of SQL Server. Please create a clustered index and try again."

For example:

{% highlight sql %}
-- This works fine
create table Test
(
  Value int not null
)
{% endhighlight %}

{% highlight sql %}
-- But this will fail
insert into Test values(1)
{% endhighlight %}

This is obviously very easy to fix... create a clustered index and all is well:

{% highlight sql %}
-- Until you create a clustered index
create clustered index IX_Value on Test (Value)

-- Inserts now work!
insert into Test values(1)
{% endhighlight %}

# 2. SELECT INTO is not supported

So fixing the lack of clustered indexes in your database is pretty easy, but there is one scenario that is much more complex: `SELECT INTO` statements, which are essentially creating a new table on the fly and then inserting rows into it. Here there is no opportunity for you to specify what indexes to create on the new table and so, Microsoft does not support SELECT INTO at all.

Eg:

{% highlight sql %}
select Value into #Temp from Test
{% endhighlight %}

Then you will get a pretty blunt error: Statement 'SELECT INTO' is not supported in this version of SQL Server.

Like most people, we use temporary tables to speed up complex queries. Many of these queries are dynamically generated by code, and so knowing the structure of the table beforehand is hard.

Our solution has been to redesign our application so that we are no longer using SELECT INTO.

If you do know the structure of the table you will be creating, then you can also pre-create it and [use INSERT INTO instead](http://azure.microsoft.com/blog/2010/05/04/select-into-with-sql-azure).

Sadly Microsoft doesn't seem too eager to remedy this issue, but you can vote on the Azure UserVoice site for [SELECT INTO support](http://feedback.azure.com/forums/217321-sql-database/suggestions/1410637-add-support-for-select-into-for-temp-tables-in)