# Workflow Code
```
{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "contentVersion": "1.0.0.0",
        "triggers": {
            "GitHubRequest": {
                "type": "Request",
                "kind": "Http",
                "inputs": {
                    "method": "POST",
                    "schema": {
                        "type": "object",
                        "properties": {
                            "workFlow": {
                                "type": "string"
                            }
                        }
                    }
                }
            }
        },
        "actions": {
            "Submit_Workflow": {
                "type": "Http",
                "inputs": {
                    "uri": "https://api.github.com/repos/OWNER/REPO/actions/workflows/main.yml/dispatches",
                    "method": "POST",
                    "headers": {
                        "Accept": "application/vnd.github+json",
                        "Authorization": "Token @{variables('InstallationToken')}",
                        "X-GitHub-Api-Version": "2022-11-28"
                    },
                    "body": {
                        "ref": "main",
                        "inputs": {
                            "logLevel": "info",
                            "call_id": "@{variables('WorkflowId')}"
                        }
                    }
                },
                "runAfter": {
                    "InitializeAllVariables": [
                        "Succeeded"
                    ]
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Response": {
                "type": "Response",
                "kind": "Http",
                "inputs": {
                    "statusCode": 200,
                    "body": "@body('GetRunInfo')"
                },
                "runAfter": {
                    "GetRunInfo": [
                        "Succeeded"
                    ]
                }
            },
            "Until": {
                "type": "Until",
                "expression": "@equals(variables('ItemFound'), true)",
                "limit": {
                    "count": 60,
                    "timeout": "PT1H"
                },
                "actions": {
                    "Get_Workflow_Runs": {
                        "type": "Http",
                        "inputs": {
                            "uri": "https://api.github.com/repos/OWNER/REPO/actions/workflows/main.yml/runs?event=workflow_dispatch",
                            "method": "GET",
                            "headers": {
                                "Accept": "application/vnd.github+json",
                                "Authorization": "Token @{variables('InstallationToken')}",
                                "X-GitHub-Api-Version": "2022-11-28"
                            }
                        },
                        "runAfter": {
                            "Delay": [
                                "Succeeded"
                            ]
                        },
                        "runtimeConfiguration": {
                            "contentTransfer": {
                                "transferMode": "Chunked"
                            }
                        }
                    },
                    "Parse_JSON": {
                        "type": "ParseJson",
                        "inputs": {
                            "content": "@body('Get_Workflow_Runs')",
                            "schema": {
                                "type": "object",
                                "properties": {
                                    "total_count": {
                                        "type": "integer"
                                    },
                                    "workflow_runs": {
                                        "type": "array",
                                        "items": {
                                            "type": "object",
                                            "properties": {
                                                "id": {
                                                    "type": "integer"
                                                },
                                                "name": {
                                                    "type": "string"
                                                },
                                                "node_id": {
                                                    "type": "string"
                                                },
                                                "head_branch": {
                                                    "type": "string"
                                                },
                                                "head_sha": {
                                                    "type": "string"
                                                },
                                                "path": {
                                                    "type": "string"
                                                },
                                                "display_title": {
                                                    "type": "string"
                                                },
                                                "run_number": {
                                                    "type": "integer"
                                                },
                                                "event": {
                                                    "type": "string"
                                                },
                                                "status": {
                                                    "type": "string"
                                                },
                                                "conclusion": {},
                                                "workflow_id": {
                                                    "type": "integer"
                                                },
                                                "check_suite_id": {
                                                    "type": "integer"
                                                },
                                                "check_suite_node_id": {
                                                    "type": "string"
                                                },
                                                "url": {
                                                    "type": "string"
                                                },
                                                "html_url": {
                                                    "type": "string"
                                                },
                                                "pull_requests": {
                                                    "type": "array"
                                                },
                                                "created_at": {
                                                    "type": "string"
                                                },
                                                "updated_at": {
                                                    "type": "string"
                                                },
                                                "actor": {
                                                    "type": "object",
                                                    "properties": {
                                                        "login": {
                                                            "type": "string"
                                                        },
                                                        "id": {
                                                            "type": "integer"
                                                        },
                                                        "node_id": {
                                                            "type": "string"
                                                        },
                                                        "avatar_url": {
                                                            "type": "string"
                                                        },
                                                        "gravatar_id": {
                                                            "type": "string"
                                                        },
                                                        "url": {
                                                            "type": "string"
                                                        },
                                                        "html_url": {
                                                            "type": "string"
                                                        },
                                                        "followers_url": {
                                                            "type": "string"
                                                        },
                                                        "following_url": {
                                                            "type": "string"
                                                        },
                                                        "gists_url": {
                                                            "type": "string"
                                                        },
                                                        "starred_url": {
                                                            "type": "string"
                                                        },
                                                        "subscriptions_url": {
                                                            "type": "string"
                                                        },
                                                        "organizations_url": {
                                                            "type": "string"
                                                        },
                                                        "repos_url": {
                                                            "type": "string"
                                                        },
                                                        "events_url": {
                                                            "type": "string"
                                                        },
                                                        "received_events_url": {
                                                            "type": "string"
                                                        },
                                                        "type": {
                                                            "type": "string"
                                                        },
                                                        "user_view_type": {
                                                            "type": "string"
                                                        },
                                                        "site_admin": {
                                                            "type": "boolean"
                                                        }
                                                    }
                                                },
                                                "run_attempt": {
                                                    "type": "integer"
                                                },
                                                "referenced_workflows": {
                                                    "type": "array"
                                                },
                                                "run_started_at": {
                                                    "type": "string"
                                                },
                                                "triggering_actor": {
                                                    "type": "object",
                                                    "properties": {
                                                        "login": {
                                                            "type": "string"
                                                        },
                                                        "id": {
                                                            "type": "integer"
                                                        },
                                                        "node_id": {
                                                            "type": "string"
                                                        },
                                                        "avatar_url": {
                                                            "type": "string"
                                                        },
                                                        "gravatar_id": {
                                                            "type": "string"
                                                        },
                                                        "url": {
                                                            "type": "string"
                                                        },
                                                        "html_url": {
                                                            "type": "string"
                                                        },
                                                        "followers_url": {
                                                            "type": "string"
                                                        },
                                                        "following_url": {
                                                            "type": "string"
                                                        },
                                                        "gists_url": {
                                                            "type": "string"
                                                        },
                                                        "starred_url": {
                                                            "type": "string"
                                                        },
                                                        "subscriptions_url": {
                                                            "type": "string"
                                                        },
                                                        "organizations_url": {
                                                            "type": "string"
                                                        },
                                                        "repos_url": {
                                                            "type": "string"
                                                        },
                                                        "events_url": {
                                                            "type": "string"
                                                        },
                                                        "received_events_url": {
                                                            "type": "string"
                                                        },
                                                        "type": {
                                                            "type": "string"
                                                        },
                                                        "user_view_type": {
                                                            "type": "string"
                                                        },
                                                        "site_admin": {
                                                            "type": "boolean"
                                                        }
                                                    }
                                                },
                                                "jobs_url": {
                                                    "type": "string"
                                                },
                                                "logs_url": {
                                                    "type": "string"
                                                },
                                                "check_suite_url": {
                                                    "type": "string"
                                                },
                                                "artifacts_url": {
                                                    "type": "string"
                                                },
                                                "cancel_url": {
                                                    "type": "string"
                                                },
                                                "rerun_url": {
                                                    "type": "string"
                                                },
                                                "previous_attempt_url": {},
                                                "workflow_url": {
                                                    "type": "string"
                                                },
                                                "head_commit": {
                                                    "type": "object",
                                                    "properties": {
                                                        "id": {
                                                            "type": "string"
                                                        },
                                                        "tree_id": {
                                                            "type": "string"
                                                        },
                                                        "message": {
                                                            "type": "string"
                                                        },
                                                        "timestamp": {
                                                            "type": "string"
                                                        },
                                                        "author": {
                                                            "type": "object",
                                                            "properties": {
                                                                "name": {
                                                                    "type": "string"
                                                                },
                                                                "email": {
                                                                    "type": "string"
                                                                }
                                                            }
                                                        },
                                                        "committer": {
                                                            "type": "object",
                                                            "properties": {
                                                                "name": {
                                                                    "type": "string"
                                                                },
                                                                "email": {
                                                                    "type": "string"
                                                                }
                                                            }
                                                        }
                                                    }
                                                },
                                                "repository": {
                                                    "type": "object",
                                                    "properties": {
                                                        "id": {
                                                            "type": "integer"
                                                        },
                                                        "node_id": {
                                                            "type": "string"
                                                        },
                                                        "name": {
                                                            "type": "string"
                                                        },
                                                        "full_name": {
                                                            "type": "string"
                                                        },
                                                        "private": {
                                                            "type": "boolean"
                                                        },
                                                        "owner": {
                                                            "type": "object",
                                                            "properties": {
                                                                "login": {
                                                                    "type": "string"
                                                                },
                                                                "id": {
                                                                    "type": "integer"
                                                                },
                                                                "node_id": {
                                                                    "type": "string"
                                                                },
                                                                "avatar_url": {
                                                                    "type": "string"
                                                                },
                                                                "gravatar_id": {
                                                                    "type": "string"
                                                                },
                                                                "url": {
                                                                    "type": "string"
                                                                },
                                                                "html_url": {
                                                                    "type": "string"
                                                                },
                                                                "followers_url": {
                                                                    "type": "string"
                                                                },
                                                                "following_url": {
                                                                    "type": "string"
                                                                },
                                                                "gists_url": {
                                                                    "type": "string"
                                                                },
                                                                "starred_url": {
                                                                    "type": "string"
                                                                },
                                                                "subscriptions_url": {
                                                                    "type": "string"
                                                                },
                                                                "organizations_url": {
                                                                    "type": "string"
                                                                },
                                                                "repos_url": {
                                                                    "type": "string"
                                                                },
                                                                "events_url": {
                                                                    "type": "string"
                                                                },
                                                                "received_events_url": {
                                                                    "type": "string"
                                                                },
                                                                "type": {
                                                                    "type": "string"
                                                                },
                                                                "user_view_type": {
                                                                    "type": "string"
                                                                },
                                                                "site_admin": {
                                                                    "type": "boolean"
                                                                }
                                                            }
                                                        },
                                                        "html_url": {
                                                            "type": "string"
                                                        },
                                                        "description": {},
                                                        "fork": {
                                                            "type": "boolean"
                                                        },
                                                        "url": {
                                                            "type": "string"
                                                        },
                                                        "forks_url": {
                                                            "type": "string"
                                                        },
                                                        "keys_url": {
                                                            "type": "string"
                                                        },
                                                        "collaborators_url": {
                                                            "type": "string"
                                                        },
                                                        "teams_url": {
                                                            "type": "string"
                                                        },
                                                        "hooks_url": {
                                                            "type": "string"
                                                        },
                                                        "issue_events_url": {
                                                            "type": "string"
                                                        },
                                                        "events_url": {
                                                            "type": "string"
                                                        },
                                                        "assignees_url": {
                                                            "type": "string"
                                                        },
                                                        "branches_url": {
                                                            "type": "string"
                                                        },
                                                        "tags_url": {
                                                            "type": "string"
                                                        },
                                                        "blobs_url": {
                                                            "type": "string"
                                                        },
                                                        "git_tags_url": {
                                                            "type": "string"
                                                        },
                                                        "git_refs_url": {
                                                            "type": "string"
                                                        },
                                                        "trees_url": {
                                                            "type": "string"
                                                        },
                                                        "statuses_url": {
                                                            "type": "string"
                                                        },
                                                        "languages_url": {
                                                            "type": "string"
                                                        },
                                                        "stargazers_url": {
                                                            "type": "string"
                                                        },
                                                        "contributors_url": {
                                                            "type": "string"
                                                        },
                                                        "subscribers_url": {
                                                            "type": "string"
                                                        },
                                                        "subscription_url": {
                                                            "type": "string"
                                                        },
                                                        "commits_url": {
                                                            "type": "string"
                                                        },
                                                        "git_commits_url": {
                                                            "type": "string"
                                                        },
                                                        "comments_url": {
                                                            "type": "string"
                                                        },
                                                        "issue_comment_url": {
                                                            "type": "string"
                                                        },
                                                        "contents_url": {
                                                            "type": "string"
                                                        },
                                                        "compare_url": {
                                                            "type": "string"
                                                        },
                                                        "merges_url": {
                                                            "type": "string"
                                                        },
                                                        "archive_url": {
                                                            "type": "string"
                                                        },
                                                        "downloads_url": {
                                                            "type": "string"
                                                        },
                                                        "issues_url": {
                                                            "type": "string"
                                                        },
                                                        "pulls_url": {
                                                            "type": "string"
                                                        },
                                                        "milestones_url": {
                                                            "type": "string"
                                                        },
                                                        "notifications_url": {
                                                            "type": "string"
                                                        },
                                                        "labels_url": {
                                                            "type": "string"
                                                        },
                                                        "releases_url": {
                                                            "type": "string"
                                                        },
                                                        "deployments_url": {
                                                            "type": "string"
                                                        }
                                                    }
                                                },
                                                "head_repository": {
                                                    "type": "object",
                                                    "properties": {
                                                        "id": {
                                                            "type": "integer"
                                                        },
                                                        "node_id": {
                                                            "type": "string"
                                                        },
                                                        "name": {
                                                            "type": "string"
                                                        },
                                                        "full_name": {
                                                            "type": "string"
                                                        },
                                                        "private": {
                                                            "type": "boolean"
                                                        },
                                                        "owner": {
                                                            "type": "object",
                                                            "properties": {
                                                                "login": {
                                                                    "type": "string"
                                                                },
                                                                "id": {
                                                                    "type": "integer"
                                                                },
                                                                "node_id": {
                                                                    "type": "string"
                                                                },
                                                                "avatar_url": {
                                                                    "type": "string"
                                                                },
                                                                "gravatar_id": {
                                                                    "type": "string"
                                                                },
                                                                "url": {
                                                                    "type": "string"
                                                                },
                                                                "html_url": {
                                                                    "type": "string"
                                                                },
                                                                "followers_url": {
                                                                    "type": "string"
                                                                },
                                                                "following_url": {
                                                                    "type": "string"
                                                                },
                                                                "gists_url": {
                                                                    "type": "string"
                                                                },
                                                                "starred_url": {
                                                                    "type": "string"
                                                                },
                                                                "subscriptions_url": {
                                                                    "type": "string"
                                                                },
                                                                "organizations_url": {
                                                                    "type": "string"
                                                                },
                                                                "repos_url": {
                                                                    "type": "string"
                                                                },
                                                                "events_url": {
                                                                    "type": "string"
                                                                },
                                                                "received_events_url": {
                                                                    "type": "string"
                                                                },
                                                                "type": {
                                                                    "type": "string"
                                                                },
                                                                "user_view_type": {
                                                                    "type": "string"
                                                                },
                                                                "site_admin": {
                                                                    "type": "boolean"
                                                                }
                                                            }
                                                        },
                                                        "html_url": {
                                                            "type": "string"
                                                        },
                                                        "description": {},
                                                        "fork": {
                                                            "type": "boolean"
                                                        },
                                                        "url": {
                                                            "type": "string"
                                                        },
                                                        "forks_url": {
                                                            "type": "string"
                                                        },
                                                        "keys_url": {
                                                            "type": "string"
                                                        },
                                                        "collaborators_url": {
                                                            "type": "string"
                                                        },
                                                        "teams_url": {
                                                            "type": "string"
                                                        },
                                                        "hooks_url": {
                                                            "type": "string"
                                                        },
                                                        "issue_events_url": {
                                                            "type": "string"
                                                        },
                                                        "events_url": {
                                                            "type": "string"
                                                        },
                                                        "assignees_url": {
                                                            "type": "string"
                                                        },
                                                        "branches_url": {
                                                            "type": "string"
                                                        },
                                                        "tags_url": {
                                                            "type": "string"
                                                        },
                                                        "blobs_url": {
                                                            "type": "string"
                                                        },
                                                        "git_tags_url": {
                                                            "type": "string"
                                                        },
                                                        "git_refs_url": {
                                                            "type": "string"
                                                        },
                                                        "trees_url": {
                                                            "type": "string"
                                                        },
                                                        "statuses_url": {
                                                            "type": "string"
                                                        },
                                                        "languages_url": {
                                                            "type": "string"
                                                        },
                                                        "stargazers_url": {
                                                            "type": "string"
                                                        },
                                                        "contributors_url": {
                                                            "type": "string"
                                                        },
                                                        "subscribers_url": {
                                                            "type": "string"
                                                        },
                                                        "subscription_url": {
                                                            "type": "string"
                                                        },
                                                        "commits_url": {
                                                            "type": "string"
                                                        },
                                                        "git_commits_url": {
                                                            "type": "string"
                                                        },
                                                        "comments_url": {
                                                            "type": "string"
                                                        },
                                                        "issue_comment_url": {
                                                            "type": "string"
                                                        },
                                                        "contents_url": {
                                                            "type": "string"
                                                        },
                                                        "compare_url": {
                                                            "type": "string"
                                                        },
                                                        "merges_url": {
                                                            "type": "string"
                                                        },
                                                        "archive_url": {
                                                            "type": "string"
                                                        },
                                                        "downloads_url": {
                                                            "type": "string"
                                                        },
                                                        "issues_url": {
                                                            "type": "string"
                                                        },
                                                        "pulls_url": {
                                                            "type": "string"
                                                        },
                                                        "milestones_url": {
                                                            "type": "string"
                                                        },
                                                        "notifications_url": {
                                                            "type": "string"
                                                        },
                                                        "labels_url": {
                                                            "type": "string"
                                                        },
                                                        "releases_url": {
                                                            "type": "string"
                                                        },
                                                        "deployments_url": {
                                                            "type": "string"
                                                        }
                                                    }
                                                }
                                            },
                                            "required": [
                                                "id",
                                                "name",
                                                "node_id",
                                                "head_branch",
                                                "head_sha",
                                                "path",
                                                "display_title",
                                                "run_number",
                                                "event",
                                                "status",
                                                "conclusion",
                                                "workflow_id",
                                                "check_suite_id",
                                                "check_suite_node_id",
                                                "url",
                                                "html_url",
                                                "pull_requests",
                                                "created_at",
                                                "updated_at",
                                                "actor",
                                                "run_attempt",
                                                "referenced_workflows",
                                                "run_started_at",
                                                "triggering_actor",
                                                "jobs_url",
                                                "logs_url",
                                                "check_suite_url",
                                                "artifacts_url",
                                                "cancel_url",
                                                "rerun_url",
                                                "previous_attempt_url",
                                                "workflow_url",
                                                "head_commit",
                                                "repository",
                                                "head_repository"
                                            ]
                                        }
                                    }
                                }
                            }
                        },
                        "runAfter": {
                            "Send_an_email_(V2)": [
                                "Succeeded"
                            ]
                        }
                    },
                    "FilterArray": {
                        "type": "Query",
                        "inputs": {
                            "from": "@body('Parse_JSON')?['workflow_runs']",
                            "where": "@equals(item()?['name'], concat('deploy-',variables('WorkflowId')))"
                        },
                        "runAfter": {
                            "Parse_JSON": [
                                "Succeeded"
                            ]
                        }
                    },
                    "Delay": {
                        "type": "Wait",
                        "inputs": {
                            "interval": {
                                "count": 15,
                                "unit": "Second"
                            }
                        }
                    },
                    "Condition": {
                        "type": "If",
                        "expression": {
                            "and": [
                                {
                                    "equals": [
                                        "@length(body('FilterArray'))",
                                        0
                                    ]
                                }
                            ]
                        },
                        "actions": {
                            "Set_variable": {
                                "type": "SetVariable",
                                "inputs": {
                                    "name": "ItemFound",
                                    "value": false
                                }
                            }
                        },
                        "else": {
                            "actions": {
                                "Parse_JSON_1": {
                                    "type": "ParseJson",
                                    "inputs": {
                                        "content": "@first(body('FilterArray'))",
                                        "schema": {
                                            "type": "object",
                                            "properties": {
                                                "id": {
                                                    "type": "integer"
                                                },
                                                "name": {
                                                    "type": "string"
                                                },
                                                "node_id": {
                                                    "type": "string"
                                                },
                                                "head_branch": {
                                                    "type": "string"
                                                },
                                                "head_sha": {
                                                    "type": "string"
                                                },
                                                "path": {
                                                    "type": "string"
                                                },
                                                "display_title": {
                                                    "type": "string"
                                                },
                                                "run_number": {
                                                    "type": "integer"
                                                },
                                                "event": {
                                                    "type": "string"
                                                },
                                                "status": {
                                                    "type": "string"
                                                },
                                                "conclusion": {},
                                                "workflow_id": {
                                                    "type": "integer"
                                                },
                                                "check_suite_id": {
                                                    "type": "integer"
                                                },
                                                "check_suite_node_id": {
                                                    "type": "string"
                                                },
                                                "url": {
                                                    "type": "string"
                                                },
                                                "html_url": {
                                                    "type": "string"
                                                },
                                                "pull_requests": {
                                                    "type": "array"
                                                },
                                                "created_at": {
                                                    "type": "string"
                                                },
                                                "updated_at": {
                                                    "type": "string"
                                                },
                                                "actor": {
                                                    "type": "object",
                                                    "properties": {
                                                        "login": {
                                                            "type": "string"
                                                        },
                                                        "id": {
                                                            "type": "integer"
                                                        },
                                                        "node_id": {
                                                            "type": "string"
                                                        },
                                                        "avatar_url": {
                                                            "type": "string"
                                                        },
                                                        "gravatar_id": {
                                                            "type": "string"
                                                        },
                                                        "url": {
                                                            "type": "string"
                                                        },
                                                        "html_url": {
                                                            "type": "string"
                                                        },
                                                        "followers_url": {
                                                            "type": "string"
                                                        },
                                                        "following_url": {
                                                            "type": "string"
                                                        },
                                                        "gists_url": {
                                                            "type": "string"
                                                        },
                                                        "starred_url": {
                                                            "type": "string"
                                                        },
                                                        "subscriptions_url": {
                                                            "type": "string"
                                                        },
                                                        "organizations_url": {
                                                            "type": "string"
                                                        },
                                                        "repos_url": {
                                                            "type": "string"
                                                        },
                                                        "events_url": {
                                                            "type": "string"
                                                        },
                                                        "received_events_url": {
                                                            "type": "string"
                                                        },
                                                        "type": {
                                                            "type": "string"
                                                        },
                                                        "user_view_type": {
                                                            "type": "string"
                                                        },
                                                        "site_admin": {
                                                            "type": "boolean"
                                                        }
                                                    }
                                                },
                                                "run_attempt": {
                                                    "type": "integer"
                                                },
                                                "referenced_workflows": {
                                                    "type": "array"
                                                },
                                                "run_started_at": {
                                                    "type": "string"
                                                },
                                                "triggering_actor": {
                                                    "type": "object",
                                                    "properties": {
                                                        "login": {
                                                            "type": "string"
                                                        },
                                                        "id": {
                                                            "type": "integer"
                                                        },
                                                        "node_id": {
                                                            "type": "string"
                                                        },
                                                        "avatar_url": {
                                                            "type": "string"
                                                        },
                                                        "gravatar_id": {
                                                            "type": "string"
                                                        },
                                                        "url": {
                                                            "type": "string"
                                                        },
                                                        "html_url": {
                                                            "type": "string"
                                                        },
                                                        "followers_url": {
                                                            "type": "string"
                                                        },
                                                        "following_url": {
                                                            "type": "string"
                                                        },
                                                        "gists_url": {
                                                            "type": "string"
                                                        },
                                                        "starred_url": {
                                                            "type": "string"
                                                        },
                                                        "subscriptions_url": {
                                                            "type": "string"
                                                        },
                                                        "organizations_url": {
                                                            "type": "string"
                                                        },
                                                        "repos_url": {
                                                            "type": "string"
                                                        },
                                                        "events_url": {
                                                            "type": "string"
                                                        },
                                                        "received_events_url": {
                                                            "type": "string"
                                                        },
                                                        "type": {
                                                            "type": "string"
                                                        },
                                                        "user_view_type": {
                                                            "type": "string"
                                                        },
                                                        "site_admin": {
                                                            "type": "boolean"
                                                        }
                                                    }
                                                },
                                                "jobs_url": {
                                                    "type": "string"
                                                },
                                                "logs_url": {
                                                    "type": "string"
                                                },
                                                "check_suite_url": {
                                                    "type": "string"
                                                },
                                                "artifacts_url": {
                                                    "type": "string"
                                                },
                                                "cancel_url": {
                                                    "type": "string"
                                                },
                                                "rerun_url": {
                                                    "type": "string"
                                                },
                                                "previous_attempt_url": {},
                                                "workflow_url": {
                                                    "type": "string"
                                                },
                                                "head_commit": {
                                                    "type": "object",
                                                    "properties": {
                                                        "id": {
                                                            "type": "string"
                                                        },
                                                        "tree_id": {
                                                            "type": "string"
                                                        },
                                                        "message": {
                                                            "type": "string"
                                                        },
                                                        "timestamp": {
                                                            "type": "string"
                                                        },
                                                        "author": {
                                                            "type": "object",
                                                            "properties": {
                                                                "name": {
                                                                    "type": "string"
                                                                },
                                                                "email": {
                                                                    "type": "string"
                                                                }
                                                            }
                                                        },
                                                        "committer": {
                                                            "type": "object",
                                                            "properties": {
                                                                "name": {
                                                                    "type": "string"
                                                                },
                                                                "email": {
                                                                    "type": "string"
                                                                }
                                                            }
                                                        }
                                                    }
                                                },
                                                "repository": {
                                                    "type": "object",
                                                    "properties": {
                                                        "id": {
                                                            "type": "integer"
                                                        },
                                                        "node_id": {
                                                            "type": "string"
                                                        },
                                                        "name": {
                                                            "type": "string"
                                                        },
                                                        "full_name": {
                                                            "type": "string"
                                                        },
                                                        "private": {
                                                            "type": "boolean"
                                                        },
                                                        "owner": {
                                                            "type": "object",
                                                            "properties": {
                                                                "login": {
                                                                    "type": "string"
                                                                },
                                                                "id": {
                                                                    "type": "integer"
                                                                },
                                                                "node_id": {
                                                                    "type": "string"
                                                                },
                                                                "avatar_url": {
                                                                    "type": "string"
                                                                },
                                                                "gravatar_id": {
                                                                    "type": "string"
                                                                },
                                                                "url": {
                                                                    "type": "string"
                                                                },
                                                                "html_url": {
                                                                    "type": "string"
                                                                },
                                                                "followers_url": {
                                                                    "type": "string"
                                                                },
                                                                "following_url": {
                                                                    "type": "string"
                                                                },
                                                                "gists_url": {
                                                                    "type": "string"
                                                                },
                                                                "starred_url": {
                                                                    "type": "string"
                                                                },
                                                                "subscriptions_url": {
                                                                    "type": "string"
                                                                },
                                                                "organizations_url": {
                                                                    "type": "string"
                                                                },
                                                                "repos_url": {
                                                                    "type": "string"
                                                                },
                                                                "events_url": {
                                                                    "type": "string"
                                                                },
                                                                "received_events_url": {
                                                                    "type": "string"
                                                                },
                                                                "type": {
                                                                    "type": "string"
                                                                },
                                                                "user_view_type": {
                                                                    "type": "string"
                                                                },
                                                                "site_admin": {
                                                                    "type": "boolean"
                                                                }
                                                            }
                                                        },
                                                        "html_url": {
                                                            "type": "string"
                                                        },
                                                        "description": {},
                                                        "fork": {
                                                            "type": "boolean"
                                                        },
                                                        "url": {
                                                            "type": "string"
                                                        },
                                                        "forks_url": {
                                                            "type": "string"
                                                        },
                                                        "keys_url": {
                                                            "type": "string"
                                                        },
                                                        "collaborators_url": {
                                                            "type": "string"
                                                        },
                                                        "teams_url": {
                                                            "type": "string"
                                                        },
                                                        "hooks_url": {
                                                            "type": "string"
                                                        },
                                                        "issue_events_url": {
                                                            "type": "string"
                                                        },
                                                        "events_url": {
                                                            "type": "string"
                                                        },
                                                        "assignees_url": {
                                                            "type": "string"
                                                        },
                                                        "branches_url": {
                                                            "type": "string"
                                                        },
                                                        "tags_url": {
                                                            "type": "string"
                                                        },
                                                        "blobs_url": {
                                                            "type": "string"
                                                        },
                                                        "git_tags_url": {
                                                            "type": "string"
                                                        },
                                                        "git_refs_url": {
                                                            "type": "string"
                                                        },
                                                        "trees_url": {
                                                            "type": "string"
                                                        },
                                                        "statuses_url": {
                                                            "type": "string"
                                                        },
                                                        "languages_url": {
                                                            "type": "string"
                                                        },
                                                        "stargazers_url": {
                                                            "type": "string"
                                                        },
                                                        "contributors_url": {
                                                            "type": "string"
                                                        },
                                                        "subscribers_url": {
                                                            "type": "string"
                                                        },
                                                        "subscription_url": {
                                                            "type": "string"
                                                        },
                                                        "commits_url": {
                                                            "type": "string"
                                                        },
                                                        "git_commits_url": {
                                                            "type": "string"
                                                        },
                                                        "comments_url": {
                                                            "type": "string"
                                                        },
                                                        "issue_comment_url": {
                                                            "type": "string"
                                                        },
                                                        "contents_url": {
                                                            "type": "string"
                                                        },
                                                        "compare_url": {
                                                            "type": "string"
                                                        },
                                                        "merges_url": {
                                                            "type": "string"
                                                        },
                                                        "archive_url": {
                                                            "type": "string"
                                                        },
                                                        "downloads_url": {
                                                            "type": "string"
                                                        },
                                                        "issues_url": {
                                                            "type": "string"
                                                        },
                                                        "pulls_url": {
                                                            "type": "string"
                                                        },
                                                        "milestones_url": {
                                                            "type": "string"
                                                        },
                                                        "notifications_url": {
                                                            "type": "string"
                                                        },
                                                        "labels_url": {
                                                            "type": "string"
                                                        },
                                                        "releases_url": {
                                                            "type": "string"
                                                        },
                                                        "deployments_url": {
                                                            "type": "string"
                                                        }
                                                    }
                                                },
                                                "head_repository": {
                                                    "type": "object",
                                                    "properties": {
                                                        "id": {
                                                            "type": "integer"
                                                        },
                                                        "node_id": {
                                                            "type": "string"
                                                        },
                                                        "name": {
                                                            "type": "string"
                                                        },
                                                        "full_name": {
                                                            "type": "string"
                                                        },
                                                        "private": {
                                                            "type": "boolean"
                                                        },
                                                        "owner": {
                                                            "type": "object",
                                                            "properties": {
                                                                "login": {
                                                                    "type": "string"
                                                                },
                                                                "id": {
                                                                    "type": "integer"
                                                                },
                                                                "node_id": {
                                                                    "type": "string"
                                                                },
                                                                "avatar_url": {
                                                                    "type": "string"
                                                                },
                                                                "gravatar_id": {
                                                                    "type": "string"
                                                                },
                                                                "url": {
                                                                    "type": "string"
                                                                },
                                                                "html_url": {
                                                                    "type": "string"
                                                                },
                                                                "followers_url": {
                                                                    "type": "string"
                                                                },
                                                                "following_url": {
                                                                    "type": "string"
                                                                },
                                                                "gists_url": {
                                                                    "type": "string"
                                                                },
                                                                "starred_url": {
                                                                    "type": "string"
                                                                },
                                                                "subscriptions_url": {
                                                                    "type": "string"
                                                                },
                                                                "organizations_url": {
                                                                    "type": "string"
                                                                },
                                                                "repos_url": {
                                                                    "type": "string"
                                                                },
                                                                "events_url": {
                                                                    "type": "string"
                                                                },
                                                                "received_events_url": {
                                                                    "type": "string"
                                                                },
                                                                "type": {
                                                                    "type": "string"
                                                                },
                                                                "user_view_type": {
                                                                    "type": "string"
                                                                },
                                                                "site_admin": {
                                                                    "type": "boolean"
                                                                }
                                                            }
                                                        },
                                                        "html_url": {
                                                            "type": "string"
                                                        },
                                                        "description": {},
                                                        "fork": {
                                                            "type": "boolean"
                                                        },
                                                        "url": {
                                                            "type": "string"
                                                        },
                                                        "forks_url": {
                                                            "type": "string"
                                                        },
                                                        "keys_url": {
                                                            "type": "string"
                                                        },
                                                        "collaborators_url": {
                                                            "type": "string"
                                                        },
                                                        "teams_url": {
                                                            "type": "string"
                                                        },
                                                        "hooks_url": {
                                                            "type": "string"
                                                        },
                                                        "issue_events_url": {
                                                            "type": "string"
                                                        },
                                                        "events_url": {
                                                            "type": "string"
                                                        },
                                                        "assignees_url": {
                                                            "type": "string"
                                                        },
                                                        "branches_url": {
                                                            "type": "string"
                                                        },
                                                        "tags_url": {
                                                            "type": "string"
                                                        },
                                                        "blobs_url": {
                                                            "type": "string"
                                                        },
                                                        "git_tags_url": {
                                                            "type": "string"
                                                        },
                                                        "git_refs_url": {
                                                            "type": "string"
                                                        },
                                                        "trees_url": {
                                                            "type": "string"
                                                        },
                                                        "statuses_url": {
                                                            "type": "string"
                                                        },
                                                        "languages_url": {
                                                            "type": "string"
                                                        },
                                                        "stargazers_url": {
                                                            "type": "string"
                                                        },
                                                        "contributors_url": {
                                                            "type": "string"
                                                        },
                                                        "subscribers_url": {
                                                            "type": "string"
                                                        },
                                                        "subscription_url": {
                                                            "type": "string"
                                                        },
                                                        "commits_url": {
                                                            "type": "string"
                                                        },
                                                        "git_commits_url": {
                                                            "type": "string"
                                                        },
                                                        "comments_url": {
                                                            "type": "string"
                                                        },
                                                        "issue_comment_url": {
                                                            "type": "string"
                                                        },
                                                        "contents_url": {
                                                            "type": "string"
                                                        },
                                                        "compare_url": {
                                                            "type": "string"
                                                        },
                                                        "merges_url": {
                                                            "type": "string"
                                                        },
                                                        "archive_url": {
                                                            "type": "string"
                                                        },
                                                        "downloads_url": {
                                                            "type": "string"
                                                        },
                                                        "issues_url": {
                                                            "type": "string"
                                                        },
                                                        "pulls_url": {
                                                            "type": "string"
                                                        },
                                                        "milestones_url": {
                                                            "type": "string"
                                                        },
                                                        "notifications_url": {
                                                            "type": "string"
                                                        },
                                                        "labels_url": {
                                                            "type": "string"
                                                        },
                                                        "releases_url": {
                                                            "type": "string"
                                                        },
                                                        "deployments_url": {
                                                            "type": "string"
                                                        }
                                                    }
                                                }
                                            }
                                        }
                                    }
                                },
                                "Set_variable_2": {
                                    "type": "SetVariable",
                                    "inputs": {
                                        "name": "WFRunId",
                                        "value": "@body('Parse_JSON_1')?['id']"
                                    },
                                    "runAfter": {
                                        "Condition_1": [
                                            "Succeeded"
                                        ]
                                    }
                                },
                                "Condition_1": {
                                    "type": "If",
                                    "expression": {
                                        "and": [
                                            {
                                                "contains": [
                                                    "@body('Parse_JSON_1')?['status']",
                                                    "completed"
                                                ]
                                            }
                                        ]
                                    },
                                    "actions": {
                                        "Set_variable_1": {
                                            "type": "SetVariable",
                                            "inputs": {
                                                "name": "ItemFound",
                                                "value": true
                                            }
                                        }
                                    },
                                    "else": {
                                        "actions": {}
                                    },
                                    "runAfter": {
                                        "Send_an_email_(V2)_1": [
                                            "Succeeded"
                                        ]
                                    }
                                },
                                "Send_an_email_(V2)_1": {
                                    "type": "ApiConnection",
                                    "inputs": {
                                        "host": {
                                            "connection": {
                                                "referenceName": "office365"
                                            }
                                        },
                                        "method": "post",
                                        "body": {
                                            "To": "john.demo@email.com",
                                            "Subject": "Test2",
                                            "Body": "<p class=\"editor-paragraph\">@{body('Parse_JSON_1')?['status']}</p>",
                                            "Importance": "Normal"
                                        },
                                        "path": "/v2/Mail"
                                    },
                                    "runAfter": {
                                        "Parse_JSON_1": [
                                            "Succeeded"
                                        ]
                                    }
                                }
                            }
                        },
                        "runAfter": {
                            "FilterArray": [
                                "Succeeded"
                            ]
                        }
                    },
                    "Send_an_email_(V2)": {
                        "type": "ApiConnection",
                        "inputs": {
                            "host": {
                                "connection": {
                                    "referenceName": "office365"
                                }
                            },
                            "method": "post",
                            "body": {
                                "To": "john.demo@email.com",
                                "Subject": "Test",
                                "Body": "<p class=\"editor-paragraph\">@{body('Get_Workflow_Runs')}</p>",
                                "Importance": "Normal"
                            },
                            "path": "/v2/Mail"
                        },
                        "runAfter": {
                            "Get_Workflow_Runs": [
                                "Succeeded"
                            ]
                        }
                    }
                },
                "runAfter": {
                    "Submit_Workflow": [
                        "Succeeded"
                    ]
                }
            },
            "InitializeAllVariables": {
                "type": "InitializeVariable",
                "inputs": {
                    "variables": [
                        {
                            "name": "WorkflowId",
                            "type": "string",
                            "value": "@{guid()}"
                        },
                        {
                            "name": "ItemFound",
                            "type": "boolean",
                            "value": false
                        },
                        {
                            "name": "WFRunId",
                            "type": "integer",
                            "value": 0
                        },
                        {
                            "name": "InstallationToken",
                            "type": "string",
                            "value": "@{outputs('GetGitHubInstallationToken')['body']['message']}"
                        }
                    ]
                },
                "runAfter": {
                    "GetGitHubInstallationToken": [
                        "SUCCEEDED"
                    ]
                }
            },
            "GetRunInfo": {
                "type": "Http",
                "inputs": {
                    "uri": "https://api.github.com/repos/OWNER/REPO/actions/runs/@{variables('WFRunId')}",
                    "method": "GET",
                    "headers": {
                        "Accept": "application/vnd.github+json",
                        "Authorization": "Bearer @{variables('InstallationToken')}",
                        "X-GitHub-Api-Version": "2022-11-28"
                    }
                },
                "runAfter": {
                    "Until": [
                        "Succeeded"
                    ]
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "GetSecret": {
                "type": "ServiceProvider",
                "inputs": {
                    "parameters": {
                        "secretName": "githubkey"
                    },
                    "serviceProviderConfiguration": {
                        "connectionName": "keyVault",
                        "operationId": "getSecret",
                        "serviceProviderId": "/serviceProviders/keyVault"
                    }
                },
                "runAfter": {},
                "runtimeConfiguration": {
                    "secureData": {
                        "properties": [
                            "inputs",
                            "outputs"
                        ]
                    }
                }
            },
            "GetGitHubInstallationToken": {
                "type": "CSharpScriptCode",
                "inputs": {
                    "CodeFile": "execute_csharp_script_code.csx"
                },
                "runAfter": {
                    "GetSecret": [
                        "SUCCEEDED"
                    ]
                }
            }
        },
        "outputs": {}
    },
    "kind": "stateful"
}
```