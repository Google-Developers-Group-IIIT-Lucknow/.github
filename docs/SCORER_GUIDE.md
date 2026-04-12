# DevSphere — Scorer / Task Creator Guide

This document explains how to configure the automated grading for your specific task.
The only file you need to edit is **`tests/run_tests.sh`**.
Everything else (CI workflows, hook, labels, leaderboard call) is handled automatically.

---

## How grading works

The grading pipeline is **all-or-nothing**: a submission either passes all checks
and gets full marks, or it gets zero. There is no partial credit.

Your `run_tests.sh` script is the single source of truth for what "correct" means.
It must:
- **Exit 0** when every test passes
- **Exit 1** when any test fails (wrong output, TLE, runtime error, compile error, etc.)

The same script runs in three places:
1. The participant's machine (pre-commit hook)
2. The participant's fork CI (on push — feedback)
3. The main repo's PR Checks workflow (authoritative)

---

## Test file conventions

Tests live in the `samples/` directory.  
Each test is a pair of files named with a consistent prefix:

| File | Purpose |
|------|---------|
| `samples/in_<name>.txt`  | Input fed to the participant's solution |
| `samples/out_<name>.txt` | Expected output to compare against |

You can have multiple test cases:
```
samples/in_sample.txt      → samples/out_sample.txt
samples/in_edge1.txt       → samples/out_edge1.txt
samples/in_large.txt       → samples/out_large.txt
```

The test runner automatically picks up all `in_*.txt` files and matches them
to their corresponding `out_*.txt` file.

### Adding test cases

1. Create the input file: `samples/in_<name>.txt`
2. Create the expected output file: `samples/out_<name>.txt`
3. Add both files to `.readonly-files` so participants cannot replace them
4. Commit and push to `main`

**Important:** The CI always restores sample files from `main` before running
tests. Participants can modify local copies but those changes are overwritten.

---

## Configuring `tests/run_tests.sh`

Open `tests/run_tests.sh` and fill in the `CONFIGURE` section.

### Example 1 — C++ competitive programming (like FOSS repos)

```bash
SOLUTION_FILE="$ROOT/solution1.cpp"
BIN="$ROOT/solution_bin"
compile() { g++ -O2 -std=c++17 -o "$BIN" "$SOLUTION_FILE"; }
run()     { "$BIN" < "$1"; }
TIMEOUT=40
```

What participants submit: a single `.cpp` file.  
What gets checked: compilation + stdout matches expected output.

### Example 2 — Python solution

```bash
SOLUTION_FILE="$ROOT/solution.py"
compile() { python3 -m py_compile "$SOLUTION_FILE"; }
run()     { python3 "$SOLUTION_FILE" < "$1"; }
TIMEOUT=30
```

### Example 3 — JavaScript / Node.js solution

```bash
SOLUTION_FILE="$ROOT/solution.js"
compile() { node --check "$SOLUTION_FILE"; }
run()     { node "$SOLUTION_FILE" < "$1"; }
TIMEOUT=30
```

### Example 4 — Java solution

```bash
compile() {
  cd "$ROOT"
  javac Solution.java
}
run() {
  java -cp "$ROOT" Solution < "$1"
}
TIMEOUT=30
```

---

## Non-stdio tasks (WEB / APP)

For web or app tasks the solution isn't a single file reading stdin.
Use a **checker script** instead of stdin/stdout comparison.

### Structure for a web task

```
samples/
  in_task1.txt     ← description / spec for the checker (can be empty)
  out_task1.txt    ← expected checker output, e.g. "PASS"
tests/
  run_tests.sh
  checker.js       ← your custom checker script
```

In `run_tests.sh`:

```bash
compile() {
  cd "$ROOT"
  npm ci --silent 2>&1
  npm run build --silent 2>&1
}

# The "input file" here is just a trigger — the checker knows
# what to test from the repo structure.
run() {
  # checker.js exits 0 and prints "PASS" on success,
  # exits 1 and prints "FAIL: <reason>" on failure
  node "$ROOT/tests/checker.js"
}

TIMEOUT=120   # builds can take a while
```

### Writing a checker script

A checker is any script that:
- Examines the participant's output or files
- Prints a single line: `PASS` on success or `FAIL: <reason>` on failure
- Exits 0 on pass, exits 1 on fail

Example `tests/checker.js` for a web task that checks if an HTML file is valid:

```javascript
const fs   = require('fs');
const path = require('path');

const root    = path.join(__dirname, '..');
const htmlFile = path.join(root, 'index.html');

if (!fs.existsSync(htmlFile)) {
  console.log('FAIL: index.html not found');
  process.exit(1);
}

const content = fs.readFileSync(htmlFile, 'utf8');

// Example checks
if (!content.includes('<title>')) {
  console.log('FAIL: missing <title> tag');
  process.exit(1);
}

if (!content.includes('id="leaderboard"')) {
  console.log('FAIL: missing leaderboard element');
  process.exit(1);
}

console.log('PASS');
process.exit(0);
```

The test runner compares the checker's stdout against `out_<name>.txt` (which should contain just `PASS`).

