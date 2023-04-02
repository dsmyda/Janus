# openquery

OpenQuery is an open source library that automates the most mundane analytics questions. It works by synthesizing your database schema and using that to generate candidate queries. You can opt-in to automatically run those queries for a nice end-to-end experience.

OpenQuery is designed with information security in mind, and ships with effective data masking techniques for both your schema and natural language queries.

As part of this initial release, only OpenAI and Postgres are supported.

demo: TBD

## Features

- SQL validation & correction
- Data masking

## How it works

TODO - Add a diagram

## Best Practices

### Least Privilege

Please only give OpenQuery the least amount of privilege required to answer your questions. Our recommendation

1. Create a seperate database user
2. Supply a read-only connection
3. Restrict who has access to OpenQuery's APIs

### Smallest Synthesis

It's recommended that you synthesize subsets of your database schema to answer specific and/or frequent questions. This reduces the size of the prompt sent to OpenAI, which will save you money
and improve the accuracy of the results.

## HTTP API 

### POST /api/synthesize 

Digest and cache a subset of your database schema. 

OpenQuery will fetch tables matching the filters. The results will be saved to disk and can be referenced when calling `/api/question`. See `/api/question` for more details.

Request Body

```typescript
{
  name: string, // must be unique 
  filters?: {
    schemas?: string[] // defaults to ['public']
    includeTables: RegExp[], // defaults to ["/*/"]
    excludeTables: RegExp[], // defaults to [] 
  }
}
```

Example
```sh
curl -X POST -H "Content-Type: application/json" -d "{ \"name\": \"public\" }" http://localhost:3000/api/synthesize
```

### POST /api/question

Ask a basic analytics question. The query used to produce the results will be attached, for review and verification.

If `synthName` is not passed, then it'll call `/api/synthesize`. To lower usage costs and improve accuracy, it's recommended you synthesize the smallest subset of tables required to answer the question.

If OpenAI returns multiple responses (default 3, configurable using the `openaiParams` field), then the first query that succeeds will be taken.

Request body

```typescript
{
  question: string,
  synthName?: string, // makes a default call to /api/synthesize, but doesn't persist the result.
  runQuery?: boolean, // defaults to true 
  openaiParams?: Record<string, any> // defaults to { "model": "text-davinci-003", "temperature": 1, "n": 3, "max_tokens": 32 } 
}
```

Example

```sh
curl -X POST -H "Content-Type: application/json" -d "{ \"synthName\": \"public-schemas\", \"question\": \"How many employees were hired in 2003?\" }" http://localhost:3000/api/question
```

## Slack

TODO

## Discord
