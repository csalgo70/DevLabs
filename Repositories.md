# DevLabs
###Repository pattern in use

```C#
public interface IDbContext
{
    DbSet<TEntity> Set<TEntity>() where TEntity : class;
}
```

```C#
public interface IRepository<out T>
{
    IQueryable<T> GetAll();
}
```

If you are using Entity Framework open your dbname.Context.tt file and update this line to derive from the IDbContext 
```C# 
<#=Accessibility.ForType(container)#> partial class <#=code.Escape(container)#> : DbContext, IDbContext
```
What the above would do is generate your dbname.Context.cs class that defines your entities class addtionally derving from **IDbContext**

```C#
public partial class MyDbEntities : DbContext, IDbContext,
```

```C#

Now comes the SqlRepository

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