---

## Making changes as a task creator

Because `.hooks/pre-commit` is active in the repo and runs `tests/run_tests.sh` before every commit, **any commit you make is subject to the same test check as participants**. If the solution file in your repo is intentionally broken (e.g. a FOSS task where the correct solution is hidden in history), the pre-commit hook will block your commits.

### Always use `--no-verify` for infrastructure commits

When committing workflow files, the hook, test files, README, or anything that is not the solution itself, bypass the hook with `--no-verify`:

```bash
git add .github/workflows/pr-grader.yml
git commit --no-verify -m "ci: update grader task_title"
git push
```

```bash
git add tests/run_tests.sh samples/ .readonly-files
git commit --no-verify -m "feat: add test cases"
git push
```

`--no-verify` skips the pre-commit hook for that one commit only. It does not affect participants — they cannot use it to bypass CI, because CI runs in GitHub Actions regardless of how the commit was made locally.

### Temporarily disabling the hook

If you are making many infrastructure changes in a session, you can disable the hook for the duration:

```bash
git config core.hooksPath .git/hooks   # disable (points to empty dir)

# ... make all your commits normally ...

git config core.hooksPath .hooks        # re-enable when done
```

### After updating a protected file

If you change any file listed in `.readonly-files` (e.g. update the hook, fix a workflow, add a test case) and participants have **already forked** the repo, their forks will have the old version. When they open a PR the readonly check will flag the difference as a violation even though they didn't touch it.

**Fix for affected participants:** ask them to sync their fork before opening a PR.
On GitHub: their fork → **Sync fork** → **Update branch**.

To avoid this during the event: finalise all changes to protected files **before** participants fork the repos.

---

## Protecting test files

Add every file participants should not modify to `.readonly-files`.

```
# Standard protected files (already in template)
.github/workflows/fork-ci.yml
.github/workflows/pr-checks.yml
.github/workflows/pr-grader.yml
.hooks/pre-commit
tests/run_tests.sh
README.md
.readonly-files

# Add your test files
samples/in_sample.txt
samples/out_sample.txt
samples/in_edge1.txt
samples/out_edge1.txt

# For WEB/APP tasks with a checker:
tests/checker.js
```

Do NOT add the participant's solution file (e.g. `solution1.cpp`) to this list — that's what they're supposed to change.

---

## FOSS repos: the `TARGET_COMMIT_HASH` secret

FOSS repos use a special "time machine" mechanic: the correct solution already exists
somewhere in the repository's git history. Participants must find and submit that commit.

**How to set it up:**

1. In your local clone of the FOSS repo, create the correct solution on a separate branch
   (or use a past commit that has the correct solution):
   ```bash
   git log --all --oneline   # browse all commits
   ```

2. Note the SHA of the commit that contains the correct solution.

3. Set the secret:
   ```bash
   gh secret set TARGET_COMMIT_HASH \
     --body "<your SHA>" \
     --repo "Google-Developers-Group-IIIT-Lucknow/DevSphere-FOSS-Easy"
   ```

**What the grader checks:**
- The PR has exactly 1 commit ahead of `main`
- That commit's SHA **or its parent's SHA** matches `TARGET_COMMIT_HASH`

This means participants can either:
- `git reset --hard <SHA>` → they ARE at the target commit (HEAD matches)
- `git cherry-pick <SHA>` → parent is the target commit (PARENT matches)

Both are valid approaches and both pass the check.

**Do not set this secret for WEB/APP repos.** When the secret is absent, the commit check is silently skipped.

---

## Checklist for task creators

- [ ] Implement `tests/run_tests.sh` for your task's language/framework
- [ ] Add sample test cases to `samples/` (at least one `in_*.txt` / `out_*.txt` pair)
- [ ] Add sample files to `.readonly-files`
- [ ] Edit the two `CHANGE_ME` lines in `pr-grader.yml` (task_title + difficulty)
- [ ] (FOSS only) Set `TARGET_COMMIT_HASH` secret
- [ ] Commit all infrastructure changes with `--no-verify` (see "Making changes as a task creator" above)
- [ ] Test locally: `bash tests/run_tests.sh`
- [ ] Test on fork: push a passing solution (any branch), confirm fork-ci.yml turns green
- [ ] Test PR flow: open a PR, confirm grader fires and leaderboard updates

---

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Committing without `--no-verify` | Hook blocks commit because solution is intentionally broken — always use `--no-verify` for infrastructure changes |
| Updating a protected file after forks exist | Existing forks show a false readonly violation — ask participants to sync fork before opening PR |
| `run_tests.sh` exits 0 even on wrong output | Use `diff` and check its exit code |
| Sample files not in `.readonly-files` | Participants can replace them with trivial cases |
| CHANGE_ME left in `pr-grader.yml` | Leaderboard API will return 404 (task not found) |
| `event_start_time` not set in Supabase | All `time_taken` values default to 1 hour ago |
| Labels not created before event | Grader step silently fails on `addLabels` call |
| `TARGET_COMMIT_HASH` set on WEB repo | Commit check runs and blocks all valid submissions |
