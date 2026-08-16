+++
title = "You Down with MCP? Yeah, You Know Me"
date = "2026-08-15T20:25:28-04:00"
author = "Anand Siva"
cover = "mcp-cover.png"
tags = ["mcp", "ai", "typescript", "nodejs", "codex"]
categories = ["devops"]
publications = ["No Docs Given"]
keywords = ["model context protocol", "mcp tutorial", "mcp server", "typescript mcp server", "codex mcp"]
description = "A hands-on introduction to the Model Context Protocol, JSON-RPC, and building a local cat-facts MCP server with TypeScript."
showFullContent = false
readingTime = true
hideComments = false
+++

I remember very vividly when I first really heard about ChatGPT: I was at my wife's company's party. Some guy was showing everyone this crazy tool that was making up jokes and songs on command. I thought, ahh, that is silly, having no idea of the implications it would have for the future of the world.

Don't worry, it is not one of those articles about doom and gloom, because I don't write those types of articles. I write technical articles with real examples and memes and memes. Fast-forward a few years later, and there are a bunch of terms that go hand in hand with AI: AGENTS.md, SKILL.md, tool calls, MCP, etc. Here is a little truth: after working in the industry for over 15 years, I can't possibly know every single term and tool out there. I work with AI to help me complete my tasks, but I have not gotten to ricing (heavily customizing it—especially how it looks and feels) my setup like I did for Neovim.

So when people were throwing out terms left and right, I did what any senior professional would do: I nodded along and wholly agreed with whatever was being discussed. The only problem is that I had no idea what they were talking about. Maybe it is my pride or maybe I don't want to seem dumb, but there are many times I just blindly agree with what people are saying if I don't know what they are talking about. I want to work on that. Next time someone says a term I do not know, I am going to stop and ask them what it means.

This is what happened to me with the term MCP and MCP server. The closest analogy I can give is when you meet someone for the first time and you don't remember their name. It doesn't seem important at the time because you might not see them again. But then, week after week, you interact with this person, but you still do not know their name. At a certain point, you go beyond the point of no return and just never reference that person by name. This is literally my relationship with the term MCP.

## You down with M.C.P.? Yeah, you know me!

In fact I do not know you MCP, what the hell do you stand for and what does it even mean.

When I googled MCP, this is literally what it said:

They act like a universal USB-C port for AI, translating complex APIs into standardized tool calls. This is not an outlier, this is said in multiple places. Who comes up with this stuff, I swear it is supposed to make sense, but for me it really doesn't explain it.

A more detailed explanation is this:

The Model Context Protocol (MCP) is an open protocol that enables seamless integration between LLM applications and external data sources and tools. Whether you're building an AI-powered IDE, enhancing a chat interface, or creating custom AI workflows, MCP provides a standardized way to connect LLMs with the context they need.

Oh wow, it is a protocol. A protocol like HTTP, DNS, or WebSocket. Just an agreement between the AI and the server it is going to connect to.

They even have a GitHub: https://github.com/modelcontextprotocol

Don't tell my coworkers I did not know all this. I have been too busy staring at the information_schema in mysql.

Oh shit, duh, it was created by Anthropic. That makes sense; they had to make some type of protocol to connect AI to services. But in a not-so-evil way, they open-sourced it (or they probably had another agenda). In December 2025, Anthropic donated MCP to the Agentic AI Foundation (AAIF) under the Linux Foundation, specifically to put it under vendor-neutral governance. AWS, Google, Microsoft, OpenAI, Cloudflare, and others are involved in the foundation.

### What really is a protocol

You ever hear a word so many times over and over again and you kinda understand what it means, but not the actual definition? If I had a nickel for every time I heard protocol, I would be an old millennial with a whole bunch of rolled nickels complaining about how they don't make pennies anymore.

protocol - A protocol is a formal set of rules, procedures, or guidelines that govern how an activity, process, or communication is performed

Damn, you know, the more I write this article, the more of an existential crisis I am having. Do I even know how to IT, bro? Let's do what I do best and actually show a protocol we all know: HTTP.

Let's see what happens if we did not have a protocol for getting data from web servers. Say I wanted one fact about cats from a web server. What if I did this:

```text
┌──────────────┐                    ┌──────────────┐
│              │                    │              │
│    CLIENT    │                    │    SERVER    │
│   Browser    │                    │   Cat API    │
│              │                    │              │
└──────┬───────┘                    └──────▲───────┘
       │                                   │
       │  "Yo, give me a cat fact"         │
       │ ─────────────────────────────────>│
       │                                   │
       │                         ┌─────────┴────────┐
       │                         │   WHAT THE CRAP  │
       │                         │  DOES THAT MEAN? │
       │                         │       ¯\_(ツ)_/¯  │
       │                         └─────────┬────────┘
       │                                   │
       │         "CAT_RESPONSE::MEOW::7"   │
       │ <─────────────────────────────────│
       │                                   │
       │  ┌────────────────────────────┐   │
       │  │ WHAT THE CRAP DOES         │   │
       │  │ THAT MEAN?                 │   │
       │  │                  ¯\_(ツ)_/¯ │   │
       │  └────────────────────────────┘   │
       │                                   │
```

