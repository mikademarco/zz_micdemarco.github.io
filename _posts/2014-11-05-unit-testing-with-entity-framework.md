---
author: micdemarco
comments: true
date: 2014-11-05 14:07:35+00:00
layout: post
slug: unit-testing-with-entity-framework
title: Unit Testing With Entity Framework
tags:
- c#
- unit testing
- visual studio
- entity framework
---

While working on a project involving Integration between Web Services and an MsSQL Database accessed through Entity Framework, I decided to use a Test Driven Approach.

At first I used a direct reference to my data model and wrote integration tests, however this required that all my tests depend upon having a database up and running to connect to.

I wanted to have my tests written to be independent of the Database.  In order to do this I followed the following steps adapted from an [excellent post by Tom FitzMacken](http://www.asp.net/web-api/overview/testing-and-debugging/mocking-entity-framework-when-unit-testing-aspnet-web-api-2):


### 1) Create an interface from your DataContext - IAppDataModel


Create an interface IAppDataModel and extract all DbSet members and SaveChanges:

```csharp
public interface IAppDataModel
{

    DbSet<TableEntityA> TableEntityA { get; set; }
    DbSet<TableEntityB> TableEntityB { get; set; }

    IEnumerable<SomeDataType> GetSomeData(string param1);

    int SaveChanges();
    Task<int> SaveChangesAsync();
    Task<int> SaveChangesAsync(CancellationToken cancellationToken);

}
```
**Listing 1: Interface from your DbContext**

**Tip**: To remove coupling, place the interface in a separate project named Solution.RepositoryInterfaces


### 2) Change your DbContext to implement the interface IAppDataModel


```csharp
 public partial class AppDataModel : DbContext, IAppDataModel
 {
    ...
```
**Listing 2: Implement the interface on your DbContext**


### 3) Change your Business Classes to use the interface IAppDataModel


```csharp
public class SettingsManager : ISettingsManager
    {
        private readonly IAppDataModel _appDataModel;

        public SettingsManager(IAppDataModel appDataModel)
        {
            _elcomDataModel = appDataModel;                      
        }
		
	...
```
**Listing 3: Use an interface instead of a direct reference to your DbContext**

By using an interface, you can choose what implementation you want to pass to the business class depending on your context.  For the test project, you will pass a Test DbContext.  At runtime you will an instance of your EF DbContext.


### 4) In your test project add a TestDbSet class


Add a class that inherits from DbSet and implements IQueryable and IEnumerable

```csharp
//
// Code taken from http://www.asp.net/web-api/overview/testing-and-debugging/mocking-entity-framework-when-unit-testing-aspnet-web-api-2
//

public class TestDbSet<T> : DbSet<T>, IQueryable, IEnumerable<T>
    where T : class
{
    ObservableCollection<T> _data;
    IQueryable _query;

    public TestDbSet()
    {
        _data = new ObservableCollection<T>();
        _query = _data.AsQueryable();
    }

    public override T Add(T item)
    {
        
        _data.Add(item);
        return item;
    }

    public override T Remove(T item)
    {
        _data.Remove(item);
        return item;
    }

    public override T Attach(T item)
    {
        _data.Add(item);
        return item;
    }

    public override T Create()
    {
        return Activator.CreateInstance<T>();
    }

    public override TDerivedEntity Create<TDerivedEntity>()
    {
        return Activator.CreateInstance<TDerivedEntity>();
    }

    public override ObservableCollection<T> Local
    {
        get { return new ObservableCollection<T>(_data); }
    }

    Type IQueryable.ElementType
    {
        get { return _query.ElementType; }
    }

    System.Linq.Expressions.Expression IQueryable.Expression
    {
        get { return _query.Expression; }
    }

    IQueryProvider IQueryable.Provider
    {
        get { return _query.Provider; }
    }

    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
    {
        return _data.GetEnumerator();
    }

    IEnumerator<T> IEnumerable<T>.GetEnumerator()
    {
        return _data.GetEnumerator();
    }
}
```
**Listing 4: TestDbSet Class**


### 5) Create a Test DbContext that implements your IAppDataModel


```csharp
public class TestAppDataModel : IAppDataModel
    {
        public TestAppDataModel()
        {
            TableEntityA = new TestDbSet<TableEntityA>();
            TableEntityB = new TestDbSet<TableEntityB>();
            Seed();
        }
    
        private void Seed()
        {
            this.TableEntityA.Add(new TableEntityA() { Id = 1, Key = "AppKeyA1" });
            this.TableEntityA.Add(new TableEntityA() { Id = 2, Key = "AppKeyA2" });
            this.TableEntityA.Add(new TableEntityA() { Id = 3, Key = "AppKeyA3" });
            
            this.TableEntityB.Add(new TableEntityB() { Id = 1, Key = "AppKeyB1" });
            this.TableEntityB.Add(new TableEntityB() { Id = 2, Key = "AppKeyB2" });
            this.TableEntityB.Add(new TableEntityB() { Id = 3, Key = "AppKeyB3" });
        }

        IEnumerable<SomeDataType> GetSomeData(string param1);
        {
            var ret = from b in TableEntityB
                      select new SomeDataType()
                      {
                         EntityId = b.Id,
                         Param = param1,
                         CreatedUtc = DateTime.UtcNow
                      };
            return ret;
        }

        public int SaveChanges()
        {
            return 0;
        }

        public Task<int> SaveChangesAsync()
        {
            throw new NotImplementedException();
        }

        public Task<int> SaveChangesAsync(CancellationToken cancellationToken)
        {
            throw new NotImplementedException();
        }

        public void Dispose() { }
    }
}
```
**Listing 5: Creating a Test DbContext with a Seed method**


### 6) Use the Test DbContext in the tests


```csharp
[TestClass()]
    public class UserManagerTests
    {
        private IAppDataModel _appDataModel;
        private ISettingsManager _settingsManager;
        
        public UserManagerTests()
        {
            _appDataModel = new TestAppDataModel();
            _settingsManager = new SettingsManager(_appDataModel);
        }
		
	[TestMethod()]
        public void UserManager_GetUserFromEmail_ReturnsUser()
        {
            IUserManager userManager = new UserManager(_appDataModel, _settingsManager);
            string userEmail = "test@office.com";
            var user = userManager.GetUserFromEmail(userEmail);
            Assert.AreEqual(user.strEmail, userEmail, "User loaded successfully");
        }
        ...
```
**Listing 6: Using the TestDbContext**


### 7) Optional - Create a DbSet that generates Identity


After implementing the Test DbContext, I realised that some of my tests were broken because they required an Identity Primary Key to be generated from the database.

In order to get around this I implemented a specially derived class with an overridden Create method for every Entity that required an Identity to be generated by the database:

```csharp
// TestTableEntityBDbSet 

    class TestTableEntityBDbSet : TestDbSet<TableEntityB >
    {
        public override TableEntityB  Add(TableEntityB item)
        {
            item.Id = Local.Count + 1;
            base.Add(item);
            return item;
        }
    }

// TestAppDataModel 

    public class TestAppDataModel : IAppDataModel
    {
        public TestAppDataModel()
        {
            TableEntityA = new TestDbSet<TableEntityA>();
            TableEntityB = new TestTableEntityBDbSet();
            Seed();
        }
        ...
```
**Listing 7: Inheriting from TestDbSet and overriding Create method**


## Conclusion


By defining an interface for your DbContext, it is possible to create an in memory Database Context that implements the same interface and can be used to simulate the physical Database.  This allows you to write Unit Tests that manipulate the Database Context without needing to connect to a physical Database.  While this is great for Unit Testing it must not be assumed that the simulated database will behave exactly as the physical database, and you will still need to perform Integration tests in order to know for certain that your code executes correctly.

For more information and a downloadable project check the [post by Tom FitzMacken](http://www.asp.net/web-api/overview/testing-and-debugging/mocking-entity-framework-when-unit-testing-aspnet-web-api-2).
