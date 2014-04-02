docker-cassandra
================

Standlone/Clustered Datastax Community running on top of a Ubuntu-based Docker image


Running as standalone
=====================

### Docker

Just start the image normally

    docker run -d zmarcantel/docker-cassandra

The necessary ports are exposed, so you can interact with the docker image as you normally do.

If you need help, see [Usage Guide](#usage-guide).


### Vagrant

Simply use the builtin `docker` provisioner to pull and start the image.

```ruby
config.vm.provision "docker" do |d|
    d.pull_images "zmarcantel/docker-cassandra"
    d.run "cass", image: "zmarcantel/docker-cassandra"
end
```


Running as a cluster
====================

To get the images to talk to each other, simply use the docker `link` and `name` options.

We can __link__ up to 10 containers.

You are able to create as many containers as you'd like, but the init script only considers up to the first 10 as seeds.

Truly, you only need one other seed and for simplicity, that's how this example works. The Cassandra cluster will gossip its way into being fully networked rather quickly. There is no reason a machine `cass9999` can't just reuse a seed from `cass[0-9]`.


### Docker

Start a first image, and then link them all up.

##### Start the first image

    docker run -d -name cass0 zmarcantel/docker-cassandra

##### Link the cluster

    docker run -d -name cass1 -link cass0:cass0 zmarcantel/docker-cassandra
    docker run -d -name cass2 -link cass1:cass1 zmarcantel/docker-cassandra


### Vagrant

We'll do the same as above, but using the `docker` provisioner.


##### Start the first image

```ruby
config.vm.provision "docker" do |d|
    d.pull_images "zmarcantel/docker-cassandra"

    d.run  "cass0",
      image: "zmarcantel/docker-cassandra"

    d.run  "cass1",
      image: "zmarcantel/docker-cassandra",
      cmd: "-link cass0:cass0"

    d.run  "cass2",
      image: "zmarcantel/docker-cassandra",
      cmd: "-link cass1:cass1"
end
```

##### Link the cluster

    docker run -d -name cass1 -link cass0:cass0 zmarcantel/docker-cassandra
    docker run -d -name cass2 -link cass1:cass1 zmarcantel/docker-cassandra



Usage Guide
===========

## Basic Docker One-Liners

Get image ID for `docker-cassandra` on your machine

    docker ps | grep "zmarcantel/docker-cassandra" | head -n1 | cut -d' ' -f1

__This is a very convenient export: the image ID__

    export CASSDOCK_ID=`docker ps | grep "zmarcantel/docker-cassandra" | head -n1 | cut -d' ' -f1`

List the IPs of containers running `docker-cassandra`

    docker inspect $CASSDOCK_ID | grep IPAddress | sed 's/"IPAddress": "/ /g' | sed 's/",//g' | sed 's/ //g'

Get the above in a comma-separated format

    docker inspect $CASSDOCK_ID | grep IPAddress | sed 's/"IPAddress": "/ /g' | sed 's/",//g' | sed 's/ //g' \
    sed -e :a -e N -e 's/\n/,/' -e ta

Or with spaces if you prefer

    docker inspect $CASSDOCK_ID | grep IPAddress | sed 's/"IPAddress": "/ /g' | sed 's/",//g' | sed 's/ //g' \
    sed -e :a -e N -e 's/\n/ /' -e ta


## Talking to Cassandra

Every docker image is given an IP to communicate with.

Use the above commands to get the IP of your docker images. Or manually look them up using `docker ps` and `docker inspect`.

The Following IPs are open (default Cassandra ports):

    7199 7000 7001 9160 9042

#### Standalone instance

Every language and client has a different syntax, but simply:

````js
    cassandra.connect("172.17.0.2")
````

Obviously modify to fit the method of connecting for your library.


#### Cluster

The IPs of the cluster are determined using [one-liners] and your application input channels.

I provide a way to get the info, but how it gets into your application is up to you.

The below IPs are examples to give a gist, but you must get your relevant IPs using the above commands.

````js
    cassandra.connect(["172.17.0.2", "172.17.0.3", "172.17.0.4"])
````
