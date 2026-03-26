# Lab 02 - End-to-End Migration of ZAVA's On-Prem PostgreSQL to Azure Database for PostgreSQL - Flexible Server (Homogeneous Migration)

## Scenario

ZAVA Express Logistics, a branch of ZAVA DIY, manages regional freight
shipping, fleet operations, warehouse inventory, and last-mile delivery.
Traditionally, the branch relied on on-premises systems and manual
processes to coordinate deliveries, track vehicles, and manage stock,
which led to frequent data silos, operational inefficiencies, and
delayed insights - especially during peak periods. To address these
challenges, ZAVA's leadership launched a cloud modernization initiative
aimed at centralizing operations, streamlining workflows, and enabling
real-time decision-making across the branch. Under the guidance of
Carlos Vega, CTO, the team conducted a thorough assessment of the
branch's on-premises environment, identifying over 50 operational
systems and selecting one representative workload for a pilot migration.
This workload, a web-based branch management system running on Red Hat
Enterprise Linux (RHEL) servers and connected to a PostgreSQL database,
was chosen because it exemplified the common components found across
other branch systems while remaining manageable for testing the
migration plan.

The branch migration project leverages Azure Database for PostgreSQL -
Flexible Server, chosen for its scalable performance, built-in security,
high availability, and AI-readiness through pgvector and Azure AI
extensions - capabilities that will support predictive route
optimization, vehicle maintenance insights, and intelligent delivery
recommendations in the future. Carlos Vega assigned this task to Marcus
Dwyer, the newly introduced cloud migration specialist, who is
responsible for executing the migration, coordinating cutover,
validating data integrity, and minimizing downtime.

Your role in this lab:

Step into the role of **Marcus Dwyer (Cloud Migration Specialist)** to
migrate the on-premises branch database to Azure Database for PostgreSQL
Flexible Server, a fully managed cloud database service.

## Objective

In this lab you will learn:

- Simulate an on-premises branch environment for ZAVA Express Logistics
  in Azure.

- Deploy and configure a web-based branch management application on
  Linux virtual machines.

- Connect the application to an internal PostgreSQL database and verify
  functionality.

- Provision an Azure Database for PostgreSQL Flexible Server as a
  managed cloud database.

- Migrate the on-premises PostgreSQL database to the Azure Flexible
  Server securely and efficiently.

- Validate the database migration, ensuring data integrity, performance,
  and availability.

- Understand the end-to-end process of migrating representative
  workloads from on-premises to Azure in a cloud modernization scenario.

## Exercise 1 - Lab Setup

In this exercise, you will deploy a simulated on-premises environment
for ZAVA Express Logistics using a custom ARM template. They will
configure Linux-based virtual machines and set up a web-based branch
management application to connect to an internal PostgreSQL database.
Participants will connect to the VMs securely using Bastion, install
required utilities, clone configuration scripts, and validate the
application, ensuring the on-premises workload is operational and
accessible.

### Task 1 - Create resources

In this task, you will leverage a custom Azure Resource Manager (ARM)
template to deploy the Azure resources and create a simulated
on-premises environment for ZAVA Express Logistics.

1.  Open a browser and navigate to +++https://portal.azure.com/+++. Now,
    sign in with your account.
        
    - Username - +++@lab.CloudPortalCredential(User1).Username+++

    - TAP Token - +++@lab.CloudPortalCredential(User1).AccessToken+++

    ![](./media/image1.png)

    ![](./media/image2.png)

    ![](./media/image3.png)

2.  Open a new tab in the browser, and navigate to the following link to
    get the ARM template: 

    +++https://github.com/technofocus-pte/MigrateLinuxworkloads/tree/main/resources/deployment+++

3.  Select **Deploy to Azure**. This will open a new browser tab to the
    Azure Portal for custom deployments.
    
    ![The GitHub page with Deploy to
    Azure button highlighted.](./media/image4.png)

4.  Fill in the required ARM template parameters.

    - **Subscription:** **@lab.CloudSubscription.Name**

    - **Resource group:** **@lab.CloudResourceGroup(ResourceGroup1).Name**

    - **Region:** **@lab.CloudResourceGroup(ResourceGroup1).Location**

    - **Resource Name Base:** +++ZavaWeb@lab.LabInstance.Id+++

    - **Password:** +++pass@dmIn234+++

    Select **Review + create.**

    ![](./media/image5.png)

5.  Click on the **Create** button to start deployment.

    ![](./media/image6.png)

6.  The deployment is now underway. On average, this process can take
    10-20 minutes to complete.

    ![](./media/image7.png)

    >[!Note] While automation can make things simpler and repeatable,
    sometimes it can fail. If at any time during the ARM template deployment
    there is a failure, review the failure, delete the Resource Group, and
    try the ARM template again. Or review the failures and adjust for errors
    as appropriate.

7.  Once the ARM template is deployed successfully, the status will
    change to complete. Click on **Go to resource group** to open the
    resource group.

    ![](./media/image8.png)

### Task 02 - Configure on-premises web application

In this task, you will configure the web application hosted on the
simulated on-premises APP virtual machine that was provisioned by the
ARM Template deployment.

1.  Select the **On-premises Workload VM** named similar
    to **ZavaWeb-onprem-workload-vm** present on page 2.

    ![](./media/image9.png)

2.  In the **Overview** page, go to **Properties**, and under the
    **Networking** section, locate the **Private IP Address** of the VM
    and copy it into Notepad.

    ![](./media/image10.png)

    >[!Note]You will need this IP address to configure the web application
    to use the database workload server.

3.  Navigate back to the **ResourceGroup1**, then select
    the **On-premises APP VM** named similar
    to **ZavaWeb-onprem-app-vm**.

    ![](./media/image11.png)

4.  Search for +++**Bastion**+++ in the left-hand menu and then select
    **Bastion**. We will use a bastion host as the method to connect to
    our VMs, as this is a more secure method.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image12.png)

5.  Within the **Bastion** page, enter the following details:

    - **Authentication Type:** VM Password

    - **Username:** +++demouser+++

    - **VM Password:** +++pass@dmIn234+++

    Then click on the **Connect** button to connect with Bastion.

    ![](./media/image13.png)

    >[!Note] You may need to **allow pop-ups** if they are blocked in your browser.

