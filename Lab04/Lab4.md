# Scenario:

ZAVA is a company with multiple divisions, including a legal services
branch, ZAVA Legal, which handles tenant and property dispute cases. The
division faces challenges in efficiently retrieving relevant legal
precedents and analyzing thousands of legal documents due to manual
processes and fragmented data.

To address this, ZAVA Legal is developing the ZAVA Legal Agent, an
AI-powered assistant that leverages Azure Database for PostgreSQL with
key extensions (azure_ai, vector, pg_diskann, age), GraphRAG, and OpenAI
models (GPT-4o and text-embedding-3-small). The agent performs semantic
searches, vector-based retrieval, and graph reasoning to provide
intelligent, context-aware legal case analysis.

Leading the initiative is Carlos Vega, CTO, supported by:

Elvia Aktins – Application Developer: As the main persona in this lab,
she is responsible for integrating the application with PostgreSQL,
deploying AI models, enabling AI-powered features, and testing the Legal
Agent.

Your role in this lab: Step into the role of Elvia Aktins, building and
testing the ZAVA Legal Agent by connecting PostgreSQL with Azure OpenAI
models, implementing semantic and graph-based queries, and enabling
advanced AI-driven legal document retrieval.

# Objective: 

In this lab, you will:

- Build the ZAVA Legal Agent, an AI-powered legal assistant.

- Integrate PostgreSQL with key extensions: azure_ai, vector,
  pg_diskann, and age.

- Deploy Azure OpenAI models: GPT-4o and text-embedding-3-small.

- Use GraphRAG to enhance query accuracy and reasoning.

- Populate and manage a legal cases database.

- Implement semantic vector search and graph-based queries for
  intelligent information retrieval.

- Develop and test the AI agent in Python for context-aware legal
  document analysis.

# Architecture Diagram

![](./media/image1.png)

# Exercise 1: Create a resource group

In this exercise, you will create a new Azure resource group to organize
and manage related resources for your lab.

1.  Open a web browser and navigate to Azure portal +++
    https://portal.azure.com/+++ and sign in using your credentials. 

2.  Go to the Resource groups, then select **+Create** to create a new
    resource. 

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image2.png)![A screenshot of a chat AI-generated
content may be incorrect.](./media/image3.png)

3.  Enter the following details: 

- Subscription: Keep the selected one as it is. 

- Resource group name: Enter +++ZAVA-Legal-RG+++ 

- Region: Select **East US 2** 

Click on **Review + create**. 

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image4.png)

4.  Click on **Create**.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image5.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image6.png)

Your resource group is created. 

# Exercise 2: Setting Up PostgreSQL Flexible Server and Database

In this task, you will provision a PostgreSQL Flexible Server, enable
necessary extensions, and create a database.

## Task 1: Provision PostgreSQL Flexible Server 

1.  Go to the Home page and search +++Azure Database for PostgreSQL
    Flexible Server+++ in the search bar.  

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image7.png)

2.  Click on **+Create**. 

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image8.png)

3.  Enter the following details:

- Subscription: Keep the default as it

- Resource group: Select ZAVA-Legal-RG

- Server name: Enter +++zava-legal-db+++

- Region: Select Central US

- PostgreSQL version: 16

- Workload type: Dev/Test

> ![](./media/image9.png)

4.  Under Authentication. Select PostgreSQL **authentication
    only** authentication method and enter the following login and
    password: 

- Administrator login: Enter +++pgAdmin+++ 

- Password: Enter +++p@ssw0rd1289+++ 

> Select **Next:Networking \>**
>
> ![](./media/image10.png)

5.  In the Networking tab, ensure that the connectivity
    method is **Public access(allowed IP addresses) and Private
    endpoint**, and allow **public access to this resource through the
    internet using a public IP address**. 

![](./media/image11.png)

6.  Under Firewall rules, enable “Allow public access from any Azure
    service within Azure to this server” and select **+Add current
    client IP address.** Click on **Create+Review.** 

![](./media/image12.png)

7.  Click **Create**.

![](./media/image13.png)

8.  Wait for the deployment to be completed. It will take 5-10 mins to
    complete. 

You have provisioned your PostgreSQL Flexible Server successfully.

![](./media/image14.png)

