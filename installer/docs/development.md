## Development requirements

- [Docker](https://docs.docker.com/install/)
- [Make](https://www.gnu.org/software/make/), this is optional you can read
the [Makefile](Makefile) and run the commands in the target by hand instead.

## Building the image

```bash
$> make build
```

This will build and tag a installer docker image, to override the image/tag
name, set the environment variable `IMAGE_NAME`, e.g:

```bash
IMAGE_NAME=my_awesome_image:my_awesome_tag make build
```

## Running the built image

```bash
$> make run
```

This will build and tag a installer docker image, mount your kubeconfig as
from `~/.kube` your current working directory in `/manifests` and run the
installer docker image, e.g:

```bash
IMAGE_NAME=my_awesome_image:my_awesome_tag make run
```

## Testing

```bash
$> make test
```

## Development workflow

- Make your changes
- If you are introducing a new configuration type that will generate extra
yaml code, to add a test case do the below:
  - copy one of the directories in
  [sysdig-chart/tests/resources](sysdig-chart/tests/resources)
  - modify the values.yaml
  - Run `make config_gen`, this should produce a new `sysdig.json` file,
  commit this along with your changes.
- Run tests `make test`
- If there are diff failures, run `make config_gen`

The rationale for the workflow is it helps the code reviewers to see the
effect(s) of the changes introduced, it also helps exercise the config
generation workflow.

## Releasing

### Versioning

The installer versioning scheme is as below:

<sysdig_platform_version>-<monotonous integer>

The integer starts at 1 and is bumped for every release of the installer,
then reset to 1 for a release of sysdig_platform_version. E.g:

For the first release of installer the release of sysdig platform is
`2.4.0`, hence the installer version is `2.4.0-1` next release of
installer for this sysdig platform is `2.4.0-2`, if a new sysdig platform
is released tomorrow as `2.4.1` a new release of installer will be cut at
`2.4.1-1`.

For uber images containing a tarball of all images for [airgap
installations](#full-airgap-installation) the versioning scheme is:

<installer_version>-uber

For release candidates the version is:

<next_installer_version>-rc-<JENKINS_BUILD_NUMBER>

### Release Candidates

On every successful build of the main branch, Jenkins tags a release candidate
version matching `<next_release_tag>-rc<$JENKINS_BUILD_NUMBER>` and pushes
the git tag, and a docker image matching the git tag.

### Full Release

The workflow for doing a full (non-rc) release is as below:

- Change `next_version` to the new tag to be released, for
example if the new tag is `2.4.0-2`, do as below:
```bash
echo -n 2.4.0-2 > next_version
```
- Commit the change, e.g: `git commit -am 'Bumping version to 2.4.0-2'`
- Push the change: `git push`
- Submit a PR and get the PR merged
- Wait for Jenkins to complete build of the main branch including your last
commit.
- Request that no one merges to the main branch till you are done releasing.
- Once Jenkins is done building and has pushed the docker image and git tag,
checkout to the last release candidate tag built by Jenkins, e.g:
```bash
git checkout 2.4.0-2-rc${REPLACE_WITH_JENKINS_BUILD_NUMBER}
```
- Read the diff of changes from the last release
```bash
git diff $(cat current_version)..
```
- Create a new release tag, e.g:
```bash
git tag -F <(git log --oneline $(cat current_version)..) $(cat next_version)
git push origin refs/tags/"$(cat next_version)"
```
- Wait for the Jenkins build for the new tag to complete successfully.
- Update every part of the README.md(this file) that indicates a version to
indicate the latest release tag.
- Update the internal wiki instructions pointing at the latest release.
- Update `current_version` to reflect the new tag, e.g:
```
cat next_version > current_version
```
- Increment the number after the hypen e.g `1` in `2.4.0-2` by `1` and update
the `next_version` file, this is hypothetically the next future
version, e.g:
```bash
echo -n 2.4.0-3 > next_version
```
- Commit the changes
```bash
git commit -am "Tagged 2.4.0-2"
```
- Push the change and submit a PR.
