---
permalink: querying-data-global-filter
---
## Introduction

Entity Framework Core 2.0 introduces global query filters that can be applied to entities when a model is created. It makes it easier to build multi-tenant applications and support soft deleting of entities.

 - It allows us to specify a filter in the model level that is automatically applied to all queries that are executed in the context of the specified type. 
 - It means that entity framework automatically adds the filter in the where clause before executing the LINQ queries. 
 - Usually, Global query filters are applied in `OnModelCreating` method of context. 

Here is a simple model which contains only one entity i.e., Customer

{% include template-example.html %} 
{% highlight csharp %}

public class Customer
{
    public int CustomerId { get; set; }
    public string Name { get; set; }
    public bool IsDeleted { get; set; }
}

{% endhighlight %}

Global query filters are defined when database context builds the model. So we need to configure the global query filter in `OnModelCreating` method of context class, using `HasQueryFilter` method, and we can apply the global filter on the entity type.

{% include template-example.html %} 
{% highlight csharp %}

public class MyContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(@"Data Source=(localdb)\ProjectsV13;Initial Catalog=CustomerDB;");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Customer>().HasQueryFilter(c => !c.IsDeleted);
    }
}

{% endhighlight %}

The expression passed in HasQueryFilter method is automatically applied to any LINQ queries for Customer Type. We have four records in the Customers table in the database.

<img src="{{ site.github.url }}/images/global-filters.png">

One record is soft deleted already, and the `IsDeleted` column contains a **True** value for that record. Now if we retrieve all the all the customers from the database using the LINQ query.

{% include template-example.html %} 
{% highlight csharp %}

using (var context = new MyContext())
{
    var customers = context.Customers.ToList();
}

{% endhighlight %}

We will get only three records, but in the database, we have four records, Because the global filter has filtered records and returns only those records whose `IsDeleted` column contains **False** value.

In some cases, we do not need these filters to be applied, we can also disable filter for individual LINQ queries using the `IgnoreQueryFilters()` method.

{% include template-example.html %} 
{% highlight csharp %}

using (var context = new MyContext())
{
    var customers = context.Customers
        .IgnoreQueryFilters().ToList();
}

{% endhighlight %}

Now the filter won't be applied, and you will get all the records.