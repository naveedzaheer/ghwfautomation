# Solution Breakdown in High Level Steps 

## Create a GitHub App 

## Store the GitHub App’s Private Key in an Azure Key Vault

## Create a Logic App (Standard) and a Workflow and use MSI to get the Private Key (See this Link)

## Use C#/PowerShell Inline Code Activity to create the Installation Token from Private Key (See this Link)

## Get/Create the name for your workflow to be tracked
 
## Use the above token to call GitHub APIs to dispatch the workflow and pass the name as parameter (See this Link and See this Link)

## Use Logic Apps Actions and GitHub APIs to track the workflow’s progress


## Use [This Link](https://github.com/naveedzaheer/ghwfautomation/blob/main/WorkflowDocumentation.md) for a detailed documentation of the Workflow
## Use [This Link](https://github.com/naveedzaheer/ghwfautomation/blob/main/WorkflowCode.md) for the Wokflow Code
## Use [This Link](https://github.com/naveedzaheer/ghwfautomation/blob/main/TokenCode.md) for the C# code to generate GitHub App Installation Access Token
