# Lab 02: End-to-End Migration of ZAVA’s On-Prem PostgreSQL to Azure Database for PostgreSQL – Flexible Server (Homogeneous Migration)

## Scenario

ZAVA Express Logistics, a branch of ZAVA DIY, manages regional freight
shipping, fleet operations, warehouse inventory, and last-mile delivery.
Traditionally, the branch relied on on-premises systems and manual
processes to coordinate deliveries, track vehicles, and manage stock,
which led to frequent data silos, operational inefficiencies, and
delayed insights — especially during peak periods. To address these
challenges, ZAVA’s leadership launched a cloud modernization initiative
aimed at centralizing operations, streamlining workflows, and enabling
real-time decision-making across the branch. Under the guidance of
Carlos Vega, CTO, the team conducted a thorough assessment of the
branch’s on-premises environment, identifying over 50 operational
systems and selecting one representative workload for a pilot migration.
This workload, a web-based branch management system running on Red Hat
Enterprise Linux (RHEL) servers and connected to a PostgreSQL database,
was chosen because it exemplified the common components found across
other branch systems while remaining manageable for testing the
migration plan.

The branch migration project leverages Azure Database for PostgreSQL –
Flexible Server, chosen for its scalable performance, built-in security,
high availability, and AI-readiness through pgvector and Azure AI
extensions — capabilities that will support predictive route
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

### Task 1 – Create resources

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

3.  Select **Deploy to Azure**. This will open a new browser tab to the
    Azure Portal for custom deployments.
    
    ![The GitHub page with Deploy to
    Azure button highlighted.](./media/image4.png)

4.  Fill in the required ARM template parameters.

    - **Subscription:** Use the default one

    - **Resource group:** Select **ResourceGroup1**

    - **Region:** Select @lab.CloudResourceGroup(ResourceGroup1).Location

    - **Resource Name Base:** Enter +++ZavaWeb@lab.LabInstance.Id+++

    - **Password:** Enter +++pass@dmIn234+++

    Select **Review + create.**

    ![](./media/image5.png)

5.  Click on the **Create** button to start deployment.

    ![](./media/image6.png)

6.  The deployment is now underway. On average, this process can take
    10-20 minutes to complete.

    ![](./media/image7.png)

    **Note**: While automation can make things simpler and repeatable,
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

1.  Select the **On-premises Workload VM** named similar
    to **ZavaWeb-onprem-workload-vm** present on page 2.

    ![](./media/image9.png)

2.  In the **Overview** page, go to **Properties**, and under the
    **Networking** section, locate the **Private IP Address** of the VM
    and copy it into Notepad.

    ![](./media/image10.png)

    **Note:** You will need this IP address to configure the web application
    to use the database workload server.

3.  Navigate back to the **ResourceGroup1**, then select
    the **On-premises APP VM** named similar
    to **ZavaWeb-onprem-app-vm**.

    ![](./media/image11.png)

4.  Search for +++**Bastion**+++ in the left-hand menu and then select
    **Bastion**. We will use a bastion host as the method to connect to
    our VMs, as this is a more secure method.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image12.png)

5.  Within the **Bastion** page, enter the following details:

    - **Authentication Type:** VM Password

    - **Username:** Enter +++demouser+++

    - **VM Password:** Enter +++pass@dmIn234+++

    Then click on the **Connect** button to connect with Bastion.

    ![](./media/image13.png)

    **Note**: You may need to **allow pop-ups** if they are blocked in your browser.

6.  When connected to the VM via the Bastion host, you will get a screen
    like this:

    **Note**: If you see a pop-up stating “See test and images copied to the
    clipboard”. Click **Allow**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image14.png)

7.  Once connected via Bastion, run the following command to install the
    git utility on the server by using the clipboard within the
    session:   

    ![A screen shot of a computer AI-generated content may be
    incorrect.](./media/image15.png) 

8.  Click the arrows, which will expand the window

    ![Clipboard within the Bastion session.](./media/image16.png)

9.  Copy the command below and paste it into the clipboard. 

    +++sudo yum install git  -y+++

    ![Clipboard within the Bastion session.](./media/image17.png) 

10. Now, right-click on the window, which will paste the command and run
    it. 

    ![](./media/image18.png)  

    **Note:** Similarly, you can run all these commands in the Bastion.

11. Enter the following command to clone the remote git repository
    holding a script which will configure the web app on the application
    server.

    +++sudo git clone
    https://github.com/technofocus-pte/TechExcel-Migrate-Linux-workloads.git+++

    ![](./media/image19.png)

12. You can now run the configuration script by using the following
    command:

    +++sudo bash
    TechExcel-Migrate-Linux-workloads/resources/deployment/onprem/APP-workload-install.sh+++

    You will get a status message of  “The script was successful”.

    ![](./media/image20.png)

13. Execute the following command to open the **orders.php** file for
    the web application in a text editor. The application needs to be
    configured to connect to the **Azure Database for PostgreSQL
    Flexible Server** database.

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
    IP Address** of the **ZavaWeb-onprem-workload-vm** instance that was
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

