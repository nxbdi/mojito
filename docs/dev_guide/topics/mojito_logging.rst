

=======
Logging
=======

Mojito has its own logging system. When you call ``Y.log`` from within your mojits, your log messages are intercepted and processed by Mojito. This allows you 
to create your own log formatting, writing, and publishing functions for both your client and server runtimes. It also allows you to enable log buffering, 
so performance during crucial runtime periods is not adversely affected.

Log Levels
##########

Mojito has the following five log levels:

- ``DEBUG``
- ``INFO``
- ``WARN``
- ``ERROR``
- ``MOJITO``

All of them should be familiar except the last, which are framework-level messages that indicate that an important framework event is occurring (one that users might want to track).

Setting a log level of ``WARN`` will filter out all ``DEBUG`` and ``INFO`` messages, while ``WARN``, ``ERROR``, and ``MOJITO`` log messages will be processed. To see all 
log messages, set the log level to ``DEBUG``.

YUI Library Logs
################

By default, all log messages generated by the YUI library itself are processed. The log level filter is also applied to these messages, but within the Mojito log output, 
a "YUI-" identifier is added to them. So when YUI emits a ``WARN`` level log message, the Mojito logs will display a ``YUI-WARN`` log level. This helps differentiate between 
application messages and YUI framework messages.

YUI logs can be turned on and off for both server and client within an application's log configuration (see below).

Log Defaults
############

The server and client log settings have the following default values:

- ``level:`` ``DEBUG`` - log level filter.
- ``yui:`` ``true`` - determines whether YUI library logs are displayed.
- ``buffer:`` ``false`` -  determines whether logs are buffered.
- ``maxBufferSize: 1024`` - the number of logs the buffer holds before auto-flushing.
- ``timestamp: true`` -  log statements are given a timestamp if value is true.
- ``defaultLevel: 'info'`` - if ``Y.log`` is called without a log level, this is the default.

Log Configuration
#################

All the values above are configurable through the `log object <../intro/mojito_configuring.html#log-object>`_ in the ``application.json`` file. In the example ``application.json`` 
below, the ``log`` object has both ``client`` and ``server`` objects that override the defaults for ``level`` and ``yui``.

.. code-block:: javascript

   [
     {
       "settings": [ "master" ],
       "log": {
         "client": {
           "level": "error",
           "yui": false
         },
         "server": {
           "level": "info",
           "yui": false
         }
       },
       ...
     }
   ]

Mutator Log Functions
#####################

You can create different write function to change the format of log messages and control where the logs are written. The logger has functions for formatting, writing, 
and publishing log messages that can be provided by a Mojito application. The function names are defined by users. For example, you could name the log formatter 
either ``formatLogs`` or ``log_formatter``.

Custom Log Formatter
====================

The log formatter function accepts the log message, the log level, a string identifying the source of the log (usually the YUI module name emitting the log), a timestamp, 
and the complete ``logOptions`` object. The function returns a string, which is passed to the log writer.

.. code-block:: javascript

   function {log_formatter_name}(message, logLevel, source, timestamp, logOptions) {
     return "formatted message";
   }

Custom Log Writer
=================

The log writer function accepts a string and does something with it. You can provide a function that does whatever you want with the log string. The default log writer 
calls ``console.log``.

.. code-block:: javascript

   function {log_writer_name}(logMessage[s]) {}

.. note:: Your log writer function must be able to handle a string or an array of strings. If you have set buffered logging, it may be sent an array of formatted log messages.

Custom Log Publisher
====================

If a log publisher function is provided, it is expected to format and write logs. Thus, a log publisher function takes the place of the log formatter and the log writer functions 
and accepts the same parameters as the log formatter function.

.. code-block:: javascript

   function {log_publisher_name}(message, logLevel, source, timestamp, logOptions) {

Custom Log Functions on the Client
==================================

To provide custom log function on the client, you add the log function to a JavaScript asset that your application will load.

In the example JavaScript asset below, the log function ``formatter`` is first defined and then set as the log formatter function.

.. code-block:: javascript

   function formatter(msg, lvl, src, ts, opts) {
     return "LOG MSG: " + msg.toLowerCase() + " -[" + lvl.toUpperCase() + "]- (" + ts + ")";
   }
   YUI._mojito.logger.set('formatter', formatter);

Using the ``formatter`` function above, the log messages will have the following format:

``>LOG MSG: dispatcher loaded and waiting to rock! -[INFO]- (1305666208939)``

Custom Log Functions on the Server
==================================

On the server, you must add log mutator functions to ``server.js``, so that Mojito will set them as the log functions before starting the server.

In this example ``server.js``, ``writeLog`` writes logs to the file system.

.. code-block:: javascript

   var mojito = require('mojito'), fs = require('fs'), logPath = "/tmp/mojitolog.txt";
   function writeLog(msg) {
     fs.writeFile(logPath, msg, 'utf-8');
   }
   // You can access log formatter, writer, or
   // publisher for the server here.
   mojito.setLogWriter(function(logMessage) {
     writeLog(logMessage + '\n');
   });
   module.exports = mojito.createServer();

Log Buffering
#############

To avoid performance issues caused by logging, you can enable buffering, which will configure Mojito to cache all logs in memory. You can force Mojito to flush the logs with 
the ``Y.log`` function or setting the maximum buffer size. The following sections show you how to enable buffering and force Mojito to flush the cached logs.

Enable Buffering
================

To configure Mojito to buffer your logs,  set the ``buffer`` property to ``true`` in the ``log`` object as shown in the example ``application.json`` below.

.. code-block:: javascript

   [
     {
       "settings": [ "master" ],
       "log": {
         "client": {
           "buffer": true
         },
         "server": {
           "buffer": true
         }
       },
       ...
     }
   ]

Flush Cached Logs
=================

Mojito provides you with two ways to forcefully flush cached logs. When you have buffering enabled, you can force Mojito to flush the cached logs with ``Y.log(({flush: true})``. 
You can also set the maximum buffer size, so that Mojito will flush cached logs after the cache has reached the maximum buffer size.

In the example ``application.json`` below, the maximum buffer size is set to be 4096 bytes. Once the log cache reaches this size, the logs are then flushed. The default size of 
the log cache is 1024 bytes.

.. code-block:: javascript

   [
     {
       "settings": [ "master" ],
       "log": {
         "client": {
           "buffer": true,
           "maxBufferSize": 4096
         },
         "server": {
           "buffer": true,
           "maxBufferSize": 4096
         }
       },
       ...
     }
   ]


