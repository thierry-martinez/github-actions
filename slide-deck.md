---
theme: gaia
size: 4:3
---

# Continuous integration with Github Actions, Gitlab pipelines and ci.inria.fr

Thierry Martinez (SED)
Willow software developper meetup, 27 July 2022

# Tentative for a map of continuous integration platforms

## Github Actions

- A platform for github-hosted projects, providing runners for Linux,
  MacOS and Windows.

- File-based workflow specification: `.github/workflows/*.yml`.
  A command-line tool, `act` is available for running workflows
  locally (or from other continuous integration platforms):
  https://github.com/nektos/act

- Very easy to extend (new reusable actions can be defined in git
  repositories), Linux runners can run docker containers,
  user-provided runners can be used.

## Gitlab Pipelines

- Continuous integration system integrated in Gitlab. gitlab.inria.fr
  provides shared docker runners, ci.inria.fr can host user-maintained VMs,
  and self-hosted gitlab runners can be registered.

- File-based workflow specification: `.gitlab.yml`.  A command-line
  tool, `gitlab-ci-local` is available for running workflows locally
  (or from other continuous integration platforms):
  https://github.com/firecow/gitlab-ci-local

- The supported YAML syntax is richer than Github Actions (support for
  anchors and file inclusion to reuse parts of code), but there is no
  built-in supports for reusable actions comparable with what Github
  Actions proposes.

## ci.inria.fr

- A cloud of virtual machines dedicated for continuous integration:
  VMs are created and maintained by the users.  Linux, Windows and
  MacOS VMs are supported, but MacOS VMs are quite fragile for the
  moment (but we are working on it!).
  
- Provides yet another continuous integration system, Jenkins, and a
  part of it can be configured by a file-based workflow specification:
  `Jenkinsfile`, difficult to run locally (except by installing and
  maintaining a local instance of Jenkins...).
  
- Very extensible through plugins but fragile and hard to
  maintain. Most plugins should be configured via the web interface;
  there is an API covering many operations, but it is not well
  documented and there is no easy tool to call it (excepting `curl`!).
