.. _install-openstack:

3. Install OpenStack
====================

Now that we’ve installed and configured MAAS and successfully deployed a Juju controller, it’s time to do some real work; use Juju to deploy `OpenStack <https://www.openstack.org>`_, the leading open cloud platform.

We have two options when installing OpenStack:

1. Install and configure each OpenStack component separately. Adding Ceph, Compute, Swift, RabbitMQ, Keystone and Neutron in this way allows you to see exactly what Juju and MAAS are doing, and consequently, gives you a better understanding of the underlying OpenStack deployment.
2. Use a `bundle <https://docs.jujucharms.com/2.4/en/charms-bundles>`_ to deploy OpenStack with a single command. A bundle is an encapsulation of a working deployment, including all configuration, resources and references. It allows you to effortlessly recreate a deployment with a single command or share that deployment with other Juju users.

If this is your first foray into MAAS, Juju and OpenStack territory, we’d recommend starting with the first option. This will give you a stronger foundation for maintaining and expanding the default deployment. Our instructions for this option continue below.
Alternatively, jump to `Deploying OpenStack as a bundle <https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/rocky/install-openstack-bundle.html>`_ to learn about deploying as a bundle.



.. _openstack-juju-controller:

3.1. Deploy the Juju controller
-------------------------------

:ref:`Previously <install-juju>`, we tested our MAAS and Juju configuration by deploying a new Juju controller called **maas-controller**. You can check this controller is still operational by typing **juju status**. With the Juju controller running, the output will look similar to the following:

.. code::
	
	Model    Controller       Cloud/Region  Version  SLA          Timestamp
   
        default  maas-controller  mymaas        2.4.4    unsupported  15:04:54+03:00



If you need to remove and redeploy the controller, use the following two commands:

.. code::
	
	juju kill-controller maas-controller
	juju bootstrap --constraints tags=juju mymaas maas-controller


During the bootstrap process, Juju will create a model called **default**, as shown in the output from **juju status** above. `Models <https://docs.jujucharms.com/2.4/en/models>`_ act as containers for applications, and Juju’s default model is great for experimentation.

We are going to create a new model called **uos** to hold our OpenStack deployment exclusively, making the entire deployment easier to manage and maintain.

To create a model called **uos** (and switch to it), simply type the following:

.. code::
	
	juju add-model test

After you add a model you can browse the Web UI in Juju using command in your terminal:

