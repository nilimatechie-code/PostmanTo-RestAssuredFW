# Known Limitations

This tool produces a **starting scaffold**, not a finished, drop-in-ready test suite. Review generated output before relying on it. Specifics below.

## Request bodies

- **Flat JSON only for POJOs.** If a request body has nested objects or arrays, the generated POJO field type falls back to `Object` / `List<Object>`, and the corresponding setter call in the test is left as:
  ```java
  payload.setAddress(null /* TODO: nested object/array */);
  ```
  You'll need to hand-write the nested class and wire it in.
- **Non-JSON bodies** (form-data with file uploads, GraphQL, binary) are not specifically handled. `urlencoded` bodies are converted to a raw query-string literal; anything else falls back to an empty/placeholder body.

## Authentication

- Only **Bearer token** auth is auto-detected and wired into `BaseTest.java` (via `config.properties` → `auth.token`).
- API key auth (header or query param), Basic auth, OAuth2, and AWS Signature are **not** auto-detected. If your collection uses these, you'll need to add the corresponding header/param logic to `BaseTest.java` manually.

## Test script conversion

`extractAssertionsFromScript` only recognizes these Postman/Chai patterns:

- `pm.response.to.have.status(NNN)`
- `pm.response.code === NNN`
- `pm.expect(jsonData.field).to.exist`
- `pm.expect(jsonData.field).to.eql(...)` / `.to.equal(...)`

Anything else in your `pm.test()` blocks — array length checks, regex matches, schema validation, custom JS logic, chained requests using `pm.collectionVariables.set(...)` — is **not** converted. The original test script content isn't discarded, but it isn't translated either; you'll need to port that logic manually into the generated `.then()` chain.

## Variables

- Collection-level variables (`{{baseUrl}}`, etc.) declared in the collection's top-level `variable[]` array are detected and mapped to `config.properties`.
- Variables defined only in a **Postman Environment** file (not the collection itself) are **not** read — this tool only takes the collection export, not a paired environment export. If your collection references environment-only variables, add them to `config.properties` manually after generation.
- Pre-request scripts that compute or chain variables (e.g. extracting an ID from a previous response) are not executed or translated — path/query values that reference such variables will appear as literal `{{varName}}` text or a `REPLACE_ME` placeholder.

## Structure

- Postman folders are used only to group requests during parsing — the generated project does **not** mirror folder structure as Java sub-packages. All test classes land flat in `.../tests/`. If you need folder-based package structure, this needs manual reorganization or a small extension (see `docs/ARCHITECTURE.md`).

## General

- Generated Java is not run through a compiler or formatter as part of generation — always run `mvn clean test` (or at least `mvn compile`) after generating and fix any issues before treating the suite as reliable.
- Field/class names are derived from Postman request names and JSON keys via simple casing rules. Unusual names (emoji, non-ASCII, heavy symbol use) may produce awkward but still valid Java identifiers — check generated class names for sanity.