6.  When connected to the VM via the Bastion host, you will get a screen
    like this:

    >[!Note] If you see a pop-up stating "See test and images copied to the
    clipboard". Click **Allow**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image14.png)

7.  Once connected via Bastion, run the following command to install the
    git utility on the server by using the clipboard within the
    session:   

    ![A screen shot of a computer AI-generated content may be
    incorrect.](./media/image15.png) 

8.  Click the arrows, which will expand the window

    ![Clipboard within the Bastion session.](./media/image16.png)

9.  Copy the command below and paste it into the clipboard. 

    +++sudo yum install git  -y+++

    ![Clipboard within the Bastion session.](./media/image17.png) 

10. Now, right-click on the window, which will paste the command and run
    it. 

    ![](./media/image18.png)  

    >[!Note] Similarly, you can run all these commands in the Bastion.

11. Enter the following command to clone the remote git repository
    holding a script which will configure the web app on the application
    server.

    +++sudo git clone https://github.com/technofocus-pte/TechExcel-Migrate-Linux-workloads.git+++

    ![](./media/image19.png)

12. You can now run the configuration script by using the following
    command:

    +++sudo bash TechExcel-Migrate-Linux-workloads/resources/deployment/onprem/APP-workload-install.sh+++

    You will get a status message of  "The script was successful".

    ![](./media/image20.png)

13. Execute the following command to open the **orders.php** file for
    the web application in a text editor. The application needs to be
    configured to connect to the **Azure Database for PostgreSQL
    Flexible Server** database.

    +++sudo nano /var/www/html/orders.php+++

14. Use the **down arrow** key to scroll down in the order.php file
    until you locate $host, $port, $dbname, $user, and $password.

    ![A screen shot of a computer AI-generated content may be
    incorrect.](./media/image21.png)

15. Check the host IP address and configure it to match the **Private IP
    Address** of the **ZavaWeb-onprem-workload-vm** instance. If the
    host IP is already correct, skip to steps 16. And press **Ctrl+X**
    to exit the editor.

    ![](./media/image22.png)

16. If the host IP is not the same, then replace it with the **Private
    IP Address** of the **ZavaWeb-onprem-workload-vm** instance that was
    copied in step 2. Then press **Ctrl+X -\> +++y+++-\>Enter** to save
    the changes.

    ![](./media/image23.png)

You are now exited from the orders.php file with the changes saved.

You have now learnt some basic Linux commands and configured the web
application to use the database on an internal network rather than
across the internet.

### Task 03 - Validate on-premises web application

In this task, you will validate the web application hosted on the
simulated on-premises APP virtual machine that was provisioned by the
ARM Template deployment.

1.  Navigate back to **Azure Portal**, open the **ResourceGroup1**, then
    select the **On-premises APP VM** named similar
    to **ZavaWeb-onprem-app-vm**

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image24.png)

2.  In the **Overview** window, locate the VM's **Public IP Address**
    and copy it into Notepad.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image25.png)

3.  Open a new browser window, then navigate to the **public ip address** to access the simulated on-premises web
    application provisioned for this lab,
    
    **For Example:** http://172.206.126.43

    +++http://*ip-address*+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image26.png)

    >[!Note]You should get the Red Hat Enterprise Linux Test Page 

4.  When the web page loads, enter the following at the end of the URL.
    
    **For example:** "http://172.206.126.43/orders.php"

    +++http://*ip-address*/orders.php+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image27.png)

    Now, things are ready for you to go through the lab.

## Exercise 2 - Migrate a PostgreSQL Database

In this exercise, you will migrate the on-premises PostgreSQL database
for the web application workload to Azure. The migration service in
Azure Database for PostgreSQL will be used to perform the database
migration from the PostgreSQL server on-premises to the Azure Database
for PostgreSQL service.

### Task 01 - Create Azure Database for PostgreSQL

In this task, you will create a new PostgreSQL database that will be the
target for the database migration.

1.  Navigate back to the **Azure portal-\>Home-\>Create a resource**.

    ![A screenshot of a computer AI-generated content may be incorrect.](./media/image28.png)

2.  In the **Marketplace** window, search for +++**PostgreSQL
    flexible**+++, then select **Azure Database for PostgreSQL Flexible
    Server** from the search results.

    ![A screenshot of a computer AI-generated content may be incorrect.](./media/image29.png)

3.  Click **Create** and then select +++Azure Database for PostgreSQL Flexible Server+++.

    ![A screenshot of a computer AI-generated content may be incorrect.](./media/image30.png)

4.  On the **New Azure Database for PostgreSQL** **Flexible
    Server** pane, select the following values:

    1.  **Subscription:** @lab.CloudSubscription.Name

    2.  **Resource group:** @lab.CloudResourceGroup(ResourceGroup1).Name

    3.  **Server name:** +++zavaweb-db@lab.LabInstance.Id+++

    4.  **Region:** @lab.CloudResourceGroup(ResourceGroup1).Location 

    5.  **PostgreSQL version:** Keep the default, as it always selects
        the latest version

    6.  **Workload type:** **Dev/Test**

    ![](./media/image31.png)

5.  Under **Compute + storage,** click **Configure server**.

6.  On the Compute + storage window, under the **compute** section,
    choose the following:

    1.  **Compute tier:** General Purpose (2-64 vCores) - Balanced
        configuration for most common workloads

    2.  **Compute Size:** Standard_D4ds_v5 (4 vCores, 16GiB memory, 6400
        max iops)

    ![](./media/image32.png)

7.  Under the **High availability** section, choose **Disabled (99.9%
    SLA)** and then click **Save.**

    ![](./media/image33.png)

8.  On the **New Azure Database for PostgreSQL** **Flexible Server**
    window, under the **Authentication** section, enter the following
    details:

    1.  **Authentication method**: Choose **PostgreSQL authentication
        only**

    2.  **Admin username**: +++pgadmin+++

    3.   **Password**: +++passaDmin342+++

    Select **Next: Networking**

