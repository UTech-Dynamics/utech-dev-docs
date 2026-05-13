# UTech Dynamics – Engineering & Release Guidelines

> **Purpose**
> This document defines a *minimal, practical* engineering standard for UTech Dynamics repositories.
> It is designed to be read once, applied consistently, and rarely changed.

---

## 1. Scope & Philosophy

* Audience: solo developers → small teams
* Primary focus: **API-first systems** (Rails, services, backend cores)
* Goals:

  * predictable releases
  * clean history
  * low coordination cost
  * production safety

**Principle:**

> *Move fast in `dev`, move safely in releases.*

---

## 2. Branching Model (Authoritative)

### `main` (or `master`)

* Production-ready only
* Must always be deployable
* No direct commits

---

### `dev`

* Active development branch
* May contain unfinished work
* UI, experiments, refactors allowed
* **Not deployable by default**

All feature work eventually merges here.

---

### Feature branches (short-lived)

Naming convention:

```
feat/<name>
fix/<name>
chore/<name>
```

Rules:

* Branch from `dev`
* Merge via PR (even for solo work)
* Delete after merge

---

### `release/x.y` (optional)

Example:

```
release/v1.0
```

Purpose:

* Post-release maintenance only
* Hotfixes or critical production fixes

Rules:

* Mostly inactive
* No feature work
* Any change requires a **new tag**

---

## 3. Versioning Standard (Semantic Versioning)

We follow **SemVer**:

```
MAJOR.MINOR.PATCH
```

| Type  | Meaning                     |
| ----- | --------------------------- |
| MAJOR | Breaking change             |
| MINOR | Backward-compatible feature |
| PATCH | Bugfix / security fix       |

Example:

```
v1.0.0 → v1.0.1 → v1.1.0 → v2.0.0
```

---

## 4. Tags (Mandatory for Releases)

### Rules

* Every release **must** be tagged
* Tags are immutable
* Never retag or delete published tags

### Example

```bash
git tag -a v1.0.0 -m "API-only production release"
git push origin v1.0.0
```

**Tags define reality.**

---

## 5. Standard Release Procedure

1. Stabilize work in `dev`
2. (Optional) validate via deploy/test branch
3. Finalize last commit
4. Tag the release
5. Deploy **from the tag**
6. Lock the version

No feature commits after tagging.

---

## 6. Hotfix Procedure (Rare)

When a production issue occurs:

1. Checkout `release/x.y`
2. Apply minimal fix
3. Commit
4. Tag new version:

   ```
   v1.0.1
   ```
5. Deploy from the new tag
6. Cherry-pick fix into `dev`

Never merge `dev` back into a release branch.

---

## 7. Commit Message Convention

We use a minimal **Conventional Commit** style:

```
feat: add authentication endpoint
fix: handle nil specification edge case
chore: update README
DB: add index to doctors
TEST: add request specs for auth
```

Rules:

* One clear intent per commit
* No mixed responsibilities

---

## 8. What We Explicitly Avoid

* Long-lived feature branches
* Empty “lock” commits
* Mixing UI into API releases
* Rewriting release history
* Deploying from `dev`

---

## 9. Repository Minimum Standard

Every repo should have:

```
README.md
/docs
  └─ engineering-guidelines.md
```

README should reference the core operating docs:

> ## Engineering Standards
> This project follows **UTech Dynamics Engineering & Release Guidelines** and the related testing standards.

Recommended links:

- [Engineering & Release Guidelines](engineering-guidelines.md)
- [Rails Model Spec Standards](model-spec-standards.md)
- [Testing Roadmap](architecture/testing_roadmap.md)
- [Rails Testing Reference](test/rails_testing_reference.md)

---

## 10. Golden Rules (Summary)

1. Tags mark releases
2. Production deploys from tags
3. `dev` can be messy — releases cannot
4. Fix forward, never rewrite history
5. If unsure, don’t merge

---

**Owner:** UTech Dynamics
**Status:** Stable
**Change policy:** Rare, deliberate updates only
