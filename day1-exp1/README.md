# Day 1, Experience 1 - Handling Big Data with SQL Server 2019 Big Data Clusters

Contoso Auto stores data in several data stores, including relational databases, NoSQL databases, data warehouses, and unstructured data stored in a data lake. They have heard of data virtualization in SQL Server 2019, and are interested to see whether this feature will allow them to more easily access their data stored in these disparate locations. They have heard of the new Big Data Clusters that can be scaled out to handle their Big Data workloads, including machine learning tasks and advanced analytics. They are also interested in any performance improvements against their internal SQL tables by moving to 2019, since the overall amount of data is growing at a rapid pace.

This experience will highlight the new features of SQL Server 2019 with a focus on Big Data Clusters and data virtualization. You will gain hands-on experience with querying both structured and unstructured data in a unified way using T-SQL. This capability will be illustrated by joining different data sets, such as telemetry data in flat CSV files in Azure Storage, CRM data stored in Oracle, parts data in Mongo DB, and transactional data in SQL Server for exploratory data analysis within Azure Data Studio. This joined data will be prepared into a table used for reporting, highlighting query performance against this table due to intelligent query processing. You will also learn how Docker containers are used to scale-out storage and compute to handle heavy workloads. With the inclusion of Apache Spark packaged with Big Data Clusters, it is now possible to use Spark to train machine learning models over data lakes and use those models in SQL Server in one system. You will learn how to use Azure Data Studio to work with Jupyter notebooks to train a simple model that can predict vehicle battery lifetime, then operationalize it within a stored procedure. Finally, you will experience the data security and compliance features provided by SQL Server 2019 by using the Data Discovery & Classification tool in SSMS to identify tables and columns with PII and GDPR-related compliance issues, then address the issues by layering on dynamic data masking to identified columns.