## Task 2: Create a database

1.  Click **Go to resource**. 

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image15.png)

2.  Expand **Settings** from the left-hand side menu and select
    **Databases**.

![](./media/image16.png)

3.  Click +Add button.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image17.png)

4.  Enter +++cases+++ database name and click **Save**.

![](./media/image18.png)

5.  This will create a new database on your server.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image19.png)

## Task 3: Enable Extensions

1.  Expand Settings from the left-hand side resource menu and select
    Server parameters. 

![](./media/image20.png)

2.  In the search bar, search for +++**azure.extensions+++** parameter. 

![](./media/image21.png)

3.  Search for the +++**VECTOR**+++ parameter in azure.extensions and
    then select the VECTOR parameter. This will allow
    the pgvector extension. 

![](./media/image22.png)

4.  Search for the +++**Diskann**+++ parameter in azure.extensions and
    then select the PG_DISKANN parameter. This will allow
    the pg_diskann extension. 

![](./media/image23.png)

5.  Search for the +++**azure_ai**+++ parameter in azure.extensions and
    then select the AZURE_AI parameter. This will allow
    the azure_ai extension. 

![](./media/image24.png)

6.  Search for the +++**age**+++ parameter in azure.extensions and then
    select the AZURE_AI parameter. This will allow the age extension. 

![](./media/image25.png)

7.  In the search bar, search for +++
    shared_preload_libraries+++ parameter. 

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image26.png)

8.  Select **AGE, AZURE_STORAGE, PG_CRON,** and **PG_STAT_STATEMENTS**
    from shared_preload_libraries.

![](./media/image27.png)![A screenshot of a computer AI-generated
content may be incorrect.](./media/image28.png)

9.  Click **Save** to save the changes.

![](./media/image29.png)

10. Click **Save and Restart**.

![](./media/image30.png)

## Task 4: Saving endpoints

1.  Click Go to resources.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image31.png)

2.  Save the endpoint of the server in a notepad for future use.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image32.png)

# Exercise 3: Provision OpenAI Service and Configure Models

In this exercise, you will provision an OpenAI service and deploy GPT-4o
and Text-Embedding-3-Small models in Foundry.

## Task 1: Provisioning OpenAI services

1.  Navigate to the home page of Microsoft Azure and select **Azure
    OpenAI**.

![](./media/image33.png)

2.  Click Create-\>Azure OpenAI.

![](./media/image34.png)

3.  Enter the following details:

- Subscription: Keep the default as it is.

- Resource group: Select **ZAVA-Legal-RG**

- Region: Select **Sweden Central**

- Name: Enter +++**azure-openai-serviceXXXXX**+++

- Pricing tier: Select **Standard S0**

Click **Next(3 times)**

![](./media/image35.png)

4.  Click Create.

![](./media/image36.png)

## Task 2: Save the endpoints and keys

1.  Click Go to resource.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image37.png)

2.  Click **Click here to view Endpoint** under the Essential section.

![](./media/image38.png)

3.  Save the value of **KEY 1** and **Endpoint** in Notepad for future
    use.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image39.png)

## Task 3: Deploy Gpt-4o model

1.  Select **Overview** from the left-hand side menu.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image40.png)

2.  Click **Go to Foundry portal**.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image41.png)

3.  Click **Deployments** from the left-hand menu.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image42.png)

4.  Click **Deploy model** -\> **Deploy base model**.

![](./media/image43.png)

5.  Search +++gpt-4o+++ model.

![](./media/image44.png)

6.  Select gpt-4o model and click Confirm.

![](./media/image45.png)

7.  Click **Deploy**.

![](./media/image46.png)

## Task 4: Deploy text-embedding-3-small model.

1.  Click **Deployments** from left-hand menu.

![](./media/image47.png)

2.  Click **Deploy model** -\> **Deploy base model**.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image48.png)

3.  Click the Interface task and select “embedding” and “Embeddings”.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image49.png)

4.  Select text-embedding-3-small and click Confirm.

![](./media/image50.png)

5.  Click **Customize** and increase **Capacity to 705k TPM,** and then
    click **Deploy**.

Your text-embedding-3-small model is deployed successfully.

![](./media/image51.png)

![](./media/image52.png)

