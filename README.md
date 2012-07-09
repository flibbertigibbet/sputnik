Description
===========

This is a base cookbook for applying sputnik developer profiles and
repositiories onto a Ubuntu 12.04 system.

Includes [a very simple recipe](https://github.com/hh-cookbooks/sputnik/blob/master/recipes/default.rb) that doesn't use any external attributes, yaml, json, etc to demonstrate simple usage of the [sputnik::default](https://github.com/hh-cookbooks/sputnik/blob/master/recipes/default.rb) recipe.

If you want to go the yaml route, check out the [sputnik::from_yaml_example](https://github.com/hh-cookbooks/sputnik/blob/master/recipes/from_yaml_example.rb) recipe, which will copy [all these files](https://github.com/hh-cookbooks/sputnik/tree/master/files/default/sputik_example_profiles) to /etc/sputnik and includes the [sputnik::from_yaml](https://github.com/hh-cookbooks/sputnik/blob/master/recipes/from_yaml.rb) recipe which processes /etc/sputnik/*yml (files can be copied in manually instead)

The result is a [local repo](https://github.com/sputnik/cookbook/blob/master/recipes/repo.rb) that contains our [MetaPackage}(https://help.ubuntu.com/community/MetaPackages) like profiles, though I've also [pushed profiles](https://github.com/hh-cookbooks/sputnik/blob/master/providers/metapackage.rb#L34) to my own ppa.

Dependent PPAs and debian-seeds via the yaml/attributes could be forthcoming.

Mini Tutorial
=============


## Install git and chef-solo, clone this repo, then run a few sputnik recipes:

```
# as root:
apt-get install chef git #this works on the sputnik xps 13 images
git clone git@github.com:hh-cookbooks/sputnik.git
chef-solo chef-solo -c sputnik/config/solo.rb -o sputnik::hippiehacker
chef-solo chef-solo -c sputnik/config/solo.rb -o sputnik::from_yaml_example
# or populate /etc/sputnik with your own yml files
chef-solo chef-solo -c sputnik/config/solo.rb -o sputnik::from_yaml
```

## Install the sputnik profiles as you would any other package:

```
apt-cache search sputnik-profile
apt-get install sputnik-profile-foo-bar
apt-get autoremove sputink-profile-foo-bar
```


## Removal of packages and the repo we created:

```
apt-get autoremove sputnik-profile-*
rm /etc/apt/sources.list.d/sputnik-local.list
rm -rf /var/lib/sputnik
```

Logic
=====

The default recipe will loop through all the sputink profiles
and create metapackages that depend on the listed packages.
The metapackages are of the form sputnik-#{id}, ie sputnik-my-profile.

A local repo will be created (in /var/lib/sputnik-repo by default), and a gpg
key will be created for root to sign packages and the repository.

This local repo is added to /etc/apt/sources.list.d, making the sputnik metpackages within it available.

At this point you can use aptitude, synaptic, apt-get to install the profiles you want,
and 'apt-get autoremove sputnick-profile-name' will remove the metapackage and all it's dependencies automatically.

We may want to automatically and and remove the profiles completely within the recipe, but I'm not doing that yet.

ie: at the end of the recipe, any sputnik-* metapackages that are not enabled are removed.

https://help.ubuntu.com/community/MetaPackages


Requirements
============

Ubuntu 12.04

Attributes
==========

```
['sputnik']['repodir'] = '/var/lib/sputnik'
```

The location of the local repository.

```
['sputnik']['maintainer'] = 'Sputnik Local <sputnik@localhost>'
```

The name of the gpg signing key to create packages and sign our local repo with.

```
['sputnik']['profiles']
```

The default recipe loops through the profile under the sputnik.profiles attribute.

They can be set via the normal methods, I included two recipes to demonstrate.

The sputnik::from_yaml recipe loops through yaml files in the /etc/sputnik/profiles.

The sputnik::hippiehacker recipe sets a few like this:

```
node.set['sputnik']['profiles']['chris-virt']['packages'] = [
  'virtinst',
  'virt-manager',
  'virt-viewer',
  'virtualbox',
  'virtualbox-guest-additions-iso'
]
```

The attributes can/will be extended with support for

* An optional list of package-repositories or ppas.
* An optional list of debconf preseeded configuration responses.


Usage
=====

### sputnik::from_yaml_example automatically copies the examples yaml templates into /etc/sputnik, then processes them

```
chef-solo -o sputnik::from_yaml_example
```

### sputnik::hippiehacker is an example of a recipe that sets up the profile attributes directly

```
chef-solo -o sputnik::hippiehacker
```

### You could easily just run the default recipe and feed it your own json file attributes, or use a role etc

```
chef-solo -o sputnik -j dna/my-dna.json
```

