# OpenReels — TypeScript Project Setup Guide

The complete tooling blueprint for maximum DX.

---

## Stack Decisions

| Layer | Choice | Why |
|-------|--------|-----|
| **Runtime** | Node.js 22 LTS | Stable subprocess handling. Bun has known pipe issues with child processes. |
| **Package manager** | pnpm | Best monorepo support, strict dependency isolation, fast |
| **TS execution (dev)** | tsx | Zero-config, 25x faster than ts-node, ~20ms startup |
| **Bundler (dist)** | tsup | esbuild-based, simple config, produces clean ESM output |
| **Testing** | Vitest | 3-5x faster than Jest, zero-config TS, ESM-first |
| **Lint + Format** | Biome | 10-25x faster than ESLint+Prettier, single binary, single config file |
| **Monorepo** | pnpm workspaces + Turborepo | Lightweight workspace isolation + build caching |

---

## Project Structure

```
openreels/
├── packages/
│   ├── cli/                     # CLI entry point
│   │   ├── src/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── tsup.config.ts
│   │   └── vitest.config.ts
│   │
│   ├── renderer/                # Remotion video renderer
│   │   ├── src/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── tsup.config.ts
│   │
│   └── shared/                  # Shared types + utilities
│       ├── src/
│       │   └── index.ts
│       ├── package.json
│       ├── tsconfig.json
│       └── tsup.config.ts
│
├── turbo.json
├── pnpm-workspace.yaml
├── biome.json
├── tsconfig.base.json
├── package.json
├── .gitignore
├── LICENSE
└── README.md
```

---

## Step-by-Step Setup

### 1. Initialize the monorepo

```bash
pnpm init

cat > pnpm-workspace.yaml << 'EOF'
packages:
  - "packages/*"
EOF
```

### 2. Root package.json

```json
{
  "name": "openreels",
  "private": true,
  "packageManager": "pnpm@9.15.0",
  "engines": {
    "node": ">=22.0.0"
  },
  "scripts": {
    "dev": "pnpm --filter @openreels/cli dev",
    "build": "turbo build",
    "test": "turbo test",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "format": "biome format --write .",
    "typecheck": "turbo typecheck",
    "clean": "turbo clean"
  },
  "devDependencies": {
    "@biomejs/biome": "^2.0.0",
    "turbo": "^2.4.0",
    "typescript": "^5.7.0"
  }
}
```

### 3. TypeScript base config

```jsonc
// tsconfig.base.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022"],
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
  }
}
```

### 4. Turborepo config

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "dev": {
      "dependsOn": ["^build"],
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "typecheck": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "clean": {
      "cache": false
    }
  }
}
```

### 5. Biome config

```json
// biome.json
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
    "ignore": ["node_modules", "dist", ".turbo"]
  }
}
```

### 6. .gitignore

```
node_modules/
dist/
.turbo/
*.tsbuildinfo
.env
```

### 7. Shared package

```bash
mkdir -p packages/shared/src
```

```json
// packages/shared/package.json
{
  "name": "@openreels/shared",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "scripts": {
    "build": "tsup",
    "dev": "tsup --watch",
    "typecheck": "tsc --noEmit",
    "clean": "rm -rf dist"
  },
  "devDependencies": {
    "tsup": "^8.4.0",
    "typescript": "^5.7.0"
  }
}
```

```typescript
// packages/shared/tsup.config.ts
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm"],
  dts: true,
  clean: true,
  sourcemap: true,
});
```

```jsonc
// packages/shared/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src/**/*"]
}
```

```typescript
// packages/shared/src/index.ts
export {};
```

### 8. CLI package

```bash
mkdir -p packages/cli/src
```

```json
// packages/cli/package.json
{
  "name": "@openreels/cli",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "bin": {
    "openreels": "./dist/index.js"
  },
  "scripts": {
    "dev": "tsx src/index.ts",
    "build": "tsup",
    "test": "vitest run",
    "test:watch": "vitest",
    "typecheck": "tsc --noEmit",
    "clean": "rm -rf dist"
  },
  "dependencies": {
    "@openreels/shared": "workspace:*"
  },
  "devDependencies": {
    "tsx": "^4.19.0",
    "tsup": "^8.4.0",
    "typescript": "^5.7.0",
    "vitest": "^3.0.0"
  }
}
```

```typescript
// packages/cli/tsup.config.ts
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

```jsonc
// packages/cli/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src/**/*"]
}
```

```typescript
// packages/cli/vitest.config.ts
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

```typescript
// packages/cli/src/index.ts
console.log("openreels");
```

### 9. Renderer package

```bash
mkdir -p packages/renderer/src
```

```json
// packages/renderer/package.json
{
  "name": "@openreels/renderer",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "scripts": {
    "build": "tsup",
    "dev": "tsup --watch",
    "test": "vitest run",
    "typecheck": "tsc --noEmit",
    "clean": "rm -rf dist"
  },
  "dependencies": {
    "@openreels/shared": "workspace:*"
  },
  "devDependencies": {
    "tsup": "^8.4.0",
    "typescript": "^5.7.0",
    "vitest": "^3.0.0"
  }
}
```

```typescript
// packages/renderer/tsup.config.ts
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm"],
  dts: true,
  clean: true,
  sourcemap: true,
});
```

```jsonc
// packages/renderer/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "jsx": "react-jsx",
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src/**/*"]
}
```

```typescript
// packages/renderer/src/index.ts
export {};
```

### 10. Install and verify

```bash
pnpm install
pnpm build
pnpm typecheck
pnpm lint
pnpm dev
```

---

## DX Commands

| Command | What it does |
|---------|-------------|
| `pnpm dev` | Run CLI via tsx (no build step) |
| `pnpm build` | Build all packages (Turborepo cached) |
| `pnpm test` | Run all tests across packages |
| `pnpm lint` | Lint + check formatting |
| `pnpm lint:fix` | Auto-fix lint + format issues |
| `pnpm format` | Format all files |
| `pnpm typecheck` | Type-check all packages |
| `pnpm clean` | Remove all dist/ and .turbo/ |
| `pnpm --filter @openreels/cli test:watch` | Watch tests in a single package |

---

## Why These Choices

**Node.js over Bun** — Bun has known subprocess pipe issues (bun#17989). Node.js 22 is the safer choice for tools that spawn child processes.

**pnpm over npm/yarn** — Strict dependency isolation prevents version conflicts across workspace packages. Content-addressable store saves disk space.

**tsx over ts-node** — Zero-config, 25x faster startup. ts-node requires tsconfig hooks and is measurably slower.

**tsup over raw tsc** — esbuild-based bundling produces clean single-file output with declaration files. tsc is used only for type-checking.

**Vitest over Jest** — 3-5x faster, native ESM + TypeScript support, zero config. The 2026 default.

**Biome over ESLint+Prettier** — 10-25x faster, single config file, single binary, zero npm dependencies. No reason to use the slower option for a new project.

**Turborepo over Nx** — Lightweight, simple JSON config, excellent for JS/TS monorepos. Nx is more powerful but adds complexity that isn't needed here.
