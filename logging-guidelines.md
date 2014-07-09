# Logging Guidelines

## Overview

This document provides guidelines for logging in Python on the Clinical and Translational Science Institute's Informatics and Technology ([CTS-IT][]) development team. It is **not** an in-depth tutorial on generic logging. For that, please familiarize yourself with the following documentation from the Python Sofware Foundation:
 
 - [Logging HOWTO][]
 - [Logging Cookbook][]

[CTS-IT]: http://it.ctsi.ufl.edu
[Logging HOWTO]: https://docs.python.org/howto/logging.html
[Logging Cookbook]: https://docs.python.org/howto/logging-cookbook.html

## Guideline #0
> The ```logging``` module is a stable dependency in Python; use it instead of re-inventing logging.

### Rationale
Python has a well thought out logging module in its standard library. Appropriately named ```logging```, it is configurable and customizable to handle our logging needs. Furthermore, Python developers from the community will be expecting it and it's very flexible in terms of extensibility and configuration. 

Despite ample documentation, it is often misused.

The primary function for the module is ```logging.log()```, but is rarely invoked directly. Instead, convenience functions whose names correspond to their logging levels are used. They are described in the following table:

***Table describing Logging Levels***

Level | When it’s used | Numerical Value | Function Name
-|
DEBUG | Detailed information, typically of interest only when diagnosing problems. | 10 | ```logging.debug()```
INFO | Confirmation that things are working as expected. | 20 | ```logging.info()```
WARNING | An indication that something unexpected happened, or indicative of some problem in the near future (e.g. ‘disk space low’). The software is still working as expected. | 30 | ```logging.warning()```
ERROR | Due to a more serious problem, the software has not been able to perform some function. | 40 | ```logging.error()```
CRITICAL | A serious error, indicating that the program itself may be unable to continue running. | 50 | ```logging.critical()```
NOTSET | Disable logging | 0 | *n/a*

You can set a level at which you wish to log. Everything at that level or greater, as indicated by the "Numerical Value" column, will be logged. DEBUG is lower than the rest. So, setting the logging level to DEBUG will include *all* messages. 

Note: **WARNING** is the default level.

Caveat: **NOTSET** is treated special in that it's numerically the lowest, but disables logging completely. Instead of logging all messages whose level is greater than zero, *no* messages are logged.

In additional to levels, just about every other aspect of logging such as formatting and destination can be configured or extended through inheritance. Explore the [Logging Cookbook][] for common scenarios.

## Guideline #1
> Use ```logging``` or the [extended form of ```print```][chevron] instead of plain ```print```.

### Rationale

Using the non-extended form of ```print``` couples your code to the command line, since it'll write to the standard output by default. Using ```logging``` and the [extended form of ```print```][chevron], however, provide a more flexible solution.

If you're considering using ```print``` to provide a User with information, use ```info()``` or ```debug()```. A Logger can be configured to write to standard out using a ```logging.StreamHandler```.

When using ```print``` for data, define your function with a default parameter and pass the file-like argument to the [extended form][chevron].

***DO NOT*** use the non-extended form of ```print``` in a library module.

[chevron]: https://docs.python.org/2/reference/simple_stmts.html#print

### Example: Using the extend form of print
    import logging
    import sys
    import tempfile
    
    def write_record(data, output=sys.stdout):
        for key,value in data.iteritems():
            logging.info("Generating record for: %s", key)
            # use the extended-form of print
            print >>output, key, ':', value
            # or use the write() method directly
            output.write(key + ' : ' + str(value) + '\n')
    
    # we can call write_record and have it use stdout by default
    write_record({'foo': False, 'bar': 42})
    # or specificy which file to write to
    with tempfile.TemporaryFile() as fp:
        write_record({'foo': False, 'bar': 42}, fp)


## Guideline #2

> Configure your logger to have two handlers:
> 
> - one for file logging, set to DEBUG; and
> - one for the command line, whose verbosity is configureable by a switch or setting specified by the User.

### Rationale

The two handlers each serve a distinct purpose. The file log is primarily beneficial for debugging. The command line output is beneficial to a user who is interacting with our programs, even if its in a read-only manner such as viewing transfer progress.

