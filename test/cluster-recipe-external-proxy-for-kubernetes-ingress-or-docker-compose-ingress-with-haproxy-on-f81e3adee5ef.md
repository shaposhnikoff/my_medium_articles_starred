Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m114[39m }

# External proxy for Kubernetes (or docker-compose) Ingress with HAProxy

Click here to share this article on LinkedIn »

If you deploy multiple non-relative applications in your Kubernetes cluster, you might think about having a separate external proxy to obtain a different public ip for each application. We can use HAProxy as it has an ability to proxy http/https request on layer 7(http) or layer 4 (tcp). In our case it is important to proxy https requests on HAProxy without tls termination, because Kubernetes Ingress (based on nginx) with kube-lego takes over this functionality. The second problem is how to pass the original client IP to the application. The solution is proxy protocol, that is supported by both HAProxy and Nginx. Take a look on how to setup such proxy on fresh Digital Ocean CentOS 7 node.

![](https://cdn-images-1.medium.com/max/2102/1*vhQRR5Lp-ut28z7uay-KoA.png)

## Setup HAProxy

Consider that the firewalld, iptables, selinux is disabled. Update OS

    yum update
    yum install epel-release

Check available version of HAProxy.

    yum info haproxy
    .. 1.5.18

Stock CentOS has relatively old version of HAProxy, so let’s install latest HAProxy from [IUS Community repositories](https://ius.io/GettingStarted/).

    yum install [https://centos7.iuscommunity.org/ius-release.rpm](https://centos7.iuscommunity.org/ius-release.rpm)
    yum install haproxy18u
    haproxy -v
    HA-Proxy version 1.8.4-1deb90d 2018/02/08

Edit haproxy config, replace X.X.X.X with your Kubernetes ingress or docker-compose “ingress” public IP.

    vi /etc/haproxy/haproxy.cfg
    global
        chroot      /var/lib/haproxy
        pidfile     /var/run/haproxy.pid
        maxconn     4000
        user        haproxy
        group       haproxy
        daemon
        stats socket /var/lib/haproxy/stats

    defaults
        mode http
        log global
        option                  redispatch
        retries                 3
        timeout http-request    10s
        timeout queue           1m
        timeout connect         10s
        timeout client          1m
        timeout server          1m
        timeout http-keep-alive 10s
        timeout check           10s
        maxconn                 3000

    frontend http_front
        mode tcp
        bind *:80
        default_backend http_back

    frontend https_front
        mode tcp
        bind *:443
        default_backend https_back

    backend http_back
        mode tcp
        server server01 X.X.X.X:80 **send-proxy**

    backend https_back
        mode tcp
        server server01 X.X.X.X:443 **send-proxy**

Enable and start HAProxy

    systemctl enable haproxy
    systemctl start haproxy
    systemctl status haproxy -l

## Kubernetes Ingress with proxy protocol support

[Nginx based Kubernetes Ingress supports proxy protocol using use-proxy-protocol directive out of the box](https://github.com/kubernetes/ingress-nginx/blob/d27829ce7ebc5f202816c52f69985bc102db9a63/deploy/provider/aws/patch-configmap-l4.yaml#L9). Just replace configmap.yaml in [ingress-nginx mandatory commands](https://github.com/kubernetes/ingress-nginx/blob/master/deploy/README.md#mandatory-commands) or apply new configMap.

    # REPLACE
    # curl [https://raw.githubusercontent.com/kubernetes/ingress](https://raw.githubusercontent.com/kubernetes/ingress-#)-nginx/master/deploy/configmap.yaml \
    #    | kubectl apply -f -
    # WITH

    curl [https://raw.githubusercontent.com/kubernetes/ingress-nginx/d27829ce7ebc5f202816c52f69985bc102db9a63/deploy/provider/aws/patch-configmap-l4.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/d27829ce7ebc5f202816c52f69985bc102db9a63/deploy/provider/aws/patch-configmap-l4.yaml) \
        | kubectl apply -f -

    # OR apply a new config
    cat <<EOF | kubectl apply -f -
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: nginx-configuration
      namespace: ingress-nginx
      labels:
        app: ingress-nginx
    data:
      use-proxy-protocol: "true"    
    EOF

## **Docker-compose ingress-like configuration with Proxy protocol support**

It is possible to build an Ingress-like environment for docker-compose using nginx, jwilder/docker-gen and jrcs/letsencrypt-nginx-proxy-companion containers. Let’s look how to add proxy-protocol support to this configuration. All required files can be found at [https://github.com/olegsmetanin/docker-compose-examples](https://github.com/olegsmetanin/docker-compose-examples).

File layout:

    |- docker-compose.yaml
    |- nginx-proxy-protocol.tmpl

docker-compose.yaml

    version: '3'
    services:
      nginx-web:
        image: nginx
        labels:
          com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy: "true"
        container_name: docker-compose-ingress
        restart: always
        ports:
          - "80:80"
          - "443:443"
        volumes:
          - ./data/conf.d:/etc/nginx/conf.d
          - ./data/vhost.d:/etc/nginx/vhost.d
          - ./data/html:/usr/share/nginx/html
          - ./data/certs:/etc/nginx/certs:ro
          - ./data/htpasswd:/etc/nginx/htpasswd:ro

    nginx-gen:
        image: jwilder/docker-gen
        command: -notify-sighup docker-compose-ingress -watch -wait 5s:30s /etc/docker-gen/templates/nginx.tmpl /etc/nginx/conf.d/default.conf
        container_name: docker-compose-ingress-nginx-gen
        restart: always
        volumes:
          - ./data/conf.d:/etc/nginx/conf.d
          - ./data/vhost.d:/etc/nginx/vhost.d
          - ./data/html:/usr/share/nginx/html
          - ./data/certs:/etc/nginx/certs:ro
          - ./data/htpasswd:/etc/nginx/htpasswd:ro
          - /var/run/docker.sock:/tmp/docker.sock:ro
          - ./nginx-proxy-protocol.tmpl:/etc/docker-gen/templates/nginx.tmpl:ro

    nginx-letsencrypt:
        image: jrcs/letsencrypt-nginx-proxy-companion
        container_name: docker-compose-ingress-nginx-letsencrypt
        restart: always
        volumes:
          - ./data/conf.d:/etc/nginx/conf.d
          - ./data/vhost.d:/etc/nginx/vhost.d
          - ./data/html:/usr/share/nginx/html
          - ./data/certs:/etc/nginx/certs:rw
          - /var/run/docker.sock:/var/run/docker.sock:ro
        environment:
          NGINX_DOCKER_GEN_CONTAINER: docker-compose-ingress-nginx-gen
          NGINX_PROXY_CONTAINER: docker-compose-ingress

    networks:
      default:
        external:
          name: webproxy

nginx-proxy-protocol.tmpl

    curl -o nginx-proxy-protocol.tmpl https://raw.githubusercontent.com/jwilder/docker-gen/master/templates/nginx.tmpl

In the template add a proxy-protocol to all listen directives.

    -listen ...;

    +listen ... proxy_protocol;

Also change X-Real-IP and X-Forwarded-For headers:

    -proxy_set_header X-Real-IP $remote_addr;

    +proxy_set_header X-Real-IP $proxy_protocol_addr;

    -proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    +proxy_set_header X-Forwarded-For $proxy_protocol_addr;

Run ingress-like docker-compose configuration:

    docker-compose -f docker-compose.yaml up -d

For testing purposes let’s run hello-world container, that shows environment variables to get the confidence that ClientIP is correct. Do not forget to change domain and email.

    version: '3'

    services:
      web:
        image: olegsmetanin/dockercloud-hello-world
        environment:
          - VIRTUAL_HOST=cloud.example.com
          - VIRTUAL_NETWORK=webproxy
          - VIRTUAL_PORT=80
          - LETSENCRYPT_HOST=cloud.example.com
          - [LETSENCRYPT_EMAIL=me@example.com](mailto:LETSENCRYPT_EMAIL=me@example.com)
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock
        restart: always

    networks:
      default:
        external:
          name: webproxy

Run docker-compose, open application web page by domain name and make sure that ClientIP matches your real external address, which means that the proxy-protocol actually works.

    docker-compose -f docker-compose.yaml up
