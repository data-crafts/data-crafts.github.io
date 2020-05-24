---
layout: post
title:  "Data enrichment with Apache NiFi and MongoDB"
author: abdelkrim
categories: [NiFi, MongoDB, Use Case, Tutorial]
image: assets/images/7-1.jpg
description: "Data enrichment with Apache NiFi and MongoDB"
toc: true
comments: true
---
Data enrichment refers to processes used to enhance and refine raw data to make it a valuable business asset. It’s a common use case when working on data ingestion and flow management. Enrichment is required when data contains references (ex. IDs) rather than the actual information. These references are used to query a data source (known as reference table) to bring more details, context or information (ex. location, name, etc). This enrichment data source can be a database, a file or an API. Data enrichment is usually done in batch using join operations. However, doing the enrichment on data streams in realtime is more interesting.

##	Enrichment in Apache NiFi

Enrichment was not natively supported in NiFi 1.2 or older version. There were few workarounds to do it but they are not performant neither integrated. This is because NiFi is a data flow tool. In Data Flow logic, each flow file is an independent item that can be processed independently. Data enrichment involves correlating and joining at several data sources which is not the sweet-spot for NiFi.

Starting from NiFi 1.3, it’s possible to do data enrichment with a set of new processors ([LookupAttribute](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi/nifi-standard-nar/1.11.4/org.apache.nifi.processors.standard.LookupAttribute/index.html) and [LookupRecord](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi/nifi-standard-nar/1.11.4/org.apache.nifi.processors.standard.LookupRecord/index.html)) and Lookup services such as [SimpleKeyValueLookupService](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi/nifi-lookup-services-nar/1.11.4/org.apache.nifi.lookup.SimpleKeyValueLookupService/index.html) and [MongoDBLookupService](https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi/nifi-mongodb-services-nar/1.11.4/org.apache.nifi.mongodb.MongoDBLookupService/index.html). LookupAttribute queries a data source and add the query result as an attribute where LookupRecord add it to the content. Defining what data source to query is the role of the lookup service.

This article explains how we can use these new features to do data enrichment in realtime. It first defines a simple use case that will be used through this article in order to explain the concepts. Later, it uses this example to explore the different possible data enrichment scenarios.

## Realtime data ingestion for retail data

Let’s consider a retailer having a set of stores in different French cities (Paris, Nice, etc). Each store sends transactions events in realtime to a central application that monitor the status of the stores. This events tells the application when a product has been sold and specifies the number of sold units. To optimise data transfert, ID are used when possible. For instance, id_product is used instead of the product name, and id_store is used instead of store_name.

Based on this event stream, we can imagine several use cases such as stock management or monitoring. For this article, let’s try to optimize the supply chain through a geographical dashboard that shows all the stores on a map and highlight the ones that have products running out of stock soon. To build such dashboard, we can store the data in Solr and build a Banana dashboard on top of it. The issue is that event that stores send us doesn’t have any information on store locations.

```js
{
	"created_at" : "Thu Sep 28 08:08:09 CEST 2017",
	"id_store" : 4,
	"event_type" : "store capacity",
	"id_transaction" : 1009331737896598289,
	"id_product" : 889,
	"value_product" : 45
}
```

This JSON tells us that 45 units of product 889 has been sold in store 4. Details of store 4 such as city, adresse, capacity, etc are available on another data source. We need to enrich these events in realtime to feed our dashboard.

## Data generation

First, let’s generate dummy data to simulate events coming from various stores. For this, we use a GenerateFlowFile processor with the following configuration.

![photo]({{site.baseurl}}/assets/images/7-2.png)

## Lookup Services

Enrichment processors leverages lookup services. A lookup service can be seen as a Key-Value service that a processor queries to get the value associated with a key. Currently, there are 7 available lookup services in NiFi (NiFi 1.4).

![photo]({{site.baseurl}}/assets/images/7-3.png)

PropertiesFileLookupService, SimpleCsvFileLookupService and IPLookupService are file-based lookup services. Reference data should be sitting in a file (CSV,XML, etc) that NiFi uses to match a value to a key. ScriptedLookupService uses a script (Python, Ruby, Groovy, etc) to generate a value corresponding to a key. The SimpleKeyValueLookupService stores the key-value pairs in NiFi directly. It is very convenient to use if your reference data is not too big and don’t evolve too much. MongoDBLookupService queries a MongoDB collection to get documents corresponding to a key. Several other interesting lookup services exists also in newer NiFi versions such as HBase, Elasticsearch, Kudu and Couchbase. In this blog, we will use SimpleKeyValueLookupService and MongoDBLookupService.

## SimpleKeyValueLookupService with LookupRecord and LookupAttribute

