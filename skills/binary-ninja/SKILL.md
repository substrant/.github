---
name: binary-ninja
description: Guidance for disciplined binary analysis using Binary Ninja MCP. Helps construct accurate program models, prioritize high-value analysis paths, and avoid redundant or low-signal decompilation work.
---

# Binary analysis guidelines

Approach reverse engineering as a professional analyst building a working model of a system. Reverse engineering is not reading code; it is constructing meaning from structure.

## Analysis principles

**Analysis is controlled reduction of uncertainty.** A function is NOT understood just because it has been decompiled. It is understood when its role, inputs, outputs, and relationships are known with evidence.

**Architecture comes first.** You MUST prioritize system-level understanding over local comprehension and resist decompiling functions in isolation. Focus on:
- Execution flow before implementation detail.
- System-level structure before function-level inspection.
- Relationships between components before local logic.
- Data flow over control speculation.

**Structure and data define behavior.** Entry points, imports, exports, strings, types, and data references define the program far more reliably than function bodies. Data is not passive; inspect it before assuming meaning.
- *Native binaries:* Prioritize xrefs, memory access patterns, and calling conventions.
- *Obfuscated binaries:* Prioritize structural MCP queries, strings, and control-flow anomalies over HLIL readability.

**Execution flow carries priority.** Analyze entry points, dispatch logic, and controllers before workers and helpers. Treat functions that route execution or interpret input as primary targets. Deprioritize wrappers and leaf functions until structural nodes are understood.

## MCP discipline

Binary Ninja MCP is not a decompiler interface; it is a structural interrogation layer. If MCP can answer a question, speculation MUST NOT be used. You MUST use MCP tools deliberately:

**MUST:**
- `get_entry_points`: Mandatory at the start of analysis.
- `list_imports`: To classify capabilities before making behavioral assumptions.
- `get_callees`: To establish context before function interpretation.
- `get_xrefs_to`: For structural tracing before refactoring assumptions.
- `list_all_strings`: As a primary structural signal source.
- `get_stack_frame_vars`: Before parameter naming.
- `get_type_info`, `get_xrefs_to_type`, `get_xrefs_to_struct`: For type recovery.
- `get_data_decl`, `hexdump_data`: To understand constants, tables, and buffers.

**MUST NOT:**
- Decompile functions without a reasoned target.
- Explore helper functions before dispatch logic.
- Speculate when structural MCP queries provide the answer.

## Efficiency & anti-waste rules

Redundant MCP calls are a waste of tokens. A program is understood by reconstructing how control and data move through it, not by reading top-to-bottom.

- **Do not repeat queries:** Repeated decompilation of the same function is a failure mode. Repeated reads are disallowed unless new evidence or context exists.
- **Do not over-investigate:** Avoid analyzing already understood functions or over-investigating wrappers.
- **Deprioritize implementation:** Do not prioritize implementation detail over control flow, or treat decompilation as progress.
- **Have semantic purpose:** Do not perform MCP calls without semantic purpose. If a function does not change your model of the program, it SHOULD NOT be analyzed further.

## Naming discipline

Naming is a conclusion, not a hypothesis. It encodes understanding. You MUST NOT rename functions based on partial evidence. Do not blindly trust user naming.

You MUST prefer conservative naming until confidence increases via evidence accumulation across multiple signals. Any time you rename, it MUST be prepended with `ag_`. Renaming MUST be treated as a late-stage action, not an exploratory one.

Names for early hypotheses, partial evidence, or generic actions MUST be prefixed with `may_`:
```
ag_may_handle_network_request
ag_may_parse_header
ag_may_xor_loop
ag_may_alloc_buffer
ag_may_check_registry_key
ag_may_write_to_log
```

Strong names that show up only after cross-validation, specific mechanisms, confirmed algorithms will appear as such:
```
ag_send_dns_beacon
ag_parse_http_headers
ag_rc4_decrypt_config
ag_allocate_payload_memory
ag_check_run_key_persistence
ag_log_to_event_viewer
```

Established library patterns MUST be prefixed with `lib_` and a comprehensive identifier:
```
ag_lib_stl_vector_push_back
ag_lib_stl_string_append
ag_lib_win32_enter_critical_section
ag_lib_ntdll_rtl_enter_critical_section
ag_lib_boost_filesystem_path_append
ag_lib_openssl_evp_decrypt_init_ex
ag_lib_crt_malloc_wrapper
ag_lib_msvc_malloc
```

If names are identified as part of an identified system, they MUST be prefixed with `_sys`, the system name, and a comprehensive identifier:
```
ag_sys_json_parser_create_ast
ag_sys_json_parser_parse_node
ag_sys_task_sched_instantiate_job
ag_sys_task_sched_start
```

## Type discipline

Types MUST be recovered, not assumed. Type reconstruction MUST precede semantic labeling; without types, interpretation degrades rapidly. If a structure, enum, or buffer is unclear, you MUST prioritize type-related MCP queries over guessing.

## Multi-pass analysis

Reverse engineering MUST be conducted in iterative passes, escalating from broad structural mapping to precise semantic validation. Do not attempt to resolve all ambiguities in a single pass.

**Pass 1: First Principles & Structural Mapping**
Establish the system boundaries and foundational architecture. Interrogate entry points, imports, exports, and broad data structures without diving into local function logic. Map the ecosystem before evaluating individual organisms.

**Pass 2: Hypothesis Formation & Triage**
Identify dispatch logic, routing mechanisms, and high-value targets. Formulate early behavioral models for structural nodes. Apply conservative naming (`ag_may_...`) to track hypotheses. Trace call graphs and data flow to prioritize subsequent investigation.

**Pass 3: Validation & Semantic Resolution**
Drill down into critical implementation details for unresolved nodes. Accumulate cross-referenced evidence (xrefs, string references, data usage) to elevate hypotheses to confident conclusions. Upgrade `ag_may_...` names to strong validated names (e.g., `ag_send_dns_beacon`).

**Pass 4: Meta-Analysis & System Identification**
Correlate interacting functions to identify bounded subsystems or established libraries. Group components into cohesive systems and apply systemic labels (`ag_sys_...` or `ag_lib_...`). Validate relationships between subsystems and resolve cross-references.

## Final rule

You MUST NOT behave like a decompiler viewer. You MUST behave like an engineer constructing a complete and internally consistent model of a system.

Every MCP call MUST have intent. It MUST answer one of the following:
- What exists?
- How is it connected?
- What does it represent?
- What is the confidence level?

If it does not reduce uncertainty, validate structure, or improve semantic understanding, it MUST NOT be made.
