---
permalink: model-relationships
---

In relational databases, a relationship exists between two tables through foreign keys. A Foreign Key is a column or combination of columns that are used to establish and enforce a link between the data in those two tables. Entity framework Core supports three types of relationships;

 - One-to-Many
 - One-to-One
 - Many-to-Many (not supported yet in EF Core)

### One-to-Many Relationship

In a one-to-many relationship, each table has a primary key that uniquely defines each row within the table. The easiest way to configure a one-to-many relationship is by convention. EF Core will create a relationship if an entity contains a navigation property. Therefore, the minimum required for a relationship is the presence of a navigation property in the principal entity.

{% include template-example.html %} 
{% highlight csharp %}

public class Author
{
    public int AuthorId { get; set; }
    public string Name { get; set; }
    public List<Book> Books { get; set; }
}

public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public int AuthorId { get; set; }
    public Author Author { get; set; }
}

{% endhighlight %}

The `Author` class contains a **Books** navigation property which is a list of Book objects, while the `Book` class also has a navigation property **Author**. Most of the time, one-to-many relationships in an Entity Framework Core model follow conventions and require no additional configuration. Now when you run the migration, you will see the following code in migration file which will create the database.

{% include template-example.html %} 
{% highlight csharp %}

protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.CreateTable(
        name: "Authors",
        columns: table => new
        {
            AuthorId = table.Column<int>(nullable: false)
                .Annotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn),
            Name = table.Column<string>(nullable: true)
        },
        constraints: table =>
        {
            table.PrimaryKey("PK_Authors", x => x.AuthorId);
        });

    migrationBuilder.CreateTable(
        name: "Books",
        columns: table => new
        {
            BookId = table.Column<int>(nullable: false)
                .Annotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn),
            AuthorId = table.Column<int>(nullable: false),
            Title = table.Column<string>(nullable: true)
        },
        constraints: table =>
        {
            table.PrimaryKey("PK_Books", x => x.BookId);
            table.ForeignKey(
                name: "FK_Books_Authors_AuthorId",
                column: x => x.AuthorId,
                principalTable: "Authors",
                principalColumn: "AuthorId",
                onDelete: ReferentialAction.Cascade);
        });

    migrationBuilder.CreateIndex(
        name: "IX_Books_AuthorId",
        table: "Books",
        column: "AuthorId");
}

{% endhighlight %}

Now, look at the database in SQL Server Object Explorer.

<img src="{{ site.github.url }}/images/relationships1.png"> 

 - The primary key in Authors table is AuthorId, while in Books table the primary key is BookId. 
 - The AuthorId column in the Books table is a Foreign Key (FK), linking a book to its author. 
 - A book is a dependent entity in the relationship, and Author becomes the principal entity. 
 - Using foreign keys, you can link one author row in the database to many book rows. 
 
Now if your model does not follow the default conventions, the Fluent API can be used to configure the correct relationship between entities. 

 - When configuring relationships with the Fluent API, you will use the Has/With pattern. 
 - The **"Has"** side of the pattern is represented by the `HasOne` and `HasMany` methods. 
 - The **"With"** side of the relationship is represented by the `WithOne` and `WithMany` methods.

{% include template-example.html %} 
{% highlight csharp %}

protected override void OnModelCreating(Modelbuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
        .HasMany(a => a.Books)
        .WithOne(b => b.Author);
}

{% endhighlight %}

### One-to-One Relationship

In a one-to-one relationship, each row of data in one table is linked to zero or one row in the second table. 

{% include template-example.html %} 
{% highlight csharp %}

public class Author
{
    public int AuthorId { get; set; }
    public string Name { get; set; }
    public AuthorBiography Biography { get; set; }
}
+-
public class AuthorBiography
{
    public int AuthorBiographyId { get; set; }
    public string Biography { get; set; }
    public DateTime DateOfBirth { get; set; }
    public string PlaceOfBirth { get; set; }
    public string Nationality { get; set; }
    public int AuthorId { get; set; }
    public Author Author { get; set; }
}

{% endhighlight %}

The `Author` class contains a **Biography** navigation property and the `AuthorBiography` class has a navigation property **Author**. Now when you run the migration, you will see the following code in migration file which will create the database.

{% include template-example.html %} 
{% highlight csharp %}
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.CreateTable(
        name: "Authors",
        columns: table => new
        {
            AuthorId = table.Column<int>(nullable: false)
                .Annotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn),
            Name = table.Column<string>(nullable: true)
        },
        constraints: table =>
        {
            table.PrimaryKey("PK_Authors", x => x.AuthorId);
        });

    migrationBuilder.CreateTable(
        name: "AuthorBiographies",
        columns: table => new
        {
            AuthorBiographyId = table.Column<int>(nullable: false)
                .Annotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn),
            AuthorId = table.Column<int>(nullable: false),
            Biography = table.Column<string>(nullable: true),
            DateOfBirth = table.Column<DateTime>(nullable: false),
            Nationality = table.Column<string>(nullable: true),
            PlaceOfBirth = table.Column<string>(nullable: true)
        },
        constraints: table =>
        {
            table.PrimaryKey("PK_AuthorBiographies", x => x.AuthorBiographyId);
            table.ForeignKey(
                name: "FK_AuthorBiographies_Authors_AuthorId",
                column: x => x.AuthorId,
                principalTable: "Authors",
                principalColumn: "AuthorId",
                onDelete: ReferentialAction.Cascade);
        });

    migrationBuilder.CreateIndex(
        name: "IX_AuthorBiographies_AuthorId",
        table: "AuthorBiographies",
        column: "AuthorId",
        unique: true);
}

{% endhighlight %}

Now, look at the database in SQL Server Object Explorer.

<img src="{{ site.github.url }}/images/relationships2.png"> 

Now if your model does not follow the default conventions, the Fluent API can be used to configure the correct relationship between entities.
{% include template-example.html %} 
{% highlight csharp %}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
        .HasOne(a => a.Biography)
        .WithOne(b => b.Author);
}

{% endhighlight %}