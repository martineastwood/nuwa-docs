# AI-Assisted Development

Nuwa Build includes a `skill.md` file designed to teach AI coding agents (Claude Code, Cursor, Windsurf, Antigravity, etc.) how to work with Nim/Python extensions.

## Why Use a Skill File?

Standard LLMs often assume Python extensions require `setup.py` or `pip install -e .`. The `skill.md` file provides your agent with the correct context to:

- **Understand the flat layout** - Knows that `.so` files are generated directly in the package directory
- **Use correct commands** - Enforces `nuwa develop` and `nuwa watch` instead of standard pip commands
- **Write correct Nim** - Reminds the agent to use `include` for shared libraries and `{.nuwa_export.}` for bindings

## Supported AI Agents

### Claude Code

**Global installation** (recommended for all projects):

```bash
# Create skills directory
mkdir -p ~/.claude/skills/nuwa-build

# Copy the skill file
cp skills/SKILL.md ~/.claude/skills/nuwa-build/skill.md
```

**Project-specific installation:**

```bash
# Create skills directory in your project
mkdir -p .claude/skills/nuwa-build

# Copy the skill file
cp skills/SKILL.md .claude/skills/nuwa-build/skill.md
```

The skill will automatically load when you work in a nuwa-build project.

### Cursor

**Global installation:**

```bash
mkdir -p ~/.cursor/skills/nuwa-build
cp skills/SKILL.md ~/.cursor/skills/nuwa-build/skill.md
```

**Project-specific installation:**

```bash
mkdir -p .cursor/skills/nuwa-build
cp skills/SKILL.md .cursor/skills/nuwa-build/skill.md
```

### Windsurf / Antigravity

**Global installation:**

```bash
mkdir -p ~/.gemini/antigravity/skills/nuwa-build
cp skills/SKILL.md ~/.gemini/antigravity/skills/nuwa-build/skill.md
```

**Project-specific installation:**

```bash
mkdir -p .agent/skills/nuwa-build
cp skills/SKILL.md .agent/skills/nuwa-build/skill.md
```

### General Chat (ChatGPT, Web UIs, etc.)

For AI tools that don't support file-based skills, you can:

1. **Upload the file** - Upload `skills/SKILL.md` directly to the chat
2. **Paste the content** - Copy and paste the skill file contents into your conversation
3. **Add to system prompt** - Include key points in your custom system prompt

## What the Skill File Teaches

The skill file covers:

- **Prerequisites** - Nim compiler installation, nuwa-build setup
- **Project structure** - Flat layout explanation
- **Commands** - All `nuwa` CLI commands with usage patterns
- **Configuration** - `pyproject.toml` options and build profiles
- **Entry point discovery** - Auto-discovery logic
- **Writing Nim** - Proper use of `{.nuwa_export.}` and `include`
- **Multi-file projects** - Using `include` vs `import`
- **Package data** - Automatic inclusion and `MANIFEST.in`
- **Common errors** - Troubleshooting tips
- **AI agent guidelines** - Best practices for assisting users

## Example Interactions

With the skill file loaded, AI agents can correctly handle:

### Creating a New Project

```markdown
User: Create a Nim/Python extension for calculating fibonacci numbers

Agent: I'll create a new nuwa-build project with a fibonacci function.

[Creates project with `nuwa new fibonacci`, writes Nim code with {.nuwa_export.}, runs `nuwa develop`]
```

### Debugging

```markdown
User: My extension isn't loading

Agent: Let me check if it's compiled first. I'll run `nuwa develop` to compile the extension.

[Understands that Python extensions need compilation, not pip install]
```

### Multi-file Projects

```markdown
User: Add a helper module for validation

Agent: I'll create `helpers.nim` and include it in the main file using `include` (not `import`) since we're building a shared library.

[Uses correct include pattern for shared libraries]
```

## Updating the Skill File

The skill file is located at `skills/SKILL.md` in any nuwa-build project. To improve AI assistance:

1. Open `skills/SKILL.md`
2. Add project-specific patterns or conventions
3. Update the file in your agent's skills directory

## Troubleshooting AI Agents

If your AI agent isn't using nuwa-build correctly:

1. **Verify skill is loaded** - Check that the skill file is in the correct directory
2. **Restart the agent** - Some agents require restart to pick up new skills
3. **Check skill format** - Ensure the file has valid YAML frontmatter for agents that require it
4. **Use project-specific skills** - Global skills may be overridden by project-specific ones
