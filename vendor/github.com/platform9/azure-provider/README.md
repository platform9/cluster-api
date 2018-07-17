# k8s Cluster API Azure Provider [![Build Status](https://travis-ci.org/platform9/azure-provider.svg?branch=master)](https://travis-ci.org/platform9/azure-provider) [![Go Report Card](https://goreportcard.com/badge/github.com/platform9/azure-provider)](https://goreportcard.com/report/github.com/platform9/azure-provider)

## Usage
1. Install the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
2. Log into the Azure CLI, `az login`
3. (Optional) Modify the templates found in `configtemplates/` 
4. Run `generate-yaml.sh`   _Note: `generate-yaml.sh` creates an Azure service principal which will not be deleted automatically._
5. Obtain `clusterctl` from the [cluster-api repository](https://github.com/kubernetes-sigs/cluster-api). You can either:
    * Build from source by cloning the repo and running `go build` while in the `cluster-api/clusterctl` directory.
    * Use one of the pre-built binaries found in the releases of the repository.
6. Use the configs generated in `generatedconfigs/` with `clusterctl`
    * Example: `./clusterctl --provider azure -m generatedconfigs/machines.yaml -c generatedconfigs/cluster.yaml -p generatedconfigs/provider-components.yaml -a generatedconfigs/addons.yaml`

## Creating and using controller images
1. [Install docker](https://docs.docker.com/install/#supported-platforms) and ensure docker works with `docker run hello-world`
2. Log into docker with `docker login`
3. Edit `cmd/azure-controller/Makefile` such that `PREFIX` is the logged in user, and `NAME` is the repository you wish to push your images to.
4. While in `cmd/azure-controller/`, run `make push` to create an image and push it to your Docker Hub repository.
5. Edit `generatedconfigs/provider-components.yaml` such that the images for `azure-machine-controller` and `azure-cluster-controller` refer to the images you just pushed, e.g. `meegul/azure-controller:0.0.17-dev`


## Testing
Unit tests can be ran with `make unit_test`, and integration tests can be ran with `make integration_test`. However, keep in mind that the integration tests will take a significant amount of time (> 1 hour) and _**will create resources in Azure**_. The integration tests should clean up the created resources, but do not take this as a guarantee.
### Additional notes for testing
cluster-api should be vendored when testing, either in Travis or locally, but should not be versioned in git. This allows the cluster-api to import `azure-provider` while avoiding a circular dependency. To vendor the cluster-api for testing purposes, un-ignore it in `Gopkg.toml` and run `dep ensure -add sigs.k8s.io/cluster-api/pkg`. After adding it, `Gopkg.lock` and `Gopkg.toml` will reference it. Prior to comitting, these references should be manually removed and the ignore restored.