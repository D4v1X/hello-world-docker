## Description
add docker to play-framework application.

#### Install Docker

```commandline
sudo apt-get install docker
```

```commandline
sudo apt-get install docker-compose
```

#### Adjust permissions

```commandline
sudo usermod -a -G docker $USER
```

#### Process
Build your project’s artifact by entering hello-world-docker and ```sbt dist```, producing ```target/universal/hello-world-docker-1.0-SNAPSHOT.zip```. Expect a lot more output and longer build time if this is your first.

Extract your artifact to a known location, ```svc``` for deployment:

```commandline
set -x && unzip -d svc target/universal/*-1.0-SNAPSHOT.zip && mv svc/*/* svc/ && rm svc/bin/*.bat && mv svc/bin/* svc/bin/start
```

DOCKERFILE

```dockerfile
FROM openjdk:8-jre
COPY svc /svc
EXPOSE 9000 9443
CMD /svc/bin/start -Dhttps.port=9443 -Dplay.http.secret.key=prod
```

Build from the same directory containing your `Dockerfile` and `svc` with 
```commandline
docker build -t hello-world-docker .
```

Run your Docker container with ports exposed if you plan to use a `browser` or `curl` to interact with it. You can also invoke `curl` from inside the container if it’s installed (`openjdk:8-jre` bundles `curl , but others may not).
```commandline
docker run -it -p 9000:9000 -p 9443:9443 --rm hello-world-docker
```
Where `-p` publishes ports, 
`-it` supplies TTY control so you can CTRL C the container, 
and `--rm` deletes the container instance (not image) container on exit (it’s two hyphens, not one en-dash). 
You can add `-d` to detach and run it in the background.

Your `docker ps` should resemble.

Interact with your container using `curl` from your host:
#### Tests

- TESTING HTTP:
```commandline
$ curl -I localhost:9000
HTTP/1.1 200 OK
Set-Cookie: PLAY_SESSION=935c3dbf94ba41f969fbf084de627bc695c2b326-csrfToken=fb0f50c61fa2e0512a83cb3fe07e6645c2ed8597-1497822065731-aebab6068130ca25cfdc16ec; Path=/; HTTPOnly
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Security-Policy: default-src 'self'
X-Permitted-Cross-Domain-Policies: master-only
Content-Length: 439
Content-Type: text/html; charset=utf-8
Date: Sun, 18 Jun 2017 21:41:05 GMT
```

- TESTING HTTPS:
```commandline
$ curl -Ikv https://localhost:9443/
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 9443 (#0)
* TLS 1.2 connection using TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* Server certificate: localhost
> HEAD / HTTP/1.1
> Host: localhost:9443
> User-Agent: curl/7.51.0
> Accept: */*
>
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
< Set-Cookie: PLAY_SESSION=23aec23a5d9dabcf8568777821a8ecc4b45d2fd8-csrfToken=89bd94f48863a982efb861e500b5439bdfbca1dc-1497822198482-3fa6567fcee05ccb0084e73e; Path=/; HTTPOnly
Set-Cookie: PLAY_SESSION=23aec23a5d9dabcf8568777821a8ecc4b45d2fd8-csrfToken=89bd94f48863a982efb861e500b5439bdfbca1dc-1497822198482-3fa6567fcee05ccb0084e73e; Path=/; HTTPOnly
< X-Frame-Options: DENY
X-Frame-Options: DENY
< X-XSS-Protection: 1; mode=block
X-XSS-Protection: 1; mode=block
< X-Content-Type-Options: nosniff
X-Content-Type-Options: nosniff
< Content-Security-Policy: default-src 'self'
Content-Security-Policy: default-src 'self'
< X-Permitted-Cross-Domain-Policies: master-only
X-Permitted-Cross-Domain-Policies: master-only
< Content-Length: 439
Content-Length: 439
< Content-Type: text/html; charset=utf-8
Content-Type: text/html; charset=utf-8
< Date: Sun, 18 Jun 2017 21:43:18 GMT
Date: Sun, 18 Jun 2017 21:43:18 GMT

<
* Curl_http_done: called premature == 0
* Connection #0 to host localhost left intact
```

- TESTING OPENSSL:
```commandline
$ sudo docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
033ea33511bb        hello-world-docker    "/bin/sh -c '/svc/bi…"   6 minutes ago       Up 6 minutes        0.0.0.0:9000->9000/tcp, 0.0.0.0:9443->9443/tcp   dazzling_meitner
```
```commandline
$ sudo docker inspect 033ea33511bb | grep "IPAddress"
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
```

```commandline
$ sudo docker run --rm -it alpine:latest sh -c "apk --no-cache add openssl && echo newline | openssl s_client -connect 172.17.0.2:9443 -cipher ECDHE-RSA-AES256-GCM-SHA384 -curves secp521r1"
```
And you’ll see a line like:
```Server Temp Key: ECDH, P-521, 521 bits```