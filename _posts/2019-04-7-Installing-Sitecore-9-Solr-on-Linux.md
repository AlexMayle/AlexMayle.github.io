---
layout: post
title: "How to Create and Configure the Sitecore 9 Solr Collections on Linux"
tags: [Sitecore, "Sitecore 9", "Solr", "Collections", "Linux"]
description: "SIF provides no way to create and configure the necessary SolrCloud collections on Linux. It just assumes you are going to run Solr on Windows, let's take that assumption away. I present a one-liner for all your Sitecore 9 solr config." 
---

TL;DR: Clone the [repo](https://github.com/AlexMayle/sitecore9-core-installer) containing the script, execute the following from the root of the repo.
``` python
python3 create_default_indexes.py | exec
```

So apparently running Solr on Windows is just a given when it comes to Sitecore. However, when it comes to operating in the cloud, or anywhere for that matter, you're going to spend more on Windows servers. As someone who works on both linux and Windows systems, I only use Windows if there is a good reason to - and there really is none in this case.

Here is a tool which you can use for the most basic to advanced scenarios to create and configure the Sitecore 9 SolrCloud collections on linux. Since it's just a python script, all you need is Python 3.x, nothing else. Moreover, it's ridiculously easy to execute it remotely via SSH or other means. 

[Sitecore 9 Core Installer](https://github.com/AlexMayle/sitecore9-core-installer)

There's two scripts `create_default_indexes.py` and `create_specific_indexes.py`. They have the same usage except that the latter requires a list of collection names to make. They'll cover the basic and advanced scenarios, respectively. 

The real goodies are in the fact that the repo contains Solr configs for the XP collections and the xDB collections, with all the changes SIF applies included. The configs that Sitecore collections use are derived from Solr's `basic_configs` configset. The XP collections require the unique id to be `_uniqueid` and the xDB collections require quite a few changes. These have already been applied and stashed in the repo. The python scripts simply provide an easy way to create the collections using these configs.

By default, if you execute `python3 create_default_indexes.py` you'll end up with the following Solr commands generated. To actually execute these commands, just run the same command with `| exec` appended.

``` python
/opt/solr/bin/solr create -c sitecore_master_index -d configsets/xp_config  && \
/opt/solr/bin/solr create -c sitecore_web_index -d configsets/xp_config  && \
/opt/solr/bin/solr create -c sitecore_marketingdefinitions_master -d configsets/xp_config  && \
/opt/solr/bin/solr create -c sitecore_marketingdefinitions_web -d configsets/xp_config  && \
/opt/solr/bin/solr create -c sitecore_marketing_asset_index_master -d configsets/xp_config  && \
/opt/solr/bin/solr create -c sitecore_marketing_asset_index_web -d configsets/xp_config  && \
/opt/solr/bin/solr create -c sitecore_testing_index -d configsets/xp_config  && \
/opt/solr/bin/solr create -c sitecore_suggested_test_index -d configsets/xp_config  && \
/opt/solr/bin/solr create -c sitecore_fxm_master_index -d configsets/xp_config  && \
/opt/solr/bin/solr create -c sitecore_fxm_web_index -d configsets/xp_config  && \
/opt/solr/bin/solr create -c xdb -d configsets/xdb_config  && \
/opt/solr/bin/solr create -c xdb_rebuild -d configsets/xdb_config
```
The script makes a good amount of assumptions, such as you having your Solr binary at `/opt/solr/bin/solr`. Furthermore, it expects that you want index names prefixed with "sitecore" - as this is the way SIF creates them. These, and more, can be overriden of course. Any parameters you could pass to the CREATECOLLECTION API command can also be passed using the `-args` flag. The README contains a number of examples for common and advanced cases. For full usage, leverage the `-h` flag. 

Feel free to fork, send pull requests, or submit issues on GitHub. It's been tested on 9.0.2, but is expected to work with 9.1 as well. 
