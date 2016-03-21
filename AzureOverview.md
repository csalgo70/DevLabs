# Azure Overview (Draft) - by Kannan Sundararajan

One thing to be aware is that Azure is constantly evolving. So things do change at rapid pace and you need be reading on a regular basis to understand the changes and newer offerings.

So lets dive in and start with a very basic question.

## What is Cloud ?

>"A style of computing in which scalable and elastic IT-enabled capabilities are delivered as a service using internet technologies" -- Gartner.

## What are the types of cloud offerings ?

  - IaaS ( Infrastructure as a Service )
      -  In this scenario you run your Virtual Machine (VM) on azure data center. You leverage the hardware / hosting infrastructure provided by azure, but you still maintain the VM in terms of updating the OS with latest patches & your application. You don't have to worry about networking, hardware, computing power as you can buy those based on your needs / scale them.
  - PaaS ( Platform as a Service )
      - Here it adds a layer of abstraction for you. Instead of creating VMs and having to deal with keeping the OS upto-date etc, you can just use the plaform to run your applications. The VMs are automatically created / updated by Azure behind the scene. You can configure so that based on the needs of your application the platform would scale the resources for you.
  - SaaS (Software as a Service)
      - These could be APIs that you use in your application like address verification or validating credit card or provide them as a service to your customers. 

The primary business driver is that it reduces your cost to provision, makes it easy for you to scale / deploy.  Say you are a startup with few developers, you can now focus on Application development as opposed to worrying about hardware infrastruture issues. It also reduces your total cost of ownership. Having said that it still doesn't prevent you with going for a hybrid solutions where you have existing on-premise systems that needs to work with a cloud offering. 

## Azure components

- **Compute** : Here as we saw earlier you have two offerings namely IaaS & PaaS.
   -   **IaaS** - Virtual Machines / Containers
       - In this scenario you create your virtual machines, configure CPU / Memory / Disk & then deploy / run your application. Example would be say you run SQL Server on a VM Or you can choose Containers like Docker to package and run your application.
   -   **PaaS** - Cloud Services ( Web / Worker Roles ) / Web Apps
        - Here you can instead rely on the abstraction layer that cloud services provide to deploy your application by packaging it, and using a **Web role to run a web applicaiton code** or **a background process using a worker role**.  You can specify how many web role / worker role intsances in your package configuration, target environment and it would be managed by **Azure Service Fabric**, so that if one instance goes down it would start another instance in the same data center or you can scale them based on load. Thus you don't have to worry about creating VMs / maintaining them, even though its a dedicated VM under the covers. **Web Apps** provides you even higher level abstraction where you just focus on your application code and don't have to package it certain way, and you can still scale, do source control integration with CI. Similar to worker role here you can use a WebJob. Azure though is moving towards **Azure App Service** model that provides a powerful fully managed plaform for developing **Web Apps, Mobile Apps, API Apps, and Logic Apps**, with built-in DevOps support.
- **Data Storage**
     - Azure provides a variety of data storage options like Relational, Key/Value pair, Blob, NoSQL, Graph, and Column. Once again you have different offerings. You can use **Azure SQL or MySQL with PasS offering** where you don't have to create VMs, install SQL server etc, but just create an Azure SQL instance and use it. If you need **key/value pair**, you can use **table storage or Redis Cache**. You can use **Blob Storage** to store variety of files or use **File Services**. **AzureDocumentDB** is Microsoft's NoSQL option, though you can still use **MongoDB** from the marketplace. As a newer offering you should look at **Data Lakes** if you need massive scalable data storage where you can store data in native format and be able to process them.
- **Messaging** 
    - Azure Service Bus (Relay, Brokered messaging with queues / topics, notification hubs, and event hubs) caters to wide variety of messaging needs for your application and plain azure storage queues with certain limitations like it only supports HTTP / SDK while service bus supports more protocols. When you need two applications across complex network to communicate you could use Relay instead of trying to setup VPNs. Use **Brokered Messaging** when you need enterprise level async communication even when your consumers are not online. Example would be an e-Commerce web site can send the orders to a Service Bus Queue, while a worker role process would process them, validate, notify etc. **Notification hubs** are used to do push notification to several devices be it on iOS or Android. **Event hubs** can be used to ingest data from various devices at scale. Example would be getting sensor data in case of **IOT application**. 
- **Data Processing**
    - Once you store the data, you do want to process them for Analytics, Decision Making, Preidctions, or just moving & tranforming them. Options are Azure SQL DataWarehouse, HDInsight (Hadoop offering), Azure Data Lake Analytics (using U-SQL), Stream Analytics (for realtime data like fraud detection, or data from event hubs) and output them to any of the storage options we discussed earlier. Predictions using Machine Learning to create a model, run the data through the models to create an output. Data Factory for ETL by creating a data pipeline for data transformation, with monitoring available. 
- **Networking**
    - In addition to Virtual Networks you have CDN, Traffic Manager to route request to nearest Data Center in the region or fault tolerence, and now Azure DNS as part of application networking. 
- **Services / APIs**
- **Active Directory**


