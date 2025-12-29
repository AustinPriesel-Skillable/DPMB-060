# Lab 01 -Deploying and Optimizing ZAVA Retail App using Azure Database for PostgreSQL – Flexible Server

## Scenario

ZAVA DIY is a retail company with stores across Washington State and a
rapidly growing online e-commerce presence. Specializing in outdoor
equipment, home improvement tools, and DIY supplies, the company has
traditionally relied on manual spreadsheets and on-premises systems to
manage product, sales, and inventory data.

This outdated approach has resulted in data silos, delayed insights, and
frequent human errors — especially as online orders continue to surge.

To address these challenges, ZAVA’s leadership launched a cloud
modernization initiative to unify data, improve scalability, and enable
data-driven decisions across all channels. The team is developing a new
internal application — **ZAVA Connect**, an AI-powered platform designed
to automate data collection, unify insights, and deliver intelligent
recommendations for store operations and customer engagement.

Leading this initiative is **Carlos Vega**, the company’s **CTO**, who
oversees technology strategy and innovation. He’s supported by:

- **Kian Lambert – Database Administrator (DBA):** Responsible for
  provisioning, securing, and optimizing the database platform.

- **Elvia Aktins – Application Developer:** Focused on building app
  integrations and enabling AI-powered features within ZAVA Connect.

After evaluating multiple options, the team selects **Azure Database for
PostgreSQL – Flexible Server** as the foundation for ZAVA Connect. The
choice is driven by its **scalable performance**, **built-in security**,
**high availability**, and **AI-readiness** through **pgvector** and
Azure AI extensions — enabling advanced recommendation models and
semantic search.

**Your role in this lab:**  
Step into the role of **Kian Lambert (DBA)** to deploy, secure, and
optimize the Azure Database for PostgreSQL environment that powers ZAVA
Connect — turning ZAVA DIY’s data into actionable business insights.

## Objective

In this lab, you will:

- Deploy and configure **Azure Database for PostgreSQL – Flexible
  Server** to host ZAVA DIY’s retail data.

- Configure compute, storage, and backup settings to support
  scalability, performance, and growth.

- Secure the database using encryption, firewall rules, SSL
  connectivity, and access controls.

- Enable PostgreSQL extensions such as **pgvector** to support AI-ready
  workloads.

- Import and validate the ZAVA DIY retail dataset.

- Run analytical queries to extract business insights and confirm
  correct database behaviour.

- Monitor database performance using Azure Metrics.

- Enable **Microsoft Fabric Mirroring** to support near real-time
  analytics without impacting operational workloads.

- Review and configure **high availability, backups, and read replicas**
  to ensure resilience and business continuity.

## Exercise 1: Provisioning PostgreSQL Flexible Server

In this exercise, you’ll provision a PostgreSQL Flexible Server instance
in Azure to serve as the backend for ZAVA Connect. This includes
configuring compute, storage, and high availability options to ensure
performance and resilience.

### Task 1: Provision PostgreSQL Flexible Server

1.  Open a web browser, navigate to the Azure portal
    +++portal.azure.com+++ and sign in using the given credentials:

    - Username:  +++@lab.CloudPortalCredential(User1).Username+++ 

    - TAP Token: +++@lab.CloudPortalCredential(User1).AccessToken+++ 

    ![](./media/image1.png) 

    ![](./media/image2.png) 

    ![](./media/image3.png) 

2.  Search +++**Azure Database for PostgreSQL Flexible Server**+++ in
    the search bar.

    ![](./media/image4.png)

3.  Click on **+Create**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image5.png)

4.  Enter the following details.

    - Subscription: Keep the default as it is.

    - Resource group: Select **ResourceGroup1**

    - Server name: Enter +++zava-connect-db@lab.LabInstance.Id+++

    - Region: Select Central US

    - PostgreSQL version: 16

    - Workload type: Select Production

    Select the **configure server** in **Comput+storage**.

    ![](./media/image6.png)

