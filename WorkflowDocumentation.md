**The following content is AI generated and might contain incorrect information. Before you use this content, make sure to review for inaccuracies.**    
# Summary    
This Azure Logic Apps workflow is designed to trigger a GitHub Actions workflow dispatch event on a specific GitHub repository. When it receives an HTTP POST request with a Workflow ID, it performs the following primary steps:  
  
1. Retrieves a secret from Azure Key Vault to obtain credentials.  
2. Uses that secret to generate a GitHub installation token for authentication.  
3. Initiates a workflow dispatch event on the specified GitHub repository's workflow (main.yml), passing the Workflow ID and log level as inputs.  
4. Polls GitHub at intervals to check the status of the triggered workflow runs, filtering for the specific run it started based on the Workflow ID.  
5. Waits until the workflow run is completed or until a timeout limit is reached.  
6. Sends email notifications with the workflow run status and details to a specified email address.  
7. Finally, responds to the original HTTP request with the workflow run information.  
  
In summary, this Azure Logic Apps automates the triggering, monitoring, and reporting of a GitHub Actions workflow run, integrating secure credential management and email notifications into the process.    
||Workflow Properties|Value|    
|-----|-----|-----|    
|1|Connector type - In App|18|    
|2|Connector type - Shared|2|    
|3|Connector type - Custom|0|    
|4|API Connections|2|    
    
# Workflow Steps    
## How the workflow starts (Triggers)    
 - **GitHubRequest**    
&ensp;&ensp; - **Type:** In App    
&ensp;&ensp; - **Description:** This trigger initiates the workflow through an HTTP POST request. It expects a JSON payload containing a string property named "workFlow" which specifies the workflow to be triggered.    
&ensp;&ensp; - **Trigger Input:** The trigger requires a POST request with a JSON body that includes a "workFlow" string property, used to determine the targeted workflow to execute.    
&ensp;&ensp; - **Trigger Output:** This trigger does not produce any direct output; it only starts the workflow upon receiving the request.    
    
## How the workflow continues (Actions)    
 - **Submit_Workflow**    
&ensp;&ensp; - **Type:** In App    
&ensp;&ensp; - **Description:** This action sends an HTTP POST request to the GitHub API to dispatch a workflow run for the main branch of a specified repository. It uses an authorization token stored in a variable to authenticate and includes workflow inputs like log level and a call ID. It executes after the successful initialization of all variables.    
&ensp;&ensp; - **Action Input:** The action makes a POST request to GitHub's workflow dispatch API endpoint with headers specifying JSON acceptance, authorization via a token variable, and API version. The body contains the branch reference "main" and input parameters: "logLevel" set to "info" and a "call_id" referencing the workflow ID variable.    
&ensp;&ensp; - **Action Output:** This action does not generate any output; it only triggers the GitHub workflow dispatch.    
 - **Response**    
&ensp;&ensp; - **Type:** In App    
&ensp;&ensp; - **Description:** This action sends an HTTP response with status code 200 back to the caller. The response body is obtained from the output of the "GetRunInfo" action, providing details about the workflow run. It runs after "GetRunInfo" completes successfully.    
&ensp;&ensp; - **Action Input:** Sends a 200 HTTP status code with a response body populated by the output content from the "GetRunInfo" action.    
&ensp;&ensp; - **Action Output:** No additional output is produced by this response action itself.    
 - **Until**    
