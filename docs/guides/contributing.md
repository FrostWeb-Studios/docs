---
title: Contributing
---

# Contributing to the Documentation

This guide explains how to contribute to the FrostWeb UE5 Plugin Documentation. Whether you are fixing a typo, adding a new guide, or improving an existing page, the process is the same.

---

## Repository Setup

The documentation source lives in the [FrostWeb-Studios/docs](https://github.com/FrostWeb-Studios/docs) repository. It is built with [MkDocs](https://www.mkdocs.org/) using the [Material for MkDocs](https://squidfunky.github.io/mkdocs-material/) theme.

### Clone the Repository

```bash
git clone https://github.com/FrostWeb-Studios/docs.git
cd docs
```

### Install Dependencies

Create a virtual environment (recommended) and install the required packages:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Run the Dev Server

Start the local development server to preview your changes:

```bash
mkdocs serve
```

This launches a live-reloading server at `http://127.0.0.1:8000`. Every time you save a Markdown file, the browser refreshes automatically.

---

## Making Changes

### File Structure

```
docs/
    index.md                    # Landing page
    getting-started.md          # Getting started guide
    plugins/
        index.md                # Plugins overview
        ai-system/
            index.md            # Plugin landing page
            installation.md
            quick-start.md
            api-reference.md
            blueprints.md
            configuration.md
            tutorials.md
            faq.md
            migration.md
            changelog.md
        chat-system/
            ...
        ...
    guides/
        architecture.md
        contributing.md
    assets/
        css/
            custom.css
```

Each plugin has its own directory under `docs/plugins/` with a consistent set of pages. The navigation structure is defined in `mkdocs.yml`.

### Editing Existing Pages

1. Locate the Markdown file you want to edit under `docs/`.
2. Make your changes.
3. Preview locally with `mkdocs serve`.
4. Commit and open a pull request.

### Adding a New Page

1. Create the Markdown file in the appropriate directory.
2. Add a front matter block with at least a `title` field:

    ```yaml
    ---
    title: Your Page Title
    ---
    ```

3. Add the page to the `nav` section of `mkdocs.yml` in the correct position.
4. Preview locally to verify navigation and rendering.

---

## Pull Request Workflow

All changes go through pull requests. Direct pushes to `main` are not permitted.

1. **Create a branch** from the latest `main`:

    ```bash
    git checkout main
    git pull origin main
    git checkout -b docs/your-change-description
    ```

2. **Make your changes** and verify with `mkdocs serve`.

3. **Commit** with a clear message:

    ```bash
    git add .
    git commit -m "docs: description of your change"
    ```

4. **Push and open a PR**:

    ```bash
    git push -u origin docs/your-change-description
    ```

    Then open a pull request on GitHub targeting `main`.

5. **Review** -- A maintainer will review your PR. Address any feedback, then the PR will be merged.

---

## Style Guidelines

Consistent documentation is easier to read and maintain. Follow these conventions across all pages.

### Headings

- Use `#` for the page title (one per page).
- Use `##` for major sections.
- Use `###` for subsections.
- Do not skip heading levels (e.g., do not jump from `##` to `####`).

### Admonitions

Use MkDocs Material admonitions to highlight important information:

```markdown
!!! note "Title"
    Content of the note.

!!! warning "Title"
    Content of the warning.

!!! tip "Title"
    Content of the tip.

!!! info "Title"
    Content of the info block.
```

Use admonitions sparingly. Reserve `warning` for things that can cause build failures, data loss, or broken functionality. Use `tip` for helpful suggestions. Use `note` and `info` for supplementary context.

### Code Blocks

Always specify the language for syntax highlighting:

````markdown
```cpp
void MyFunction()
{
    // C++ code
}
```

```json
{
    "key": "value"
}
```

```bash
mkdocs serve
```
````

All code blocks automatically include a copy button (enabled in `mkdocs.yml`).

### Tables

Use Markdown tables for structured data. Align columns for readability in the source:

```markdown
| Column A | Column B | Column C |
|---|---|---|
| Value 1 | Value 2 | Value 3 |
```

### Content Tabs

Use tabs when showing platform-specific or alternative instructions:

```markdown
=== "C++"

    ```cpp
    UFWInventoryComponent* Inv = GetOwner()->FindComponentByClass<UFWInventoryComponent>();
    ```

=== "Blueprint"

    Use the **Get Component by Class** node with `FWInventoryComponent` selected.
```

### Links

- Use relative links for internal pages: `[Getting Started](../getting-started.md)`
- Use full URLs for external resources: `[Unreal Documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/)`

### General Rules

- Write in clear, direct language.
- Use present tense ("The component handles..." not "The component will handle...").
- Do not add decorative elements or emoji to documentation.
- Keep paragraphs short -- aim for 3-4 sentences maximum.
- Define acronyms on first use (e.g., "Gameplay Ability System (GAS)").

---

## Versioning with mike

This documentation uses [mike](https://github.com/jimporter/mike) for version management. Versioned documentation allows users to view docs for the plugin version they are using.

### How It Works

- Each release is deployed as a named version (e.g., `1.0`, `2.0`).
- The `latest` alias always points to the most recent stable release.
- The version selector in the site header lets users switch between versions.

### Deploying a Version

!!! note
    Version deployment is handled by maintainers. Contributors do not need to run these commands.

```bash
# Deploy the current docs as version 1.0
mike deploy --push --update-aliases 1.0 latest

# Set a default version
mike set-default --push latest
```

---

## Getting Help

If you have questions about contributing or need help with the documentation tooling, open an issue on the [docs repository](https://github.com/FrostWeb-Studios/docs/issues) or reach out on the [FrostWeb Discord](https://discord.gg/frostweb).
