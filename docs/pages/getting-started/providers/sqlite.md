---
permalink: providers-sqlite
---

`Microsoft.EntityFrameworkCore.Sqlite` database provider allows Entity Framework Core to be used with to be used with SQLite. The provider is maintained as part of the [Entity Framework Core](https://github.com/aspnet/EntityFrameworkCore) Project.

#### How to Use SQLite Provider

To use SQLite database provider, the first step is to install [Microsoft.EntityFrameworkCore.Sqlite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/) NuGet package. Let's consider a simple model which contains three entities.

{% include template-example.html %} 
{% highlight csharp %}

public class Customer
{
    public int CustomerId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Address { get; set; }
    public virtual List<Invoice> Invoices { get; set; }
}

public class Invoice
{
    public int Id { get; set; }
    public DateTime Date { get; set; }

    public int CustomerId { get; set; }

    [ForeignKey("CustomerId")]
    public virtual Customer Customer { get; set; }
    public virtual List<InvoiceItem> Items { get; set; }
}

public class InvoiceItem
{
    public int InvoiceItemId { get; set; }
    public int InvoiceId { get; set; }
    public string Code { get; set; }

    [ForeignKey("InvoiceId")]
    public virtual Invoice Invoice { get; set; }
}

{% endhighlight %}

The next step is to create a custom `DbContext` class.

{% include template-example.html %} 
{% highlight csharp %}

public class MyContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Invoice> Invoices { get; set; }
    public DbSet<InvoiceItem> InvoiceItems { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder
            .UseSqlite(@"Data Source=CustomerDB.db;");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Customer>().ToTable("Customers");
        modelBuilder.Entity<Invoice>().ToTable("Invoices");
        modelBuilder.Entity<InvoiceItem>().ToTable("InvoiceItems");
    }
}

{% endhighlight %}

In EF Core, the DbContext has a virtual method called onConfiguring which will get called internally by EF Core, and it will also pass in an optionsBuilder instance, and you can use that optionsBuilder to configure options for the DbContext. 

The optionsBuilder has UseSqlite method; it expects a connection string as a parameter. Once you have a model, you can use [migrations](/migrations) to create a database.

Run the following command in Package Manager Console.

`PM> Add-Migration Initial` 

This command scaffold a migration to create the initial set of tables for your model. When it is executed successfully, then run the following command.

`PM> Update-Database`

It will apply the new migration to the database. Now you can use SQLite database to insert, delete and update data.

{% include template-example.html %} 
{% highlight csharp %}

using (var context = new MyContext())
{
    var customer = new Customer
    {
        CustomerId = 1,
        FirstName = "Elizabeth",
        LastName = "Lincoln",
        Address = "23 Tsawassen Blvd."
    };

    context.Customers.Add(customer);
    context.SaveChanges();
}

{% endhighlight %}

## Limitations

The SQLite provider has some migrations limitations, and mostly these limitations are not EF Core specific but underlying SQLite database engine.

 - The SQLite provider does not support schemas and Sequences. 
 - The SQLite database engine does not support the following schema operations that are supported by the majority of other relational databases.
   - AddForeignKey
   - AddPrimaryKey
   - AddUniqueConstraint
   - AlterColumn
   - DropColumn
   - DropForeignKey
   - DropPrimaryKey
   - DropUniqueConstrain
   - RenameColumn
