# Python bindings for the Myo SDK

[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/NiklasRosenstein/myo-python?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge) [![Code Issues](http://www.quantifiedcode.com/api/v1/project/cf45bc5553f14a799abd736fdb4c6441/badge.svg)](http://www.quantifiedcode.com/app/project/cf45bc5553f14a799abd736fdb4c6441)

The Python `myo` package is a ctypes based wrapper for the Myo shared libraries provided by Thalmic Labs. The aim is to provide a complete Python interface for the Myo SDK as a high level API to developers using Pytho 2 or 3.

There are two ways you can use the Myo bindings, by polling data or by getting push notifications. Basically, your way to start and end a connection with Myo(s) is like this:

```python
import myo as libmyo; libmyo.init()

hub = libmyo.Hub()
hub.run(1000, listener)
try:
    # Wait until something happens or the hub is shut down.
    while hub.running:
        time.sleep(0.25)
finally:
    hub.shutdown()
```

> *Note*: It is very important that you wrap everything in a try-finally clause to be able to shut down the hub when you want to exit the application as the Hub starts a non-daemon thread. Also, if the hub is garbage collected without it being `shutdown()`, a warning will be printed.

By the way, we prefer to import the `myo` package as `libmyo` as it is very likely you will have a `myo` variable in your local scope that represents a single armband.

## Pushing

You must implement the `myo.device_listener.DeviceListener` class and pass an instance of it to `myo.Hub.run()`. Any event that is sent via the Myo will end up in your listener.

Generally, implementing a `DeviceListener` should be preferred if it will not involve any more complexities in your application.

```python
class Listener(libmyo.DeviceListener):

    def on_pair(self, myo, timestamp, firmware_version):
        print("Hello, Myo!")

    def on_unpair(self, myo, timestamp):
        print("Goodbye, Myo!")

    def on_orientation_data(self, myo, timestamp, quat):
        print("Orientation:", quat.x, quat.y, quat.z, quat.w)
```

## Pulling

Pulling can be done by using an instance of `myo.device_listener.Feed` as the listener to `Hub.run()`. The Feed will cache any data when its received and it can be accessed through a MyoProxy object at any time.

```python
feed = libmyo.device_listener.Feed()
hub.run(1000, feed)
try:
    myo = feed.wait_for_single_device(timeout=10)
    if not myo:
        print("No Myo connected after 10 seconds.")
        return

    print("Hello, Myo!")
    while hub.running and myo.connected:
        quat = myo.orientation
        print("Orientation:", quat.x, quat.y, quat.z, quat.w)
    print("Goodbye, Myo!")
finally:
    hub.shutdown()
```

Check the [examples](examples/) directory for more.

## Requirements

- [six](https://pypi.python.org/pypi/six) for Python 2 and 3 compatibility

## Getting Started

1. Download myo-python or clone it from GitHub
2. Install it, for testing with `pip install -e myo-python/` or add it to your `PYTHONPATH`
3. [Download the Myo SDK](https://developer.thalmic.com/downloads) and make sure make it's in your `PATH`

> These example assume that the Myo SDK is inside your current working directory.

### Windows

    > git clone git@github.com:NiklasRosenstein/myo-python.git
    > virtualenv env --no-site-packages
    > env/Scripts/activate
    > pip install -e .
    > set PATH=%PATH%;.\myo-sdk-win-0.9.0\bin
    > python myo-python\examples\hello_myo.py

### Windows (Cygwin)

    $ git clone git@github.com:NiklasRosenstein/myo-python.git
    $ virtualenv env --no-site-packages
    $ source env/bin/activate
    $ pip install -e .
    $ export PATH=$PATH:$(pwd)/myo-sdk-win-0.9.0/bin
    $ python myo-python/examples/hello_myo.py

### Mac

    $ git clone git@github.com:NiklasRosenstein/myo-python.git
    $ virtualenv env --no-site-packages
    $ source env/bin/activate
    $ pip install -e .
    $ export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:$(pwd)/myo-sdk-mac-0.9.0/myo.framework
    $ python myo-python/examples/hello_myo.py

For a list of all contributors, see [here](https://github.com/NiklasRosenstein/myo-python/graphs/contributors).

## Projects using Myo Python

- [Myo Matlab](https://github.com/yijuilee/myomatlab)

------------------------------------------------------------------------

This project is licensed under the MIT License. Copyright &copy; 2015 Niklas Rosenstein