9.  In the **Networking** tab, under **Public access**, clear the
    checkbox to disable public access.

    ![](./media/image35.png)

10. Under **Private endpoint** section, select **Create private
    endpoint.**

    ![](./media/image36.png)

11. On the **Create private endpoint** window, enter the following
    details:

    1.  **Subscription:** @lab.CloudSubscription.Name

    2.  **Resource group:** @lab.CloudResourceGroup(ResourceGroup1).Name

    3.  **Location**: @lab.CloudResourceGroup(ResourceGroup1).Location 

    4.  **Name:** +++post-priv-endpoint+++

    5.  **Virtual network:** +++ZavaWeb@lab.LabInstance.Id-spoke-vnet (ResourceGroup1)+++

    6.  **Subnet**: Select **default**(10.2.0.0/24)

    7.  **Integrate with privet DNS zone:** Select **No** (You will use an
        IP address rather than a DNS entry when connecting.)

    8.  Click **OK**

    ![](./media/image37.png)

12. Select **Review + create**.

    ![](./media/image38.png)

13. Select **Create** to provision the service.

    ![](./media/image39.png)

14. Click **Create server without firewall rules** - as you will use the
    private endpoint for access.

    ![](./media/image40.png)

15. Wait for deployment to complete; it will take 5-10 mins. Once
    provisioning has finished, click on **Go to resource**.

    ![](./media/image41.png)

    ![](./media/image42.png)

16. In the Overview tab of the **Azure Database for PostgreSQL flexible
    server,**  copy and save the **Server name** in Notepad for use
    later.

    ![](./media/image43.png)

### Task 02 - Migrate your database to Azure Database for PostgreSQL flexible server

In this task, you will set up a migration project and configure the
Source and Target connections. You will then execute and monitor a
migration of your on-premises PostgreSQL database into Azure Database
for PostgreSQL - flexible server.

1.  Select **Migration** from the menu on the left of the flexible
    server blade.

    ![](./media/image44.png)

2.  Click on the **+ Create**.

    ![](./media/image45.png)

    >[!Note] If the **+ Create** option is unavailable,
    select **Compute + storage** and change the compute tier to
    either **General Purpose** or **Memory Optimized** and try to create
    the Migration process again. After the Migration is successful, you
    can change the compute tier back to **Burstable**.

3.  On the **Setup** tab, enter each field as follows:

    1.  **Migration name:** +++Migration-Zava-northwind+++

    2.  **Source server type:** On-premise Server.

    3.  **Migration option:** Validate and Migrate.

    4.  **Migration option:** Offline

    5.  Select **Next: Runtime server \>**

    ![](./media/image46.png)

    >[!Note]The Runtime server \> button might be enabled after 20-30 mins.

4.  We will **not** use a Runtime Server, so just select **Next: Source
    server \>**.

    ![](./media/image47.png)

5.  On the **Source server** tab, enter each field as follows:

    1.  **Server name:** The public IP address of the
        "ZavaWeb-onprem-workload-vm".

    2.  **Port:** 5432

    3.  **Server admin login name:** +++rootuser+++ (the VM has been setup with
        an admin user called rootuser)

    4.  **Password:** +++123rootpass456+++

    5.  **SSL mode:** Prefer.

    6.  Click on the **Connect to source** option to validate the
        connectivity details provided.

    7.  Click on the **Next: Target server\>** button to progress.

    ![](./media/image48.png)

6.  The connectivity details should be automatically completed for the
    target server we are migrating to.

    1.  In the password field - +++passaDmin342+++

    2.  Click on the **Connect to target** option to validate the
        connectivity details provided.

    3.  Click on the **Next: Databases to validate or migrate \>** button to progress.

    ![](./media/image49.png)

7.  On the **Databases to validate or migrate** tab, select the
    **northwind** database because you want to migrate to the flexible
    server. Then click on the **Next : Summary \>** button to progress
    and review the data provided.

    ![](./media/image50.png)

8.  On the **Summary** tab, review the information and then click
    the **Start Validation and Migration** button to start the migration
    to the flexible server.

    ![](./media/image51.png)

9.  On the **Migration** tab, you can monitor the migration progress by
    using the **Refresh** button in the top menu to view the progress
    through the validation and migration process.

    ![](./media/image52.png)

10. By clicking on the **Migration-northwind** activity, you can view
    detailed information about the migration activity's progress.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image53.png)

    During this migration, a database will be automatically created in the
    PostgreSQL Flexible Server, and the data will be migrated into it.

### Task 03: Validate Database Migration

1.  Click the **Back** button to return to the **Migration** window.

    ![](./media/image54.png)

2.  From the left-hand menu, select **Settings**, then select
    **Networking**.

    ![](./media/image55.png)

3.  Click **Private endpoint**.

    ![](./media/image56.png)

4.  In the **Private endpoint** page, locate and click the **Network
    interface** link.

    ![](./media/image57.png)

5.  In the **Network Interface Overview** page, locate the **Private
    IPv4 address** and copy it to **Notepad** for later use. Using this
    IPv4 address, we will connect to the PostgreSQL flexible server.

    ![](./media/image58.png)

6.  Navigate to **Home-\> ResourceGroup1** and select
    **ZavaWeb-onprem-app-vm**.

    ![](./media/image59.png)

7.  In the Overview tab, click **Connect -\>Connect as Bastion**.

    ![](./media/image60.png)

8.  Within the **Bastion** page, enter the following details:

    - **Authentication Type:** VM Password

    - **Username:** +++demouser+++

    - **VM Password:** +++pass@dmIn234+++

    Click **Connect**.

    ![](./media/image61.png)

9.  Run the following command to connect to Azure PostgreSQL in the VM.

    +++psql -h 10.2.0.4 -U pgadmin -d northwind -p 5432+++

    When prompted, enter:

    +++passaDmin342+++

    ![](./media/image62.png)

10. Run the following command to list all tables in the database:

    +++dt+++

    ![](./media/image63.png)

11. Run the following SQL queries to verify record counts in key tables:

    +++SELECT COUNT(\*) FROM orders;+++

    +++SELECT COUNT(\*) FROM customers;+++

    ![](./media/image64.png)

