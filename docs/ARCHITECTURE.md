# Architecture

Everything lives in [`index.html`](../index.html) — a single static file with inline CSS and JS. No build step, no bundler, no dependencies beyond a CDN-hosted [JSZip](https://stuk.github.io/jszip/) for the final zip export.

## Pipeline

```
Postman Collection JSON
        │
        ▼
 parseCollection()          — walks item[] recursively (handles nested folders),
        │                      extracts collection-level {{variables}}
        ▼
 parseRequestItem()         — per request: method, URL, path vars, query params,
        │                      headers, auth, body (raw JSON / urlencoded), test script
        ▼
 extractAssertionsFromScript() — regex-matches common pm.test() patterns:
        │                        status code, field-exists, field-equals
        ▼
 generateTestClass()        — builds one Java TestNG class per request
        │                      (RequestSpec chain, path/query params, assertions)
        ▼
 generatePojoFromJson()     — best-effort POJO from a flat JSON request body
        │                      (Jackson-annotated fields, getters/setters)
        ▼
 generateBaseTest() / generateConfigProperties() / generatePomXml() / generateTestNgXml()
        │
        ▼
 JSZip                      — assembles all generated strings into a zip,
                               triggers browser download
```

## Key functions (all in the `<script>` block of `index.html`)

| Function | Responsibility |
|---|---|
| `walkItems` | Recursively flattens Postman folders into a flat request list |
| `parseRequestItem` | Extracts everything needed from one Postman request item |
| `extractAssertionsFromScript` | Regex-based `pm.test()` → assertion object mapping |
| `pascalCase` / `camelCase` | Naming conversion for class/variable names — preserves existing internal casing (e.g. `userId` → `UserId`, not `Userid`) |
| `generatePojoFromJson` | Infers Java field types from a JSON sample, generates a Jackson POJO |
| `generateTestClass` | Assembles the full REST Assured `given().when().then()` chain per request |
| `generateBaseTest` | Shared `RequestSpecification` setup, reads `config.properties` |
| `generatePomXml` / `generateTestNgXml` / `generateReadme` | Supporting project files |

## Why client-side only

Keeping this as a single static HTML file (rather than a Node/Express service) means:
- Nothing to host, deploy, or keep running
- The Postman collection — which may contain internal URLs, tokens, or business logic — never leaves the browser
- Zero install friction: clone the repo, open `index.html`, done

## Extending it

The most useful places to extend:

- **`generatePojoFromJson`** — currently only handles flat JSON. Nested objects/arrays fall back to a `TODO` comment in the generated setter call. A recursive version could emit nested static classes.
- **Auth detection** (`genBtn` click handler, `authHint` logic) — currently only detects Bearer token patterns. API key (header or query) and Basic auth would follow the same detection approach.
- **Output flavor** — swapping TestNG annotations/imports for JUnit 5 would mostly touch `generateTestClass` and `generateTestNgXml` (which would become a JUnit suite or just rely on Surefire's default discovery).
- **Assertion mapping** (`extractAssertionsFromScript`) — more Postman/Chai patterns could be regex-matched here (e.g. `to.be.an('array')`, `to.have.lengthOf`).
