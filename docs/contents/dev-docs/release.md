# Release

Here we document the release process for ACK service controllers.

Remember that there is no single ACK binary. Rather, when we build a release
for ACK, that release is for one or more individual ACK service controllers
binaries, each of which are installed separately.

This documentation covers the steps involved in building a service controller's
release artifacts, including the Helm charts and binary Docker images for an
ACK service controller, publishing those artifacts and tagging the Git source
repository appropriately to create an official "release".

## What is a release exactly?

A "release" is the combination of a Git tag containing a SemVer version tag
against this source repository and the collection of *artifacts* that allow the
individual ACK service controllers included in that Git commit to be easily
installed via Helm.

The Git tag points at a specific Git commit referencing the exact source code
that comprises the ACK service controllers in that "release".

The release artifacts include the following for one or more service
controllers:

* Docker image
* Helm chart

The Docker image is built and pushed with an image tag that indicates the
release version for the controller along with the AWS service. For example,
assume a release semver tag of `v0.1.0` that includes service controllers for
S3 and SNS. There would be two Docker images built for this release, one each
containing the ACK service controllers for S3 and SNS. The Docker images would
have the following image tags: `s3-controller-v0.1.0` and
`sns-controller-v0.1.0`. Note that the full image name would be
`amazon/aws-controllers-k8s:s3-v0.1.0`

The Helm chart artifact can be used to install the ACK service controller as a
Kubernetes Deployment; the Deployment's Pod image will refer to the exact
Docker image tag matching the release tag.

## Release steps

1. First check out a git branch for your release:
 
```bash
export RELEASE_VERSION=v0.0.1
git checkout -b release-$RELEASE_VERSION
 ```

2. Build the release artifacts for the controllers you wish to include in the
   release

   Run `scripts/build-controller-release.sh` for each service. For
   instance, to build release artifacts for the SNS and S3 controllers I would
   do:

```bash
for SERVICE in sns s3;\
    do ./scripts/build-controller-release.sh $SERVICE $RELEASE_VERSION;
done
```

3. You can review the release artifacts that were built for each service by
   looking in the `services/$SERVICE/helm` directory:

    `tree services/$SERVICE/helm`

    or by doing:

    `git diff`

!!! note
    When you run `scripts/build-controller-release.sh` for a service, it will
    overwrite any Helm chart files that had previously been generated in the
    `services/$SERVICE/helm` directory with files that refer to the
    Docker image with an image tag referring to the release you've just built
    artifacts for.

4. Commit your code and create a pull request:

```bash
git commit -a -m "release artifacts for release $RELEASE_VERSION"
```

5. Get your pull request reviewed and merged.

6. Upon merging the pull request, a Github Action should trigger that does the
   following:

```bash
git tag -a $RELEASE_VERSION $( git rev-parse HEAD )
git push upstream main --tags
```

which will end up associating a Git tag (and therefore a Github release) with
the SHA1 commit ID of the source code for the controllers and the release
artifacts you built for that release version.

The same Github Action should run the `scripts/publish-controller-images.sh`
script to build the Docker images for the service controllers included in the
release and push the images to the `amazon/aws-controllers-k8s` image
repository.
