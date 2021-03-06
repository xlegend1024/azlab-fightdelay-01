# 3. Azure Data Factory

Scenario Architecture

![arch](../images/3.png)

## 3.1 Copy data in bulk

Copy multiple tables from SQL Database to ADLS gen 2

Use two pipelines

First pipeline will look up SQL DB to get table list and then second pipeline will use the name of the tables to select and copy to ADSL gen 2

![pipeline](../images/tutorial-copy-multiple-tables.png)

### 3.1.1 Create Azure Data Factory

![overview](../images/01.00.01.png)

### 3.1.2 Create Linked Servcies

#### 3.1.2.1 Create Linked Service for Source

![overview](../images/01.00.02.png)

SQL Database is source

|Key|Value|
|-|-|
|Server Name|```azlab0sql.database.windows.net```|
|Database Name|```azlab0db```|
|Authentication Type|SQL Authentication|
|User Name|```sqladmin```|
|Password|```_password_```|

#### 3.1.2.2 Create Linked Service for Sink (destination)

![overview](../images/01.00.03.png)

Azure Data Lake Storage Gen 2 is destination

### 3.1.3 Create Datasets

#### 3.1.3.1 reate Dataset for Source

![overview](../images/01.00.04.png)

Create Azure SQL Database for source dataset

Check on edit checkbox and use any randomn table name

> The Table name will be over writen by Copy Activity later

![parameter](../images/3.2.png)

#### 3.1.3.2 Create Dataset for Sink (destination)

![overview](../images/01.00.05.png)

Create _Parquet_ for destination dataset

Add Paramter

![parameter](../images/3.0.png)

Use ```landingzone``` as file path and ```@dataset().name``` as 

![parameter](../images/3.1.png)

### 3.1.4 Create Pipeline

Create two pipeline like following

![parameter](../images/3.5.png)

First create copy pipeline and then create second pipleine for lookup

#### 3.1.4.1 Create Copy Pipeline

![overview](../images/01.00.06.png)

Create pipeline and name it as ```CopyTableToADLSg2```

Add parameter at pipeline level, name it ```tableList``` and choose type as _Array_

> This _tableList_ name will be used to get table name from another pipeline activity

![parameter](../images/3.6.png)

Drag and drop _ForEach_ activity on canvas

Select the _ForEach_ activity and go to _Settings_ 

![parameter](../images/3.7.png)

Double click the _ForEach_ Activity

Drag and drop Copy Data activity 

![parameter](../images/3.8.png)

Go to _Source_ tab and use following query 

```sql
SELECT * FROM [@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]
```

> `TABLE_SCHEMA` and `TABLE_NAME` will be given from another activity later in this lab

Go to _Sink_ tab

Select _Parqeut_ for _sink dataset_ use following value

```text
@{item().TABLE_SCHEMA}_@{item().TABLE_NAME}
```

> `TABLE_SCHEMA` and `TABLE_NAME` will be given from another activity later in this lab

![parameter](../images/3.9.png)

#### 3.1.4.2 Create Lookup Pipeline

Create a pipeline and name it as ```GetTableList```

Drag and drop Lookup activity

Go to _Settings_ and select _Query_ and use following query to run

```sql
SELECT TABLE_SCHEMA, TABLE_NAME FROM information_schema.TABLES WHERE TABLE_TYPE = 'BASE TABLE' and TABLE_SCHEMA = 'SalesLT' and TABLE_NAME <> 'ProductModel'
```
![parameter](../images/3.3.png)

Drag and drop _Execute Pipeline_

Select _CopyTableToADLSg2_ for invoked pipeline

Add a parameter ```tableList``` and use following value 

```@activity('Lookup1').output.value```

![parameter](../images/3.4.png)

### 3.2 Run a Pipeline

### 3.3 Monitor the Pipeline Run

---
Ref: https://docs.microsoft.com/en-us/azure/data-factory/tutorial-bulk-copy-portal