# Postman → REST Assured Framework Generator

A browser-based tool that converts a **Postman Collection** (v2.0/v2.1 export) into a ready-to-run **REST Assured + TestNG + Maven** test automation project.

No installs, no build step, no server — it's a single static HTML file that runs entirely client-side. Your collection never leaves your machine.

![stack](https://img.shields.io/badge/stack-REST%20Assured%20%7C%20TestNG%20%7C%20Maven-orange)
![license](https://img.shields.io/badge/license-MIT-blue)

---

## What it does

Point it at a Postman collection export and it generates:

- **`pom.xml`** — REST Assured, TestNG, Jackson dependencies wired up
- **`BaseTest.java`** — shared `RequestSpecification`, loads config, auto-adds Bearer auth header if detected
- **`config.properties`** — base URL + auth token placeholders (auto-detects `{{baseUrl}}`-style collection variables)
- **One test class per Postman request** — `.pathParam()`, `.queryParam()`, headers, and body wired in automatically
- **POJOs** generated (best-effort) from JSON request bodies
- **`testng.xml`** wiring every generated class into a runnable suite
- **`README.md`** for the generated project itself

Postman `pm.test()` scripts are mapped to REST Assured assertions where the pattern is recognized:

| Postman | REST Assured |
|---|---|
| `pm.response.to.have.status(201)` | `.statusCode(201)` |
| `pm.expect(jsonData.id).to.exist` | `.body("id", notNullValue())` |
| `pm.expect(jsonData.email).to.eql("x")` | `.body("email", equalTo("x"))` |

Anything more custom in your original test scripts is left as a `// TODO` for manual porting.

## Usage

1. Download [`index.html`](./index.html) (or clone this repo)
2. Open it in any browser — double-click, no server needed
3. Export your collection from Postman: `...` menu → **Export** → Collection v2.1
4. Drop the exported JSON onto the page
5. Set `groupId` / `artifactId` / base package
6. Click **Generate Framework** → **Download project.zip**
7. Unzip and import into IntelliJ / Eclipse as a Maven project
8. Fill in `src/main/resources/config.properties` with your real base URL / token
9. `mvn clean test`

Try it first against the sample collections in [`/samples`](./samples) to see what the output looks like before pointing it at your real API.

## Sample collections

| File | Covers |
|---|---|
| [`samples/sample-kaas-collection.json`](./samples/sample-kaas-collection.json) | POST/GET/PUT/GET-list/DELETE, path variables, query params, Bearer auth |
| [`samples/product-crud-collection.json`](./samples/product-crud-collection.json) | Full CRUD incl. PATCH, partial body updates |

## Known limitations

This is a scaffold generator, not a lossless translator. It will not correctly handle, and will mark with a `TODO` instead:

- Nested JSON objects/arrays in request bodies (POJOs only generate flat fields)
- Non-bearer auth (API key, OAuth2, Basic) — only Bearer token is auto-wired
- Postman pre-request scripts (variable chaining, dynamic tokens, etc.)
- Custom/complex `pm.test()` assertions beyond status code, field-exists, field-equals
- Postman collection folders are flattened — nesting/grouping isn't preserved in package structure

Review every generated file before relying on it. See [`docs/LIMITATIONS.md`](./docs/LIMITATIONS.md) for details and workarounds.

## How it works

Everything runs in-browser:
- Parses the Postman Collection JSON (recursing through folders)
- Extracts method, URL, headers, auth, body, and test script per request
- Generates Java source as strings (POJOs, test classes, config)
- Zips the output client-side with [JSZip](https://stuk.github.io/jszip/) and triggers a download

See [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md) for a breakdown of the generation logic if you want to extend it.

## Contributing

Issues and PRs welcome — especially around:
- Nested POJO generation
- API key / OAuth2 auth detection
- JUnit 5 output mode as an alternative to TestNG

## License

MIT — see [LICENSE](./LICENSE).
