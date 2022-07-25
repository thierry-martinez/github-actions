---
theme: gaia
size: 4:3
---

# Continuous integration with [GitHub Actions], [Gitlab pipelines] and ci.inria.fr

[GitHub Actions]: https://docs.github.com/en/actions
[Gitlab pipelines]: https://docs.gitlab.com/ee/ci/pipelines/

Thierry Martinez (SED)
Willow software developper meetup
27 July 2022

---

# Tentative for a map of continuous integration platforms

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

## Gitlab Pipelines

(introduced in [Gitlab 8.8.0, on 2016-05-22](https://gitlab.com/gitlab-org/gitlab-foss/-/blob/master/changelogs/archive.md#880-2016-05-22))

- Continuous integration system integrated in Gitlab. gitlab.inria.fr
  provides shared docker runners, ci.inria.fr can host user-maintained VMs,
  and self-hosted gitlab runners can be registered.

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