1.  Navigate back to **Azure Portal**, open the **ResourceGroup1**, then
    select the **On-premises APP VM** named similar
    to **ZavaWeb-onprem-app-vm**

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image24.png)

2.  In the **Overview** window, locate the VM’s **Public IP Address**
    and copy it into Notepad.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image25.png)

3.  Open a new browser window, then navigate to the
    following **http:// URL** to access the simulated on-premises web
    application provisioned for this lab. Be sure to replace
    the **\<ip-address\>** placeholder with the Public IP Address for
    the VM. For example: http://172.206.126.43

    +++http://\<ip-address\>+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image26.png)

    **Note:** You should get the Red Hat Enterprise Linux Test Page 

4.  When the web page loads, enter the following at the end of the URL.
    For example: "http://172.206.126.43/orders.php"

    +++http://\<ip-address\>/orders.php+++

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

2.  In the **Marketplace** window, search for +++**PostgreSQL
    flexible**+++, then select **Azure Database for PostgreSQL Flexible
    Server** from the search results.

    ![A screenshot of a computer AI-generated content may be incorrect.](./media/image29.png)

3.  Click **Create** and then select **Azure Database for PostgreSQL
    Flexible Server.**

    ![A screenshot of a computer AI-generated content may be incorrect.](./media/image30.png)

4.  On the **New Azure Database for PostgreSQL** **Flexible
    Server** pane, select the following values:

    1.  **Subscription:** Keep default

    2.  **Resource group:** Select **ResourceGroup1**

    3.  **Server name:** Enter +++**zavaweb-db@lab.LabInstance.Id**+++

    4.  **Region:** Select
        @lab.CloudResourceGroup(ResourceGroup1).Location 

    5.  **PostgreSQL version:** Keep the default, as it always selects
        the latest version

    6.  **Workload type:** Select **Dev/Test**

    ![](./media/image31.png)

5.  Under **Compute + storage,** click **Configure server**.

6.  On the Compute + storage window, under the **compute** section,
    choose the following:

    1.  **Compute tier:** General Purpose (2-64 vCores) - Balanced
        configuration for most common workloads\*\*

    2.  **Compute Size:** Standard_D4ds_v5 (4 vCores, 16GiB memory, 6400
        max iops)

    ![](./media/image32.png)

7.  Under the **High availability** section, choose **Disabled (99.9%
    SLA)** and then click **Save.**

    ![](./media/image33.png)

8.  On the **New Azure Database for PostgreSQL** **Flexible Server**
    window, under the **Authentication** section, enter the following
    details:

    1.  **Authentication method**: Choose **PostgreSQL authentication
        only**

    2.  **Admin username**: Enter +++pgadmin+++

    3.   **Password**: Enter +++passaDmin342+++

    Select **Next: Networking**

    ![](./media/image34.png)

9.  In the **Networking** tab, under **Public access**, clear the
    checkbox to disable public access.

    ![](./media/image35.png)

10. Under **Private endpoint** section, select **Create private
    endpoint.**

    ![](./media/image36.png)

11. On the **Create private endpoint** window, enter the following
    details:

    1.  **Subscription:** Keep default

    2.  **Resource group:** ResourceGroup1

    3.  **Location**: Select
        @lab.CloudResourceGroup(ResourceGroup1).Location 

    4.  **Name:** Enter +++post-priv-endpoint+++

    5.  **Virtual network:** Select
        **ZavaWeb.Id-spoke-vnet(ResourceGroup1)**

    6.  **Subnet**: Select **default**(10.2.0.0/24)

    7.  **Integrate with privet DNS zone:** Select No (You will use an
        IP address rather than a DNS entry when connecting.)

    8.  Click **OK**

    ![](./media/image37.png)

12. Select **Review + create**.

    ![](./media/image38.png)

13. Select **Create** to provision the service.

    ![](./media/image39.png)

14. Click **Create server without firewall rules** - as you will use the
    private endpoint for access.

    ![](./media/image40.png)

15. Wait for deployment to complete; it will take 5-10 mins. Once
    provisioning has finished, click on **Go to resource**.

    ![](./media/image41.png)

    ![](./media/image42.png)

16. In the Overview tab of the **Azure Database for PostgreSQL flexible
    server,**  copy and save the **Server name** in Notepad for use
    later.

    ![](./media/image43.png)

### Task 02 - Migrate your database to Azure Database for PostgreSQL flexible server

In this task, you will set up a migration project and configure the
Source and Target connections. You will then execute and monitor a
migration of your on-premises PostgreSQL database into Azure Database
for PostgreSQL - flexible server.

1.  Select **Migration** from the menu on the left of the flexible
    server blade.

    ![](./media/image44.png)

