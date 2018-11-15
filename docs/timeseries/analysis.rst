.. _timeseries-analysis:

Manipulation and analysis of time series
****************************************

.. |Table| replace:: :class:`~astropy.table.Table`
.. |SampledTimeSeries| replace:: :class:`~astropy_timeseries.SampledTimeSeries`
.. |BinnedTimeSeries| replace:: :class:`~astropy_timeseries.BinnedTimeSeries`

Combining time series
=====================

The  :func:`~astropy.table.vstack`, and :func:`~astropy.table.hstack` functions
from the :mod:`astropy.table` module can be used to stack time series in
different ways.

Time series can be stacked 'vertically' or row-wise using the
:func:`~astropy.table.vstack` function (although note that sampled time
series cannot be combined with binned time series and vice-versa)::

    >>> from astropy.table import vstack
    >>> from astropy import units as u
    >>> from astropy_timeseries import SampledTimeSeries
    >>> ts_a = SampledTimeSeries(time='2016-03-22T12:30:31',
    ...                          time_delta=3 * u.s,
    ...                          data={'flux': [1, 4, 5, 3, 2] * u.mJy})
    >>> ts_b = SampledTimeSeries(time='2016-03-22T12:50:31',
    ...                          time_delta=3 * u.s,
    ...                          data={'flux': [4, 3, 1, 2, 3] * u.mJy})
    >>> ts_ab = vstack([ts_a, ts_b])
    >>> ts_ab
    <SampledTimeSeries length=10>
              time            flux
                              mJy
             object         float64
    ----------------------- -------
    2016-03-22T12:30:31.000     1.0
    2016-03-22T12:30:34.000     4.0
    2016-03-22T12:30:37.000     5.0
    2016-03-22T12:30:40.000     3.0
    2016-03-22T12:30:43.000     2.0
    2016-03-22T12:50:31.000     4.0
    2016-03-22T12:50:34.000     3.0
    2016-03-22T12:50:37.000     1.0
    2016-03-22T12:50:40.000     2.0
    2016-03-22T12:50:43.000     3.0

Time series can also be combined 'horizontally' or column-wise with other tables
using the :func:`~astropy.table.hstack` function, though these should not be
time series (as having multiple time columns would be confusing)::

    >>> from astropy.table import Table, hstack
    >>> data = Table(data={'temperature': [40., 41., 40., 39., 30.] * u.K})
    >>> ts_a_data = hstack([ts_a, data])
    >>> ts_a_data
    <SampledTimeSeries length=5>
              time            flux  temperature
                              mJy          K
             object         float64    float64
    ----------------------- ------- -----------
    2016-03-22T12:30:31.000     1.0        40.0
    2016-03-22T12:30:34.000     4.0        41.0
    2016-03-22T12:30:37.000     5.0        40.0
    2016-03-22T12:30:40.000     3.0        39.0
    2016-03-22T12:30:43.000     2.0        30.0

Sorting time series
===================

Sorting time series in-place can be done using the
:meth:`~astropy.table.Table.sort` method, as for |Table|::

    >>> ts = SampledTimeSeries(time='2016-03-22T12:30:31',
    ...                        time_delta=3 * u.s,
    ...                        data={'flux': [1., 4., 5., 3., 2.]})
    >>> ts
    <SampledTimeSeries length=5>
              time            flux
             object         float64
    ----------------------- -------
    2016-03-22T12:30:31.000     1.0
    2016-03-22T12:30:34.000     4.0
    2016-03-22T12:30:37.000     5.0
    2016-03-22T12:30:40.000     3.0
    2016-03-22T12:30:43.000     2.0
    >>> ts.sort('flux')
    >>> ts
    <SampledTimeSeries length=5>
              time            flux
             object         float64
    ----------------------- -------
    2016-03-22T12:30:31.000     1.0
    2016-03-22T12:30:43.000     2.0
    2016-03-22T12:30:40.000     3.0
    2016-03-22T12:30:34.000     4.0
    2016-03-22T12:30:37.000     5.0

