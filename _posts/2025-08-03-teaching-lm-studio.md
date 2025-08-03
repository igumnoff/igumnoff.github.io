---
layout: post
title:  "Teaching LM Studio to Browse the Internet When Answering Questions"
date:   2025-08-03 12:21:01 +0500
categories: ai, mcp, brave, url
---
I really like LM Studio because it allows you to run AI models locally, preserving the privacy of your conversations with the AI. However, compared to commercial online models, LM Studio doesn’t support internet browsing “out of the box.” Those models can’t use up-to-date information from the Internet to answer questions.


Not long ago, LM Studio added the ability to connect MCP servers to models. The very first thing I did was write a small MCP server that can extract text from a URL. It can also extract the links present on the page. This makes it possible, when querying the AI, to specify an address and ask it to extract text from there or retrieve links to use in its response.

To get all of this working, we first create a `pyproject.toml` file in the `mcp-server` folder.


```toml 
[build-system]
requires = ["setuptools>=42", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "url-text-fetcher"
version = "0.1.0"
description = "FastMCP server for URL text fetching"
authors = [{ name="Evgeny Igumnov", email="igumnovnsk@gmail.com" }]
dependencies = [
  "fastmcp",
  "requests",
  "beautifulsoup4",
]
[project.scripts]
url-text-fetcher = "url_text_fetcher.mcp_server:main"
```
Then we create the `mcp_server.py` file in the `mcp-server/url_text_fetcher` folder.
```python
from mcp.server.fastmcp import FastMCP
import requests
from bs4 import BeautifulSoup
from typing import List  # for type hints

mcp = FastMCP("URL Text Fetcher")

@mcp.tool()
def fetch_url_text(url: str) -> str:
    """Download the text from a URL."""
    resp = requests.get(url, timeout=10)
    resp.raise_for_status()
    soup = BeautifulSoup(resp.text, "html.parser")
    return soup.get_text(separator="\n", strip=True)

@mcp.tool()
def fetch_page_links(url: str) -> List[str]:
    """Return a list of all URLs found on the given page."""
    resp = requests.get(url, timeout=10)
    resp.raise_for_status()
    soup = BeautifulSoup(resp.text, "html.parser")
    # Extract all href attributes from <a> tags
    links = [a['href'] for a in soup.find_all('a', href=True)]
    return links

def main():
    mcp.run()

if __name__ == "__main__":
    main()
```

Next, create an empty `__init__.py` in the `mcp-server/url_text_fetcher` folder.

And finally, for the MCP server to work, you need to install it:

```bash
pip install -e .
```

At the bottom of the chat window in LM Studio, where you enter your query, you can choose an MCP server via “Integrations.” By clicking “Install” and then “Edit mcp.json,” you can add your own MCP server in that file.

```json
{
  "mcpServers": {
    "url-text-fetcher": {
      "command": "python",
      "args": [
        "-m",
        "url_text_fetcher.mcp_server"
      ]
    }
  }
}
```

The second thing I did was integrate an existing MCP server from the Brave search engine, which allows you to instruct the AI—in a request—to search the Internet for information to answer a question. To do this, first check that you have `npx` installed. Then install `@modelcontextprotocol/server-brave-search`:

```bash
npm i -D @modelcontextprotocol/server-brave-search
```

Here’s how you can connect it in the `mcp.json` file:

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-brave-search"
      ],
      "env": {
        "BRAVE_API_KEY": ".................."
      }
    },
    "url-text-fetcher": {
      "command": "python",
      "args": [
        "-m",
        "url_text_fetcher.mcp_server"
      ]
    }
  }
}
```

You can obtain the `BRAVE_API_KEY` for free, with minor limitations of up to 2,000 requests per month and no more than one request per second.

As a result, at the bottom of the chat window in LM Studio—where the user enters their query—you can select the MCP server via “Integrations,” and you should see two MCP servers listed: “mcp/url-text-fetcher” and “mcp/brave-search.”


