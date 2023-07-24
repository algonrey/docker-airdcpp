

# Deprecated use gangefors/airdcpp-webclient instead


##### Using the lastest Version 

---
AirDC++ Web Client Docker image
===============================

You must have proper knowledge of [Docker] to use this image.


Run the application
-------------------

    docker volume create --name airdcpp
    docker run -d --name airdcpp \
        -p 80:5600 -p 443:5601 -p 21248:21248 -p 21248:21248/udp -p 21249:21249 \
        -v airdcpp:/.airdcpp \
        -v $HOME/Downloads:/Downloads \
        -v $HOME/Share:/Share \
        algonrey/airdcpp

The web UI will be available on [http://localhost].

If you want to run the application on any other port than 80, just update
the `-p` option in the command, e.g `-p 5600:5600` to bind to port 5600.
See [Exposed Ports] below for details.

Username / password for the default admin account is: `admin` / `password`

##### Explanation

    docker volume create --name airdcpp

This command creates a named volume that will store the application settings.

> NOTE Run the `volume create` command only once.

    docker run -d --name airdcpp \
        -p 80:5600 -p 443:5601 -p 21248:21248 -p 21248:21248/udp -p 21249:21249 \
        -v airdcpp:/.airdcpp \ 
        -v $HOME/Downloads:/Downloads \
        -v $HOME/Share:/Share \
        algonrey/airdcpp

This command starts a container using the default settings built into the
image, binding the application to port 80/443 (default http/https port) so
it's readily available on [http://localhost] and [https://localhost]
It will also mount Downloads and Share from you home directory, change these
according to your personal setup.


Run as non-privileged user
--------------------------

If you'd like to run in a non-privileged container you can do that as well.
It might even be preferable since then you get to decide who owns the
downloaded files.

    docker run -d --name airdcpp \
        -p 80:5600 -p 443:5601 -p 21248:21248 -p 21248:21248/udp -p 21249:21249 \
        -v airdcpp:/.airdcpp \
        -v $HOME/Downloads:/Downloads \
        -v $HOME/Share:/Share \
        -u $(id -u):$(id -g) \
        algonrey/airdcpp

> NOTE
If you already have run the container as root, the files in the volume might
be owned by root. Fix that by `chown`ing the files to the user you run as.

    docker run --rm \
        -v airdcpp:/.airdcpp \
        ubuntu:16.04 \
        chown -R $(id -u):$(id -g) /.airdcpp


docker-compose
--------------

There is a docker-compose file available to set up the application as a
service on a docker host. Just run the following.

    docker-compose up -d

### Environment

You can configure some aspects of the application when using docker-compose
by setting these environment variables before running `docker-compose up -d`.

- `UID`

  Container is started with this user id. Defaults to 0 (root).
  Usually you want this to be $(id -u).

- `GID`

  Container is started with this group id. Defaults to 0 (root).
  Usually you want this to be $(id -g).

- `HTTP_PORT`

  Published HTTP port. Defaults to 5600.

- `HTTPS_PORT`

  Published HTTPS port. Defaults to 5601.

- `TCP_PORT`

  Published TCP port for incoming connections. Defaults to 21248. If this is
  changed you have to change it in the application settings as well.

- `UDP_PORT`

  Published UDP port for incoming connections. Defaults to 21248. If this is
  changed you have to change it in the application settings as well.

- `TLS_PORT`

  Published TLS port for incoming connections. Defaults to 21249. If this is
  changed you have to change it in the application settings as well.


Volumes
-------

- `/.airdcpp`

  This volume stores the application settings.

  > NOTE
  If you mount this directory from your host you will not have the default
  configuration files in the settings directory. You need to copy them from
  this repo. The files are found in the [.airdcpp] directory.

- `/Downloads`

  This is the default Download folder, but you can change this in the
  settings through the web UI.

- `/Share`

  This is the default share folder.


Exposed Ports
-------------

- `5600` HTTP port

- `5601` HTTPS port

- `21248` TCP and UDP port for incoming connections. You have to publish this
  on the same port number otherwise clients will not be able to connect.

- `21249` TCP port for incoming encrypted connections. You have to publish this
  on the same port number otherwise clients will not be able to connect.

If you want to use other ports for incoming connections you are can change
them under Settings>Connectivity>Advanced>Ports in the web UI.

The incoming connection ports are used to be able to be in *active mode*. This
allows you to connect to all peers in a hub, including the ones in *passive mode*.

Read more about connectivity modes in the [official FAQ][conn_faq].


Add/modify admin users
----------------------

To add/modify _adminitrative_ users to the web interface, run the following.

    docker stop airdcpp
    docker run --rm -it --volumes-from airdcpp \
        algonrey/airdcpp --add-user
    docker start airdcpp

> NOTE
You must stop the webclient application container before running this
command. If you add a user while it's running, the configuration will be
overwritten when the application shuts down.


Upgrade
-------

1. Pull the latest image.
2. Stop and remove the container.
3. Start a new container with the same command you started the old one.

Example:

    docker pull algonrey/airdcpp
    docker stop airdcpp
    docker rm airdcpp
    docker run -d --name airdcpp \
        -p 80:5600 -p 443:5601 -p 21248:21248 -p 21248:21248/udp -p 21249:21249 \
        -v airdcpp:/.airdcpp \
        -v $HOME/Downloads:/Downloads \
        -v $HOME/Share:/Share \
        algonrey/airdcpp


Enable HTTPS
------------

The image comes with self-signed certificates so you should be able to use
HTTPS out of the box. But if you want to generate your own certificates here's
how you do it.

> NOTE The container must be running.

    docker exec -it airdcpp openssl req \
        -subj "/C=US/ST=State/L=City/O=/CN=localhost" \
        -x509 -nodes -days 365 -newkey rsa:2048 \
        -keyout /.airdcpp/Certificates/client.key \
        -out /.airdcpp/Certificates/client.crt

Change the CN string to whatever the domain name or IP you are running your
service on. You can also add more information in the -subj string if you want.
Check [this site][certs] for more information on the different fields.


[docker]: https://docs.docker.com/learn/
[Exposed Ports]: #exposed-ports
[http://localhost]: http://localhost
[https://localhost]: https://localhost
[.airdcpp]: .airdcpp
[conn_faq]: http://dcplusplus.sourceforge.net/webhelp/faq_connection.html
[certs]: http://www.shellhacks.com/en/HowTo-Create-CSR-using-OpenSSL-Without-Prompt-Non-Interactive
