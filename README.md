Description
===========

Installs and configures the Cloudkick Agent, and integrates it with Chef.

Requirements
============

Requires Chef 0.10.10+

* `chef_gem` in default recipe
* `default_action` in check and monitor LWRPs

Requires Ohai 0.6.12+ for `node['platform_family']`.

Platform
--------

Platform families Debian, RHEL and Fedora are supported in the general
sense, though not all may have appropriate Cloudkick packages
available upstream.

Cookbooks
---------

* apt
* yum

The `apt_repository` and `yum_repository` LWRPs are used from these
cookbooks to create the proper repository entries so the Cloudkick
agent can be downloaded and installed.

Attributes
==========

The `/etc/cloudkick.conf` is built using these attribute values:

* `node['cloudkick']['oauth_key']` - the OAuth key used for
  authentication with the Cloudkick API.
* `node['cloudkick']['oauth_secret']` - the OAuth secret used for
  authentication with the Cloudkick API.
* `node['cloudkick']['node_name']` - host name for the Cloudkick dashboard,
  taken from ohai node.name by default.
* `node['cloudkick']['additional_tags']` - additional Cloudkick tags,
  added to the tags taken automatically from Role names.
* `node['cloudkick']['local_plugins_path']` - path to a directory containing custom agent plugins,
  defaults to `/usr/lib/cloudkick-agent/plugins`, the same as Cloudkick.
* `node['cloudkick']['data']` - data for the Cloudkick API about the
  node.

Resources & Providers
=====================

This cookbook contains two Resource Providers:

* `cloudkick_check` - For creating & manipulating Cloudkick Checks.
* `cloudkick_monitor` - For creating & manipulating Cloudkick Monitors.

## Example Usage:

* Create a Cloudkick Monitor for all of our appservers:

    cloudkick_monitor "appserver monitor" do
      oauth_key 'xxx'
      oauth_secret 'yyy'
      query 'tag:appserver'
      action :create
    end

* Create a Cloudkick Check for the root partition on our appservers:

    cloudkick_check "root check" do
      oauth_key 'xxx'
      oauth_secret 'yyy'
      code 51 # 'DISK'
      details({:path => '/', :fs_critical => 99, :fs_warn => 95})
      monitor_id 'q1234'
      action :create
    end

Usage
=====

In order for the agent to function, you'll need to have defined your Cloudkick API key and secret.  We recommend you do this in a Role, which should also take care of applying the cloudkick::default recipe.

Assuming you name the role 'cloudkick', here is the required json:

    {
      "name": "cloudkick",
      "chef_type": "role",
      "json_class": "Chef::Role",
      "default_attributes": {
        "cloudkick": {
          "additional_tags": [ "agent" ],      # Optional.
          "node_name": "YOUR UNIQUE HOSTNAME"  # Optional.
        }
      },
      "description": "Configures Cloudkick",
      "run_list": [
        "recipe[cloudkick]"
      ],
      "override_attributes": {
        "cloudkick": {
          "oauth_key": "YOUR KEY HERE"
          "oauth_secret": "YOUR SECRET HERE"
        }
      }
    }

If you want Cloudkick installed everywhere, we recommend you just add the cloudkick attributes to a base role.

All of the data about the node from Cloudkick is available in node['cloudkick'] - for example:

    "cloudkick": {
      "oauth_key": "YOUR KEY HERE",
      "oauth_secret": "YOUR SECRET HERE",
      "data": {
        "name": "slice204393",
        "status": "running",
        "ipaddress": "173.203.83.199",
        "provider_id": "padc2665",
        "tags": [
          "agent",
          "cloudkick"
        ],
        "agent_state": "connected",
        "id": "n87cfc79c5",
        "provider_name": "Rackspace",
        "color": "#fffffff"
      }
    }

Of particular interest is the inclusion of the Cloudkick tags.  This will allow you to search Chef via tags placed on nodes within Cloudkick:

    $ knife search node 'cloudkick_data_tags:agent' -a fqdn
    {
      "rows": [
        {
          "fqdn": "slice204393",
          "id": "slice204393"
        }
      ],
      "start": 0,
      "total": 1
    }

We automatically add a tag for each Role applied to your node.  For example, if your node had a run list of:

    "run_list": [ "role[webserver]", "role[database_master]" ]

The node will automatically have the 'webserver' and 'database_master' tags within Cloudkick.

License and Author
==================

Author:: Adam Jacob (<adam@opscode.com>)
Author:: Seth Chisamore (<schisamo@opscode.com>)
Copyright:: 2010-2011, Opscode, Inc

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