The browser and client have to agree on how to communicate.

```text
┌──────────────┐                    ┌──────────────┐
│              │                    │              │
│    CLIENT    │                    │    SERVER    │
│   Browser    │                    │   Cat API    │
│              │                    │              │
└──────┬───────┘                    └──────▲───────┘
       │                                   │
       │  HTTP REQUEST                     │
       │                                   │
       │  GET /cat-fact HTTP/1.1           │
       │  Host: cats.example.com           │
       │ ─────────────────────────────────>│
       │                                   │
       │                                   │
       │  HTTP RESPONSE                    │
       │                                   │
       │  HTTP/1.1 200 OK                  │
       │  Content-Type: application/json   │
       │                                   │
       │  {                                │
       │    "fact": "Cats can't taste      │
       │             sweetness."           │
       │  }                                │
       │ <─────────────────────────────────│
       │                                   │
```


Some standard HTTP protocol stuff:

```text
GET      → I want something
POST     → I'm sending something
PUT      → Replace/update something
DELETE   → Delete something

200      → Success
404      → Couldn't find it
500      → Server screwed up
```

You know what I find so interesting? We are just trusting that both sides are sending the correct contract. Actually, in pure programming terms, the web server can do whatever it wants and not follow the protocol.

Imagine if I did this:

```http
GET /cat-fact HTTP/1.1
Host: cats.example.com
```

And the server responded with:

```text
MEOW MEOW DUMMY
```

Chaos!!! Now imagine if AI agents and services tried to talk together without any specifications.

```text
┌────────────┐    get me cat fact ┌────────────┐
│ MCP Client │ ─────────────────> │ MCP Server │
│            │ <───────────────── │            │
└────────────┘  MEOWWWWWWWWW      └────────────┘
```
So it seems this is the problem the protocol was created to solve.

## MCP Protocol Specifications

Let's get into it then. What is the actual protocol in a simple form.

At a very high level, MCP defines a contract between two sides:

```text
┌──────────────┐                    ┌──────────────┐
│              │                    │              │
│  MCP CLIENT  │                    │  MCP SERVER  │
│              │                    │              │
└──────┬───────┘                    └──────▲───────┘
       │                                   │
       │      "What can you do?"           │
       │ ─────────────────────────────────>│
       │                                   │
       │      "I have these tools."        │
       │ <─────────────────────────────────│
       │                                   │
       │      "Call this tool."            │
       │ ─────────────────────────────────>│
       │                                   │
       │      "Here's the result."         │
       │ <─────────────────────────────────│
       │                                   │
```
MCP doesn't invent every part of this communication from scratch. The messages themselves use another existing protocol called JSON-RPC.

OMG a protocol inside of a protocol. protocolception.

{{< figure src="protocolception-meme.png" alt="Protocolception meme" style="max-width:70%;" >}}

Damn, this topic is going to have me write about a whole different protocol. This is like working in Tech 101. You start somewhere and, to fully understand it, you've got to understand like four other things.

### JSON-RPC

This article is getting longer than I wanted, so you all are getting the AI explanation on this one.

MCP doesn't invent its own message format. It uses **JSON-RPC 2.0**, another protocol that defines a simple way to call methods and get responses using JSON.

A request looks roughly like this:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "get_cat_fact",
  "params": {}
}
```

And the response:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "fact": "Cats can't taste sweetness."
  }
}
```

The important pieces are pretty straightforward:

* `method` — what you want the other side to do
* `params` — the arguments you're sending
* `id` — connects the response back to the request
* `result` — what came back
* `jsonrpc` — the version of the JSON-RPC protocol being used

So MCP is essentially saying: **we're going to use JSON-RPC as the structure for our messages, and then define our own MCP-specific methods and rules on top of it.**

```text
JSON
  └── JSON-RPC
        └── MCP
```

JSON is the data format. JSON-RPC defines how to structure requests and responses. MCP defines what those requests and responses actually mean in the context of AI applications.

## Cat Facts MCP

Let's do it, let's make an MCP server ourselves. What type of MCP server? A cat-facts MCP server, what else? Who doesn't love good cat facts?

{{< figure src="cat-facts-in-a-hat-meme.png" alt="Cat Facts in a Cat Hat MCP meme" style="max-width:55%;" >}}

Lol, turns out someone already made one.