2.  Click on the **+ Create**.

    ![](./media/image45.png)

    **Note**: If the **+ Create** option is unavailable,
    select **Compute + storage** and change the compute tier to
    either **General Purpose** or **Memory Optimized** and try to create
    the Migration process again. After the Migration is successful, you
    can change the compute tier back to **Burstable**.

3.  On the **Setup** tab, enter each field as follows:

    1.  **Migration name:** +++Migration-Zava-northwind+++

    2.  **Source server type:** On-premise Server.

    3.  **Migration option:** Validate and Migrate.

    4.  **Migration option:** Offline

    5.  Select **Next: Runtime server \>**

    ![](./media/image46.png)

    **Note:** The Runtime server \> button might be enabled after 20-30 mins.

4.  We will **not** use a Runtime Server, so just select **Next: Source
    server \>**.

    ![](./media/image47.png)

5.  On the **Source server** tab, enter each field as follows:

    1.  **Server name:** The public IP address of the
        “ZavaWeb-onprem-workload-vm”.

    2.  **Port:** 5432

    3.  **Server admin login name:** rootuser(the VM has been setup with
        an admin user called rootuser)

    4.  **Password:** Enter+++123rootpass456+++

    5.  **SSL mode:** Prefer.

    6.  Click on the **Connect to source** option to validate the
        connectivity details provided.

    7.  Click on the **Next: Target server\>** button to progress.

    ![](./media/image48.png)

6.  The connectivity details should be automatically completed for the
    target server we are migrating to.

    1.  In the password field -  Enter +++passaDmin342+++

    2.  Click on the **Connect to target** option to validate the
        connectivity details provided.

    3.  Click on the **Next: Databases to validate or migrate\>** button to progress.

    ![](./media/image49.png)

7.  On the **Databases to validate or migrate** tab, select the
    **northwind** database because you want to migrate to the flexible
    server. Then click on the **Next : Summary \>** button to progress
    and review the data provided.

    ![](./media/image50.png)

8.  On the **Summary** tab, review the information and then click
    the **Start Validation and Migration** button to start the migration
    to the flexible server.

    ![](./media/image51.png)

9.  On the **Migration** tab, you can monitor the migration progress by
    using the **Refresh** button in the top menu to view the progress
    through the validation and migration process.

    ![](./media/image52.png)

10. By clicking on the **Migration-northwind** activity, you can view
    detailed information about the migration activity’s progress.

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

8.  Within the **Bastion** page, enter the following details:

    - **Authentication Type:** VM Password

    - **Username:** Enter +++demouser+++

    - **VM Password:** Enter +++pass@dmIn234+++

    Click **Connect**.

    ![](./media/image61.png)

9.  Run the following command to connect to Azure PostgreSQL in the VM.

    +++psql -h 10.2.0.4 -U pgadmin -d northwind -p 5432+++

    When prompted, enter:

    +++ passaDmin342+++

    ![](./media/image62.png)

10. Run the following command to list all tables in the database:

    +++dt+++

    ![](./media/image63.png)

11. Run the following SQL queries to verify record counts in key tables:

    +++ SELECT COUNT(\*) FROM orders;+++

    +++SELECT COUNT(\*) FROM customers;+++

    ![](./media/image64.png)

12. Retrieve a small sample of data from the database to verify
    accuracy:

    +++ SELECT \* FROM orders LIMIT 5;++++

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

# Lab 3: Oracle to Azure PostgreSQL Migration Using Microsoft First‑Party AI‑Assisted Migration Tool

# Objective

To migrate Oracle database schemas and client application code to Azure
Database for PostgreSQL using Visual Studio Code migration tooling and
GitHub Copilot (Claude Sonnet), ensuring PostgreSQL compatibility and
generating validated migration artifacts.

# Exercise 1 - Schema migration from Oracle to Azure PostgreSQL

This lab guides you through converting Oracle database schemas to Azure
Database for PostgreSQL using the Visual Studio PostgreSQL extension
with Azure OpenAI to automate and validate schema translation. It covers
connecting to your Oracle source and Azure Database for PostgreSQL
target, configuring Azure OpenAI, running the Migration Wizard, and
reviewing the generated PostgreSQL artifacts.  
Before you begin, ensure you have network access, credentials for both
servers, and a deployed Azure OpenAI model.

During the conversion, you will experience the following steps:

- **Schema Discovery:** The tool analyzes your Oracle schema objects.

- **AI Processing:** Azure OpenAI processes and converts compatible
  objects.

- **Validation:** Converted objects are validated in the scratch
  database.

- **Review Tasks:** Objects requiring manual attention are flagged for
  your review.

- **Output Generation:** Successfully converted objects are saved as
  PostgreSQL files.

## Task 1: Provision PostgreSQL Flexible Server

1.  Open a web browser and sign in to the **Azure Portal** at
    +++**https://portal.azure.com+++** using your cloud slice
    credentials

    - Username - +++@lab.CloudPortalCredential(User1).Username+++

    - Password - +++@lab.CloudPortalCredential(User1).Password+++

    ![A screenshot of a login box AI-generated content may be
    incorrect.](./media/image66.png)

    ![A screenshot of a login box
    AI-generated content may be incorrect.](./media/image67.png)

    ![A
    screenshot of a computer screen AI-generated content may be
    incorrect.](./media/image68.png)

