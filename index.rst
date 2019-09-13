:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

Known Issues
============

Addresing Is Inconsistent With Summit/Base Production Network
-------------------------------------------------------------

It has been stated that the primary reason for using RFC1918 address space
in the lab environment was to be consistent with the production network
design.  The current summit/base networks primarily use routable IP address
space and it is our understanding that there is no intention to migrate to
largely RFC1918 address space in the near or distant future.

There are many valid reasons to deploy RFC1918 space but its use in a
lab/test environment is not consistent with the production environment.

We also note that LSE-309 appears silent on the issue of RFC1918 address
space.


DNS
---

confused source of authority for ``lsst.org`` domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The internal nameservers at ``140.252.32.21`` and ``140.252.32.45`` believe
they are authoritative for the ``lsst.org`` domain despite route53 being
configured in the global DNS hierarchy as being authoritative.  Due to
this manual "split view" configuration ``A``, ``CNAME``, etc. records
must be manually sync'd between the two views. In addition, these
resolves are returning ``AAAA`` records even though ipv6 is not
intentionally in use.

Internal view:

.. code-block:: yaml

   ~ $ dig +nocomments +nocmd +nostats +authority SOA lsst.org @140.252.32.21
  ;lsst.org.			IN	SOA
  lsst.org.		60	IN	SOA	callisto.lsst.local. hostmaster.lsst.local. 511 900 600 3600 60
  callisto.lsst.local.	60	IN	A	140.252.32.21
  callisto.lsst.local.	60	IN	AAAA	2002:8cfc:2015::8cfc:2015
   ~ $ dig +nocomments +nocmd +nostats +authority SOA lsst.org @140.252.32.45
  ;lsst.org.			IN	SOA
  lsst.org.		60	IN	SOA	algol.lsst.local. hostmaster.lsst.local. 511 900 600 3600 60
  algol.lsst.local.	60	IN	A	140.252.32.45
  algol.lsst.local.	60	IN	AAAA	2002:8cfc:202d::8cfc:202d

External view:

.. code-block:: yaml

   ~ $ dig +nocomments +nocmd +nostats +authority SOA lsst.org @1.0.0.1
  ;lsst.org.			IN	SOA
  lsst.org.		900	IN	SOA	ns-1750.awsdns-26.co.uk. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400
  lsst.org.		3100	IN	NS	ns-1296.awsdns-34.org.
  lsst.org.		3100	IN	NS	ns-635.awsdns-15.net.
  ns-1296.awsdns-34.org.	3100	IN	A	205.251.197.16
   ~ $ dig +nocomments +nocmd +nostats +authority SOA lsst.org @8.8.8.8
  ;lsst.org.			IN	SOA
  lsst.org.		896	IN	SOA	ns-1750.awsdns-26.co.uk. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400
  lsst.org.		3096	IN	NS	ns-635.awsdns-15.net.
  lsst.org.		3096	IN	NS	ns-1296.awsdns-34.org.
  ns-1296.awsdns-34.org.	3096	IN	A	205.251.197.16


confused source of authority for reverse resolution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Similar to the SOA issue for ``lsst.org``, the internal nameservers believe
they are authoritative for reverse DNS.

Internal view:

.. code-block:: yaml

  ~ $ dig +nocomments +nocmd +nostats +authority SOA 32.252.140.in-addr.arpa @140.252.32.21
 ;32.252.140.in-addr.arpa.	IN	SOA
 32.252.140.in-addr.arpa. 3600	IN	SOA	callisto.lsst.local. hostmaster.lsst.local. 22451 900 600 86400 3600
 callisto.lsst.local.	60	IN	A	140.252.32.21
 callisto.lsst.local.	60	IN	AAAA	2002:8cfc:2015::8cfc:2015
  ~ $ dig +nocomments +nocmd +nostats +authority SOA 32.252.140.in-addr.arpa @140.252.32.45
 ;32.252.140.in-addr.arpa.	IN	SOA
 32.252.140.in-addr.arpa. 3600	IN	SOA	algol.lsst.local. hostmaster.lsst.local. 22451 900 600 86400 3600
 algol.lsst.local.	60	IN	A	140.252.32.45
 algol.lsst.local.	60	IN	AAAA	2002:8cfc:202d::8cfc:202d

External view:

.. code-block:: yaml

  ~ $ dig +nocomments +nocmd +nostats +authority SOA 32.252.140.in-addr.arpa @1.0.0.1
 ;32.252.140.in-addr.arpa.	IN	SOA
 32.252.140.in-addr.arpa. 873	IN	SOA	ns-231.awsdns-28.com. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400
 32.252.140.in-addr.arpa. 10481	IN	NS	ns-231.awsdns-28.com.
 32.252.140.in-addr.arpa. 10481	IN	NS	ns-1704.awsdns-21.co.uk.
 32.252.140.in-addr.arpa. 10481	IN	NS	ns-558.awsdns-05.net.
 32.252.140.in-addr.arpa. 10481	IN	NS	ns-1502.awsdns-59.org.
 ns-231.awsdns-28.com.	78809	IN	A	205.251.192.231
 ns-558.awsdns-05.net.	155959	IN	A	205.251.194.46
  ~ $ dig +nocomments +nocmd +nostats +authority SOA 32.252.140.in-addr.arpa @8.8.8.8
 ;32.252.140.in-addr.arpa.	IN	SOA
 32.252.140.in-addr.arpa. 865	IN	SOA	ns-231.awsdns-28.com. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400
 32.252.140.in-addr.arpa. 10473	IN	NS	ns-1704.awsdns-21.co.uk.
 32.252.140.in-addr.arpa. 10473	IN	NS	ns-558.awsdns-05.net.
 32.252.140.in-addr.arpa. 10473	IN	NS	ns-1502.awsdns-59.org.
 32.252.140.in-addr.arpa. 10473	IN	NS	ns-231.awsdns-28.com.
 ns-558.awsdns-05.net.	155951	IN	A	205.251.194.46
 ns-231.awsdns-28.com.	78801	IN	A	205.251.192.231


