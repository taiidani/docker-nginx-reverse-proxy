# Docker Nginx Reverse Proxy

With Docker 1.10 a new networking paradigm went into effect where Docker containers have an embedded DNS server to assist with service discovery within their network. This configuration replaced the existing modification of containers `/etc/hosts` files, and is the recommended way to perform discovery in the future.

One major benefit to this idea is that, with reverse proxies such as Nginx and HAProxy 1.6+, standard DNS resolution can take place to hook up these services to their containers. [gesellix/docker-haproxy-network](https://github.com/gesellix/docker-haproxy-network) is a great example of how to perform this with HAProxy 1.6+ and is highly recommended, but HAProxy does not at present allow the services to be _unavailable at startup_. Following the directions in that article, if you start your proxy before you start your web server the proxy will crash.

This example uses Nginx for the reverse proxy instead, and supports services being unavailable at startup. It is set up so that the proxy will route to a second Nginx container running as a web server. Using any "web.*" or "web-*" URL resolving to your local machine will route to the "web" container. This enables you to use subdomains to point to each of your containers, such as:
* web.localhost.com
* db.local.dev
* www-qa1.testboxes.com

## What Do I Need

You're going to need:
* Docker 1.10+
* A curious nature

The below directions will need:
* Docker Compose 1.6.0+

## What Do I Do

First, Git clone this code to your machine running Docker:
```bash
git clone https://github.com/taiidani/docker-nginx-reverse-proxy.git
cd docker-nginx-reverse-proxy
```

Second, run Docker Compose to build the image and run the containers. The below command will ensure that the image definition is always rebuilt and will attach to the containers as they start so that you may view the logs. When you Ctrl+C to end the session, `down` will run and clean up the stopped containers & network.
```bash
docker-compose up --build && docker-compose down
```

Third, alter your `/etc/hosts` file so that you resolve "web.localhost.com" to 127.0.0.1. Here is my file -- you will only need to add the last line.
```
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost

127.0.0.1 web.localhost.com
```

Finally open a web browser and navigate to "http://web.localhost.com". The fact that your URL starts with "web" will drive the proxy to load the "web" container and serve a "Success" message.

## What About Lazy DNS?

Right, I promised that the proxy could run first! Try this out:
```bash
docker-compose up -d proxy
```

That starts the proxy. View its logs with `docker-compose logs proxy` to see if it survived starting up, then check out "http://web.localhost.com". Your browser will throw ugly 502 errors all over the place, but the proxy will keep on kickin'. Fix the errors by starting the web container with:
```bash
docker-compose up -d web
```

Now when you try "http://web.localhost.com" again the routing will succeed. Don't forget to clean up after you're done!
```bash
docker-compose down
```