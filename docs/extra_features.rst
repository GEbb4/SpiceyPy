Extra Features
==============

SpiceyPy was always intended to be a thin wrapper around the CSPICE routines, however
as it is implemented in Python there are a small number of extra features to help
make end-users' code more Pythonic. One such feature is a context manager for
loading and unloading kernels, which ensures that loaded kernels are always
properly unloaded.

For example, one common paradigm when using SpiceyPy is needing to catch errors
in order to guarantee kernels get unloaded.

.. code:: python

    import spiceypy as spice

    utc = input("Enter a time in UTC: ")

    spice.furnsh("win.tm")
    try:
        et = spice.str2et(utc)
    finally:
        spice.unload("win.tm")

    print("UTC {0} corresponds to ET {1}".format(utc, et))

The try/finally construct is used because otherwise, if an error were raised by
`str2et`, the `unload` would never be run.

However, SpiceyPy includes a context manager, :py:class:`spiceypy.context.KernelPool` ,
which loads the kernels and then guarantees that they will be unloaded even if
an exception occurs inside it.

.. code:: python

    import spiceypy as spice
    from spicepy.context import KernelPool

    utc = input("Enter a time in UTC: ")
    with KernelPool("win.tm"):
        et = spice.str2et(utc)

    print("UTC {0} corresponds to ET {1}".format(utc, et))

This context manager can also be used to decorate a function, and will ensure
that the necessary kernel files are loaded when that function executes and
unloaded when it completes.

.. code:: python

    import spiceypy as spice
    from spiceypy.context import KernelPool

    @KernelPool("win.tm")
    def get_ephemeris_time(utc):
        return spice.str2et(utc)

    utc = input("Enter a time in UTC: ")
    et = get_ephemeris_time(utc)
    print("UTC {0} corresponds to ET {1}".format(utc, et))
