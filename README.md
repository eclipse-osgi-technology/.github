# .github
github organisation repository, defaults for all other Repositories

## Reusable workflows for the Maven repos

The CI logic and every action version live in the reusable workflows of this
repo. A repo only carries the stubs below; it chooses its JDKs in `build.yml`
and `release.yml` and nothing else.

The stubs that are the same in every repo (`codeql.yml`,
`scorecard-analysis.yml` and `stale.yml`) are rolled out and kept identical by
the otterdog blueprints in
[.eclipsefdn](https://github.com/eclipse-osgi-technology/.eclipsefdn/tree/main/otterdog/blueprints):
a repo gets them by being listed there, and otterdog opens a pull request when a
stub is missing or differs.

| Workflow | Does |
|---|---|
| `maven-build.yml` | builds and tests with every JDK of `jdks`, plus bnd-next (allowed to fail), checks the javadoc, and deploys a `-SNAPSHOT` from `main` |
| `maven-release.yml` | deploys a release when a tag equal to the version is pushed |
| `codeql.yml` | CodeQL analysis of the Java code and the workflows, without a build |
| `scorecard.yml` | OSSF Scorecard analysis |
| `stale.yml` | marks issues and pull requests stale after a year without activity and closes them three weeks later |

The version lives only in `.mvn/maven.config`, e.g. `-Drevision=1.4.0-SNAPSHOT`,
and the POMs use `${revision}`. To release 1.4.0, set `-Drevision=1.4.0`, push
the tag `1.4.0` (or publish a GitHub release with the new tag `1.4.0`), and then
move to `-Drevision=1.5.0-SNAPSHOT`. Only the tag deploys the release: the build
of `main` deploys nothing while the version has no `-SNAPSHOT`.

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
