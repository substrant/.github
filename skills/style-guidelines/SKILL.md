---
name: style-guidelines
description: Guidance for consistent coding standards. Helps ensure implementation aligns with specifications and universal architectural rules without relying on default or templated code generation.
---

# Semantic Code Style Guidelines

Approach formatting as a senior principal engineer enforcing architectural integrity. Every line of code MUST have a reason. Make deliberate, opinionated choices about structure and style, and take the time to ensure your implementation adheres strictly to the universal rules below.

## Design principles

Formatting is not taste; it is communication. A strict, consistent house style MUST be applied to all languages in the repository. Match the surrounding code; you MUST NOT reformat to personal preference.

**Braces carry the structure.** K&R braces MUST be used: the opening brace MUST be on the same line as the enclosing declaration or control keyword, and the closing brace MUST be flush-left at the enclosing indent. Cuddled control flow blocks MUST NOT be used. The closing brace of the preceding block MUST sit on its own line, and subsequent blocks (e.g., `else`, `catch`) MUST begin on the next line. Single-line constructs like `} else {` MUST NOT be used. Folded function signatures (multi-line function signatures) is HIGHLY
DISCOURAGED without valid reason. Folded function calls MAY be used. Comments before braced statements are RECOMMENDED and MUST NOT be written after the opening brace. Inline comments MAY be used.

**Whitespace is rhythm.** 4-space indentation MUST be used. Blank lines are RECOMMENDED to separate logical groups between members, methods, control blocks, and operational phases. A good engineer chunks a file by its blank-line rhythm. Do not pack statements together or stack members with no separation.

**Naming is architecture.** Type names, methods, properties, enums, enum members, locals and parameters MUST be consistent. Multi-statement bodies MUST ALWAYS be braced, though single-expression guard clauses MAY be a single unbraced line. Prefer expression-bodied forms for one-liners. Redundant access modifiers are DISCOURAGED.

**Documentation is strictly scoped.** Public API SHOULD be documented. Internal implementations MUST NOT be cluttered with redundant comments. Related members SHOULD be grouped using block comments and section dividers, encoding something true about the code structure rather than merely decorating it.

**The developer always knows better.** NEVER attempt to apply style guidelines on code that already exists unless otherwise instructed to do so explicitly. The developer will tell you where to apply code styles. ALWAYS remember developer critique on code styling. Styling is not logic, it's a frame for logic.

## Language and dependency paradigms

Languages and libraries have their own vernacular, and rules appear to serve the paradigm. Bring the same intentionality to specific features that you would bring to universal formatting. Before writing anything, ask what it requires, and how it can best be utilized to enforce the style guidelines.

**Managed / VM-Oriented Languages:** Write from the compiler's side of the screen. Default access modifiers MUST be omitted if implied. Type inference (e.g., `var`) SHOULD be preferred for locals; explicit types SHOULD be used only where readability is aided. File-scoped namespaces MUST be used where supported. Primary constructors MUST be used for classes whose construction primarily captures parameters. Unsafe blocks MAY be permitted for FFI pointer marshalling, but MUST NOT be used for general logic.

**Typed Scripting Languages:** Strict compiler configurations (e.g., strict typing, verbatim module syntax, no unchecked indexed access) MUST be enabled. Type-only imports MUST be isolated from runtime imports. Immutable bindings MUST be preferred by default; mutable bindings MUST be used only when reassignment is required. Private fields MUST use native private-name syntax where encapsulation matters.

**Systems / Ownership-Oriented Languages:** Error propagation MUST be preferred over throwing exceptions. Redundant comments MUST NOT be added. Standard formatting tools SHOULD be followed, except where they conflict with universal rules. If standard formatting forces cuddling, house style MUST take precedence.

## Design examples

Good C#:
```csharp
namespace Example.Application;

internal class DataProcessor : IService {
    readonly IStore _store;
    int _saveCount = 0; // Prefer inline construction for fields

    // Attributes ALWAYS on prior line.
    [IProperty]
    public int SaveCount => _saveCount;

    // NEVER fold parameters
    public DataProcessor(IStore store) {
        _store = store;
    }

    /// <summary>
    /// Processes the specified payload and returns the result.
    /// </summary>
    /// <exception cref="ArgumentNullException">A null argument was provided.</exception>
    public async Task<Result> Process(Payload payload, CancellationToken cancelToken = default) {
        if (payload == null) throw new ArgumentNullException(nameof(payload));

        // Comment before braced statement RECOMMENDED
        if (payload.IsValid) {
            Validate(payload); // MAY use inline comments
            Transform(payload);
        }
        else {
            Reject(payload);
        }

        var res = await _store.Save(payload, cancelToken);
        return res;
    }

    private void Validate(Payload payload) => payload.CheckIntegrity();
}
```