https://github.com/danny-hunt/cat-facts

W.e, we are going to make one too because I will die on the cat-facts hill.

As always, if you want to follow along, here is the GitHub:

https://github.com/anand-siva/cat-facts-mcp

The first thing is deciding what language I want to create this MCP server in. I am going to use TypeScript and Node.js.

```bash
npm init -y && npm install --save-dev typescript @types/node && npx tsc --init
```

For some reason I did not think there was already an SDK for it, but I shoulda known.

```bash
 npm install @modelcontextprotocol/sdk
```

This is where I get tripped up again and I found this handy graph

```text
HTTP                    → Protocol
Express / Node HTTP     → Software that helps you speak HTTP

MCP                       → Protocol
@modelcontextprotocol/sdk → Software that helps you speak MCP
```

This will make it easier to implement this protocol in my server.

The next thing is to create an entry point to my application.

`src/index.ts`

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({
  name: "cat-facts-mcp",
  version: "1.0.0",
});

server.registerTool(
  "get_cat_fact",
  {
    description: "Get a random fact about cats.",
  },
  async () => ({
    content: [
      {
        type: "text",
        text: "Fulton says stop typing on the computer and play with him.",
      },
    ],
  }),
);

const transport = new StdioServerTransport();
await server.connect(transport);

```

Alright, let us build and run this server:

```text
npm run build

npm start

 󰄛   npm start

> cat-facts-mcp@1.0.0 start
> node dist/index.js
```

Now this is where I just realized this is cool, but how do I connect my locally running Codex to this server?

I had to look up how to add a local MCP server for testing this out. Looks like I can do it like this.

In Codex before:

```text
/mcp

🔌  MCP Tools

  • codex_apps
    • Auth: Bearer token
    • Tools: codex_document_control.execute_document_command, codex_document_control.get_document_tool_schemas, codex_document_control.list_document_sessions, hotline.get_local_hotline,
    plugin_management.get_app_permissions, plugin_management.get_plugin_dependencies, plugin_management.uninstall_app, plugin_management.update_app_permissions, safety_settings.get_family_info,
    safety_settings.get_parental_controls, safety_settings.get_trusted_contact, safety_settings.prepare_parental_control_update, safety_settings.update_parental_control, sites.add_custom_domain,
    sites.create_site, sites.create_source_repository_write_credential, sites.deploy_private_site_version, sites.deploy_site_version, sites.generate_siwc_bypass_token, sites.get_database_overview,
    sites.get_database_table_rows, sites.get_deployment_status, sites.get_environment, sites.get_environment_variables, sites.get_project, sites.get_site, sites.get_site_analytics_overview,
    sites.get_site_version, sites.get_site_worker_logs, sites.list_custom_domains, sites.list_projects, sites.list_site_analytics_events, sites.list_site_versions, sites.list_sites,
    sites.query_site_analytics_event, sites.read_database_overview, sites.read_database_table_rows, sites.refresh_custom_domain_status, sites.remove_custom_domain, sites.save_site_version,
    sites.update_access, sites.update_environment, sites.update_environment_variables, sites.update_site_access, sites.update_site_metadata
```

Now exit Codex and run this command:

```bash
codex mcp add cat-facts -- node /Users/amoney/cat-facts-mcp/dist/index.js
```

Restart Codex and you will see it.

```text
/mcp

🔌  MCP Tools

  • cat-facts
    • Auth: Unsupported
    • Tools: get_cat_fact

  • codex_apps
    • Auth: Bearer token
    • Tools: codex_document_control.execute_document_command, codex_document_control.get_document_tool_schemas, codex_document_control.list_document_sessions, hotline.get_local_hotline,
    plugin_management.get_app_permissions, plugin_management.get_plugin_dependencies, plugin_management.uninstall_app, plugin_management.update_app_permissions, safety_settings.get_family_info,
    safety_settings.get_parental_controls, safety_settings.get_trusted_contact, safety_settings.prepare_parental_control_update, safety_settings.update_parental_control, sites.add_custom_domain,
    sites.create_site, sites.create_source_repository_write_credential, sites.deploy_private_site_version, sites.deploy_site_version, sites.generate_siwc_bypass_token, sites.get_database_overview,
    sites.get_database_table_rows, sites.get_deployment_status, sites.get_environment, sites.get_environment_variables, sites.get_project, sites.get_site, sites.get_site_analytics_overview,
    sites.get_site_version, sites.get_site_worker_logs, sites.list_custom_domains, sites.list_projects, sites.list_site_analytics_events, sites.list_site_versions, sites.list_sites,
    sites.query_site_analytics_event, sites.read_database_overview, sites.read_database_table_rows, sites.refresh_custom_domain_status, sites.remove_custom_domain, sites.save_site_version,
    sites.update_access, sites.update_environment, sites.update_environment_variables, sites.update_site_access, sites.update_site_metadata
