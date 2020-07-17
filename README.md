# sdc-kubernetes: Sysdig Platform deployment on Kubernetes

[Sysdig](https://sysdig.com/) is the the first unified approach to container security, monitoring and forensics.

This project contains the tools you need to deploy the on premise version of Sysdig platform 
to your Kubernetes infrastructure.

## Deploy using Sysdig Installer binary

Starting with 3.5.0 this repository hosts the releases for the Sysdig Installer binary.  
Please refer to [official docs](https://docs.sysdig.com/en/on-premises-installation.html) for usage guidance.

## Releases

* [Release Bundles](https://github.com/draios/sysdigcloud-kubernetes/releases)
* [Release Notes](https://docs.sysdig.com/en/sysdig-on-premises-release-notes.html)

## Deploy using YAML files

### Deprecation Note

Please note version 3.5.0 and greater, install using this repository YAML files is not supported. Please refer to installer documentation for installation [official docs](https://docs.sysdig.com/en/on-premises-installation.html).

### Infrastructure Overview <a id="Infrastructure-Overview"></a>

[Architecture Overview](https://docs.sysdig.com/en/architecture.html)

### Installation Guide <a id="installation-guide"></a>

[Installation Guide](https://docs.sysdig.com/en/on-premises-installation.html)

### Upgrading to a new version

[Upgrade Notes](https://docs.sysdig.com/en/on-premises-upgrades.html)

#### Upgrading/Migrating Datastores 

PLEASE NOTE: At this time HA upgrades are a beta feature.

Supported [migrations](https://github.com/draios/sysdigcloud-kubernetes/tree/master/migrations) are in a set of directories, one for each datastore.
