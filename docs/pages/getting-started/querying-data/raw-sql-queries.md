---
permalink: querying-data-raw-sql-queries
---

## Introduction

Entity Framework Core allows you to execute raw SQL queries directly against the database where you cannot use LINQ to represent the query if the generated SQL is not efficient enough. EF Core provides the `DbSet.FromSql()` method to execute raw SQL queries.

{% include template-example.html %} 
{% highlight csharp %}

using (var context = new MyContext())
{
    var customers = context.Customers
        .FromSql("SELECT * FROM dbo.Customers")
        .ToList();
}
{% endhighlight %}

In the above example, the raw SQL query is specified in the `FromSql()` which will return all the records from the Customers table and transform into Customer entities.

#### Passing parameters

As with any API that accepts SQL, it is important to parameterize any user input to protect against a SQL injection attack. The `DbSet.FromSql()` method also supports parameterized queries using string interpolation syntax in C#.

{% include template-example.html %} 
{% highlight csharp %}

using (var context = new MyContext())
{
     var customers = context.Customers
         .FromSql("Select * from dbo.Customers where LastName = '{0}'", firstName)
         .ToList();
}

{% endhighlight %}

#### LINQ Operators

You can also use LINQ Operators after a raw query using `FromSql()` method.

{% include template-example.html %} 
{% highlight csharp %}

using (var context = new MyContext())
{

    var customers = context.Customers
        .FromSql("Select * from dbo.Customers where FirstName = 'Andy'")
        .OrderByDescending(c => c.Invoices.Count)
        .ToList();

}

{% endhighlight %}