12. Retrieve a small sample of data from the database to verify
    accuracy:

    +++SELECT * FROM orders LIMIT 5;+++

    ![](./media/image65.png)

This confirms that **data was migrated without corruption**.

## Summary

In this lab, you **simulated** an on-premises branch environment for
ZAVA Express Logistics by deploying Linux VMs and configuring a
web-based branch management application connected to a PostgreSQL
database. You **provisioned** an Azure Database for PostgreSQL Flexible
Server and **migrated** the on-premises database to Azure. The lab
**covered** validating the migration, ensuring data integrity, and
optimizing performance. You **gained** hands-on experience in deploying,
configuring, and migrating workloads from on-premises to Azure,
demonstrating a real-world cloud modernization process.

===

# Lab 3: Migrating ZAVA's Oracle ERP Database to Azure Database for PostgreSQL - Flexible Server (Heterogeneous Migration)

## Scenario

ZAVA DIY, a leading home improvement and retail chain, has recently
expanded its e-commerce operations, managing nationwide online orders,
customer accounts, and warehouse inventory. The ZAVA E-Commerce Division
relies on a legacy Oracle ERP system hosted on-premises to track orders,
stock movements, and generate reports. With growing transaction volumes
and seasonal spikes, the on-premises system has become a bottleneck,
causing delayed order processing and inconsistent inventory data.

To address these challenges, ZAVA's executive team launched a cloud
modernization initiative aimed at centralizing the ERP database,
enabling real-time insights, and improving scalability. Carlos Vega,
CTO, identified the **Order Management Module** of the Oracle ERP as the
ideal candidate for a pilot migration. This module handles transactional
data for online orders, inventory updates, and customer interactions,
representing the complexity of the larger ERP system.

ZAVA's target platform is **Azure Database for PostgreSQL - Flexible
Server**, chosen for its high availability, scalable performance, and AI
capabilities through **pgvector** and **Azure AI extensions**. These
features will eventually support predictive stock replenishment,
intelligent order routing, and customer personalization.

Carlos Vega assigned **Marcus Dwyer**, the Cloud Migration Specialist,
to lead the migration using **Ora2Pg**, a tool designed for
heterogeneous migrations from Oracle to PostgreSQL. Marcus's
responsibilities include:

- Assessing the existing Oracle database and workloads.

- Configuring Ora2Pg to analyze schemas, convert data types, and
  generate PostgreSQL-compatible SQL scripts.

- Migrating the Order Management Module to Azure Database for PostgreSQL
  - Flexible Server.

- Validating data integrity, ensuring referential constraints are
  preserved, and testing end-to-end workflows.

**Your role in this lab:**

Step into the role of Marcus Dwyer (Cloud Migration Specialist) to
migrate ZAVA DIY's on-premises Oracle ERP Order Management Module to
**Azure Database for PostgreSQL Flexible Server** using **Ora2Pg**. You
will:

1.  Configure Ora2Pg for schema extraction and conversion.

2.  Generate and review PostgreSQL-compatible SQL scripts.

3.  Load data into Azure Database for PostgreSQL - Flexible Server.

4.  Validate data consistency, run sample queries, and ensure that
    critical e-commerce workflows continue to function seamlessly in the
    cloud.

## Task 1: Provision PostgreSQL Flexible Server

In this task, you will create the target Azure Database for PostgreSQL
Flexible Server that will host the migrated Oracle ERP data. This server
will serve as the destination platform for schema and data migration
using Ora2Pg.

1.  Open a web browser and sign in to the Azure
    portal at +++https://portal.azure.com+++ using the following cloud
    slice credentials.

    - Username - +++@lab.CloudPortalCredential(User1).Username+++

    - TAP Token - +++@lab.CloudPortalCredential(User1).AccessToken+++

    ![](./media/image1.png)

    ![](./media/image2.png)

    ![](./media/image3.png)

2.  In the search bar, enter +++PostgreSQL Flexible Server+++ and then
    select **azure database for PostgreSQL flexible servers**.

    ![](./media/image66.png)

3.  Click **+Create**.

4.  Now enter the following details:

    **Resource group:** Select **ResourceGroup1**

    **Server name:** Enter +++postgrenewserver@lab.LabInstance.Id+++

    **Region:** @lab.CloudResourceGroup(ResourceGroup1).Location

    **PostgreSQL Version:** 16

    **Workload type:** Select **Dev/Test**

5.  Under Authentication, select **PostgreSQL authentication only** and
    then enter the following credential values:

    - Username: +++pgAdmin+++

    - Password: +++pAssw0rd1289+++
	
	![](./media/image34.png)

    Select **Networking**
	

6.  In the networking tab, please ensure that the connectivity method is
    set to Public access and "Allow public access to this resource
    through the internet using a public IP address" is enabled.

    

7.  Under the Firewall section, enable **Allow public access from any
    Azure service within Azure to this server** and click **+Add current
    client IP address**. Then it will add a firewall rule for your
    current IP. Click **Review + Create**.

8.  Click **Create**.

9.  Wait for 10-15 mins to complete the deployment.

10. Click **Go to resource**.

    ![](./media/image77.png)

13. On the left-hand navigation menu, click on **Settings-\>Server
    parameters**.

    

12. In the search bar, enter +++azure.extension+++.

	![](./media/image73.png)

13. From the drop-down menu, select **BTREE_GIN** and **PG_TRGM**
    extensions.
	
	
    ![](./media/image74.png)

    ![](./media/image75.png)

14. Click **Save**.

15. After completing the deployment successfully. Click on the **Home**
    tab.

## Task 2: Create a virtual machine

In this task, you will provision an Oracle Linux virtual machine in
Azure to host the source Oracle database and migration tools. This VM
represents the on-premises Oracle environment used for the migration
exercise.

1.  Navigate to the Home page of the Azure portal. Select **Virtual
    machines** from the Azure services.

    ![](./media/image78.png)

2.  Click **+Create**.

3.  Select **Virtual Machine**.

    ![](./media/image80.png)

