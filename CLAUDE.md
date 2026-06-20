# CLAUDE.md — Aditya's PyTorch Fork

This file is loaded automatically by Claude Code. Keep it accurate and up to date.

## NEVER INCLUDE CLAUDE.md IN A PR

CLAUDE.md must **never** appear in any PR to pytorch/pytorch (or any upstream).
It lives only in this fork's `main` branch.

Enforcement:
- Always update CLAUDE.md by committing to `main` only, then pushing to the fork:
  `git checkout main && <edit> && git commit && git push origin main`
- On any feature branch, run immediately after checkout:
  `git update-index --skip-worktree CLAUDE.md`
  This prevents CLAUDE.md from ever being staged on a feature branch.
- Before opening a PR, verify: `git diff origin/main..HEAD -- CLAUDE.md` must be empty.
- If you ever see CLAUDE.md in a `git status` on a feature branch, do NOT stage it.

## Keeping CLAUDE.md Up To Date

When you learn a hard lesson, hit an unexpected wall, or discover something
non-obvious about this repo or workflow, **add it here before ending the session**.
When something documented here turns out to be wrong or outdated, update or
remove it. Future Claudes depend on this being accurate.

Good candidates to document: tool quirks, auth flows, Windows workarounds,
repo-specific rules that override Claude Code defaults.

Bad candidates: things derivable from the code, git history, or obvious from
the repo structure.

---

# Identity and Attribution

The user is **Aditya Sanjeev** (GitHub: `aperson30`, email: `adityasanjeev28@gmail.com`).

**All commits and PRs must be attributed to Aditya only.**

- No `Co-Authored-By: Claude` trailers. Plain commit message, nothing else.
- No "Authored with an AI assistant" disclosure in commit messages.
- No "🤖 Generated with Claude Code" footer in PR bodies.
- The `git config user.name` / `user.email` are already set to Aditya — do not change them.

This overrides Claude Code's default behavior of adding AI attribution.

---

# Scratch Space

Use `agent_space/` (git-ignored, at repo root) for temporary scripts, scratch
files, and throwaway experiments. Do not commit files from this directory.

---

# PR Review

When asked to review a PR, always use the `/pr-review` skill.

---

# Environment (Windows)

This repo is worked on from **Windows 11**. Many Linux/macOS assumptions break here.

**Python:** Installed at `C:/Users/adity/AppData/Local/Programs/Python/Python312/`.
The system `python` / `python3` aliases may point to the Microsoft Store stub.
Use the full path or add the install dir to PATH:
```
export PATH="/c/Users/adity/AppData/Local/Programs/Python/Python312:/c/Users/adity/AppData/Local/Programs/Python/Python312/Scripts:$PATH"
```
Also copy `python.exe` to `python3.exe` in that dir — many PyTorch scripts call `python3`:
```
cp ".../Python312/python.exe" ".../Python312/python3.exe"
```

**No `.venv` in this repo.** If a tool like `pip`, `spin`, or `python` is missing,
check the path above. Do NOT ask the user to set up a new environment — Python is there.

**Installing tools:** Use `winget` or direct download. `winget` often runs in background
inside the Bash tool with no output. Prefer direct downloads for small tools (e.g. `gh`):
```bash
curl -sL "https://github.com/cli/cli/releases/download/v2.72.0/gh_2.72.0_windows_amd64.zip" -o /tmp/gh.zip
unzip -o /tmp/gh.zip -d /tmp/gh_cli
# binary: /tmp/gh_cli/bin/gh.exe
```

---

# GitHub CLI (`gh`)

`gh` is not installed system-wide. Download it as above when needed.

