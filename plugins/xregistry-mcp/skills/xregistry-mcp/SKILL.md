---
name: xregistry-mcp
description: >
  Access xRegistry-compliant registries to discover and navigate specifications,
  schemas, APIs, and other registry resources. Use this skill when the user wants
  to search for specifications, list registry groups or resources, get resource
  details, or export registry content. Activate when the user mentions xRegistry,
  specification registries, or needs to discover standards-based resources.
license: MIT
compatibility: Requires Node.js 18+ or later.
metadata:
  author: spec-works
  version: "1.0.0"
  repository: https://github.com/spec-works/xRegistry-MCP-Server
---

# xRegistry MCP Server

Model Context Protocol server for accessing xRegistry-compliant registries.

## Overview

The xRegistry MCP Server provides tools for connecting to and querying xRegistry instances,
enabling AI assistants to discover and navigate specifications, schemas, APIs, and other
registry resources.

## Prerequisites

- Node.js 18 or later

## Installation

```bash
npm install -g @spec-works/xregistry-mcp-server
```

Or run directly with npx:

```bash
npx -y @spec-works/xregistry-mcp-server
```

## MCP Tools

| Tool | Description |
|------|-------------|
| `connect` | Connect to an xRegistry instance |
| `list_groups` | List resource groups in the registry |
| `list_resources` | List resources within a group |
| `get_resource` | Get details of a specific resource |
| `search` | Search across registry resources |
| `export` | Export registry content |

## Usage

Once connected to an xRegistry instance, you can discover and navigate registry content:

| Tool | Parameters |
|------|------------|
| `connect` | `{ "url": "https://registry.example.com" }` |
| `list_groups` | `{}` |
| `search` | `{ "query": "CloudEvents" }` |