&ensp;&ensp; - **Type:** In App    
&ensp;&ensp; - **Description:** This is a loop action that repeatedly executes its nested actions until a specified condition is met. It starts after the "Submit_Workflow" action succeeds. It iteratively checks for the completion of the GitHub workflow run by polling the workflow runs and evaluating conditions inside the loop.    
&ensp;&ensp; - **Action Output:** This action does not directly produce outputs; it controls the execution flow based on the internal condition and actions.    
&ensp;&ensp; - Actions:    
&ensp;&ensp;&ensp;&ensp; - **Get_Workflow_Runs**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** In App    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** This action sends a GET request to GitHub API to retrieve a list of workflow runs triggered by the "workflow_dispatch" event on the specified workflow YAML file. It authenticates using an installation token and allows chunked content transfer. It runs after a delay action succeeds.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Input:** Makes a GET request to the GitHub API endpoint for workflow runs of the main.yml workflow triggered by manual dispatch events, with necessary authorization headers.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** This action produces a JSON response containing details about all recent workflow runs triggered by manual dispatch on the repository.    
&ensp;&ensp;&ensp;&ensp; - **Parse_JSON**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** In App    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Parses the JSON output from "Get_Workflow_Runs" to a structured object using a defined schema. This allows the workflow to reference individual run details more easily. It runs after the email action completes successfully.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Input:** Takes the raw JSON content of all workflow runs from "Get_Workflow_Runs" and parses it according to the specified schema describing the run properties and nested objects.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** Outputs a strongly typed object containing properties like total count and detailed array of workflow runs.    
&ensp;&ensp;&ensp;&ensp; - **FilterArray**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** In App    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Filters the array of workflow runs to find those with a name matching the pattern "deploy-{WorkflowId}" where "WorkflowId" is a variable. This narrows down the workflow runs relevant to the current execution. It runs after successful parsing of JSON.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Input:** Filters the "workflow_runs" array from the parsed JSON to items whose "name" property equals the concatenation of "deploy-" and the current workflow ID variable.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** Returns an array with workflow runs that match the specified naming pattern.    
&ensp;&ensp;&ensp;&ensp; - **Delay**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** In App    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Introduces a 15-second waiting period between attempts inside the Until loop, preventing excessive API polling and managing rate limits.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Input:** Specifies a wait interval of 15 seconds as the delay duration.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** Does not produce outputs; simply pauses the workflow execution temporarily.    
&ensp;&ensp;&ensp;&ensp; - **Condition**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** In App    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Evaluates whether the filtered workflow runs array is empty. If empty, it sets a variable indicating no item was found; otherwise, it proceeds to parse and process the first matched workflow run.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** Controls branching logic; no direct outputs produced.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - Actions:    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Set_variable**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** In App    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Sets a boolean variable "ItemFound" to false indicating no relevant workflow run was found.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Input:** Assigns the value false to the "ItemFound" variable.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** No outputs generated.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - Else:    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Parse_JSON_1**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** In App    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Parses the first item from the filtered workflow runs array, using a detailed schema matching the GitHub workflow run object for further processing.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Input:** Inputs the first array element from "FilterArray" and parses it against a defined schema containing all essential run properties.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** Outputs a structured object corresponding to a single workflow run's detailed information.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Set_variable_2**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** In App    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Stores the "id" of the parsed workflow run into the variable "WFRunId" for referencing in subsequent actions.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Input:** Sets variable "WFRunId" with the "id" property of the workflow run obtained from "Parse_JSON_1".    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** No direct output generated.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Condition_1**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** In App    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Further evaluates a condition after an email action named "Send_an_email_(V2)_1" succeeds, controlling additional variable setting.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** Manages flow control; no standalone outputs.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - Actions:    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Set_variable_1**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** In App    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Sets the "ItemFound" boolean variable to true indicating a workflow run matching criteria was found.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Input:** Assigns true to "ItemFound" variable.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** No output produced directly.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Send_an_email_(V2)_1**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** Shared    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Sends an email using the Office 365 connector to "azureadmin@nzazuredemo.com" with a subject "Test2". The email body contains the status of the parsed workflow run. It executes after parsing JSON of the filtered item.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Input:** Sends a POST request to the Office 365 Mail API with recipient email, subject "Test2", HTML body containing the workflow run status, and normal importance.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** No output returned; it performs the email sending operation.    
&ensp;&ensp;&ensp;&ensp; - **Send_an_email_(V2)**    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Type:** Shared    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Description:** Sends an email to "azureadmin@nzazuredemo.com" with subject "Test". The body contains the JSON output of the "Get_Workflow_Runs" action. It runs after "Get_Workflow_Runs" completes successfully.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Input:** Posts to the Office 365 "Mail" endpoint with recipient email, subject "Test", and body displaying the raw workflow runs JSON data with normal importance.    
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; - **Action Output:** No output is returned; this action sends an informational email.    
 - **InitializeAllVariables**    
&ensp;&ensp; - **Type:** In App    
&ensp;&ensp; - **Description:** Initializes variables needed for the workflow run including "WorkflowId" as a new GUID, "ItemFound" boolean as false, "WFRunId" integer as zero, and "InstallationToken" string assigned from the "GetGitHubInstallationToken" action output. It runs after successful token retrieval.    
&ensp;&ensp; - **Action Input:** Defines and assigns initial values to variables: a unique workflow GUID, an item found flag (false), a workflow run ID (0), and assigns the GitHub installation token from the previous token action output.    
&ensp;&ensp; - **Action Output:** Produces no direct output; variables are set for use in subsequent actions.    
 - **GetRunInfo**    
&ensp;&ensp; - **Type:** In App    
&ensp;&ensp; - **Description:** Performs a GET request to the GitHub API to fetch detailed information about a workflow run identified by the "WFRunId" variable. The request is authenticated with a bearer token stored in the "InstallationToken" variable. It runs after the "Until" loop completes successfully.    
&ensp;&ensp; - **Action Input:** Sends an HTTP GET request to the GitHub API endpoint for a single workflow run, using the run ID variable and authorization bearer token headers.    
&ensp;&ensp; - **Action Output:** Returns detailed information about the specified GitHub Actions workflow run in JSON format.    
 - **GetSecret**    
&ensp;&ensp; - **Type:** In App    
&ensp;&ensp; - **Description:** Retrieves a secret named "githubkey" from an Azure Key Vault using the configured key vault connector. This action securely fetches credentials or tokens required by the workflow. It runs independently without dependencies.    
&ensp;&ensp; - **Action Input:** Calls the Key Vault "getSecret" operation specifying the secret name "githubkey" with established connection details.    
&ensp;&ensp; - **Action Output:** Outputs the retrieved secret value securely for use in subsequent actions.    
 - **GetGitHubInstallationToken**    
&ensp;&ensp; - **Type:** In App    
&ensp;&ensp; - **Description:** Runs a C# script (code file "execute_csharp_script_code.csx") to generate or retrieve a GitHub installation token needed for authenticating API requests. This action runs after "GetSecret" action succeeds.    
&ensp;&ensp; - **Action Input:** Executes the specified C# script file to process the secret and produce an installation token.    
&ensp;&ensp; - **Action Output:** Produces an object containing the GitHub installation token message used for API authentication.    
.