---
permalink: querying-data-stored-procedure
---

The Entity Framework allows you to use stored procedures to perform predefined logic on database tables. Raw SQL queries can be used to execute a stored procedure. 

Here is a simple stored procedure, it will return all the records from Customers table when executed.

{% include template-example.html %} 
{% highlight csharp %}

IF NOT EXISTS (SELECT * FROM sys.objects WHERE object_id = 
   OBJECT_ID(N'[dbo].[GetAllCustomers]') AND type in (N'P', N'PC'))

BEGIN

   EXEC dbo.sp_executesql @statement = N'
   CREATE PROCEDURE [dbo].[GetAllCustomers]
   AS
   SELECT * FROM dbo.Customers
   '
END
GO

{% endhighlight %}

In EF Core, you can execute stored procedures using `FromSql()` method.

{% include template-example.html %} 
{% highlight csharp %}

using (var context = new MyContext())
{
    var customers = context.Customers
        .FromSql("EXECUTE dbo.GetAllCustomers")
        .ToList();
}

{% endhighlight %}

You can also pass parameters when executing stored procedures.

{% include template-example.html %} 
{% highlight csharp %}

IF NOT EXISTS (SELECT * FROM sys.objects WHERE object_id = 
   OBJECT_ID(N'[dbo].[GetCustomer]') AND type in (N'P', N'PC'))

BEGIN

   EXEC dbo.sp_executesql @statement = N'
   CREATE PROCEDURE [dbo].[GetCustomer]
   @CustomerID int
   AS
   SELECT * FROM dbo.Customers 
   WHERE CustomerID = @CustomerID
   '
END
GO

{% endhighlight %}

It will return a specific record from Customers table based on CustomerID passed as a parameter.

{% include template-example.html %} 
{% highlight csharp %}

using (var context = new MyContext())
{
    var customer = context.Customers
        .FromSql("EXECUTE dbo.GetCustomer 1")
        .ToList();
}

{% endhighlight %}

You can also pass a parameter value using C# string interpolation.

{% include template-example.html %} 
{% highlight csharp %}

using (var context = new MyContext())
{
    int customerId = 1;
    var customer = context.Customers
        .FromSql($"EXECUTE dbo.GetCustomer {customerId}")
        .ToList();
}

{% endhighlight %}