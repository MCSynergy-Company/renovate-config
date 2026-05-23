# Repository Dependency Automation (Renovate)

This repository uses [Renovate](https://docs.renovatebot.com/) to automatically monitor, test, and update dependencies. Centralizing this configuration ensures consistent dependency guardrails across our organization.

## ⚙️ Core Guardrails

* **Timezone**: Set to `Europe/Amsterdam` to align update schedules with localized working hours.
* **Guard Band (`minimumReleaseAge`)**: Standard updates are delayed by **3 days** after their initial release. This protects our builds from immediate, faulty upstream releases (e.g., a broken minor version yanked shortly after launch).
* **Automerge**: Disabled (`false`). All dependency upgrades must pass through human code review and regular CI/CD checks before entering production.

---

## 🔍 Digest Pinning & Developer Readability

To maximize build reproducibility without sacrificing developer readability, this configuration enforces global SHA/digest pinning (`:pinDigests`) while ensuring human-readable versions are **always** preserved or generated:

* **Docker Images**: 
  * Every image is pinned to an immutable `@sha256` digest.
  * If a developer omits a version tag (implicitly using `latest`), Renovate will automatically look up the active underlying semantic version and rewrite the definition to include both the human-readable tag and the SHA (e.g., `image: postgis/postgis:16-3.4@sha256:...`).
* **GitHub Actions**: 
  * Actions are pinned to strict commit SHAs for security. 
  * Renovate automatically appends an inline comment with the human-readable version tag (e.g., `uses: actions/checkout@b4ffde... # v4.1.1`) so developers can easily see what version is running at a glance.

---

## 🛡️ Security Strategy

Security vulnerabilities bypass standard delay constraints to ensure rapid patching.

* **Immediate Execution**: Vulnerability patches run `at any time` and ignore the 3-day buffer rule (`minimumReleaseAge: null`).
* **Isolation**: Security fixes are grouped into a dedicated PR titled **"Security Fixes"** and tagged with a `security` label for immediate visibility.

---

## 📦 Custom Package Registries

For `.NET / C#` projects, Renovate is configured to look beyond the public NuGet gallery to check for internal releases:
1. **Public**: `https://api.nuget.org/v3/index.json`
2. **Internal**: `https://nuget.riderapp.nl/v3/index.json`

---

## 🗂️ PR Grouping & Scheduling Rules

To prevent developers from experiencing "PR fatigue," updates are sorted into specific streams:

### 1. Weekly Non-Major Updates
* **Target**: Minor and patch updates for packages that are already at version `1.0.0` or higher.
* **Schedule**: Delivered **Monday mornings before 8:00 AM Amsterdam time**. 
* **Behavior**: All matching updates are batched into a single, combined Pull Request to keep the repository history clean.

### 2. Internal RideR Updates
* **Target**: Packages originating from our own ecosystem (`MCSynergy.RideR.*`).
* **Behavior**: Separated into an independent tracking group. This ensures internal organizational alignment doesn't get lost or bundled alongside public open-source updates.