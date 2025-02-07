
Linescan project
================

Since February 2024 at BMM, we have been recording data about
alignment scans.  These data are the basis of this project.

The goal is to develop a function that predicts the interpretation of
an alignment scan and offers that prediction to the user as a solution
to alignment problem.

Each time an alignment scan has been made in the last year, a flat
text file has been written with entries containing:

+ The date and time of the scan
+ The kind of scan, i.e. which motor was moved and which detector was
  probed
+ The UID of the Tiled record
+ The position that was selected either by the human or by the
  automated process running the scan

The concept is to consume the data from these measurements and develop
a predictor for future measurements.
 

Interpreting the log files
==========================

There are currently two log files:

+ ``linescan_evaluation.txt.1``
+ ``linescan_evaluation.txt.2``

They are identical in structure.  I rotate the logs every few months.
I just rotated them today.  Starting Monday, BMM will begin making new
log entries that can be added to the training/testing set in the
future.

An entry in a log file looks like this:

.. code-block:: text

    2024-02-12T09-33-05
         mode = xafs_x/It
         uid = e9979416-fd6d-4ea5-9d56-2257f52c9572
         position = 144.1771555806452

The lines are:

1.  data and time of the scan.  This might be helpful in at least one
    way: two similar scans happening in quick succession might
    represent a refinement of an alignment.  One could imagine a model
    understanding this.

2.  The second line indicates what motor and detector is involved in the
    scan.  I will explain below how to read these tokens.

3.  The Tiled UID of the record of the scan.

4.  The position chosen by the human or the automation algorithm.
    While the majority of these will be adequate choices, not all of
    them are.  Humans make bad decisions.  Automation algorithms
    fail.  That's what makes this a fun AI/ML project!


Getting started with Tiled
==========================

The examples below use the symbol ``bmm_catalog`` to access the
historical data from BMM.  Do this:

.. code-block:: python

  from tiled.cllient import from_uri
  container = from_uri('https://tiled.nsls2.bnl.gov')
  bmm_catalog = container['bmm']['raw']

Examining bmm_catalog:

.. code-block:: python

    In [5]: from_uri('https://tiled.nsls2.bnl.gov')['bmm']['raw']
    Out[5]: <Catalog {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, ...} ~208814 entries>

Apparently, as of the time of this writing, BMM has done just over
208,000 things that have been recorded in Databroker.

The examples below all use the scan UID as indices, for example
consider the most recent scan:

.. code-block:: python

   most_recent = bmm_catalog[-1]

We can find the UID by

.. code-block:: python

   most_recent.metadata['start']['uid']

It is ``3b5dc5fa-9252-43cc-b70d-8288f03dbb69``, so

.. code-block:: python

    uid = '3b5dc5fa-9252-43cc-b70d-8288f03dbb69'
    most_recent = bmm_catalog[uid]

returns the record for that scan.  When examining historical data, we
will almost always use the UIDs.

The metadata, which is structured as a nested dict (or, equivalently,
a JSON string) can be examined by

.. code-block:: text

    bmm_catalog[uid].metadata

The actual data can be examined by

.. code-block:: text

    bmm_catalog[uid].primary.read()

and the columns can be obtained as arrays by things like

.. code-block:: text

    bmm_catalog[uid].primary.read()['I0']

That should be enough to get you started exploring Tiled records.



Motors
======

All (or almost all) of the motors have names like ``xafs_y`` or
``xafs_pitch``.  At BMM, the naming convention for motors is that
``xafs_*`` indicates a motor that is involved in positioning the sample
in the beam.

If any log entry involves a non ``xafs_*`` motor, I would suggest
skipping it.

Using the UID of the example above to examine on of these Tiled
records:

.. code-block:: text

    BMM C.111 [258] ▶ bmm_catalog['e9979416-fd6d-4ea5-9d56-2257f52c9572'].primary.read()
    Out[258]: 
    <xarray.Dataset> Size: 1kB
    Dimensions:               (time: 31)
    Coordinates:
      * time                  (time) float64 248B 1.708e+09 1.708e+09 ... 1.708e+09
    Data variables:
        It                    (time) float64 248B 4.354 4.465 4.576 ... 5.278 4.62
        xafs_x                (time) float64 248B 138.8 139.2 139.6 ... 150.4 150.8
        xafs_x_user_setpoint  (time) float64 248B 138.8 139.2 139.6 ... 150.4 150.8
        Ir                    (time) float64 248B 0.03847 0.04006 ... 0.0472 0.04105
        I0                    (time) float64 248B 39.48 39.48 39.48 ... 39.58 39.59
    Attributes:
        stream_name:  primary


So, since the ``mode`` of the log entry is ``xafs_x/It``, the abscissa of
a plot of this alignment scan would be:

