=============
Scan/Discover
=============

To discover Bluetooth devices that can be connected to:

.. code-block:: python

    import asyncio
    from bleak import discover

    async def run():
        devices = await discover()
        for d in devices:
            print(d)

    loop = asyncio.get_event_loop()
    loop.run_until_complete(run())

This will produce a printed list of detected devices:

.. code-block:: sh

    24:71:89:CC:09:05: CC2650 SensorTag
    4D:41:D5:8C:7A:0B: Apple, Inc. (b'\x10\x06\x11\x1a\xb2\x9b\x9c\xe3')

The first part, a MAC address in Windows and Linux and a UUID in macOS, is what is
used for connecting to a device using Bleak. The list of objects returned by the `discover`
method are instances of :py:class:`bleak.backends.device.BLEDevice` and has ``name``, ``address``
and ``rssi`` attributes, as well as a ``metadata`` attribute, a dict with keys ``uuids`` and ``manufacturer_data``
which potentially contains a list of all service UUIDs on the device and a binary string of data from
the manufacturer of the device respectively.


BleakScanner
------------

A new scanning class is being implemented, and is ready to be used for the .NET backend,
and partly for the macOS backend as well.
It can be used in a fashion similar to the old ``discover`` method, using its class method:

.. code-block:: python

    import asyncio
    from bleak import BleakScanner

    async def run():
        devices = await BleakScanner.discover()
        for d in devices:
            print(d)

    loop = asyncio.get_event_loop()
    loop.run_until_complete(run())

But it can also be used as a separate object, either in a synchronous context manager way:

.. code-block:: python

    import asyncio
    from bleak import BleakScanner

    async def run():
        async with BleakScanner() as scanner:
            await asyncio.sleep(5.0)
            devices = await scanner.get_discovered_devices()
        for d in devices:
            print(d)

    loop = asyncio.get_event_loop()
    loop.run_until_complete(run())

or separately, calling ``start`` and ``stop`` methods on the scanner manually:

.. code-block:: python

    import asyncio
    from bleak import BleakScanner

    def detection_callback(*args):
        print(args)

    async def run():
        scanner = BleakScanner()
        scanner.register_detection_callback(detection_callback)
        await scanner.start()
        await asyncio.sleep(5.0)
        await scanner.stop()
        devices = await scanner.get_discovered_devices()

        for d in devices:
            print(d)

    loop = asyncio.get_event_loop()
    loop.run_until_complete(run())

In the manual mode, it is possible to add an own callback that you want to call upon each
scanner detection, as can be seen above. There is also possibilities of adding scanning filters,
but these differ so widely between implementations, so these details are recorded there instead.

Scanning Filters
----------------

There are some scanning filters that can be applied, that will reduce your scanning
results prior to them getting to bleak. These are pretty quite backend specific, but
they are generally used like this:

- On the `discover` method, send in keyword arguments according to what is
  described in the docstring of the method.
- On the backend's `BleakScanner` implementation, either send in keyword arguments
  according to what is described in the docstring of the class or use the
  ``set_scanning_filter`` method to set them after the instance has been created.

Scanning filters are currently implemented in Windows and BlueZ backends, but not yet
in the macOS backend.
