> 目录已整理：文档在「项目文档」，构建、缓存与暂存输入在「Build」。从仓库根目录运行 `python3 构建.py --build`；如需使用本文原有源码命令，先运行 `python3 构建.py --stage --ci`，再进入 `Build/源码`。暂存会恢复原输入路径。现有版本和历史验证记录按各自提交理解。

# A64Dispatch

A C++20 library and CLI for AArch64 dispatch analysis, control-flow reports and
same-size branch rewriting gated by known-vector and coverage checks.
Version 1.0.1 is the current macOS arm64 source release. It includes later
fixes and the Build/项目文档 layout; the 1.0.0 delivery records below are historical.
Build, tests and installation for this release are checked by the exact-commit
GitHub macOS workflow. No prebuilt binary is distributed with this release.
The [validation summary](<validation/README.md>) records the local acceptance
scope and explains availability of the detailed evidence.

The CLI produces reports and candidate snapshots without overwriting input files.
The optional IDAPython bridge exports an IDA database, asks the native program to
analyze it, and submits accepted byte edits or candidate graph references on IDA's main thread.
This is a new workflow and configuration schema, not a drop-in Python API replacement.

The current local Release result is **742 checks**. Debug and ASan/UBSan each
passed **472 checks in the seven groups touched by the latest fixes**, not every
group. After those fixes, IDA 9.4 passed **26 checks** on one owned fixture in a
fresh disposable database, using the Debug binary. That host run is not a Release
test and does not cover other databases. The
[hosted Release run at `0ea2ca9`](https://github.com/dhtfish-98/A64Dispatch/actions/runs/36086125433)
passed 707 checks and the installed consumer; it predates these fixes and did not
run IDA. Finite checks do not prove that a rewritten branch matches every input.

The native library currently provides mapped ELF64/snapshot/flat-file inputs, instruction
semantics, bounded constant and binary-choice tracking, two-level and single-level
table dispatch analysis, experimental comparison-tree recovery, static and observed
target grading, native Unicorn execution, branch planning and an in-memory rewrite
transaction. The CLI now includes source-bound staged artifacts, full workflow, graph ownership,
restore/regression, command oracles, bounded libc call models and cleanup proposals.
Comparison-tree conditional back edges, bounded state expansion, table graph
candidates, external traces, independent batch jobs and regression expectations
are implemented and exercised. See the [migration inventory](<docs/MIGRATION.md>).

Candidate verification executes the original and modified image separately, checks
known outputs and requires coverage of changed instructions and the proposed
indirect-branch targets. Finite examples do not establish equivalence for all inputs.
The library rejects unsupported reasoning cases instead of substituting a guess.

## Development build

Validated local dependencies are available with Homebrew:

```sh
brew install cmake ninja pkgconf nlohmann-json libgcrypt capstone unicorn yaml-cpp llvm lld
cmake --preset debug
cmake --build --preset debug
ctest --preset debug --verbose
```

The test build compiles the neutral AArch64 assembly in `samples/dispatch_cases.S`
using a cross-target LLVM compiler and ELF linker. Counts and scope for the
current 742-check Release run, the 26-check Debug IDA session and the earlier
707-check hosted run are stated above. Their records are the
[re-audit](<validation/re-audit-2026-09-25/result.json>), the
[latest IDA run](<validation/ida-re-audit-2026-09-25.json>) and the
[hosted summary](<validation/hosted-release-2026-09-25.json>). Earlier IDA sessions
remain in [validation/README.md](<validation/README.md>). Linux/Windows execution
is OPEN. An independent installed C++ consumer has executed the library and exact
graph restoration.

Current development commands:

```sh
build/debug/a64-dispatch decode 0xb8a95948 0x100000
build/debug/a64-dispatch snapshot build/debug/dispatch_cases.elf snapshot.json
build/debug/a64-dispatch resolve --image build/debug/dispatch_cases.elf --config samples/settings.json --output analysis.json
build/debug/a64-dispatch run --config samples/workflow.json --output build/preview.json
build/debug/a64-dispatch run --config samples/workflow.json --apply --output build/applied.json
build/debug/a64-dispatch regress --config samples/workflow.json --output build/regression.json
build/debug/a64-dispatch restore --config samples/workflow.json --from build/applied.json --output build/restored.json
```

`survey` and `classify` select earlier stages. `--from` checks a prior stage against
fresh source/configuration analysis. The CLI returns separate snapshots; the thin
IDA adapter explicitly submits verified changes to its database. Read
[configuration and protocol details](<docs/CONFIG.md>), the [IDA bridge guide](<integrations/ida/README.md>),
and [validation record](<docs/CHECKPOINT.md>).

## Licensing

Copyright 2026 dhtfish98, licensed under GPL-2.0-only. The native build links
Unicorn; this distribution keeps that GPL license together with the dependency
notices. No third-party target binaries or private target data are included in
the source tree.

## Install and use the library

```sh
cmake --preset release
cmake --build --preset release
ctest --preset release --verbose
cmake --install build/release --prefix "$PWD/dist/A64Dispatch-macos-arm64"
cmake -S examples/consumer -B build/consumer -DCMAKE_PREFIX_PATH="$PWD/dist/A64Dispatch-macos-arm64"
cmake --build build/consumer
build/consumer/consumer build/release/dispatch_cases.elf
```

The installed CMake target is `A64Dispatch::a64dispatch`. Native binary archives
include the program, worker, static library, headers, host adapter, docs, examples
and licenses. Homebrew runtime dependencies are installed separately; the archive
is not a self-contained macOS application. See dependencies.lock.json and the
runtime dependency record in the delivery evidence.

Local validation and publication scope: [validation/README.md](<validation/README.md>).