4.  In the Create a virtual machine window, enter the following details:

    - **Subscription:** @lab.CloudSubscription.Name

    - **Resource group:** @lab.CloudResourceGroup(ResourceGroup1).Name

    - **Virtual machine name:** +++oracleVirtual@lab.LabInstance.Id+++

    - **Region:** @lab.CloudResourceGroup(ResourceGroup1).Location

    - **Image:** +++Oracle Linux 8.10 (LVM)-x64 Gen2+++

    Other than the above settings, keep the remaining as the default.

    Click **Review + create**.

    ![](./media/image81.png)

    ![](./media/image82.png)

5.  Click **Create**.

    ![](./media/image83.png)

6.  Click **Download private key and create a resource**.

    ![](./media/image84.png)

7.  Your deployment has started. Wait for 5-10 mins to complete.

	![](./media/image79.png)

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image85.png)

    Your VM has been deployed successfully.

    ![](./media/image86.png)

8.  Now click **Go to resource.**

    ![](./media/image87.png)

9.  Locate and save the public IP of your VM in Notepad. You will use it
    in the next steps for the connection.

    ![](./media/image88.png)

10. Now you will set up SSH. Open **PowerShell** as administrator and
    run the following command to check if the .ssh folder exists in your
    system.

    +++dir $HOME\\ssh+++

    If you get an error like this, that means this folder is not available.

    ![A computer screen shot of a blue screen AI-generated content may be
    incorrect.](./media/image89.png)

11. Run the following command to create the .ssh folder in the home
    directory.

    +++mkdir $HOME\\ssh+++

    It will provide the directory location where it creates the .ssh file.
    So save this location.

    >[!Note]This location may vary for every VM.

    ![](./media/image90.png)

12. Locate your downloaded **".pem file**" in the File Explorer and copy
    it.

    ![](./media/image91.png)

13. Paste it into the given directory. The path will look like this:

    C:\Users\Admin

    So the actual path will look like this:

    C:\Users\Admin\\ssh\oraclelinux-key.pem

    ![](./media/image92.png)

14. Navigate back to the PowerShell and run the following command to
    know the current user:

    +++whoami+++

    Save the output of this command in notepad for future use.

    ![](./media/image93.png)

15. Set the permission for .ssh file.

    +++icacls "$HOME\\ssh\oracleVirtual_key.pem" /inheritance:r+++

    +++icacls $HOME\\ssh\oracleVirtual_key.pem /grant "\<CURRENT
    USER\>:R"+++

    >[!Note] Get the current user value from the previous step.

    For example:

    +++icacls "$HOME\\ssh\oracleVirtual_key.pem" /inheritance:r+++

    +++icacls $HOME\\ssh\oracleVirtual_key.pem /grant "base22c\Admin:R"+++

    ![](./media/image94.png)

16. Run the following command to connect to your VM:

    +++ssh -i "$HOME\\ssh\oracleVirtual_key.pem"
    azureuser@\<VM_PUBLIC_IP\>+++

    For Example:

    +++ssh -i "$HOME\\ssh\oracleVirtual_key.pem" azureuser@20.83.43.16+++

    When you first connect with your VM, it will prompt: "Are you sure you
    want to continue connecting (yes/no)?", the you have to enter +++yes+++

    ![](./media/image95.png)

    Now you are successfully connected to the VM.

17. Exit from the VM using the following command:

    +++exit+++

    ![](./media/image96.png)

## Task 3: Setup the Oracle Environment

In this task, you will install and configure Oracle Database 21c Express
Edition on the virtual machine. This database simulates the legacy
Oracle ERP system used by ZAVA's Order Management module.

1. Open the **Edge** browser.

1. Navigate to +++https://download.oracle.com/otn-pub/otn_software/db-express/oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm+++ to download Oracle Database.

1.  Navigate to the PowerShell and upload this .rmp file to the VM using
    the following command:

    +++scp -i "$HOME\\ssh\oracleVirtual_key.pem"
    "$HOME\Downloads\oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm"
    azureuser@\<Your-VM-IP\>:~/+++

    For Example:

    +++scp -i "$HOME\\ssh\oracleVirtual_key.pem" "$HOME\Downloads\oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm" azureuser@20.83.53.154:~/+++

    Wait for some time to download.

    ![](./media/image97.png)

2.  Connect to the VM using the following command:

    +++ssh -i "$HOME\\ssh\oracleVirtual_key.pem" azureuser@20.83.53.154+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image98.png)

3.  Run the following command to install pre-requisites for Oracle XE
    21c.

    +++sudo dnf install -y oracle-database-preinstall-21c+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image99.png)

    After completing this you will get successfully message.

    ![A computer screen shot of a blue screen AI-generated content may be
    incorrect.](./media/image100.png)

4.  Now run the following command to install Oracle Database 21c Express
    Edition in the VM.

    +++sudo dnf localinstall -y oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm+++

    ![](./media/image101.png)

5.  Configure the database:

    +++sudo /etc/init.d/oracle-xe-21c configure+++

    It will ask for a password. You can enter this +++pAssw0rd1289+++

    ![](./media/image102.png)

    Save the pulugged database name that is XEPDB1.

    >[!Note] It will take 10-15 mins to complete.

6.  Verify the status of Oracle database.

    +++sudo /etc/init.d/oracle-xe-21c status+++

    It shows that the database is running, so we have successfully
    installed and configured the Oracle Database 21c Express Edition in
    the VM.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image103.png)

7.  Confirm the path where SQLPlus is installed. Oracle Database 21c
    Express Edition comes with SQLPlus and the Instant Client, so we do
    not need to install them separately.

    +++sudo find / -name sqlplus 2\>/dev/null+++

    ![A blue screen with red text AI-generated content may be
    incorrect.](./media/image104.png)

8.  Add SQL\*Plus to PATH permanently.

    +++echo 'export ORACLE_HOME=/opt/oracle/product/21c/dbhomeXE' \>\> ~/.bashrc+++

    +++echo 'export ORACLE_SID=XE' \>\> ~/.bashrc+++

    +++echo 'export PATH=$ORACLE_HOME/bin:$PATH' \>\> ~/.bashrc+++

    +++source ~/.bashrc+++

    ![](./media/image105.png)

