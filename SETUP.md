# OpenReels — TypeScript Project Setup Guide

Tooling blueprint for maximum DX.

---

## Stack

| Layer | Choice | Why |
|-------|--------|-----|
| **Runtime** | Node.js 22 LTS | Stable subprocess handling. Bun has known child process pipe issues. |
| **Package manager** | pnpm | Fast, strict dependency isolation, content-addressable store |
| **TS execution (dev)** | tsx | Zero-config, 25x faster than ts-node, ~20ms startup |
| **Bundler (dist)** | tsup | esbuild-based, simple config, produces clean ESM output |
| **Testing** | Vitest | 3-5x faster than Jest, zero-config TS, ESM-first |
| **Lint + Format** | Biome | 10-25x faster than ESLint+Prettier, single binary, single config |

---

## Project Structure

```
openreels/
├── src/
│   └── index.ts
├── package.json
├── tsconfig.json
├── tsup.config.ts
├── vitest.config.ts
├── biome.json
├── .gitignore
├── .env.example
├── LICENSE
└── README.md
```

---

## Step-by-Step Setup

### 1. Initialize

```bash
pnpm init
```

### 2. package.json

```json
{
  "name": "openreels",
  "version": "0.1.0",
  "type": "module",
  "bin": {
    "openreels": "./dist/index.js"
  },
  "engines": {
    "node": ">=22.0.0"
  },
  "scripts": {
    "dev": "tsx src/index.ts",
    "build": "tsup",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "format": "biome format --write .",
    "typecheck": "tsc --noEmit",
    "clean": "rm -rf dist"
  },
  "devDependencies": {
    "@biomejs/biome": "^2.0.0",
    "tsx": "^4.19.0",
    "tsup": "^8.4.0",
    "typescript": "^5.7.0",
    "vitest": "^3.0.0"
  }
}
```

### 3. tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022"],
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": false
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 4. tsup.config.ts

```typescript
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm"],
  dts: true,
  clean: true,
  sourcemap: true,
  banner: {
    js: "#!/usr/bin/env node",
  },
});
```

### 5. vitest.config.ts

```typescript
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: true,
    environment: "node",
    include: ["src/**/*.test.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "lcov"],
    },
  },
});
```

### 6. biome.json

```json
{
  "$schema": "https://biomejs.dev/schemas/2.0.0/schema.json",
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true
  },
  "organizeImports": {
    "enabled": true
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "complexity": {
        "noExcessiveCognitiveComplexity": "warn"
      },
      "correctness": {
        "noUnusedImports": "error",
        "noUnusedVariables": "error"
      },
      "style": {
        "noNonNullAssertion": "warn",
        "useConst": "error"
      }
    }
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "double",
      "semicolons": "always",
      "trailingCommas": "all"
    }
  },
  "files": {
    "ignore": ["node_modules", "dist"]
  }
}
```

### 7. .gitignore

```
node_modules/
dist/
*.tsbuildinfo
.env
```

### 8. Entry point

```typescript
// src/index.ts
console.log("openreels");
```

### 9. Install and verify

```bash
pnpm install
pnpm typecheck
pnpm lint
pnpm build
pnpm dev
```

---

## DX Commands

| Command | What it does |
|---------|-------------|
| `pnpm dev` | Run via tsx (no build step) |
| `pnpm build` | Build to dist/ |
| `pnpm test` | Run all tests |
| `pnpm test:watch` | Watch mode tests |
| `pnpm lint` | Lint + check formatting |
| `pnpm lint:fix` | Auto-fix lint + format |
| `pnpm format` | Format all files |
| `pnpm typecheck` | Type-check |
| `pnpm clean` | Remove dist/ |

---

## Why These Choices

**Node.js over Bun** — Bun has known subprocess pipe issues (bun#17989). Node.js 22 is the safer choice for tools that spawn child processes.

**pnpm over npm/yarn** — Faster installs, strict dependency isolation, content-addressable store saves disk space.

**tsx over ts-node** — Zero-config, 25x faster startup. ts-node requires tsconfig hooks and is measurably slower.

**tsup over raw tsc** — esbuild-based bundling produces clean single-file output with declaration files. tsc is only for type-checking.

**Vitest over Jest** — 3-5x faster, native ESM + TypeScript, zero config. The 2026 default.

**Biome over ESLint+Prettier** — 10-25x faster, single config file, single binary. No reason to use the slower option for a new project.
