---
name: repo-architecture
description: Repository architecture guidance. Enforces standard repository layout, project organization, file placement, and ownership boundaries. Prevents ad-hoc directory structures and ensures repositories remain predictable across teams.
---

# Repository Architecture Guidelines

Approach repository structure as a staff engineer responsible for long-term maintainability. Directories communicate ownership, dependency boundaries, and deployment intent. Every file MUST have an intentional home.

Repository organization is architecture. The repository MUST be organized according to the universal structure below unless the developer explicitly instructs otherwise.

## Core philosophy

**Consistency is more valuable than novelty.** A developer should be able to clone any repository in the organization and immediately understand where source code, projects, dependencies, documentation, and build outputs belong.

**The repository structure MUST be treated as a contract.** New directories MUST NOT be introduced without a clear architectural purpose.

**The developer always knows better.** Existing repository conventions or abrupt user changes in conventions take precedence over these guidelines. Never reorganize a repository unless explicitly instructed.

## Repository root

The root of the repository SHOULD remain minimal:

```text
Root/
├── build/
├── dependencies/
├── docs/
├── project/
├── services/
├── README.md
├── [project files]
└── [repository-wide files]
```

### Root directory rules

#### build/

Build outputs, generated artifacts, packaging outputs, temporary release assets, and generated bundles MUST reside here:

```text
build/
├── project-1/
└── project-2/
```

Source files MUST NOT be committed into build directories. `build/` MUST be added to the
root `.gitignore`.

#### dependencies/

Project-wide, explicit third-party source dependencies, vendored libraries, and submodules SHOULD reside here:

```text
dependencies/
├── dependency-1/
└── dependency-2/
```

Internal projects MUST NOT be placed inside dependencies unless they are submodules.

#### docs/

Documentation SHOULD be isolated from source code:

```text
docs/
├── architecture/
├── guides/
├── decisions/
├── api/
└── images/
```

Large documentation collections SHOULD remain entirely within docs.

#### project/

The project directory MUST be used when the repository contains multiple projects:

```text
project/
├── my-backend/
├── frontend-name/
├── sdk/
└── shared/
```

Repository-wide project configuration SHOULD live here when appropriate:

```text
project/
├── CMakeLists.txt
└── Directory.Build.props
```

#### services/

The services directory MUST only exist when the repository contains both application projects and service projects:

```text
project/
├── admin-app/
├── dashboard/
└── command-line/

services/
├── backend/
├── discord-bot/
└── nginx/
```

Repositories containing only services SHOULD NOT introduce a services directory.

#### README.md

Every repository SHOULD contain a root README.

The README SHOULD explain:

- Repository purpose
- Minimal project layout
- Build instructions
- Development workflow
- Deployment overview

## Project files

Repository-level project files SHOULD remain at the repository root:

```text
/.gitignore
/.gitattributes
/.editorconfig
/CMakeLists.txt
/package.json
/project.slnx
```

Project files SHOULD NOT be scattered throughout unrelated directories.

## Multi-project repositories

Multi-project repositories SHOULD use `project/`:

```text
project/
├── project-one/
├── project-two/
└── project-three/
```

Directory names SHOULD use dash-separated naming:

```text
project/
├── web-api/
├── desktop-client/
└── shared-library/
```

**EXCEPTION: .NET projects MAY use namespace-style naming:**

```text
project/
├── Company.Platform.Core/
├── Company.Platform.Api/
└── Company.Platform.Web/
```

## Single-project repositories

Simple repositories SHOULD use `src/`:

```text
src/
├── [source files]
└── [source directories]
```

Introducing `project/` for a single-project repository is DISCOURAGED.

## Project structure

Each project SHOULD maintain a minimal root:

```text
project/project-name/
├── src/
├── [project file]
└── [minimal build files]
```

```text
project/web-api/
├── src/
├── package.json
└── tsconfig.json
```

```text
project/Company.Platform.Api/
├── Config/
├── Utility/
└── Company.Platform.Api.csproj
```

Project roots MUST NOT become dumping grounds.

## Source organization

The developer understands domain boundaries better than any specification. Source directories SHOULD be organized by responsibility and ownership:

```text
src/
├── middleware/
├── routes/
├── models/
├── services/
├── helpers/
└── utils/
```

```text
src/
├── plugins/
│   ├── pluginHost/
│   └── pluginService/
├── config/
└── runtime/
```

Classification SHOULD be favored over arbitrary grouping. Directories such as:

```text
misc/
stuff/
temp/
new/
old/
test2/
```

MUST NOT be introduced.

**EXCEPTION: An "/archive" directory is OPTIONAL and MAY contain phased-out code that needs to be re-integrated. This folder must exist as `/archive` in the `.gitignore` if it exist.**

## Ownership boundaries

Directory boundaries SHOULD communicate ownership.

A directory should answer one of the following:

- Who owns this code?
- What responsibility does this code serve?
- What deployment artifact contains this code?
- What dependency boundary does this code belong to?

If none of those questions are answered, the directory likely should not exist.

## Edge-case rules

### Existing repositories

Existing repository structures MUST be respected unless the developer explicitly requests restructuring. Repository consistency is more important than theoretical perfection.

### New repositories

New repositories SHOULD begin with the standard layout and evolve only when justified by actual requirements. NEVER offer to commit a repository initialization for the developer.

### File placement

When introducing a new file:

1. Prefer an existing directory.
2. Prefer an existing ownership boundary.
3. Create a new directory only when a new responsibility emerges.
4. Avoid single-file directories unless the directory itself is a meaningful boundary.

The correct location for a file is the place a future developer would instinctively search for it.
