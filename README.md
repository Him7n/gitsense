# GitSense - AI-Powered Git Commit & PR Assistant

GitSense is an interactive CLI tool that uses AI to generate conventional commit messages and PR descriptions from your git changes. It now includes powerful JIRA integration features!

## Features

### 🎯 Core Features
- **AI-Generated Commit Messages**: Analyzes your git diff and generates 5 conventional commit message options
- **Interactive Selection**: Navigate with arrow keys or vim-style (j/k) shortcuts
- **Smart Diff Analysis**: Automatically detects staged vs unstaged changes
- **Commit Style Learning**: Learns from your recent commit history to match your style

### 🎫 JIRA Integration (NEW!)
- **Automatic Ticket Detection**: Extracts JIRA ticket ID from branch name (e.g., `feature/CDS-116548-auth` → `CDS-116548`)
- **PR Description Generator**: Creates minimal, focused PR descriptions in WHAT/WHY/HOW/TESTS format
- **JIRA Field Helper**: Generates text for common JIRA fields:
  - Bug Resolution
  - CFD Escape Analysis
  - Public Release Notes Summary
- **Branch-Origin Diff**: Calculates all changes since branch diverged from main/master

## Installation

### Prerequisites
```bash
# Required
npm install -g cursor-agent  # AI model access

# Optional (for JIRA features)
# Install JIRA CLI: https://github.com/ankitpokhrel/jira-cli
```

### Setup
```bash
# Make executable
chmod +x gitsense

# Optional: Add to PATH
sudo ln -s $(pwd)/gitsense /usr/local/bin/gitsense
```

## Usage

### Basic Commit Flow
```bash
# Navigate to your git repository
cd your-project

# Stage your changes (or don't - gitsense handles both!)
git add .

# Run gitsense
./gitsense
```

### Keyboard Shortcuts

#### Navigation
- `↑/↓` or `j/k` - Move up/down through options (vim-style)
- `1-5` - Select option directly by number

#### Actions
- `Enter` - Commit with selected message
- `e` - Edit message in nvim before committing
- `c` - Copy message to clipboard
- `d` - Preview full diff
- `r` - Refine AI suggestions with feedback

#### JIRA Features (when ticket detected in branch)
- `p` - Generate PR description from JIRA ticket + git changes
- `j` - Generate JIRA field text (Bug Resolution, CFD Escape, Release Notes)

#### Other
- `?` - Show help menu
- `q` - Quit without committing

## JIRA Integration Examples

### PR Description Generation

When on a branch like `feature/CDS-116548-add-authentication`:

1. Press `p` in the menu
2. GitSense will:
   - Extract ticket ID: `CDS-116548`
   - Fetch JIRA ticket info via `jira issue view CDS-116548`
   - Analyze all commits since branch diverged from main
   - Calculate diff from branch origin point
   - Generate minimal PR description in format:

```markdown
## WHAT
Add JWT-based authentication to API endpoints

## WHY
Enable secure user access as per CDS-116548 requirements

## HOW
- Implement JWT token generation and validation middleware
- Add authentication guards to protected routes
- Store refresh tokens in Redis cache

## TESTS
- tests/auth/jwt.test.ts
- tests/middleware/auth.test.ts
```

3. Copy to clipboard automatically (optional)

### JIRA Field Text Generation

Press `j` to open JIRA field generator menu:

**Bug Resolution Example:**
```
Root cause: Null pointer exception when user profile was missing optional fields.
Fixed by adding null checks and default values in ProfileService.validate().
Prevents recurrence by enforcing schema validation at API boundary.
```

**CFD Escape Analysis Example:**
```
This escaped because our test suite didn't cover the edge case of missing
optional fields. The gap was lack of property-based testing for partial
object scenarios. Future prevention: Add property-based tests using fast-check
library for all API models.
```

**Public Release Notes Example:**
```
Users can now save incomplete profiles without required fields causing errors.
This improves the onboarding experience for new users.
```

## How It Works

### Diff Detection Priority
1. **Staged changes** (if any): `git diff --cached`
2. **All unstaged changes** (if nothing staged): `git diff HEAD`
3. **Untracked files**: Warning shown but not analyzed

### Branch Origin Calculation
For JIRA/PR features, GitSense finds where your branch diverged:
```bash
# Finds merge-base with main or master
git merge-base main HEAD

# Gets all commits since divergence
git log <merge-base>..HEAD

# Calculates full diff from branch start
git diff <merge-base>..HEAD
```

This ensures PR descriptions include ALL work on the branch, not just latest commit.

