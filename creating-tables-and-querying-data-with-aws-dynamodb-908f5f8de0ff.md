
# Creating Tables and Querying data with AWS DynamoDB

AWS Database

## Creating Tables and Querying data with AWS DynamoDB

### In this article, we will look at the NoSQL database service offered by AWS called DynamoDB and how to perform basic database operations with it.

[**AWS DynamoDB](https://aws.amazon.com/dynamodb/)** is a fully managed NoSQL database service offered by AWS. The major selling point of DynamoDB is that there are no servers to maintain, no backups to schedule and no problem scaling the database to millions of users.

DynamoDB offers single millisecond latency (meaning crazy fast) and is a great choice to build RESTful APIs, IoT data stores, etc. It is also the usual choice of developers while working with other AWS services like Lambda.

In this article, we will go through the steps of creating a table and manipulating data along with using the search options provided by DynamoDB.

### WORKING WITH TABLES

Unlike MySQL or MongoDB, you don’t have to create a database and then create tables within the database. DynamoDB offers a centralized database per account in which you can create any number of tables.

Search for ‘DynamoDB’ in the ‘Find Services’ textbox in your [AWS Console](https://console.aws.amazon.com/). You will be greeted with the DynamoDB dashboard. Let's create a new table.

![DynamoDB Dashboard](https://cdn-images-1.medium.com/max/5940/1*_HrGiu7hTt1ly-19kOU54A.png)*DynamoDB Dashboard*

Click on ‘Create table’. You will be asked for the table name and primary key. A primary key is used to uniquely identify a record in a table. DynamoDB allows you to create two types of primary keys:

* **Partition Key** — A simple primary key that has the value of the partition in which the data is stored. If a partition key is used as the primary key, no two records can be in the same partition. **eg. unique email address.**

* **Composite Key** — A composite key is a combination of a partition key and a sort value. This enables multiple records to be stored in a single partition with unique sort values within that partition **eg. unique email address (partition key) + customer id(sort key)**

Once you have entered a table name and the primary key (with or without the sort key), click ‘Create’. Your table should be created and you can start adding data into your DynamoDB table.

![Items view](https://cdn-images-1.medium.com/max/4684/1*RH2CVtA_Wrd9tcSxIiWmVg.png)*Items view*

Click on ‘Items’. This is where all the items in your table will be displayed. Let’s create a new item. Click on ‘Create item. You will see the popup with the option of adding values to the fields that you have created. You can also add/remove new columns using the ‘+’ icon.

![Adding a record](https://cdn-images-1.medium.com/max/5648/1*Kz0euRwx8uHdAvBJORfCvQ.png)*Adding a record*

Make sure to provide a unique primary key while adding records or DynamoDB will return an error. Once you have created a few records, its easy to update/delete records from your table.

Select one or more records and click ‘Actions’ to edit or delete the records from the table.

![List/Edit/Delete view](https://cdn-images-1.medium.com/max/4696/1*QWYwzjcfkf-CErY97kc-sQ.png)*List/Edit/Delete view*

### SEARCHING (SCAN & QUERY)

The main purpose of adding data to a database is to retrieve that data when needed. DynamoDB offers two data retrieval methods: Scan and Query.

* [Scanning](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Scan.html) returns every record in the database. Scanning a table is useful when you try to work with an entire table and all the values contained in the table. However, scanning is a compute-heavy operation and if your table contains a large data set, you can easily run out of the free tier capacity offered by AWS.

* [Query](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Query.html) is the other data retrieval method offered by DynamoDB. Query lets you use filters to select a range of data to be returned, making the operation more efficient compared to a Scan operation. Query is also similar to the data querying methods used in traditional SQL or the ‘Find’ command in MongoDB.

![Query operation](https://cdn-images-1.medium.com/max/4684/1*P1ujNjtbdPCuSeYMX-ob6w.png)*Query operation*

### INDEXING

A database is never complete without the option of indexing data. Indexes are used to improve search performances of specific columns in a table by storing an additional searchable version of column(s). Even though indexes use additional space, they offer great improvements in performance.

Creating an index in DynamoDB is easy. Click on ‘Indexes’ and click ‘Create Index’. Choose the partition key and an optional sort key along with a name for the index. You can also choose the ‘Projected Attributes’ which will only include the keys you choose while returning search results.

![Creating an Index](https://cdn-images-1.medium.com/max/4684/1*zXV_nxBse7k_LnlLmx3QQw.png)*Creating an Index*

Creating indexes can also incur additional monthly costs in addition to the DynamoDB pricing. For complete information on DynamoDB pricing, [visit the official DynamoDB pricing guide here](https://aws.amazon.com/dynamodb/pricing/).

### SUMMARY

DynamoDB is a perfect database choice for most use cases including mobile backends. It also works well with most programming languages like Python and Golang through the [AWS SDK](https://aws.amazon.com/tools/).

Hope this tutorial helped you to understand DynamoDB better. If you have any questions, let me know in the comments.

*If you have a topic you would like me to write about, let me know in the comments. Signup for my [weekly newsletter](https://www.manishmshiva.com/) and I’ll send you a summary of articles and videos every week. No spam. No ads.*
