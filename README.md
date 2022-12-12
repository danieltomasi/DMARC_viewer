# DMARC_viewer

Docker compose file and folder structure that brings up a Grafana panel showing DMARC report results for your email domain.

This is a slight variation to what [@debricked](https://github.com/debricked) did here: https://github.com/debricked/dmarc-visualizer

## Instructions

1. Set up Docker (Docker engine and Docker Compose, both are included in Docker Desktop) 
2. Download or clone this repository
3. Change to folder where all the files are and enter this command: `docker compose up`. You must be at the same level as `docker-compose.yml` file is.
4. After all the building process finishes (give it a few minutes) you should have an empty _Grafana_ panel here -> http://localhost:3000
5. Then you can start throwing the _.zip_ files with all DMARC reports inside the `/files` directory
6. Give _parsedmarc_ and _Elasticsearch_ a few momnents to process it all. When they are finished, you should be able to see all your data on _Grafana_.
7. Now you can move all your _.zip_ files out of the `/files` folder. Yu can store them anywhere else. If you leave them there, _parsedmarc_ will try to read them again and againn but _Elasticsearch_ won't ingest because it will detect that is duplicated data.

## A little more on this Docker compose multicontainer application

This Docker app is tested on Windows 10 with Docker Desktop 4.15.0 (v20.10.7 for the engine and v2.13.0 for Compose)

The `docker-compose.yml` file tells Docker what services you need and some configuration for each of those services. Inside parsedmarc or grafana folders is also a file named `Dockerfile` with instructions on how to build the image for each service. The buid command in the `docker-compose.yml` tells Compose where to look for that `Dockerfile`. Docker calls that a _"context"_. The `Dockerfile` has all the commands that you would write at the terminal to setup the image you've created. 

More info on Dockerfile: https://docs.docker.com/engine/reference/builder/ and on Compose file: https://docs.docker.com/compose/compose-file/

Other configuration settings on `docker-compose.yml` do this:

**PARSEDMARC SERVICE**

* `container_name`: So that you see that name on Docker Desktop instead of an automatic generated one.

![image](/images/Containers.png?raw=true)

* `volumes`: Maps the `/files` folder to a folder named `/input` on the container and it mounts it as Read Only. [In other words](https://open.spotify.com/album/1eTCeDKIWYskz2k0W7aiOf), it maps the outside folder `/files` (where you put your _.zip_ report files) to Docker container folder `/input` wich is also mounted Read Only. You can open a terminal console in the _parsedmarc_ container an issue an `ls` command to see the mapped `/input` folder. Anything you copy to `/files` folder will be available in `/input` folder.

![image](/images/CLI_parsedmarc.png?raw=true)

* `command`: Runs that command on the container. This command runs _parsedmarc_ binary and tells where to look for the configruation file and also where to look for the DMARC _.zip_ report files. The configuration file is inside `/parsedmarc` folder and is copied to the container with a command on the `Dockerfile` that configures _parsedmarc_ -> `COPY parsedmarc.ini /` which copies `parsedmarc.ini` from `/parsedmarc` folder to root `/` at _parsedmarc_ container.
* `depends_on`: just tells Docker to wait to boot _parsedmarc_ after _elasticsearch_ is done it's own boot. Anyway, _elasticsearch_ really takes longer too fully boot, so _parsedmarc_ will boot while elasticsearch hasn't finished, and it's possible that some errors can appear.
* `restart`: _parsedmarc_ does it's job and shuts down, this is a Docker thing. Each time it shuts down it restarts, you can leave it running and each time you put a _.zip_ file inside `/files` it will read it and pass it to _elasticsearch_.

**ELASTICSEARCH SERVICE:**

* `image`: Pulls _elasticsearch_ image as it is, no additional configuration or `Dockerfile`. _Parsedmarc_ binds to _elasticsearch_ via `parsedmarc.ini` file.
* `environment`: This are a set of variables needed for running _elasticsearch_ on that version (7.9.1). If you try to pull another version of _elaasticsearch_ (latest is 8.5.3 as the writing of this document) the image won't work because that version needs other environment variables to work.

**GRAFANA SERVICE:**
* `ports`: Which port range in which to serve _Grafana_
* `user`: the username that is going to be used to start the container
* `environment`: variables needed to start _Grafana_


I hope it works for you. 

![image](/images/Grafana_view.png?raw=true)
