.. _timeseries-io:

Reading/writing time series
***************************

.. |Table| replace:: :class:`~astropy.table.Table`
.. |TimeSeries| replace:: :class:`~astropy_timeseries.TimeSeries`
.. |BinnedTimeSeries| replace:: :class:`~astropy_timeseries.BinnedTimeSeries`

Build-in readers
================

Since |TimeSeries| and |BinnedTimeSeries| are sub-classes of |Table|,
they have :meth:`~astropy.table.Table.read` and
:meth:`~astropy.table.Table.write` methods that can be used to read time series
from files. We include a few readers for well-defined formats in `astropy_timeseries` -
for example we have readers for light curves in FITS format from the
`Kepler <https://www.nasa.gov/mission_pages/timeseries/main/index.html>`_ and
`TESS <https://tess.gsfc.nasa.gov/>`_ missions.

Here is an example of using Kepler FITS time series - we start off by fetching an example
file:

.. plot::
   :include-source:
   :context: reset
   :nofigs:

   from astropy.utils.data import get_pkg_data_filename
   example_data = get_pkg_data_filename('timeseries/kplr010666592-2009131110544_slc.fits')

This will set ``example_data`` to the filename of the downloaded file (so you can
replace this by the filename for the file you want to read in). We can then read in
the time series using:

.. plot::
   :include-source:
   :context:
   :nofigs:

   from astropy_timeseries import TimeSeries
   kepler = TimeSeries.read(example_data, format='kepler.fits')

Let's check that the time series has been read in correctly:

.. plot::
   :include-source:
   :context:

   import matplotlib.pyplot as plt

   plt.plot(kepler.time.jd, kepler['sap_flux'], 'k.', markersize=1)
   plt.xlabel('Barycentric Julian Date')
   plt.ylabel('SAP Flux (e-/s)')

Reading other formats
=====================

At the moment only a few formats are defined in astropy itself, in part because
there are not many well documented formats for storing time series. So in many
cases, you will likely have to first read in your files using e.g. the more
generic |Table| class (see :ref:`read_write_tables`). In fact, the
:meth:`TimeSeries.read <astropy_timeseries.TimeSeries.read>` and
:meth:`BinnedTimeSeries.read <astropy_timeseries.BinnedTimeSeries.read>` methods
can do this behind the scenes - if the table cannot be read by any of the time
series readers, these methods will try and use some of the default :class:`~astropy.table.Table`
readers and then require users to specify the name of the important columns.

For example, if you are reading in a file called ``sampled.dat`` where the time
column is called ``date`` and is an ISO string, you can do::

    >>> from astropy_timeseries import TimeSeries
    >>> ts = TimeSeries.read('sampled.dat', format='ascii.csv',
    ...                      time_column='date')  # doctest: +SKIP

If you are reading in a binned time series from a file called ``binned.dat``
and with a column ``date_start`` giving the start time and ``date_end`` giving
the end time of each bin, you can do::

    >>> from astropy_timeseries import BinnedTimeSeries
    >>> ts = BinnedTimeSeries.read('binned.dat', format='ascii.csv',
    ...                            time_bin_start_column='date_start',
    ...                            time_bin_end_column='date_end')  # doctest: +SKIP


See the documentation for :meth:`TimeSeries.read
<astropy_timeseries.TimeSeries.read>` and :meth:`BinnedTimeSeries.read
<astropy_timeseries.BinnedTimeSeries.read>` for more details.

Alternatively, you can read in the table using your own code then construct the
time series object as described in :ref:`timeseries-initializing`.

If you have written a reader/writer for a commonly used format, please feel free
to contribute it to astropy!
