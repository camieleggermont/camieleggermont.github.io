---
title: "GPT Actions: Organizing The Chaos"
tag: ChatGPT GPT OpenAI
---

A few weeks ago, Sam Altmann introduced GPTs in ChatGPT. GPTs allow creators to develop ChatGPT assistants using a specific set of data.

The discussion about GPTs often revolves around how you can seed data to it and craft a prompt for a specific purpose or subject. However, one less discussed feature is the ability to call REST APIs from the GPT. This exciting option, known as Actions, allows GPTs to interact with external systems and source information from them!

Actions are an advancement of ChatGPT plugins. They utilize the OpenAPI standard to describe REST APIs. The ChatGPT's LLM "understands" the OpenAPI specification and decides whether to call an API based on its description.

Unlike ChatGPT plugins, which required hosting the OpenAPI specification externally with the API itself, Actions eliminates this need. Now, it's possible to store the OpenAPI specification directly in the GPT. This means APIs without an OpenAPI spec can be utilized by creating a specification for the API in the GPT.

While unauthorized APIs can be used, GPTs can also access authorized APIs via an API Key or OAuth. The latter, in particular, opens up a world of possibilities!

Despite my enthusiasm, it's clear that this technology is still in its early stages. The tooling is somewhat inconsistent. For example, the OAuth Callback URL seems to change sporadically, which can make the authentication step frustrating.

When I first started exploring Actions, there was a graphical interface for defining the API actions. This was later replaced by a plain text field for authoring the OpenAPI specification code. It wasn't a problem for me, but it does highlight how much the product is still evolving.

One exciting prospect of GPT Actions is that it could enable GPTs not only to read and interpret information for use in user conversations, but also to **DO** something.

People who know me also know that I can be slightly chaotic at times. This is why I have a long dream of having a personal AI assistant, who can manage my tasks. I use ToDoist for my task management, so I thought: let’s see if I can hook up a GPT to ToDoist, and have a GPT help me managing my tasks.

## Setting up the GPT

To create a GPT, you must be a ChatGPT subscriber, as this option is not available to free accounts.

You can create a new GPT in the "Explore" section of ChatGPT.

![My_GPTs.png]({{ site.baseurl }}/images/gpt-actions/My_GPTs.png)

GPTs are created using the GPT Builder in a conversational manner. This means you can interact with ChatGPT (located on the left side of the screen) to construct a GPT. This is quite impressive! You can test the GPT you create on the right side of the builder.

I began with this prompt. It's not optimized yet, but that's not the focus of this post.

```yaml
Your initial prompt is a good starting point for creating a task management assistant using GPT for Todoist. However, let's refine it a bit to ensure clarity and specificity:

"I'm developing an AI assistant to help me manage my tasks in Todoist, my preferred task manager. This assistant will play the role of a 'critical friend' who engages in conversations with me about my tasks and helps me plan them effectively.

To achieve this, I'll create specific actions that enable the assistant to interact with Todoist. These actions will include the ability to:

1. Read my tasks from Todoist.
2. Retrieve the projects in Todoist.
3. Mark tasks as complete.
4. Create new tasks in Todoist.

It's important to note that the assistant's primary function will be to assist with tasks related to Todoist and nothing else. It should provide recommendations, reminders, and task management guidance within the scope of my Todoist account.
```

After this initial prompt, the builder suggest an name (Task Companion, in my case), and generates a profile picture. After that it engages in a conversation to finetune the GPTs behaviour.

![GPT-Builder.png]({{ site.baseurl }}/images/gpt-actions/GPT-Builder.png)

This whole discussions results in a set of instructions, which you can find under the “Configure” section. There are a few other things you can configure here as well.

![GPT-Builder-Configure.png]({{ site.baseurl }}/images/gpt-actions/GPT-Builder-Configure.png)

## The API specification

All of this configures the behaviour of the GPT, but it can still not talk to ToDoist.

As mentioned, GPTs use an OpenAPI specification to describe the REST APIs it needs to use. For a lot of developers, OpenAPI is better known under it’s previous name: swagger.