2.  In the Azure Portal search bar, type +++**PostgreSQL Flexible
    Server+++**, then select **Azure Database for PostgreSQL flexible
    servers**.

    ![](./media/image69.png)

3.  Click **+Create.**

    ![](./media/image70.png)

4.  Now Enter the following details:

    - **Basics**

      - **Resource group:** ResourceGroup1

      - **Server name:** +++**postgrenewserver1234+++**

      - **Region:** *Central US*

      - **PostgreSQL version:** *16*

      - **Workload type:** *Dev/Test*

    - **Authentication**

        - Select **PostgreSQL authentication only**

        - **Username:** **+++pgAdmin+++**

        - **Password:** **+++pAssw0rd1289+++**

        - After entering the information, select **Networking**.

    ![](./media/image71.png)

    ![](./media/image72.png)

5.  In In the **Networking** tab, ensure:

    - **Connectivity method:** Public access

    - **Allow public access to this resource through the internet using
      a public IP address** is **enabled**.

    ![](./media/image73.png)

6.  Under the **Firewall** section:

    - Enable **Allow public access from any Azure service within Azure to
    this server**.

    - Click **+ Add current client IP address** to automatically create a
    firewall rule for your IP.

    - Click **Review + create**.

    ![](./media/image74.png)

7.  Click **Create**.

    ![](./media/image75.png)

8.  WaiWait **5–10 minutes** for the deployment to complete.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image76.png)

9.  Click **Go to resource**.

    ![](./media/image77.png)

10. Make a note of the **Endpoint** value—you will need this as the
    **Host** in **Task 8**

    ![](./media/image78.png)

11. In the left navigation pane, go to **Settings → Server
    parameters**.  
    In the search bar, type +++**azure.extension+++**.From the dropdown
    menu, enable the **BTREE_GIN** and **PG_TRGM** extensions.Click
    **Save**.

    ![](./media/image79.png)

    ![](./media/image80.png)

12. After completing the deployment successfully, return to the **Home**
    tab.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image81.png)

    ![](./media/image82.png)

## Task 2: Create Microsoft Foundry Resource and Deploy the GPT‑4.1 Model

1.  Switch back to the **Azure Portal**, search for +++**Microsoft
    Foundry+++**, and select it

    ![](./media/image83.png)

2.  Under **Create a Foundry Resource**, click **Create a resource**.

    ![](./media/image84.png)

3.  Enter Enter the following values, then click **Review + create**:

    - **Resource Group:** Select the existing resource group

    - **Name:** +++**ogtopgmsfoundry+++**

    - **Region:** *East US*

    ![](./media/image85.png)

4.  After validation completes successfully, click **Create**.

    ![](./media/image86.png)

5.  Once deployment is complete, click **Go to resource**

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image87.png)

    ![](./media/image88.png)

6.  In the left navigation pane, expand **Resource Management** and
    select **Keys and Endpoint**.Copy the **API endpoint** and **Key 1**
    values—these will be required later in **Task 8**.

    ![](./media/image89.png)

7.  Click **Go to Foundry portal**.

    ![](./media/image90.png)

8.  In the Foundry portal, select **Models + endpoints**, then open the
    **Model deployments** tab.Click **Deploy model** and choose **Deploy
    base model**.

    ![](./media/image91.png)

9.  Search for +++**gpt‑4.1+++**, select the model, and click
    **Confirm**.

    ![](./media/image92.png)

10. For the deployment:

    - Set **Deployment type** to **Standard**

    - Increase **Tokens per Minute Rate Limit** to the maximum

    - Click **Deploy**

    ![](./media/image93.png)

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image94.png)

## Task 3: Create a virtual machine

1.  Navigate to the **Home** page of the Azure Portal. Search for
    +++**Virtual machines+++** and select it.

    ![](./media/image95.png)

2.  Click **+Create**. Select **Virtual Machine**.

    ![](./media/image96.png)

3.  In In the *Create a virtual machine* window, enter the following
    details:

    - **Subscription:** Keep the default

    - **Resource group:** **ResourceGroup1**

    - **Virtual machine name:** +++**oracleVirtual123+++**

    - **Region:** *Central US*

    - **Security type:** *Trusted launch virtual machines*

    - **Image:** *Oracle Linux 8.10 (LVM) – x64 Gen2*

    - Keep all other settings as default.Click **Review + create**.

    ![](./media/image97.png)

    ![](./media/image98.png)

4.  Click **Create**.

    ![](./media/image99.png)

5.  When prompted,click **Download private key and create a resource**.

    ![](./media/image100.png)

6.  Deployment will begin. Wait **up to 5 minutes** until it completes,
    then click **Go to resource**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image101.png)

    ![](./media/image102.png)

