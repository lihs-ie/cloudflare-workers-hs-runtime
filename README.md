# @cloudflare-workers-hs/runtime

[日本語](README.ja.md)

Typed ESM bridges between GHC WASM reactors and Cloudflare Workers. The package initializes WASI, validates reactor exports, and connects Worker, Durable Object, and Workflow entrypoints while application behavior remains in Haskell.

## Trial installation

The package is not published to npm yet. Pin an exact Git commit during the trial period:

```sh
pnpm add 'github:lihs-ie/cloudflare-workers-hs-runtime#COMMIT_SHA'
```

Consumers do not run `pnpm pack`. pnpm installs the Git dependency and the package `prepare` script builds `dist` automatically. Replace `COMMIT_SHA` deliberately when accepting a runtime update.

## Connect a reactor

```ts
import {
  bindExport,
  createReactor,
  decodeResponse,
  defineWorker,
} from "@cloudflare-workers-hs/runtime";
import makeImports from "./application-jsffi.mjs";
import wasmModule from "./application.wasm";

const reactor = await createReactor(wasmModule, makeImports, (exports) => ({
  fetch: bindExport<[request: Request, env: Env, context: ExecutionContext], Response>(
    exports,
    "fetch",
    decodeResponse,
  ),
}));

export default defineWorker({ fetch: reactor.fetch });
```

Run `pnpm exec wrangler types` in the application and include the generated declaration file. Its `Env` and `ExecutionContext` describe the real Cloudflare configuration. Generated JSFFI and WASM imports also need declarations in the application.

`createReactor` validates `memory`, `_initialize`, and the optional GHC scheduler export before constructing the adapter. `bindExport` requires a callable export and decodes every resolved result. The package supplies `decodeResponse`, `decodeString`, and `decodeVoid`; application-specific decoders should validate `unknown` inputs.

## Platform helpers

| API                    | Purpose                                                              |
| ---------------------- | -------------------------------------------------------------------- |
| `defineWorker`         | Validate and freeze fetch, queue, scheduled, and tail handlers.      |
| `initializeObject`     | Register Durable Object initialization with `blockConcurrencyWhile`. |
| `runObject`            | Run an object operation with optional native serialization.          |
| `decodeWorkflowResult` | Validate a Workflow result envelope.                                 |
| `unwrapWorkflowResult` | Return success or preserve native and non-retryable failures.        |

Durable Object and Workflow classes remain explicit Cloudflare subclasses. This keeps generated Cloudflare types available at every entrypoint and avoids unchecked type assertions.

## Development

Node 24 LTS and pnpm 12 are required.

```sh
corepack pnpm install
pnpm check
```

`pnpm check` runs oxfmt, typed Oxlint, TypeScript 7 checks, TSDoc validation, Vitest coverage, build, publint, and Are the Types Wrong. Runtime source has a 100% threshold for statements, branches, functions, and lines. `pnpm pack` builds a normal npm tarball when one is needed for release validation.

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution rules and [SECURITY.md](SECURITY.md) for private vulnerability reporting.
