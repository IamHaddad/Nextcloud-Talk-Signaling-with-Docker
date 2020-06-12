# Nextcloud-Talk Signaling with Docker
Instructions on how to use the [Nextcloud-Talk Signaling Backend](https://github.com/strukturag/nextcloud-spreed-signaling) with 3 docker containers for Janus, Nats, and Coturn. Note almost all the files are from the Strukturag's "Nextcloud-Spreed-Signaling" Server I just modified a couple of things and added simpler installation instructions.

## Create Signalling Server

### Building

You will need at least go 1.6 and make to build the signaling server. All other dependencies are fetched automatically while building.

Download The [Signalling Server](https://github.com/strukturag/nextcloud-spreed-signaling), unzip then cd into the directory ``cd nextcloud-spreed-signaling`` and run:

    $ make build

### Configuration

There is a default configuration file included called `server.conf.in` copy this to `server.conf` and adjust the following for installing with the [docker-compose.yml](docker-compose.yml) as is,

Uncomment ```listen = 127.0.0.1:8080``` under `[http]` this is the port the signaling server will be listening on.

Replace `nextcloud.domain.invalid` in `allowed = nextcloud.domain.invalid` under `[backend]` so that the signaling server knows which hostnames its allowed to use.

Uncomment ```url = nats://localhost:4222``` under `[nats]` the signaling server will use this to connect to the nat server in the docker container.

Add `ws://localhost:8188` after the `url = ` under `[mcu]` the signaling server will use this to connect to the Janus server in the docker container.

Create a apikey for the turn server `apikey = the-api-key-for-the-rest-service`.

Also add a secret which will be used to connect to the turn server (This needs to be the same as in the docker-compose.yml file for coturn) `apikey = the-api-key-for-the-rest-service`.

And add a list of turnservers after the `servers =` ```servers = turn:localhost:3478?transport=udp,turn:localhost:3478?transport=tcp``` this is used to connect to the Coturn server in the docker container.

You also need to change the `REALM:` in the `docker-compose.yml` to the domain name of your nextcloud instance.

And make sure the `STATIC_SECRET:` is the same as in the `server.conf` file.

### Docker Compose

Then to deploy the Janus Nats and Coturn servers run:

```bash
docker-compose build
docker-compose up -d
```

### Running

Once all your docker container are up and running you can start the signaling server

     $ ./bin/signaling
     
By default, the configuration is loaded from server.conf in the current directory, but a different path can be passed through the --config option.

     $ ./bin/signaling --config /etc/signaling/server.conf
     
### Running as daemon

To run as systemd create a dedicated group:
```bash
sudo groupadd signaling
```

Create a dedicated user:

```bash
sudo useradd --system \
    --gid signaling \
    --shell /usr/sbin/nologin \
    --comment "Standalone signaling server for Nextcloud Talk." \
    signaling
```

Copy `server.conf.in` to `/etc/signaling/server.conf` and fix permissions:

```bash
sudo chmod 600 /etc/signaling/server.conf
sudo chown signaling: /etc/signaling/server.conf
```

Copy `dist/init/systemd/signaling.service` to `/etc/systemd/system/signaling.service`. Also copy `./bin/signaling` to `/usr/bin/signaling`.

Enable and start service:

```bash
systemctl enable signaling.service
systemctl start signaling.service
```

### Apache

To configure the Apache webservice as frontend for the standalone signaling
server, the modules `mod_proxy_http` and `mod_proxy_wstunnel` must be enabled
so WebSocket and API backend requests can be proxied:

    $ sudo a2enmod proxy
    $ sudo a2enmod proxy_http
    $ sudo a2enmod proxy_wstunnel
    $ sudo a2enmod redirect

Now the Apache `VirtualHost` configuration can be extended to forward requests
to the standalone signaling server (assuming the server is running on the local
interface on port `8080` below):

    <VirtualHost *:443>

        # ... existing configuration ...

        # Enable proxying Websocket requests to the standalone signaling server.
        ProxyPass "/standalone-signaling/"  "ws://127.0.0.1:8080/"

        RewriteEngine On
        # Websocket connections from the clients.
        RewriteRule ^/standalone-signaling/spreed$ - [L]
        # Backend connections from Nextcloud.
        RewriteRule ^/standalone-signaling/api/(.*) http://127.0.0.1:8080/api/$1 [L,P]

        # ... existing configuration ...

    </VirtualHost>
    
Note you need an ssl certificate for the Virtual Host

## Setup of Nextcloud Talk

Login to your Nextcloud as admin and open the additional settings page. Scroll
down to the "Talk" section and enter the base URL of your standalone signaling
server in the field "External signaling server".
Please note that you have to use `https` if your Nextcloud is also running on
`https`. Usually you should enter `https://myhostname/standalone-signaling` as
URL.

The value "Shared secret for external signaling server" must be the same as the
property `secret` in section `backend` of your `server.conf`.

If you are using a self-signed certificate for development, you need to uncheck
the box `Validate SSL certificate` so backend requests from Nextcloud to the
signaling server can be performed.