7.  Note the **public IP address** of the VM. Save it—you will use this
    as your **Oracle host** in later tasks

    ![](./media/image103.png)

8.  In the left navigation menu, expand **Networking → Network
    settings**.  
    Click **Create port rule** and select **Inbound port rule**.

    ![](./media/image104.png)

9.  Add a new inbound port rule with the following values:

    - **Source:** Any

    - **Source port ranges:** \*

    - **Destination:** Any

    - **Destination port:** +++1521+++

    - **Protocol:** TCP

    - **Action:** Allow

    - **Priority:** Default

    - **Name:** +++**allow-oracle-1521+++**

    - Click **Add**.

    ![](./media/image105.png)

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image106.png)

10. For **SSH Setup**, Open **PowerShell** as Administrator.Check if the
    .ssh folder exists:

    ![](./media/image107.png)

11. Run the following command to check if the .ssh folder exists in your
    system. If you receive an error, the folder does not exist

    +++**dir $HOME\.ssh**+++

    ![A computer screen shot of a blue screen AI-generated content may be
    incorrect.](./media/image108.png)

12. Run the following command to create the .ssh folder in the home
    directory.

    +++mkdir $HOME\.ssh+++

    **Note**: The directory path where the folder is created.  
    (*This may differ depending on the VM/user.*)

    ![](./media/image109.png)

13. Locate your downloaded .pem file in File Explorer and **copy it**

    ![](./media/image110.png)

14. Paste the .pem file into the .ssh directory.  
    Example path:

    **C:\Users\Student\.ssh\oraclelinux-key.pem**

    ![](./media/image111.png)

15. Return to PowerShell and check the current user:

    +++**whoami**+++

    Save the output of this command in notepad for future use.

    ![](./media/image112.png)

16. Set the permission for .ssh file.

    +++**icacls "$HOME\.ssh\oracleVirtual_key.pem"/inheritance:r**+++

    +++icacls $HOME\.ssh\oracleVirtual_key.pem /grant "\<CURRENT
    USER\>:R"+++

    **Note:** Get the current user value from the previous step.

    For example:

    +++icacls "$HOME\.ssh\oracleVirtual_key.pem" /inheritance:r+++

    +++icacls $HOME\.ssh\oracleVirtual_key.pem /grant
    "sea-dev\student:R"+++

    ![](./media/image113.png)

17. Run the following command to connect to your VM:

    +++ssh -i "$HOME\.ssh\oracleVirtual_key.pem"
    azureuser@\<VM_PUBLIC_IP\>+++

    For Example:

    +++ssh -i "$HOME\.ssh\oracleVirtual_key.pem" azureuser@20.83.43.16+++

    When prompted with *“Are you sure you want to continue connecting
    (yes/no)?”*, type +++**yes**+++

    ![](./media/image114.png)

18. Exit from the VM using the following command:

    +++exit+++

    ![](./media/image115.png)

## Task 4: Set Up the Oracle Environment

1.  Open a web browser and navigate to +++
    **https://www.oracle.com/database/technologies/xe-downloads.html**+++
    and download : Oracle Database 21c Express edition for Linus
    x64(OL8)+++ and download **Oracle Database 21c Express edition for
    Linus x64(OL8)**

    ![](./media/image116.png)

2.  Navigate to the PowerShell and upload this .rmp file to the VM using
    the following command:

    +++ scp -i "$HOME\.ssh\oracleVirtual_key.pem"
    "$HOME\Downloads\oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm"
    azureuser@\<Your-VM-IP\>:~/+++

    For Example:

    +++ scp -i "$HOME\.ssh\oracleVirtual_key.pem"
    "$HOME\Downloads\oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm"
    azureuser@20.83.43.16:~/+++

    Wait for the file upload to complete.

    ![](./media/image117.png)

3.  Connect to the VM using the following command

    +++ssh -i "$HOME\.ssh\oracleVirtual_key.pem"<azureuser@20.83.43.16>+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image118.png)

4.  Run the following command to install pre-requisites for Oracle XE
    21c.

    +++ sudo dnf install -y oracle-database-preinstall-21c+++

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image119.png)

    After completing this you will get successfully message.

    ![A computer screen shot of a blue screen AI-generated content may be
    incorrect.](./media/image120.png)

5.  Now run the following command to install Oracle Database 21c Express
    Edition in the VM.

    +++sudo dnf localinstall -y oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm+++

    ![](./media/image121.png)

6.  Configure the Oracle database:

    +++sudo /etc/init.d/oracle-xe-21c configure+++

    It will ask for a password. You can enter this +++**pAssw0rd1289**+++

    ![](./media/image122.png)

    Save the pulugged database name that is **XEPDB1**.

    **Note:** It will take 10-15 mins to complete.

7.  Verify the Oracle database status.

    +++sudo /etc/init.d/oracle-xe-21c status+++

    If the database is running, Oracle Database 21c Express Edition has
    been installed and configured successfully

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image123.png)

