# cicd

The CICD repository hosts the automated reusable workflow archetypes that build and release each package.
This repository centralises CI/CD processes in a single place. While each consuming repository can be locked down to a version or tag of the cicd repository, consuming projects maintain their references and can update them independently.

### Architecture & Build Split

- **Archetypes (`.github/workflows/*.yml`):** Reusable workflow templates triggered exclusively via `workflow_call` (and `workflow_dispatch` for manual testing). These are provided as objects to be consumed by external repositories (e.g. `python_image`, `ai_services`, etc.) pinned to tags, branches, or commit SHAs. They do not run automatically on push/PR commits to the `cicd` repository itself.
- **Repository Build (`.github/workflows/build.yml`):** The dedicated build pipeline for the `cicd` module itself. On push or pull request to the `cicd` repository, only `build.yml` runs. It invokes `release_only.yml` to automatically calculate versions, tag the repository, and create/publish a source archive with GitHub releases.

---

## Available Templates

### `python_build.yml` — Python Build, Test, and Publish

Full build pipeline for Python packages: runs unit tests with coverage reporting, builds the wheel, and publishes to the PyPI registry.

**Usage — include in your GitHub Actions workflow:**

```yaml
jobs:
  build:
    uses: kewilson-ssai/cicd/.github/workflows/python_build.yml@main
    with:
      python_version: '3.13' # Optional: defaults to 3.13 or ${{ vars.PYTHON_VERSION }}
    secrets: inherit
```

**Variables / Inputs (override at the workflow, repository, or job level):**

| Variable / Input | Default | Description |
|---|---|---|
| `python_version` / `PYTHON_VERSION` | `3.13` | Python Docker image tag (e.g. `3.13`, `3.12`, `3.11`) |
| `python_container` / `PYTHON_CONTAINER` | `python:3.13` | Python container image override (e.g. `python:3.13-slim`) |
| `SKIP_TEST` | `FALSE` | Set to `TRUE` to skip the test job entirely |
| `USE_POETRY` | `TRUE` | Use Poetry to install deps; set `FALSE` for pip |
| `REQUIREMENTS_FILE` | `requirements.txt` | Used when `USE_POETRY=FALSE` |
| `TEST_REQUIREMENTS_FILE` | _(empty)_ | Extra test deps file when `USE_POETRY=FALSE` |
| `COVERAGE_SOURCE` | `.` | Directory passed to `--cov=` |
| `PYTEST_VERSION` | `9.0.3` | Pinned pytest version |
| `PYTEST_COV_VERSION` | `7.1.0` | Pinned pytest-cov version |

**Configuring Python Version in Subprojects:**
- **Via Workflow Input (`with:`):**
  ```yaml
  jobs:
    build:
      uses: kewilson-ssai/cicd/.github/workflows/python_build.yml@main
      with:
        python_version: '3.12'
      secrets: inherit
  ```
- **Via Repository / Organization Variable:** Set `PYTHON_VERSION` (e.g. `3.12`) or `PYTHON_CONTAINER` (e.g. `python:3.12`) in your repository Settings -> Secrets and variables -> Actions -> Variables. All included reusable workflows will automatically inherit and use the configured version if no workflow input is specified.

**Usage — include in your `.gitlab-ci.yml` (legacy):**

```yaml
include:
  - project: isnp/gsai/cicd
    ref: $GSAI_CICD_VERSION
    file: python_build.yml

variables:
  PYTHON_VERSION: "3.13" # Optional, defaults to 3.13

stages:
  - test
  - build
  - publish
```

**Poetry-based repo** (default, no overrides needed):
```yaml
# No variable overrides required — poetry install handles everything.
```

**Pip-based repo** (e.g. ai_services):
```yaml
variables:
  USE_POETRY: "FALSE"
  REQUIREMENTS_FILE: "container/requirements.txt"
  TEST_REQUIREMENTS_FILE: "container/requirements_test.txt"
  COVERAGE_SOURCE: "container"
```

**Artifacts produced** (available to downstream jobs):
- `coverage.xml` — Cobertura coverage report consumed by `sonarcloud-check`
- `test-results.xml` — JUnit test results consumed by `sonarcloud-check`

**SonarCloud integration:**
`sonar.yml` is included automatically by `python_build.yml` and declares `needs: [test_python_module]` with `optional: true`, so it picks up coverage and test result artifacts without any extra configuration. Requires GitLab 14.0+.

---

## Getting started

## Add your files

