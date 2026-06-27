# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A schema-only repository: Protobuf definitions in `schema/api/v1/` are the single source of truth. `buf` generates Go (protobuf + ConnectRPC) and TypeScript clients into `gen/`, which are consumed as packages by other repos (Go module `github.com/yadaatdev/proto/gen/go`, npm package `proto` exporting `./gen/ts/*`). No application code or runtime lives here — only `.proto` and generated output.

## Workflow

After editing any `.proto`, regenerate before committing — the `gen/` tree is checked in and must stay in sync with `schema/`:

```
buf lint            # STANDARD lint rules (buf.yaml)
buf generate        # regenerates gen/go and gen/ts per buf.gen.yaml
buf breaking --against '.git#branch=main'   # check backward compatibility
```

## Conventions

- All services share package `api.v1`; `common.proto` holds shared messages (e.g. `Option`).
- One service per file, named `<Domain>Service` (e.g. `CityService`), with CRUD-style `Get`/`Create`/`Update`/`Delete`/`List` RPCs and per-RPC `XRequest`/`XResponse` messages.
- `List` returns `repeated Option` (id/value pairs) rather than full entities — used to populate dropdowns.
- Go package prefix and option overrides are set via `managed` mode in `buf.gen.yaml`; do not hand-edit `go_package`.

## Gateway

`api_config.yaml` is a Google Cloud API Gateway (`google.api.Service`) config that fronts the backend with Firebase Auth on every endpoint and routes to a Cloud Run service. It references service paths (`api.root.v1`, `api.admin.v1`) and a project ID that are placeholders to fill per deployment — it is not auto-generated from the protos here.
