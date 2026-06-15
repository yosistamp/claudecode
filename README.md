# ClaudeCode

## ローカルLLMの実行

```bash
ollama launch claude --model gemma4:12b-mlx
```

## MCP

scope: userとすることでプロジェクト二依存せずに導入。特定プロジェクトに導入する場合はscope: projectとする。

```bash
claude mcp add github --scope user -- npx -y @modelcontextprotocol/server-github
claude mcp add context7 --scope user -- npx -y @upstash/context7-mcp
```

aws

```bash
claude mcp add-json aws-mcp --scope user \
 '{"command":"uvx","args":["mcp-proxy-for-aws@latest","https://aws-mcp.us-east-1.api.aws/mcp"]}'
```

terraform

```bash
claude mcp add terraform --scope user -- docker run -i --rm hashicorp/terraform-mcp-server
```


