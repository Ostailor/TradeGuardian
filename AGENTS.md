Step 1: repo-bootstrap
1.1 Objective
Set up a clean, language-agnostic mono-repo with automatic lint-build-test feedback so every later agent can branch off a consistent foundation.

1.2 Tasks & Solo-Tests (list format)
1.2.1 Create the repo

Action: git init TradeGuardian && cd TradeGuardian

Immediate test: git status shows On branch main and nothing to commit.

1.2.2 Skeleton directories

Action: mkdir -p research src dash ops infra

Immediate test: ls -1 lists the five top-level folders.

1.2.3 README stubs

Action: create README.md in root and placeholder READMEs in each top-level folder.

Immediate test: grep -R "## TradeGuardian" . finds the root header.

1.2.4 .gitignore

Action: combine GitHub Python, C++, and Node templates; add /dist/ and /coverage/.

Immediate test: git check-ignore -v /tmp/foo.pyc prints the ignore rule.

1.2.5 Pre-commit configuration

Action: pre-commit init-template and add hooks:

yaml
Copy
Edit
- repo: https://github.com/psf/black
  rev: 24.4.2
  hooks: [id: black]
- repo: https://github.com/pre-commit/mirrors-clang-format
  rev: v17.0.6
  hooks: [id: clang-format]
Immediate test: pre-commit run --all-files finishes with Passed.

1.2.6 clang-format style file

Action: add .clang-format (Google style, ColumnLimit: 100).

Immediate test: clang-format --dry-run -Werror src/example.cpp prints nothing.

1.2.7 Black config

Action: add pyproject.toml with line-length = 100.

Immediate test: black --check research/ exits with code 0.

1.2.8 Python virtual-env helper

Action: add make venv target that creates .venv.

Immediate test: make venv && . .venv/bin/activate && python -V shows Python 3.x.

1.2.9 CMake bootstrap

Action: minimal infra/tooling/CMakeLists.txt to compile “hello-world”.

Immediate test: cmake -S . -B build configures without errors.

1.2.10 Node front-end init

Action: npm create next-app dash --ts --eslint --tailwind.

Immediate test: npm run --workspace=dash lint passes.

1.2.11 GitHub Action: lint-build-test

Action: create .github/workflows/lint-build-test.yml with jobs:

lint-py: black --check

lint-cpp: clang-format --dry-run

lint-js: npm run lint in dash

build-cpp: cmake .. && make -j

pytest: placeholder that always passes

Immediate test: push a branch; PR shows all jobs green.

1.2.12 Branch protection

Action: enable “Require status checks to pass” on main.

Immediate test: attempt to merge a failing commit—GitHub blocks it.

1.3 Deliverables
Committed directory tree: research/, src/, dash/, ops/, infra/

.pre-commit-config.yaml, .clang-format, pyproject.toml

GitHub Action lint-build-test visible in the Actions tab

1.4 End-to-End Self-Test
bash
Copy
Edit
# from repo root
pre-commit run --all-files               # local lint check
gh workflow run lint-build-test          # trigger CI manually
gh run list --limit 1 --status success   # confirm ✅
If each task-level test passes and the final CI run is green, Step 1 is complete, providing a solid scaffold and automated quality gates for all future work.
