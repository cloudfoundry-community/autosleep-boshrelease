# BOSH Release for cloudfoundry-operators-tools

## Purpose

The purpose of this bosh release is to offer some preconfigured cloudfoundry applications components, for use in Bosh / Cloudfoundry operations.

The idea is to ease bosh ops activities, providing them usefull features, packaged in a bosh standard way (manifest, errand, etc ..)

NB: the mechanism used is to push operator level cf apps inside cloudfoundry, including sensitive credentials. Take care to target an admin only visible organization (ie: system_domain)
moreover, these operator tools wont be more available than the target cf deployment. Try cross deploying if you have multilple cf depl (ie: for cachet portal / cachet monitor) for high availability

### Orange AutoSleep Service for Cloudfoundry
The aim of the auto-sleep project is to give the ability for Cloud Foundry operators to automatically have cf apps idled, after a given period of inactivity, and then automatically restarted when end users access apps again through traffic their routes.
This features, with some CF Diego overprovisionning, enables an economic management of Cloudfoundry Development orgs.

This bosh release provides 3 errands :
* publish_autosleep : install and publish the autosleep service broker 
* publish_autowake_proxy: install and publish the autowake app, binding it to wildcard route domains
* provision_autosleep: batch creates autosleep service instances in all the spaces of a given org 
recommanded reading before use:
* https://github.com/Orange-OpenSource/cloudfoundry-operators-tools-boshrelease

prerequisite:
* a cloudfoundry autosleep account
* cf marketplace with mysql for autosleep broker persistence
* open cf application security group, so that autosleep and autowakeup cf apps can acces the cloudfoundry api



### Cachet Server and Monitor

Cachet server enables cloudfoundry operator to give health status for end users. 

This release provide 2 bosh errands, pushing cf application for :
* cachet server
* cachet monitor

recommanded reading before use:
* https://github.com/ArthurHlt/Cachet-cf
* https://github.com/CastawayLabs/cachet-monitor/

prerequisite:
* a cloudfoundry org admin account
* cf marketplace with mysql for cachet persistent
* optional redis cf marketplace service, for cachet caching
* open cf application security group, so that cachet monitor can reach cachet server




## Usage

To use this bosh release, first upload it to your bosh:

```
bosh target BOSH_HOST
git clone https://github.com/cloudfoundry-community/cloudfoundry-operators-tools-boshrelease.git
cd cloudfoundry-operators-tools-boshrelease
bosh upload release releases/cloudfoundry-operators-tools-1.yml
```

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest & deploy a cluster:

```
templates/make_manifest warden
bosh -n deploy
```

For AWS EC2, create a single VM:

```
templates/make_manifest aws-ec2
bosh -n deploy
```

### Override security groups

For AWS & Openstack, the default deployment assumes there is a `default` security group. If you wish to use a different security group(s) then you can pass in additional configuration when running `make_manifest` above.

Create a file `my-networking.yml`:

``` yaml
---
networks:
  - name: cloudfoundry-operators-tools1
    type: dynamic
    cloud_properties:
      security_groups:
        - cloudfoundry-operators-tools
```

Where `- cloudfoundry-operators-tools` means you wish to use an existing security group called `cloudfoundry-operators-tools`.

You now suffix this file path to the `make_manifest` command:

```
templates/make_manifest openstack-nova my-networking.yml
bosh -n deploy
```

### Development

As a developer of this release, create new releases and upload them:

```
bosh create release --force && bosh -n upload release
```

### Final releases

To share final releases:

```
bosh create release --final
```

By default the version number will be bumped to the next major number. You can specify alternate versions:


```
bosh create release --final --version 2.1
```

After the first release you need to contact [Dmitriy Kalinin](mailto://dkalinin@pivotal.io) to request your project is added to https://bosh.io/releases (as mentioned in README above).
