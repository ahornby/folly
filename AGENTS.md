# Repository Guidelines

## Project Structure & Module Organization
- `folly/` — core C++ sources and headers (subdirs by domain).
- `folly/**/test/` — unit tests and micro‑benchmarks.
- `CMake/`, `CMakeLists.txt` — CMake build configuration.
- `BUCK`, `buck2` — Buck2 targets and helper.
- `build/`, `build.sh` — fbcode_builder tooling (getdeps wrapper).

## Build, Test, and Development Commands
- Preferred (fast path via getdeps):
  - Install system deps first to avoid building them:
    - `sudo ./build/fbcode_builder/getdeps.py install-system-deps --recursive`
    - Preview only: `./build/fbcode_builder/getdeps.py install-system-deps --dry-run --recursive`
  - Build using system packages: `python3 ./build/fbcode_builder/getdeps.py --allow-system-packages build`
  - Test using system packages: `python3 ./build/fbcode_builder/getdeps.py --allow-system-packages test`
  - Wrapper: `./build.sh` invokes the same flow.
- CMake (direct):
  - Environment: export build env from getdeps before using cmake/ctest:
    - `eval "$(python3 ./build/fbcode_builder/getdeps.py env)"`
  - `cmake -S . -B _build -DCMAKE_BUILD_TYPE=Debug`
  - `cmake --build _build -j`
  - `ctest --test-dir _build --output-on-failure`
- Buck2 (CI-aligned):
  - `./buck2 targets //...`
  - `./buck2 build //folly/...`
  - `./buck2 test //folly/...`

## Coding Style & Naming Conventions
- C++: use modern C++ (C++17+), RAII, and `folly::*` utilities where suitable.
- Formatting: match surrounding code; prefer `clang-format` if available.
- Indentation: 2 spaces; avoid hard tabs.
- Filenames: headers `.h`, inline headers `-inl.h`, tests `*Test.cpp`, benchmarks `*Benchmark.cpp`.
- Naming: Types `CamelCase`; functions/methods `camelCase`; constants `kCamelCase`.

## Testing Guidelines
- Framework: GoogleTest via `folly/portability/GTest.h`.
- Location: place tests under the corresponding `folly/<module>/test/` folder.
- Conventions: one logical unit per `*Test.cpp`; prefer small, deterministic tests.
- Running: `ctest --test-dir _build` or `./buck2 test //folly/...`.
- Fast iteration with getdeps filtering:
  - `python3 ./build/fbcode_builder/getdeps.py --allow-system-packages test --filter 'UuidTest'`
  - Use a regex to narrow further, e.g. `--filter '.*Channel.*'` or a specific test binary/class.
- Using ctest directly on the getdeps build:
  - `(cd $(python3 ./build/fbcode_builder/getdeps.py show-build-dir) && ctest -R 'UuidTest' --output-on-failure)`
  - Replace `'UuidTest'` with a regex for the target test(s).
- Coverage: add/extend tests for all new behavior and bug fixes.

## Commit & Pull Request Guidelines
- Commits: imperative mood, concise subject (≤72 chars), explain “why”, reference issues (e.g., `Fixes #123`).
- PRs: clear description of change, rationale, and impact; link issues; include tests and benchmarks for perf‑sensitive changes; update docs where relevant.
- CI: ensure CMake/Buck2 builds and tests pass; avoid introducing new external deps without discussion.

## Security & Portability Tips
- Prefer `folly/portability/*` wrappers for platform APIs.
- Validate inputs and use safe containers/algorithms from Folly where possible.
- Keep public headers minimal and stable; avoid leaking internal dependencies.