Resampling
==========

The |SampledTimeSeries| class has a
:meth:`~astropy_timeseries.SampledTimeSeries.downsample` method that can be used
to bin values from the time series into bins of equal time, using a custom
function (mean, median, etc.). This operation returns a |BinnedTimeSeries|.
The following example shows how to use this to bin a light curve from the Kepler
mission into 20 minute bins using a median function. First, we read in the
data using:

.. plot::
   :context: reset
   :nofigs:

    from astropy_timeseries import SampledTimeSeries
    from astropy.utils.data import get_pkg_data_filename
    example_data = get_pkg_data_filename('timeseries/kplr010666592-2009131110544_slc.fits')
    kepler = SampledTimeSeries.read(example_data, format='kepler.fits')

(see :ref:`timeseries-io` for more details about reading in data). We can then
downsample using:

.. plot::
   :include-source:
   :context:
   :nofigs:

    import numpy as np
    from astropy import units as u
    kepler_binned = kepler.downsample(bin_size=20 * u.min, func=np.nanmedian)

We can take a look at the results:

.. plot::
   :include-source:
   :context:

    import matplotlib.pyplot as plt
    plt.plot(kepler.time.jd, kepler['sap_flux'], 'k.', markersize=1)
    plt.plot(kepler_binned.start_time.jd, kepler_binned['sap_flux'], 'r-', drawstyle='steps-pre')
    plt.xlabel('Barycentric Julian Date')
    plt.ylabel('SAP Flux (e-/s)')

Folding
=======

The |SampledTimeSeries| class has a
:meth:`~astropy_timeseries.SampledTimeSeries.fold` method that can be used to
return a new time series with a relative and folded time axis. This method
takes the period as a :class:`~astropy.units.Quantity`, and optionally takes
an epoch as a :class:`~astropy.time.Time`, which defines a zero time offset:

.. plot::
   :context: reset
   :nofigs:

   import numpy as np
   from astropy import units as u
   import matplotlib.pyplot as plt
   from astropy_timeseries import SampledTimeSeries
   from astropy.utils.data import get_pkg_data_filename

   example_data = get_pkg_data_filename('timeseries/kplr010666592-2009131110544_slc.fits')
   kepler = SampledTimeSeries.read(example_data, format='kepler.fits')

.. plot::
   :include-source:
   :context:

    kepler_folded = kepler.fold(period=2.2 * u.day, midpoint_epoch='2009-05-02T20:53:40')

    plt.plot(kepler_folded.time.jd, kepler_folded['sap_flux'], 'k.', markersize=1)
    plt.xlabel('Time from midpoint epoch (days)')
    plt.ylabel('SAP Flux (e-/s)')

Arithmetic
==========

Since time series objects are sub-classes of |Table|, they naturally support
arithmetic on any of the data columns. As an example, we can take the folded
Kepler time series we have seen in the examples above, and normalize it to the
sigma-clipped median value.

.. plot::
   :context: reset
   :nofigs:

   import numpy as np
   from astropy import units as u
   import matplotlib.pyplot as plt
   from astropy_timeseries import SampledTimeSeries
   from astropy.utils.data import get_pkg_data_filename

   example_data = get_pkg_data_filename('timeseries/kplr010666592-2009131110544_slc.fits')
   kepler = SampledTimeSeries.read(example_data, format='kepler.fits')
   kepler_folded = kepler.fold(period=2.2 * u.day, midpoint_epoch='2009-05-02T20:53:40')

.. plot::
   :include-source:
   :context:

    from astropy.stats import sigma_clipped_stats

    mean, median, stddev = sigma_clipped_stats(kepler_folded['sap_flux'])

    kepler_folded['sap_flux_norm'] = kepler_folded['sap_flux'] / median

    plt.plot(kepler_folded.time.jd, kepler_folded['sap_flux_norm'], 'k.', markersize=1)
    plt.xlabel('Time from midpoint epoch (days)')
    plt.ylabel('Normalized flux')
