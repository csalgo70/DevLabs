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
Now you can use dependency injection using either Unity or others like ninject.

```C#
public static IUnityContainer RegisterComponents()
{
     var container = new UnityContainer();

      // don't directly read from App.config in your controller use interface & pass it in - IOC pattern
      container.RegisterType<IAppSettings, ServiceAppSettings>(); 
      container.RegisterType<IServiceAppSettings, ServiceAppSettings>();

      container.RegisterType<IRepository<myEntityClass>, SqlRepository<myEntityClass>>();

      GlobalConfiguration.Configuration.DependencyResolver = new UnityDependencyResolver(container);

      return container;
}
```