8.  Confirm the location of the SQL*Plus executable. (Oracle XE includes
    SQLPlus and the Instant Client.)*

    +++**sudo find / -name sqlplus 2>/dev/null**+++

    ![A blue screen with red text AI-generated content may be
    incorrect.](./media/image124.png)

9.  Add SQL\*Plus to PATH permanently.

    +++echo 'export ORACLE_HOME=/opt/oracle/product/21c/dbhomeXE' >>~/.bashrc+++

    +++echo 'export ORACLE_SID=XE' >> ~/.bashrc+++

    +++echo 'export PATH=$ORACLE_HOME/bin:$PATH' >> ~/.bashrc+++

    +++source ~/.bashrc+++

    ![A blue screen with white text AI-generated content may be
    incorrect.](./media/image125.png)

10. Verify that SQL\*Plus is installed correctly

    +++sqlplus -v+++

    ![](./media/image126.png)

## Task 5: Create a Database in Oracle.

1.  Connect to the Pluggable Database (PDB) as **SYSDBA**.This logs you
    in with full administrative privileges required for configuration
    and migration tasks.

    +++sqlplus 'sys/pAssw0rd1289' as sysdba+++

    ![](./media/image127.png)

2.  Switch the session to the Pluggable Database.This command changes
    your session from the CDB to the target PDB so you can run
    operations inside the PDB.

    +++ALTER SESSION SET CONTAINER=XEPDB1;+++

    ![](./media/image128.png)

3.  Create the **ERP user** and grant the required privileges. It
    creates a new application user inside the PDB and assigns the
    permissions needed to create and manage database objects.

    ```
    -- Create a new user

    CREATE USER og_user

    IDENTIFIED BY ogtest1234

    DEFAULT TABLESPACE USERS

    TEMPORARY TABLESPACE TEMP;

    GRANT CONNECT, RESOURCE, CREATE TABLE, CREATE VIEW, CREATE SEQUENCE,
    CREATE PROCEDURE TO og_user;

    ALTER USER og_user QUOTA UNLIMITED ON USERS;

    ALTER PLUGGABLE DATABASE XEPDB1 SAVE STATE;

    ALTER SESSION SET CONTAINER = XEPDB1;

    GRANT SELECT ANY TABLE TO og_user;

    GRANT SELECT ON dba_users TO og_user;
    ```

    ![A screenshot of a computer program AI-generated content may be
    incorrect.](./media/image129.png)

4.  Run +++exit+++ to exit SQL\*Plus

    ![](./media/image130.png)

    ![](./media/image131.png)

## Task 6 – Create Multiple Schemas

1.  Run below commands to create Application Schemas (Users)

    ```

    -- Login

    sqlplus 'sys/pAssw0rd1289' as sysdba

    ALTER SESSION SET CONTAINER =XEPDB1;

    -- Schema 1

    CREATE USER flight_core IDENTIFIED BY Flight123;

    GRANT CONNECT, RESOURCE, CREATE VIEW, CREATE PROCEDURE TO flight_core;

    ALTER USER flight_core QUOTA UNLIMITED ON USERS;

    -- Schema 2

    CREATE USER booking_mgmt IDENTIFIED BY Book123;

    GRANT CONNECT, RESOURCE, CREATE VIEW, CREATE PROCEDURE TO booking_mgmt;

    ALTER USER booking_mgmt QUOTA UNLIMITED ON USERS;

    You now have **2 schemas**:

    - flight_core (master data)

    - booking_mgmt (transactions)
    ```

    ![](./media/image132.png)

## Task 7 –Create Tables, Views, SPs

Object-to-Schema Mapping
| Object  | Schema(User) | 
|----------|----------|
| Flights table   | flight_core   | 
| Bookings table  | booking_mgmt   | 
| Views  | booking_mgmt   | 
| Stored Procedures  |    | 
| Admin queries   | zava_erp    | 

1. Run below queries to create table
    ```

    -- flight_core schema

    CREATE TABLE flight_core.flights (

    flight_id NUMBER PRIMARY KEY,

    origin VARCHAR2(50),

    destination VARCHAR2(50),

    base_price NUMBER

    );

    CREATE TABLE booking_mgmt.bookings (

    booking_id NUMBER PRIMARY KEY,

    flight_id NUMBER,

    seats NUMBER,

    total_price NUMBER

    );
    ```

    ![A screenshot of a computer program AI-generated content may be
    incorrect.](./media/image133.png)

2.  Run below query to create view
    ```

    GRANT SELECT ON flight_core.flights TO booking_mgmt;

    CREATE VIEW booking_mgmt.vw_flight_prices AS

    SELECT f.flight_id, f.origin, f.destination, f.base_price

    FROM flight_core.flights f;
    ```

    ![A blue screen with white text AI-generated content may be
    incorrect.](./media/image134.png)

