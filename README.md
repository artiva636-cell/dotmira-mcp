<!-- This repository describes how to connect to the Dotmira MCP server. It contains no catalog code. -->
> **Русская версия:** [README.ru.md](README.ru.md) · **Documentation on the site:** https://dotmira.com/en/developers · **Registry entries:** [registry/](registry/)

# Dotmira MCP server and API

[![Dotmira MCP connector](https://glama.ai/mcp/connectors/com.dotmira/catalog/badges/score.svg)](https://glama.ai/mcp/connectors/com.dotmira/catalog)

Dotmira has an API. It comes as two programmatic interfaces: an **MCP server** for AI assistants, AI agents and code editors — Claude, ChatGPT, Cursor, VS Code and any other MCP client — and a **shadcn-compatible component registry** for installing from the command line. Your assistant finds the right web effect, animation or UI component in the catalog and receives its code with the author, source and license.

[Account → MCP & registry](https://dotmira.com/en/account#mcp)

## At a glance

| What | Value |
| --- | --- |
| MCP server | `https://dotmira.com/mcp` — a remote server, nothing to install |
| Protocol | Model Context Protocol (MCP) over Streamable HTTP; specification 2026-07-28, clients of earlier versions connect too |
| Sign-in | OAuth 2.1 (sign in with your Dotmira account) or a personal token (Authorization: Bearer dtm_…) |
| Tools | `search_effects`, `get_effect`, `list_categories`, `get_preview` — read-only |
| Registry | `https://dotmira.com/r/{name}.json`, namespace @dotmira, shadcn registry format |
| Catalog | 16,000+ web effects, animations, UI components and templates: React, Tailwind, HTML/CSS, TypeScript, Vue |
| Not available | a REST or GraphQL API, an SDK, an OpenAPI description, a CLI of our own |

## MCP server

The server speaks the open Model Context Protocol (MCP) over Streamable HTTP. It is a remote server: there is nothing to install — you need only the address and a sign-in.

### Server address

- dotmira.com `https://dotmira.com/mcp`
- dotmira.ru `https://dotmira.ru/mcp`

Each site has its own address. An assistant connected with sign-in works only on the site where you allowed it.

### Tools: what an assistant can do

- `search_effects` — searches the catalog by words, section and technology; returns titles, authors and links, without code.
- `get_effect` — the code of one element: files, dependencies, license, author and source, and the command to install it from the registry.
- `list_categories` — the tree of catalog sections with the number of elements in each.
- `get_preview` — a link to the preview image and to the element page with the live preview.

The tools only read the catalog and change nothing in your account.

### What you can ask

- “Find a glowing button for React in Dotmira and add it to my project.”
- “Show five animated backgrounds for Tailwind from Dotmira, with preview links.”
- “Which sections does the Dotmira catalog have?”

### Assistants that sign in: Claude, ChatGPT

In the assistant’s connector settings, add the server address `https://dotmira.com/mcp`, sign in to Dotmira and click “Allow” (OAuth 2.1). This is how claude.ai and ChatGPT connect, for example. Where these settings are depends on the assistant and your account with it.

The assistant gets access to the catalog only: it does not see your name, e-mail, phone or payments. You can revoke access at any time in your account (Connected apps) or by removing the connector in the assistant.

> Whether an assistant itself is available where you are is decided by its maker: Anthropic and OpenAI publish lists of countries where Claude and ChatGPT work, and not every country is on them. Dotmira’s MCP server and registry do not depend on your country.

### Code editors and terminal agents: personal token

Cursor, VS Code, Claude Code and any other MCP client that connects to remote servers and can send headers work with a personal token. Create the token in your account, section “MCP & registry”, and send it in the header `Authorization: Bearer dtm_…`.

| Setting | Value |
| --- | --- |
| Address (URL) | `https://dotmira.com/mcp` |
| Transport | Streamable HTTP (“HTTP” or “streamable-http” in client settings) |
| Header | `Authorization: Bearer <your token>` |

Cursor — the file `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "dotmira": {
      "url": "https://dotmira.com/mcp",
      "headers": {
        "Authorization": "Bearer ${env:DOTMIRA_TOKEN}"
      }
    }
  }
}
```

VS Code — the file `.vscode/mcp.json` (VS Code asks for the token on the first connection):

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "dotmira-token",
      "description": "Dotmira token (dtm_…)",
      "password": true
    }
  ],
  "servers": {
    "dotmira": {
      "type": "http",
      "url": "https://dotmira.com/mcp",
      "headers": {
        "Authorization": "Bearer ${input:dotmira-token}"
      }
    }
  }
}
```

Claude Code — run in the terminal:

```sh
claude mcp add --transport http dotmira https://dotmira.com/mcp \
  --header "Authorization: Bearer $DOTMIRA_TOKEN"
```

### Check the connection

A direct request to the server, to check the token and the connection:

```sh
curl -s https://dotmira.com/mcp \
  -H "Authorization: Bearer $DOTMIRA_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"search_effects","arguments":{"query":"glow button","limit":3}}}'
