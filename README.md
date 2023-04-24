# OCD: how to

This document is intended to give a starting point to setup and run OCD.

For the full (and up-to-date) documentation refer to
[hetdex-ocd.readthedocs.io](https://hetdex-ocd.readthedocs.io)

## Install OCD

To install OCD follow [this
instructions](https://ocd.readthedocs.io/en/latest/install.html#from-local-ocd-copy).
Since OCD doesn't have a regular release schedule (if any), checkout or update
the svn repository and install using 

    pip install -e .

In order to run shots, OCD needs to talk with an external MySQL database. If you
don't have a MySQL server running or do not want to set up a MySQL database, run
the following command instead (see [MySQL database section](#MySQL-database))

    pip install -e .[docker]

## Files used by OCD

* ``ocd.cfg``: main configuration file; the entries in this file override
  the once from the one shipped with OCD. This allows to use a local, small
  configuration file with only the relevant modifications and doesn't make it
  necessary to update it every time it is modified upstream.
  This file can be retrieved with the ``ocd config copy -b`` command
* ``loggers.cfg``: file defining the loggers used by ``ocd.tcs_proxy`` in case
  the ``TCSLog`` and the ``tcssubsystem`` are not used. To be able to use it set
  ``tcs_log_mock_path = loggers.cfg`` in the ``[urls]`` section.
  This file can be retrieved with the ``ocd config copy -b`` command
* ``shot_list.dat``: list of shots
* ``HETcommands``: ``*cfg`` and ``*hetcommands`` files for each of the shots in
  ``shot_list.dat``
* ``ACAMimages``: ACAM images for each of the shots in ``shot_list.dat``

## Extra executables

OCD relies on ``$CUREBIN/autoschedule_main`` to schedule the next shot.
The full name of the executable and some of its command line options are taken
from the ``[autoschedule]`` configuration section.

## Shuffle configuration file

Each configuration file (``HETcommands/*cfg``) has the following format:

    HETcommands/{shotid}_{QIFU}_{track_initial}.cfg

For shot with id HS0050_E this resolves to:

    HETcommands/HS0050_E_000_E.cfg

The template for the name of the shuffle configuration file is passed to ocd via
the ``shuffle_conf_template`` option of the ``[run_shot]`` section

## Time

The shot list file and the associated files arrived with the following note:

    Time of execution UT time for day 2018-05-15:

    HS00510_W was observed starting at 09:15
    HS0526_W was observed starting at 09:36 (which was non optimal but allowed)

So, when testing OCD with this ``shot_list.dat``, I suggest to mock the times
that OCD uses when running ``autoschedule_main``. To do so set the following
configuration entry:

    [dates]
    mock_time = 2018-05-15T09:00:00
 
## Test OCD

### Check the configuration

First thing to do is to get the configuration file and go through it making sure
that all the entries are correctly set. Typically you will have to:

* modify the location of the input files (shuffle and ACAM paths, shot list);
* if you are at HET and/or want receive/send event outside of the localhost,
  update the ``[urls]`` as necessary;
* if you want to drive OCD replaying a database instead of listening to the TCS
  event stream, comment the ``event_urls`` and uncomment the ``ocd_db_replay``
  options in the ``[urls]`` configuration section;
* if you don't have the ``TCSLog`` and ``tcssubsystem`` packages and want to
  have some feedback about the execution, set ``tcs_log_mock_path`` to a valid
  logging configuration file, e.g. the ``loggers.cfg`` described before;
* adjust the ``mysql_*`` configurations in the ``[database]`` section; if you
  don't have a MySQL server or it's not configured either create a data table as
  described
  [here](https://hetdex-ocd.readthedocs.io/en/latest/ocd_notes.html#mysql-note);
  or refer to [MySQL database section](#MySQL-database);

### MySQL database

As already hinted in this document, OCD relies on an external MySQL database. It
does in order to be able to retrieve a global observation number from the HET
database. To help deploy a MySQL database in a test environment, OCD ships with
a subcommand, [``ocd
docker_mysql``](https://hetdex-ocd.readthedocs.io/en/latest/ocd_executables.html#ocd-docker-mysql),
that uses [``docker``](https://www.docker.com) to create an
isolated container running a MySQL server, containing a database with the table
described in
[here](https://hetdex-ocd.readthedocs.io/en/latest/ocd_notes.html#mysql-note).

Run

    ocd docker_mysql up [--config ocd.cfg]

to start the MySQL container. The command prints the URL and port that can be
used to connect the database. Update the relevant configuration options with
those values.

Once done, you can pull down the container with:

    ocd docker_mysql down [--config ocd.cfg]


### Run OCD

Now everything should be ready to run. Just run 

    ocd run --config ocd.cfg

If you setup a mock time and the set ``tcs_log_mock_path``, you should see
something like:

    You are about to start OCD with mock times starting at 2018-05-15T09:00:00.
    This feature is enabled for tests and should not be used when running OCD to
    drive observations. If you really want to proceed? [y/n] y
    Welcome to OCD
    [Mock TCSLog - 2018-05-17 15:37:17,866 - INFO] TCSLog initialized
    [Mock TCSLog - 2018-05-17 15:37:17,866 - INFO] TCS subsystem tcs initialized
    [Mock TCSLog - 2018-05-17 15:37:17,866 - INFO] TCS subsystem virus initialized
    [Mock TCSLog - 2018-05-17 15:37:17,866 - INFO] TCS subsystem pfip initialized
    [Mock TCSLog - 2018-05-17 15:37:17,866 - INFO] TCS subsystem pas initialized
    [Mock TCSLog - 2018-05-17 15:37:18,048 - INFO] Database initialized and shot file stored
    [Mock TCSLog - 2018-05-17 15:37:18,049 - INFO] OCD ZeroMQ servers initialized
    Run OCD

### Replay the database

At this point, if you are listening to TCS, you should see more log messages as
events come in, states are modified and run shots.

Alternatively it is possible to replay a Sqlite3 database containing the TCS
event produced in one night, or a part of it. You can do it with:

    ocd db_replay --config ocd.cfg -s 10 databases/smaller.db

### Enable/disable hetdex

Finally, you need to tell OCD whether it is allowed to run shots or not. To do
this run:

    ocd allow_hetdex {start,stop} --config ocd.cfg

# lighetr_reduction