3.  Run below query to create stored procedure
    ```
    CREATE OR REPLACE PROCEDURE booking_mgmt.calculate_price ( 

      p_flight_id IN NUMBER, 

      p_seats     IN NUMBER, 

      p_total     OUT NUMBER 

    ) AS 

      v_price NUMBER; 

    BEGIN 

      SELECT f.base_price 

        INTO v_price 

        FROM flight_core.flights f 

       WHERE f.flight_id = p_flight_id; 

    

      p_total := v_price * p_seats; 

    END; 

    / 

    SHOW ERRORS PROCEDURE booking_mgmt.calculate_price; 
    ```
    ![A computer screen shot of white text AI-generated content may be
    incorrect.](./media/image135.png)

    ![A screenshot of a computer program AI-generated content may be
    incorrect.](./media/image136.png)

2. Run below commands to grant access to ERP user
    ```

    -- allows querying DBA_* and other catalog views safely

    GRANT SELECT_CATALOG_ROLE TO og_user;

    -- some tools also require broader dictionary access

    GRANT SELECT ANY DICTIONARY TO og_user;

    -- required for extracting DDL via DBMS_METADATA

    GRANT EXECUTE ON DBMS_METADATA TO og_user;

    -- (optional but often useful for dependency resolution)

    GRANT CREATE ANY VIEW TO og_user;

    GRANT CREATE ANY PROCEDURE TO og_user;

    GRANT CREATE ANY SEQUENCE TO og_user;
    ```

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image137.png)

4.  Run exit command to come from SQL window

    +++exit+++

5.  Run these commands on the VM to ensure port 1521.
    ```

    sudo firewall-cmd --add-port=1521/tcp --permanent

    sudo firewall-cmd --reload

    sudo firewall-cmd --list-ports

    lsnrctl stop

    lsnrctl start
    ```

    ![A computer screen shot of a blue screen AI-generated content may be
    incorrect.](./media/image138.png)

6.  Open new Powershell and run to test VM connection status

    +++Test-NetConnection \<YOUR_VM_IP\> -Port 1521+++

    ![A computer screen with white text AI-generated content may be
    incorrect.](./media/image139.png)

## Task 8 – Oracle to Azure Database for PostgreSQL Schema Conversion

1.  Open **Visual Studio Code** and go to the **Extensions** view
    (Ctrl + Shift + X).

2.  Search for +++***PostgreSQL+++*** and **install**
    the **PostgreSQL** extension.

    ![](./media/image140.png)

3.  In the PostgreSQL extension panel, create a connection to your
    **Azure Database for PostgreSQL** by clicking **Add Connection**.

    ![](./media/image141.png)

4.  Enter the required connection details from the PostgreSQL server you
    created in **Task 1**, then click **Test Connection**:

    - Host : **postgrenewserverXXXX.postgres.database.azure.com** (Replace
    XXXX with your server number)

    - User Name : **+++pgAdmin+++**

    - Password : +++**pAssw0rd1289+++**

    - DATABASE NAME : +++**postgres+++**

    ![](./media/image142.png)

5.  Click on **Save & Connect.**

    ![](./media/image143.png)

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image144.png)

6.  Create a new folder on your local machine to store your migration
    project.

7.  In **Visual Studio Code, go to File → Open Folder**

    ![](./media/image145.png)

8.  Select the folder you created and open it.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image146.png)

9.  In the PostgreSQL extension pane, click **PostgreSQL server** to
    connect, then select **Create Migration Project**

    ![](./media/image147.png)

10. In the **Migration Wizard**, enter your **project name** as
    +++**ogtopgaimigration+++** and then click on **Next :** **Oracle
    Connection**

    ![](./media/image148.png)

11. Enter your **Oracle connection details** :

    1.  Host or server name : \<Your VM’s IP Address\>

    2.  Port number : **1521**

    3.  Oracle SID or Server Name : +++**XEPDB1+++**

    4.  Username : +++**og_user+++**

    5.  Password : +++**ogtest1234+++**

    6.  The tool will automatically load schemas and connect to the
        Oracle database.

    ![](./media/image149.png)

12. Expand the **Schemas** dropdown, select **FLIGHT_CORE** and
    **BOOKING_MGMT**, then click **Next: PostgreSQL Connection**

    ![](./media/image150.png)

13. Click **Load Databases** to load your PostgreSQL server databases.  
    After loading, click **Next: Language Model Configuration**.

    ![](./media/image151.png)

14. Enter the following details for Microsoft Foundry:

    - **OpenAI Endpoint:** *Your Foundry API endpoint*

    - **OpenAI API Key:** *Your Foundry Key 1 value*

    Then click **Test OpenAI Connection**

    ![](./media/image152.png)

15. Click **Create Migration Project**

    ![](./media/image153.png)

16. Under the **Schema Migration** tile, click the **Migrate** button.

    ![](./media/image154.png)

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image155.png)

17. Monitor the schema conversion progress inside Visual Studio Code.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image156.png)

18. Once the conversion completes, a **Schema Conversion Report** is
    generated.  
    Review objects that were successfully converted and those that were
    skipped.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image157.png)

