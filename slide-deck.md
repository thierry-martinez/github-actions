---
size: 4:3
---

<style>
{
  font-size: 16pt;
}
</style>

# Continuous integration with [GitHub Actions], [GitLab pipelines] and ci.inria.fr

[GitHub Actions]: https://docs.github.com/en/actions
[GitLab pipelines]: https://docs.gitlab.com/ee/ci/pipelines/

Thierry Martinez (SED)
Willow software developper meetup
27 July 2022

---

# Tentative for a cartography of continuous integration platforms

---

## GitHub Actions

([launched on 16 October 2018](https://github.blog/2018-10-16-future-of-software/))

- A platform for GitHub projects, providing [GitHub-hosted runners]
  for Linux, MacOS and Windows.

  [GitHub-hosted runners]: https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources

- File-based workflow specification: `.github/workflows/*.yml`.
  A command-line tool, `act` is available for running workflows
  locally (or from other continuous integration platforms):
  https://github.com/nektos/act

- Very easy to extend (new [reusable actions] can be defined in git
  repositories), Linux runners can run docker containers,
  user-provided runners can be used.

[reusable actions]: https://docs.github.com/en/actions/creating-actions/creating-a-composite-action

---

## GitLab Pipelines

(introduced in [GitLab 8.8.0, on 22 May 2016](https://gitlab.com/gitlab-org/gitlab-foss/-/blob/master/changelogs/archive.md#880-2016-05-22))

- Continuous integration system integrated in GitLab. gitlab.inria.fr
  provides shared docker runners, ci.inria.fr can host user-maintained VMs,
  and self-hosted GitLab runners can be registered.

- File-based workflow specification: [`.gitlab.yml`].  A command-line
  tool, `gitlab-ci-local` is available for running workflows locally
  (or from other continuous integration platforms):
  https://github.com/firecow/gitlab-ci-local

  [`.gitlab.yml`]: https://docs.gitlab.com/ee/ci/yaml/

- The supported YAML syntax is richer than GitHub Actions (support for
  [anchors], [extends] and [file inclusion] to reuse parts of code), but
  there is no built-in supports for reusable actions comparable with
  what GitHub Actions proposes.

  [anchors]: https://docs.gitlab.com/ee/ci/yaml/yaml_optimization.html#anchors
  [extends]: https://docs.gitlab.com/ee/ci/yaml/yaml_optimization.html#use-extends-to-reuse-configuration-sections
  [file inclusion]: https://docs.gitlab.com/ee/ci/yaml/includes.html

---

## ci.inria.fr

- A cloud of virtual machines dedicated for continuous integration:
  VMs are created and maintained by the users.  Linux, Windows and
  MacOS VMs are supported, but MacOS VMs are quite fragile for the
  moment (but we are working on it!).

- Provides yet another continuous integration system, Jenkins, and a
  part of it can be configured by a file-based workflow specification:
  [`Jenkinsfile`], difficult to run locally (except by installing and
  maintaining a local instance of Jenkins...).

  [`Jenkinsfile`]: https://www.jenkins.io/doc/book/pipeline/jenkinsfile/

- Very extensible through plugins but fragile and hard to
  maintain. Most plugins should be configured via the web interface;
  there is an API covering many operations, but it is not well
  documented and there is no easy tool to call it (excepting `curl`!).

---

# A primer on GitHub Action

---

## Repository initialization

- We will use GitHub command line: https://cli.github.com/

- GitHub Actions run every workflow specified in files
  `.github/workflows/*.yml` in a GitHub repository.

- `gh repo create github-actions-primer --public --clone`

- Put some contents in `github-actions-primer/.github/workflows/main.yml`
```yaml
name: main
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Preparing the environment
        run: |
          <enter some shell commands>
```

See [Choosing GitHub hosted runners] for a list of available platforms
for `runs-on` entry. Note that `ubuntu-latest` is currently Ubuntu 20.04.
There is `ubuntu-22.04` available in beta.

[Choosing GitHub hosted runners]: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#choosing-github-hosted-runners

- Use `gh run list` to check the status of workflow runs on the command-line.

---

## Run workflow locally

```bash
$ act
```

For platforms not supported out-of-box, one can provide a Docker
image: for instance, to support `ubuntu-22.04`
```bash
act -P ubuntu-22.04=local-ubuntu-22.04
```

where `local-ubuntu-22.04` is a tag for an image built with a `Dockerfile`
such as
```Dockerfile
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get upgrade --yes
RUN apt-get install --yes sudo curl psmisc
```

> :warning: Using versions of Ubuntu ⩾ 21.10 in Docker images requires
> Docker ⩾ 20.10.9 ([issue with syscall `clone3`]).

[issue with syscall `clone3`]: https://pascalroeleven.nl/2021/09/09/ubuntu-21-10-and-fedora-35-in-docker/

> :warning: GitHub-hosted runners reduce interactions much more than
> `act` knows to do locally: think about adding options `--yes` and
> passing `DEBIAN_FRONTEND=noninteractive` in `apt-get` environment...

---

## [Running jobs in a container](https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container)

Build environments can be prepared once for all in a Docker image to
reduce build times:

- `docker build -t ghcr.io/‹user›/‹image name› .`

- [create a personal access token] with scope `write:packages`,
  save it in a file

  [create a personal access token]: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token

- `docker login ghcr.io -u ‹user› --password-stdin < ‹token path›`

- `docker push ghcr.io/‹user›/‹image name›`

- create a personal access token with scope `read:packages`,
  store it in a [secret] (using `gh secret set`)

  [secret]: https://docs.github.com/en/actions/security-guides/encrypted-secrets

- reference the container in the job
```yaml
    container:
      image: ghcr.io/‹user›/‹image name›
      credentials:
        username: ${{ github.actor }}
        password: ${{ secrets.‹secret name› }}
```

- to run the workflow locally, use `act --secret-file ‹file name›`

---

## Use a job to build the environment

- Store a personal access token] with scope `write:packages` in a
  secret.
  
- Check out the repository! Add the following action

```
      - name: Checkout
        uses: actions/checkout@v3
```

- Steps for `docker build` and `docker push`.

> :warning: `checkout` action wipes out the current directory!
> Should be run before any actions writing useful things in it (local setup, etc.).

---

## Run a job only if a file has changed

- Checkout with the input `fetch-depth: 2` to get the two last commits (by
  default, only the last commit is checked out, _i.e._ `git fetch --depth=1`)

- Use `git diff --quiet --exit-code HEAD^ HEAD -- ‹path›` to check if
  a file changed.

> :warning: Commands should succeed (with return code 0). Use `if-then-else-fi`
> to control the result of `git diff`.

- Can be done in another job, using [job outputs] and [conditions].

  [job outputs]: https://docs.github.com/en/actions/using-jobs/defining-outputs-for-jobs
  
  [conditions]: https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution

- We only want to build the image if `Dockerfile` has changed, but the
  main job should be run even if the build job has been skipped: use
  [`always()`] and check [`needs.‹job_id›.result`] for `success` or `skipped`.

  [`always()`]: https://docs.github.com/en/actions/learn-github-actions/expressions#always
  
  [`needs.‹job_id›.result`]: https://docs.github.com/en/actions/learn-github-actions/contexts#needs-context

---

## Using artifacts and deploy release

- [Storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts): `actions/upload-artifact@v3` with inputs `name` and `path`, `actions/download-artifact@v3` with input `name`.

- [softprops/action-gh-release](https://github.com/softprops/action-gh-release) with input `files`

---

## Some notes on Windows runner

- `windows-latest` runners come with `choco` and [some pre-installed tools], such as `7z`.

  [some pre-installed tools]: https://github.com/actions/virtual-environments/blob/main/images/win/Windows2019-Readme.md#tools
  
---

## [Adding self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/adding-self-hosted-runners)

- In Project Settings › Actions › Runners, button _New self-hosted runner_.
  Follow the instructions. Tags match the values of `runs-on:` field.

- `./run.sh` can be run in `tmux` or as a service.

---

## Matrix job

- Use [`strategy.matrix`] to build the same job with different
  combinations of parameters.
  
  [`strategy.matrix`]: https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs

- Set [`strategy.fail-fast: false`] to continue the build of other
  combinations when a combination failed.
  
  [`strategy.fail-fast: false`]: https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs#handling-failures
