---
PermaID: 1000251
Name: SaveChangesAsync
---

# SaveChangesAsync

## Introduction

Asynchronous saving avoids blocking a thread while the changes are written to the database. This can be useful to avoid freezing the UI of a thick-client application. Entity Framework Core provides `DbContext.SaveChangesAsync()` as an asynchronous alternative to `DbContext.SaveChanges()`.


```csharp
public static async Task AddCustomerAsync(string firstName, string lastName, string address)
{
    using (var context = new MyContext())
    {
        var customer = new Customer
        {
            FirstName = firstName,
            LastName = lastName,
            Address = address
        };

        context.Customers.Add(customer);
        await context.SaveChangesAsync();
    }
}
```

 - EF Core does not support multiple parallel operations being run on the same context instance. 
 - You should always wait for an operation to complete before beginning the next operation. 
 - This is typically done by using the await keyword on each asynchronous operation.