```

The server answers with an event stream: the line `data:` carries the MCP (JSON-RPC) message with the elements found, without code.

## Component registry (shadcn)

The @dotmira registry works with the shadcn CLI: `npx shadcn@latest add` puts the element’s files right into your project. It uses the same personal token.

For programs the registry is an HTTP address: the request `GET https://dotmira.com/r/<id>.json` with the token returns the element in the shadcn registry format (JSON).

Answer codes:

- `401` — no token, or the token is wrong
- `403` — access to code is closed for this account
- `404` — no such element
- `429` — too many requests, try again later

1. Create a personal token: [Account → MCP & registry](https://dotmira.com/en/account#mcp) → “Create token”.
   Copy the token now: it is shown only once. Dotmira keeps only its fingerprint.
2. Add the registry to `components.json` in your project:

   ```json
   {
     "registries": {
       "@dotmira": {
         "url": "https://dotmira.com/r/{name}.json",
         "headers": {
           "Authorization": "Bearer ${DOTMIRA_TOKEN}"
         }
       }
     }
   }
   ```

3. Put the token in `.env.local` (do not commit this file to git) and install an element:

   ```sh
   # .env.local
   DOTMIRA_TOKEN=dtm_…
   
   npx shadcn@latest add @dotmira/glow-button
   ```

- The element id is in the address of its page (`…/e/<id>`) or in the answer of `search_effects`.
- React elements are installed with their files and npm dependencies; HTML/CSS elements — as files in `components/dotmira/<id>/`.
- Each file carries a comment with the author, source and license, and your personal mark.
- The shadcn MCP server (`npx shadcn@latest mcp`) understands this registry too: you can ask the assistant to add `@dotmira/glow-button`.

## For AI agents and automation

An AI agent acting for a user gets code only through the MCP server or the registry — with that user’s sign-in or token. Crawling the site automatically, scraping pages and collecting code in any other way are prohibited by the Terms of Use (clause 5.1). Using the catalog’s content to train AI models is prohibited too (clauses 5.1 and 5.3), including content received through the MCP server and the registry. Search engines and AI search and answer services may show and quote the public pages with a link to the site (clause 4.3).

## Rules

- Connect the MCP server and the registry only to assistants that do not use your data for training, or turn such use off in the assistant’s settings — this is a condition of the [License terms](https://dotmira.com/en/legal/subscriber-license) (clause 5.2).
- A token is shown once: Dotmira keeps only its fingerprint. A revoked token stops working at once.
- Access is personal. Do not give your token or access to your account to anyone, including colleagues and clients.
- Do not remove the author, source, license or your personal mark from the code, and do not publish the catalog’s code as a set of components.

Full terms: [License terms](https://dotmira.com/en/legal/subscriber-license) and [Terms of Use](https://dotmira.com/en/legal/terms).

## Questions and answers

### Does Dotmira have an API?

Yes. It is a remote MCP server (https://dotmira.com/mcp) and a shadcn-compatible component registry (https://dotmira.com/r/{name}.json). Both work over open protocols. There is no REST or GraphQL API, no SDK and no OpenAPI description.

### Does Dotmira have an MCP server?

Yes: a remote MCP server at https://dotmira.com/mcp, Streamable HTTP transport, sign-in with OAuth 2.1 or a personal token. It has four read-only tools: search_effects, get_effect, list_categories and get_preview.

### Which AI assistants and code editors does it work with?

With any MCP client that connects to remote servers over Streamable HTTP. This page has instructions for claude.ai, ChatGPT, Cursor, VS Code and Claude Code.

### Which countries does it work in?

Whether an assistant itself is available where you are is decided by its maker: Anthropic and OpenAI publish lists of countries where Claude and ChatGPT work, and not every country is on them. Dotmira’s MCP server and registry do not depend on your country.

### Is there a REST API, an SDK or an OpenAPI description?

No. Programs work with Dotmira through the MCP server and the registry. The registry answers plain HTTP requests with JSON; the MCP server answers JSON-RPC requests over HTTP.

### Can I call the API from my own program, without an AI assistant?

Yes — with your personal token. Extracting the whole catalog or a substantial part of it is prohibited (License terms, clause 5.1).

### What does the assistant learn about my account?

Nothing but the catalog. It does not see your name, e-mail, phone or payments. You can revoke access at any time in your account (Connected apps).

### Is there a command-line tool?

Dotmira has no CLI of its own: elements are installed with the shadcn CLI — npx shadcn@latest add @dotmira/<id>.

### Can an AI agent browse the site and copy the code by itself?

No. Code is issued only on the site, through the MCP server and through the registry. Crawling the site automatically and collecting code are prohibited by the Terms of Use (clause 5.1).

### Can the code be used to train AI models?

No. The Terms of Use (clauses 5.1 and 5.3) and the License terms (clauses 5.1 and 5.2) prohibit it. Connect only assistants that do not train on your data.

### What is the difference between the MCP server and the registry?

The MCP server lets an assistant search the catalog and fetch code right in a conversation. The registry installs an element by its id with one command. Both use the same personal token.

### Where do I get the element id?

In the address of the element page (…/e/<id>) or in the answer of search_effects.

Updated: November 12, 2026