19. The report displays the **success percentage** of the conversion.
    The report will display the **overall conversion success
    percentage**.

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image158.png)

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image159.png)

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image160.png)

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image161.png)

    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image162.png)

# Exercise 2 : Oracle to Azure Database for PostgreSQL Application Conversion (Preview)

This exercise guides you through converting Oracle client application
code to Azure Database for PostgreSQL using the **Application
Conversion** feature in the Oracle to Azure Database for PostgreSQL
migration tooling available in the **Visual Studio Code PostgreSQL
extension**.

You will learn how to:

- Import your source application code into the migration workspace

- Start the automated application code conversion process

- View the generated migration report

- Review and compare converted files using the built‑in diff tools

While it is not mandatory to perform a schema conversion beforehand, we
**strongly recommend** completing a schema migration first.  
If your Oracle schema has already been converted to PostgreSQL, the
application conversion process produces **more accurate** and
**higher‑quality** transformation results.

## **Task 1:** Build your application 

1.  Open **Git Bash** and navigate to the application directory

    +++cd "Users/Student/containerize-and-deploy-Java-app-to-Azure/"+++

    ![](./media/image163.jpeg)

2.  Build the Airlines application by running the Maven command from the
    project folder

    +++cd "Project/Airlines"+++

    +++mvn clean package+++

    ![](./media/image164.jpeg)

    ![](./media/image165.jpeg)
    ![](./media/image166.jpeg)
    ![](./media/image167.jpeg)
    ![](./media/image168.jpeg)
    ![](./media/image169.jpeg)

## Task 2 : Set up Copilot environment

1.  In Visual Studio Code, click the **Copilot** icon in your project

    ![](./media/image170.png)

2.  Select **Continue with GitHub**

    ![](./media/image171.png)

3.  Sign in with your GitHub account that has a **Copilot Premium**
    license assigned

    ![](./media/image172.png)

4.  Switch back to Visual Studio Code. Open the **GitHub Copilot Chat**
    interface and select **Claude Sonnet 4** (or a higher supported
    model).

    ![](./media/image173.jpeg)

    ![](./media/image174.jpeg)

5.  In the Explorer panel, select the project folder and expand the
    **.github/** directory.

    ![](./media/image175.jpeg)

6.  Refresh **application_code** folder

    ![](./media/image176.jpeg)

## Task 3: Start Application migration

1.  Locate the **application_code** folder in your project
    under .github/postgres-migration/project_name/application_code.

    ![](./media/image177.png)

2.  Expand **application_code** folder and make sure the application
    code available in migration folder

    ![](./media/image178.jpeg)

3.  From the left navigation menu, open the **PostgreSQL** extension and
    select your migration project folder

    ![](./media/image179.png)

4.  Under the **Application Migration** tile, click **Migrate
    Application**

    ![](./media/image180.png)

5.  In the **Application Conversion Folder** dropdown, select the
    appropriate application folder

    ![](./media/image181.png)

6.  Click on PostgreSQL connection drop down and select PostgreSQL
    connection

    ![](./media/image182.png)

7.  Click on Load Databases and Select PostgreSQL database dropdown

8.  Click on **Convert Application**

    ![](./media/image183.png)

    ![](./media/image184.png)

9.  Enable access to the **Claude Sonnet 4 model.** Allow model to
    connect to the PostgreSQL database server

    ![](./media/image185.png)

    ![](./media/image186.png)

10. Allow to list all database servers’ registration.

    ![](./media/image187.png)

11. Allow to connect to the PostgreSQL server. Allow to fetch database
    objects

    ![](./media/image188.png)

    ![](./media/image189.png)

    ![](./media/image190.png)

12. Allow to query database. Allow to fetch **booking_mgmt** database
    object

    ![](./media/image191.png)

    ![](./media/image192.png)

13. Click **Continue** to allow Copilot to iterate and refactor Java
    files for better understanding of database access patterns.

    ![](./media/image193.png)

14. Allow copilot to build the project using maven

    ![](./media/image194.png)

    ![](./media/image195.png)

    ![](./media/image196.png)

    ![](./media/image197.jpeg)

15. Generating migration report. Migration completed and report got
    generated

    ![](./media/image198.png)

16. Explore the generated report and review the following sections:

    - PostgreSQL environment and required extensions

    - Directory structure before and after migration

    - Files successfully converted

    - Database schema impact analysis

    - Key schema changes applied

    - Updated database connection settings

    - Data access layer updates

    - Features detected and handled during migration

    ![](./media/image199.jpeg)
    ![](./media/image200.jpeg)
    ![](./media/image201.jpeg)
    ![](./media/image202.jpeg)
    ![](./media/image203.jpeg)
    ![](./media/image204.jpeg)
    ![](./media/image205.jpeg)
    ![](./media/image206.jpeg)
    ![](./media/image207.jpeg)
    ![](./media/image208.jpeg)
    ![A screenshot of a computer AI-generated content may be
    incorrect.](./media/image209.png)
