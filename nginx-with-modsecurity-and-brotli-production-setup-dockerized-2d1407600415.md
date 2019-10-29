Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m356[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m156[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m50[39m, end: [33m94[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m66[39m }

# NGINX With ModSecurity and Brotli: Production setup (Dockerized)



**Have you ever heard of Load Balancer? Reverse Proxy?
** If you’ve ever heard the term ReverseProxy or Load Balancer being thrown around and wondered to yourself what that term meant or how to use NGINX as Reverse Proxy, this article aims to alleviate those concerns and develop a basic understanding. Setup a production grade Customized NGINX Docker Image with ModSecurity and Google’s, Brotli Lossless file compression.

**NGINX (Engine-x) ?
** Load balancing is an excellent way to scale out your application and increase its performance and redundancy. Nginx, which is a popular web server software, can be configured as a simple yet powerful load balancer to improve your servers resource availability and efficiency. Nginx is one of a handful of servers written to address the [C10K problem](https://en.wikipedia.org/wiki/C10k_problem). Unlike traditional servers, Nginx doesn’t rely on threads to handle requests. Instead, it uses a much more scalable event-driven (asynchronous) architecture. In a load balancing configuration, NGINX acts as a single entry point to a distributed web application working on multiple separate servers. In a Microservice Architecture, NGINX helps serve services running on multiple servers.

In My organisation, I was asked to serve Multiple Web Applications (SPA’s) and Backend API’s using only LoadBalancer/ReverseProxy being exposed to internet. We are running light weight services over a dockerized environment with docker-compose and hence we chose NGINX as a reverse proxy and used ModSecurity to handle 97% of known security vulnerabilities.

![](https://cdn-images-1.medium.com/max/2016/1*8BdsAoe7fnD5bSOcz3MIeg.jpeg)

Since we are into Microservice Environment, we have developed an API Gateway for all the internal microservices communication through docker bridge network and exposed Gateway to NGINX. We are going to eliminate API Gateway and include the reverse proxy conf with NGINX itself to route each requests from client to respective services, and of course each request from client will be authenticated using Authentication service using OAuth. In this way we can scale the backend services with replicas on a same host or on multiple hosts and NGINX will be configured in a way that it can route the requests to services running on hosts based on the number of requests.

**ModSecurity
***“Web applications — yours, mine, everyone’s — are terribly insecure on average. We struggle to keep up with the security issues and need any help we can get to secure them.” *– Ivan Ristić, creator of ModSecurity

OSS version of NGINX does not handle security issues and it’s included as part of NGINX plus. Since [NGINX WAF](https://www.nginx.com/products/nginx-waf/) uses open sourced ModSecurity to handle most of the known security breaches and vulnerabilities, We have built a customized Docker image to include ModSecurity.
[ *ModSecurity is an open source, cross-platform web application firewall (WAF) module. Known as the “Swiss Army Knife” of WAFs, it enables web application defenders to gain visibility into HTTP(S) traffic and provides a power rules language and API to implement advanced protections.](http://www.modsecurity.org/about.html)*

**Google/Brotli
**All the Web browsers support file encoding and for several years gzip had become the primary encoding type until google introduced another lossless compression format which is supported by most of the modern web browsers.
**Brotli **is a generic-purpose lossless compression algorithm that compresses data using a combination of a modern variant of the [LZ77 algorithm](https://en.wikipedia.org/wiki/LZ77_and_LZ78), Huffman coding and 2nd order context modeling, with a compression ratio comparable to the best currently available general-purpose compression methods. It is similar in speed with deflate but offers more dense compression. The specification of the Brotli Compressed Data Format is defined in [RFC 7932](https://tools.ietf.org/html/rfc7932). 
Brotli is open-sourced under the MIT License.

**As part of production Setup and Performance improvements we have integrated following changes into nginx conf.**

1. All the API’s req/res are secured through **IAM **— Authorization and authentication.

1. NGINX as reverse proxy and load balancer for all the requests

1. Protocol change from **HTTP/1.1 to HTTP/2**

1. **Encoding compression algorithm**: upgradation from **gzip **to **Brotli **(introduced and used in Google’s infrastructure)** **reduced the content compression over the network for **80%** and above compression (ie. 10MB over the network compressible to ~1.5MB)

1. **Browser Cache: **Cache the static files completely in browser for 365 days so the UI will be loading from browser’s in-memory/disk cache.

1. **Rate Limiting: **We can rate limit the API’s req/res to stop **DDOS **attack which makes our server’s to block such requests and improve **HA **(High availability). We are allowing 30 req/res for unique client IP’s.

1. Content type limiting

1. Security headers to avoid cross platform attacks through scripts (X-Frame-Options, X-XSS-Protection, X-Content-Type-Options, Referrer-Policy, Content-Security-Policy, Strict-Transport-Security)

1. Strict SSL Policies

1. Efficient use of Buffers

1. **ModSecurity** to handle 97% of Known Cyber attacks, suspicious requests, cross origin requests, anomaly detection etc..,

1. HTTP request timeouts to handle stalled requests

1. Header Content size limitation for handling bulk upload attacks

1. Black List and Whitelisting known and unknown source requests

1. Control Allowed Request headers

1. Improve TTFB — TIME to FIRST BYTE for serving static files by caching files into temporary folder */tmp.*

**NGINX Load Balancer configuration**

    http {
        upstream backend {
            least_conn
            server srv1.example.com;
            server srv2.example.com;
            server srv3.example.com;
            server 10.0.0.123;
            server 192.168.0.3;
        }

    server {
            listen 80;

    location / {
                proxy_pass [http://backend](http://backend);
            }
        }
    }

NGINX By default follows Round-Robin algorithm to route the requests to configured upstream servers unless we explicitly mention one of the following load balancing conf:
**Least connected load balancing: **With the least-connected load balancing, nginx will try not to overload a busy application server with excessive requests, distributing the new requests to a less busy server instead.
**Session persistence: **If there is the need to tie a client to a particular application server — in other words, make the client’s session “sticky” or “persistent” in terms of always trying to select a particular server — the ip-hash load balancing mechanism can be used.
**Weighted load balancing: **When the [weight](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) parameter is specified for a server, the weight is accounted as part of the load balancing decision.

## **Hardening Security**
> **#1 Nginx Configuration path and port
**The nginx server configuration directory and /usr/local/nginx/conf/nginx.conf is main configuration file.
**/usr/local/nginx/html/** or **/var/www/html**– The default document location.
**/usr/local/nginx/logs/** or **/var/log/nginx** — The default log file location.
Nginx **HTTP default port** : TCP 80
Nginx **HTTPS default port** : TCP 443
> **#2 Turn On SELinux
**Security-Enhanced Linux (SELinux) is a Linux kernel feature that provides a mechanism for supporting access control security policies which provides great protection. It can stop many attacks before your system rooted.
> **#3 Use mod_security
** mod_security provides an application level firewall and stops many injection attacks
> **#4 Controlling Buffer Overflow Attack**
client_body_buffer_size 10K;
client_header_buffer_size 1k;
client_max_body_size 8m;
large_client_header_buffers 4 32k;
> **#5 Control Simultaneous Connections
**limit_zone slimits $binary_remote_addr 5m;
limit_conn slimits 5;
> **#6 Limit Available Methods**
> **#7 Limiting Access By Ip Address**
> **#8 Configure strict SSL**
> **#9 Avoid clickjacking
** add_header X-Content-Type-Options nosniff;
> #**10 Enable the Cross-site scripting (XSS) filter
** add_header X-XSS-Protection “1; mode=block”;

## Creating Docker Image to include ModSecurity:3.0.0 and Brotli

<iframe src="https://medium.com/media/187c24dec4eba5247de56f2757f1da6d" frameborder=0></iframe>

## **NGINX Production Setup**

<iframe src="https://medium.com/media/a8bd4bb05751ce9892e3201df8b94ae6" frameborder=0></iframe>
> Github Link: 
[https://github.com/vijay-nallagatla/nginx_production_config](https://github.com/vijay-nallagatla/nginx_production_config)
> NGINX Docker Image: 
docker pull vijaynallagatla/nginx_modsecurity
