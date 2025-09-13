# Magician79 Coding Standard
[![CI](https://github.com/magician79/coding-standard/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/magician79/coding-standard/actions/workflows/ci.yml)

Shared coding standards for PHP projects:

- **PHPCS** — Doctrine Coding Standard (PSR-12 + curated Slevomat) as a baseline, with Pragmatic and Strict profiles
- **PHPStan** — Pragmatic and Strict configs
- **Syntax linting** — via php-parallel-lint (with optional var-dump check)

Packaged for easy reuse across repositories with sensible defaults and stricter opt-ins.

---

## Table of Contents

- [What this package provides](#what-this-package-provides)
- [Installation](#installation)
- [PHPCS ruleset profiles](#phpcs-ruleset-profiles)
- [PHPStan profiles](#phpstan-profiles)
- [PHP syntax linting](#php-syntax-linting-php-parallel-lint)
- [Quick usage options](#quick-usage-options)
- [Choosing a PHPCS profile](#choosing-a-phpcs-profile)
- [Excluding or tuning sniffs](#excluding-or-tuning-sniffs)
- [PHPStan: extend in a project](#phpstan-extend-in-a-project-recommended-for-larger-repos)
- [Composer and installation behavior](#composer-and-installation-behavior-faq)
- [CI examples](#ci-examples-consumer-project)
- [Pre-commit hook example](#pre-commit-hook-example-consumer-project)
- [Versioning and upgrade policy](#versioning-and-upgrade-policy)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## What this package provides

- **PHPCS rulesets**
  - *Pragmatic* (default): Based on Doctrine coding standard, this profile balances strictness with low noise and high autofix coverage. It avoids overlapping or conflicting PHPCS sniffs by carefully selecting compatible rules.
  - *Strict* (opt-in): Extends the Pragmatic profile with a stricter set of sniffs from Doctrine and Slevomat for maximum code quality and rigor, suitable for new or greenfield modules.

- **PHPStan configurations**
  - *Pragmatic* (default): Level 6 + noise reduction toggles.
  - *Strict* (opt-in): Level max, bleedingEdge, phpstan-strict-rules included.

- **PHP syntax linting**
  - `php-parallel-lint` (fast, parallel parse error detection)
  - Optional `var-dump-check` to block accidental debug prints.

- **Optional Composer scripts**
  - Predefined tasks (`cs`, `stan`, `check`, etc.) for quick setup.

- **Out-of-the-box tooling**
  - PHPCS, Doctrine Coding Standard, Slevomat, PHPStan, php-parallel-lint are bundled so consumers don’t need to install them separately.

---

## Installation

Install in your project as a dev dependency:

```bash
composer require --dev magician79/coding-standard
```

This installs (into your project’s `vendor/`):

- `doctrine/coding-standard`
- `squizlabs/php_codesniffer`
- `slevomat/coding-standard`
- `dealerdirect/phpcodesniffer-composer-installer`
- `phpstan/phpstan` (+ `phpstan/phpstan-strict-rules`)
- `php-parallel-lint/php-parallel-lint`
- (optional) `php-parallel-lint/php-var-dump-check`, `php-parallel-lint/php-console-highlighter`

Note: tools are installed in dev mode (`composer install`). They are omitted in production installs (`composer install --no-dev`).

---

## PHPCS ruleset profiles

There are two PHPCS profiles in this package:

- **Pragmatic profile (recommended default)**
  - File: `vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml`
  - Intent: Low-friction adoption. Doctrine baseline + curated Slevomat. Enforces `declare(strict_types=1)`, type hints, imports hygiene, strict comparisons, unused code cleanup, and a pragmatic line length rule. Most findings are auto-fixable via `phpcbf`.

- **Strict profile (opt-in)**
  - File: `vendor/magician79/coding-standard/ruleset/phpcs.strict.xml`
  - Intent: Maximum rigor by extending the Doctrine baseline and Pragmatic profile with nearly all Slevomat sniffs. Best for new projects or modules. Expect to exclude or tune some sniffs.

---

## PHPStan profiles

This package ships two PHPStan configurations:

- **Pragmatic (default)**
  - File: `vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist`
  - Level 6 with noise-reducing toggles. Designed for adoption on existing code.

- **Strict (opt-in)**
  - File: `vendor/magician79/coding-standard/phpstan/strict.neon.dist`
  - Level max, phpstan-strict-rules included. Ideal for greenfield modules.

Run directly:

```bash
# Pragmatic
vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist

# Strict
vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/strict.neon.dist
```

---

## PHP syntax linting (php-parallel-lint)

`php-parallel-lint` runs a fast, parallel parse check of your PHP files. It catches syntax/parse errors before PHPCS or PHPStan run.

Useful flags:
- `-j 8` or higher for concurrency
- `--colors` for readable output
- `--exclude` to skip vendor/cache/build directories
- `--blame` to show the author of the failing line

Run directly:

```bash
# Basic syntax check (fast fail)
vendor/bin/parallel-lint --colors -j 8 --exclude vendor --exclude var --exclude storage --exclude cache --exclude .git .
```

Optional: disallow debug prints using `php-var-dump-check`:

```bash
vendor/bin/var-dump-check --skip-dir=vendor --skip-dir=var --skip-dir=storage --skip-dir=cache .
```

---

## Quick usage options

### Option A: Use shared profiles via Composer scripts

Add to your project’s `composer.json`:

```json
{
  "scripts": {
    "lint:php": "parallel-lint --colors -j 8 --exclude vendor --exclude var --exclude storage --exclude cache --exclude .git .",
    "lint:debug": "var-dump-check --skip-dir=vendor --skip-dir=var --skip-dir=storage --skip-dir=cache .",

    "cs": "sh -c 'if [ -f phpcs.xml ]; then vendor/bin/phpcs -p --standard=phpcs.xml; else vendor/bin/phpcs --standard=vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml; fi'",
    "cs:strict": "sh -c 'if [ -f phpcs.xml ]; then vendor/bin/phpcs -p --standard=phpcs.xml; else vendor/bin/phpcs --standard=vendor/magician79/coding-standard/ruleset/phpcs.strict.xml; fi'",
    "fix": "sh -c 'if [ -f phpcs.xml ]; then vendor/bin/phpcbf --standard=phpcs.xml; else vendor/bin/phpcbf --standard=vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml; fi'",
    "fix:strict": "sh -c 'if [ -f phpcs.xml ]; then vendor/bin/phpcbf --standard=phpcs.xml; else vendor/bin/phpcbf --standard=vendor/magician79/coding-standard/ruleset/phpcs.strict.xml; fi'", 

    "stan": "sh -c 'if [ -f phpstan.neon.dist ]; then vendor/bin/phpstan analyse -c phpstan.neon.dist; else vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist; fi'",
    "stan:strict": "sh -c 'if [ -f phpstan.neon.dist ]; then vendor/bin/phpstan analyse -c phpstan.neon.dist; else vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/strict.neon.dist; fi'",

    "check": [
      "@lint:php",
      "@cs",
      "@stan"
    ],
    "check:strict": [
      "@lint:php",
      "@cs:strict",
      "@stan:strict"
    ]
  }
}
```

Run:
- `composer lint:php`
- `composer cs`
- `composer fix`
- `composer stan`
- `composer check` (syntax lint → PHPCS → PHPStan)

#### Option B: Extend rules in your project (recommended for larger projects)

Create `phpcs.xml` in your project root:

```xml
<?xml version="1.0"?>
<ruleset name="Project Standard">
  <!-- Choose one to start from -->
  <rule ref="vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml"/>
  <!-- Or use strict -->
  <!-- <rule ref="vendor/magician79/coding-standard/ruleset/phpcs.strict.xml"/> -->

  <!-- Scope your paths -->
  <file>src</file>
  <file>app</file>
  <file>config</file>
  <file>tests</file>
  <exclude-pattern>vendor/*</exclude-pattern>
  <exclude-pattern>var/*</exclude-pattern>
  <exclude-pattern>generated/*</exclude-pattern>

  <!-- Optional project-specific tweaks -->
  <!-- Example: exclude a legacy folder -->
  <!-- <exclude-pattern>src/Legacy/*</exclude-pattern> -->

  <!-- Disable or tune a sniff -->
  <!-- <exclude name="SlevomatCodingStandard.Namespaces.AlphabeticallySortedUses"/> -->
  <!--
  <rule ref="SlevomatCodingStandard.Operators.DisallowEqualOperators">
    <properties>
      <property name="error" value="false"/>
      <property name="warning" value="true"/>
    </properties>
  </rule>
  -->
</ruleset>
```
Run:
- `vendor/bin/phpcs`
- `vendor/bin/phpcbf`

---

## Choosing a PHPCS profile

- **Pragmatic**: best for existing/legacy code.
- **Strict**: best for new projects or modernized code.

You can also mix: Pragmatic globally, Strict on certain paths:

```xml
<file>src/NewModule</file>
<rule ref="vendor/magician79/coding-standard/ruleset/phpcs.strict.xml">
  <include-pattern>src/NewModule/*</include-pattern>
</rule>
```

---

## Excluding or tuning sniffs

You can always exclude or adjust any sniff in your project’s `phpcs.xml`:

- Disable globally:
  ```xml
  <exclude name="SlevomatCodingStandard.Functions.UnusedParameter"/>
  ```

- Demote error to warning:
  ```xml
  <rule ref="SlevomatCodingStandard.Operators.DisallowEqualOperators">
    <properties>
      <property name="error" value="false"/>
      <property name="warning" value="true"/>
    </properties>
  </rule>
  ```

- Exclude by path:
  ```xml
  <exclude-pattern>src/Legacy/*</exclude-pattern>
  ```

---

## PHPStan: extend in a project (recommended for larger repos)

Create a `phpstan.neon.dist` file in your project root and include one of the shared profiles.

Pragmatic baseline:

```neon
includes:
  - vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist

parameters:
  level: 7
  paths:
    - src
    - tests
  parallel:
    maximumNumberOfProcesses: 6

# Optionally include a project-specific baseline
# includes:
#   - phpstan-baseline.neon
```

Or Strict mode:

```neon
includes:
  - vendor/magician79/coding-standard/phpstan/strict.neon.dist

parameters:
  paths:
    - src
    - tests

# Optional extensions (install in your project first):
# includes:
#   - vendor/phpstan/phpstan-doctrine/extension.neon
#   - vendor/phpstan/phpstan-phpunit/extension.neon
```

Baselines:
- This package does not ship a baseline. Generate one in your project only if needed:
  - `vendor/bin/phpstan analyse --generate-baseline=phpstan-baseline.neon`
- Consider starting with Pragmatic without a baseline; use Strict for new modules or as a second CI job.

Mix-and-match:
- Use Pragmatic globally and enforce Strict on certain paths (e.g., `src/NewModule`) by running a second job or restricting analysis paths:
```bash
vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/strict.neon.dist src/NewModule
```

Performance tips:
- Parallel analysis is enabled by default; tweak `maximumNumberOfProcesses` per project.
- Limit paths for faster iteration during refactors.
- Ensure `var/cache/phpstan` is writable in your environment.

---

## Composer and installation behavior (FAQ)

- Add under `require-dev` in consumer projects.
- In dev installs, all tools are present.
- In production (`--no-dev`), the package is omitted and none of the tooling is present.
- Rule of thumb:
  - Shared package: put tools in `require` so they’re available whenever the consumer installs your package as a dev dependency.
  - Consumer project: add `magician79/coding-standard` under `require-dev`.

---

## CI examples (consumer project)

Run php-parallel-lint first to fail fast, then PHPCS and PHPStan. This workflow uses local config if present, otherwise falls back to vendor defaults.

Minimal pragmatic job:

```yaml
name: Code Quality

on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          tools: composer:v2
          coverage: none
      - run: composer install --no-interaction --prefer-dist

      - name: PHP syntax lint (fast fail)
        run: vendor/bin/parallel-lint --colors -j 8 --exclude vendor --exclude var --exclude storage --exclude cache --exclude .git .

      - name: PHPCS (Pragmatic, local override if exists)
        run: |
          if [ -f phpcs.xml ]; then
            vendor/bin/phpcs -p --standard=phpcs.xml
          else
            vendor/bin/phpcs -p --standard=vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml --ignore=vendor/* -q .
          fi

      - name: PHPStan (Pragmatic, local override if exists)
        run: |
          if [ -f phpstan.neon.dist ]; then
            vendor/bin/phpstan analyse -c phpstan.neon.dist --no-progress
          else
            vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist --no-progress .
          fi
```

Separate jobs for strict profiles:
```yaml
jobs:
  rule-pragmatic:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          tools: composer:v2
      - run: composer install --no-interaction --prefer-dist
      - run: vendor/bin/parallel-lint --colors -j 8 --exclude vendor --exclude var --exclude storage --exclude cache --exclude .git .
      
      - name: PHPCS (Pragmatic, local override if exists)
        run: |
          if [ -f phpcs.xml ]; then
            vendor/bin/phpcs -p --standard=phpcs.xml
          else
            vendor/bin/phpcs -p --standard=vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml --ignore=vendor/* -q .
          fi

      - name: PHPStan (Pragmatic, local override if exists)
        run: |
          if [ -f phpstan.neon.dist ]; then
            vendor/bin/phpstan analyse -c phpstan.neon.dist --no-progress
          else
            vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist --no-progress .
          fi

  rule-strict:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          tools: composer:v2
      - run: composer install --no-interaction --prefer-dist
      - run: vendor/bin/parallel-lint --colors -j 8 --exclude vendor --exclude var --exclude storage --exclude cache --exclude .git .

      - name: PHPCS (Strict, local override if exists)
        run: |
          if [ -f phpcs.xml ]; then
            vendor/bin/phpcs -p --standard=phpcs.xml
          else
            vendor/bin/phpcs -p --standard=vendor/magician79/coding-standard/ruleset/phpcs.strict.xml --ignore=vendor/* -q .
          fi

      - name: PHPStan (Strict, local override if exists)
        run: |
          if [ -f phpstan.neon.dist ]; then
            vendor/bin/phpstan analyse -c phpstan.neon.dist --no-progress
          else
            vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/strict.neon.dist --no-progress .
          fi
      # Optionally scope strict to a directory:
      # - run: vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/strict.neon.dist src/NewModule
```

---

## Pre-commit hook example (consumer project)

Create `.git/hooks/pre-commit`:

```sh
#!/bin/sh
vendor/bin/parallel-lint -j 8 --exclude vendor --exclude var --exclude storage --exclude cache --exclude .git . || exit 1

if [ -f phpcs.xml ]; then
  vendor/bin/phpcs --standard=phpcs.xml || exit 1
else
  vendor/bin/phpcs --standard=vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml --ignore=vendor/* -q . || exit 1
fi

if [ -f phpstan.neon.dist ]; then
  vendor/bin/phpstan analyse -c phpstan.neon.dist || exit 1
else
  vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist . || exit 1
fi
```

Make it executable:

```sh
chmod +x .git/hooks/pre-commit
```

---

## Versioning and upgrade policy

- SemVer:
  - MAJOR: enabling stricter default rules or changing defaults in a way that could fail existing code.
  - MINOR: new profiles, optional rules, docs, examples that don’t change defaults.
  - PATCH: bug fixes and small non-breaking updates.
- Consumers can pin the package version in `composer.json` and choose when to upgrade.
- As of v2.0.0, the Pragmatic and Strict PHPCS profiles are based on the Doctrine Coding Standard with curated Slevomat rules. This reduces overlaps and conflicts. External usage (filenames, commands) remains unchanged. Consumers should re-check their codebase when upgrading to v2.x.

---

## Troubleshooting

- **PHPCS can’t find Slevomat rules**: ensure `dealerdirect/phpcodesniffer-composer-installer` is installed (brought by this package). Run `composer dump-autoload` if needed.
- **Too many findings initially**: start with Pragmatic; run `phpcbf` to auto-fix; then suppress or exclude remaining sniffs that don’t fit your context.
- **PHPStan seems slow**: enable `parallel` in your project config and scope paths to `src` and `tests`.
- **php-parallel-lint finds errors your PHP version accepts**: remember it validates against the PHP version it runs under. In CI, use a matrix or the production PHP version to ensure compatibility.

---

## License

MIT. See LICENSE in this repository.