reverse dns resolution may return invalid or corrupt records
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: yaml

  ~ $ dig +short -x 140.252.32.145 @140.252.32.21
 aver.lsst.local,\032lsst.org.
  ~ $ dig +short -x 140.252.32.145 @140.252.32.45
 aver.lsst.local,\032lsst.org.
  ~ $ dig +short -x 140.252.32.145 @1.0.0.1
  ~ $ dig +short -x 140.252.32.145 @8.8.8.8
  ~ $


no name resolution for rfc1918 subnet(s)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There is no forward or reverse DNS resolution for RFC1918 subnets, other than
the ``test`` domain kludge used in the comcam test environment, used as the
"lab" or "test stand" environment.

Note that this is particularly problematic for web services that use TLS as the
common name of x509 certificates can not be validated by DNS.

Example of reverse resolution failing:

.. code-block:: yaml

   ~ $ dig +short -x 10.0.100.1 @140.252.32.21
   ~ $ dig +short -x 10.0.100.1 @140.252.32.45
   ~ $ dig +nocomments +nocmd +nostats +authority SOA 100.0.10.in-addr.arpa @140.252.32.21
  ;100.0.10.in-addr.arpa.		IN	SOA
   ~ $ dig +nocomments +nocmd +nostats +authority SOA 100.0.10.in-addr.arpa @140.252.32.45
  ;100.0.10.in-addr.arpa.		IN	SOA

This is partially being mitigated by manually updating ``/etc/hosts`` files,
which requires manual synchronization and is probably error prone.

.. code-block:: yaml

   127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
   ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
   10.0.103.101 comcam-fp01.test comcam-fp01
   10.0.103.102 comcam-mcm.test comcam-mcm
   10.0.103.103 comcam-dc01.test comcam-dc01
   10.0.103.104 comcam-hcu01.test comcam-hcu01
   10.0.103.105 comcam-vw01.test comcam-vw01
   10.0.103.106 comcam-db01.test comcam-db01
   10.0.103.107 comcam-hcu02.test comcam-hcu02


No Direct Routing For RFC1918 Subnet(s)
---------------------------------------

Currently, the primary means of accessing hosts in private address space is via
a bastion host named ``stargate.lsst.org``.  This host also acts as a gateway
for all of the current private address subnets and runs NAT.  There is now
routing, without NAT, between the lab subnets and the rest of the network. In
addition, there is no VPN facility or other mechanism supported for tunneling
directly into the private subnets.  This leads to a number of usability issues.

ssh private key exposure on a public host
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Users must preform an extra ssh "hop" through the bastion host for shell
access.  This is only a minor convenience for users with enough technically skill
to use an ``ssh-agent``.  However, many users will likely result to having to
copy ssh private keys onto the bastion host.

ssh port forward is tedious and does not always work
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to attempt to access http/s services, ssh port forwarding for each
service is required. Eg.,

.. code-block:: yaml

   ssh stargate.lsst.org -A -L4430:10.0.103.101:443

This will work for some but not all www applications and it will fail to
provide a usable HTML interface if FQDN URLs and/or ports are embedded in the
HTML or Javascript.  It is also tedious to setup, especially for accessing
multiple end points.

end-users are using remote display from the bastion host (nx)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Due to the limitations and inconvenience of ssh port forwarding, end users are
resorting to remote displaying a web browser or a complete desktop environment.
While this does largely resolve the problem for end-users it may start to put
resource pressure on the bastion host. If end users are allowed to do this
(which is necessary in order to for perform work), which effectively bypasses
all access control, the use of a bastion host is not providing any value to end
users or administrators.

hosts are bridging public and rfc1918 subnets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Another work around to the lack of layer3 access between end users systems and
lab machines, is to configure an additional network interface in the large
public ``/23`` subnet which is mixed servers and desktops.  As the need for
external access to services is growing, it seems probably that this practice
will become more common and defeats the purpose of having an isolated "lab"
environment.

Network Architecture
====================

Current
-------

.. figure:: /_static/lab-current.png
   :name: fig-lab-current
   :alt: simplified lab network diagram

Revised With Single Border Router
---------------------------------

.. figure:: /_static/lab-proposed-shared-router.png
   :name: fig-lab-proposed-shared-router
   :alt: revised lab network diagram with shared router

Revised With Lab Dedicated Border Router
----------------------------------------

.. figure:: /_static/lab-proposed-dedicated-router.png
   :name: fig-lab-proposed-dedicated router
   :alt: revised lab network diagram with dedicated router