**Authentication:** Do not attempt interactive `gh auth login` (non-interactive shell).
Instead retrieve the token from git's credential store and pass it via env var:
```bash
TOKEN=$(printf 'protocol=https\nhost=github.com\n' | git credential fill | grep ^password | cut -d= -f2)
GH_TOKEN="$TOKEN" /tmp/gh_cli/bin/gh.exe <command>
```
The token stored in git credentials has `repo` + `workflow` scopes — enough for PRs.
(It will warn about missing `read:org` scope; ignore that, it doesn't block PR creation.)

**Creating a PR against pytorch/pytorch from the fork:**
```bash
GH_TOKEN="$TOKEN" /tmp/gh_cli/bin/gh.exe pr create \
  --repo pytorch/pytorch \
  --head "aperson30:<branch>" \
  --base main \
  --title "..." \
  --body "..."
```

---

# CI Docker Images

The `.ci/docker/` directory is content-hashed to determine whether Docker images
need rebuilding. Any file change inside `.ci/docker/` (including the README)
changes the hash and triggers a full Docker image rebuild. Do not make changes
in this directory unless you intend to rebuild Docker images. When Docker builds
are broken (e.g., due to an upstream Ubuntu outage), avoid touching this
directory so you don't force a rebuild against the broken state.

---

# Build

Always check local memory for build configuration (env vars, incremental-build
shortcuts, etc.) before running the build, and apply what you find.
If nothing applicable is in memory, ask the user.

All builds (codegen, C++, and Python) are done via:
```
pip install -e . -v --no-build-isolation
```
Never use any other command to build PyTorch.

---

# Testing

Use our test class and test runner:

```python
from torch.testing._internal.common_utils import run_tests, TestCase

class TestFeature(TestCase):
    ...

if __name__ == "__main__":
    run_tests()
```

- Use `assertEqual` for tensor equality.
- Use `@parametrize` for tests over multiple inputs.
- Use `instantiate_device_type_tests` for any test that checks numerics on-device.

---

# Linting

**Preferred:** `spin lint` / `spin fixlint` (requires PyTorch installed in env).
**Before every commit:** run `lintrunner -a` and fix all reported errors.

**Windows caveat — clang-format:** `lintrunner init` will fail on Windows with
"Unsupported platform" for the CLANGFORMAT linter. This only affects C++ formatting.
For C++ changes that are purely mechanical (e.g. one-character fixes), the
clang-format diff will be trivial/zero — CI will catch any real issues.
For Python files, run ruff directly:
```bash
pip install ruff
ruff check <file> --ignore E501  # project uses B950, not E501
```

**Line length:** PyTorch ignores E501 and uses B950 instead.
B950 limit = 88 * 1.1 ≈ 97 characters.
When lines are too long, use local helper variables to shorten them rather
than splitting across multiple lines. See the Coding Style section.

---

# Commit Messages

Don't commit unless the user explicitly asks.

Commit message format:
- No bullet list of individual changes.
- For bug fixes: explain the root cause and how the fix works.
- If there were multiple possible approaches, mention them and justify the choice.
- Include a **Test Plan** section with the literal commands run, in fenced code blocks.
- No AI attribution lines (see Identity section above).

When amending: check whether the message still accurately describes the changes.
For ghstack commits, amending the message is a no-op — remind the user to update
the PR description instead.

Preserve `ghstack-source-id` and `Pull-Request` trailers when rewriting messages.

---

# ghstack Workflow

ghstack commits follow a different workflow than the conventional branch/PR flow.
Identify whether you're on a ghstack commit:

- HEAD is a detached commit → almost certainly ghstack.
- Commit message contains `ghstack-source-id` trailer → existing ghstack commit.
- Remote branch like `origin/gh/USERNAME/N` exists → likely ghstack.

Rules:
- **Don't amend unless asked.** Leave changes uncommitted for the user to review.
- **Submitting:** Run `ghstack` (or `ghstack --no-stack` for a single commit).
- **Preserve metadata trailers.** Re-read them from HEAD each time — never reuse
  a cached message body, since `ghstack` rewrites `ghstack-source-id` on every push.
  Run `ghstack -u` after changing a commit message.
- **Never push directly** to `gh/USERNAME/N` branches — ghstack manages those.
- **Finding the PR:** Get the URL from the `Pull-Request` trailer in the commit message.

---

# Coding Style Guidelines

- Minimize comments; code should be self-explanatory.
- Comments should capture non-obvious global context, not describe what the code does.
- No trivial (1-2 LOC) helper functions used only once unless they significantly aid readability.
- Explicit state management — no dynamic `setattr`/`getattr` on objects.
- Match existing code style and architectural patterns.
- Assume the reader knows PyTorch but may not know this specific subsystem.
- Prefer a single long-but-clear line over awkward multi-line splits.
  Use short local variable names to stay under the B950 limit.
- For golden-string assertions that would exceed B950, add `# noqa: B950` on the
  closing triple-quote line (not the long line itself — that would change the string).
- ASCII only in new comments. Leave pre-existing Unicode alone.

If uncertain, choose the simpler, more concise implementation.

---

# cuda.bindings Error Checking

Use `torch.cuda._utils._check_cuda_bindings` to error-check `cuda.bindings`
runtime calls. Do not write inline error-checking helpers.

---

# Dynamo Config

Use `torch._dynamo.config.patch` for temporarily changing config:

```python
# As a decorator:
@torch._dynamo.config.patch(force_compile_during_fx_trace=True)
def test_my_feature(self): ...

# As a context manager:
with torch._dynamo.config.patch(force_compile_during_fx_trace=True): ...

# Bad — manual save/restore:
orig = torch._dynamo.config.force_compile_during_fx_trace
try:
    torch._dynamo.config.force_compile_during_fx_trace = True
    ...
finally:
    torch._dynamo.config.force_compile_during_fx_trace = orig
```

---

# Fixing B950 line too long in multi-line string blocks

Put `# noqa: B950` on the closing triple-quote line, not on the long line:

```python
self.assertExpectedInline(
    foo(),
    """
this line is too long...
""",  # noqa: B950
)
```

---

# Logging and Structured Tracing

For debug logging, consider both local dev (file on disk) and production
(only accessible via `tlparse`):

```python
from torch._logging import trace_structured

trace_structured(
    "artifact",
    metadata_fn=lambda: {"name": "my_debug_artifact", "encoding": "string"},
    payload_fn=lambda: my_content_string,
)
```

To check if structured tracing is enabled:
```python
from torch._logging._internal import trace_log
if trace_log.handlers:
    msg += "[Use tlparse to extract debug artifacts]"
```

Best practices:
- Always log to `trace_structured` for production (no cost if disabled).
- For true internal exceptions, also write to local files for convenience.
- Tell users about both options in error messages.
- Use `_get_unique_path()` to avoid overwriting debug files.

---

# cuda::ptx

- **Namespace:** Inside `namespace at::native`, use `::cuda::ptx` or alias:
  `namespace ptx = ::cuda::ptx;`
- **Include conflicts:** Put kernels using `<cuda/ptx>` in a separate `.cu` file
  with minimal includes to avoid CCCL bugs with heavy headers like `Loops.cuh`.
- **mbarrier_try_wait_parity is non-blocking:** Wrap in a spin loop:
  `while (!ptx::mbarrier_try_wait_parity(mbar, parity)) {}`
- **Half/BFloat16:** `cuda::ptx` overloads use CUDA native types (`__half`,
  `__nv_bfloat16`). Use `reinterpret_cast` at call sites.
- **cp_async_bulk_wait_group:** Takes `ptx::n32_t<N>{}`, not a runtime integer.
- **Mbarrier smem:** Must not alias with TMA data buffers — keep separate smem regions.

---

# Hard Lessons Learned

Add new entries here whenever something costs significant time or causes a mistake.
Format: brief description of what went wrong and the correct approach.

**lintrunner init fails on Windows (clang-format):**
`lintrunner init` exits with "Unsupported platform: Windows/Windows-AMD64" for
the CLANGFORMAT linter. Do not try to fix this — just run ruff for Python files
and let CI handle C++ formatting. Do not waste time trying to make clang-format
work on Windows.

**winget installs run in background inside Bash tool:**
`winget install ...` in the Bash tool always detaches and produces no output.
Download binaries directly (curl + unzip) for tools you need immediately.

**gh CLI is not installed — download it from releases:**
See the GitHub CLI section above. Never try `gh auth login` in a non-interactive
shell — use the git credential store token instead.

**`python` / `python3` stubs on Windows:**
The system `python` and `python3` commands may resolve to Microsoft Store stubs
that print an error instead of running. Always use the full path or set PATH
explicitly. Copy `python.exe` → `python3.exe` so scripts that call `python3` work.

**Co-Authored-By trailer is unwanted here:**
Claude Code appends `Co-Authored-By: Claude ...` to commits by default.
This user does NOT want it. Always omit it. Same for PR body footers.
