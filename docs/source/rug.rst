.. _rug:

Service VM Orchestration and Management
=======================================

RUG - Router Update Generator
-----------------------------

:program:`akanda-rug-service` is a multiprocessed, multithreaded Python process
composed of three primary subsystems, each of which are spawned as a subprocess
of the main :py:mod:`akanda.rug` process:

L3 and DHCP Event Consumption
-----------------------------

:py:mod:`akanda.rug.notifications` uses `kombu <https://pypi.python.org/pypi/kombu>`_
and a Python :py:mod:`multiprocessing.Queue` to listen for specific Neutron service
events (e.g., ``router.interface.create``, ``subnet.create.end``,
``port.create.end``, ``port.delete.end``) and normalize them into one of
several event types:

    * ``CREATE`` - a router creation was requested
    * ``UPDATE`` - services on a router need to be reconfigured
    * ``DELETE`` - a router was deleted
    * ``POLL`` - used by the :ref:`health monitor<health>` for checking aliveness
      of a Service VM
    * ``REBUILD`` - a Service VM should be destroyed and recreated

As events are normalized and shuttled onto the :py:mod:`multiprocessing.Queue`,
:py:mod:`akanda.rug.scheduler` shards (by Tenant ID, by default) and
distributes them amongst a pool of worker processes it manages.

This system also consumes and distributes special :py:mod:`akanda.rug.command` events
which are published by the :program:`rug-ctl` :ref:`operator tools<operator_tools>`.


State Machine Workers and Router Lifecycle
------------------------------------------
Each multithreaded worker process manages a pool of state machines (one
per virtual router), each of which represents the lifecycle of an individual
router.  As the scheduler distributes events for a specific router, logic in
the worker (dependent on the router's current state) determines which action to
take next:

.. graphviz:: worker_diagram.dot

For example, let's say a user created a new Neutron network, subnet, and router.
In this scenario, a ``router-interface-create`` event would be handled by the
appropriate worker (based by tenant ID), and a transition through the state
machine might look something like this:

.. graphviz:: sample_boot.dot

State Machine Flow
++++++++++++++++++

The supported states in the state machine are:

    :CalcAction: The entry point of the state machine.  Depending on the
        current status of the Service VM (e.g., ``ACTIVE``, ``BUILD``, ``SHUTDOWN``)
        and the current event, determine the first step in the state machine to
        transition to.

    :Alive: Check aliveness of the Service VM by attempting to communicate with
        it via its REST HTTP API.
    
    :CreateVM: Call ``nova boot`` to boot a new Service VM.  This will attempt
        to boot a Service VM up to a (configurable) number of times before
        placing the router into ``ERROR`` state.
    
    :CheckBoot: Check aliveness (up to a configurable number of seconds) of the
        router until the VM is responsive and ready for initial configuration.
    
    :ConfigureVM: Configure the Service VM and its services.  This is generally
        the final step in the process of booting and configuring a router.  This
        step communicates with the Neutron API to generate a comprehensive network
        configuration for the router (which is pushed to the router via its REST
        API).  On success, the state machine yields control back to the worker
        thread and that thread handles the next event in its queue (likely for
        a different Service VM and its state machine).
    
    :ReplugVM: Attempt to hot-plug/unplug a network from the router via ``nova
        interface-attach`` or ``nova-interface-detach``.

    :StopVM: Terminate a running Service VM.  This is generally performed when
        a Neutron router is deleted or via explicit operator tools.

    :ClearError: After a (configurable) number of ``nova boot`` failures, Neutron
        routers are automatically transitioned into a cooldown ``ERROR`` state
        (so that :py:mod:`akanda.rug` will not continue to boot them forever; this is
        to prevent further exasperation of failing hypervisors).   This state
        transition is utilized to add routers back into management after issues
        are resolved and signal to :py:mod:`akanda-rug` that it should attempt
        to manage them again.
    

.. _health:

Health Monitoring
-----------------

``akanda.rug.health`` is a subprocess which (at a configurable interval)
periodically delivers ``POLL`` events to every known virtual router.  This
event transitions the state machine into the ``Alive`` state, which (depending
on the availability of the router), may simply exit the state machine (because
the router's status API replies with an ``HTTP 200``) or transition to the
``CreateVM`` state (because the router is unresponsive and must be recreated).
