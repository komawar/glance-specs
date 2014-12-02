..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Add capabilities to storage driver
====================================

https://blueprints.launchpad.net/glance/+spec/store-capabilities

This features will enable the static and dynamic ability of a storage driver
instance based on it's implementation as well as the configuration. The runtime 
status of the backend storage will influence the capacity too.

Henceforth, glance_store library can do more proper operations on
storage to support upper layer request, like to enable or disable image
adding function to glance, or if allow glance reuse driver instance for all
request according to whether the driver and/or the operation session of
backend storage is stateless.

Problem description
===================

The capabilities of store driver can be affected by the following factors:

    1. Status of a connected backend storage.
    2. The deployment configuration.
    3. State of the driver code.
    4. Technically one of them is static whereas the other is dynamic.

Currently glance_store and glance don't use these capabilities. At a lower
layer of the code, the driver doesn't have a way to expose its capabilities.
On the other hand in the upper layer of the code there is no common method to
check these generic operations of the store drivers.

Currently we observe at least two kind of problems:

    * Some necessary applicability check on the operation related with store access
      are missing, which lows the usability of glance service and glance_store lib.
    
    * Hard to implement such check logic neatly, including common check routine on
      these generic operations of store driver, and the check for particular
      feature.

Proposed change
===============

1. Existing drivers need to be updated to expose proper capabilities.
	a) List of required store capabilities as needed currently.

	b) Available capabilities:
      +----------------+------------------------------------------------------+
      | Capability     | Description                                          |
      +================+======================================================|
      |	READ_ACCESS    | # Generic read access                                |
      +----------------+------------------------------------------------------+
      |	WRITE_ACCESS   | # Generic write access                               |
      +----------------+------------------------------------------------------+
      |	RW_ACCESS      | # READ_ACCESS and WRITE_ACCESS                       |
      +----------------+------------------------------------------------------+
      |	READ_OFFSET    | # Read all bits from a offset (Included READ_ACCESS) |
      +----------------+------------------------------------------------------+
      | WRITE_OFFSET   | # Write all bits to a offset (Included WRITE_ACCESS) |
      +----------------+------------------------------------------------------+
      |	RW_OFFSET      | # READ_OFFSET and WRITE_OFFSET                       |
      +----------------+------------------------------------------------------+
      |	READ_CHUNK     | # Read required length of bits (Included READ_ACCESS)|
      +----------------+------------------------------------------------------+
      |	WRITE_CHUNK    | # Write required length of bits                      |
      |                |   (Included WRITE_ACCESS)                            |
      +----------------+------------------------------------------------------+
      |	RW_CHUNK       | # READ_CHUNK and WRITE_CHUNK                         |
      +----------------+------------------------------------------------------+
      |	READ_RANDOM    | # READ_OFFSET and READ_CHUNK                         |
      +----------------+------------------------------------------------------+
      |	WRITE_RANDOM   | # WRITE_OFFSET and WRITE_CHUNK                       |
      +----------------+------------------------------------------------------+
      |	RW_RANDOM      | # RW_OFFSET and RW_CHUNK                             |
      +----------------+------------------------------------------------------+
      |	DRIVER_REUSABLE| # driver is stateless and its instance can be reused |
      |                |   safely                                             |
      +----------------+------------------------------------------------------+

2. Add common check routine on these generic operations for the store drivers.

3. Refactoring existing drivers to leverage these capabilities.

4. Add the logic to recreate driver instance if the storage or driver
   isn't stateless.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance Impact
------------------

There is no expectation for any obvious degradation in the performance. 

The reason being:

	There is a simple addition of a checker function being triggered in the
	form of a hook for each store operation, with current inclusion of the
	('get', 'delete', 'add') operations.

Other deployer impact
---------------------

None.

Developer impact
----------------

Developers implementing new drivers to the glance_store library would need to
be aware of this concept. The static and dynamic ability of the storage backend
could influence their and their implementation design.

All changes maintain backward-compatibility.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  zhiyan (lzy-dev)

Other contributors:
  <launchpad-id or None>

Reviewers
---------

Core reviewer(s):
  Nikhil Komawar (nikhil-komawar)
  Stuart McLaren (stuart-mcLaren)

Work Items
----------

* :ref:`glance_store change`
* :ref:`Glance change`

Dependencies
============

None

Testing
=======

Necessary unit and functional test cases, will be added into
glance_store as well as glance.

Documentation Impact
====================

None

References
==========

Corresponding changes:

* :ref:`glance_store change`
* :ref:`Glance change`

.. _glance_store change: https://review.openstack.org/#/c/137416

.. _Glance change: https://review.openstack.org/#/c/141825
