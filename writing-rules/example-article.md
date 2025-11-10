[AWS Database Blog](https://aws.amazon.com/blogs/database/)
-----------------------------------------------------------

 

Connect Amazon Bedrock Agents with Amazon Aurora PostgreSQL using Amazon RDS Data API
=====================================================================================

by Nihilson Gnanadason and Senthil Mohan on 23 MAY 2025 in [Advanced (300)](https://aws.amazon.com/blogs/database/category/learning-levels/advanced-300/ "View all posts in Advanced (300)"), [Amazon Aurora](https://aws.amazon.com/blogs/database/category/database/amazon-aurora/ "View all posts in Amazon Aurora"), [Amazon Bedrock](https://aws.amazon.com/blogs/database/category/artificial-intelligence/amazon-machine-learning/amazon-bedrock/ "View all posts in Amazon Bedrock"), [PostgreSQL compatible](https://aws.amazon.com/blogs/database/category/database/amazon-aurora/postgresql-compatible/ "View all posts in PostgreSQL compatible"), [Technical How-to](https://aws.amazon.com/blogs/database/category/post-types/technical-how-to/ "View all posts in Technical How-to") [Permalink](https://aws.amazon.com/blogs/database/connect-amazon-bedrock-agents-with-amazon-aurora-postgresql-using-amazon-rds-data-api/) [Comments](https://aws.amazon.com/blogs/database/connect-amazon-bedrock-agents-with-amazon-aurora-postgresql-using-amazon-rds-data-api/#Comments) [Share](#)

*   [](https://www.facebook.com/sharer/sharer.php?u=https://aws.amazon.com/blogs/database/connect-amazon-bedrock-agents-with-amazon-aurora-postgresql-using-amazon-rds-data-api/)
*   [](https://twitter.com/intent/tweet/?text=Connect%20Amazon%20Bedrock%20Agents%20with%20Amazon%20Aurora%20PostgreSQL%20using%20Amazon%20RDS%20Data%20API&via=awscloud&url=https://aws.amazon.com/blogs/database/connect-amazon-bedrock-agents-with-amazon-aurora-postgresql-using-amazon-rds-data-api/)
*   [](https://www.linkedin.com/shareArticle?mini=true&title=Connect%20Amazon%20Bedrock%20Agents%20with%20Amazon%20Aurora%20PostgreSQL%20using%20Amazon%20RDS%20Data%20API&source=Amazon%20Web%20Services&url=https://aws.amazon.com/blogs/database/connect-amazon-bedrock-agents-with-amazon-aurora-postgresql-using-amazon-rds-data-api/)
*   [](mailto:?subject=Connect%20Amazon%20Bedrock%20Agents%20with%20Amazon%20Aurora%20PostgreSQL%20using%20Amazon%20RDS%20Data%20API&body=Connect%20Amazon%20Bedrock%20Agents%20with%20Amazon%20Aurora%20PostgreSQL%20using%20Amazon%20RDS%20Data%20API%0A%0Ahttps://aws.amazon.com/blogs/database/connect-amazon-bedrock-agents-with-amazon-aurora-postgresql-using-amazon-rds-data-api/)
*   

[Generative artificial intelligence](https://aws.amazon.com/ai/generative-ai/) (AI) applications and relational databases are increasingly being used together to create new solutions across industries. The integration of these technologies allows organizations to use the vast amounts of structured data stored in relational databases to train and refine AI models. AI can then be used to generate insights, predict trends, and even augment database management tasks.

In this post, we describe a solution to integrate generative AI applications with relational databases like [Amazon Aurora PostgreSQL-Compatible Edition](https://aws.amazon.com/rds/aurora/) using [RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html) (Data API) for simplified database interactions, [Amazon Bedrock](https://aws.amazon.com/bedrock) for AI model access, [Amazon Bedrock Agents](https://aws.amazon.com/bedrock/agents/) for task automation and [Amazon Bedrock Knowledge Bases](https://aws.amazon.com/bedrock/knowledge-bases/) for context information retrieval. Data API support is currently available only with Aurora databases. If you intend to use the solution with [Amazon Relational Database Service](https://aws.amazon.com/rds) (Amazon RDS), you can customize the integration using conventional database connectivity approaches.

Solution overview
-----------------

This solution combines the AI capabilities of Amazon Bedrock Agents with the robust database functionality of Aurora PostgreSQL through the Data API. Amazon Bedrock Agents, powered by [large language models](https://aws.amazon.com/what-is/large-language-model/) (LLMs), interprets natural language queries and generates appropriate SQL statements using action groups fulfilled by [AWS Lambda](https://aws.amazon.com/lambda/) function and schema artifacts stored in [Amazon Simple Storage Service](https://aws.amazon.com/s3/) (Amazon S3). These queries are then run against Aurora PostgreSQL using the Data API, which provides a serverless, connection-free method for database interactions. This enables dynamic data retrieval and analysis without managing direct database connections. For instance, a user request to “show sales data for the last quarter” is transformed into SQL, run through the Data API, and the results are presented in a user-friendly format. The solution is represented in the following architecture diagram.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2025/05/20/image-1-3.png)

The detailed steps in this architecture are:

1.  The generative AI application invokes the Amazon Bedrock agent with natural language input data to orchestrate the integration with the backend relational database.
2.  The agent invokes the foundational model (FM) on Amazon Bedrock for pre-processing the prompt to determine the actions to be taken.
3.  The agent then decides to use the generate-query action group.
4.  The agent invokes the `/generate` API implemented by the Lambda function.
5.  The Lambda function uses the schema artifacts from the Amazon S3 bucket as context to augment the prompt.
6.  The Lambda function then invokes an LLM on Amazon Bedrock to generate the SQL query and returns the generated SQL query back to the agent.
7.  The agent then decides to use the execute-query action group.
8.  The agent invokes the /execute API implemented by the Lambda function, passing the generated SQL query.
9.  The Lambda function use Data API with a read-only role to run the SQL query against the Aurora PostgreSQL database.
10.  The agent finally returns the formatted query results to the application.

While the solution can technically support write operations, allowing AI generated queries to modify the database presents significant risks to data integrity and security. Therefore, production implementations should allow access to read-only operations through proper IAM policies and database role permissions. From a security and data integrity perspective, we strongly recommend implementing this solution exclusively for read-only workloads such as analytics, reporting, and data exploration.

Security guardrails
-------------------

The solution implements multiple layers of security controls to ensure safe and controlled access to the database. These layered security controls ensure that the solution maintains data integrity while providing the desired natural language query capabilities:

*   **Agent-level instructions:** The Bedrock agents are explicitly configured to support only read-only operations. Instructions embedded in the agent’s prompt prevent it from generating queries that could modify the database (INSERT, UPDATE, DELETE).
    
        You are a SQL query assistant that helps users interact with a PostgreSQL database. 
        You can generate read only (SELECT) SQL queries 
        based on natural language prompts and execute queries against the database. 
        Do not generate SQL queries that can modify or update 
        any underlying data or schema in the database. 
        Always validate queries for security before execution. 
        Use the generate-query action to create SQL queries 
        and the execute-query action to run them.
    
    Code
    
*   **Action group validation**: The generate-query function implements additional validation of user input to prevent injection attacks and unauthorized operations. The execute-query function validates the generated SQL queries against an allowlist of operations and syntax patterns. Both functions work in tandem to make sure query safety before they are run.
*   **Read-only database access**: Database interactions are exclusively performed using a read-only role (configured via READONLY\_SECRET\_ARN). This provides a critical security boundary at the database level, preventing any potential write operations even if other controls fail.
*   **Bedrock guardrails**: Additional security is implemented through [Amazon Bedrock Guardrails](https://aws.amazon.com/bedrock/guardrails/) features. This further allows filtering of specific words or phrases like INSERT, UPDATE, DELETE from user prompts to prevent potentially harmful or unauthorized requests before they reach the query generation stage.

Prerequisites
-------------

To follow along with the steps in this post, you need the following resources.

*   An AWS account with [AWS Identity and Access Management](https://aws.amazon.com/iam/) (IAM) permissions to create an Aurora PostgreSQL database and Amazon Bedrock.
*   An integrated development environment (IDE) such as Visual Studio Code.
*   Python installed with the [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html) library on your IDE.
*   [AWS Cloud Development Kit](https://docs.aws.amazon.com/cdk/v2/guide/getting-started.html#getting_started_install) (AWS CDK) installed on your IDE.

Clone the sample Python project from the AWS Samples [repository](https://github.com/aws-samples/sample-to-connect-bedrock-agent-with-aurora) and follow the development environment setup instructions in the readme:

    git clone https://github.com/aws-samples/sample-to-connect-bedrock-agent-with-aurora
    cd sample-to-connect-bedrock-agent-with-aurora

Code

**Setting up the database environment**
---------------------------------------

You begin by deploying an Aurora PostgreSQL cluster using the AWS CDK. To deploy the database infrastructure, run the command:

    cdk deploy RDSAuroraStack

Bash

The RDSAuroraStack is an AWS CDK construct that provisions an Aurora PostgreSQL Serverless v2 database in a secure VPC environment. It creates a dedicated VPC with public and private subnets, sets up security groups, and manages database credentials through AWS Secrets Manager. The stack also implements a custom Lambda-based solution to create a read-only database user with appropriate permissions, making it suitable for applications that need segregated database access levels, such as connecting Amazon Bedrock agents to Aurora PostgreSQL databases.

**Deploy the agent**
--------------------

Agents orchestrate interactions between [foundation models](https://aws.amazon.com/what-is/foundation-models/) (FMs), data sources, software applications, and user conversations. Also, agents can automatically call APIs to take actions and invoke [Amazon Bedrock Knowledge Bases](https://aws.amazon.com/bedrock/knowledge-bases/) to augment with contextual information. Integrating the agent with Amazon Aurora, you gain the ability to transform natural language inputs into precise SQL queries using generative AI capabilities. Deploy the agent using the AWS CDK command:

    cdk deploy BedrockAgentStack

Bash

The BedrockAgentStack is an AWS CDK construct that creates an Amazon Bedrock agent designed to interact with an Aurora PostgreSQL database through natural language queries. It provisions a Lambda function that can generate and execute SQL queries, implements a comprehensive guardrail system to prevent data modification operations (allowing only SELECT queries), and establishes the necessary IAM roles and permissions for secure communication between Bedrock and Aurora. The stack creates two action groups—one for generating SQL queries from natural language prompts and another for executing those queries—while integrating with the previously deployed Aurora PostgreSQL database using Data API.

After the CDK stack is deployed, you can switch to the Bedrock Agent builder console to review the configurations of the `query-agent` as shown in the following screenshot.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2025/05/20/image-2-3.png)

You will first notice that the agent is configured to use Anthropic’s Claude LLM. Next, from the Agent builder console examine the agent’s instructions because they define the agent’s functionality. The agent also features two key [action groups](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-action-create.html): `generate-sql-query` and `execute-sql-query`. An action group is a logical collection of related functions (actions) that an agent can perform to accomplish specific tasks. The `generate-sql-query` group invokes the `generate-query` Lambda function, accepting the input user question, and returns the generated SQL query. The `execute-sql-query` group invokes the `execute-query` Lambda function, accepting the query and parameter values. We explore these functions in the next sections.

### The generate-query function

The `generate_sql_query` function uses an LLM to create SQL queries from natural language questions and a provided database schema. The function uses a detailed prompt containing instructions for SQL generation, the database schema, and example question-SQL pairs. It then formats this prompt and the user’s question into a structured input for the LLM. The function proceeds to call an `invoke_llm` method to obtain a response from the LLM. The SQL query is then extracted from the LLM’s output and returned. This method enables dynamic SQL generation based on natural language input while providing the LLM with necessary context about the database structure for accurate query creation. The following code shows the prompt used to invoke the LLM.

    def generate_sql_query(question):
        validated_question = validate_input(question)
        schema_content = read_schema_file()
        
        # Construct the prompt with schema context
        contexts = f"""
        <Instructions>
            Read database schema inside the <database_schema></database_schema> tags which contains the tables and schema information to do the following:
            1. Create a syntactically correct SQL query to answer the question.
            2. Format the query to remove any new line with space and produce a single line query.
            3. Never query for all the columns from a specific table, only ask for a few relevant columns given the question.
            4. Pay attention to use only the column names that you can see in the schema description. 
            5. Be careful to not query for columns that do not exist. 
            6. Pay attention to which column is in which table. 
            7. Qualify column names with the table name when needed.
            8. Return only the sql query without any tags.
        </Instructions>
        <database_schema>{schema_content}</database_schema>
        <examples>
        <question>"How many users do we have?"</question>
        <sql>SELECT SUM(users) FROM customers</sql>
        <question>"How many users do we have for Mobile?"</question>
        <sql>SELECT SUM(users) FROM customer WHERE source_medium='Mobile'</sql>
        </examples>
        <question>{validated_question}</question>
        Return only the SQL query without any explanations.
        """
        prompt = f"""
        Human: Use the following pieces of context to provide a concise answer to the question at the end. If you don't know the answer, just say that you don't know, don't try to make up an answer.
        <context>
        {contexts}
        </context
        Question: {validated_question}
        Assistant:
        """
        messages = [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": prompt.format(contexts, validated_question)}
                ],
            }
        ]
        llm_response = invoke_llm(messages)
        return llm_response["content"][0]["text"]

Python

In this example, we are embedding the entire schema file as context into the prompt because the schema is small and simple.

    # Add function to read SQL file
    def read_schema_file():
        try:
            # Get the directory where the lambda function code is located
            lambda_dir = os.path.dirname(os.path.abspath(__file__))
            schema_path = os.path.join(lambda_dir, "schema.sql")
            with open(schema_path, "r") as file:
                schema_content = file.read()
            return schema_content

Python

For large and complex schemas, you can further enhance the agent’s capabilities by integrating [Amazon Knowledge Bases](https://aws.amazon.com/bedrock/knowledge-bases/) through vector embeddings and semantic search, storing comprehensive schema definitions, table relationships, sample queries, and business context documents. By implementing a retrieval-augmented generation (RAG) approach, the agent first searches the Knowledge Base for contextual information before invoking the generate-query action group function, significantly reducing model inference time.

### The execute-query function

The `execute_query` function uses Data API to execute a SQL query against an Aurora PostgreSQL database, taking the SQL query and parameters as inputs. It returns the response from the database. The Data API eliminates the complexity of managing database connections in serverless architectures by providing a secure HTTPS endpoint that handles connection management automatically, removing the need for VPC configurations and connection pools in Lambda functions.

The `lambda_handler` processes an incoming event, extracting parameters and their values into a dictionary. It then retrieves a SQL query from these parameters, replacing newlines with spaces for proper formatting. Finally, it calls the `execute_query` function with the extracted query and parameters to execute the SQL statement. This setup allows for dynamic SQL query execution in a serverless environment, with the ability to pass in different queries and parameters for each invocation of the Lambda function:

    def execute_query(query, parameters=None):
        try:
            # Base request parameters
            request_params = {
                "resourceArn": DB_CLUSTER_ARN,
                "secretArn": DB_SECRET_ARN,
                "database": DB_NAME,
                "sql": query,
            }
    
            # Only add parameters if they exist and are not empty
            if parameters and len(parameters) > 0:
                request_params["parameters"] = parameters
    
            # Execute the query
            response = rds_data.execute_statement(**request_params)
            return response
    
        except Exception as e:
            print(f"Error executing query: {str(e)}")
            raise

Python

Test the solution
-----------------

You must create a sample schema with data using [`scripts/create_schema.py`](https://github.com/aws-samples/sample-to-connect-bedrock-agent-with-aurora/blob/main/scripts/create_schema.py) before you can proceed with the testing. This script will create a few schemas and tables and ingest sample data. The test is very straightforward. You first send a natural language prompt as input to the agent, which then has to generate the necessary SQL query and execute the query against the configured Aurora PostgreSQL database using Data API. The agent should then return a response that is based on the results queried from the Aurora PostgreSQL database using Data API. Before you run the [`scripts/create_schema.py`](https://github.com/aws-samples/sample-to-connect-bedrock-agent-with-aurora/blob/main/scripts/create_schema.py) script from your IDE, update it with your `DB_CLUSTER_ARN`, `DB_SECRET_ARN`, `DB_NAME` noted from your RDSAuroraStack CDK deployment output.

    python3 scripts/create_schema.py

Bash

After you’ve created the schema and loaded the test data, there are few ways you can test the deployed agent. One approach is to use the [Amazon Bedrock console](https://console.aws.amazon.com/bedrock/), and the other is to make use of the [InvokeAgent API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_InvokeAgent.html).

### Using the test window of Amazon Bedrock Agents

To test the solution using the test window, follow these steps:

1.  On the [Amazon Bedrock Agents console](https://console.aws.amazon.com/bedrock/agents/), on the panel on the right, open the **Test** window.
2.  Enter your test input for the agent, as shown in the following screenshot.  
    ![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2025/05/20/image-3-3.png)
3.  To troubleshoot and review all the steps the agent has used to generate the response, expand the test window and review the **Trace** section, as shown in the following screenshot.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2025/05/20/image-4-1.png)

### Using the Amazon Bedrock InvokeAgent API

Another way to test is by using the AWS SDK for InvokeAgent API. Applications use this API for interacting with the agent. You should find a utility script [`scripts/test_agent.py`](https://github.com/aws-samples/sample-to-connect-bedrock-agent-with-aurora/blob/main/scripts/test_agent.py) in the [repository](https://github.com/aws-samples/sample-to-connect-bedrock-agent-with-aurora) to test the agent and the integration with the Aurora PostgreSQL database. Make sure you update the script with the Amazon Bedrock agent ID before you run the script. The test script provides options for you to run with a single test prompt or run with multiple test prompts. You can also run the test with trace enabled to review the steps and reasoning that the agent used to complete the request.

Run a single test without trace:

    python3 scripts/test_agent.py --test-type single

Bash

Run a single test with trace:

    python3 scripts/test_agent.py --test-type single --trace

Bash

Run all tests without trace:

    python3 scripts/test_agent.py --test-type all

Bash

Run all tests with trace:

    python3 scripts/test_agent.py --test-type all --trace

Bash

The following is the sample output from the test, showing the actual data retrieved from the Aurora PostgreSQL database table:

    BEDROCK AGENT TEST RESULTS
    ==================================================
    
    Single Test
    --------------------------------------------------
    Prompt: Show me all students and their major department names ?
    
    Invoking Agent...
    
    Agent Response:
    Here are the students and their major department names:
    
    John Doe - Computer Science 
    Jane Smith - Physics
    Alice Johnson - Mathematics
    
    ==================================================

Code

The following is the sample output with trace enabled showing all the steps of the agent:BEDROCK AGENT TEST RESULTS

    BEDROCK AGENT TEST RESULTS
    ==================================================
    
    Single Test
    --------------------------------------------------
    Prompt: Can you find the members of AI in Education project ?
    
    Invoking Agent...
    
    Agent Response:
    The members of the AI in Education project are:
    
    Robert Brown - Principal Investigator
    Emily Davis - Co-Investigator
    
    📋 Trace Steps:
    ================================================================================
    
    🔍 Step 1 - Skip printing trace entry:
    ----------------------------------------
    
    🔍 Step 2 - Pre-processing:
    ----------------------------------------
    Response:  <category>D>
    
    ⚙️ Step 3 - Skip printing trace entry:
    ----------------------------------------
    
    ⚙️ Step 4 - Orchestration:
    ----------------------------------------
    Response: To answer this question, I will:
    
    1. Call the generate-sql-query::generate-sql function to generate the SQL query to get the members of the "AI in Education" project.
    
    2. Call the execute-sql-query::execute-sql function to execute the generated SQL query.
    
    3. Return the results from the execute-sql-query function to the user.
    
    I have checked that I have access to the generate-sql-query::generate-sql and execute-sql-query::execute-sql functions.
    
    </scratchpad>
    
    <function_call>
    generate-sql-query::generate-sql(question="Can you find the members of AI in Education project ?")
    
    ⚙️ Step 5 - Skip printing trace entry:
    ----------------------------------------
    
    ⚙️ Step 6 - Orchestration:
    ----------------------------------------
    invocationInput: {'actionGroupInvocationInput': {'actionGroupName': 'generate-sql-query', 'executionType': 'LAMBDA', 'function': 'generate-sql', 'parameters': [{'name': 'question', 'type': 'string', 'value': 'Can you find the members of AI in Education project ?'}]}, 'invocationType': 'ACTION_GROUP', 'traceId': '5018e885-ffa2-4a74-ac12-1635c58ceb88-0'}
    
    ⚙️ Step 7 - Orchestration:
    ----------------------------------------
    Observation: {'actionGroupInvocationOutput': {'text': '{"result": "SELECT e.first_name, e.last_name, pm.role FROM staff.employees e JOIN research.project_members pm ON e.employee_id = pm.employee_id JOIN research.projects p ON pm.project_id = p.project_id WHERE p.title = \'AI in Education\';"}'}, 'traceId': '5018e885-ffa2-4a74-ac12-1635c58ceb88-0', 'type': 'ACTION_GROUP'}
    
    ⚙️ Step 8 - Skip printing trace entry:
    ----------------------------------------
    
    ⚙️ Step 9 - Orchestration:
    ----------------------------------------
    Response: <scratchpad>
    I have received the generated SQL query to get members of the "AI in Education" project. I will now call the execute-sql-query::execute-sql function to execute this query.
    </scratchpad>
    <function_call>execute-sql-query::execute-sql(query="SELECT e.first_name, e.last_name, pm.role FROM staff.employees e JOIN research.project_members pm ON e.employee_id = pm.employee_id JOIN research.projects p ON pm.project_id = p.project_id WHERE p.title = 'AI in Education';")
    
    ⚙️ Step 10 - Skip printing trace entry:
    ----------------------------------------
    
    ⚙️ Step 11 - Orchestration:
    ----------------------------------------
    invocationInput: {'actionGroupInvocationInput': {'actionGroupName': 'execute-sql-query', 'executionType': 'LAMBDA', 'function': 'execute-sql', 'parameters': [{'name': 'query', 'type': 'string', 'value': "SELECT e.first_name, e.last_name, pm.role FROM staff.employees e JOIN research.project_members pm ON e.employee_id = pm.employee_id JOIN research.projects p ON pm.project_id = p.project_id WHERE p.title = 'AI in Education';"}]}, 'invocationType': 'ACTION_GROUP', 'traceId': '5018e885-ffa2-4a74-ac12-1635c58ceb88-1'}
    
    ⚙️ Step 12 - Orchestration:
    ----------------------------------------
    Observation: {'actionGroupInvocationOutput': {'text': '{"result": {"ResponseMetadata": {"RequestId": "2fca6c2c-c29d-4730-84d7-540e92ffe157", "HTTPStatusCode": 200, "HTTPHeaders": {"x-amzn-requestid": "2fca6c2c-c29d-4730-84d7-540e92ffe157", "date": "Wed, 29 Jan 2025 17:42:07 GMT", "content-type": "application/json", "content-length": "285", "connection": "keep-alive"}, "RetryAttempts": 0}, "records": [[{"stringValue": "Robert"}, {"stringValue": "Brown"}, {"stringValue": "Principal Investigator"}], [{"stringValue": "Emily"}, {"stringValue": "Davis"}, {"stringValue": "Co-Investigator"}]], "numberOfRecordsUpdated": 0}}'}, 'traceId': '5018e885-ffa2-4a74-ac12-1635c58ceb88-1', 'type': 'ACTION_GROUP'}
    
    ⚙️ Step 13 - Skip printing trace entry:
    ----------------------------------------
    
    ⚙️ Step 14 - Orchestration:
    ----------------------------------------
    Response: <answer>
    The members of the AI in Education project are:
    
    Robert Brown - Principal Investigator
    Emily Davis - Co-Investigator
    
    ⚙️ Step 15 - Orchestration:
    ----------------------------------------
    Observation: {'finalResponse': {'text': 'The members of the AI in Education project are:\n\nRobert Brown - Principal Investigator\nEmily Davis - Co-Investigator'}, 'traceId': '5018e885-ffa2-4a74-ac12-1635c58ceb88-2', 'type': 'FINISH'}
    
    ==================================================

Code

Here is another example showing how the agent responds for the input prompt to add data into the database. This solution only allows read operations (SELECT). From a security and data integrity perspective, we do not recommend implementing this solution for write operations. If you need your agent to support inserts and updates of the data, you should instead do this via an API that provides a layer of abstraction with the database. Moreover, you need to also implement validations and controls to make sure your data is consistent.

    BEDROCK AGENT TEST RESULTS
    ==================================================
    Single Test
    --------------------------------------------------
    Prompt: Can you add new student - 'Ryan', 'Nihilson', '2001-03-22', '2022-09-01', 2 
    Invoking Agent...
    Agent Response:
    Sorry, I don't have enough information to answer that.
    📋 Trace Steps:
    ================================================================================
    🔍 Step 1 - Skip printing trace entry:
    ----------------------------------------
    🔍 Step 2 - Pre-processing:
    ----------------------------------------
    Response:  <thinking>
    The given input is attempting to get the agent to execute an SQL query to add a new student to a database. However, the agent has not been provided with a function to add data, only to query data. Therefore, this input falls into Category C - questions that the agent will be unable to answer using only the functions it has been provided.
    </thinking>
    <category>C</category>
    

Code

Considerations and best practices
---------------------------------

When integrating Amazon Bedrock Agents with Aurora PostgreSQL and using generative AI capabilities for generating and executing SQL queries, several key considerations should be observed.

*   Enable this integration approach only for read-only workloads such as analytics and reporting where you need to provide flexible data querying access to your users using natural language. For read-write and transactional workloads, instead of generating the SQL query, you can use well-defined APIs to interface with the database.
*   Database schemas that undergo frequent changes in columns and data types can also benefit from this generative AI based integration approach that generates SQL queries on the fly based on the latest schema. You must make sure the changes to the schema are made visible to the query generation function.
*   Implement parameter validation in Amazon Bedrock Agents and the action group Lambda functions to prevent SQL injection and ensure data integrity. Refer to [Safeguard your generative AI workloads from prompt injections](https://aws.amazon.com/blogs/security/safeguard-your-generative-ai-workloads-from-prompt-injections/).
*   Implement caching strategies where appropriate to reduce database load for frequently requested information. For more details refer to: [Database Caching Strategies Using Redis](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/welcome.html).
*   Implement comprehensive logging and auditing to track interactions between Amazon Bedrock Agents and your database, promoting compliance and facilitating troubleshooting. Regularly monitor and analyze the generated query patterns to identify opportunities for performance tuning. For more details review the blog: [Improve visibility into Amazon Bedrock usage and performance with Amazon CloudWatch](https://aws.amazon.com/blogs/machine-learning/improve-visibility-into-amazon-bedrock-usage-and-performance-with-amazon-cloudwatch/).
*   If the application is multi-tenant, ensure you have the right isolation controls. For details on implementing row-level security with the Data API see [Enforce row-level security with the RDS Data API.](https://aws.amazon.com/blogs/database/enforce-row-level-security-with-the-rds-data-api/)
*   If you are looking for a managed implementation of text to SQL query generation functionality, then you can make use of the [GenerateQuery](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_GenerateQuery.html) API supported with the Bedrock Knowledge Base.

Clean up
--------

To avoid incurring future charges, delete all the resources created through CDK.

    cdk destroy --all

Code

Conclusion
----------

In this post, we demonstrated how to integrate Amazon Bedrock Agents and Aurora PostgreSQL using RDS Data API, enabling natural language interactions with your database. This solution showcases how AWS services can be combined to streamline database interactions through AI-driven interfaces, making data more accessible to nontechnical users. The integration pattern can be extended to support more complex use cases, such as automated reporting, natural language–based data exploration, and intelligent database monitoring.

We encourage you to try this solution in your environment and share your experiences. For additional support and resources, visit our [repository](https://github.com/aws-samples/sample-to-connect-bedrock-agent-with-aurora).

* * *

### About the authors

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2021/12/20/Nihilson-Gnanadason.jpg)**Nihilson Gnanadason** is a Senior Solutions Architect at Amazon Web Services (AWS). He works with ISVs in the UK to build, run, and scale their software products on AWS.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2025/05/20/skmohan.jpg)**Senthil Mohan** is a Solutions Architect at AWS working with EMEA customers, helping them migrate, modernize, and optimize their SaaS workloads for the AWS Cloud.

Like (4)(4)

Share

Comments
--------

Log in to commentLog in

* * *

[![](//d1.awsstatic.com/Digital%20Marketing/House/Editorial/other/SiteMerch-3066-Podcast_Editorial.65839609a8dda387937ed07dc8dc4f3c3b870546.png)

AWS Podcast

Subscribe for weekly AWS news and interviews

Learn more](https://aws.amazon.com/podcasts/aws-podcast/?sc_icampaign=aware_aws-podcast&sc_ichannel=ha&sc_icontent=awssm-2021&sc_iplace=blog_tile&trk=ha_awssm-2021) 

[![](//d1.awsstatic.com/webteam/homepage/editorials/Site-Merch_APN_Editorial.12df33fb7e0299389b086fb48dba7b9deeef07df.png)

AWS Partner Network

Find an APN member to support your cloud business needs

Learn more](https://aws.amazon.com/partners/find/?sc_icampaign=aware_apn_recruit&sc_ichannel=ha&sc_icontent=awssm-2021&sc_iplace=blog_tile&trk=ha_awssm-2021) 

[![](//d1.awsstatic.com/webteam/homepage/editorials/Site-Merch_Training_Editorial.5cc72ab0552ba66ef4e36a1a60ee742bc31113c7.png)

AWS Training & Certifications

Free digital courses to help you develop your skills

Learn more](https://aws.amazon.com/training/?sc_icampaign=aware_aws-training_blog&sc_ichannel=ha&sc_icontent=awssm-2021&sc_iplace=blog_tile&trk=ha_awssm-2021)