### JIRA Ticket ID Extraction
Supports standard JIRA patterns:
- `feature/CDS-116548-description` → `CDS-116548`
- `bugfix/PROJ-123-fix-auth` → `PROJ-123`
- `CDS-116548` → `CDS-116548`

Pattern: `[A-Z]+-[0-9]+` (uppercase letters, dash, numbers)

## Configuration

### AI Model
Currently uses `grok-code-fast-1` via cursor-agent. Can be modified in the script:
```bash
cursor-agent --model grok-code-fast-1 --print
```

### Commit Message Format
Follows Conventional Commits spec:
```
<type>: [<branch-name>]: <description>

Types: feat, fix, chore, docs, refactor, style, perf, test, ci, build
```

### JIRA CLI Setup
Ensure `jira` CLI is configured:
```bash
# Test JIRA access
jira issue view YOUR-TICKET-123

# If not configured, follow jira-cli setup docs
```

## Workflow Examples

### Example 1: Simple Commit
```bash
$ ./gitsense
👀 Analyzing changes with AI (grok-code-fast-1)...
------------------------------------------------
Select a commit message (↑/↓ to move, Enter to select):
------------------------------------------------
 -> feat: [master]: add user authentication system
    feat: [master]: implement JWT-based auth
    feat: [master]: enable secure user login
    fix: [master]: add missing auth middleware
    chore: [master]: update auth dependencies
------------------------------------------------
 [Enter] Commit  [e] Edit  [c] Copy  [d] Diff  [r] Refine  [?] Help  [q] Quit
```

### Example 2: PR Description (JIRA Branch)
```bash
# On branch: feature/CDS-116548-add-auth
$ ./gitsense

# Press 'p' for PR description
📋 Generating PR Description for CDS-116548...

================================================
📋 PR Description for CDS-116548
================================================

## WHAT
Add JWT authentication to API endpoints

## WHY
Enable secure user sessions as required by security audit (CDS-116548)

## HOW
- Implement JWT middleware with RS256 signing
- Add refresh token rotation
- Integrate with existing user service

## TESTS
- tests/auth/jwt.test.ts (12 new test cases)
- tests/integration/auth-flow.test.ts

================================================

Copy to clipboard? [Y/n]:
```

### Example 3: JIRA Field Generation
```bash
# Press 'j' for JIRA fields
================================================
🎯 JIRA Field Generator - CDS-116548
================================================
 1) Bug Resolution
 2) CFD Escape Analysis
 3) Public Release Notes Summary
 q) Back to main menu
================================================
Select field: 1

--- Bug Resolution ---
Root cause: Race condition in token refresh logic caused expired tokens
to be accepted during concurrent requests. Fixed by implementing atomic
token validation with Redis distributed lock. Prevents recurrence by
adding integration tests for concurrent auth scenarios.

Copy to clipboard? [Y/n]:
```

## Tips & Best Practices

1. **Stage Important Changes**: Stage changes you want to commit, leave exploratory code unstaged
2. **Descriptive Branch Names**: Include JIRA ticket in branch name for auto-detection
3. **Use Refine**: Press `r` to give feedback if AI suggestions miss the mark
4. **Edit When Needed**: Press `e` to fine-tune AI suggestions in nvim
5. **Preview Diffs**: Press `d` before committing to review changes
6. **One PR Per Branch**: JIRA features work best with feature-branch workflow

## Troubleshooting

### "cursor-agent not found"
```bash
npm install -g cursor-agent
```

### "jira CLI not found" (for JIRA features)
JIRA features are optional. Install from: https://github.com/ankitpokhrel/jira-cli

### "No JIRA ticket found"
Ensure your branch name includes ticket ID: `feature/CDS-116548-description`

### AI generates wrong message type
Press `r` and provide feedback like: "This is a bugfix, not a feature"

### No changes detected
Make sure you have either staged changes (`git add`) or modified files

## Other Scripts

### `gitsense_fast` (Alternative - Fast)
- Uses OpenAI's gpt-4o-mini model via direct API calls
- Much faster than CLI wrappers
- Requires `OPENAI_API_KEY` environment variable

### `gitsense_gemini` (Alternative)
- Uses Google's Gemini CLI
- Slower due to CLI overhead
- Requires `gemini` CLI to be installed

## Contributing

Suggestions and improvements welcome! This is a productivity tool, so PRs for:
- Better AI prompts
- Additional JIRA field types
- Support for other issue trackers (Linear, GitHub Issues)
- Performance optimizations

## License

MIT License - feel free to use and modify!

## Credits

- Built with `cursor-agent` for AI completions
- Inspired by conventional commit standards
- JIRA integration via `jira-cli`
