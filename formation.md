# [Github Actions]

[GitHub Actions]: https://docs.github.com/en/actions

# Introduction

## General concepts

[GitHub Actions] is a [continuous integration] (CI) platform for GitHub-hosted
projects, [launched on 16 October 2018].

[continuous integration]: https://en.wikipedia.org/wiki/Continuous_integration
[launched on 16 October 2018]: https://github.blog/2018-10-16-future-of-software/

### About [continuous integration]

[Continuous integration] (CI) is the practice of short-lived
development cycles, automatically tested and shared regularly between
developers involved in a project.

Continuous integration platforms, such as [Github Actions] (among many
others; for instance, at Inria, we have ci.inria.fr) have been developed to help
automating testing.

Automating testing (and CI in general) relies on [version control] and
automated builds. Together, these technics speed up development
process, ease collaboration and allow programmers to be more confident
for not introducing regression and bugs.

[version control]: https://en.wikipedia.org/wiki/Version_control

This is a step towards broader goals such as [reproducible
builds] and [reproducible research].

[reproducible builds]: https://en.wikipedia.org/wiki/Reproducible_builds
[reproducible research]: https://en.wikipedia.org/wiki/Reproducibility

Once the infrastructure is here for automating build and tests, the
same infrastructure can be used for automating release process
([continuous delivery], CD) and deployment ([continuous deployment],
also abbreviated CD: the nuance is that continuous delivery
automatically ensures that a release is always ready to be deployed,
and continuous deployment automatically deploys the release). The
whole approach is abbreviated [CI/CD].

[continuous delivery]: https://en.wikipedia.org/wiki/Continuous_delivery
[continuous deployment]: https://en.wikipedia.org/wiki/Continuous_deployment
[CI/CD]: https://en.wikipedia.org/wiki/CI/CD

### About [version control]

Version control systems are software dedicated for managing history
and collaborative edition of source code or any other kind of
documents. The prominent software for version control is now [git],
initially developed in 2005 by Linus Torvalds to manage the Linux
source code.

[git]: https://en.wikipedia.org/wiki/Git

Even if [git] was initially thought, and can still be used as a
decentralized tool (where versions are directly exchanged between
peers), most uses of it now rely on [software forges], like [GitHub] or
gitlab.inria.fr for instance, which provide a central place, against
which the programmers synchronize their code.  Software forges provide
other services related to version control, such as [CI/CD] facilities.

[software forges]: https://en.wikipedia.org/wiki/Forge_(software)

Keeping the history of a code is central to make change in the code
without losing information and to identify where regressions have been
introduced (in particular, automated tests enable automatic
[bisection]). Moreover, version control systems like [git] allow code
to be modified concurrently by offering merging facilities
([three-way merge]).

[bisection]: https://en.wikipedia.org/wiki/Bisection_(software_engineering)
[three-way merge]: https://en.wikipedia.org/wiki/Merge_(version_control)#Three-way_merge

## GitHub actions platform

machines fournies/self-hosted runners, dépendances entre jobs (graphe), status (commit, PR), artéfacts

# Docker and registry

## About [docker]

[docker]: https://en.wikipedia.org/wiki/Docker_(software)

## [Running jobs in a container]

[Running jobs in a container]: https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container

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

## Use a job to build the environment

- Store a personal access token with scope `write:packages` in a
  secret.
  
- Check out the repository! Add the following action

```
      - name: Checkout
        uses: actions/checkout@v3
```

- Steps for `docker build` and `docker push`.

> :warning: `checkout` action wipes out the current directory!
> Should be run before any actions writing useful things in it (local setup, etc.).

# Continuous Delivery/Continuous Deployment

## Continuous Delivery: Preparing new releases

- [Storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts): `actions/upload-artifact@v3` with inputs `name` and `path`, `actions/download-artifact@v3` with input `name`.

- [softprops/action-gh-release](https://github.com/softprops/action-gh-release) with input `files`

## Continuous Deployment: Pushing on pypi

- [pypa/gh-action-pypi-publish](https://github.com/pypa/gh-action-pypi-publish)


## Publishing documentation

- [JamesIves/github-pages-deploy-action](https://github.com/JamesIves/github-pages-deploy-action)

```
      - name: Deploy
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          folder: build
```