You can also view the deployed model in **Deployments**.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image53.png)

# Exercise 4: Set Up the Development Environment

In this task, you will clone the repository, start Docker, enable the
Azure PostgreSQL extension in VS Code, and connect PostgreSQL with VS
Code.

## Task 1: Clone the repo.

1.  Open a command prompt/terminal and clone the following repo:

+++git clone
https://github.com/microsoft/ignite25-LAB515-build-advanced-ai-agents-with-postgresql.git+++

2.  Open the cloned repo in VS Code.

![](./media/image54.png)

## Task 2: Enable Azure PostgreSQL extension

1.  On the left-hand menu, click **Extensions**.

![](./media/image55.png)

2.  Search +++PostgreSQL+++.

![](./media/image56.png)

3.  Click install.

![](./media/image57.png)  
Now you are able to view and access the PostgreSQL extension icon on the
left-hand menu.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image58.png)

## Task 3: Run desktop docker

1.  Open **Docker Desktop**. It should be running in the background.

Note: Do not close Docker.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image59.png)

## Task 4: Connecting Azure PostgreSQL Flexible Server to VS Code.

1.  Click the Elephant Icon on the left navigation.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image58.png)

2.  Once the extension loads, click the **Add Connection** button in the
    POSTGRESQL panel

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image60.png)

3.  Enter the following details:

- Server Name: Enter the name of your server
  e.g,”zava-legal-db.postgres.database.azure.com”

- Authentication type: Select Password

- User Name: Enter +++ pgAdmin+++

- Password: Enter +++p@ssw0rd1289+++

- Server Group: Keep Servers as it is

> Click **Save & Connect**.
>
> ![A screenshot of a computer AI-generated content may be
> incorrect.](./media/image61.png)![A screenshot of a computer
> AI-generated content may be incorrect.](./media/image62.png)

Your Database is connected.

# Exercise 5: Populate and Explore the Database

In this exercise, you will populate the database with sample data and
explore its structure and contents.

## Task 1: Connect and Populate the Database with Sample Data

1.  From the left-hand side menu, expand the **Databases** node.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image63.png)

2.  You can view all the databases present on your server. Now,
    **right-click** on the database named **cases** and select the
    **Connect with PSQL** option.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image64.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image65.png)

3.  Run the following command to create the “cases” tables and data

+++ \i ./Scripts/initialize_dataset.sql;+++

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image66.png)

## Task 2: Explore Database

1.  When working with psql at the VS Code Command Line Shell, enabling
    the extended display for query results may be helpful, as it
    improves the readability of output for subsequent commands. Execute
    the following command to allow the extended display to be
    automatically applied.

+++\x auto+++

![](./media/image67.png)

2.  Retrieve a sample of data from the cases table. This allows you to
    examine the structure and content of the data stored in the
    database.

+++ SELECT name FROM cases LIMIT 5;+++

![](./media/image68.png)

## Task 3: Install and configure the azure_ai extension

1.  Execute the following command in VS Code PSQL Command Line Shell to
    verify that the azure_ai, vector, age and pg_diskann extensions were
    successfully added to your server's allowlist.

+++SHOW azure.extensions;+++

![](./media/image69.png)

2.  Run the following command to install the azure_ai extension.

+++ CREATE EXTENSION IF NOT EXISTS azure_ai;+++

![](./media/image70.png)

## Task 4: Set up the .env file

1.  Go to the project explorer.

![](./media/image71.png)

2.  Locate **.env.sample** file in the project folder.

![](./media/image72.png)

3.  Rename it to **.env file** and enter all the values of the following
    fields:

> \# Azure OpenAI Configuration
>
> AZURE_OPENAI_ENDPOINT=your_openai_endpoint_here
>
> AZURE_OPENAI_KEY=your_openai_key_here
>
> \# Database Configuration
>
> AZURE_PG_HOST=your_postgres_host_here
>
> AZURE_PG_NAME=cases
>
> AZURE_PG_USER=your_postgres_username_here
>
> AZURE_PG_PASSWORD=your_postgres_password_here
>
> AZURE_PG_PORT=5432
>
> AZURE_PG_SSLMODE=require

![](./media/image73.png)

4.  Press **Ctrl+S** to save the file.

