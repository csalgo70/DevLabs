#Evaluate Azure DocumentDB for Fort  

Fraud Review Tool is a Web based manual review tool that shows transaction details of a purchase, to a Fraud Review Agent.

The transactions are received as JSON. These need to be stored in a data store so that we can provider agents ability to Check Out / Update / Check In documents akin to a source code control repository. 

Thus we are currently evaluating Document DB.

##Data

FORT tool receives its data from Modern Risk API as JSON data. This JSON represents the original transaction in review along with additional information like Account details, and Payment Instrument. So far based on observations of existing data from legacy system, the size of this JSON is well within the **max document size of 256 KB** of azure document db.

##Database Operations

Current thought is that we would have a  **Manual Review database** which would have a **Risk Mod Collection** and we need to support the following operations. Another idea is to have **one collection per** tenant since we essentially **want to have a sorted queue by tenant**.

#### <i class="icon-file"></i> Create a document

We would poll Modern Risk API and create new documents in the database based on how many records we get back from the API since the last poll. 

Based on anecdotal evidence, we would **only get 1 JSON per minute** to begin with for at least modern payout scenario. **Worst case is not known to me at this point**.

#### <i class="icon-upload"></i>Update document
  > - Check-out document 
  > - Update some fields in a Checked-out document
  > - Check-in document

#### <i class="icon-download"></i> Retrieve Document

Retrieve the document based on some query filter.

#### <i class="icon-trash"></i> Delete a document

Eventually after manual decisions are made, these documents would be moved out of this collection / deleted. 

##Queries

Need to support the following queries in an efficient manner as the response times are very important for us.  

- Get the top 1 document ordered by SLA date field. 
    > - Filter criteria based on tenant or agent name
    > - Order By is not supported but we could use Store Procedure to achieve it. Performance issues due to full collection scan?  
    
- Get a list of transactions (brief data) based on filter criteria
    > - Select is flexible enough to meet our needs along with projections 
- Get a transaction document given a key field value  


##Pending Investigations
 - Performance based on say 10,000 documents ( currently 5000 is daily avg. )
 - Encryption / Decryption 
 - Modelling 
   - Wrap the original payload into document payload with checkout status / permissions.
   - We can also model Account Info / Payment Info as separate collection as its unique by ID.