- [Day 1, Experience 1 - Handling Big Data with SQL Server 2019 Big Data Clusters](#day-1-experience-1---handling-big-data-with-sql-server-2019-big-data-clusters)
  - [Experience requirements](#experience-requirements)
  - [Before the lab: Connecting to SQL Server 2019](#before-the-lab-connecting-to-sql-server-2019)
    - [Connect with Azure Data Studio](#connect-with-azure-data-studio)
    - [Connect with SQL Server Management Studio](#connect-with-sql-server-management-studio)
    - [Enlarge the ContosoAutoDW database](#enlarge-the-contosoautodw-database)
  - [Task 1: Query and join data from flat files, data from external database systems, and SQL Server](#task-1-query-and-join-data-from-flat-files-data-from-external-database-systems-and-sql-server)
  - [Task 2: Train a machine learning model and deploy it to a SQL stored procedure](#task-2-train-a-machine-learning-model-and-deploy-it-to-a-sql-stored-procedure)
  - [Task 3: Scale out both storage and compute capabilities of a SQL cluster](#task-3-scale-out-both-storage-and-compute-capabilities-of-a-sql-cluster)
  - [Task 4: Query performance improvements with intelligent query processing](#task-4-query-performance-improvements-with-intelligent-query-processing)
  - [Task 5: Identify PII and GDPR-related compliance issues using Data Discovery & Classification in SSMS](#task-5-identify-pii-and-gdpr-related-compliance-issues-using-data-discovery--classification-in-ssms)
  - [Task 6: Fix compliance issues with dynamic data masking](#task-6-fix-compliance-issues-with-dynamic-data-masking)

## Experience requirements

- Azure subscription
- [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) (SSMS) v17.9.1 or greater
- [Azure Data Studio](https://docs.microsoft.com/sql/azure-data-studio/download?view=sql-server-ver15)
  - [SQL Server 2019 extension](https://docs.microsoft.com/sql/azure-data-studio/sql-server-2019-extension?view=sql-server-ver15)
- SQL Server 2019 login credentials provided for your lab environment
- Azure SQL Database login credentials provided for your lab environment

## Before the lab: Connecting to SQL Server 2019

Follow the steps below to connect to your SQL Server 2019 cluster with both Azure Data Studio and SQL Server Management Studio (SSMS).

### Connect with Azure Data Studio

1.  On the bottom-left corner of your Windows desktop, locate the search box next to the Start Menu. Type **Azure Data Studio**, then select the Azure Data Studio desktop app in the search results.

    ![The search box has "Azure Data Studio" entered into it and the desktop app is highlighted in the results.](media/launch-azure-data-studio.png 'Launch Azure Data Studio')

2.  Within Azure Data Studio, select **Servers** from the top of the left-hand menu, then select **New Connection** from the top toolbar to the right of the menu.

    ![The Servers menu icon is selected, as well as the new connection icon.](media/ads-new-connection-link.png 'Azure Data Studio')

3.  Within the Connection dialog, configure the following:

    - **Connection type:** Select Microsoft SQL Server.
    - **Server:** Enter the IP address, followed by port number `31433`. For example: `123.123.123.123,31433`.
    - **Username:** Enter "sa".
    - **Password:** Enter the password provided to you for this lab.
    - **Remember password:** Checked.
    - Leave all other options at their default values.

    ![The Connection form is filled out with the previously mentioned settings entered into the appropriate fields.](media/ads-new-connection.png 'Azure Data Studio - New Connection')

4.  Click **Connect**.

### Connect with SQL Server Management Studio

1.  On the bottom-left corner of your Windows desktop, locate the search box next to the Start Menu. Type **SQL Server Management Studio**, then select the SQL Server Management Studio desktop app in the search results.

    ![The search box has "SQL Server Management Studio" entered into it and the desktop app is highlighted in the results.](media/launch-ssms.png 'Launch SQL Server Management Studio')

2.  Within the Connection dialog that appears, configure the following:

    - **Server name:** Enter the IP address, followed by port number `31433`. For example: `123.123.123.123,31433`.
    - **Login:** Enter "sa".
    - **Password:** Enter the password provided to you for this lab.
    - **Remember password:** Checked.

    ![The Connect form is filled out with the previously mentioned settings entered into the appropriate fields.](media/ssms-connection.png 'SQL Server Management Studio - Connect')

3.  Click **Options >>**.

4.  Select the **Additional Connection Parameters** tab. In the text area below, enter `TrustServerCertificate=True`. This is needed because the server certificates are dynamically generated for the Big Data Clusters, and are self-signed.

    ![The Additional Connection Parameters tab is selected and the TrustServerCertificate=True value is highlighted.](media/ssms-connection-additional.png 'Additional Connection Parameters')

5.  Click **Connect**.

### Enlarge the ContosoAutoDW database

The script you will execute below enlarges the ContosoAutoDW database. This helps make the impact of the intelligent QP scripts you execute in [Task 4 below](#task-4-query-performance-improvements-with-intelligent-query-processing) more pronounced.

1.  Open SQL Server Management Studio (SSMS) and connect to your SQL Server 2019 cluster. If you are unsure of how to do this, refer to [Connect with SQL Server Management Studio](#connect-with-sql-server-management-studio) at the top of this guide.

2.  Right-click on `ContosoAutoDW`, then select **New Query**. This will open a new query window into which you can paste the following queries. You may wish to reuse the same query window, replacing its contents with each SQL statement blocks below, or follow these same steps to create new query windows for each.

    ![ContosoAutoDW is selected and the New Query menu option is highlighted.](media/ssms-new-query.png 'New Query')

3.  Paste the following query into the new query window:

    ```sql
    USE ContosoAutoDW;
    GO

    /*
      Assumes a fresh restore of ContosoAutoDW.
    */

    IF OBJECT_ID('Fact.OrderHistory') IS NULL BEGIN
        SELECT [Order Key], [City Key], [Customer Key], [Stock Item Key], [Order Date Key], [Picked Date Key], [Salesperson Key], [Picker Key], [WWI Order ID], [WWI Backorder ID], Description, Package, Quantity, [Unit Price], [Tax Rate], [Total Excluding Tax], [Tax Amount], [Total Including Tax], [Lineage Key]
        INTO Fact.OrderHistory
        FROM Fact.[Order];
    END;

    ALTER TABLE Fact.OrderHistory
    ADD CONSTRAINT PK_Fact_OrderHistory PRIMARY KEY NONCLUSTERED([Order Key] ASC, [Order Date Key] ASC)WITH(DATA_COMPRESSION=PAGE);
    GO

    CREATE INDEX IX_Stock_Item_Key
    ON Fact.OrderHistory([Stock Item Key])
    INCLUDE(Quantity)
    WITH(DATA_COMPRESSION=PAGE);
    GO

    CREATE INDEX IX_OrderHistory_Quantity
    ON Fact.OrderHistory([Quantity])
    INCLUDE([Order Key])
    WITH(DATA_COMPRESSION=PAGE);
    GO

    /*
      Reality check... Starting count should be 231,412
    */
    SELECT COUNT(*) FROM Fact.OrderHistory;
    GO

    /*
      Make this table bigger (exec as desired)
      Notice the "GO 4"
    */
    INSERT Fact.OrderHistory([City Key], [Customer Key], [Stock Item Key], [Order Date Key], [Picked Date Key], [Salesperson Key], [Picker Key], [WWI Order ID], [WWI Backorder ID], Description, Package, Quantity, [Unit Price], [Tax Rate], [Total Excluding Tax], [Tax Amount], [Total Including Tax], [Lineage Key])
    SELECT [City Key], [Customer Key], [Stock Item Key], [Order Date Key], [Picked Date Key], [Salesperson Key], [Picker Key], [WWI Order ID], [WWI Backorder ID], Description, Package, Quantity, [Unit Price], [Tax Rate], [Total Excluding Tax], [Tax Amount], [Total Including Tax], [Lineage Key]
    FROM Fact.OrderHistory;
    GO 4

    /*
      Should be 3,702,592
    */
    SELECT COUNT(*) FROM Fact.OrderHistory;
    GO

    IF OBJECT_ID('Fact.OrderHistoryExtended') IS NULL BEGIN
        SELECT [Order Key], [City Key], [Customer Key], [Stock Item Key], [Order Date Key], [Picked Date Key], [Salesperson Key], [Picker Key], [WWI Order ID], [WWI Backorder ID], Description, Package, Quantity, [Unit Price], [Tax Rate], [Total Excluding Tax], [Tax Amount], [Total Including Tax], [Lineage Key]
        INTO Fact.OrderHistoryExtended
        FROM Fact.[OrderHistory];
    END;

    ALTER TABLE Fact.OrderHistoryExtended
    ADD CONSTRAINT PK_Fact_OrderHistoryExtended PRIMARY KEY NONCLUSTERED([Order Key] ASC, [Order Date Key] ASC)
    WITH(DATA_COMPRESSION=PAGE);
    GO

    CREATE INDEX IX_Stock_Item_Key
    ON Fact.OrderHistoryExtended([Stock Item Key])
    INCLUDE(Quantity);
    GO

    /*
      Should be 3,702,592
    */
    SELECT COUNT(*) FROM Fact.OrderHistoryExtended;
    GO

    /*
      Make this table bigger (exec as desired)
      Notice the "GO 2"
    */
    INSERT Fact.OrderHistoryExtended([City Key], [Customer Key], [Stock Item Key], [Order Date Key], [Picked Date Key], [Salesperson Key], [Picker Key], [WWI Order ID], [WWI Backorder ID], Description, Package, Quantity, [Unit Price], [Tax Rate], [Total Excluding Tax], [Tax Amount], [Total Including Tax], [Lineage Key])
    SELECT [City Key], [Customer Key], [Stock Item Key], [Order Date Key], [Picked Date Key], [Salesperson Key], [Picker Key], [WWI Order ID], [WWI Backorder ID], Description, Package, Quantity, [Unit Price], [Tax Rate], [Total Excluding Tax], [Tax Amount], [Total Including Tax], [Lineage Key]
    FROM Fact.OrderHistoryExtended;
    GO 2

    /*
      Should be 14,810,368
    */
    SELECT COUNT(*) FROM Fact.OrderHistoryExtended;
    GO

    UPDATE Fact.OrderHistoryExtended
    SET [WWI Order ID] = [Order Key];
    GO
    ```

4.  Click the **Execute** button on the top toolbar to run the script, or simply enter _F5_ on your keyboard.

    It will take **up to 10 minutes** to execute this query. You may continue performing the tasks below **while this runs**.

    ![The New Query window has the SQL query pasted into it, and the Execute button is highlighted.](media/ssms-execute.png 'SSMS Execute query')

5.  After the query is finished, you should see output similar to the screenshot below (highlighted in red) in the Results window. Also, the lower-right portion of the status bar shows the total execution time, in this case just under 11 minutes.

    ![The query results are highlighted, as well as the execution time.](media/ssms-execution-completed.png 'SSMS query results')

## Task 1: Query and join data from flat files, data from external database systems, and SQL Server

One of the key new features of SQL Server 2019 is data virtualization. What this means is that you can _virtualize_ external data in a SQL Server instance, regardless of source, location, and format, so that it can be queried like any other table, or sets of tables, within your SQL Server instance. In essence, data virtualization helps you create a single "virtual" layer of data from these disparate sources, providing unified data services to support multiple applications and users. A more familiar term we could use is data lake, or perhaps data hub. Unlike a typical data lake, however, you do not have to move data out from where it lives, yet you can still query that data through a consistent interface. This is a huge advantage over traditional ETL (extract-transform-load) processes where data must be moved from its original source to a new destination, oftentimes with some data transformation or mapping. This causes delays, extra storage, additional security, and a fair amount of engineering in most cases. With data virtualization, no data movement is required, which means the data sets are up-to-date, and it is possible to query and join these different data sources through these new capabilities, thanks to the use of new [PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide?view=sql-server-ver15) connectors. The data sources you can connect to include Cosmos DB, SQL Server (including Azure SQL Database), Oracle, HDFS (for flat files), and DB2.

![Data Virtualization compared to traditional ETL.](media/data-virtualization-vs-etl.png 'ETL vs. Data Virtualization')

The image to the left represents traditional data movement using ETL. Compare that to data virtualization, which does not require data movement and provides a unified layer over top of existing data sources.

In this task, you will experience how to configure data virtualization in SQL Server 2019 by joining data in a single query from a SQL Server 2019 table, an external file stored in HDFS (Hadoop Filesystem), and an external database.

Learn more about using data virtualization with [relational data sources](https://docs.microsoft.com/sql/relational-databases/polybase/data-virtualization?toc=%2fsql%2fbig-data-cluster%2ftoc.json&view=sql-server-ver15) and [CSV files](https://docs.microsoft.com/sql/relational-databases/polybase/data-virtualization-csv?view=sql-server-ver15), using the External Table Wizard.

To start, we will use the External Table Wizard in Azure Data Studio to connect to an external Azure SQL Database.

1.  Open Azure Data Studio and connect to your SQL Server 2019 cluster, following the [connection steps](#connect-with-azure-data-studio) above.

2.  Expand the Databases folder, right-click on the **sales** database, then select **Create External Table**.

    ![The sales database and the Create External Table sub-menu item are highlighted.](media/ads-create-external-table-sales.png 'Create External Table')

3.  Select the **SQL Server** data source type, then click **Next**.

    ![The SQL Server data source type is selected.](media/ads-external-table-wizard-data-source-type.png 'Select data source type')

4.  The next step is to create a database master key, if it does not already exist. This secures the credentials used by an external data source. Enter `MySecure@MasterKey1` in the **Password** and **Confirm Password** fields. If you see a message stating that a master key already exists, you may skip this step. Click **Next**.

    ![The Master Key step is displayed.](media/ads-external-table-wizard-master-key.png 'Master Key')

5.  Now, enter the credentials provided to you for the **CA_Commerce** Azure SQL Database within the following fields:

    - **External Data Source Name:** Enter "SQLReviews".
    - **Server Name:** Enter the value provided to you for the Azure SQL Server name. The name should end with `.database.windows.net`.
    - **Database Name:** Enter "CA_Commerce".
    - **Choose Credential:** Select "-- Create New Credential --".
    - **Username:** Enter the Azure SQL Server username provided to you for this lab.
    - **Password:** Enter the Azure SQL Server password provided to you for this lab.

    ![The external data source connection form is filled out with the previously mentioned settings entered into the appropriate fields.](media/ads-external-table-wizard-data-source.png 'Create a connection to your Data Source')

6.  Click **Next**. This process will take a few moments while the External Table Wizard attempts to connect to your data source.

7.  The next screen allows you to configure external table mapping and select the tables for which you want to create external views. Expand the **CA_Commerce** database node, then expand Tables, and check the box next to the **dbo.Reviews** table. Click on the table name to highlight it as well. It is here where you can rename the external table if you wish. For now, just click **Next**.

    ![The external tables are listed and the Reviews table is checked and selected.](media/ads-external-table-wizard-table-mapping.png 'External tables')

8.  In the Summary page that follows, you can see the name of the database scoped credential and external data source objects to be created in the destination database. Here you can click Generate Script to view the SQL script that will run to create the external table. Instead, click **Create**.

    ![A screenshot of the summary is displayed.](media/ads-external-table-wizard-summary.png 'Summary')

9.  After a few moments, a "Create External Table succeeded" message will display.

    ![The Create External Table succeeded message is displayed.](media/ads-external-table-wizard-succeeded.png 'Create External Table succeeded message')

10. Select the Servers link (Ctrl+G) on the left-hand menu, then expand the Tables list underneath your **sales** database and find the **dbo.Reviews (External)** table. If you do not see it, right-click on the Tables folder, then select Refresh. The "(External)" portion of the table name denotes that it is a virtual data object that was added as an external table.

    ![The Reviews external table is displayed in the sales tables list.](media/ads-reviews-table-in-list.png 'Reviews external table')

11. Right-click the **dbo.Reviews (External)** table, then select the **Select Top 1000** menu option to display the table contents.

    ![The Select Top 1000 rows menu item is highlighted.](media/ads-reviews-select-top-1000.png 'Select Top 1000')

12. You should see a SQL query selecting the top 1000 records from the Reviews table and its results. The interesting thing to note is that the query selects the table and fields using the same syntax you would use to select from any other table in the sales database. The fact that the Reviews table is external is completely seamless and transparent to the user. This is the power of data virtualization in SQL Server 2019.

    ![The Reviews query and results are displayed.](media/ads-reviews-query-results.png 'Reviews query results')

    ```sql
    SELECT TOP (1000) [product_id]
        ,[customer_id]
        ,[review]
        ,[date_added]
    FROM [sales].[dbo].[Reviews]
    ```

13. The next data source we will be virtualizing is a CSV file that you will upload to HDFS. **Download** the [stockitemholdings.csv](../lab-files/files/stockitemholdings.csv) file, making note of where it is being saved.

14. Within Azure Data Studio, scroll down below the list of SQL Server 2019 databases to find the **Data Services** folder. Expand that folder, then expand the **HDFS** sub-folder. **Right-click on HDFS**, then select **New directory** on the context menu.


    ![The HDFS folder and New directory menu items are highlighted.](media/ads-new-directory-link.png "New directory")

15. In the new dialog that appears, type "data", then press Enter on your keyboard.

    ![The new directory dialog is displayed with data typed in as the new directory name.](media/ads-new-directory.png 'New directory dialog')

16. **Right-click** on your newly created **data** folder, then select **Upload files**.

    ![The data folder and Upload files menu item are highlighted.](media/ads-upload-files-link.png 'Upload files')

17. Browse to the location to which you downloaded the `stockitemholdings.csv` file and select it. Click **Upload**.

    ![The file browse dialog is displayed with the CSV file selected and the Upload button highlighted.](media/ads-upload-files.png 'Select file and upload')

18. Right-click on the `stockitemholdings.csv` file that was just uploaded to the **data** folder, then select **Create External Table From CSV Files**.


    ![The CSV file and the Create External Table From CSV Files menu item are highlighted.](media/ads-create-external-table-csv.png "Create External Table From CSV Files")

19. The first dialog has you select the SQL Server Master instance containing your Big Data Cluster. Select the connection underneath **Active SQL Server connections** that includes the cluster's IP address and the **sales** database name.

    ![The active SQL Server connection is highlighted.](media/ads-external-table-csv-wizard-active-connection.png 'Active SQL Server connections')

20. Click **Next**.

21. In the destination database step, select the **sales** database underneath **Database the external table will be created in**. Leave the name and schema at their defaults, then click **Next**.

    ![The sales database is selected and highlighted.](media/ads-external-table-csv-destination.png 'Destination database')

22. The next step displays a preview of the first 50 rows CSV data for validation. Click **Next** to continue.

    ![A preview of the CSV data is displayed.](media/ads-external-table-csv-preview.png 'Preview Data')

23. In the next step, you will be able to Modify the columns of the external table you intend to create. you are able to alter the column name, Change the data type, and allow for Nullable rows. Leave everything as-is and click **Next**.

    ![The Modify Columns step is displayed.](media/ads-external-table-csv-modify.png 'Modify Columns')

24. Verify that everything looks correct in the Summary step, then click **Create Table**.

    ![The Summary step is displayed.](media/ads-external-table-csv-create.png 'Summary')

25. As with the previous external table you created, a "Create External Table succeeded" dialog will appear under your task history in a few moments. Select the Servers link (Ctrl+G) on the left-hand menu, then expand the Tables list underneath your **sales** database and find the **dbo.stockitemholdings (External)** table. If you do not see it, right-click on the Tables folder, then select Refresh. **Right-click** the **dbo.stockitemholdings (External)** table, then select **Select Top 1000** from the context menu.

    ![The Select Top 1000 rows menu item is highlighted.](media/ads-stockitemholdings-select-top-1000.png 'Select Top 1000')

26. Just as before, you should see a SQL query selecting the top 1000 rows and its query results, this time from the `stockitemholdings` table. Again, the SQL query is the same type of query you would write to select from a table internal to the sales database.

    ![The stockitemholdings query and results are displayed.](media/ads-stockitemholdings-results.png 'Stockitemholdings results')

    ```sql
    SELECT TOP (1000) [StockItemID]
        ,[QuantityOnHand]
        ,[BinLocation]
        ,[LastStocktakeQuantity]
        ,[LastCostPrice]
        ,[ReorderLevel]
        ,[TargetStockLevel]
        ,[LastEditedBy]
        ,[LastEditedWhen]
    FROM [sales].[dbo].[stockitemholdings]
    ```

27. Now that we have our two external tables added, we will now join those two external tables and two internal tables with a new SQL query to demonstrate how you can seamlessly combine all these data sources without having to copy any files or with separate queries or additional processing of that data. **Right-click** the **sales** database, then select **New Query**.

    ![The sales database and New Query menu item are highlighted.](media/ads-new-query.png 'New Query')

28. Paste the following into the new query window:

    ```sql
    SELECT i.i_item_sk AS ItemID, i.i_item_desc AS Item, c.c_first_name AS FirstName,
      c.c_last_name AS LastName, s.QuantityOnHand, r.review AS Review, r.date_added AS DateReviewed
    FROM dbo.item as i
    JOIN dbo.Reviews AS r ON i.i_item_sk = r.product_id
    JOIN dbo.customer AS c ON c.c_customer_sk = r.customer_id
    JOIN dbo.stockitemholdings AS s ON i.i_item_sk = s.StockItemID
    ```

29. Click the **Run** button above the query window to execute.

    ![The Run button above the query window is highlighted.](media/ads-run.png 'Run')

30. At the bottom of the query window, you will see results that include columns from the four data sources.

    ![Query results from the four data sets.](media/ads-query-results.png 'Query results')

## Task 2: Train a machine learning model and deploy it to a SQL stored procedure

TBD

## Task 3: Scale out both storage and compute capabilities of a SQL cluster

TBD

## Task 4: Query performance improvements with intelligent query processing

In this task, you will execute a series of SQL scripts in SQL Server Management Studio (SSMS) to explore the improvements to family of intelligent query processing (QP) features in SQL Server 2019. These features improve the performance of existing workloads with minimal work on your part to implement. The key to enabling these features in SQL Server 2019 is to set the database compatibility level to `150`. You will be executing these queries against the `ContosoAutoDW` database.

Read more about [intelligent query processing](https://docs.microsoft.com/sql/relational-databases/performance/intelligent-query-processing?view=sql-server-ver15) in SQL databases.

1.  Open SQL Server Management Studio (SSMS) and connect to your SQL Server 2019 cluster. If you are unsure of how to do this, refer to [Connect with SQL Server Management Studio](#connect-with-sql-server-management-studio) at the top of this guide.

2.  Right-click on `ContosoAutoDW`, then select **New Query**. This will open a new query window into which you can paste the following queries. You may wish to reuse the same query window, replacing its contents with each SQL statement blocks below, or follow these same steps to create new query windows for each.

    ![ContosoAutoDW is selected and the New Query menu option is highlighted.](media/ssms-new-query.png 'New Query')

3.  Paste the following query into the query window. This ensures that the database compatibility level is set to `150`, which is the new compatibility level for SQL Server 2019, enabling the most recent intelligent QP features. It also creates a new user-defined function (UDF) named `customer_category` that will be called inline from the two queries that follow in order to show QP improvements on scalar UDF inlining.

    ```sql
    USE ContosoAutoDW;
    GO

    ALTER DATABASE ContosoAutoDW
    SET COMPATIBILITY_LEVEL = 150;
    GO

    ALTER DATABASE SCOPED CONFIGURATION
    CLEAR PROCEDURE_CACHE;
    GO
    /*
    Adapted from SQL Server Books Online
    https://docs.microsoft.com/en-us/sql/relational-databases/user-defined-functions/scalar-udf-inlining?view=sqlallproducts-allversions
    */
    CREATE OR ALTER FUNCTION
      dbo.customer_category(@CustomerKey INT)
    RETURNS CHAR(10) AS
    BEGIN
      DECLARE @total_amount DECIMAL(18,2);
      DECLARE @category CHAR(10);

      SELECT @total_amount =
      SUM([Total Including Tax])
      FROM [Fact].[OrderHistory]
      WHERE [Customer Key] = @CustomerKey;

      IF @total_amount < 500000
        SET @category = 'REGULAR';
      ELSE IF @total_amount < 1000000
        SET @category = 'GOLD';
      ELSE
        SET @category = 'PLATINUM';

      RETURN @category;
    END
    GO
    ```

    > Scalar UDF inlining automatically transforms [scalar UDFs](https://docs.microsoft.com/sql/relational-databases/user-defined-functions/create-user-defined-functions-database-engine?view=sql-server-2017#Scalar) into relational expressions. It embeds them in the calling SQL query. This transformation improves the performance of workloads that take advantage of scalar UDFs. Scalar UDF inlining facilitates cost-based optimization of operations inside UDFs. The results are efficient, set-oriented, and parallel instead of inefficient, iterative, serial execution plans. This feature is enabled by default under database compatibility level 150. _For more information, see [Scalar UDF inlining](https://docs.microsoft.com/sql/relational-databases/user-defined-functions/scalar-udf-inlining?view=sql-server-2017)_.

4.  Click the **Execute** button on the top toolbar to run the script, or simply enter _F5_ on your keyboard.

    ![The New Query window has the SQL query pasted into it, and the Execute button is highlighted.](media/ssms-execute-udf.png 'SSMS Execute query')

5.  After executing the query, you should see a message below that says, `Commands completed successfully.`

    ![The Messages output says Commands completed successfully.](media/ssms-completed-successfully.png 'SSMS Messages')

6.  Next, either highlight and delete everything in the query window, or repeat step 2 above to open a new query window.

7.  The next query selects the top 100 rows from the `Customer` dimension table, calling the UDF you created inline for each row. It uses the `DISABLE_TSQL_SCALAR_UDF_INLINING` hint to disable the new scalar UDF inlining QP feature. Paste the following query into the the empty query window. **Do not execute yet**.

    ```sql
    USE ContosoAutoDW;
    GO

    -- Before (show actual query execution plan for legacy behavior)
    SELECT TOP 100
        [Customer Key], [Customer],
          dbo.customer_category([Customer Key]) AS [Discount Price]
    FROM [Dimension].[Customer]
    ORDER BY [Customer Key]
    OPTION (RECOMPILE,USE HINT('DISABLE_TSQL_SCALAR_UDF_INLINING'));
    ```

8.  Click the **Include Actual Execution Plan** (Ctrl+M) button in the toolbar above the query window. This will allow us to view the actual (not estimated) query plan after executing the query.

    ![The Actual Query Plan button is highlighted in the toolbar.](media/ssms-enable-actual-query-plan.png 'Enable Actual Query Plan')

9.  **Execute** the query.

10. After the query executes, select the **Execution plan** tab. As the plan shows, SQL Server adopts a simple strategy here: for every tuple in the `Customer` table, invoke the UDF and output the results. This strategy is naïve and inefficient. Also, make note of the query execution time. In our case in the screenshot below, it took 9 seconds to complete.

    ![This screenshot shows the query execution plan using the legacy method.](media/ssms-udf-inlining-old.png 'Query execution plan with legacy method')

11. Clear the query window, or open a new one, then paste the following query that makes use of the scalar UDF inlining QP feature. If you opened a new query window instead of reusing this one, make sure to click the **Include Actual Execution Plan** button to enable it. **Execute** the query.

    ```sql
    USE ContosoAutoDW;
    GO

    -- After (show actual query execution plan for Scalar UDF Inlining)
    SELECT TOP 100
        [Customer Key], [Customer],
          dbo.customer_category([Customer Key]) AS [Discount Price]
    FROM [Dimension].[Customer]
    ORDER BY [Customer Key]
    OPTION (RECOMPILE);
    ```

12. After the query executes, select the **Execution plan** tab once again. With scalar UDF inlining, this UDF is transformed into equivalent scalar subqueries, which are substituted in the calling query in place of the UDF.

    ![This screenshot shows the query execution plan using the new QP feature.](media/ssms-udf-inlining-new.png 'Query execution plan with new method')

    > As you can see, the query plan no longer has a user-defined function operator, but its effects are now observable in the plan, like views or inline TVFs. Here are some key observations from the above plan:

    1. SQL Server has inferred the implicit join between `CUSTOMER` and `ORDERS` and made that explicit via a join operator.
    2. SQL Server has also inferred the implicit `GROUP BY O_CUSTKEY on ORDERS` and has used the IndexSpool + StreamAggregate to implement it.
    3. SQL Server is now using parallelism across all operators.

    > Depending upon the complexity of the logic in the UDF, the resulting query plan might also get bigger and more complex. As we can see, the operations inside the UDF are now no longer a black box, and hence the query optimizer is able to cost and optimize those operations. Also, since the UDF is no longer in the plan, iterative UDF invocation is replaced by a plan that completely avoids function call overhead.

13. Either highlight and delete everything in the query window, or open a new query window. Paste the following query. If you opened a new query window instead of reusing this one, make sure to click the **Include Actual Execution Plan** button to enable it. **Execute** the query.

    ```sql
    USE [master];
    GO

    ALTER DATABASE [ContosoAutoDW] SET COMPATIBILITY_LEVEL = 140;
    GO

    USE [ContosoAutoDW];
    GO

    DECLARE @Order TABLE
      ([Order Key] BIGINT NOT NULL,
      [Quantity] INT NOT NULL
      );

    INSERT @Order
    SELECT [Order Key], [Quantity]
    FROM [Fact].[OrderHistory]
    WHERE  [Quantity] > 99;

    -- Look at estimated rows, speed, join algorithm
    SELECT oh.[Order Key], oh.[Order Date Key],
        oh.[Unit Price], o.Quantity
    FROM Fact.OrderHistoryExtended AS oh
    INNER JOIN @Order AS o
      ON o.[Order Key] = oh.[Order Key]
    WHERE oh.[Unit Price] > 0.10
    ORDER BY oh.[Unit Price] DESC;
    GO
    ```

    > The script above assigns a table variable, `@Order`, storing the `Order Key` and `Quantity` fields from the `OrderHistory` table to be used in an INNER JOIN further below. Since the database compatibility level was set to 140, the table variable deferred compilation QP feature is not used (more on this below).

14. After the query executes, select the **Execution Plan** tab. There are two plans. The one you want to observe is the second query plan. Hover over the second INNER JOIN to view the estimated number of rows and the output list, which shows the join algorithm. The estimated number of rows should be around 1. Also, observe the execution time. In our case, it took 11 seconds to complete.

    ![This screenshot shows the query execution plan using the legacy method.](media/ssms-tvdc-old.png 'Query execution plan with old method')

15. Either highlight and delete everything in the query window, or open a new query window. Paste the following query that makes use of the table variable deferred compilation feature by setting the compatibility level to 150. If you opened a new query window instead of reusing this one, make sure to click the **Include Actual Execution Plan** button to enable it. **Execute** the query.

    ```sql
    USE [master]
    GO

    ALTER DATABASE [ContosoAutoDW] SET COMPATIBILITY_LEVEL = 150
    GO

    USE [ContosoAutoDW]
    GO

    DECLARE @Order TABLE
      ([Order Key] BIGINT NOT NULL,
      [Quantity] INT NOT NULL
      );

    INSERT @Order
    SELECT [Order Key], [Quantity]
    FROM [Fact].[OrderHistory]
    WHERE  [Quantity] > 99;

    -- Look at estimated rows, speed, join algorithm
    SELECT oh.[Order Key], oh.[Order Date Key],
        oh.[Unit Price], o.Quantity
    FROM Fact.OrderHistoryExtended AS oh
    INNER JOIN @Order AS o
      ON o.[Order Key] = oh.[Order Key]
    WHERE oh.[Unit Price] > 0.10
    ORDER BY oh.[Unit Price] DESC;
    GO
    ```

16. After the query executes, select the **Execution plan** tab once again. Notice this time that the join algorithm is a hash match, and that the overall query execution plan looks different. When you hover over the INNER JOIN, notice that there is a high value for estimated number of rows and that the output list shows the use of hash keys and an optimized join algorithm. Once again, observe the execution time. In our case, it took 5 seconds to complete, which is less than half the time it took to execute without the table variable deferred compilation feature.

    ![This screenshot shows the query execution plan using the new method.](media/ssms-tvdc-new.png 'Query execution plan with new method')

    > Table variable deferred compilation improves plan quality and overall performance for queries that reference table variables. During optimization and initial compilation, this feature propagates cardinality estimates that are based on actual table variable row counts. This accurate row count information optimizes downstream plan operations. Table variable deferred compilation defers compilation of a statement that references a table variable until the first actual run of the statement. This deferred compilation behavior is the same as that of temporary tables. This change results in the use of actual cardinality instead of the original one-row guess. _For more information, see [Table variable deferred compilation](https://docs.microsoft.com/sql/t-sql/data-types/table-transact-sql?view=sql-server-2017#table-variable-deferred-compilation)._

17. Either highlight and delete everything in the query window, or open a new query window. Paste the following query to simulate out-of-date statistics on the `OrderHistory` table, followed by a query that executes a hash match. If you opened a new query window instead of reusing this one, make sure to click the **Include Actual Execution Plan** button to enable it. **Execute** the query.

    ```sql
    ALTER DATABASE ContosoAutoDW SET COMPATIBILITY_LEVEL = 150;
    GO

    ALTER DATABASE SCOPED CONFIGURATION CLEAR PROCEDURE_CACHE;
    GO

    USE ContosoAutoDW;
    GO

    -- Simulate out-of-date stats
    UPDATE STATISTICS Fact.OrderHistory
    WITH ROWCOUNT = 1;
    GO

    SELECT
      fo.[Order Key], fo.Description,
      si.[Lead Time Days]
    FROM    Fact.OrderHistory AS fo
    INNER HASH JOIN Dimension.[Stock Item] AS si
      ON fo.[Stock Item Key] = si.[Stock Item Key]
    WHERE   fo.[Lineage Key] = 9
      AND si.[Lead Time Days] > 19;
    ```

18. After the query executes, select the **Execution plan** tab. Hover over the Hash Match step of the execution plan. You should see a warning toward the bottom of the Hash Match dialog showing spilled data. Also observe the execution time. In our case, this query took 26 seconds to execute.

    ![The Hash Match dialog shows spilled data warnings.](media/ssms-memory-grant-feedback.png 'Query execution plan showing spilled data')

19. Either highlight and delete everything in the query window, or open a new query window. Paste the following query to execute the select query that contains the hash match once more. If you opened a new query window instead of reusing this one, make sure to click the **Include Actual Execution Plan** button to enable it. **Execute** the query.

    ```sql
    USE ContosoAutoDW;
    GO

    SELECT
      fo.[Order Key], fo.Description,
      si.[Lead Time Days]
    FROM    Fact.OrderHistory AS fo
    INNER HASH JOIN Dimension.[Stock Item] AS si
      ON fo.[Stock Item Key] = si.[Stock Item Key]
    WHERE   fo.[Lineage Key] = 9
      AND si.[Lead Time Days] > 19;
    ```

20. After the query executes, select the **Execution plan** tab. Hover over the Hash Match step of the execution plan. You should **no longer** see a warning about spilled data. Also observe the execution time. In our case, this query took 4 seconds to execute.

    ![The Hash Match dialog no longer contains spilled data warnings.](media/ssms-memory-grant-feedback-fixed.png 'Query execution plan with no spilled data')

    > So what happened? A query's post-execution plan in SQL Server includes the minimum required memory needed for execution and the ideal memory grant size to have all rows fit in memory. Performance suffers when memory grant sizes are incorrectly sized. Excessive grants result in wasted memory and reduced concurrency. Insufficient memory grants cause expensive spills to disk. By addressing repeating workloads, batch mode memory grant feedback recalculates the actual memory required for a query and then updates the grant value for the cached plan. **When an identical query statement is executed**, the query uses the revised memory grant size, reducing excessive memory grants that impact concurrency and fixing underestimated memory grants that cause expensive spills to disk. Row mode memory grant feedback expands on the batch mode memory grant feedback feature by adjusting memory grant sizes for both batch and row mode operators. _For more information, see [Row mode memory grant feedback](https://docs.microsoft.com/sql/relational-databases/performance/adaptive-query-processing?view=sql-server-2017#row-mode-memory-grant-feedback)._

## Task 5: Identify PII and GDPR-related compliance issues using Data Discovery & Classification in SSMS

Contoso Auto has several databases that include tables containing sensitive data, such as personally identifiable information (PII) like phone numbers, social security numbers, financial data, etc. Since some of their personnel and customer data include individuals who reside within the European Union (EU), they need to adhere to the General Data Protection Regulation ([GDPR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation)) as well. Because of this, Contoso Auto is required to provide periodic data auditing reports to identify sensitive and GDPR-related data that reside within their various databases.

With SQL Server Management Studio, they are able to identify, classify, and generate reports on sensitive and GDPR-related data by using the [SQL Data Discovery & Classification](https://docs.microsoft.com/sql/relational-databases/security/sql-data-discovery-and-classification?view=sql-server-ver15) tool. This tool introduces a set of advanced services, forming a new SQL Information Protection paradigm aimed at protecting the data, not just the database:

- **Discovery & recommendations** - The classification engine scans your database and identifies columns containing potentially sensitive data. It then provides you an easy way to review and apply the appropriate classification recommendations, as well as to manually classify columns.
- **Labeling** - Sensitivity classification labels can be persistently tagged on columns.
- **Visibility** - The database classification state can be viewed in a detailed report that can be printed/exported to be used for compliance & auditing purposes, as well as other needs.

In this exercise, you will run the SQL Data Discovery & Classification tool against their data warehouse, which includes personnel, customer, and sales data.

1.  Open SQL Server Management Studio (SSMS) and connect to your SQL Server 2019 cluster.

2.  Right-click on the **ContosoAutoDW** database, then choose **Tasks > Classify Data...**.

    ![The ContosoAutoDW database, Tasks menu, and Classify Data items are highlighted.](media/ssms-classify-data-link.png 'Data Classification')

3.  When the tool runs, it will analyze all of the columns within all of the tables and recommend appropriate data classifications for each. What you should see is the Data Classification dashboard showing no currently classified columns, and a classification recommendations box at the top showing that there are 34 columns that the tool identified as containing sensitive (PII) or GDPR-related data. **Click** on this classification recommendations box.

    ![The data classification recommendations box is highlighted.](media/ssms-classification-recommendations-box.png 'Data classification recommendations box')

4.  The list of recommendations displays the schema, table, column, type of information, and recommended sensitivity label for each identified column. You can change the information type and sensitivity labels for each if desired. In this case, accept all recommendations by **checking the checkbox** in the recommendations table header.

    ![The recommendations are shown with each checkbox checked.](media/ssms-recommendations.png 'Classification recommendations')

5.  Click **Accept selected recommendations**.

    ![The Accept selected recommendations button is highlighted.](media/ssms-accept-selected-recommendations.png 'Accept selected recommendations')

6.  Click **Save** in the toolbar above to apply your changes.

    ![The Save button is highlighted.](media/ssms-save-classification-changes.png 'Save classification changes')

7.  After the changes are saved, click **View Report**.

    ![The View Report button is highlighted.](media/ssms-view-report.png 'View Report')

8.  What you should see is a report with a full summary of the database classification state. When you right-click on the report, you can see options to print or export the report in different formats.

    ![The report is displayed, as well as the context menu showing export options after right-clicking on the report.](media/ssms-report.png 'SQL Data Classification Report')

## Task 6: Fix compliance issues with dynamic data masking

TBD