![](./media/image74.png)

## Task 5: Connecting azure_ai PostgreSQL extension to Azure OpenAI

1.  Navigate back to the psql terminal.

2.  Run the following commands:

> +++SELECT azure_ai.set_setting('azure_openai.endpoint',
> '{AZURE_OPENAI_ENDPOINT}');+++
>
> +++SELECT azure_ai.set_setting('azure_openai.subscription_key',
> '{AZURE_OPENAI_KEY}');+++
>
> **Note**: Replace
> the {AZURE_OPENAI_ENDPOINT} and {AZURE_OPENAI_KEY} tokens with the
> values.
>
> These commands configure the azure_ai extension by **setting the Azure
> OpenAI endpoint** and **providing the API key** so the database can
> connect and authenticate with Azure OpenAI.
>
> ![](./media/image75.png)![A screen shot of a computer AI-generated
> content may be incorrect.](./media/image76.png)

3.  You can verify the settings written into the azure_ai.settings table
    using the azure_ai.get_setting() function in the following queries:

> +++SELECT azure_ai.get_setting('azure_openai.endpoint');+++
>
> +++SELECT azure_ai.get_setting('azure_openai.subscription_key');+++
>
> ![A screenshot of a computer screen AI-generated content may be
> incorrect.](./media/image77.png)

Congratulations, the azure_ai PostgreSQL extension is now connected to
your Azure OpenAI account.

## Task 6: Review the Azure OpenAI schema

1.  Run the following query, which creates a vector embedding for a
    sample query. The deployment_name parameter in the function is set
    to embedding, which is the name of the deployment of
    the text-embedding-3-small model in your Azure OpenAI service:

> +++SELECT
> LEFT(azure_openai.create_embeddings('text-embedding-3-small', 'Sample
> text for PostgreSQL Lab')::text, 100) AS vector_preview;+++
>
> ![A screenshot of a computer AI-generated content may be
> incorrect.](./media/image78.png)
>
> **Note:** For brevity, the vector embeddings are abbreviated in the
> above output.
>
> Embeddings convert text or data into vectors so models can measure
> relationships and similarities efficiently. The azure_ai extension can
> generate these embeddings, which can be stored in the database using a
> vector extension.

# Exercise 6: Using AI-Driven Features in PostgreSQL

In this exercise, you will use pattern matching, semantic vector search
with a DiskANN index, and perform a semantic search query in PostgreSQL.

Task 1: Using Pattern matching for queries

1.  Click on the Elephant Icon on the left navigation to open the VS
    Code **PostgreSQL extension**.

![](./media/image79.png)

2.  Expand the **Databases** Node, and right-click
    on **cases** database, select the **New Query** option.

![](./media/image80.png)

This will open a new **query editor window**.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image81.png)

3.  We will start by searching for cases mentioning "Water leaking into
    the apartment from the floor above." Enter the following query in
    the query edit window and press the **play** button to run the
    query.

SELECT id, name, opinion

FROM cases

WHERE opinion ILIKE '%Water leaking into the apartment from the floor
above';

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image82.png)

> However, it does not find any results because those exact words are
> not mentioned in the opinion. As you can see there are no results for
> what to user wants to find. We need to try another approach.
>
> Next we will see how we can improve on this using Semantic Vector
> Search.

## Task 2: Using Semantic Vector Search and DiskANN Index

1.  Install the vector extension using the following command in the
    query edit window and press the **play** button to run the query.

+++CREATE EXTENSION IF NOT EXISTS vector;+++

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image83.png)

2.  Add the embedding vector column. The text-embedding-3-small model is
    configured to return 1,536 dimensions, so use that for the vector
    column size.

+++ALTER TABLE cases ADD COLUMN opinions_vector vector(1536);+++

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image84.png)

3.  Generate an embedding vector for the opinion of each case by calling
    Azure OpenAI through the create_embeddings user-defined function,
    which is implemented by the azure_ai extension:

