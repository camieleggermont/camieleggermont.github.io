---
title: "Why Isn't 'x-openai-isConsequential' Working in GPT Actions? Solved it!"
excerpt_separator: <!--more-->
---

In my ongoing exploration of GPT Actions, I've stumbled upon a crucial fix that might save you time and frustration. In a previous [blog post]({% post_url 2023-12-08-GPT Actions-Connecting-My-GPT-To-ToDoist %}), I discussed the challenges I faced while integrating GPT with ToDoist, particularly with the `x-openai-isConsequential` property not functioning as expected.
<!--more-->

## Recap of the Problem
The issue was with the GPT not recognizing the `x-openai-isConsequential` property in my OpenAPI specification. This led to the need for manual consent for each API call, disrupting the workflow significantly.

Here's the original snippet causing the issue:

```yaml
/rest/v2/tasks": {
  "get": {
    "description": "Gets all the active tasks",
    "operationId": "Get all tasks",
    "x-openai-isConsequential": false,
    "parameters": [
      {
        "name": "project_id",
        "in": "query",
        "description": "A project id",
        "required": false,
        "schema": {
          "type": "integer"
        }
      }
    ],
    "deprecated": false,
    "security": [
      {
        "oauth2": []
      }
    ]
  }
}
```

# The Discovery
After some trial and error, I realized the solution was in the details of the `operationId` property.

## The Solution

Here's the corrected version of the OpenAPI specification:

```yaml
/rest/v2/tasks": {
  "get": {
    "description": "Gets all the active tasks",
    "operationId": "GetAllTasks",
    "x-openai-isConsequential": false,
    "parameters": [
      {
        "name": "project_id",
        "in": "query",
        "description": "A project id",
        "required": false,
        "schema": {
          "type": "integer"
        }
      }
    ],
    "deprecated": false,
    "security": [
      {
        "oauth2": []
      }
    ]
  }
}
```

Changing "Get all tasks" to "GetAllTasks" surprisingly resolved the issue. It turns out that the `operationId` in the OpenAPI document needs to contain only alpha-numeric characters for `x-openai-isConsequential` to work effectively. This requirement, while not part of the standard [OpenAPI specifications](https://swagger.io/docs/specification/paths-and-operations/), is also not documented in the [GPT Actions documentation](https://platform.openai.com/docs/actions/consequential-flag).

## Reflections
This experience was a reminder of the intricate details that can make or break technical implementations. It also highlights the evolving nature of documentation in rapidly advancing tech fields like AI. My hope is that sharing this discovery will smooth the path for others working with GPT Actions.

I'd love to hear about your experiences and any insights you might have on similar topics. Feel free to share in the comments below!