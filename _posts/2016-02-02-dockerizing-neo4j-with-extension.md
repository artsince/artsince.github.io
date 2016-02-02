---
layout: post
title: Dockerizing a Neo4j instance with an extension
tags: neo4j, docker, elasticsearch
excerpt: Setting up neo4j docker image to work with an unmanaged extension
---

### {{page.title}}

In this post I will demonstrate how to setup a Neo4j container with an existing database and a custom extension that synchronizes data with an Elastic Search instance. The extension I am using is a fork of [this project](https://github.com/neo4j-contrib/neo4j-elasticsearch) on GitHub.

To get started, pull the 2.3.2 tag of Neo4j image. Some functionality I will explain later in the post depend on this version (and up).

<script src="https://gist.github.com/artsince/111e145b2250db2257be.js?file=one.sh"></script>

#### Adding volumes
Volumes are a way to persist data across several invocations of a container by mapping directories from the host file system to the container. For this Neo4j instance, I will need to add three volume definitions.

1. data volume: This is the `data` folder inside neo4j installation which containing the `graph.db` directory which actually holds the database files.
2. plugins volume: the elastic search extension has some jar files that should go to the plugins folder of Neo4j.
3. extension volume: This is my custom volume that contains a script file to be run before Neo4j starts.

#### Before-Start Hook
With 2.3.2 Neo4j image, I can use `EXTENSION_SCRIPT` environment variable to specify a script file to execute before Neo4j starts. The Elastic Search extension needs two parameters to be set in `neo4j.properties` file, so I will create a script that concatenates two lines to the end of the file.

<script src="https://gist.github.com/artsince/111e145b2250db2257be.js?file=two.sh"></script>

So, my command looks like this:
<script src="https://gist.github.com/artsince/111e145b2250db2257be.js?file=three.sh"></script>

As a bonus, I can add service definitions for neo4j and elasticsearch in a docker-compose file for easier networking configuration and maintainability.

<script src="https://gist.github.com/artsince/111e145b2250db2257be.js?file=four.yml"></script>

Now, starting docker-compose with `--x-networking` flag will create both containers with `container_name` as their host names.

As another bonus, I can add modify my extension script to make sure the Elastic Search container starts before Neo4j starts, so that I will not get any Connection Refused errors on startup.

<script src="https://gist.github.com/artsince/111e145b2250db2257be.js?file=five.sh"></script>

<br/>