> UPDATE cases
>
> SET opinions_vector =
>
> azure_openai.create_embeddings(
>
> 'text-embedding-3-small',
>
> name || LEFT(opinion, 8000),
>
> max_attempts =\> 10,
>
> retry_delay_ms =\> 2000
>
> )::vector
>
> WHERE ctid IN (
>
> SELECT ctid FROM cases
>
> WHERE opinions_vector IS NULL
>
> ORDER BY id
>
> LIMIT 25
>
> );
>
> Note: This may take several minutes, depending on the available quota.
>
> ![A screenshot of a computer AI-generated content may be
> incorrect.](./media/image85.png)

4.  Run the following command to add DiskANN Vector Index to improve
    vector search speed.

+++ CREATE EXTENSION IF NOT EXISTS pg_diskann;+++

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image86.png)

5.  Create the diskann index on a table column that contains vector
    data.

+++CREATE INDEX cases_cosine_diskann ON cases USING
diskann(opinions_vector vector_cosine_ops);+++

> As you scale your data to millions of rows, DiskANN makes vector
> search more efficient.
>
> ![A screenshot of a computer AI-generated content may be
> incorrect.](./media/image87.png)

## Task 3: Perform a Semantic Search Query

1.  Now that you have listing data augmented with embedding vectors,
    it's time to run a semantic search query. To do so, get the query
    string embedding vector, then perform a cosine search to find the
    cases whose opinions are most semantically similar to the query.

2.  Generate the embedding for the query string.

+++SELECT azure_openai.create_embeddings('text-embedding-3-small',
'Water leaking into the apartment from the floor above.');+++

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image88.png)

3.  Use the embedding in a cosine search (\<=\> represents cosine
    distance operation), fetching the top 10 most similar cases to the
    query.

> SELECT id, name
>
> FROM cases
>
> ORDER BY opinions_vector \<=\>
> azure_openai.create_embeddings('text-embedding-3-small', 'Water
> leaking into the apartment from the floor above.')::vector
>
> LIMIT 10;
>
> ![](./media/image89.png)

4.  You may also project the opinion column to be able to read the text
    of the matching rows whose opinions were semantically similar. For
    example, this query returns the best match:

> SELECT id, opinion
>
> FROM cases
>
> ORDER BY opinions_vector \<=\>
> azure_openai.create_embeddings('text-embedding-3-small', 'Water
> leaking into the apartment from the floor above.')::vector
>
> LIMIT 1;
>
> ![A screenshot of a computer AI-generated content may be
> incorrect.](./media/image90.png)
>
> To intuitively understand semantic search, observe that the opinion
> mentioned doesn't actually contain the terms "Water leaking into the
> apartment from the floor above.". However, it does highlight a
> document with a section that says "nonsuit and dismissal, in an action
> brought by a tenant to recover damages for injuries to her goods,
> caused by leakage of water from an upper story" which is similar.

# Exercise 7: Run the AI-Powered Legal Agent

In this exercise, you will run the Agent App, integrating PostgreSQL,
semantic and graph-based queries, OpenAI models, and memory to test the
AI-powered legal assistant.

1.  Navigate to the project Explorer.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image91.png)

2.  Right-click on the project to open the integrated terminal.

![](./media/image92.png)

3.  In the terminal, run the following commands to create a virtual
    Python environment.

+++python3.12 -m venv venv+++

+++source venv/bin/activate+++

Note: Here Python version should be Python 3.12.11

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image93.png)

4.  Run the following command to install all the requirements:

+++pip install -r requirements.txt+++

![A screenshot of a computer screen AI-generated content may be
incorrect.](./media/image94.png)

5.  From the Explorer, expand the **Code** folder and select
    **lab.ipynb** file.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image95.png)

![](./media/image96.png)

6.  Run the code block present in the **Setup the Agent App Python
    imports** section using the **play** button. This section sets up
    the Python environment for the Agent App by importing necessary
    libraries for database access, asynchronous execution, environment
    management, and AI agent functionality. It prepares all core modules
    and Agent Framework components required to build and run the
    AI-powered legal assistant.

7.  When you click on the Play button, you will get a drop-down menu.
    From the given menu, select Python Environments.

![](./media/image97.png)

8.  Select **venv(Python3.12.11)**

![](./media/image98.png)

9.  Re-run it.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image99.png)

10. Run the code block present in the **Environment Setup** section
    using the play button. This section configures the lab environment
    by loading environment variables from the **.env** file, including
    Azure OpenAI credentials and PostgreSQL connection details. It
    ensures the Agent App has access to all required services before
    running any code.

