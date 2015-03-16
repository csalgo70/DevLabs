# DevLabs
### Builder Pattern

```C#
public class JsonTransformBuilder<T> where T : class
{
	private readonly JsonTransactionTransformer<T> _transformer;

	private JsonTransformBuilder()
	{
		_transformer = new JsonTransactionTransformer<T>();
	}

	public static JsonTransformBuilder<T> Default()
	{
		return new JsonTransformBuilder<T>();
	}

	public JsonTransformBuilder<T> SetMappings(TransactionMap mappings)
	{
		_transformer.PropertyMapping = mappings;
		return this;
	}

	public JsonTransformBuilder<T> MapProperty(string jsonProperty, string toModelProperty)
	{
		_transformer.PropertyMapping.Mappings.Add(jsonProperty, toModelProperty);

		return this;
	}


	public JsonTransformBuilder<T> MapAttribute(string jsonProperty, string toModelProperty, bool isValueEncrypted = false)
	{
		_transformer
			.PropertyMapping
			.AttributeMappings
			.Add(new AttributeMap
			{
				JsonName = jsonProperty,
				Name = toModelProperty,
				IsValueEncrypted = isValueEncrypted
			});

		return this;
	}


	public JsonTransactionTransformer<T> Build()
	{
		return _transformer;
	}

}
```

This would enable fluent code to build the transformer that would convert from a JSON to a C# POCO

```C#
 var payoutTransformer = JsonTransformBuilder<PayoutTransaction>
                .Default()
                .SetMappings(mappings)
                .Build();
```

or 

```C#
var payoutJsonTransformer = 
                JsonTransformBuilder<PayoutTransaction>
                .Default()
                .MapProperty(jsonProperty: "id", toModelProperty: "Id")
                .Build();
```
