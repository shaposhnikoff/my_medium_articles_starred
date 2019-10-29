Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m21[39m }

# Traefik and Consul Catalog Example

Traefik and Consul Catalog Example

*Files and build steps can be found here: [https://github.com/mattmcla/traefik-consul](https://github.com/mattmcla/traefik-consul)*

[Traefik](https://traefik.io/) (spelled [Træfik](https://traefik.io/)) is a modern reverse proxy with dynamic configuration informed by service discovery services such as [Consul](https://www.consul.io/). Combined the two can be quite powerful, especially when your container orchestration uses dynamic port assignments (such as Hashicorp’s [Nomad](https://www.nomadproject.io/)) No need to manage ports or restart your reverse proxy for config changes. Traefik even provides a convenient dashboard that visualizes the configuration.

First, you will need to create a Traefik configuration file declaring entry points and to use Consul Catalog to discover backends. Traefik silo’s it’s configuration into three main parts, entry points, front ends, and back ends; for more information refer to the [Traefik documentation](https://docs.traefik.io/basics/).

<iframe src="https://medium.com/media/c07f547ae00f71510f999ae8cfff3279" frameborder=0></iframe>

In this configuration file, we declare our Consul endpoint at 127.0.0.1 which is how I would run it in production (using Dockers host network mode). It is possible to override the configuration file using command line arguments. The prefix attribute is used by Traefik to identify configuration options stored as tags for a given Consul service.

Next, we need a simple Dockerfile that copies our configuration file and exposes port 8080, so we can use the web dashboard (port 80 is exposed by default in the imported Traefik container).

<iframe src="https://medium.com/media/9c6a299d2e6dabb9f16a5f35306157de" frameborder=0></iframe>

Build the container: docker build -t reverse-proxy .

To complete the example stack let’s use a hello world back-end service, and Registrator to register containers with Consul. Registrator is important because it listens to docker events and registers/unregisters services in Consul as containers start and stop; it also looks for specially formatted environment variables assigned to containers and forwards that data to Consul.

<iframe src="https://medium.com/media/5b4e937349b3512eadf6d74491f468fe" frameborder=0></iframe>

Finally, use docker-compose to run the full stack. In the compose file you’ll notice the “web-server” service has an environment variable SERVICE_TAGS. Registrator parses this comma delimited string and passes the values as tags when registering the container with Consul. Through tags, a service can configure its Traefik front end.

traefik.enable=true Enables the Consul service as a valid backend

traefik.frontend.entryPoints=http Identifies which entry point traffic is served through

traefik.frontend.rule=Host:localhost Tells the Traefik front end which host, or hostname, requests should come from (overriding the default in the config file)

At no point do we need to declare ports, IP’s or service names for the back-end configuration; this is all handled by Consul.

Read the Traefik documentation to see [the full list of supported tags](https://docs.traefik.io/configuration/backends/consul/).

Run docker-compose up then visit the following links:

[http://localhost:8500](http://localhost:8500) to see the services registered in Consul. Click on the web-server service to see its tags

[http://localhost:8080](http://localhost:8080) to see the Traefik dashboard. You should see the front end as described in Consul tags as well as the backend

[http://localhost:8081](http://localhost:8081) to see a darling container whale saying hello!

You may add or remove services at will and Traefik will watch for the changes in Consul and update it’s configuration accordingly.

The only downside for me to this approach is what feels like a misappropriation of the Consul service tags. It’s clunky to configure via Registrator and doesn’t fit with Consuls intentions in using tags for DNS routing. You can use Consuls KV as an alternative backend to the Catalog, but this separates the configuration from the service and increases complexity.

In the future, I’m hoping Consul provides more options to store metadata in the Catalog, but in the meantime, this is a small price to pay for dynamic reverse proxy routing that requires no additional services, no extra code, consolidates configuration and reduces complexity.
