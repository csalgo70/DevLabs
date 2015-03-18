# DevLabs
###Repository pattern in use


```C#
public interface IRepository<out T>
{
    IQueryable<T> GetAll();
}
```

If you are using Entity Framework lets define an interface for the DbContext 

```C#
public interface IDbContext
{
    DbSet<TEntity> Set<TEntity>() where TEntity : class;
}
```

Now Open your dbname.Context.tt file and update this line to derive from the IDbContext 
```C# 
<#=Accessibility.ForType(container)#> partial class <#=code.Escape(container)#> : DbContext,IDbContext
```
What the above would do is generate your dbname.Context.cs class that defines your entities class addtionally derving from **IDbContext**

```C#
public partial class MyDbEntities : DbContext, IDbContext,
```

Now comes the SqlRepository

```C#
public class SqlRepository<T> : IRepository<T> where T : class
{
    private readonly IDbContext _dbContext;

    public SqlRepository(IDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public IQueryable<T> GetAll()
    {
        return _dbContext.Set<T>();
    }
 }
```

Now you can use dependency injection using either Unity or others like ninject to map your repository to a SQL repository

```C#
public static IUnityContainer RegisterComponents()
{
     var container = new UnityContainer();

      container.RegisterType<IDbContext, MyDbEntities>();

     container.RegisterType<IRepository<myEntityClass>, SqlRepository<myEntityClass>>();

     GlobalConfiguration.Configuration.DependencyResolver = new UnityDependencyResolver(container);

      return container;
}
```

Now with the contoller below it automatically gets passed in the SqlRepository based on DI. You controller class itself is unaware of the fact if its talking to SQL database or Azure tables or some other data source with clean seperation of concerns. At anytime you can change your backend data source and not have to touch your controllers ! 

```C#
public class MyController : ApiController 
{
    private IRepository<myEntityClass> _repository;
    public MyController(IRepository<myEntityClass> repository)
    {
        _repository = repository;
    }
}
```

Lets look how the above design helps with unit testing. You can now create an InMemoryRepository which would mimick the rows in a table. During unit testing you can now pass this in to the above controller and thus it won't know its not talking to database but an in memory repository allowing you to fully unit test your controller without any database or external data source.

```C#
public class InMemoryRepository<T> : IRepository<T> where T : class
{
	private readonly IQueryable<T> _entityList;

	public InMemoryRepository(IEnumerable<T> entitList)
	{
		if (entitList == null)
		{
			throw new ArgumentNullException("entitList");
		}

		_entityList = entitList.AsQueryable();
	}

	public IQueryable<T> GetAll()
	{
		return _entityList;
	}
}
```