.. code-block:: python

    x = bmm_catalog['e9979416-fd6d-4ea5-9d56-2257f52c9572'].primary.read()['xafs_x']


Detectors
=========

The ``It`` part of the mode of this example tells us that a detector
named ``It`` is the signal used in the alignment scan.  Note that ``It``
is one of the Data variables in the Tiled record.

So, the ordinate of a plot of this alignment scan would be

.. code-block:: python

    y = bmm_catalog['e9979416-fd6d-4ea5-9d56-2257f52c9572'].primary.read()['It']


Plotting the alignment scan
===========================

Armed with the abscissa and ordinate above, an unadorned plot would
be:

.. code-block:: python

    import matplotlib.pyplot as plt
    plt.plot(x, y)

The position indicated in the log entry -- 144.177 -- should be within
range and is the position that was chosen by the human or the
algorithm.

In that sense, this is a supervised training set and the chosen
positions represent the supervised tag for the data.


Detector types
==============

I think you will find only three detector names in this entire
collection:

 + ``It``
 + ``Ir``
 + ``Xs``

``It`` and ``Ir`` are easy to interpret, there will be entries with those
names in the Data variables of the Tiled record.

``Xs`` is a bit more complicated as it represents the sum of 1, 4, or 7
entries in the Data variables.

Here's an example of log entry with ``Xs``:

.. code-block:: text

    2024-07-28T13-22-35
         mode = xafs_y/Xs
         uid = c0cedeca-5503-4f82-8928-77b2416a73e2
         position = 92.80597935383064

If we look at the metadata of the record:

.. code-block:: python

  md = bmm_catalog['c0cedeca-5503-4f82-8928-77b2416a73e2'].metadata

``md['start']['detectors']`` is

.. code-block:: python

    ['quadem1', 'Ic0', 'Ic1', '4-element SDD']

This tells us that the 4-element SDD detector was used in the alignment:

.. code-block:: text

    BMM C.111 [261] ▶ bmm_catalog['c0cedeca-5503-4f82-8928-77b2416a73e2'].primary.read()
    Out[261]: 
    <xarray.Dataset> Size: 4MB
    Dimensions:                      (time: 31, bin_count: 4096)
    Coordinates:
      * time                         (time) float64 248B 1.722e+09 ... 1.722e+09
    Dimensions without coordinates: bin_count
    Data variables: (12/13)
        Ir                           (time) float64 248B 3.118e-05 ... -2.156e-05
        4-element SDD_channel01_xrf  (time, bin_count) float64 1MB 0.0 0.0 ... 0.0
        La1                          (time) float64 248B 39.02 62.45 ... 11.27 4.165
        4-element SDD_channel02_xrf  (time, bin_count) float64 1MB 0.0 0.0 ... 0.0
        La2                          (time) float64 248B 45.01 78.01 ... 18.0 13.0
        4-element SDD_channel03_xrf  (time, bin_count) float64 1MB 0.0 1.0 ... 0.0
        ...                           ...
        4-element SDD_channel04_xrf  (time, bin_count) float64 1MB 0.0 0.0 ... 0.0
        La4                          (time) float64 248B 40.0 95.01 ... 12.0 10.0
        I0                           (time) float64 248B 46.16 46.14 ... 45.92 45.91
        It                           (time) float64 248B 0.0006042 ... -8.478e-05
        xafs_y                       (time) float64 248B 91.58 91.71 ... 95.44 95.58
        xafs_y_user_setpoint         (time) float64 248B 91.58 91.71 ... 95.44 95.58
    Attributes:
        stream_name:  primary
    

There are Data variable entries called ``La1``, ``La2``, ``La3``, and ``La4``.

This means that we were using signal from the Lanthanum (periodic
table symbol La) to do the alignment.

Any ``Xs`` scan using the ``4-element SDD`` will have 4 entries using
where the Data variable names are an element symbol followed by the
numbers 1/2/3/4.

So, in this case, the ordinate would be

.. code-block:: python

    y = bmm_catalog['c0cedeca-5503-4f82-8928-77b2416a73e2'].primary.read()['La1'] +
        bmm_catalog['c0cedeca-5503-4f82-8928-77b2416a73e2'].primary.read()['La2'] +
        bmm_catalog['c0cedeca-5503-4f82-8928-77b2416a73e2'].primary.read()['La3'] +
        bmm_catalog['c0cedeca-5503-4f82-8928-77b2416a73e2'].primary.read()['La4']

There are also examples of alignment scans using the ``1-element SDD``
and the ``7-element SDD``.  These will have 1 or 7 entries in the Data
variables list and the ordinate should be constructed accordingly.


This is awfully long wall of text, but hopefully it clarifies how to
interpret the log files.

	
    
