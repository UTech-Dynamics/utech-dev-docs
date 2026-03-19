# VS Code Lean Setup for Rails (API + RSpec)

## Objective

A minimal, stable, and performant VS Code setup for Rails backend development (API-focused), with Rubocop, RSpec, and clean navigation.

---

## 1. Required Extensions (Install Only These)

### Core

* Ruby LSP
* Rails (Hridoy)
* Endwise
* ERB Formatter (only if using ERB views)

### Optional

* GitLens (advanced Git usage)
* Better Comments (TODO highlighting)

> Avoid installing too many extensions. Add only when a real need appears.

---

## 2. Global User Settings

Open:
`Preferences: Open User Settings (JSON)`

Add:

```json
{
  "editor.formatOnSave": true,
  "editor.tabSize": 2,
  "editor.detectIndentation": false,

  "ruby.format": "rubocop",

  "files.exclude": {
    "**/log": true,
    "**/tmp": true,
    "**/node_modules": true
  },

  "search.exclude": {
    "**/log": true,
    "**/tmp": true
  }
}
```

---

## 3. Project-Level Settings (Recommended)

Inside each Rails project:

```
.vscode/settings.json
```

```json
{
  "editor.formatOnSave": true,
  "ruby.format": "rubocop",

  "files.exclude": {
    "**/log": true,
    "**/tmp": true,
    "**/storage": true
  }
}
```

---

## 4. Rubocop Setup (Required)

Add to Gemfile:

```ruby
group :development do
  gem 'rubocop', require: false
end
```

Run:

```bash
bundle install
```

### Verify

* Write badly formatted Ruby code
* Save file
* Code should auto-correct

---

## 5. Snippets Setup

Global snippet file (Linux):

```
mkdir -p ~/.config/Code/User/snippets
code ~/.config/Code/User/snippets/ruby.json
```

Example:

```json
{
  "Rails Model": {
    "prefix": "rmodel",
    "body": [
      "class ${1:ModelName} < ApplicationRecord",
      "  ${2}",
      "end"
    ]
  },
  "Rails Controller": {
    "prefix": "rctrl",
    "body": [
      "class ${1:Name}Controller < ApplicationController",
      "  def index",
      "  end",
      "end"
    ]
  }
}
```

---

## 6. Navigation Workflow (No Heavy Extensions)

VS Code does not provide full Rails navigation like RubyMine. Use:

### File Navigation

```
Ctrl + P
```

Type:

* `user.rb`
* `users_controller`

### Symbol Search

```
Ctrl + T
```

Search class or method name.

### Command Palette

```
Ctrl + Shift + P
```

Type:

```
Rails:
```

---

## 7. Key Shortcuts

| Action          | Shortcut         |
| --------------- | ---------------- |
| Command Palette | Ctrl + Shift + P |
| Open File       | Ctrl + P         |
| Search Symbol   | Ctrl + T         |
| Format          | Shift + Alt + F  |
| Terminal        | Ctrl + `         |

---

## 8. Verification Checklist

Ensure:

* [ ] Snippets expand (e.g., `rmodel`)
* [ ] `end` auto-inserts after `def`
* [ ] Saving file triggers Rubocop
* [ ] No errors in Problems panel (`Ctrl + Shift + M`)

---

## 9. What NOT to Do

* Do not install many extensions
* Do not modify default settings.json
* Do not rely on global gems for tooling
* Do not use Solargraph unless needed

---

## 10. Philosophy

* Keep setup minimal
* Prefer Bundler-managed tools
* Optimize workflow, not editor
* Add tools only when a real problem appears

---

## Final State

This setup is sufficient for:

* Rails API development
* RSpec testing
* Clean code formatting (Rubocop)
* Lightweight and fast VS Code experience

No further editor optimization is required.