Bad C#:
```csharp
namespace Example;

internal class DataProcessor
{
    [IProperty] public int Count => 0; // Attribute on same line

    // Folded signatures DISCOURAGED unless using 'where'
    public void Process(
        Payload p,
        CancellationToken c
    ) {

        if (p.IsValid) { // Comment after opening brace
            DoThing();
        } else { // Cuddled else
            Fail();
        }
    }
}
```

Good C++:
```cpp
#include <stdexcept>
#include <vector>

namespace example_app::data {
    /// @brief Processes the data buffer and returns a status code.
    /// @param buffer The data buffer to process.
    /// @return 0 on success, non-zero on failure.
    int process_data(data_buffer& buffer) {
        if (buffer.empty()) throw std::invalid_argument("Buffer is empty");

        if (buffer.is_valid()) {
            transform(buffer);
            store(buffer);
        }
        else {
            reject(buffer);
        }

        return 0;
    }

    void transform(data_buffer& buffer) {
        buffer.scale(2.0);
    }
}
```

Bad C++:
```cpp
// Missing public API doc
int processData(DataBuffer& b) {
    if (b.is_valid()) { Transform(b); } else { Reject(b); } // Packed & cuddled
    return 0;
}

// Redundant internal comments
// Scales buffer
void Transform(DataBuffer& b) { b.Scale(2.0); }
```

Good TypeScript/JavaScript:
```typescript
/**
 * Retrieves the configuration state for the specified session.
 * @param sessionId - The unique identifier for the session.
 * @returns The current configuration state.
 */
export async function getConfig(sessionId: string): Promise<ConfigState> {
    if (sessionId == null) throw new Error("Session ID is required");

    if (sessionId.startsWith("admin_")) {
        validateAdmin(sessionId);
    }
    else {
        validateUser(sessionId);
    }

    const state = await loadState(sessionId);
    return state;
}

// Internal implementations MUST NOT be cluttered.
async function loadState(id: string): Promise<ConfigState> {
    const res = await globalStore.fetch(id)
        .then(value => ({ value }))
        .catch(error => { error });
    
    return { success: res.error != undefined, ...res };
}
```

Bad TypeScript/JavaScript:
```typescript
function GetConfig( // Folded signature
    id: string
) {
    if (id == null) throw new Error();
    if (id.startsWith("a_")) { doA(); } else { doB(); } // Cuddled & packed
    return load(id);
}
export { GetConfig };
```

Good Rust:
```rust
pub struct DataPayload {
    content: String,
}

pub fn process_payload(payload: &mut DataPayload) -> Result<(), ProcessingError> {
    // Single-expression guard clauses MAY be unbraced.
    if payload.content.is_empty() return Err(ProcessingError::EmptyPayload);

    if payload.content.len() > 100 {
        transform(payload)?;
    }
    else {
        validate(payload)?;
    }

    Ok(())
}

fn transform(payload: &mut DataPayload) -> Result<(), ProcessingError> {
    payload.content.make_ascii_uppercase();
    Ok(())
}
```

Bad Rust:
```rust
// No section dividers
pub fn process(p: &mut Data) -> Result<(), Error> {
    if p.len() > 100 { transform()?; } else { validate()?; } // Cuddled & packed
    Ok(())
}

// Make uppercase
fn transform(p: &mut Data) -> Result<(), Error> { p.make_upper(); Ok(()) }
```

## Design caveats

**Nested braced statements.** Only the deepest statements can be non-braced in a simple nest.
```
if statement_1 {
    if statement_2 {
        if statement_3
            /* this code is non-braced */
    }

    if statement_4
        /* this code is non-braced */
}
```

**Multi-line function signatures and calls.** Only do when one-line function signatures are obnoxious (>200 characters).
```
typ_0 func(
    typ_1 arg_0,
    typ_2 arg_1,
    typ_3 arg_2 
) [where ...] {

    /* write code after blank line */
}
```