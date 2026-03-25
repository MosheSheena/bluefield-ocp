# BlueField-OCP

### Pre-requisites
Container image build requirements:
- Podman
- subscription-manager (Will enable podman to automount the subscription entitlements)

Ensure you are logged in to the Red Hat subscription manager
```bash
sudo dnf install subscription-manager
sudo subscription-manager register --username YOUR_USERNAME
```


### Clone the project
The project uses Mellanox's bfb-build as a git submoudle, so be sure to clone it as well:
```bash
git clone --recursive https://github.com/rh-ecosystem-edge/bluefield-ocp.git
```

Make sure you export PULL_SECRET, you can obtain it from https://console.redhat.com/openshift/install/pull-secret.
```bash
export PULL_SECRET=<path to pull secret file>
```

Use `oc` to get the release image for the RHCOS version you want to build.
```bash
export RHCOS_VERSION="4.21.6"

# Get Driver Toolkit image
export BUILDER_IMAGE=$(oc adm release info --image-for driver-toolkit "quay.io/openshift-release-dev/ocp-release:"$RHCOS_VERSION"-aarch64")

# Get RHEL CoreOS OCP OS Image
export TARGET_IMAGE=$(oc adm release info --image-for rhel-coreos "quay.io/openshift-release-dev/ocp-release:"$RHCOS_VERSION"-aarch64")
```

Set Nvidia DPU stack versions:
```bash
export DOCA_VERSION="3.3.0"
export DOCA_DISTRO="rhel9.6"
export OFED_VERSION="26.01-1.0.0.0"
```

Build the container image:

```bash
podman build -f bluefield-ocp.Containerfile \
  --authfile $PULL_SECRET \
  --build-arg RHCOS_VERSION=$RHCOS_VERSION \
  --build-arg BUILDER_IMAGE=$BUILDER_IMAGE \
  --build-arg TARGET_IMAGE=$TARGET_IMAGE \
  --build-arg D_DOCA_VERSION=$DOCA_VERSION \
  --build-arg D_OFED_VERSION=$OFED_VERSION \
  --build-arg D_DOCA_DISTRO=$DOCA_DISTRO \
  --tag "bluefield-ocp:$RHCOS_VERSION-latest" .
```

Optionally, you can override the DOCA repository baseurl by adding: `-build-arg D_DOCA_BASEURL=<custom_doca_repo_baseurl>` to the above `podman build` command.
