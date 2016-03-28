# Docker ELK stack with Twitter analysis

This Docker / ELK stack example uses Vagrant + Ansible in place of docker-compose to launch an ELK stack that gathers data based on specified terms from Twitter. 

Note that data does not persist by default; if you kill your containers, your data gathering will start all over again. 

The following nice things are provided for you to read in this readme:
* How to Run This Thing
* How to authorize Twitter, and where to put those details before running
* Info about where this was forked from and how it was changed
* Previously supplied via fork info about "usage"
* "Things I learned" -- which may or may not be helpful to you, fine reader, but I provide them anyway. This includes: vagrant stuff, docker stuff, ELK stack stuff. 
* Random helpful links and resources

Scroll downeth, fine human!

## Run This Thing!

Simple setup:

1. Clone this repository, then cd into this space on your local machine.
2: Read the following section, "Before you Run This Thing!", to add your Twitter authentication details. *If you don't do that, this will not work.*
3. vagrant up
4. Hang out for a few minutes until things are done.
5: In your browser, go to localhost:9200 to see Elasticsearch up and running (should output some pretty json with name, clustername, etc.) 
6: In your browser, you can also visit localhost:9200/twitter/_count/?pretty to see a count of how many tweets matching your keywords have come in. (Note: if nothing is showing up, either something is wrong or nobody has tweeted anything with that keyword *since you started* (this is not searching past tweets, only tweets coming directly off the public timeline *right now*.)
7. In your browser, go to localhost:5601 to see working Kibana UI. (Note: It will be up and running, but that's *it* (for now); scroll down a little for more on Kibana setup. 

## Twitter application authorization, or: Do This, Before you Run This Thing!

This demo magic uses Twitter, which means you will have to modify files/Logstash/config/logstash.conf to add your Super Twitter Secret Details if you want this to work.

Super easy steps:
(This assumes you have a twitter account. If you don't, you will have to do thisfirst, and welcome to 2016! Otherwise, proceed ahead.)
1. [Create](https://apps.twitter.com/app/new) (or use an [existing](https://apps.twitter.com) an Application on Twitter. You can give it whatever name/description you'd like,  but giving it something that helps you remember what it is for can be helpful, since in the event that you someday need to figure out which of your Twitter-related authorizations is doing something, "Sample Twitter App 85" is not helpful. The website field, similarly, does not matter, as this demo will not be posting to twitter on our behalf, but it does need some sort of address. No Callback URL is needed. Click the button at the bottom and sign your life away, you're a Developer now! 
2: Congrats! You made an application. You're currently on the Details tab; click the "Keys and Access Tokens" tab. 
3: Make note of your *consumer_key* and *consumer_token*. 
4: Scroll down to "Your Access Token" and click "create my access token" -- and then make note of your Access Token and Access Token Secret. 
5: In your fork of this repository, navigate your way over to files/Logstash/config/logstash.conf. Replace the areas between the quotes with the actual details you made note of in the previous steps. And to be clear: Yes, the quotes stay. 
6: Optional: logstash.conf specifies *ansible* and *openstack* as the search keywords for which tweets are gathered. You can change this as needed. 

## Kibana Things
"*But I thought I was going to SEEEEEEEE stuff!*" Yeah, I know. Hold your horses. 

Assuming that you really do have tweets in an index named "twitter" in Elasticsearch (visit localhost:9200/twitter/_count/?pretty in your browser to verify; if you see json with a "count" listed, then yes, you do!) -- then do this:

1. Go here in your browser -- localhost:5601
2. A page should exist saying "configure an index pattern"
3. In the "index name or pattern" box replace "logstash-" with "twitter" (**NO DASH**)
4: Leave "Index contains time-based events" checked, Leave "use even times to created index names" unchecked, and use @timestamp as the time-field name. 
5: Press the pretty green create button that should appear once your "twitter" index name was matched. 
6: Yay! You have an index pattern. Now click on the "Discover" tab at the top of the screen. A lovely green bar chart should appear with details below showing times and info about each tweet that has been indexed.  
7: This probably shows the last 15 or 30 minutes, depending on how long it took you to get to this point;; for a different length of time, click "Last 15 minutes" in the upper right-hand corner to change it. 
8: Use the search box to further filter; for example, if you want to filter Ansible tweets, you can type *ansible into the search, and you should see the count of how many tweets included the word Ansible (without double-counting if Ansible was mentioned twice in the tweet, etc.)


## Notes on Forked / Changed Things
Note that this is forked from:
https://github.com/ftl-toolbox/ansible-docker-examples/tree/master/02-elk/atomic-elk

Which was forked from:
https://github.com/deviantony/docker-elk.git

Main differences between this stack and [atomic-elk](https://github.com/ftl-toolbox/ansible-docker-examples/tree/master/02-elk/atomic
-elk): 

* __Vagrant.__ The Vagrantfile has changed doubled both *libvirt.memory* and *vbox.memory* from 1024 to 2048. Why? Who knows! Experimention. My elasticsearch container kept halting, and after changing this, all was better. (Note: I was also at the time using 'trump' as a keyword in *logstash.conf* and that resulted in a lot, and i mean, __a lot__ of tweets coming in. Keywords associated with less contentious terms and lower tweet volume may be more appropriate. Or maybe my computer just doesn't like Donald Trump. Smart computer. Good job!) 
* __Logstash container.__ Previously, *tasks/main.yml* simply launched the logstash container, directly pulling from [Docker Hub](https://hub.docker.com/_/logstash/). Since logstash plugins need to be installed separately, an additional task was added to *tasks/main.yml* to build a Logstash container, and the launching of that container is now specified as the local image name, rather than the previous *logstash:latest*. A Dockerfile was added in *files/Logstash*, which builds upon the previously directly used Docker image, adding one line to install the twitter plugin. 
* __Logstash configuration.__ The logstash configuration (*files/Logstash/config/logstash.conf*) was replaced with inputs for keywords from the twitter timeline, and outputs to elasticsearch with an index name specified (twitter). As noted in the above section on Twitter authorization, this configuration file will need to have your details from twitter oauthy-app stuff inserted, since I am not a fan of publicly sharing my API authorizations. Sooooory!  

# Usage

By default, the stack exposes the following ports:
* 5000: Logstash TCP input.
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana

*WARNING*: If you're using *boot2docker*, you must access it via the *boot2docker* IP address instead of *localhost*.

*WARNING*: If you're using *Docker Toolbox*, you must access it via the *docker-machine* IP address instead of *localhost*.

# Configuration

*NOTE*: Configuration is not dynamically reloaded, you will need to restart the stack after any change in the configuration of a component.

## How can I tune Kibana configuration?

The Kibana default configuration is stored in `kibana/config/kibana.yml`.

## How can I tune Logstash configuration?

The logstash configuration is stored in `logstash/config/logstash.conf`.

The folder `logstash/config` is mapped onto the container `/etc/logstash/conf.d` so you
can create more than one file in that folder if you'd like to. However, you must be aware that config files will be read from the directory in alphabetical order.

## How can I specify the amount of memory used by Logstash?

The Logstash container use the *LS_HEAP_SIZE* environment variable to determine how much memory should be associated to the JVM heap memory (defaults to 500m).

If you want to override the default configuration, add the *LS_HEAP_SIZE* environment variable to the container in the `docker-compose.yml`:

```yml
logstash:
  image: logstash:latest
  command: logstash -f /etc/logstash/conf.d/logstash.conf
  volumes:
    - ./logstash/config:/etc/logstash/conf.d
  ports:
    - "5000:5000"
  links:
    - elasticsearch
  environment:
    - LS_HEAP_SIZE=2048m
```

## How can I enable a remote JMX connection to Logstash?

As for the Java heap memory, another environment variable allows to specify JAVA_OPTS used by Logstash. You'll need to specify the appropriate options to enable JMX and map the JMX port on the docker host.

Update the container in the `docker-compose.yml` to add the *LS_JAVA_OPTS* environment variable with the following content (I've mapped the JMX service on the port 18080, you can change that), do not forget to update the *-Djava.rmi.server.hostname* option with the IP address of your Docker host (replace **DOCKER_HOST_IP**):

```yml
logstash:
  image: logstash:latest
  command: logstash -f /etc/logstash/conf.d/logstash.conf
  volumes:
    - ./logstash/config:/etc/logstash/conf.d
  ports:
    - "5000:5000"
    - "18080:18080"
  links:
    - elasticsearch
  environment:
    - LS_JAVA_OPTS=-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.port=18080 -Dcom.sun.management.jmxremote.rmi.port=18080 -Djava.rmi.server.hostname=DOCKER_HOST_IP -Dcom.sun.management.jmxremote.local.only=false
```

## How can I tune Elasticsearch configuration?

The Elasticsearch container is using the shipped configuration and it is not exposed by default.

If you want to override the default configuration, create a file `elasticsearch/config/elasticsearch.yml` and add your configuration in it.

Then, you'll need to map your configuration file inside the container in the `docker-compose.yml`. Update the elasticsearch container declaration to:

```yml
elasticsearch:
  build: elasticsearch/
  command: elasticsearch -Des.network.host=_non_loopback_
  ports:
    - "9200:9200"
  volumes:
    - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
```

You can also specify the options you want to override directly in the command field:

```yml
elasticsearch:
  build: elasticsearch/
  command: elasticsearch -Des.network.host=_non_loopback_ -Des.cluster.name: my-cluster
  ports:
    - "9200:9200"
```

# Storage

## How can I store Elasticsearch data?

The data stored in Elasticsearch will be persisted after container reboot but not after container removal.

In order to persist Elasticsearch data even after removing the Elasticsearch container, you'll have to mount a volume on your Docker host. Update the elasticsearch container declaration to:

```yml
elasticsearch:
  build: elasticsearch/
  command: elasticsearch -Des.network.host=_non_loopback_
  ports:
    - "9200:9200"
  volumes:
    - /path/to/storage:/usr/share/elasticsearch/data
```

This will store elasticsearch data inside `/path/to/storage`.

## Things I learned

### Vagrant and Docker

* *"Is this thing on?"*  Well, if your vagrant up showed a bunch of fun things and it seemed to result in your ansible run showing Great Success, then, probably. But who knows! Here's a few handy things you can do though:

1. vagrant global-status: run this in the same path where you're running vagrant.  It should return a list that includes an id that is associated with what you are running. If it doesn't say "running", well, then it's not running.
2: Assuming it is running: you can do *vagrant ssh <id>* -- which will bring you into the nice virtualbox where all the magic is happening, which is running on a CentOS 7 Atomic machine / host / whatever. There, you can type *docker ps -a* to get a list of all your containers that are running. If status appears to not be happy, troubleshoot!
* Note: In the case of this particular stack -- you should see that your container ID associated with the localkibana image should have *0.0.0.0:5601->5601/tcp* listed under ports, and your container ID associated with the elasticsearch:latest image should have *0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp* listed under ports. If the ports aren't showing up, then something is wrong somewhere, and while the containers may be happily running independently, they're not shuffling data around between them. That will need fixing. :D 
3: If a container is stopped, you can do a *docker start -i <container id>* and see if it will come up. This will run it in interactive mode.  
4: Need Moar Context? Get more verbose Ansible output from Vagrant uncommenting the ansible.verbose line in your Vagrantfile.

## Resources
Thanks, Internet!

* [David Pilato's nice blog on indexing twitter with logstash and elasticsearch](http://david.pilato.fr/blog/2015/06/01/indexing-twitter-with-logstash-and-elasticsearch/). Note: This is for older versions of the ELK stack; you no longer need to specify http as a protocol in a logstash configuration, as that is the default, and *host* in logstash.conf should now say *hosts*. 
* [ELK / Twitter example](https://github.com/elastic/examples/tree/master/ELK_twitter). Note that importing twitter_kibana.json is a thing that was cool in Kibana 3, and not Kibana 4, as we are using here, so that part doesn't wind up being as helpful as you might wish.
* [Some Vagrant + Docker](https://technology.amis.nl/2015/08/22/first-steps-with-provisioning-of-docker-containers-using-vagrant-as-provider/) tips, while not ansible-specific, are helpful anyway in some places to help you figure out what the heck is going on if you haven't really Dockered all the Things before.

