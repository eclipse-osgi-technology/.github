# .github
github organisation repository, defaults for all other Repositories

## Reusable workflows for the Maven repos

The CI logic and every action version live in the reusable workflows of this
repo. A repo only carries the stubs below; it chooses its JDKs in `build.yml`
and `release.yml` and nothing else.

| Workflow | Does |
|---|---|
| `maven-build.yml` | builds and tests with every JDK of `jdks`, plus bnd-next (allowed to fail), checks the javadoc, and deploys a `-SNAPSHOT` from `main` |
| `maven-release.yml` | deploys a release when a tag equal to the version is pushed |
| `codeql.yml` | CodeQL analysis without a build |
| `scorecard.yml` | OSSF Scorecard analysis |

The version lives only in `.mvn/maven.config`, e.g. `-Drevision=1.4.0-SNAPSHOT`,
and the POMs use `${revision}`. To release 1.4.0, set `-Drevision=1.4.0`, push
the tag `1.4.0` (or publish a GitHub release with the new tag `1.4.0`), and then
move to `-Drevision=1.5.0-SNAPSHOT`.

### `.github/workflows/build.yml`

```yaml
name: build
on:
  push:
    branches: [ main ]
  pull_request:
  workflow_dispatch:
  schedule:
    - cron: '23 3 * * 1'
permissions:
  contents: read
jobs:
  build:
    permissions:
      contents: read
      # the test report is published as a check run
      checks: write
    uses: eclipse-osgi-technology/.github/.github/workflows/maven-build.yml@main
    secrets: inherit
    with:
      jdks: '["21"]'
      jdk-main: '21'
      deploy: ${{ github.event_name == 'push' || github.event_name == 'schedule' || github.event_name == 'workflow_dispatch' }}
```

### `.github/workflows/release.yml`

```yaml
name: release
on:
  push:
    tags:
      - '[0-9]*.[0-9]*.[0-9]*'
permissions:
  contents: read
jobs:
  release:
    uses: eclipse-osgi-technology/.github/.github/workflows/maven-release.yml@main
    secrets: inherit
    with:
      jdk: '21'
```

### `.github/workflows/codeql.yml`

```yaml
name: CodeQL
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '17 4 * * 3'
permissions:
  contents: read
jobs:
  analyze:
    permissions:
      contents: read
      security-events: write
    uses: eclipse-osgi-technology/.github/.github/workflows/codeql.yml@main
```

### `.github/workflows/scorecard-analysis.yml`

```yaml
name: Scorecard analysis
on:
  branch_protection_rule:
  schedule:
    - cron: '43 5 * * 2'
  push:
    branches: [ main ]
permissions: read-all
jobs:
  analysis:
    permissions:
      security-events: write
      id-token: write
    uses: eclipse-osgi-technology/.github/.github/workflows/scorecard.yml@main
```
