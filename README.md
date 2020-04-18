gitlab-templates
================

Set of reusable templates for gitlab-ci.

docker-dind-custom-ca.yml
-------------------------

When deploying gitlab-ci via helm charts using wildcard certificate ([Option 2: Use your own wildcard certificate](https://docs.gitlab.com/charts/installation/tls.html#option-2-use-your-own-wildcard-certificate)), the following error may occur when trying to build a docker image with AutoDevops / build stage:

```
 $ if [[ -z "$CI_COMMIT_TAG" ]]; then # collapsed multi-line command
 $ /build/build.sh
 Logging to GitLab Container Registry with CI credentials...
 WARNING! Using --password via the CLI is insecure. Use --password-stdin.
 Error response from daemon: Get https://registry.example.com/v2/: x509: certificate signed by unknown authority
```

The root cause was that [AutoDevops/Build](https://gitlab.com/gitlab-org/gitlab-foss/-/blob/master/lib/gitlab/ci/templates/Jobs/Build.gitlab-ci.yml) was not adding your private CA certificate to the /etc/ssl/certs directory so when the docker-dind tries to access your registry (with your private crt) it is unable to verify its authenticity.

This issue has been reported in [gitlab ticket 1350](https://gitlab.com/gitlab-org/gitlab-runner/issues/1350) and the solution is based on [this comment there](https://gitlab.com/gitlab-org/gitlab-runner/issues/1350#note_120086329) which adds the custom CA to the /etc/ssl/certs/custom-ca.crt before starting docker-dind.

### Usage

Option 1: Including the file from this repository. It would work in from a different public location or a fork of this repo.

```
include:
  - template: Auto-DevOps.gitlab-ci.yml
  - remote: https://raw.githubusercontent.com/joaocenoura/gitlab-templates/master/docker-dind-custom-ca.yml
```

Option 2: Copy/paste the contents of the file:

```
include:
  - template: Auto-DevOps.gitlab-ci.yml

build:
  services:
    - name: docker:19.03.5-dind
      command:
        - /bin/sh
        - -c
        - |
          echo "$CI_SERVER_TLS_CA_FILE" > /etc/ssl/certs/custom-ca.crt || exit
          dockerd-entrypoint.sh || exit
```