As a first step of data enrichment, let’s add a new SimpleKeyValueLookupService by clicking on Configure -> Controller Services -> ‘+’. Search for KeyValue and click on add

![photo]({{site.baseurl}}/assets/images/7-4.png)

After adding the SimpleKeyValueLookupService, click on configure and populate it with reference data. Here, I added the city of each one of my stores. Store 1 is Paris, store 2 is in Lyon, and so on.

![photo]({{site.baseurl}}/assets/images/7-5.png)

### Enrichment with LookupAttribute

The easiest way for data enrichment is to use LookupAttribute. For each flow file, LookupAttribute queries the lookup service and adds the query result as an attribute to the flow file. In our scenario, the LookupAttribute uses the value of the attribute id_store as a key and queries the lookup service. The returned value is added as the 'city' attribute. To make this work, the flow files should have an attribute 'id_store' before entering the lookup processor. Currently, this information is only in the content. We need to use an EvaluateJsonPath to get this information from the content and add it as an attribute.

![photo]({{site.baseurl}}/assets/images/7-6.png)

Next add a LookupAttribute with the following configuration:

![photo]({{site.baseurl}}/assets/images/7-7.png)

The final flow looks as the following. NiFi generates a random event, gets the id_store from the content and add as an attribute. Then, this attribute is used to query the lookup service and add the corresponding city name as a new attribute.

![photo]({{site.baseurl}}/assets/images/7-8.png)

To verify that our enrichment is working, let’s see the attribute of a flow file after the EvaluateJsonPath and then the LookupAttribute:

![photo]({{site.baseurl}}/assets/images/7-9.png)
![photo]({{site.baseurl}}/assets/images/7-10.png)

Once the city name added as an attribute, we can use it with several processors. As an example, we can route data based on city name to two relations, North and South. This is very useful to manage data when it’s inside NiFi. However, attributes don’t go beyond NiFi and are not send to the destination system. Only the content does. So if we need the enriched data to be stored in Solr to build our dashboard, we should add the city name to the content of flow files. This is where LookupRecord is useful.

### Enrichment with LookupRecord

LookupRecord belongs to the record based processors that has been added in NiFi 1.2. Record based processors use Record Reader/Writers to guarantee efficient serialization/deserialization mechanisms across NiFi. This makes easier the integration with a Schema Registry for better governance. So let’s add a LookupRecord and configure it as following:

- Record Reader: Create a new JSONTreeReader and configure it. For the sake of simplicity, we won’t use any schema registry here. Choose “Use schema text property” as a “Schema Access Strategy” and add the following schema to the “Schema Text” property.

```js
{
	"namespace": "nifi",
	"name": "store_event",
	"type": "record",
	"fields": [   
		{
			"name": "created_at", 
			"type”: ”string” 
		},   
		{ 
			"name": "id_store", 
			"type": "int" 
		},   
		{
			"name": "event_type",
			"type": "string" 
		},
		{
			"name": "id_transaction", 
			"type": "string" 
		},
		{
			"name": "id_product", 
			"type": "int" 
		},
		{ 
			"name": "value_product",
			"type": "int" 
		}
	]
}
```

![photo]({{site.baseurl}}/assets/images/7-11.png)

- Record Writer: create a new JsonRecordSetWriter and configure it. Set the different attributes as follow and use this schema for the Schema Text property:

```js
{
	"namespace": "nifi",
	"name": "store_event",
	"type": "record",
	"fields": [   
		{
			"name": "created_at",
			"type": "string" 
		},   
		{ 
			"name": "id_store", 
			"type": "int" 
		},   
		{ 
			"name": "event_type", 
			"type": "string" 
		},   
		{ 
			"name": "id_transaction", 
			"type": "string" 
		},   
		{
			"name": "id_product", 
			"type": "int" 
		},   
		{ 
			"name": "value_product", 
			"type": "int" 
		},   
		{ 
			"name": "city", 
			"type": "string" 
		}
	]
}
```

![photo]({{site.baseurl}}/assets/images/7-12.png)

Note that the writer schema is slightly different from the reader schema. Indeed, it has an additional field ‘city’ that the processor will populate when doing enrichment.

To finalize the configuration of the lookup processor, add a custom property “key” and set it to the JSON path of the field used for the lookup. Here, it’s the store ID so Key = /id_store. Result RecordPath tells the processor where to store the retrieved value. Finally, route to ‘matched’ or ‘unmatched’ strategy tells the processor what to do after the lookup.

![photo]({{site.baseurl}}/assets/images/7-13.png)

Now, connect all the processors like the below picture. LookupRecord receives random events from GenerateFlowFile and do the lookup with the Key/Value service. The name of the city is added to the event JSON to be sent to Solr with PutSolrContentStream processor. To reduce indexing overhead and achieve better performance when writting to Solr, let’s merge the enriched JSON events into groups of 100 events before pushing them to Solr.