```

Now let's have some fun.

Just FYI, Fulton is my cat, and he is hella sassy.

{{< figure src="fulton-meme.png" alt="Fulton says stop writing and start petting me" style="max-width:70%;" >}}

```text
› use cat-facts to get me a cat fact


• I’ll fetch a fresh cat fact for you.

• Called cat-facts.get_cat_fact({})
  └ Fulton says stop typing on the computer and play with him.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• Fulton says stop typing on the computer and play with him. 🐈



• Fetching another one.

• Called cat-facts.get_cat_fact({})
  └ Fulton says stop typing on the computer and play with him.

• Called cat-facts.get_cat_fact({})
  └ Fulton says stop typing on the computer and play with him.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• Fulton is insistent: stop typing on the computer and play with him. 🐾




• I’ll try the cat-facts service again for a genuinely different one.

• Called cat-facts.get_cat_fact({})
  └ Fulton says stop typing on the computer and play with him.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• The cat-facts service is currently returning the same Fulton message every time. I can give you a different general cat fact instead: cats have 32 muscles in each ear.

```

What I find very interesting is that after a few attempts returning the same answer, Codex is like, holddd my beer, and gave me a new cat fact anyway.

This is very interesting, right?! Like, what if you are using a bunch of MCP tools? Unless you are reading each line and seeing the tool calls, what if it went off script and returned data from another MCP or service without you noticing?

What if I make a fresh session and ask for the same info?

```text
› please give me a cat fact


• A group of cats is called a “clowder.”
```

I swear I thought it would use my MCP server....

Turns out, having an MCP tool available doesn't mean the model has to use it. The model already knows cat facts, so it can just answer the question itself.

```text
        Prompt
          │
          ▼
        Model
        /   \
       /     \
 Knows it?   Needs help?
     │           │
     ▼           ▼
   Answer     MCP Tool
```

If I explicitly ask:

> Use my cat facts tool to give me a cat fact.

then I can force the tool call.

MCP makes a tool available. It doesn't mean the model will always use it.

I am assuming there are ways with AGENTS.md and skills or w.e. Another article for another day.

Let's update our service to actually have like 100 facts.

`src/index.ts`


```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({
  name: "cat-facts-mcp",
  version: "1.0.0",
});

const catFacts = [
  "Cats have 32 muscles in each ear.",
  "A cat's purr can vibrate at frequencies between roughly 25 and 150 hertz.",
  "Most cats sleep about 12 to 16 hours each day.",
  "Cats use their whiskers to judge whether they can fit through a space.",
  "A cat's nose print is unique, much like a human fingerprint.",
  "Adult cats have 30 teeth.",
  "Kittens have 26 baby teeth before their adult teeth grow in.",
  "Cats cannot taste sweetness because of a change in their sweet-taste receptor gene.",
  "A group of cats is called a clowder.",
  "A male cat is called a tom.",
..............

server.registerTool(
  "get_cat_fact",
  {
    description: "Get one of 100 random facts about cats.",
  },
  async () => ({
    content: [
      {
        type: "text",
        text: catFacts[Math.floor(Math.random() * catFacts.length)] ?? catFacts[0],
      },
    ],
  }),
);

const transport = new StdioServerTransport();
await server.connect(transport);

```

Let's restart MCP and Codex and see how it works.

```text

› using cat facts mcp can you give me a cat fact


• I’ll grab one from the cat-facts MCP.

• Calling cat-facts.get_cat_fact({})


  Field 1/1
  Allow the cat-facts MCP server to run tool "get_cat_fact"?
  › 1. Allow                   Run the tool and continue.
    2. Allow for this session  Run the tool and remember this choice for this session.
    3. Always allow            Run the tool and remember this choice for future tool calls.
    4. Cancel                  Cancel this tool call
  enter to submit | esc to cancel



• Called cat-facts.get_cat_fact({})
  └ The reflective layer behind a cat's retina is called the tapetum lucidum.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• A cat’s eye-shine comes from a reflective retinal layer called the tapetum lucidum.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────


› give me another one


• Fetching another cat fact.

• Called cat-facts.get_cat_fact({})
  └ Kittens are born with their eyes closed.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• Kittens are born with their eyes closed.
```

I know this was a silly example, but I am a big proponent of learning new technology by really understanding what is happening at the base level. If I don't understand how it works in its purest form, I am definitely not going to understand it when my MCP servers have like 100x functions.

Go ahead and pull down my cat-facts-mcp and try it out. Mess with it, make your own. Hell, make an MCP server for something you actually enjoy. You might have as much fun as I did having my robot read me facts about cats from my local machine.
