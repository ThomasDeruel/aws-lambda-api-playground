# AWS SAM – Simple Lambda + API Gateway

REST API (`GET /blogs`) protected by an API key, a usage plan (throttling + quota) and a required `X-Client-Id` header. The Lambda (TypeScript, bundled with esbuild) returns `Hello World`.

## Prerequisites

- AWS CLI configured (`aws configure`)
- SAM CLI
- Node.js + esbuild (`npm install -g esbuild`)

## Build & deploy

```bash
# First deployment (creates samconfig.toml)
sam build && sam deploy --guided

# Next deployments
sam build && sam deploy
```

Dev only – redeploys automatically on every file change:

```bash
sam sync --stack-name sam-app --watch
```

## Test

Show the stack outputs (URL + API key ID):

```bash
sam list stack-outputs --stack-name sam-app
```

Store the API key value and the URL in variables:

```bash
KEY=$(aws apigateway get-api-keys --name-query sam-app-api-key --include-values --query "items[0].value" --output text)
URL=$(aws cloudformation describe-stacks --stack-name sam-app --query "Stacks[0].Outputs[?OutputKey=='UrlEndpoint'].OutputValue" --output text)
```

Call the API:

```bash
curl -i -H "x-api-key: $KEY" -H "X-Client-Id: test" "$URL"
```

Should return `200` with `{"message":"Hello World"}`.

### Expected errors

| Request | Response |
|---|---|
| Without `x-api-key` | `403 Forbidden` |
| Without `X-Client-Id` | `400 Bad Request` |
| More than 10 simultaneous requests | `429 Too Many Requests` |