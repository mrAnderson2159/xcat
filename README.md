# xcat

**Build clean, structured LLM context files from your codebase.**

`xcat` is a command-line tool that concatenates files and directories into a single readable text output.

It is designed for developers who work with LLMs and need a fast, reliable way to share project context without manually copying files one by one.

```bash
xcat src tests README.md pyproject.toml --python -o context.txt
```

The generated file keeps every file clearly separated:

```text
----------> src/main.py <----------

...

----------> tests/test_main.py <----------

...

----------> README.md <----------

...
```

## Why xcat?

When you ask an LLM to help with a real project, one file is often not enough.

You may need to share:

- source code
- tests
- configuration files
- package metadata
- documentation
- selected folders from a larger repository

Copying those files manually is slow. Dumping the entire repository often includes too much noise. Uploading random archives is not always convenient.

`xcat` gives you a terminal-native middle ground:

> select the files and directories you care about, then generate one clean context file

## Installation

```bash
pip install xcat-llm
```

This installs the `xcat` command:

```bash
xcat --help
```

For isolated CLI installation, you can also use `pipx`:

```bash
pipx install xcat-llm
```

or `uv`:

```bash
uv tool install xcat-llm
```

## Usage

```bash
xcat PATH [PATH ...] [OPTIONS]
```

`PATH` can be a file or a directory.

Directories are scanned recursively.

## Quick start

Scan the current directory:

```bash
xcat .
```

Write the output to a file:

```bash
xcat . -o context.txt
```

Scan a Python project:

```bash
xcat . --python -o context.txt
```

Scan only selected parts of a project:

```bash
xcat src tests README.md pyproject.toml --python -o context.txt
```

Overwrite an existing output file explicitly:

```bash
xcat . --python -o context.txt -f
```

## Examples

### Build a Python context file

```bash
xcat src tests README.md pyproject.toml --python -o context.txt
```

### Build a React context file

```bash
xcat src package.json vite.config.ts README.md --react -o context.txt
```

### Build a C#/.NET context file

```bash
xcat . --csharp -o context.txt
```

### Ignore custom files or directories

```bash
xcat . --python -I "*.log" -I "private" -I "secrets.py" -o context.txt
```

### Use `.gitignore`

```bash
xcat . --gitignore -o context.txt
```

### Use a custom ignore file

```bash
xcat . --gitignore .xcatignore -o context.txt
```

### Write to stdout

If `-o/--output` is not provided, `xcat` writes to stdout:

```bash
xcat src --python
```

You can still use shell redirection:

```bash
xcat src --python > context.txt
```

For normal use, `-o/--output` is recommended because `xcat` can protect existing files from accidental overwrites.

## Options

| Option | Description |
|---|---|
| `PATH [PATH ...]` | One or more files or directories to scan |
| `-I, --ignore PATTERN` | Ignore a file, directory, path, or glob-style pattern. Can be repeated |
| `-o, --output PATH` | Write output to a file instead of stdout |
| `-f, --force` | Overwrite the output file if it already exists |
| `--gitignore [PATH]` | Load ignore rules from `.gitignore` or from a custom ignore file |
| `--base` | Apply the base ignore preset |
| `--python` | Apply Python-oriented ignore rules |
| `--node` | Apply Node.js-oriented ignore rules |
| `--react` | Apply React-oriented ignore rules |
| `--csharp` | Apply C#/.NET-oriented ignore rules |
| `-a, --all, --ignore-all` | Apply all available ignore presets |

## Output format

Each file is rendered with a clear header:

```text
----------> path/to/file.py <----------

file content
```

Example:

```bash
xcat src README.md pyproject.toml --python -o context.txt
```

Output:

```text
----------> src/main.py <----------

print("hello")

----------> README.md <----------

# Project

----------> pyproject.toml <----------

[project]
name = "example"
```

The output is plain text, so it can be:

- opened in any editor
- copied into an LLM chat
- uploaded as a context file
- committed temporarily for debugging
- piped into other command-line tools

## Relative headers

`xcat` avoids leaking local absolute paths.

Instead of printing paths like:

```text
/home/user/projects/my-app/src/main.py
```

it prints paths relative to the deepest common parent shared by the input paths:

```text
src/main.py
```

With multiple roots:

```bash
xcat src README.md pyproject.toml -o context.txt
```

the output keeps useful project structure:

```text
----------> src/main.py <----------

...

----------> README.md <----------

...

----------> pyproject.toml <----------

...
```

## Multiple input paths

`xcat` accepts multiple files and directories in a single command:

```bash
xcat src tests README.md pyproject.toml --python -o context.txt
```

This is useful when you want a focused context file instead of a full repository dump.

Input paths are processed in the order they are provided.

If paths overlap, files may appear more than once. For example:

```bash
xcat project project/src
```

Since `project/src` is already inside `project`, files under `project/src` may be emitted twice.

`xcat` keeps this behavior explicit and predictable.

## Ignore rules

Ignore rules can come from:

1. built-in presets
2. repeated `-I/--ignore` options
3. `.gitignore` or a custom ignore file

Example:

```bash
xcat . --python --gitignore -I "*.txt" -I "private" -o context.txt
```

Ignore patterns support common matching styles:

```text
__pycache__
*.pyc
private
src/private
src/private/**
```

## Built-in presets

### `--base`

Ignores common noise:

```text
.git
.venv
__pycache__
*.pyc
*.pyo
*.egg-info
.idea
.vscode
.DS_Store
Thumbs.db
desktop.ini
```

### `--python`

Extends `--base` with common Python artifacts:

```text
.pytest_cache
.mypy_cache
.ruff_cache
dist
build
htmlcov
.coverage
```

### `--node`

Extends `--base` with common Node.js artifacts:

```text
node_modules
dist
build
coverage
```

### `--react`

Extends `--node` with common React tooling artifacts:

```text
.vite
.next
```

### `--csharp`

Extends `--base` with common C#/.NET and IDE artifacts:

```text
bin
obj
.vs
*.suo
*.user
*.csproj
*.resx
packages
*.nupkg
TestResults
coverage
```

### `--all`

Applies all available presets.

```bash
xcat . --all -o context.txt
```

## Output safety

`xcat` does not overwrite files by default.

If the output file already exists:

```bash
xcat . -o context.txt
```

the command fails with:

```text
error: output file already exists: context.txt
hint: pass -f/--force to overwrite it
```

To overwrite intentionally:

```bash
xcat . -o context.txt -f
```

or:

```bash
xcat . -o context.txt --force
```

The current output file is also excluded from the scan automatically.

This command does not read `context.txt` as input while generating it:

```bash
xcat . -o context.txt -f
```

## Design philosophy

`xcat` is intentionally small.

It does not parse your project semantically. It does not call an LLM. It does not summarize your code. It does not try to be an IDE.

It does one thing:

> turn selected files and directories into a clean text context

That makes it easy to understand, script, pipe, and combine with other tools.

## Development

Clone the repository:

```bash
git clone https://github.com/mrAnderson2159/xcat.git
cd xcat
```

Install dependencies:

```bash
uv sync
```

Install the local CLI:

```bash
uv tool install -e .
```

Run:

```bash
xcat --help
```

Smoke test:

```bash
mkdir -p /tmp/xcat-smoke/src
echo 'print("hello xcat")' > /tmp/xcat-smoke/src/main.py

xcat /tmp/xcat-smoke --python -o /tmp/xcat-smoke/context.txt
cat /tmp/xcat-smoke/context.txt
```

Expected output:

```text
----------> src/main.py <----------

print("hello xcat")
```

## Requirements

`xcat` requires Python 3.12 or newer.

## License

MIT