### Example Implementation
    import getpass
    import logging
    import sys
    
    # set up file logging
    logging.basicConfig(level=logging.DEBUG,
                        format='%(asctime)s %(name)s: %(levelname)s: %(message)s',
                        filename='myapp.log',
                        filemode='w')
    
    # Create a console handler
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.INFO)
    
    # make the verbosity configurable: python example.py --verbose
    if '--verbose' in sys.argv:
        console_handler.setLevel(logging.DEBUG)
    
    # set a format more terse for a user interacting with our program
    formatter = logging.Formatter('%(relativeCreated)-15s %(levelname)+8s: %(message)s')
    console_handler.setFormatter(formatter)
    
    # add the handler to the root logger
    logging.getLogger().addHandler(console_handler)
    
    # Examples of using our configuration
    logging.info("Program started.")
    logging.debug("Current user: %s", getpass.getuser())
    logging.warn("Program does nothing!")
    logging.info("Program ended.")

*Console output from* ```python example.py```

    0.606060028076      INFO: Program started.
    0.802993774414   WARNING: Program does nothing!
    0.868082046509      INFO: Program ended.

*Console output from* ```python example.py --verbose```

    0.617027282715      INFO: Program started.
    0.75888633728      DEBUG: Current user: admin
    0.830888748169   WARNING: Program does nothing!
    0.893115997314      INFO: Program ended.

[Logging HOWTO: When to use Logging]: https://docs.python.org/howto/logging.html#when-to-use-logging

## Guideline #3

> Report on normal operations using  ```info()``` and supplement them with details using ```debug()```.

We want to provide enough information in our logs so that we can diagnose what has gone wrong with our programs, much like an airplane's black-box flight recorder records information for use in the event of an investigation.

To that end, use a combination of ```info()``` and ```debug()``` to report normal behavior. If one were to look at just ```info()``` messages, they should see an easy-to-follow storyline of normal application behavior. 

Use ```debug()``` to add more details to messages. In this way, ```info()``` acts as a well-written subject line of an email, and ```debug()``` the contents of that email.

### Example
    import logging
    import ctsit.project42 as lib
    
    # do very important things including configure logging 
    # then...
    logging.info('File upload complete.')
    logging.debug('Filename: %s', FILE_NAME)
    logging.debug('Server: %s', SERVER)
    
    logging.info('Sending email notification to %s', config['admin_email'])
    lib.send_email()
    logging.info('Email sent to %s', config['admin_email'])
    
    logging.info('All done.')

Looking at just the ```INFO``` messages, the log reads:

	 ...
    INFO:root:File upload complete.
    INFO:root:Sending email notification to leptodactylus@ranarium
    INFO:root:Email sent to leptodactylus@ranarium
    INFO:root:All done.

With the ```DEBUG``` messages, the log reads:

    ...
    INFO:root:File upload complete.
    DEBUG:root:Filename: frogs.dat
    DEBUG:root:Server: ranarium
    INFO:root:Sending email notification to leptodactylus@ranarium
    DEBUG:root:SMTP Server: smtp.ranarium
    DEBUG:root:SMTP Port: 50000
    INFO:root:Email sent to leptodactylus@ranarium
    INFO:root:All done.

### Example 2
    def reticulating_splines(items):
    	 logger.info('Reticulating splines')
        
        for i, item in enumerate(items):
            # do some complex algorithm computation
            logger.debug('%s iteration, item=%s', i, item)
    	 
    	 logger.info('Splines reticulated')

## Guideline #4

> Detail exceptions using ```logging.exception()```.

## Rationale

Exceptions indicate that something has gone wrong, one of the main reasons we read the logs. Therefore, log them and their tracebacks.

Note: ```logging.exception(msg, *args)``` is equivalent to ```logging.error(msg, exc_info=True, *args)```.

## Example
*Code*

    try:
        open('/path/to/missing/file', 'rb')
    except IOError as error:
        logging.exception('Failed to open file "/path/to/missing/file"')
        raise

*Log*                

    ERROR:root:Failed to open file "/path/to/missing/file"
    Traceback (most recent call last):
      File "<stdin>", line 3, in fail
    IOError: [Errno 2] No such file or directory: '/path/to/missing/file'

## Guideline #5
> Logger should be configured once per application; as close as possible to the program's entry point—usually the ```main()``` method.

Set it and forget it!


***DO NOT*** configure logging in libraries and other modules.

## Resources

* https://docs.python.org/howto/logging.html
* https://docs.python.org/howto/logging-cookbook.html
* http://victorlin.me/posts/2012/08/26/good-logging-practice-in-python
* http://www.robg3d.com/2012/02/python-logging-best-practices/

