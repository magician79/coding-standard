### Magician79 Coding Standard
[![CI](https://github.com/magician79/coding-standard/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/magician79/coding-standard/actions/workflows/ci.yml)

Shared coding standards for PHP projects: PHPCS (PSR-12 + Slevomat), PHPStan (pragmatic/strict profiles), and PHP syntax linting via php-parallel-lint. Packaged for easy reuse across repositories.

#### What this package provides
- PHPCS rulesets:
  - Pragmatic profile (default): PSR-12 + curated Slevomat sniffs with low noise and high auto-fix coverage.
  - Strict profile (opt-in): PSR-12 + the full Slevomat standard (with a few global excludes). Best for greenfield or teams seeking maximum rigor.
- PHPStan configurations:
  - Pragmatic profile (default): level 6 plus noise reduction toggles.
  - Strict profile (opt-in): level max, bleedingEdge, and phpstan-strict-rules included.
- PHP syntax linting:
  - php-parallel-lint (fast, parallel parse error detection)
  - Optional var-dump checker to block accidental debug prints.
- Optional Composer scripts (examples) you can copy into consumer projects.
- Out-of-the-box tooling: this package requires PHPCS, Slevomat, PHPStan, php-parallel-lint, and the installer plugin, so consumers don’t need to install them separately.

---

### Installation

Install in your project as a dev dependency:
```bash
composer require --dev magician79/coding-standard:^1.0
```
This installs (into your project’s vendor/):
- squizlabs/php_codesniffer
- slevomat/coding-standard
- dealerdirect/phpcodesniffer-composer-installer (auto-registers standards)
- phpstan/phpstan (+ phpstan/phpstan-strict-rules)
- php-parallel-lint/php-parallel-lint (+ optional: php-parallel-lint/php-var-dump-check, php-parallel-lint/php-console-highlighter)

Note: Tools are present when you run `composer install` (dev mode). They are omitted in production installs: `composer install --no-dev`.

---

### PHPCS ruleset profiles

There are two PHPCS profiles in this package:

- Pragmatic profile (recommended default)
  - File: `vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml`
  - Intent: Low-friction adoption. Enforces `declare(strict_types=1)`, type hints, imports hygiene, strict comparisons, unused code cleanup, and a pragmatic line length rule. Most findings are auto-fixable via phpcbf.

- Strict profile (opt-in)
  - File: `vendor/magician79/coding-standard/ruleset/phpcs.strict.xml`
  - Intent: Maximum rigor by enabling the entire Slevomat standard (noisier). Ideal for new code. Expect to exclude or tune some sniffs to match project preferences.

---

### PHPStan profiles

This package ships two PHPStan configurations:

- Pragmatic (default)
  - File: `vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist`
  - Level 6 with noise-reducing toggles. Designed for quick adoption on existing code.

- Strict (opt-in)
  - File: `vendor/magician79/coding-standard/phpstan/strict.neon.dist`
  - Level max, bleedingEdge, phpstan-strict-rules included, unmatched @ignore reporting. Best for greenfield modules or when modernizing code.

Run directly:
```bash
# Pragmatic
vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist

# Strict
vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/strict.neon.dist
```

---

### PHP syntax linting (php-parallel-lint)

php-parallel-lint runs a fast, parallel parse check of your PHP files. It catches hard failures (syntax/parse errors, stray merge markers) before PHPCS or PHPStan run.

Useful flags:
- `-j 8` or higher for concurrency
- `--colors` for readable output
- `--exclude` to skip vendor/cache/build directories
- `--blame` to show the author of the failing line (helpful in repos with multiple contributors)

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

### Quick usage options

#### Option A: Use the shared profiles directly via Composer scripts
Add scripts to your project’s `composer.json`:
```json
{
  "scripts": {
    "lint:php": "parallel-lint --colors -j 8 --exclude vendor --exclude var --exclude storage --exclude cache --exclude .git .",
    "lint:debug": "var-dump-check --skip-dir=vendor --skip-dir=var --skip-dir=storage --skip-dir=cache .",

    "cs": "phpcs --standard=vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml",
    "cs:strict": "phpcs --standard=vendor/magician79/coding-standard/ruleset/phpcs.strict.xml",
    "fix": "phpcbf --standard=vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml",

    "stan": "phpstan analyse -c vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist",
    "stan:strict": "phpstan analyse -c vendor/magician79/coding-standard/phpstan/strict.neon.dist",

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
- `composer check` (runs syntax lint → PHPCS → PHPStan)

#### Option B: Extend rules in a project `phpcs.xml` (recommended for larger projects)
Create `phpcs.xml` in your project root:
```xml
<?xml version="1.0"?>
<ruleset name="Project Standard">
  <!-- Choose one to start from -->
  <rule ref="vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml"/>
  <!-- or -->
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

### Choosing a PHPCS profile

- Start with Pragmatic for existing/legacy code to get quick wins with minimal friction. It emphasizes:
  - `declare(strict_types=1)`
  - Type hints (parameters/returns/properties)
  - Imports hygiene (unused uses, ordering)
  - Strict comparisons
  - Unused variables/parameters
  - Auto-fixable cleanup rules
- Use Strict for new projects or modules where you’re comfortable meeting broader Slevomat expectations. It references the entire Slevomat standard and may require additional exclusions to align with your preferences.

You can mix: keep Pragmatic globally and selectively enforce Strict on specific paths:
```xml
<file>src/NewModule</file>
<rule ref="vendor/magician79/coding-standard/ruleset/phpcs.strict.xml">
  <include-pattern>src/NewModule/*</include-pattern>
</rule>
```

---

### Excluding or tuning sniffs

You can always exclude or adjust any sniff in your project’s `phpcs.xml`:
- Disable globally:
```xml
<exclude name="SlevomatCodingStandard.Functions.UnusedParameter"/>
```
- Demote an error to a warning:
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

### PHPStan: extend in a project (recommended for larger repos)

Extend in a project (recommended for larger repos):

phpstan.neon.dist (Pragmatic baseline with project overrides)
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

Or strict:
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

### Composer and installation behavior (FAQ)

- Add this package under `require-dev` in consumer projects. When you run `composer install` (dev mode), the tools from this package’s `require` are installed into the consumer’s `vendor/`.
- In production installs (`composer install --no-dev`), this package is not installed and none of the tooling is present.
- Rule of thumb:
  - Shared package: put tools in `require` so they’re available whenever the consumer installs your package as a dev dependency.
  - Consumer project: add `magician79/coding-standard` under `require-dev`.

---

### CI examples (consumer project)

Run php-parallel-lint first to fail fast, then PHPCS and PHPStan.

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

      - name: PHPCS (Pragmatic)
        run: vendor/bin/phpcs --standard=vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml

      - name: PHPStan (Pragmatic)
        run: vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist --no-progress
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
      - run: vendor/bin/phpcs --standard=vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml
      - run: vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist --no-progress

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
      - run: vendor/bin/phpcs --standard=vendor/magician79/coding-standard/ruleset/phpcs.strict.xml
      - run: vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/strict.neon.dist --no-progress
      # Optionally scope strict to a directory:
      # - run: vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/strict.neon.dist src/NewModule
```

---

### Pre-commit hook example (consumer project)

```sh
#!/bin/sh
vendor/bin/parallel-lint -j 8 --exclude vendor --exclude var --exclude storage --exclude cache --exclude .git . || exit 1
vendor/bin/phpcs --standard=vendor/magician79/coding-standard/ruleset/phpcs.pragmatic.xml || exit 1
vendor/bin/phpstan analyse -c vendor/magician79/coding-standard/phpstan/pragmatic.neon.dist || exit 1
```
Make executable:
```sh
chmod +x .git/hooks/pre-commit
```

---

### Versioning and upgrade policy

- SemVer:
  - MAJOR: enabling stricter default rules or changing defaults in a way that could fail existing code.
  - MINOR: new profiles, optional rules, docs, examples that don’t change defaults.
  - PATCH: bug fixes and small non-breaking updates.
- Consumers can pin the package version in `composer.json` and choose when to upgrade.

---

### Troubleshooting

- PHPCS can’t find Slevomat rules: ensure `dealerdirect/phpcodesniffer-composer-installer` is installed (brought by this package). Run `composer dump-autoload` if needed.
- Too many findings initially: start with Pragmatic; run `phpcbf` to auto-fix; then suppress or exclude remaining sniffs that don’t fit your context.
- PHPStan seems slow: enable `parallel` in your project config and scope paths to `src` and `tests`.
- php-parallel-lint finds errors your PHP version accepts: remember it validates against the PHP version it runs under. In CI, use a matrix or the production PHP version to ensure compatibility.

---

### License

MIT. See LICENSE in this repository.