5.  The **Compute + storage** page opens. Now you can configure the
    following settings:

    - Compute tier: General Purpose

    - Storage autogrow: Select the checkbox

    - Zonal resiliency: Disable

    - Under Backups, select the Geo-redundancy checkbox.

    Click on **Save** button to save updates.

    ![](./media/image7.png)
    
    ![](./media/image8.png)

6.  Under Authentication. Select **PostgreSQL authentication only**
    authentication method and enter the following login and password:

    - Administrator login: Enter +++postgres+++

    - Password: Enter +++PAssw0rd1a+++

    Select **Next:Networking**

    ![](./media/image9.png)

7.  In the Networking tab, ensure that the **connectivity method** is
    **Public access(allowed IP addresses) and Private endpoint**, and
    allow public access to this resource through the internet using a
    public IP address.

    ![A screenshot of a computer screen AI-generated content may be
    incorrect.](./media/image10.png)

8.  Under Firewall rules, enable “Allow public access from any Azure
    service within Azure to this server” and select +Add current client
    IP address. Then, rename the name of the firewall rule to
    +++ZAVA-HQ+++. Click on **Security.**

    ![](./media/image11.png)
    
    ![](./media/image12.png)

9.  In the Security tab, make sure that the Data encryption key is
    “Service-managed key” for both Primary and paired regions. Then
    click **Review + create**.

    ![](./media/image13.png)

10. Click on **Create**.

    ![](./media/image14.png)

11. Wait for the deployment to be completed. It will take 5-10 mins to
    complete.

    ![](./media/image15.png)
    
    ![](./media/image16.png)

You have provisioned your PostgreSQL Flexible Server successfully.

## Exercise 2: Connect ZAVA Connect Database to PostgreSQL Flexible Server

In this exercise, you’ll enable the required PostgreSQL extensions and
load the **ZAVA DIY dataset** into your Azure Database for PostgreSQL
Flexible Server. You’ll also configure the connection parameters and
generate the ZAVA Connect retail database using the provided dataset
scripts.

### Task 1: Enable pgvector Extension

1.  Click Go to resource.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image17.png)

2.  Expand Settings from the left-hand side resource menu and select
    Server parameters.

    ![](./media/image18.png)

3.  In the search bar, search for +++**azure.extensions+++** parameter.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image19.png)

4.  Search for the +++**VECTOR**+++ parameter and then select the
    **VECTOR** parameter. This will allow the pgvector extension.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image20.png)

5.  Select **Save**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image21.png)

6.  Select **Go to resource**.

    ![](./media/image22.png)

7.  Save the **Server endpoint** in Notepad for future use.

    ![](./media/image23.png)

### Task 2: Clone the repo and create ZAVA Connect’s Database

1.  Open **cloud shell**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image24.png)

2.  Click **Bash**.

    ![](./media/image25.png)

3.  Select the **Mount storage account**, then select your
    **subscription**. Click **Apply**.

    ![](./media/image26.png)

4.  Select **We will create a storage account for you**. Click **Next**.

    ![](./media/image27.png)

5.  Wait for a few minutes to set up.

    ![](./media/image28.png)

6.  Clone the GitHub repo+++ git clone https://github.com/microsoft/ai-tour-26-zava-diy-dataset-plus-mcp.git+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image29.png)

7.  Navigate to the cloned repository

    +++ cd ai-tour-26-zava-diy-dataset-plus-mcp+++

    ![](./media/image30.png)

