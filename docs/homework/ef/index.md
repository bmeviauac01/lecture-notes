﻿# Exercise: Entity Framework

This exercise is optional. You may earn **2+2 points** by completing this exercise.

Use GitHub Classroom to get your git repository. You can find the **invitation link in Moodle**. Clone the repository created via the link. It contains a skeleton and the expected structure of your submission. After completing the exercises and verifying them, commit and push your submission.

!!! warning "Entity Framework _Core_"
    We are using Entity Framework **Core** in this exercise. This is different from Entity Framework used in the seminar exercises; this is a platform-independent technology.

## Required tools

- Windows, Linux, or macOS: All tools are platform-independent, or a platform-independent alternative is available.
- Microsoft SQL Server
    - The free Express version is sufficient, or you may also use _localdb_ installed with Visual Studio
    - A [Linux version](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup) is also available.
    - On macOS, you can use Docker.
- [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms), or you may also use the platform-independent [Azure Data Studio](https://docs.microsoft.com/en-us/sql/azure-data-studio/download) is
- Database initialization script: [mssql.sql](https://raw.githubusercontent.com/bmeviauac01/adatvezerelt/master/docs/db/mssql.sql)
- GitHub account and a git client
- Microsoft Visual Studio 2019 [with the settings here](../VisualStudio.md)
    - When using Linux or macOS, you can use Visual Studio Code, the .NET Core SDK, and [dotnet CLI](https://docs.microsoft.com/en-us/dotnet/core/tools/).
- [.NET Core **3.1** SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1)

    !!! warning ".NET Core 3.1"
        Mind the version! You need .NET Core SDK version **3.1** to solve these exercises.

        On Windows, it might already be installed along with Visual Studio (see [here](../VisualStudio.md#check-and-install-net-core-sdk) how to check it); if not, use the link above to install (the SDK and _not_ the runtime). You need to install it manually when using Linux or macOS.

## Exercise 0: Neptun code

Your very first task is to type your Neptun code into `neptun.txt` in the root of the repository.

## Exercise 1: Database mapping using Code First model and queries (2 points)

Prepare the (partial) mapping of the database using Entity Framework _Code First_ modeling. The Entity Framework Core package is part of the project, so you can start coding. The central class for database access is the DbContext. This class already exists with the name `ProductDBContext`.

1. Map the product entity. Create a new class with the name `DbProduct` with the following code. (The _Db_ prefix indicates that this class is within the scope of the database. This will be relevant in the next exercise.) We rely on conventions as much as possible: use property names that match the column names to make mapping automatic.

    ```C#
    using System.ComponentModel.DataAnnotations.Schema;

    namespace ef
    {
        [Table("Product")]
        public class DbProduct
        {
            [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
            public int ID { get; set; }
            public string Name { get; set; }
            public double Price { get; set; }
            public int Stock { get; set; }
        }
    }
    ```

    Open the source code of class `ProductDbContext` and uncomment the `Products` property.

1. Create a new class with the name `DbVat` for mapping the `VAT` database table similarly as seen before. Do not forget to add a new DbSet property into `ProductContext` with the name `Vat`.

1. Map the Product - VAT connection.

    Add a new get-set property into class `DbProduct` with name `Vat` and type `DbVat`. Use the `ForeignKey` [attribute on this property](https://docs.microsoft.com/en-us/ef/core/modeling/relationships?tabs=data-annotations%2Cdata-annotations-simple-key%2Csimple-key#foreign-key), to indicate the foreign key used to store the relationship ("VatID").

    Create the “other side” of this one-to-many connection from class `DbVat` to `DbProduct`. This should be a new property of type `System.Collections.Generic.List` with name `Products`. (See an example in the link above.)

There are unit tests available in the solution. The test codes are commented out because it does not compile until you write the code. Select the whole test code and use _Edit / Advanced / Uncomment Selection_. You can [run the unit tests in Visual Studio](https://docs.microsoft.com/en-us/visualstudio/test/run-unit-tests-with-test-explorer?view=vs-2019), or if you are using another IDE (e.g., VS Code, or `dotnet cli`), then [run the tests using the cli](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-test). You may update the database connection string in class `TestConnectionStringHelper` if needed.

!!! important "Tests"
    The tests presume that the database is in its initial state. Re-run the database initialization script to restore this state.

    Do **NOT** change the unit tests. You may temporarily alter the unit tests if you need to, but make sure to reset your changes before committing.

!!! danger "If the tests do not compile"
    If the test code does not compile, you may have used slightly different property names. Fix these in **your code and not in the tests**!

!!! danger "`OnConfiguring`"
    You need no connection string in the _DbContext_. The constructor handles the connection to the database. Do not create `OnConfiguring` method in this class!

!!! example "SUBMISSION"
    Upload the changed C# source code.

    Create a screenshot displaying the successfully executed unit tests. You can run the tests in Visual Studio or using `dotnet cli`. Make sure that the screenshot includes the **source code of the DbContext** and the **test execution outcome**! Save the screenshot as `f1.png` and upload as part of your submission!

    If you are using `dotnet cli` to run the tests, make sure to display the test names too. Use the `-v n` command line switch to set detailed logging.

    The image does not need to show the exact same source code that you submit; there can be some minor changes here and there. That is, if the tests run successfully and you create the screenshot, then later you make some **minor** change to the source, there is no need for you to update the screenshot.

## Exercise 2: Repository implementation using Entity Framework (2 points)

!!! note ""
    In the evaluation, you will see the text “imsc” in the exercise title; this is meant for the Hungarian students. Please ignore that.

The Entity Framework DbContext created above has some drawbacks. For example, we need to trigger loading related entities using `Include` in every query, and the mapped entities are bound to precisely match the database schema. In complex applications, the DbContext is frequently wrapped in a repository that handles all peculiarities of the data access layer.

Implement class `ProductRepository` that helps with listing and inserting products. You are provided with a so-called _model_ class representing the product entity, only in a more user-friendly way: it contains the tax percentage value directly. An instance of this class is built from database entities, but represents all information in one instance instead of having to handle a product and a VAT record separately. Class `Model.Product` contains most properties of class `DbProduct`, but _instead of_ the navigation property to `DbVat` it contains the referenced percentage value (`VAT.Percentage`) directly.

Implement the methods of class `ProductRepository.

- `List` shall return all products mapped to instances of `Model.Product`.
- `Insert` shall insert a new product into the database. This method shall find the matching `VAT` record in the database based on the tax percentage value in the model class; if there is no match, it shall insert a new VAT record too!
- Do not change the definition of class `ProductRepository` (do not change the name of the class, nor the constructor or method declarations); only write the method bodies.
- In the repository code use `ProductRepository.createDbContext()` to instantiate the _DbContext_  (do **not** use `TestConnectionStringHelper` here).

!!! example "SUBMISSION"
    Upload the changed C# source code.