9.  Verify that the SQL plus is set successfully.

    +++sqlplus -v+++

    ![](./media/image106.png)

## Task 4: Create a Database in Oracle

In this task, you will create an application user and schema in Oracle
and populate it with sample ERP data. This data will later be assessed,
converted, and migrated to PostgreSQL.

1.  Connect to the Pluggable Database (PDB) as SYSDBA. This logs you in
    with full administrative privileges required for database
    configuration and migration tasks.

    +++sqlplus 'sys/pAssw0rd1289' as sysdba+++

    ![](./media/image107.png)

2.  Switched to the pluggable database. This command changes your
    session from the CDB to the target PDB so you can run operations
    inside the PDB.

    +++ALTER SESSION SET CONTAINER=XEPDB1;+++

    ![](./media/image108.png)

3.  Create the ERP user and grant the required privileges. It creates a
    new application user inside the PDB and assigns the permissions
    needed to create and manage database objects.
    
    Create a new user

    +++CREATE USER zava_erp IDENTIFIED BY Zava1234 DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;+++

    Grant privileges

    +++GRANT CONNECT, RESOURCE, CREATE TABLE, CREATE VIEW, CREATE SEQUENCE TO zava_erp;+++

    Give unlimited quota on USERS tablespace

    +++ALTER USER zava_erp QUOTA UNLIMITED ON USERS;+++

    +++ALTER PLUGGABLE DATABASE XEPDB1 SAVE STATE;+++

    ![](./media/image109.png)

4.  Run +++exit+++to exit.

    ![](./media/image110.png)

5.  Connect as the new ERP user.

    +++sqlplus zava_erp/Zava1234@localhost:1521/XEPDB1+++

    ![](./media/image111.png)

6.  Create tables and insert sample data.

    Create tables
    
    +++CREATE TABLE customers (customer_id NUMBER PRIMARY KEY, first_name VARCHAR2(50), last_name VARCHAR2(50), email VARCHAR2(100));+++
    
    +++CREATE TABLE orders (order_id NUMBER PRIMARY KEY, customer_id NUMBER REFERENCES customers(customer_id), order_date DATE, amount NUMBER(10,2));+++
    
    Insert sample data
    +++INSERT INTO customers VALUES (1, 'John', 'Doe', 'john.doe@example.com');+++
    +++INSERT INTO customers VALUES (2, 'Jane', 'Smith', 'jane.smith@example.com');+++
    
    +++INSERT INTO orders VALUES (1001, 1, SYSDATE, 250.75);+++
    +++INSERT INTO orders VALUES (1002, 2, SYSDATE, 120.50);+++
    
    +++COMMIT;+++
    
    Verify data
    +++SELECT * FROM customers;+++
    +++SELECT * FROM orders;+++
    
    ![](./media/image112.png)

7.  Verify the orders table.

    +++SELECT * FROM orders;+++

    ![](./media/image113.png)

8.  Check constraints and table structure.

    +++DESCRIBE customers;+++

    +++DESCRIBE orders;+++

    And exit from the current console using +++exit+++.

    ![](./media/image114.png)

## Task 5: Install and Set Up the Ora2pg tool

In this task, you will install Ora2Pg along with the required Oracle
client libraries and Perl dependencies. Ora2Pg will be used to assess
compatibility and perform the Oracle-to-PostgreSQL migration.

1.  Install **perl** using the following command.

    +++sudo dnf install -y perl perl-core perl-devel+++

    ![](./media/image115.png)

2.  Verify the **perl version**.

    +++perl -v+++

    ![](./media/image116.png)

3.  Install required Perl dependencies for Ora2Pg:

    +++sudo dnf install -y perl perl-DBI perl-DBD-Pg gcc make+++

    ![A screenshot of a computer program AI-generated content may be
    incorrect.](./media/image117.png)

4.  Now switch to root environment.

    +++sudo -i+++

5.  Set environment variables for root:

    +++export ORACLE_HOME=/opt/oracle/product/21c/dbhomeXE+++

    +++export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH+++

    +++export PATH=$ORACLE_HOME/bin:$PATH+++

    +++perl -V # optional: confirm ORACLE_HOME/bin is in PATH+++

    ![](./media/image118.png)

6.  Verify the value of the environment that you have set in the
    previous step: +++echo $ORACLE_HOME+++

    +++echo $LD_LIBRARY_PATH+++

    +++echo $PATH+++

    ![A blue screen with white text AI-generated content may be
    incorrect.](./media/image119.png)

7.  Install **libaio** and **libaio-devel** libraries

    +++sudo dnf install -y libaio libaio-devel+++

    +++sudo dnf install -y gcc make+++

    ![](./media/image120.png)

    ![](./media/image121.png)

8.  Verify the installed libraries.

    +++ls /usr/lib64/libaio*+++

    ![](./media/image122.png)

9.  Launch CPAN

    +++cpan+++

    It will prompt: Would you like to configure as much as possible
    automatically? \[yes\] then enter +++yes+++

    ![A computer screen with white text AI-generated content may be
    incorrect.](./media/image123.png)

10. Install DBD::Oracle using the following command in CPAN:

    +++force install DBD::Oracle+++

    ![](./media/image124.png)

11. Exit from CPAN using the following command:

    +++exit+++

    ![](./media/image125.png)

12. Verify DBD.

    +++perl -MDBD::Oracle -e 'print $DBD::Oracle::VERSION."\n";'+++

    ![](./media/image126.png)

13. Navigate back to the azureuser directory.

    +++su - azureuser+++

    ![](./media/image127.png)

14. Download Ora2Pg using the following command.

    +++wget https://github.com/darold/ora2pg/archive/master.zip+++

    ![](./media/image128.png)

15. Unzip the mazter.zip file.

    +++unzip master.zip+++

    ![](./media/image129.png)

16. Change the directory

    +++cd ora2pg-master+++

    ![](./media/image130.png)

17. Run the following commands to install Ora2pg:

    +++perl Makefile.PL+++

    +++make+++

    +++sudo make install+++

    ![](./media/image131.png)