8.  Navigate to the \`data\` directory inside the repository:

    +++cd data+++

    ![A screenshot of a computer program AI-generated content may be
    incorrect.](./media/image31.png)

9.  Install all the requirements using the following command:

    +++ pip install --user -r requirements.txt+++

    ![](./media/image32.png)

10. Connect to the PostgreSQL flexible server.

    +++ psql "host=zava-connect-db@lab.LabInstance.Id.postgres.database.azure.com user=postgres password='PAssw0rd1a' dbname=postgres sslmode=require"+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image33.png)

11. Create a new database in the PostgreSQL Flexible server.

    +++ CREATE DATABASE zava_connect_retail;+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image34.png)

12. Close the PostgreSQL server using the +++\q+++ command.

    ![A screenshot of a computer screen AI-generated content may be
    incorrect.](./media/image35.png)

13. Connect to the newly created zava_connect_retail database using the
    following command:

    +++ psql
    "host=zava-connect-db@lab.LabInstance.Id.postgres.database.azure.com
    port=5432 dbname=zava_connect_retail user=postgres password= PAssw0rd1a
    sslmode=require"+++

    ![A screenshot of a computer screen AI-generated content may be
    incorrect.](./media/image36.png)

14. Run the following query to create the pgvector extension in the
    target database.

    +++ CREATE EXTENSION vector;+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image37.png)

15. Close the database using the +++\q+++ command.

    ![A screenshot of a computer screen AI-generated content may be
    incorrect.](./media/image38.png)

16. Change the current directory to the database directory.

    +++cd database+++

    ![](./media/image39.png)

17. Open generate_zava_postgres.py using +++ nano
    generate_zava_postgres.py+++.

    ![](./media/image40.png)

18. In the nano generate_zava_postgres.py file, look for the following
    lines:

    PostgreSQL connection configuration

    POSTGRES_CONFIG = {

    'host': 'db',

    'port': 5432,

    'user': 'postgres',

    'password': 'P@ssw0rd!',

    'database': 'zava'

    }

    ![](./media/image41.png)

    Now replace the values of fields with the following values:

    host: +++zava-connect-db.postgres.database.azure.com+++

    port: +++5432+++

    user: +++‘postgres’+++

    password: +++‘PAssw0rd1a’+++

    database: +++‘zava_connect_retail’+++

    ![](./media/image42.png)

19. In the same file, Press Ctrl + W -\> type 50000-\>press Enter. You
    will reach to the following function:

    ![](./media/image43.png)  
    Now replace 50000 with +++1000+++.

    ![](./media/image44.png)

    Again, Press Ctrl + W -\> type 50000-\>press Enter. You will reach here:

    ![](./media/image45.png)

    Now replace 50000 with +++1000+++.

    ![](./media/image46.png)

20. To save and exit from the nano generate_zava_postgres.py file press
    ctrl + O -\> press Enter -\>Ctrl + X.

21. Install the following package:

    +++ pip install --user asyncpg+++

    ![](./media/image47.png)

22. Run +++ python generate_zava_postgres.py+++ command. This command
    creates the ZAVA DIY PostgreSQL database and fills it with sample
    retail data. It sets up all tables, schemas, and data.

    ![](./media/image48.png)

    It will take 10-15 minutes to complete, and after completing, you will
    get output like this:

    ![](./media/image49.png)

### Task 3: Querying and Exploring Azure PostgreSQL Data

1.  Connect to the target database.

    +++ psql
    "host=zava-connect-db@lab.LabInstance.Id.postgres.database.azure.com
    port=5432 dbname=zava_connect_retail user=postgres password= PAssw0rd1a
    sslmode=require"+++

    ![](./media/image50.png)

2.  See all the schemas.

    +++\dn+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image51.png)

3.  Switch to retail schema.

    +++ SET search_path TO retail;+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image52.png)

4.  List all the tables in the retail schema.

    +++ \dt+++

    ![](./media/image53.png)

5.  Find the first 10 customer records stored in the retail.customers
    table.

    +++SELECT * FROM retail.customers LIMIT 10;+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image54.png)

6.  Let’s find the top 5 stores that generated the highest total revenue
    by summing up sales from all their orders.

    ```
    SELECT

    s.store_name,

    ROUND(SUM(oi.quantity * oi.unit_price), 2) AS total_revenue

    FROM retail.orders o

    JOIN retail.order_items oi ON o.order_id = oi.order_id

    JOIN retail.stores s ON o.store_id = s.store_id

    GROUP BY s.store_name

    ORDER BY total_revenue DESC

    LIMIT 5;

    ```

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image55.png)

7.  Now retrieves the **top 5 stores** with the **highest total
    revenue**, calculated by multiplying product quantity and unit price
    from all their sales.
    ```
    SELECT

    c.category_name,

    SUM(oi.quantity) AS total_units_sold

    FROM retail.order_items oi

    JOIN retail.products p ON oi.product_id = p.product_id

    JOIN retail.categories c ON p.category_id = c.category_id

    GROUP BY c.category_name

    ORDER BY total_units_sold DESC

    LIMIT 5;
    ```

    Now close the connection using +++\q+++.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image56.png)

You have successfully explored and verified the ZAVA Connect database
hosted on Azure Database for PostgreSQL-Flexible Server.

## Exercise 3: Verify Secure Connectivity 

In this exercise, you’ll verify that your **Azure Database for
PostgreSQL – Flexible Server** is configured securely. You’ll check
encryption settings, firewall rules, and secure access policies to
ensure that only authorized users and trusted services can connect to
the database.

### Task 1: Validate Network Access Controls

1.  Minimize the Azure CLI window but keep it open.

2.  In the **Azure Portal**, navigate to your **Azure Database for PostgreSQL Flexible Server** resource.

    ![](./media/image57.png)

3.  From the left-hand menu, select **Networking** under **Settings**.

    ![](./media/image58.png)

4.  Review the **Connectivity method**:

    - Ensure it is set to **Public access (allowed IP addresses)**.

    - For this lab, public access is enabled, but only from specific trusted
    IPs.

    ![](./media/image59.png)

5.  Under **Firewall rules**, confirm that:

    - “Allow public access from any Azure service within Azure to this
    server” is **enabled**.

    - Only the rule **ZAVA-HQ** (your current client IP) appears — this
    prevents open access from the internet.

    ![](./media/image60.png)

6.  Try connecting again from Cloud Shell using the psql command:

    +++psql "host=zava-connect-db@lab.LabInstance.Id.postgres.database.azure.com port=5432 dbname=zava_connect_retail user=postgres password=P@ssw0rd#a sslmode=require"+++

    ![](./media/image61.png)

7.  From the Azure portal, temporarily remove the **ZAVA-HQ** firewall
    rule to check and disable: “Allow public access from any Azure
    service within Azure”. Click Save.

    ![](./media/image62.png)

8.  Wait 2 minutes for changes to apply.

9.  Open Azure CLI and attempt to reconnect using the following psql
    command:

    +++psql "host=zava-connect-db@lab.LabInstance.Id.postgres.database.azure.com port=5432 dbname=zava_connect_retail user=postgres password=P@ssw0rd#a sslmode=require"+++

    You should receive an error such as:

    ![](./media/image63.png)

    This confirms that the firewall and IP restrictions are working
    correctly.

10. Navigate back to the Azure portal of PostgreSQL flexible server and
    re-add the current client IP rule (**ZAVA-HQ**) and enable “Allow
    public access from any Azure service within Azure” to restore
    access. Click Save.

    ![](./media/image64.png)

    Now we it is working fine.

    ![](./media/image65.png)

### Task 2: Verify Encryption Settings

1.  In the Azure portal, navigate to the PostgreSQL Flexible Server, go
    to **Security** under **Settings**.

2.  Under **Data encryption**, confirm:

    - The **Primary region key** is **Service-managed key**.

    - The **Geo-redundant key** (paired region) is also **Service-managed
    key**.

    This ensures that **Transparent Data Encryption (TDE)** is applied
    automatically to protect your data at rest.

3.  To verify encryption in transit, note that your psql connection
    string includes: **sslmode=require.** This enforces SSL encryption,
    protecting data during transmission between the client and the
    database.

4.  Run this query in your connected psql session:

    +++ SHOW ssl;+++

    You should see:

    ![](./media/image66.png)

    This confirms SSL encryption is active for your connection.

    **Note:** Azure Database for PostgreSQL enforces SSL connections by
    default. Even without explicitly specifying sslmode=require,
    connections are automatically encrypted with TLS.

## Exercise 4: Monitoring Performance in Azure Database for PostgreSQL – Flexible Server

In this exercise, you’ll learn how to monitor the performance of your
Azure Database for PostgreSQL – Flexible Server. You’ll generate a
sample workload and observe key performance metrics such as CPU, memory,
and storage utilization in the Azure Portal.

### Task 1: Generate Query Workload

1.  Run the following multiple times to create a noticeable load on the
    server.

    ```
    -- 1. Total revenue by store

    SELECT s.store_name, SUM(oi.quantity * oi.unit_price) AS total_revenue

    FROM retail.orders o

    JOIN retail.order_items oi ON o.order_id = oi.order_id

    JOIN retail.stores s ON o.store_id = s.store_id

    GROUP BY s.store_name

    ORDER BY total_revenue DESC

    LIMIT 5;
    ```

    ```
    -- 2. Top 10 best-selling products

    SELECT p.product_name, SUM(oi.quantity) AS total_sold

    FROM retail.order_items oi

    JOIN retail.products p ON oi.product_id = p.product_id

    GROUP BY p.product_name

    ORDER BY total_sold DESC

    LIMIT 10;
    ```
    ```
    -- 3. Daily order volume

    SELECT order_date, COUNT(*) AS total_orders

    FROM retail.orders

    GROUP BY order_date

    ORDER BY total_orders DESC

    LIMIT 10;
    ```

### Task 2: Monitor Performance Metrics

1.  Minimize the Azure CLI window. In the Azure Portal, go to PostgreSQL
    Flexible Server.

    ![](./media/image67.png)

2.  Expand Monitoring from the left-hand side menu and select Metrics.

    ![](./media/image68.png)

3.  In the chart, select **CPU percent** under Metric.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image69.png)

4.  Set the local time to the last hour so that we can see the metrics
    for the latest executed queries. And select **Apply**.

    ![A screen shot of a computer AI-generated content may be
    incorrect.](./media/image70.png)

5.  This metric displays the percentage of CPU resources currently being
    utilized by your PostgreSQL server. Use this chart to identify
    periods of high query processing or when your workload is consuming
    more compute power than expected

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image71.png)

6.  Click on **+New Chart**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image72.png)

7.  Add another chart and select **Memory percent** under **Metric**.

    ![](./media/image73.png)

8.  This metric shows how much of the server’s available memory is being
    used. Consistently high memory usage can indicate that queries are
    performing large sorts, joins, or caching operations, and may
    benefit from scaling or query optimization.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image74.png)

9.  Again, click on **+New Chart**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image75.png)

10. In the new chart, from the Metric dropdown, select **Storage used**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image76.png)

11. This metric shows how much of your allocated database storage is
    currently being utilized. You can use it to monitor data growth over
    time or detect sudden increases caused by large inserts or index
    creation.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image77.png)

12. Run your SQL queries again and watch for metric spikes in real time.

    ![A screenshot of a computer screen AI-generated content may be
    incorrect.](./media/image78.png)

## Exercise 5: Enable Microsoft Fabric Mirroring for Real-Time Analytics on PostgreSQL Flexible Server

### Task 1: Prepare PostgreSQL Flexible Server for Fabric Mirroring

1.  Minimize the Azure CLI window. From the left-hand menu, select
    **Settings → Server parameters**.

    ![](./media/image79.png)

2.  Search for **max_replication_slots** parameter and make sure it is
    set to 10.

    ![](./media/image80.png)

3.  Search for **max_wal_senders** parameter and make sure it is set to
    10.

    ![](./media/image81.png)

4.  Search for **wal_level** parameter and set its value to logical.

    ![](./media/image82.png)

5.  Click **Save** and then **Save and Restart**.

    ![](./media/image83.png)

6.  Wait for a few minutes to complete the deployment.

    ![](./media/image84.png)

7.  Click on **Go to resource**.

    ![](./media/image85.png)

8.  Expand **Security** from the left-hand menu and then select
    **Identity**.

    ![](./media/image86.png)

9.  Select off to turn off system system-assigned managed identity.
    Click **Save**.

    ![](./media/image87.png)

10. Wait for we second to complete the deployment. Then click **Go to
    resource**.

    ![](./media/image85.png)

11. From the left-hand menu, select **Fabric Mirroring**.

12. Click **Get started**.

13. Select the zava_connect_retail database and click **Prepare**.

14. Wait for a few seconds, and under Mirroring readiness, you will get
    a message “Server is ready for mirroring”.

    ![](./media/image88.png)

15. Open cloud shell.

16. Run the following command to enable azure_cdc extension.

    +++ CREATE EXTENSION azure_cdc;+++

17. Enter +++\q+++ to close the connection.

Now your server is ready for fabric mirroring.

### Task 2: Create a Fabric Workspace

1.  Open **Microsoft Fabric** +++https://fabric.microsoft.com+++.

2.  Click **New** **Workspaces**.

    ![](./media/image89.png)

3.  Enter workspace name +++zavaworkspace@lab.LabInstance.Id+++.

    ![](./media/image90.png)

4.  Scroll down and expand Advanced. Under Licence mode, ensure the
    capacity is selected. If you are using a trial version, then it will
    automatically choose trial. In my case, I’m using Fabric Capacity so
    it selects Fabric Capacity. Click **Apply**.

    ![](./media/image91.png)

Now your workspace is ready for mirroring.

    ![](./media/image92.png)

### Task 3: Create a Mirrored Database in Microsoft Fabric

1.  Click **+New Item**.

    ![](./media/image93.png)

2.  Scroll down and under the Mirror data section, select **Mirrored
    Azure Database for PostgreSQL(preview)**.

    ![](./media/image94.png)

3.  Select **Azure Database for PostgreSQL**.

    ![](./media/image95.png)

4.  Enter the connection details:

    - Server name: Enter
    +++zava-connect-db@lab.LabInstance.Id.postgres.database.azure.com+++

    - Database name: Enter +++zava_connect_retail+++

    - Username: Enter +++postgres+++

    - Password: Enter +++ PAssw0rd1a+++

    Click **Connect**.

    ![](./media/image96.png)

5. Select retail.categories, retail.prodects, and retail.customers table for mirroring. Click **Connect**. 

    ![](./media/image97.png)

6.  Verify the database name and then click **Create mirrored
    database**.

    ![](./media/image98.png)

7.  Wait for few minutes to complete the mirroring of database.

    ![](./media/image99.png)

8.  You will see all the tables and their data that you have mirrored.

    ![](./media/image100.png)

    **Note:** If you did not see any data, then after 1-2 minutes, click
    on the Refresh button, and your data will appear on the screen.

    So Fabric Mirroring is running successfully for your zava_connect_retail
    database

### Task 4: Query Mirrored PostgreSQL Data in Fabric

1.  Select Query in T-SQL.

    ![](./media/image101.png)

2.  Click **New SQL query**.

    ![](./media/image102.png)

3.  Run the following query to validate real-time analytics and click
    Run button to execute the query:

    +++SELECT TOP 10 * FROM [retail].[customers];+++

    ![](./media/image103.png)

4.  Observe that the query runs **entirely in Fabric**, without touching
    the PostgreSQL server.

    ![](./media/image104.png)

### Task 5: Validate Real-Time Data Synchronization

1.  Open **Cloud Shell** and run the following command to connect to the
    zava_connect_retail database.

    +++psql
    "host=zava-connect-db@lab.LabInstance.Id.postgres.database.azure.com
    port=5432 dbname=zava_connect_retail user=postgres password=P@ssw0rd#a
    sslmode=require"+++

    ![](./media/image105.png)

2.  Insert a new order into PostgreSQL:
    ```
    INSERT INTO retail.customers (customer_id, first_name, last_name, email)

    VALUES (9999, 'Test', 'User', 'testuser@zava.com');
    ```

    ![](./media/image106.png)

3.  Wait **10–30 seconds**.

4.  Navigate back to the Fabric portal window and re-run the same SQL
    query in **Fabric**.
    ```
    SELECT TOP 10 * 

    FROM [retail].[customers] 

    ORDER BY customer_id DESC; 
    ```

    ![](./media/image107.png)

5.  Confirm that the new data appears in the results.

    ![](./media/image108.png)

This verifies **real-time replication** from PostgreSQL to Fabric.

## Exercise 6: Ensure High Availability & Backups

After enabling Microsoft Fabric Mirroring, the ZAVA Connect platform is
ready for production hardening. Since zonal high availability was
intentionally disabled during initial provisioning to simplify Fabric
Mirroring setup, the DBA now configures **alternative high availability
and disaster recovery mechanisms** using PostgreSQL Flexible Server
features that can be enabled post-deployment.

### Task 1: Setting High Availability Configuration

1.  Navigate back to **Azure Portal**.

2.  In PostgreSQL Flexible Server, expand Settings from the menu and
    then select **High availability**.

    ![](./media/image109.png)

3.  Select **Enabled** to enable **Zonal resiliency**. This ensures that
    your database is deployed across availability zones for fault
    tolerance.
    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image110.png)

4.  Click **Save**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image111.png)

5.  Click **Enable high availability**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image112.png)

### Task 2: Review Backup and disaster recovery Configuration

1.  Click **Go to resource**.

    ![](./media/image113.png)

2.  Expand Settings from the left-hand menu and then select **Backup and
    restore**.

    ![](./media/image114.png)

3.  Confirm that Backup redundancy is set to Geo-redundant and backups
    are taking automatically.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image115.png)

Geo-redundant backups store copies in a paired region, ensuring business
continuity even in regional outages.

### Task 3: Configure Read Replica

1.  In the left-hand menu, select **Replication**. Click **+Create
    Replica**.

    ![](./media/image116.png)

2.  Enter the following details:

    - **Resource group**: Select ResourceGroup1

    - **Server name**: Enter +++zava-connect-replica@lab.LabInstance.Id+++

    - **Location**: Central US

    Click **Next: Networking**

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image117.png)

3.  Keep all the default settings. Click **Review+create**.

    ![](./media/image118.png)

4.  Click **Create**.

    ![](./media/image119.png)

5.  Wait for a few minutes to complete the deployment. It will take 5-10
    mins.

6.  After completing the deployment, click **Go to resource**.

    ![](./media/image120.png)

    ![](./media/image121.png)

7.  Navigate back to the Replication window of your primary server. You
    should now see the **zava-connect-replica@lab.LabInstance.Id**
    listed as an active read replica.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image122.png)

**Replication helps improve performance and availability** by offloading
read workloads to replica servers and ensuring rapid failover in case
the primary database becomes unavailable.

## Summary

In this lab, you built and secured the data foundation for ZAVA Connect
using Azure Database for PostgreSQL – Flexible Server. You provisioned
and optimized the database, enabled pgvector for AI-ready workloads, and
loaded the ZAVA DIY retail dataset. You verified secure connectivity,
monitored performance metrics, and executed analytical queries to
validate the environment. Microsoft Fabric Mirroring was enabled to
support near real-time analytics without impacting operational
workloads. By the end, ZAVA DIY achieved a modern, secure, scalable, and
resilient data platform ready to power smarter retail operations and
future innovation.
