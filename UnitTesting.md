# DevLabs - by Kannan Sundararajan
### Unit Testing 

```C#
[TestClass]
public abstract class BaseUnitTest
{

    [TestInitialize]
    public void Setup()
    {
       SetupContext();
       BecauseOf();
    }

    public virtual void SetupContext() { } // Derived test would override this to setup mocks
    public virtual void BecauseOf() { } // Derived test would invoke a method to test 
}
```

Usage sample

```C#
public class MyControllerBaseUnitTest : BaseUnitTest 
{
      protected IRepository<myEntityClass> MockInMemoryRepository;
      protected IList<myEntityClass> EntityList;
      protected MyController Controller;
      
      public override void SetupContext()
      {
        // Add records here to EntityList to mimick a table in some database or data source
        // ...
        MockInMemoryRepository = new InMemoryRepository<myEntityClass>(EntityList);
        Controller = new MyController(MockInMemoryRepository);
      }
}

public class When_I_Get_By_Id_On_MyController : MyControllerBaseUnitTest
{
  private myEntityClass _returnedEntity;
  private int _id = 1234;
  
   public override void BecauseOf()
   {
      _returnedEntity = Controller.Get(_id);
   }
   
  [TestMethod]
  public void It_Should_Return_The_Requested_Entity()
  {
      Assert.AreEqual(_returnedEntity.Id, _Id);
  }
}
```

This design pattern for unit testing helps reduce repeated clutter in your unit tests. Lot of times for a group of tests you might need the same set of mocks etc. Here using the above pattern you can create a base test where you setup the mocks and inherit from that for you particular testing scenario. Also if you observe each test method has only one assert. So when your test fails you know exactly what failed just by looking at the test name. This improves your developer / team velocity. 
