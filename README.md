# Akanda
[akanda.io](https://akanda.io)

Akanda is the only open source network virtualization solution built by
OpenStack operators for real OpenStack clouds. Originally developed by
[DreamHost](http://dreamhost.com) for their OpenStack-based public cloud,
[DreamCompute](http://dreamhost.com/cloud/dreamcompute), Akanda eliminates the
need for complex SDN controllers, overlays, and multiple plugins by providing
a simple integrated networking stack (routing, firewall, and load balancing via
a virtual software router) for connecting and securing multi-tenant OpenStack
environments.

----

## About

### Subprojects

The code for the Akanda project lives in several separate repositories to ease
packaging and management:

  * [Akanda Rug](https://github.com/akanda/akanda-rug) – Orchestration
    service for managing the creation, configuration, and health of Akanda
    Software Routers in an OpenStack cloud.

  * [Akanda Appliance](https://github.com/akanda/akanda-appliance) –
    Supporting software for the Akanda Software Router appliance, which is
    a Linux-based service VM that provides routing and L3+ services in
    a virtualized network environment. This includes a REST API for managing
    the appliance via the `Akanda Rug` orchestration service.

  * [Akanda Neutron](https://github.com/akanda/akanda-neutron) – 
    Ancillary subclasses of several OpenStack Neutron plugins and supporting code.

  * [Akanda Nova](https://github.com/dreamhost/akanda-nova) – Extensions to
    OpenStack Nova supporting the creation and management of Akanda Software
    Router appliances.

As such, *this* repository focuses on project overview and documentation.

### The Name

Project names are a powerful tool because they can be used to bond teams,
communicate effectively and convey the end goal. Like many projects, we
considered many names until a member of our team sought out to find a word
appropriate for an open project. This word enables us to say something more
clearly and with a bevy of excellent synonyms by using the Sanskrit word
अखण्ड (akhaNDa) which has such lovely connotations as "non-stop, "undivided,
"entire," "whole," and most importantly, "**not broken**."

## Get Involved

Additional details and documentation on Akanda can be found at
[akanda.io](http://akanda.io) and  [docs.akanda.io](http://docs.akanda.io).

Most Akanda interaction is done via the #akanda channel on
[FreeNode](http://freenode.net) IRC.

## License and Copyright

Akanda is licensed under the Apache-2.0 license and is Copyright 2015,
Akanda, Inc.
