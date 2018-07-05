## Cosmos Document DB Repository by Kannan Sundararajan
```csharp
using Newtonsoft.Json;

    public abstract class BaseDocumentModel
    {
        [JsonProperty(PropertyName = "id")]
        public string Id
        {
            get;
            set;
        }
    }
```

```csharp
public interface IPartitioned
{
    Object PartitionKey { get;  }
}
```

```csharp
public class EventMetadata : BaseDocumentModel, IPartitioned
{
    [JsonProperty(PropertyName = "accountNo")]
    public string AccountNo { get; set; }
      
    //.... additional properties....
    
    [JsonIgnore]
    public Object PartitionKey => AccountNo;
}
```

```csharp
public interface IRepository<T> 
{
    Task<T> CreateAsync(T item);
    Task<T> UpsertAsync(T item);
    Task<T> DeleteAsync(T item);
    Task<IList<T>> FindAsync(Expression<Func<T, bool>> predicate);
}
```

```csharp
  public class CosmosDocumentRepository<T> : IRepository<T> where T : BaseDocumentModel
  { 
      private DocumentClient _client;
      private Uri _databaseUri;
      private Uri _collectionUri;

      public string DatabaseId { get; private set; }
      public string CollectionName { get; private set; }


      public CosmosDocumentRepository(string url, string authKey, string databaseId, string collectionName)
      {
          _databaseUri = UriFactory.CreateDatabaseUri(databaseId);
          _collectionUri = UriFactory.CreateDocumentCollectionUri(databaseId, collectionName);

          DatabaseId = databaseId;
          CollectionName = collectionName;

          _client = new DocumentClient(new Uri(url), authKey);

          _client.CreateDatabaseIfNotExistsAsync(new Database { Id = databaseId });
          _client.CreateDocumentCollectionIfNotExistsAsync(_databaseUri, 
                                                           new DocumentCollection { Id = collectionName });
      }

      public async Task<T> UpsertAsync(T item)
      {
          RequestOptions requestOptions = GetRequestOptions(item);

          Document document = await _client.UpsertDocumentAsync(_collectionUri, item, requestOptions);
          return (T)(dynamic)document;
      }

      public async Task<T> CreateAsync(T item)
      {
          RequestOptions requestOptions = GetRequestOptions(item);

          Document document = await _client.CreateDocumentAsync(_collectionUri, item, requestOptions);
          return (T)(dynamic)document;
      }

      public async Task<T> DeleteAsync(T item)
      {
          RequestOptions requestOptions = GetRequestOptions(item);

          Document document = 
              await _client
                      .DeleteDocumentAsync(UriFactory.CreateDocumentUri(DatabaseId, CollectionName, item.Id), 
                                           requestOptions);
              
          return (T)(dynamic)document;

      }

      public Task<IList<T>> FindAsync(Expression<Func<T, bool>> predicate)
      {
          throw new NotImplementedException();
      }
      
      /// <summary>
      /// Partition key is used to identify the target partition for this request. It must be set on read 
      /// and delete operations for all document requests; 
      /// create, read, update and delete operations for all document attachment requests; and execute 
      /// operation on stored producedures.
      /// For create and update operations on documents, the partition key is optional. When absent, the 
      /// client library will extract the partition key from the document before sending the request to the server.
      /// </summary>
      /// <param name="item">Entity </param>
      /// <returns></returns>
      private RequestOptions GetRequestOptions(T item)
      {
          IPartitioned partitioned = item as IPartitioned;
          RequestOptions requestOptions = null;

          if (partitioned != null)
          {
              requestOptions = new RequestOptions { PartitionKey = new PartitionKey(partitioned.PartitionKey) };
          }

          return requestOptions;
      }
  }
```
