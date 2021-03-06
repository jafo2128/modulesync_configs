ModuleSync Configs
==================

This repository contains my
[modulesync](http://github.com/puppetlabs/modulesync) configs. It is forked
from [puppetlabs/modulesync_configs](https://github.com/puppetlabs/modulesync_configs).

A full description of ModuleSync can be found in [ModuleSync's
README](https://github.com/puppetlabs/modulesync). This README describes how
the templates are rendered in the Puppet Labs configuration.

Quickly Running
----------------

    gem install modulesync
    # symlink your module clones into modules/
    msync update --noop -f MODULE_NAME

Configuring ModuleSync
----------------------

**`modulesync.yml`**

A key-value store of arguments to pass to ModuleSync. Each key is the name of a
flag argument to the msync command. For example, `namespace: myusername`
represents passing `--namespace myusername` to msync. This file does not appear
in this repository because it only serves to override default configuration. To
override the default configuration, the file may look something like this:

```
---
namespace: yourusername
branch: yourbranch
```

**`managed_modules.yml`**

A YAML array containing the names of the modules to manage.

Defining Module Files
---------------------

**`config_defaults.yml`**

Each first-level key in this file is the name of a file in a module to manage.
These files only appear here if there are templates in the moduleroot/
directory that need to be rendered with some default values that might be
overridden. The files listed do not necessarily represent all the files that
will be managed. The files in moduleroot/ represent all the files that will be
managed, except for unmanaged and deleted files (see [#Special Options]).

**`.sync.yml`**

This file should appear in the module itself if there are any values to
override from the config_defaults.yml file or if there are any additional
values to assign. A description of what optional values can be defined in
.sync.yml follows in the description of each file in moduleroot/. .sync.yml
will have the same format as config_defaults.yml.

#### Note

Each template is rendered in slightly different ways. Your templates to not
need to be identical to these, as long as your config_defaults.yml or .sync.yml
files contain as first-level keys the exact names of the files you are
managing and appropriately handle the data structures you use in your templates
(arrays versus hashes versus single values).

**`moduleroot/CONTRIBUTING.md`**

Flat file, documentation on how to contribute.

**`moduleroot/Gemfile`**

The Gemfile contains a list of gems, optionally with versions, to install in
the development and test groups. config_defaults.yml contains a list of
"required" gems to install, in the form of an array where each element contains
the names and versions of the gems. This section of config_defaults.yml might
look like

```
Gemfile:
  required:
  - gem: rake
    version: '~>1.2'
  - gem: rspec-puppet
#...
```

The template also looks in .sync.yml for a group of optional gems to install,
and merges this list with the list found in config_defaults.yml. This section
of .sync.yml will look the same as the section of config_defaults.yml, but the
name will be "optional" rather than "required". See the Gemfile template for
the list of supported selectors.

**`moduleroot/Rakefile`**

The Rakefile gets most of its tasks from the puppetlabs_spec_helper. The
variables in the template represent lint checks to disable. config_defaults.yml
contains an array of checks to pass in to PuppetLint.configuration.send. The
key for this array is called default_disabled_lint_checks. .sync.yml may
contain an additional array of checks to disable, with the key
extra_disabled_lint_checks.

**`moduleroot/.gitignore`**

Contains some standard files to ignore. You can pass in additional files as an
array with the key "paths" in your .gitignore section in .sync.yml.

**`moduleroot/.gitattributes`**

Contains some standard attributes (EOL style). You can pass in additional attributes as an
array with the key "paths" in your .gitattributes section in .sync.yml.

**`moduleroot/.travis.yml`**

The TravisCI file is itself a YAML file with values defined in the YAML files
config_defaults.yml and .sync.yml. You can pass a custom one-line script to the
script parameter. The file gets a set of default test environments from
config_defaults.yml under the includes key. The .travis.yml section in
config_defaults.yml might look like:

```
.travis.yml:
  script: "\"bundle exec rake validate && bundle exec rake lint && bundle exec rake spec SPEC_OPTS='--format documentation'\""
  includes:
  - rvm: 1.8.7
    env: PUPPET_GEM_VERSION="~> 3.0"
  - rvm: 1.9.3
    env: PUPPET_GEM_VERSION="~> 3.0"
  - rvm: 2.0.0
    env: PUPPET_GEM_VERSION="~> 3.0"
```

You can add additional environments for a specific module to test by adding an
extras: section with the same format to the module's .sync.yml.

By setting the `docker_sets`, the .travis.yml file sets up full-system tests running in a docker container. `docker_defaults` allows you to specify or replace global defaults. Use the `options` entry on a docker set, to override individual options.

You can add environment variable settings by adding an `env:` section to the
module's `.sync.yml`.  The `env:` section may contain a `global:` section
containing environment variables to be added to all builds, and/or a `matrix:`
section, where each entry creates a new matrix entry to test.  For information,
see https://docs.travis-ci.com/user/environment-variables/#Global-Variables .
```
.travis.yml:
  env:
    global:
    # Reduce load on Travis servers to prevent tests from being killed
    - PARALLEL_TEST_PROCESSORS=16
```

**`moduleroot/.rspec`**

Flat file containing recommended default rspec options.

**`moduleroot/spec/spec_helper.rb`**

Flat file that simply requires the module_spec_helper from the
puppetlabs_spec_helper.

**`moduleroot/spec/acceptance/nodesets/*`**

Flat files containing default nodesets to run beaker-rspec on.

Special Options
---------------

### Unmanaged Files

A file can be marked "unmanaged" in .sync.yml, in which case modulesync will
not try to modify it. This is useful if, for example, the module has special
Rake tasks in the Rakefile which is difficult to manage through a template.

To mark a file "unmanaged", list it in .sync.yml with the value `unmanaged:
true`. For example,

```
---
spec/spec_helper.rb:
  unmanaged: true
```

### Deleted Files

Managing files may mean removing files. You can ensure a file is absent by
marking it "delete". This is useful for purging nodesets.

To mark a file deleted, list it in .sync.yml with the value `delete: true`. For
example,

```
---
spec/acceptance/nodesets/sles-11sp1-x64.yml
  delete: true
```

# Adding a new module

To add a new module, append it to the `managed_modules.yml` and `config_defaults.yml` files, and create a PR for the change here. To avoid any surprises, your module should already be up-to-date with the current templates. To do so, run `[bundle exec] msync update --noop -f <module name>` and commit and push the changes in `modules/<module name>` to your module. If you need to make local changes to this, add a `.sync.yml` as described above and rerun `msync` with `--offline` added. This will avoid blowing away the current checked out state. You can repeat this process as often as you need to get the module in an acceptable state.