![](./media/image100.png)

11. Execute the code present in the **create agent functions for basic
    database queries** section. In this section, you define custom agent
    functions to interact with the PostgreSQL database, including
    counting cases and performing keyword searches. These functions are
    registered with the Agent Framework to be callable by the AI agent
    during queries.

![](./media/image101.png)

12. Execute the code present in **Test Run of our New Agent**. This
    section runs the agent with the basic database functions to verify
    that it can query the PostgreSQL database and return results
    correctly.

![](./media/image102.png)

13. Execute the code present in improve agent accuracy with semantic
    re-ranking using the .rank() function. Here, update the value of
    LIMIT 15 to LIMIT 5.

![](./media/image103.png)![](./media/image104.png)

14. Click on PostgreSQL extension from left-hand menu.

![](./media/image105.png)

15. Expand the **Databases** Node, and right-click on the
    **cases** database, select the **Connect with PSQL** option.

![](./media/image106.png)

16. Run the following command to drop the existing public.temp_cases.

+++ DROP TABLE IF EXISTS public.temp_cases;+++

![A black screen with white text AI-generated content may be
incorrect.](./media/image107.png)

17. Create a new public.temp_cases table using the following command.

+++ CREATE TABLE public.temp_cases(data jsonb);+++

![A black screen with white text AI-generated content may be
incorrect.](./media/image108.png)

18. Run the following command to copy the data from cases.csv file.

\COPY public.temp_cases (data)

FROM '/Users/ankitasaini/Desktop/cases.csv'

WITH (FORMAT csv, HEADER true);

![A black screen with white text AI-generated content may be
incorrect.](./media/image109.png)

19. Navigate back to the **explorer**. From the left-hand menu, expand
    the **Scripts** folder.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image110.png)

20. Open the **create_graph.sql** file and then click on the **Play**
    button to run it.

![A screen shot of a computer AI-generated content may be
incorrect.](./media/image111.png)

21. It will ask you about your current server details, so select your
    server.

![A screenshot of a computer program AI-generated content may be
incorrect.](./media/image112.png)

![](./media/image113.png)

22. Navigate back to PostgreSQL extension from left-hand menu and expand
    the **Databases** Node, and right-click on the **cases** database,
    select the **Connect with PSQL** option.

23. Run the following command to enable the age extension.

+++CREATE EXTENSION IF NOT EXISTS age CASCADE;+++

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image114.png)

24. View all the extensions using +++\dx+++

![](./media/image115.png)

25. Set the search path.

+++ SET search_path = ag_catalog, "$user", public;+++

![A screen shot of a computer AI-generated content may be
incorrect.](./media/image116.png)

26. Create a case graph and then verify that the graph is created
    successfully or not.

+++ SELECT create_graph('case_graph');+++

+++ SELECT \* FROM ag_graph;++++

![](./media/image117.png)

27. Navigate back to the lab.ipynb file and run the code present in
    **add a GraphRAG query function to the agent for additional accuracy
    improvements** section.

![](./media/image118.png)

28. Execute the code block of **re-assemble our agent with new advanced
    functions and re-test** section.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image119.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image120.png)

29. Add a **new weather agent function to the agent** and you will get
    the following output.

![A screenshot of a computer program AI-generated content may be
incorrect.](./media/image121.png)

30. Re-testing the agent after adding a new weather agent function.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image122.png)

31. Now you will add memory to the agent and execute the final agent.
    Run all the remaining cells.

![](./media/image123.png)

# Summary

In this lab, you built the **ZAVA Legal Agent**, an AI-powered legal
assistant by integrating **GraphRAG**, **Azure OpenAI models** (GPT-4o
and text-embedding-3-small), and **PostgreSQL** with key extensions
including **azure_ai, vector, pg_diskann, and age**. You provisioned and
configured PostgreSQL as the backend database for legal cases, deployed
OpenAI models to generate embeddings for semantic search, and leveraged
GraphRAG along with the extensions to enhance query accuracy and
performance. Finally, you developed and tested the agent in Python,
demonstrating intelligent, context-aware legal document retrieval
through a combination of vector search, graph reasoning, and AI-powered
analysis.
