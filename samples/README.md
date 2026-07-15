# Sample Collections

Use these to try the converter before pointing it at a real API.

| File | Requests | Demonstrates |
|---|---|---|
| `sample-kaas-collection.json` | Create User (POST), Get User By Id (GET), Update User (PUT), List Users (GET), Delete User (DELETE) | Path variables, query params (incl. a `response` param), Bearer auth, mixed assertion types |
| `product-crud-collection.json` | Create/Get/List/Update/Partially-Update/Delete Product | Full CRUD including `PATCH` with a partial body |

Both use `{{baseUrl}}` as a collection variable and `bearer` auth on write operations, so you can also use them to sanity-check the generator's auto-detection of base URL and auth pattern.
