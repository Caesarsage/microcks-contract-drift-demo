# Contract drift, caught on the diff

A deliberately small repo that does one thing: run a [Microcks](https://microcks.io) contract test
on every pull request, and annotate the OpenAPI file when the service stops matching it.

It exists to demonstrate that the test you run locally in the
[VS Code extension](https://github.com/Caesarsage/microcks-vscode) is the same test CI runs — same
CLI, same spec, same runner, no Microcks server to maintain in either place.

## What's here

```
api/ecommerce-api-openapi.yml   the contract
server/                          a zero-dependency Go service implementing it
.github/workflows/               the contract test, on every PR
```

## The contract test

```bash
microcks test --dry-run --output=github-actions \
  --artifact api/ecommerce-api-openapi.yml \
  --filteredOperations '["GET /products"]' \
  "E-Commerce Platform API:2.0.0" \
  http://localhost:3001 \
  OPEN_API_SCHEMA
```

`--dry-run` starts an ephemeral Microcks in a container, imports the spec, runs the test, and tears
the container down. No server, no Keycloak, nothing to clean up. `--output=github-actions` turns each
failing operation into an annotation carrying the spec's path and the line the operation is declared
on, so GitHub renders it against the file rather than leaving it in the job log.

## Running it locally

```bash
go -C server run .        # conforming
go -C server run . -drift # returns price as a string, violating the contract
```

Then run the command above, or open `api/ecommerce-api-openapi.yml` in VS Code with the Microcks
extension installed and run **Microcks: Run Dry-Run for API File**.

## Requirements

Docker or Podman, Go 1.22+, and a Microcks CLI containing the `github-actions` output formatter —
on `master` today, not yet in a tagged release (latest is 1.0.2):

```bash
go install github.com/microcks/microcks-cli@master
```