ToDoist has an [REST API](https://developer.todoist.com/rest/v2/), but they don’t provide a OpenAPI specification themselves. This is, I think, mostly because not all of the parameters in the endpoints can be described in an OpenAPI specification. For instance, the filter parameter for [get-active-tasks](https://developer.todoist.com/rest/v2/#get-active-tasks) requires a [supported filter format](https://todoist.com/help/articles/introduction-to-filters-V98wIH), which can’t be described in a specification I think.

Because of this, I created a simple OpenAPI specification myself. One of the big advantages working this way is that you can specify specific actions, for instance using the beforementioned get-active-tasks with different filter parameter. 

```yaml
{
  "openapi": "3.1.0",
  "info": {
    "title": "ToDoist task management",
    "description": "ToDoist is software to manage tasks",
    "version": "v0.0.1"
  },
  "servers": [
    {
      "url": "https://api.todoist.com"
    }
  ],
  "paths": {
    "/rest/v2/tasks": {
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
      },
      "post": {
        "description": "Creates a new task",
        "operationId": "Create a task",
        "x-openai-isConsequential": true,
        "parameters": [
          {
            "name": "content",
            "in": "body",
            "description": "The content of the task. This is used as a title as well",
            "required": true,
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "description",
            "in": "body",
            "description": "The description of the task",
            "required": false,
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "due_string",
            "in": "body",
            "description": "Human defined task due date (ex.: \"next Monday\", \"Tomorrow\"). Value is set using local (not UTC) time.",
            "required": false,
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "project_id",
            "in": "body",
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
    },
    "/rest/v2/tasks/{taskid}/close": {
      "post": {
        "description": "Closes a task",
        "operationId": "Close a task",
        "x-openai-isConsequential": true,
        "parameters": [
          {
            "name": "taskid",
            "in": "path",
            "description": "A task id",
            "required": true,
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
    },
    "/rest/v2/projects": {
      "get": {
        "description": "Gets all the projects",
        "operationId": "Get all projects",
        "x-openai-isConsequential": false,        
        "parameters": [],
        "deprecated": false,
        "security": [
          {
            "oauth2": []
          }
        ]
      }
    }
  },
  "components": {
    "schemas": {},
    "securitySchemes": {
      "oauth2": {
        "type": "oauth2"
      }
    }
  }
}
```

In the “Configure” section, where you also find the GPTs instructions, you will find the button “Create new action”. This opens a dialog where you can edit the OpenAPI specification. Pasting the specification above results in this.

![GPT-Builder-Configure-Actions.png]({{ site.baseurl }}/images/gpt-actions/GPT-Builder-Configure-Actions.png)

The ToDoist API is authenticated through OAuth. The GPT builder provides an option to configure OAuth. 

![GPT-Builder-Configure-Authentication.png]({{ site.baseurl }}/images/gpt-actions/GPT-Builder-Configure-Authentication.png)

This first requires you to set up an OAuth client at ToDoist side. Configuring an OAuth client differs between providers, but in the end should always provide a ClientId, Client Secret, Authorization URL and Token URL. You should also provide one or more scopes your app (in this case our GPT) needs. 

For ToDoist you can create a client in their [App Management](https://developer.todoist.com/appconsole.html). You need to configure the allowed scopes here. In our case, we need “data:read_write”.

![ToDoist-client-settings.png]({{ site.baseurl }}/images/gpt-actions/ToDoist-client-settings.png)

![ToDoist-client-scopes.png]({{ site.baseurl }}/images/gpt-actions/ToDoist-client-scopes.png)

Take the clientid and client secret, and copy them to the auth options for your GPT.

The [ToDoist documentation](https://developer.todoist.com/guides/#authorization) provides the necessary URLs:

- Auth URL: [https://todoist.com/oauth/authorize](https://todoist.com/oauth/authorize)
- Token URL: [https://todoist.com/oauth/access_token](https://todoist.com/oauth/access_token)

Once this is done, you need to get the Callback URL from your GPT, and copy it to the ToDoist Client settings. At the time of writing, there was a bug in the GPT builder. causing in an incorrect callback url to show. You first need to save the GPT to get the correct one; just save and publish to “only me”. Then, go to “Explore” again, edit your GPT, and go to “Configure”. At the botttom of the screen you will find your callback url.

![Callback-URL.png]({{ site.baseurl }}/images/gpt-actions/Callback-URL.png)

You need to copy this URL and paste it in the “OAuth redirect URL” in your ToDoist client, and save the settings.

![ToDoist-client-callback-URL.png]({{ site.baseurl }}/images/gpt-actions/ToDoist-client-callback-URL.png)

And that should be it! Now you can use the test pane of the GPT builder to ask some questions.

## Testing the GPT

Testing is simple! Just ask the GPT a question about your task. Doing so in the test pane will provide you with debug info about the http calls being send to the API, which can be helpfull.

![Testing-Login.png]({{ site.baseurl }}/images/gpt-actions/Testing-Login.png)

The GPT will prompt you to sign in with ToDoist. When you press the button you need to give consent.

![Testing-Consent.png]({{ site.baseurl }}/images/gpt-actions/Testing-Consent.png)

The feedback should say that you are logged on. A bug (again) in the GPT builder sends you to the wrong page though, so again, go to “Explore”, edit the GPT, and start the conversation again.

![Testing-AllowAction.png]({{ site.baseurl }}/images/gpt-actions/Testing-AllowAction.png)

When you give consent to execute the action, GPT will call the API and get your tasks!

[GPT-ToDoist.mp4]({{ site.baseurl }}/images/gpt-actions/GPT-ToDoist.mp4)

## Granting Consent for Actions

During my test, I had to provide consent for every action, every time. This becomes somewhat bothersome as it disrupts the conversation flow in the chat and precludes the possibility of conducting speech conversations.

To circumvent this issue, one could set the “x-openai-isConsequential” property in the OpenAPI specification to false. This signifies that the action has no real consequence and can be executed. Ideally, this should introduce an “Allow Always” option when the API is first called.

However, I have not been successful in using this feature: I am still required to give permission for the Action to execute each time. This could potentially be another bug or perhaps a mistake on my part. 🙂

## Conclusion

Using Actions in GPTs is possible, though it's clear that this feature is in its early stages of development. There are numerous bugs in the builder when setting up Actions such as displaying incorrect callback URLs and issues with OAuth redirects. The builder's user interface also changes frequently.

It's important to exercise caution when allowing an LLM to interact with your data, particularly when the APIs have the ability to modify the information. 

I do believe the technology *is* quite impressive! Although it's in its early stages, it's not hard to envision its potential. It hints at a future where configurable AI assistants can help us with a variety of tasks.