![photo]({{site.baseurl}}/assets/images/7-14.png)

To check that our enrichment is working, let’s see the content of flow files using the data provenance feature.

![photo]({{site.baseurl}}/assets/images/7-15.png)

First of all, you can notice that LookupRecord is adding an attribute called avro.schema. This is due to the used write strategy. It’s not useful here but just wanted to highlight this. By using a Schema Registry, we can add the name of the schema only.

Let’s see the content of a the flow file now. As you can see, a new field “city” is added to the JSON. Here the city is Toulouse since the Store ID is 4. It’s worth noting that it is possible to write the file in other format (Avro for instance) to have enrichment and conversion in one step.

![photo]({{site.baseurl}}/assets/images/7-16.png)

## Using MongoDBLookupService with LookupRecord
In the previous section, we showed how to use LookupRecord and LookupAttribute to enrich the content/metadata of a flow file with a SimpleKeyValueLookup Service. Using this lookup service helped us implement an enrichment scenario without deploying any external system. This is perfect for scenarios where reference data is not too big and don’t evolve too much. However, managing entries in the SimpleKV Service can become cumbersome if our reference data is dynamic or large.

Fortunately, NiFi 1.4 introduced a new interesting Lookup Service with NIFI-4345 : MongoDBLookupService. This lookup service can be used in NiFi to enrich data by querying a MongoDB store in realtime. With this service in hand, our reference data can live in a MongoDB and can be updated by external applications. In this section, we describe how we can use this service to implement the same retail scenario. We will use a hosted MongoDB (BDaaS) on [MLab](https://mlab.com/) to host refernce data. I created a database “bigdata” and added a collection “stores” in which I inserted 5 documents.

![photo]({{site.baseurl}}/assets/images/7-17.png)

Each Mongo document contains information on a store as described below:


```js
{
	"id_store" : 1,
	"address_city" : "Paris",
	"address" : "177 Boulevard Haussmann, 75008 Paris",
	"manager" : "Jean Ricca",
	"capacity" : 464600
}
```

The complete database looks like this:

![photo]({{site.baseurl}}/assets/images/7-18.png)

Let’s use the same flow constructed in the previous section. The only difference is using a MongoDBLookupService instead of SimpleKVLookupService. The configuration of the LookupRecord processor looks like this:

![photo]({{site.baseurl}}/assets/images/7-19.png)

Now let’s see configure the MongoDBLookupService as follows:

- **Mongo URI:** the URI used to access the MongoDB database in the format mongodb://user:password@hostname:port
- **Mongo Database Name :** the name of your database. It’s bigdata in our case
- **Mongo Collection Name :** the name of the collection to query for enrichment. It’s stores in our case
- **SSL Context Service and Client Auth :** use your preferred security options
- **Lookup Value Field :** the name of the field you want the lookup service to return. For us, it’s address_city since we are looking to enrich events with the city of each store. If we don’t specify which field we want, the whole Mongo document is returned. This is useful if we want to enrich the flow with several attributes.

![photo]({{site.baseurl}}/assets/images/7-20.png)

Let’s test our flow by enabling all processors and inspecting data provenance. As you can see, the attribute city has been added to the content of the flow file. The city Paris has been added to Store 1 which correspond to the data in MongoDB. What happened here is that the lookup up service extracted the id_store which is “1” from the flow file, generated a query to MongoDB to get the corresponding address_city field, and added it to the field city in the new generated flow files. Note that if the query has returned several results from MongoDB, only the first document is used.

![photo]({{site.baseurl}}/assets/images/7-21.png)

By setting an empty Lookup Value Field, we can retrieve the complete document corresponding to the query { “id_store” : “1” }

![photo]({{site.baseurl}}/assets/images/7-22.png)

> Notice: the current implementation of the MongoDB lookup service has an issue. The type of the key used should be String in MongoDB. I created a JIRA to follow the improvements of this service : https://issues.apache.org/jira/browse/NIFI-4634

## Conclusion

Data enrichment is a common use case for ingestion and flow processing. With Lookup processors and services, we can now easily enrich data in NiFi. Lookup processors and services added to NiFi are a powerful feature for data enrichment in realtime. Using Simple Key/Value lookup service is straightforward and easy to use. In addition, it doesn’t require external data source. For more complex scenarios, NiFi supports lookup from external data source such as MongoDB, ElasticSearch, Kudu or HBase.

Thanks for reading me. As always, feedback and suggestions are welcome.

---
Post photo designed by [Andrik Langfield](https://unsplash.com/@andriklangfield) on [Unsplash](https://unsplash.com/)