18. Verify your Ora2pg using the following command:

    +++ora2pg -v+++

    ![](./media/image132.png)

19. Change the directory.

    +++cd ~+++

    ![](./media/image133.png)

20. Copy the config file to your home directory:

    +++cp /etc/ora2pg/ora2pg.conf.dist ora2pg.conf+++

    ![](./media/image134.png)

21. Edit the ora2pg.conf file.

    +++nano ora2pg.conf+++

22. In the ora2pg.conf file, locate ORACLE_DSN, ORACLE_USER, and
    ORACLE_PWD

    ![](./media/image135.png)
    Now replace the existing value with your values:

    ORACLE_DSN +++dbi:Oracle:host=localhost;port-1521;service_name=XEPDB1+++

    ORACLE_USER +++zava_erp+++

    ORACLE_PWD +++Zava1234+++

    After setting up these values, press **Ctrl+X-\>Y-\>Enter** to save
    the file.

    ![](./media/image136.png)

23. Run the following command to set environment variables:

    +++echo 'export ORACLE_HOME=/opt/oracle/product/21c/dbhomeXE' \>\> ~/.bashrc+++

    +++echo 'export LD_LIBRARY_PATH=$ORACLE_HOME/lib' \>\> ~/.bashrc+++

    +++echo 'export PATH=$ORACLE_HOME/bin:$PATH' \>\> ~/.bashrc+++

    +++source ~/.bashrc+++

    ![](./media/image137.png)

24. Verify Oracle client files exist.

    +++ls $ORACLE_HOME/lib | grep libclntsh+++

    ![](./media/image138.png)

25. Add Oracle lib path to ldconfig

    +++sudo sh -c 'echo /opt/oracle/product/21c/dbhomeXE/lib \> /etc/ld.so.conf.d/oracle.conf'+++

    +++sudo ldconfig+++

    ![](./media/image139.png)

26. Test DBD::Oracle connection.

    +++perl -MDBD::Oracle -e 'print $DBD::Oracle::VERSION'+++

    ![](./media/image140.png)

    Now the or2pg is successfully connect with Oracle database.

## Task 6: Pre-Migration Validation

In this task, you will prepare the Oracle database for migration by
gathering optimizer and dictionary statistics. This ensures accurate
cost estimation and schema analysis during the Ora2Pg assessment phase.

1.  Connect to Oracle as SYSDBA

    +++sqlplus sys/pAssw0rd1289 as sysdba+++

    ![](./media/image141.png)

2.  Switch to the Pluggable Database.

    +++ALTER SESSION SET CONTAINER=XEPDB1;+++

    ![](./media/image142.png)

3.  Gather Oracle Statistics.


    +++EXEC DBMS_STATS.GATHER_SCHEMA_STATS('ZAVA_ERP');+++
    +++EXEC DBMS_STATS.GATHER_DICTIONARY_STATS;+++
    +++EXIT;+++ 


    ![](./media/image143.png)

    This ensures accurate Ora2Pg cost and size estimates.

## Task 7: Run Ora2Pg Migration Assessment

In this task, you will generate a high-level compatibility and effort
assessment report using Ora2Pg. This report helps identify
Oracle-specific features and potential migration challenges before
execution.

1.  Generate Assessment Report.

    +++ora2pg -t SHOW_REPORT -c ~/ora2pg.conf --estimate_cost+++

    ![](./media/image144.png)Now in the generated report review-

    - Object compatibility

    - Estimated migration effort

    - Oracle-specific features requiring manual changes

## Task 8: Create Ora2Pg Migration Template

In this task, you will initialize an Ora2Pg project structure to
organize migration artifacts. This project template separates
configuration, schema exports, data files, and reports.

1.  Create Migration Working Directory.

    +++mkdir -p ~/ora_migrate+++

2.  Initialize Ora2Pg Project Template.

    +++ora2pg --project_base ora_migrate --init_project zava_project+++

    ![](./media/image145.png)

## Task 9: Configure Ora2Pg Project

In this task, you will configure the Ora2Pg project to scope the
migration to the ZAVA ERP schema. You will also define which database
objects are included in the migration.

1.  Edit Project Configuration File.

    +++cd ~/ora_migrate/zava_project/config+++

    +++nano ora2pg.conf+++

2.  In the ora2pg.conf file update Oracle connection settings. Here you
    replace the existing value with your values:

    ORACLE_DSN +++dbi:Oracle:host=localhost;port=1521;service_name=XEPDB1+++
    
    ORACLE_USER +++zava_erp+++

    ORACLE_PWD +++Zava1234+++

    ![](./media/image146.png)

    ![](./media/image147.png)

3.  In the same file update limit mirgation scope to application scope.

    SCHEMA +++ZAVA_ERP+++

    ![](./media/image148.png)

4.  Specify Object Types to Migrate.

    +++TABLE,SEQUENCE,INDEX,CONSTRAINT+++

    ![](./media/image149.png)

    To save and exit, press **Ctrl + X → Y → Enter**.

    ![](./media/image150.png)

## Task 10: Run Migration Assessment

In this task, you will generate a detailed compatibility report using
the Ora2Pg project configuration. This report provides object-level
insights and highlights areas requiring manual remediation.

1.  Generate Detailed Compatibility Report.

    +++cd ~/ora_migrate/zava_project+++

    +++./export_schema.sh -t SHOW_REPORT+++

    ![](./media/image151.png)

2.  Review Report Location.

    +++ls reports/+++

    ![](./media/image152.png)

    You can open each file and review:

    - Compatibility issues

    - Oracle-specific features

    - Estimated migration effort

## Task 11: Export Oracle Schema

In this task, you will export the Oracle schema objects as
PostgreSQL-compatible SQL scripts. These scripts will later be applied
to the Azure PostgreSQL target database.

1.  Generate Schema Scripts.

    +++./export_schema.sh+++

    ![](./media/image153.png)

2.  Validate Generated Schema Files.

    +++ls schema/+++

    ![](./media/image154.png)

## Task 12: Prepare Azure PostgreSQL Target Environment

In this task, you will configure the PostgreSQL target database, schema,
and extensions required for the migrated workload. This prepares the
destination environment for schema and data import.

