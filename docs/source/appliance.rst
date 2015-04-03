.. _appliance:

The Service VM (the Akanda Appliance)
=====================================

Akanda uses Linux-based images (stored in OpenStack Glance) to provide layer
3 routing and advanced networking services.  Akanda, Inc provides stable image
releases for download at `akanda.io <http://akanda.io>`_, but it's also
possible to build your own custom Service VM image (running additional
services of your own on top of the routing and other default services provided
by Akanda).

.. _appliance_rest:

REST API
--------
The Akanda Appliance REST API is used by the :ref:`rug` service to manage
health and configuration of services on the router.

Router Health
+++++++++++++

``HTTP GET /v1/status/``
~~~~~~~~~~~~~~~~~~~~~~~~

Used to confirm that a router is responsive and has external network connectivity.

::

    Example HTTP 200 Response

    Content-Type: application/json
    {
        'v4': true,
        'v6': false,
    }

Router Configuration
++++++++++++++++++++

``HTTP GET /v1/firewall/rules/``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Used to retrieve an overview of configured firewall rules for the router (from
``iptables -L`` and ``iptables6 -L``).

::

    Example HTTP 200 Response

    Content-Type: text/plain
    Chain INPUT (policy DROP)
    target     prot opt source               destination
    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0            icmptype 8

    ...


``HTTP GET /v1/system/interface/<ifname>/``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Used to retrieve JSON data about a specific interface on the router.

::

    Example HTTP 200 Response

    Content-Type: application/json
    {
        "interface": {
            "addresses": [
                "8.8.8.8",
                "2001:4860:4860::8888",
            ],
            "description": "",
            "groups": [],
            "ifname": "ge0",
            "lladdr": "fa:16:3f:de:21:e9",
            "media": null,
            "mtu": 1500,
            "state": "up"
        }
    }

``HTTP GET /v1/system/interfaces``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Used to retrieve JSON data about a `every` interface on the router.

::

    Example HTTP 200 Response

    Content-Type: application/json
    {
        "interfaces": [{
            "addresses": [
                "8.8.8.8",
                "2001:4860:4860::8888",
            ],
            "description": "",
            "groups": [],
            "ifname": "ge0",
            "lladdr": "fa:16:3f:de:21:e9",
            "media": null,
            "mtu": 1500,
            "state": "up"
        }, {
            ...
        }]
    }

``HTTP PUT /v1/system/config/``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Used (generally, by :program:`akanda-rug-service`) to push a new configuration
to the router and restart services as necessary:

::

    Example HTTP PUT Body

    Content-Type: application/json
    {
        "configuration": {
            "networks": [
                {
                    "address_allocations": [],
                    "interface": {
                        "addresses": [
                            "8.8.8.8",
                            "2001:4860:4860::8888"
                        ],
                        "description": "",
                        "groups": [],
                        "ifname": "ge1",
                        "lladdr": null,
                        "media": null,
                        "mtu": 1500,
                        "state": "up"
                    },
                    "name": "",
                    "network_id": "f0f8c937-9fb7-4a58-b83f-57e9515e36cb",
                    "network_type": "external",
                    "v4_conf_service": "static",
                    "v6_conf_service": "static"
                },
                {
                    "address_allocations": [],
                    "interface": {
                        "addresses": [
                            "..."
                        ],
                        "description": "",
                        "groups": [],
                        "ifname": "ge0",
                        "lladdr": "fa:16:f8:90:32:e3",
                        "media": null,
                        "mtu": 1500,
                        "state": "up"
                    },
                    "name": "",
                    "network_id": "15016de1-494b-4c65-97fb-475b40acf7e1",
                    "network_type": "management",
                    "v4_conf_service": "static",
                    "v6_conf_service": "static"
                },
                {
                    "address_allocations": [
                        {
                            "device_id": "7c400585-1743-42ca-a2a3-6b30dd34f83b",
                            "hostname": "10-10-10-1.local",
                            "ip_addresses": {
                                "10.10.10.1": true,
                                "2607:f298:6050:f0ff::1": false
                            },
                            "mac_address": "fa:16:4d:c3:95:81"
                        }
                    ],
                    "interface": {
                        "addresses": [
                            "10.10.10.1/24",
                            "2607:f298:6050:f0ff::1/64"
                        ],
                        "description": "",
                        "groups": [],
                        "ifname": "ge2",
                        "lladdr": null,
                        "media": null,
                        "mtu": 1500,
                        "state": "up"
                    },
                    "name": "",
                    "network_id": "31a242a0-95aa-49cd-b2db-cc00f33dfe88",
                    "network_type": "internal",
                    "v4_conf_service": "static",
                    "v6_conf_service": "static"
                }
            ],
            "static_routes": []
        }
    }

Survey of Software and Services
-------------------------------
The Akanda Appliance uses a variety of software and services to manage routing
and advanced services, such as:

    * ``iproute2`` tools (e.g., ``ip neigh``, ``ip addr``, ``ip route``, etc...)
    * ``dnsmasq``
    * ``bird6``
    * ``iptables`` and ``iptables6``

In addition, the Akanda Appliance includes two Python-based services:

    * The REST API (which :program:`akanda-rug-service)` communicates with to
      orchestrate router updates), deployed behind `gunicorn
      <http://gunicorn.org>`_.
    * A Python-based metadata proxy.

Proxying Instance Metadata
--------------------------

When OpenStack VMs boot with ``cloud-init``, they look for metadata on a
well-known address, ``169.254.169.254``.  To facilitate this process, Akanda
sets up a special NAT rule (one for each local network)::

    -A PREROUTING -i eth2 -d 169.254.169.254 -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.10.10.1:9602

...and a special rule to allow metadata requests to pass across the management
network (where OpenStack Nova is running, and will answer requests)::

    -A INPUT -i !eth0 -d <management-v6-address-of-router> -j DROP

A Python-based metadata proxy runs locally on the router (in this example,
listening on ``http://10.10.10.1:9602``) and proxies these metadata requests
over the management network so that instances on local tenant networks will
have access to server metadata.
