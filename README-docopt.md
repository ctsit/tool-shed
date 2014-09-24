# docopt

## Introduction

**doctopt** helps in defining an interface for any command-line application.

## Implementation
By using *docopt*, we can eliminate the difficulty of many of conventional argument parsers. For adopting to *docopt* just write the following in the comments of the code file.

1. Application name
2. Usage
3. List of options

## Installation

[Github repo](https://github.com/docopt/docopt)

	sudo easy_install docopt

## Example
"""

REDi - Converter from raw clinical data in XML format to REDCap API data

Usage:
	
	redi.py -h | --help
	redi.py [-v] -c=<config-path> -k [-e] [-d=<is_it_dryrun>]

Options:

    -h --help                   show this help message and exit
    -v --verbose                Increase verbosity of output
    -c --config-path=<path>     Specify the path to the configuration directory
    -k --keep                   Specify `yes` to preserve the files generated during execution
    -e --emrdata                Specify `yes` to get EMR data
    -d --dryrun=<is_it_dryrun>  To execute redi.py in dry run state. This is to be
                                able to test each release by doing a dry run, where
                                the data is fetched and processed but not transferred
                                to the production REDCap. Email is also not sent. The
                                processed data is stored as output files under the
                                "out" folder under project root. No need to use -k or
                                provide any input when -d is used. [default: yes]
"""

from docopt import docopt

print docopt ( \__doc__, version= 'Redi v1.0')

## Explanation of example

* Start the comments
* 'REDi' : it is the application name
* Usage:  This prints the usage guide of the application, when given a wrong argument
* Options  : This is the list of the options that the application 'image.py' can be called with.

	*redi.py -h | --help* : indicates to display the help with all options available
	*redi.py [-v] -c=<config-path> -k [-e] [-d=<is_it_dryrun]* : the options in *[]* indicates it is an optional one. but the option *-k* indicates that it is a mandatory option and its default value is defined against its option description.


## References

* <http://docopt.org>
* <http://try.docopt.org>