.. code::

	juju gui
   
   
 Sled vuvejdaneto na komandata juju gui juju-to shte pokaje informaciq  kak da se lognem v syotveti[ model
 Sushto i info za user-a i parolata 
 
.. code::
 
      
    
      GUI 2.14.0 for model "admin/test" is enabled at:
       https://192.168.40.53:17070/gui/u/admin/test
      Your login credential is:
       username: admin
       password: 67d4c5dbbb2c56990c3fdaab1d5a355c
  
 
 
   
 Tova izglejda po sledniq nachin .
   
  
.. figure::
  /images/guimodellogin.png 
  
   
.. _openstack-deploy:
	
3.2. Deploy OpenStack
---------------------

We are now going to step through adding the OpenStack components to the new model. The applications will be installed from the `eniware-org/openstack-bundles repository <https://github.com/eniware-org/openstack-bundles>`_. We’ll be providing the configuration for the charms as a **yaml** file which we include as we deploy it.
When you Clone the repository to your juju machine, go to folder ``stable/openstack-base``

The configuration is held in the file called **bundle.yaml**.
Deployment requires no further configuration than running the following command:

.. note::
   Do not use autocomplete with Tab button

.. code::

	juju deploy bundle.yaml
   
You can see what Juju doing with command in terminal

.. code::

   watch juju status 
   
   
koeto izglejda po sledniq nachin    

.. code::

      Model  Controller       Cloud/Region  Version  SLA          Timestamp
      test   maas-controller  mymaas        2.4.4    unsupported  16:23:02+03:00
      
      App                    Version        Status       Scale  Charm                  Store       Rev  OS      Notes
      ceph-mon                              waiting        2/3  ceph-mon               jujucharms   26  ubuntu
      ceph-osd               13.2.1+dfsg1   blocked          3  ceph-osd               jujucharms  269  ubuntu
      ceph-radosgw                          maintenance      1  ceph-radosgw           jujucharms  259  ubuntu
      cinder                                waiting        0/1  cinder                 jujucharms  273  ubuntu
      cinder-ceph                           waiting          0  cinder-ceph            jujucharms  234  ubuntu
      glance                                waiting        0/1  glance                 jujucharms  268  ubuntu
      keystone                              maintenance      1  keystone               jujucharms  283  ubuntu
      mysql                  5.7.20-29.24   active           1  percona-cluster        jujucharms  269  ubuntu
      neutron-api                           maintenance      1  neutron-api            jujucharms  262  ubuntu
      neutron-gateway        13.0.1         waiting          1  neutron-gateway        jujucharms  253  ubuntu
      neutron-openvswitch    13.0.1         waiting          3  neutron-openvswitch    jujucharms  251  ubuntu
      nova-cloud-controller                 waiting        0/1  nova-cloud-controller  jujucharms  311  ubuntu
      nova-compute           18.0.1         waiting          3  nova-compute           jujucharms  287  ubuntu
      ntp                    4.2.8p10+dfsg  maintenance      4  ntp                    jujucharms   27  ubuntu
      openstack-dashboard                   maintenance      1  openstack-dashboard    jujucharms  266  ubuntu
      rabbitmq-server        3.6.10         active           1  rabbitmq-server        jujucharms   78  ubuntu
      
      Unit                      Workload     Agent       Machine  Public address  Ports     Message
      ceph-mon/0                maintenance  executing   1/lxd/0  192.168.40.110            (install) installing charm software
      ceph-mon/1                waiting      allocating  2/lxd/0                            waiting for machine
      ceph-mon/2*               maintenance  executing   3/lxd/0  192.168.40.105            (install) installing charm software
      ceph-osd/0*               waiting      idle        1        192.168.40.58             Incomplete relation: monitor
      ceph-osd/1                blocked      idle        2        192.168.40.59             Missing relation: monitor
      ceph-osd/2                waiting      idle        3        192.168.40.101            Incomplete relation: monitor
      ceph-radosgw/0*           maintenance  executing   0/lxd/0  192.168.40.103            (install) Installing radosgw packages
      cinder/0                  waiting      allocating  1/lxd/1                            waiting for machine
      glance/0                  waiting      allocating  2/lxd/1                            waiting for machine
      keystone/0*               maintenance  executing   3/lxd/1  192.168.40.109            (install) installing charm software
      mysql/0*                  active       idle        0/lxd/1  192.168.40.102  3306/tcp  Unit is ready
      neutron-api/0*            maintenance  executing   1/lxd/2  192.168.40.108            (install) installing charm software
      neutron-gateway/0*        waiting      idle        0        192.168.40.57             Incomplete relations: network-service, messaging
        ntp/0*                  active       idle                 192.168.40.57   123/udp   Ready
      nova-cloud-controller/0   waiting      allocating  2/lxd/2                            waiting for machine
      nova-compute/0*           waiting      idle        1        192.168.40.58             Incomplete relations: image, messaging, storage-backend
        neutron-openvswitch/0*  waiting      idle                 192.168.40.58             Incomplete relations: messaging
        ntp/1                   active       idle                 192.168.40.58   123/udp   Ready
      nova-compute/1            waiting      executing   2        192.168.40.59             Incomplete relations: messaging, storage-backend, image
        neutron-openvswitch/2   maintenance  executing            192.168.40.59             (install) Installing apt packages
        ntp/3                   maintenance  executing            192.168.40.59             (install) installing charm software
      nova-compute/2            waiting      executing   3        192.168.40.101            Incomplete relations: messaging, image, storage-backend
        neutron-openvswitch/1   maintenance  executing            192.168.40.101            (install) Installing apt packages
        ntp/2                   maintenance  executing            192.168.40.101            (install) installing charm software
      openstack-dashboard/0*    maintenance  executing   3/lxd/2  192.168.40.106            (install) installing charm software
      rabbitmq-server/0*        active       executing   0/lxd/2  192.168.40.104            (config-changed) Enabling queue mirroring
      
      Machine  State    DNS             Inst id              Series  AZ       Message
      0        started  192.168.40.57   skyhk8               bionic  default  Deployed
      0/lxd/0  started  192.168.40.103  juju-4052d2-0-lxd-0  bionic  default  Container started
      0/lxd/1  started  192.168.40.102  juju-4052d2-0-lxd-1  bionic  default  Container started
      0/lxd/2  started  192.168.40.104  juju-4052d2-0-lxd-2  bionic  default  Container started
      1        started  192.168.40.58   t678hy               bionic  default  Deployed
      1/lxd/0  started  192.168.40.110  juju-4052d2-1-lxd-0  bionic  default  Container started
      1/lxd/1  pending                  juju-4052d2-1-lxd-1  bionic  default  Container started
      1/lxd/2  started  192.168.40.108  juju-4052d2-1-lxd-2  bionic  default  Container started
      2        started  192.168.40.59   dsktqg               bionic  default  Deployed
        
  
  
  


	
The deployed **yaml** file includes the following applications:

.. list-table::
    :header-rows: 0
    :stub-columns: 0

    * - * `Openstack dashboard <https://jujucharms.com/openstack-dashboard/>`_ - it provides a Django based web interface for use by both administrators and users of an OpenStack Cloud. It allows you to manage Nova, Glance, Cinder and Neutron resources within the cloud.
    * - * `Keystone <https://jujucharms.com/keystone/>`_ - this charm provides Keystone, the OpenStack identity service. Its target platform is (ideally) Ubuntu LTS + OpenStack.
    * - * `Glance <https://jujucharms.com/glance/>`_ - The Glance project provides an image registration and discovery service and an image delivery service. These services are used in conjunction by **Nova** to deliver images from object stores, such as OpenStack's Swift service, to Nova's compute nodes.
    * - * `MySQL <https://jujucharms.com/percona-cluster/>`_ - Percona XtraDB Cluster is a high availability and high scalability solution for MySQL clustering. Percona XtraDB Cluster integrates Percona Server with the Galera library of MySQL high availability solutions in a single product package which enables you to create a cost-effective MySQL cluster. This charm deploys Percona XtraDB Cluster onto Ubuntu.
    * - * `Cinder <https://jujucharms.com/cinder/>`_ - Cinder is the block storage service for the OpenStack. This charm provides the Cinder volume service for OpenStack. It is intended to be used alongside the other OpenStack components. Cinder is made up of 3 separate services: an API service, a scheduler and a volume service. This charm allows them to be deployed in different combination, depending on user preference and requirements.
    * - * `Cinder Ceph <https://jujucharms.com/cinder-ceph/>`_ - This charm provides a Ceph storage backend for **Cinder** charm. This allows multiple Ceph storage clusters to be associated with a single Cinder deployment, potentially alongside other storage backends from other vendors.
    * - * `RabbitMQ <https://jujucharms.com/rabbitmq-server/>`_ - RabbitMQ is an implementation of AMQP, the emerging standard for high performance enterprise messaging. The RabbitMQ server is a robust and scalable implementation of an AMQP broker. This charm deploys RabbitMQ server and provides AMQP connectivity to clients.
    * - * `Nova Compute <https://jujucharms.com/nova-compute/>`_ - this charm is a cloud computing fabric controller which provides the OpenStack compute service. This charm provides the Nova Compute hypervisor service and should be deployed directly to physical servers. Its target platform is Ubuntu (preferably LTS) + OpenStack. 
    * - * `Ceph OSD <https://jujucharms.com/ceph-osd/>`_ - Ceph is a distributed storage and network file system designed to provide excellent performance, reliability, and scalability. This charm deploys additional Ceph OSD storage service units and should be used in conjunction with the **Ceph-mon** charm to scale out the amount of storage available in a Ceph cluster.
    * - * `Ceph Mon <https://jujucharms.com/ceph-mon/>`_ - This charm deploys a Ceph monitor cluster.
    * - * `Ceph Radosgw <https://jujucharms.com/ceph-radosgw/>`_ - This charm provides the RADOS HTTP gateway supporting S3 and Swift protocols for object storage.
    * - * `Neutron API <https://jujucharms.com/neutron-api/>`_ - Neutron is a virtual network service for OpenStack. Neutron provides an API to dynamically request and configure virtual networks. These networks connect "interfaces" from other OpenStack services (e.g., virtual NICs from Nova VMs). The Neutron API supports extensions to provide advanced network capabilities (e.g., QoS, ACLs, network monitoring, etc.). This principle charm provides the OpenStack Neutron API service which was previously provided by the **Nova-cloud-controller** charm. When this charm is related to the Nova-cloud-controller charm the Nova-cloud controller charm will shutdown its api service, de-register it from Keystone and inform the compute nodes of the new Neutron url.
    * - * `Nova Cloud Controller <https://jujucharms.com/nova-cloud-controller/>`_ - OpenStack Compute, codenamed Nova, is a cloud computing fabric controller. This charm provides the cloud controller service for OpenStack Nova and includes **nova-scheduler**, **nova-api** and **nova-conductor** services.
    * - * `Neutron OpenvSwitch <https://jujucharms.com/neutron-openvswitch/>`_ - This charm provides the OpenStack Neutron Open vSwitch agent, managing L2 connectivity on **nova-compute** services. This subordinate charm provides the Neutron OpenvSwitch configuration for a compute node. Once deployed it takes over the management of the Neutron base and plugin configuration on the compute node.
    * - * `Neutron Gateway <https://jujucharms.com/neutron-gateway>`_ - This charm provides central **Neutron networking** services as part of a Neutron based OpenStack deployment.
    * - * `NTP <https://jujucharms.com/ntp/>`_ - NTP, the Network Time Protocol, provides network based time services to ensure synchronization of time across computers. This charm can be deployed alongside principal charms to enable NTP management across deployed services.

.. note::
   Remember, you can check on the status of a deployment using the ``juju status`` command. To see the status of a single charm of application, append the charm name. For example, for a Ceph OSD charm:
   
   .. code::
       
      juju status ceph-osd	

	  
	

.. _openstack-test:
	
3.3. Test OpenStack
-------------------

After everything has deployed and the output of **juju status** settles, you can check to make sure OpenStack is working by logging into the Horizon dashboard.

The quickest way to get the IP address for the dashboard is with the following command:

.. code::
	
	juju status --format=yaml openstack-dashboard | grep public-address | awk '{print $2}'

The URL will be **http://<IP ADDRESS>/horizon**. When you enter this into your browser you can login with ``admin`` and ``openstack``, unless you changed the password in the configuration file.

If everything works, you will see something similar to the following:

.. _install-openstack-horizon:

.. figure:: /images/3-install-openstack_horizon.png
   :alt: Horizon dashboard
   
	
3.4. Next steps
---------------

Congratulations, you’ve successfully deployed a working OpenStack environment using both Juju and MAAS. The next step is to configure OpenStack for use within a production environment.