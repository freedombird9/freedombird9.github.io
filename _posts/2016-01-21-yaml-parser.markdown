---
layout: post
title:  "Command Line YAML Parer & Auto-exporter"
description: A small command line tool to parse YAML files and auto-export environment variables.
date:   2016-01-21 17:34:44
categories: technology
---

Very often I need to run an app locally to debug. How to quickly read the environment variables in the production server and export them locally? We can just parse the server's config file. I'll use GAE (Google App Engine) as an example here. Its config file is in YAML format, which is a very common human-readable data serialization language used in many places. So the tool introduced here is applicable in many situations.

A typical YAML config file for App engine is like the following:
<div class='md-10-suffix-1'>
{% highlight yaml %}
application: project-id
version: 1
runtime: python27
api_version: 1
# other config settings
env_variables:
 DJANGO_SETTINGS_MODULE: 'base.settings'
 GAE_SQL_HOST: '/cloudsql/:dbname'
 GAE_SQL_DBNAME: 'dbname'
 LOCAL_SQL_HOST: 'host_ip_address'
 LOCAL_SQL_DBNAME: 'dbname'
 DEBUG: 'off'
 TEMPLATE_DEBUG: 'off'
 GCM_PUSH_API_KEY: 'key'
{% endhighlight %}
</div>

App engine will use the variables under `env_variables` as its environment variables. We need to parse this file and export them locally one by one.

### Usage
Get the source code [here] first. To use it, you need to have bash 4 and Python YAML module.

#### Install bash 4 on OS X

You can see your bash version by running `bash --version`. If it's not version 4, you need to install one.
<div class='md-10-suffix-1'>
{% highlight bash %}
# Homebrew installs the new bash under /usr/local/bin/bash
brew update && brew install bash
sudo bash -c 'echo /usr/local/bin/bash >> /etc/shells'
# Change to the new shell
chsh -s /usr/local/bin/bash
{% endhighlight %}
</div>

Now close your current terminal and start a new one. Run `bash --version`, you should see version 4.X.X

Install PyYAML:

The shell script relies on a python program to parse the yaml file. You need this module to run the python parser (`read_yaml.py`).

<div class='md-10-suffix-1'>
{% highlight bash %}
pip install pyyaml
{% endhighlight %}
</div>

Then you're ready to go. Remember to execute it with `. set_envs.sh /path/to/yamlfile` or `source set_envs.sh /path/to/yamlfile`

If you run it directly, the environment variables won't be exported to your current working shell, rather, they'll be exported to a sub-shell and be gone.

### How?

Fairly simple. Consume the Python-parsed YAML in the bash, store the key-value pairs in an associative array. Iterate through it and export. Note that associative arrays are only available in bash 4, and that's why we need to upgrade our bash.

<div class='md-10-suffix-1'>
{% highlight bash %}
declare -A items=()
while IFS= read -r -d '' key && IFS= read -r -d '' value; do
    items[$key]=$value
done < <(python $my_dir/read_yaml.py $1)

for i in "${!items[@]}"
do
  echo "$i: ${items[$i]}"
  export $i=${items[$i]}
done
{% endhighlight %}
</div>


[here]: https://github.com/freedombird9/work-happy-anywhere
