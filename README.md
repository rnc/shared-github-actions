# Introduction

This repository is used to store GitHub Action workflows (and other stuff) that we'll re-use in other PNC repositories

These workflows are designed for Maven-based Java projects and can be included
in other repositories using the `workflow_call` mechanism.

# Available Workflows

## Maven CI (`maven-ci.yml`)
Standard Continuous Integration workflow for Maven projects that we use to test
PRs. For some of the workflows, we can also further customize it by specifying
the Java version etc. It is possible to use this within a matrix job.

- **Tasks**: Checkout code, set up Java (default: 21), set up Maven (default: 3.9.15), run build command (`mvn
  clean install`), and check for code formatting errors.

<details>
<summary>Here is an example of using this in a matrix job</summary>

```yaml
    strategy:
      fail-fast: false
      matrix:
        maven-version: [3.3.9, 3.5.4, 3.6.3, 3.8.7, 3.9.0, 3.9.8]
        java-version: ['11']
        # Use include to add a specific Maven/JDK combination
        include:
          - maven-version: 3.9.11
            java-version: '17'
          - maven-version: 3.9.15
            java-version: '17'
    uses: project-ncl/shared-github-actions/.github/workflows/maven-ci.yml@ef31e53140344cac0421b7211cc0940182409106 # v0.0.16
    with:
      build_command: "mvn -B -V clean install -Prun-its"
      java_version: ${{ matrix.java-version }}
      maven_version: ${{ matrix.maven-version }}
```
</details>

## Maven Release (`maven-release.yml`)
Workflow for performing a release to Maven Central (Sonatype).

- **Tasks**: Configures Git, sets up Java and GPG, performs `release:prepare`
  and `release:perform`, and pushes changes/tags back to the repository. The
  next version is set by bumping the patch version by 1 and putting the
  `-SNAPSHOT` suffix.


## Maven Snapshot (`maven-snapshot.yml`)
Workflow for deploying snapshot versions to Maven Central.

- **Tasks**: Deploy our SNAPSHOT version of our project to Maven Central
  Optionally builds and pushes a Quarkus Jib image to Quay.io.


## Maven Set Version (`maven-set-version.yml`)
Workflow to update the version in a Maven `pom.xml`.

- **Tasks**: Updates the version using `versions:set` and commits/pushes the change.
 
This workflow should be manually called and an example of that may be seen [here](https://github.com/project-ncl/environment-driver/blob/main/.github/workflows/maven-set-version.yml). Its recommended that `on.workflow_dispatch` is used so the user can enter the appropriate values e.g.
```
on:
  workflow_dispatch: # Manual trigger
    inputs:
      new_version:
        description: '[Required] Version to set'
        required: true
        type: string
...
jobs:
    call-maven-set-version-job:
      permissions:
        contents: write # needed to push commit and tag back to the repository
      uses: project-ncl/shared-github-actions/.github/workflows/maven-set-version.yml@504efc82df7d4f73e3bd2b67bec31bcd532ed9e4 # v0.0.14
      with:
        new_version: ${{ inputs.new_version }}
```


## Maven Jib (`maven-jib.yml`)
*Work in Progress* workflow to build Jib images and upload to a container
registry.


## Validate GitHub Action (`validate-gh-action.yml`)
Workflow used to lint and validate GitHub Actions using `actionlint` and
`zizmor`. This is used in this repository's `validate.yml` to run for PRs.

## Usage Example

To use one of these workflows in your repository, add a new workflow file
(e.g., `.github/workflows/ci.yml`) and reference the shared workflow:

```yaml
name: Run PR test

on:
  pull_request:
    branches: [ * ]

jobs:
  build:
    uses: project-ncl/shared-github-actions/.github/workflows/maven-ci.yml@<sha> # <tag>
    with:
      # if you want to customize the java_version
      java_version: '17'
```

A Github repository example using those workflows can be found
(here)[https://github.com/project-ncl/environment-driver/tree/main/.github/workflows]

# Misc

## Dependabot

A sample dependabot file in `.github/dependabot.yml` is available that is both used in this repository and may be copied to other ProjectNCL repositories.

## GitHub Releases
A sample `.github/release.yml` configuration file from the [GitHub documentation](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes#configuring-automatically-generated-release-notes) has been added that may be copied to other ProjectNCL repositories.

## Github Action validations
We use (zizmor)[https://docs.zizmor.sh/] and
[actionlint](https://github.com/rhysd/actionlint) to validate that our Github
Actions are secure.

One of the requirements is to explicitly define a permissions key to specify
the level of access that the injected `GITHUB_TOKEN` should have.

For workflows that do not require to `git push` any changes, we set an empty
permissions field:
```
permissions: {}
```

and for those that do:
```
permissions:
  contents: write
```
For example, `maven-release.yml` requires to `git push` changes back to the
repository and needs those permissions.

Note that the final workflow should also specify the same permissions as the
shared workflow being used.