1.  Navigate back to the Azure portal and **Resource
    group-\>ResourceGroup1-\>Azure Database for PostgreSQL Flexible
    Server**.

    ![](./media/image155.png)

2.  Save the endpoint in the Notepad for future use.

    ![](./media/image156.png)

3.  Open **Cloud shell**.

    ![](./media/image157.png)

4.  Click **Bash**.

    ![](./media/image158.png)

5.  Select **Mount storage account** and select @lab.CloudSubscription.Name.
    Click **Apply**.

    ![](./media/image159.png)

6.  Select **we will create storage for you**. Click **Next.**

    ![](./media/image160.png)

    ![](./media/image161.png)

7.  Connect to PostgreSQL Flexible Server.

    +++psql "host=postgrenewserver.postgres.database.azure.com port=5432 dbname=postgres user=pgAdmin password=pAssw0rd1289 sslmode=require"+++

    ![](./media/image162.png)

8.  Create Target Database.

    +++CREATE DATABASE ora_migrate;+++

    ![](./media/image163.png)

9.  Connect to the Database.

    +++\c ora_migrate+++

    ![](./media/image164.png)

10. Create Target Schema.

    +++CREATE SCHEMA zava_erp;+++

    +++ALTER SCHEMA zava_erp OWNER TO "pgAdmin";+++

    ![](./media/image165.png)

11. Enable Required Extensions.

    +++CREATE EXTENSION IF NOT EXISTS pg_trgm;+++

    +++CREATE EXTENSION IF NOT EXISTS btree_gin;+++

    ![](./media/image166.png)

12. Enter +++\q+++ to exit.

## Task 13: Import Schema into Azure PostgreSQL

In this task, you will apply the converted schema scripts to Azure
Database for PostgreSQL in the correct dependency order. This ensures
tables, constraints, indexes, and sequences are created successfully.

1.  Navigate back to Powershell.

2.  Run the following command to install PostgreSQL client on your
    Oracle VM.

    +++sudo dnf install -y postgresql postgresql-contrib+++

3.  Set PostgreSQL Environment Variables:

    +++export PGHOST=postgrenewserver.postgres.database.azure.com+++

    +++export PGPORT=5432+++

    +++export PGDATABASE=ora_migrate+++

    +++export PGUSER=pgAdmin+++

    +++export PGPASSWORD=pAssw0rd1289+++

    ![](./media/image167.png)

4.  Import files in correct order. Run these commands one by one:

    1. Import base tables first
    +++psql "host=$PGHOST port=$PGPORT dbname=$PGDATABASE user=$PGUSER password=$PGPASSWORD sslmode=require" -f ./schema/tables/table.sql+++
    
    2. Import constraints (foreign keys)
    +++psql "host=$PGHOST port=$PGPORT dbname=$PGDATABASE user=$PGUSER password=$PGPASSWORD sslmode=require" -f ./schema/tables/CONSTRAINTS_table.sql+++
    
    3. Import indexes
    +++psql "host=$PGHOST port=$PGPORT dbname=$PGDATABASE user=$PGUSER password=$PGPASSWORD sslmode=require" -f ./schema/tables/INDEXES_table.sql+++
    
    4. Import logical sequences if any
    +++psql "host=$PGHOST port=$PGPORT dbname=$PGDATABASE user=$PGUSER password=$PGPASSWORD sslmode=require" -f ./schema/tables/LOGICAL_table.sql+++
    
    ![](./media/image168.png)

## Task 14: Migrate Data from Oracle to Azure PostgreSQL

In this task, you will extract data from Oracle in PostgreSQL COPY
format using Ora2Pg. You will then load this data into the PostgreSQL
target database.

1.  Run the following commands to grant required Oracle permissions for
    Ora2Pg:

    +++sqlplus sys/pAssw0rd1289 as sysdba+++

    +++ALTER SESSION SET CONTAINER=XEPDB1;+++

    +++GRANT SELECT ON v_$database TO ZAVA_ERP;+++

    ![](./media/image169.png)

2.  Run the following command to extract Oracle data in PostgreSQL COPY
    format:

    +++ora2pg -t COPY -o data.sql -b ./data -c ./config/ora2pg.conf+++

    ![](./media/image170.png)

3.  After successful execution, the data file will be created
    at:./data/data.sql. So to confirm the data file was created run the
    following command:

    +++ls -l ./data+++

    ![](./media/image171.png)

4.  Use psql to load the data into PostgreSQL:

    +++psql "host=$PGHOST port=$PGPORT dbname=$PGDATABASE user=$PGUSER password=$PGPASSWORD sslmode=require" -f ./data/data.sql+++

    ![](./media/image172.png)

## Task 15: Validate Data Migration

In this task, you will validate the migration by checking row counts,
sample records, and table accessibility in PostgreSQL. Successful
validation confirms that the ERP Order Management data migrated
correctly.

1.  Connect to PostgreSQL:  
    +++psql "host=$PGHOST port=$PGPORT dbname=$PGDATABASE user=$PGUSER password=$PGPASSWORD sslmode=require"+++

    ![](./media/image173.png)

2.  Run validation queries:

    List tables
    
    +++SET search_path TO public;+++
    
    Check row counts 
    +++SELECT COUNT(\*) FROM customers;+++
    +++SELECT COUNT(\*) FROM orders;+++
    
    Sample data check
    +++SELECT * FROM customers LIMIT 10;+++
    +++SELECT * FROM orders LIMIT 10;+++

    ![](./media/image174.png)

    ![](./media/image175.png)

## Summary

This lab demonstrates a heterogeneous migration of ZAVA DIY's Oracle
ERP Order Management module to Azure Database for PostgreSQL -
Flexible Server. You provision the target PostgreSQL environment and
set up an Oracle source system on an Azure virtual machine. Using
Ora2Pg, you assess schema compatibility, convert Oracle objects, and
generate PostgreSQL-compatible scripts. The lab guides you through
migrating schema and data while preserving relationships and
constraints. Finally, you validate the migration to ensure data
consistency and continuity of critical e-commerce workflows.
