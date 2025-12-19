# Solution Breakdown in High Level Steps 

## 1. Create a GitHub App 

## 2. Store the GitHub App’s Private Key in an Azure Key Vault

## 3. Create a Logic App (Standard) and a Workflow and use MSI to get the Private Key (See this Link)

## 4. Use C#/PowerShell Inline Code Activity to create the Installation Token from Private Key (See this Link)

## 5. Get/Create the name for your workflow to be tracked
 
## 6. Use the above token to call GitHub APIs to dispatch the workflow and pass the name as parameter (See this Link and See this Link)

## 7. Use Logic Apps Actions and GitHub APIs to track the workflow’s progress

# Links to Solution Components and Details 

## Use [This Link](https://github.com/naveedzaheer/ghwfautomation/blob/main/WorkflowDocumentation.md) for a detailed documentation of the Workflow
## Use [This Link](https://github.com/naveedzaheer/ghwfautomation/blob/main/WorkflowCode.md) for the Wokflow Code
## Use [This Link](https://github.com/naveedzaheer/ghwfautomation/blob/main/TokenCode.md) for the C# code to generate GitHub App Installation Access Token
