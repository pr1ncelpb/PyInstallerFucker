# 🔬 PyInstaller Fucker

> Decompile Python `.pyc` bytecode back into readable source code — supporting Python 3.10 through 3.14.

---

## ✨ Features

- **Multi-version support** — handles Python 3.10, 3.11, 3.12, 3.13, and 3.14 bytecode
- **Auto Python detection** — detects the target Python version from the `.pyc` magic number and relaunches automatically with the right interpreter if available on your system
- **High-level reconstruction** — rebuilds `if/else`, `for`, `while`, `try/except`, classes, functions, decorators, closures, and comprehensions
- **ctypes enrichment** — auto-injects missing `ctypes` type definitions, structures, and unions (`--enrich-ctypes`)
- **Import inference** — suggests missing imports based on symbols detected in the reconstructed code (`--suggest-imports`)
- **Quality analysis** — scores the reconstruction fidelity and reports decompilation coverage (`--analyze`)
- **Version compatibility matrix** — shows which bytecode features are supported per Python version (`--compat-report`)
- **300+ ctypes types** — covers Windows types, POSIX types, OpenGL, Vulkan, COM/OLE, network sockets, and more
- **Semantic fixer** — post-processes the output to fix anonymous variables, inlined lambdas, and common bytecode artifacts

---

## 📦 Requirements

- Python 3.10 or higher
- No external dependencies — standard library only

---

## 🚀 Usage

```bash
python main.py <input> <output.py> [options]
```

### Inputs accepted

| Input type | Description |
|---|---|
| `script.pyc` | Compiled Python bytecode file |
| `dump.txt` | Raw disassembly text dump |
| `source.py` | Python source (passes through the semantic fixer) |

### Options

| Flag | Description |
|---|---|
| `--verbose` / `-v` | Show detailed output and save the disassembly dump |
| `--force` | Skip Python version mismatch warnings |
| `--analyze` | Run a quality analysis after decompilation |
| `--enrich-ctypes` | Auto-inject missing ctypes structures |
| `--suggest-imports` | Suggest missing imports in the reconstructed code |
| `--compat-report` | Print the bytecode compatibility matrix across Python versions |
| `--quality-only` | Run quality analysis only on an already-decompiled file |

### Examples

```bash
# Basic decompilation
python main.py script.pyc output.py

# Full pipeline with analysis
python main.py script.pyc output.py --verbose --analyze

# ctypes-heavy binary
python main.py script.pyc output.py --enrich-ctypes --suggest-imports

# Force decompilation regardless of Python version mismatch
python main.py script.pyc output.py --force --verbose

# Analyze an already-decompiled file
python main.py output.py output_fixed.py --quality-only

# Print Python version compatibility matrix
python main.py anything.pyc out.py --compat-report
```

---

## 🗂️ Project Structure

```
decompiler/
│
├── main.py                          Entry point — CLI, argument parsing, banner
├── pipeline.py                      Orchestrates the full decompilation pipeline
│
├── maps/                            Static lookup tables and data
│   ├── ctypes_types.py              300+ Windows/C type mappings
│   ├── ctypes_categories.py         Categorized ctypes library + helper functions
│   ├── modules.py                   Known Python module symbols
│   └── opcodes.py                   Opcode maps (binary ops, compare ops, tier2 normalization)
│
├── bytecode/                        .pyc reading and disassembly
│   ├── opcode_tables.py             Per-version opcode tables (3.10 → 3.14)
│   ├── pyc_reader.py                PycCodeObject, linetable decoders
│   ├── marshal_reader.py            Custom marshal reader for all Python versions
│   └── disassembler.py              CrossVersionDisassembler + version detection + relaunch
│
├── engine/                          Core decompilation logic
│   ├── scope.py                     Data structures: Instr, StackVal, ScopeInfo, ClosureScopeTracker
│   ├── translator.py                BytecodeTranslator — stack-based instruction-to-AST translation
│   └── reconstructor.py             HighLevelReconstructor — rebuilds functions, classes, modules
│
├── postprocess/                     Output cleanup and fixing
│   ├── post_processor.py            PostProcessor — indent fixing, artifact removal
│   ├── post_processor_v5.py         PostProcessorV5 — advanced ctypes and signal cleanup
│   ├── reorder.py                   Reorders definitions for valid forward-reference ordering
│   └── semantic_fixer.py            SemanticFixer — anonymous vars, globals, lambda inlining
│
├── analysis/                        Quality metrics and reporting
│   ├── quality_analyzer.py          BytecodeQualityAnalyzer — coverage and fidelity scoring
│   ├── version_matrix.py            PythonVersionCompatibilityMatrix
│   └── fidelity_checker.py          SourceFidelityChecker + run_quality_analysis
│
└── ctypes_tools/                    ctypes-specific enrichment
    ├── structure_generator.py       CtypesStructureGenerator — injects struct/union definitions
    ├── import_inference.py          ImportInferenceEngine — detects and suggests missing imports
    └── enrichment.py                apply_ctypes_enrichment() — full ctypes enrichment pipeline
```

---

## ⚙️ How It Works

1. **Read** — the `.pyc` header is parsed to detect the Python version from the magic number
2. **Relaunch** *(optional)* — if a matching Python interpreter is found locally, the tool relaunches itself with the correct version for best results
3. **Disassemble** — the bytecode is disassembled using a cross-version disassembler that handles opcode differences between 3.10–3.14
4. **Translate** — `BytecodeTranslator` walks the instruction stream, simulates the evaluation stack, and emits Python expressions
5. **Reconstruct** — `HighLevelReconstructor` groups instructions into functions, classes, and modules and rebuilds their structure
6. **Post-process** — multiple passes clean up indentation, remove artifacts, fix anonymous variables, reorder definitions, and repair semantics
7. **Enrich** *(optional)* — missing `ctypes` structures are injected and missing imports are inferred
8. **Analyze** *(optional)* — a quality report is printed showing reconstruction coverage and fidelity score

---

## 📊 Output Quality

The `--analyze` flag prints a table like:

```
+-----------------------------+--------------------+
| Lines reconstructed         | 312                |
| Functions recovered         | 18                 |
| Classes recovered           | 4                  |
| TODO artifacts remaining    | 2                  |
| ctypes usages               | 47                 |
| Fidelity score              | 97.4%              |
| Python version detected     | 3.14               |
+-----------------------------+--------------------+
```

Lines marked `TODO (decompile):` indicate instructions that could not be fully reconstructed — usually due to version-specific opcodes or highly optimized bytecode.

---

## 📝 Notes

- Run from inside the project directory: `cd decompiler && python main.py ...`
- For best results, use the same Python version as the target `.pyc` — the tool will try to find and relaunch with it automatically
- The `--force` flag skips version checks if you want to attempt decompilation anyway
- ctypes-heavy binaries benefit significantly from `--enrich-ctypes`