- [ ] [Create](https://docs.gitlab.com/ee/user/project/repository/web_editor.html#create-a-file) or [upload](https://docs.gitlab.com/ee/user/project/repository/web_editor.html#upload-a-file) files
- [ ] [Add files using the command line](https://docs.gitlab.com/ee/gitlab-basics/add-file.html#add-a-file-using-the-command-line) or push an existing Git repository with the following command:

```
cd existing_repo
git remote add origin https://gitlab.com/isnp/gsai/cicd.git
git branch -M main
git push -uf origin main
```

## Integrate with your tools

- [ ] [Set up project integrations](https://gitlab.com/isnp/gsai/cicd/-/settings/integrations)

## Collaborate with your team

- [ ] [Invite team members and collaborators](https://docs.gitlab.com/ee/user/project/members/)
- [ ] [Create a new merge request](https://docs.gitlab.com/ee/user/project/merge_requests/creating_merge_requests.html)
- [ ] [Automatically close issues from merge requests](https://docs.gitlab.com/ee/user/project/issues/managing_issues.html#closing-issues-automatically)
- [ ] [Enable merge request approvals](https://docs.gitlab.com/ee/user/project/merge_requests/approvals/)
- [ ] [Set auto-merge](https://docs.gitlab.com/ee/user/project/merge_requests/merge_when_pipeline_succeeds.html)

## Test and Deploy

Use the built-in continuous integration in GitLab.

- [ ] [Get started with GitLab CI/CD](https://docs.gitlab.com/ee/ci/quick_start/index.html)
- [ ] [Analyze your code for known vulnerabilities with Static Application Security Testing (SAST)](https://docs.gitlab.com/ee/user/application_security/sast/)
- [ ] [Deploy to Kubernetes, Amazon EC2, or Amazon ECS using Auto Deploy](https://docs.gitlab.com/ee/topics/autodevops/requirements.html)
- [ ] [Use pull-based deployments for improved Kubernetes management](https://docs.gitlab.com/ee/user/clusters/agent/)
- [ ] [Set up protected environments](https://docs.gitlab.com/ee/ci/environments/protected_environments.html)

***

# Editing this README

When you're ready to make this README your own, just edit this file and use the handy template below (or feel free to structure it however you want - this is just a starting point!). Thanks to [makeareadme.com](https://www.makeareadme.com/) for this template.

## Suggestions for a good README

Every project is different, so consider which of these sections apply to yours. The sections used in the template are suggestions for most open source projects. Also keep in mind that while a README can be too long and detailed, too long is better than too short. If you think your README is too long, consider utilizing another form of documentation rather than cutting out information.

## Name
Choose a self-explaining name for your project.

## Description
Let people know what your project can do specifically. Provide context and add a link to any reference visitors might be unfamiliar with. A list of Features or a Background subsection can also be added here. If there are alternatives to your project, this is a good place to list differentiating factors.

## Badges
On some READMEs, you may see small images that convey metadata, such as whether or not all the tests are passing for the project. You can use Shields to add some to your README. Many services also have instructions for adding a badge.

## Visuals
Depending on what you are making, it can be a good idea to include screenshots or even a video (you'll frequently see GIFs rather than actual videos). Tools like ttygif can help, but check out Asciinema for a more sophisticated method.

## Installation
Within a particular ecosystem, there may be a common way of installing things, such as using Yarn, NuGet, or Homebrew. However, consider the possibility that whoever is reading your README is a novice and would like more guidance. Listing specific steps helps remove ambiguity and gets people to using your project as quickly as possible. If it only runs in a specific context like a particular programming language version or operating system or has dependencies that have to be installed manually, also add a Requirements subsection.

## Usage
Use examples liberally, and show the expected output if you can. It's helpful to have inline the smallest example of usage that you can demonstrate, while providing links to more sophisticated examples if they are too long to reasonably include in the README.

## Support
Tell people where they can go to for help. It can be any combination of an issue tracker, a chat room, an email address, etc.

## Roadmap
If you have ideas for releases in the future, it is a good idea to list them in the README.

## Contributing
State if you are open to contributions and what your requirements are for accepting them.

For people who want to make changes to your project, it's helpful to have some documentation on how to get started. Perhaps there is a script that they should run or some environment variables that they need to set. Make these steps explicit. These instructions could also be useful to your future self.

You can also document commands to lint the code or run tests. These steps help to ensure high code quality and reduce the likelihood that the changes inadvertently break something. Having instructions for running tests is especially helpful if it requires external setup, such as starting a Selenium server for testing in a browser.

## Authors and acknowledgment
Show your appreciation to those who have contributed to the project.

## License
For open source projects, say how it is licensed.

## Project status
If you have run out of energy or time for your project, put a note at the top of the README saying that development has slowed down or stopped completely. Someone may choose to fork your project or volunteer to step in as a maintainer or owner, allowing your project to keep going. You can also make an explicit request for maintainers.

<!-- approval rule canary test, safe to revert -->
