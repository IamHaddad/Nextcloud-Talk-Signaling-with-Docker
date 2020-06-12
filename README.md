# Janus-Nats-Coturn
Creates 3 docker containers for Janus Nats and Coturn for use with [Nextcloud-Talk Signaling Backend](https://github.com/strukturag/nextcloud-spreed-signaling)
### Building

You will need at least go 1.6 and make to build the signaling server. All other dependencies are fetched automatically while building.

Download The [Signalling Server](https://github.com/strukturag/nextcloud-spreed-signaling), unzip then cd into the directory ``cd nextcloud-spreed-signaling`` and run:

    $ make build

### Configuration

There is a default configuration file included called `server.conf.in` copy this to `server.conf` and adjust the following for installing with the [docker-compose.yml](docker-compose.yml) as is,

Uncomment ```listen = 127.0.0.1:8080``` under `[http]` this is the port the signalling server will be listening on.

Replace `nextcloud.domain.invalid` in `allowed = nextcloud.domain.invalid` under `[backend]` so that the signalling server knows which hostnames its allowed to use.

Uncomment ```url = nats://localhost:4222``` under `[nats]` the signalling server will use this to connect to the nat server in the docker container.

Add `ws://localhost:8188` after the `url = ` under `[mcu]` the signalling server will use this to connect to the Janus server in the docker container.

Create a apikey for the turn server `apikey = the-api-key-for-the-rest-service`.

Also add a secret which will be used to connect to the turn server (This needs to be the same as in the docker-compose.yml file for coturn) `apikey = the-api-key-for-the-rest-service`.

And add a list of turnservers after the `servers =` ```servers = turn:localhost:3478?transport=udp,turn:localhost:3478?transport=tcp``` this is used to connect to the Coturn server in the docker container.

You also need to change the `REALM:` in the `docker-compose.yml` to the domain name of your nextcloud instance.

And make sure the `STATIC_SECRET:` is the same as in the `server.conf` file.

#### Docker Compose

To deploy the Janus Nats and Coturn servers run:

```bash
docker-compose build
docker-compose up -d
```

### Running

Once all your docker container are up and running you can start the signalling server

     $ ./bin/signaling
     
By default, the configuration is loaded from server.conf in the current directory, but a different path can be passed through the --config option.

     $ ./bin/signaling --config /etc/signaling/server.conf
