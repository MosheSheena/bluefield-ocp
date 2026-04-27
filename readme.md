# BlueField-OCP

## Pre-requisites

### Container image build requirements

- Podman
- qemu-user-static-binfmt (needed for building on non-aarch64 machines)
- Active Red Hat subscription and the `subscription-manager` package.

## Building the Image

1. Clone the repository including submodules:

    ```bash
    git clone --recursive https://github.com/rh-ecosystem-edge/bluefield-ocp.git
    ```

2. Obtain the OpenShift pull secret file and export it as an environment variable. You can obtain it from [Red Hat OpenShift Console](https://console.redhat.com/openshift/install/pull-secret).

    ```sh
    export PULL_SECRET=<path to pull secret file>
    ```

3. Get the RHCOS release image from the OCP release payload. In this example we use 4.21.8:

    ```bash
    export RHCOS_VERSION="4.21.8"
    export TARGET_IMAGE=$(oc adm release info --image-for rhel-coreos "quay.io/openshift-release-dev/ocp-release:${RHCOS_VERSION}-aarch64")
    ```

4. Set NVIDIA DOCA stack versions:

    ```bash
    export DOCA_VERSION="3.3.0"
    export DOCA_DISTRO="rhel9.6"
    ```

---

## Build options

There are two Containerfiles depending on how kernel modules and SoC drivers are sourced.

### Option A — Pre-compiled packages (`bluefield-ocp-precompiled.Containerfile`)

Use this when a DOCA repository containing pre-compiled kernel module
packages is available. No compiler toolchain or driver-toolkit image is
required.

```bash
export DOCA_BASEURL="<url-or-file-path-to-your-doca-repo>"

podman build --squash -f bluefield-ocp-precompiled.Containerfile \
  --authfile $PULL_SECRET \
  --build-arg RHCOS_VERSION=$RHCOS_VERSION \
  --build-arg TARGET_IMAGE=$TARGET_IMAGE \
  --build-arg D_DOCA_VERSION=$DOCA_VERSION \
  --build-arg D_DOCA_DISTRO=$DOCA_DISTRO \
  --build-arg D_DOCA_BASEURL=$DOCA_BASEURL \
  --tag "bluefield-ocp:$RHCOS_VERSION-latest" .
```

### Option B — Compile from source (`bluefield-ocp.Containerfile`)

Use this when pre-compiled kernel module packages are not available.
OFED kernel modules and SoC drivers are compiled from source inside a
driver-toolkit builder stage.

Additional variables required:

```bash
export BUILDER_IMAGE=$(oc adm release info --image-for driver-toolkit "quay.io/openshift-release-dev/ocp-release:${RHCOS_VERSION}-aarch64")
export OFED_VERSION="26.01-1.0.0.0"
```

```bash
podman build --squash -f bluefield-ocp.Containerfile \
  --authfile $PULL_SECRET \
  --build-arg RHCOS_VERSION=$RHCOS_VERSION \
  --build-arg TARGET_IMAGE=$TARGET_IMAGE \
  --build-arg BUILDER_IMAGE=$BUILDER_IMAGE \
  --build-arg D_DOCA_VERSION=$DOCA_VERSION \
  --build-arg D_OFED_VERSION=$OFED_VERSION \
  --build-arg D_DOCA_DISTRO=$DOCA_DISTRO \
  --tag "bluefield-ocp:$RHCOS_VERSION-latest" .
```

---

Optionally, you can override the DOCA repository base URL in either build by adding
`--build-arg D_DOCA_BASEURL=<custom_doca_repo_baseurl>` to the `podman build` command.
