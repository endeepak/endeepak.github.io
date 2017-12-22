---
layout: post
title: "Docker swarm in production"
date: 2017-12-22 09:58:33 +0530
comments: true
categories: devops docker
---

This is a experience report on using docker swarm(17.06) in production for ~3 months

### Context

Prior to this, we were deploying services on VMs using ansible. For a new project, we wanted to explore benefits of running service as containers and decided to use docker swarm due to its simpler setup and consistency with docker engine APIs

<!-- More -->

### Benefits

Using container orchestration engine to run services provided following benefits

* Cleaner boundary between Developers building/packaging the services and DevOps team providing platform for running these services. It is a great cultural shift for the organization!
* Consistent way to operate services - start/stop, find logs, scale. This makes it easier to operate
* Implementing cross cutting concerns like log aggregation, monitoring gets easier due to consistency
* Promotes stateless & immutable infrastructure. It is easier to setup new environments & tear down as and when needed
* You can achieve better resource utilization with same level of isolation compared to running services on VMs. Helps overcoming resource management issues when running services in public cloud VMs - which doesn't allow fine gained control on customizing memory, CPU and network resources for a service

Docker swarm specific

* Easy to setup & operate - single binary/service has all the bells and whistles
* Smaller learning curve due to API consistency with the standalone docker engine
* Built in [service discovery](https://docs.docker.com/engine/swarm/networking/#configure-service-discovery)(via DNS)
* Built in [load balancing](https://docs.docker.com/engine/swarm/ingress/) across service replicas using battle tested Linux services like IPVS. Load balancing is based on health of the instance determined by [health checks](https://docs.docker.com/compose/compose-file/#healthcheck) defined for the service
* Support for distributed [configuration](https://docs.docker.com/engine/swarm/configs/) and [secrets](https://docs.docker.com/engine/swarm/secrets/)
* Support for [rolling upgrades](https://docs.docker.com/compose/compose-file/#update_config)

### Challenges

* Sometimes you'll run into issues [moby#34163](https://github.com/moby/moby/issues/34163), [moby#25432](https://github.com/moby/moby/issues/25432) during deployments. Having a [play-book](https://github.com/project-sunbird/sunbird-devops/wiki/Docker-swarm-troubleshooting) will help in faster recovery from these failures
* Issues in overlay networking([swarm#2161](https://github.com/docker/swarm/issues/2161)) can make few containers unreachable some times. Having good monitoring is essential for identification and remediation of these issues
* Rolling update is harder to achieve if you are using docker config/secret. Swarm doesn't support updating config/secret. You would have create new config(with new version number) & update the service to use new version of config. It would have been good if this was handled internally using checksum of the config

### Conclusion

Overall, using a container orchestration engine in production has proven to be very productive and useful. Docker swarm is maturing over time, with improved stability it is a promising platform for running containers in production
