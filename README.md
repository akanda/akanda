# Akanda

A set of Layer 3 plus Services for OpenStack.

----

## About

### Subprojects

The code for the Akanda project lives in several separate repositories to ease
packaging and management:

  * [Akanda Appliance](https://github.com/dreamhost/akanda-appliance) –
    Supporting software for the Akanda Software Router appliance, which is a
    service VM running Linux and IPTables for providing L3+ services in a
    virtualized network environment. This includes a REST API for managing the
    appliance.

  * [Akanda Neutron](https://github.com/dreamhost/akanda-quantum) – User-Facing
    REST service implemented as OpenStack Neutron API Extensions. Additionally,
    subclasses of the several Neutron plugins and supporting code.

  * [Akanda Nova](https://github.com/dreamhost/akanda-nova) – Extensions to
    OpenStack Nova supporting the creation and management of Akanda Software
    Router appliances.

  * [Akanda Ceilometer](https://github.com/dreamhost/akanda-ceilometer)
    – Integration with OpenStack Ceilometer for metering of activity inside of
    Akanda Software Routers.
  
  * [Akanda Horizon](https://github.com/dreamhost/akanda-horizon) – OpenStack
    Horizon extensions to enable the management of Akanda Software Routers.

  * [Akanda Rug](https://github.com/dreamhost/akanda-rug) – Orchestration
    service for managing the creation, configuration, and health of Akanda
    Software Routers in an OpenStack cloud.

As such, this repository focuses on project overview and documentation.

### The Name

Project names are a powerful tool because they can be used to bond teams,
communicate effectively and convey the end goal. Like many projects, we
considered many names until a member of our team sought out to find a word
appropriate for an open project. This word enables us to say something more
clearly and with a bevy of excellent synonyms by using the Sanskrit word
अखण्ड (akhaNDa) which has such lovely connotations as "non-stop, "undivided,
"entire," "whole," and most importantly, "**not broken**."

## The Akanda REST APIs

Akanda comes with two REST APIs:

1. The REST API that runs on the router instance itself, recieving simple
   pf-related administrative commands (e.g., "take this data and have pf parse
   it"). This REST API runs only so long a router instance is up and running.
   This is not the user-facing, 24/7 REST API.

2. Then there is the user-facing, 24/7, load-balanced REST API. This is what
   users will be able to interact with in order to programmatically manage their
   router instances (e.g., set NAT, port-forwarding, and basic firewall rules).
   This API is exposed as extensions to OpenStack Neutron's API.

## Additional Documentation

Akanda is in use at [DreamHost](http://dreamhost.com) for our OpenStack-based
public cloud, [DreamCompute](http://dreamhost.com/cloud/dreamcompute). As we
work on bringing Akanda to the community, we will be working on additional
documentation, user guides, etc.

Mailing lists and a project website are on the way!

## License and Copyright

Akanda is licensed under the Apache-2.0 license and is Copyright 2014,
DreamHost, LLC.
