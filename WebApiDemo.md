# DevLabs
Web API App Demo [ Draft, but functional with real azure deployment to test ]

Goal of this demo is to write / deploy a Azure Web API using Azure App Service. So that when we invoke the following api 
for an employee by his/her alias
 
http://apiappdemo.azurewebsites.net/api/Employees/jdoe , we return 

```json
{
    "Employee": {
        "EmployeeId": 4,
        "Firstname": "Jane",
        "Lastname": "Doe",
        "Alias": "jdoe"
    },

    "Manager": {
        "EmployeeId": 3,
        "Firstname": "Brian",
        "Lastname": "Flinch",
        "Alias": "bflinch"
    },

    "DirectReports": [{
        "EmployeeId": 5,
        "Firstname": "Kannan",
        "Lastname": "Sundararajan",
        "Alias": "v-kansu"
    }, {
        "EmployeeId": 6,
        "Firstname": "Kevin",
        "Lastname": "Johnson",
        "Alias": "v-kevin"
    }]
}
```

The data is stored in Azure SQL Database. 

| EmployeeId    |	Firstname	    | Lastname	| Alias|Type       |	ManagerId|
| ------------- |:-------------:| ---------:| ----:| ---------:|----------:|
|1	| Big         |	Boss |	bboss| 	F|	NULL |
|2	| Joh |	Braddy |	jbraddy |	F |	1 |
|3	| Brian |	Flinch |	bflinch |	F |	2 |
|4	| Jane |	Doe |	jdoe |	F	| 3 |
|5	| Kannan |	Sundararajan |	kannans |	v |	4 |
|6	| Kevin	| Johnson	| v-kevin |	v |	4 |


```csharp
public class EmployeesController : ApiController
{
    private readonly IReadOnlyRepository<Employee> _repository;
    
    public EmployeesController(IReadOnlyRepository<Employee> repository)
    {
        _repository = repository;
    }

    [Route("api/Employees/{alias}")]
    public async Task<IHttpActionResult> Get(string alias)
    {
        Employee emp = await _repository.FindAsync(e => e.Alias.Equals(alias));

        if(emp == null)
        {
            return NotFound();
        }
        // this is intentionally done here for the demo. UOW would be a better place.
        var manager = await _repository.FindAsync(e => e.EmployeeId == emp.ManagerId);

        var directReports = await _repository.FindAllAsync(e => e.ManagerId == emp.EmployeeId);
        var returnModel = new { Employee = emp, Manager=manager, DirectReports = directReports.ToList() };

        return Ok(returnModel);
    }
}
```
## ApiAppDemo.DataAccess.Abstract

```csharp
public interface IReadOnlyRepository<T>
{
	IQueryable<T> GetAll();
	Task<T> FindAsync(Func<T, bool> predicate);
	Task<IEnumerable<T>> FindAllAsync(Func<T, bool> filter);
}

public interface IDbContext
{
	DbSet<TEntity> Set<TEntity>() where TEntity : class;
}

public class SqlReadOnlyRepository<T> : IReadOnlyRepository<T> where T : class
{
	private readonly IDbContext _dbContext;

	public SqlReadOnlyRepository(IDbContext dbContext)
	{
		_dbContext = dbContext;
	}

	public IQueryable<T> GetAll()
	{
		return _dbContext.Set<T>();
	}

	public async Task<T> FindAsync(Func<T, bool> predicate)
	{
		return await Task.Factory.StartNew(() => { return GetAll().FirstOrDefault(predicate); });
	}

	public async Task<IEnumerable<T>> FindAllAsync(Func<T, bool> filter)
	{
		return await Task.Factory.StartNew(() => { return GetAll().Where(filter).ToArray(); });
	}
}
```
### Dependency Injection using Unity
```csharp
public static class UnityConfig
{
	public static void RegisterComponents()
	{
		var container = new UnityContainer();
 
		container.RegisterType<IDbContext, EmployeeDbContext>();

		container.RegisterType<IReadOnlyRepository<Employee>, SqlReadOnlyRepository<Employee>>();

		GlobalConfiguration.Configuration.DependencyResolver = new UnityDependencyResolver(container);
	}
}
```
