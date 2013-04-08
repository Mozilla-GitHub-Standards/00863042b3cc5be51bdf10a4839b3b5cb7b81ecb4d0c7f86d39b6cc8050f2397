# SASL BrowserID #
SASL BrowserID is a new [SASL mechanism](http://asg.web.cmu.edu/sasl/sasl-library.html) for client and servers who want to authenticate using BrowserID.

Who da what?

SASL stands for Simple Authentication and Secruity Layer. It is a standardized API for re-using authentication mechanisms.

[BrowserID](https://browserid.org) is an open web standard for providing a verified email address to websites for authentication.

This project aims to provide a plugin written in C for the popular CMU Cyrus SASL API Implementation. This can be used by:

* OpenLDAP directory server
* Email servers (CMU, postfix, etc)
* ??? Tell us other use cases!

## Status ##
This was successfully used in Production on a Mozilla website, but that site has 
moved off LDAP, so no active use within Mozilla currently.

## Security Notes ##

Make sure the client and server applications recognize and 
[block unknown email addresses](docs/security_block_unknown_email.md)!

## Quick Start ##

    vagrant up
    vagrant ssh

If you have vagrant install, this will give you a fully working setup to play
around with or hack on.

Otherwise...

## Requirements ##
This plugin is under development on i686 Ubuntu 10.04 with:

* Cyrus SASL 2.1.23 or later
* OpenLDAP 2.4.23 or later
* libcurl
* [yajl](https://github.com/lloyd/yajl) 2.0.2 or later
* MySQL client libraries for C

### Ubuntu Tips ###
1) sudo aptitude install ruby cmake automake libcurl-dev libmysqlclient-dev libsasl2-dev libcurl4-gnutls-dev

ruby and cmake are only needed to compile yajl.

automate is only needed to compile sasl-browserid.

2) Compile [yajl](https://lloyd.github.com/yajl/)

We want yajl 2.0.2 or greater, which most distros haven't packaged.

    wget http://github.com/lloyd/yajl/tarball/2.0.2 -O yajl-2.0.2.tar.gz
    tar zxvf yajl-2.0.2.tar.gz
    cd lloyd-yajl-g5b0e7df
    ./configure
    sudo make install
    ldconfig /usr/local/lib

## Install SASL-BrowserID ##

Assuming you have the requirements installed, you can:

    ./configure
    make
    sudo make install

This will create libbrowserid plugins under /usr/lib/sasl2

Configure any sasl enabled servers. Example: see configs/slapd.conf.fragment.txt

Restart any sasl enabled servers, such as slapd.

Configuration details can be found in the INSTALL doc or 

    ./configure --help

## Setup MySQL Session ##

    $ mysql -uroot -p
    mysql> create database sasl_browserid;
    $ exit
    $ mysql -u root -p sasl_browserid < configs/browserid_session.ddl

You know how a BrowserID enabled server. Next let's make sure things are working...

## Sanity Tests ##

The following are ways to test this plugin.

For all of the following tests, it's best to watch syslog

    sudo tail -f /var/log/auth.log

There are 3 ways to test, pluginviewer, slapd, and sample program.

### pluginviewer ###

*Note:* On some systems this is called pluginviewer.

    sudo saslpluginviewer

Do you see BROWSER-ID in the list of SASL client and server mechanisms?

### Test values for assertion and audience ###

Where tests below require an Assertion and Audience, use browserid_debug.html and a local webserver. Example:

    $ cd test/www/
    $ python -m SimpleHTTPServer 8001

Point your browser at http://locahost:8001/browserid_debug.html

    Assertion: eyJhbGci...blah...blah...2mtVg68723mlBPAQds_bPsG8mllYg
    Audience: localhost:8001

### OpenLDAP (slapd) ###

Setup:

    sudo cp configs/slapd.conf /usr/lib/sasl2
    # restart slapd
    cd test/www/

Each time

    # request http://localhost:8001 in your browser and do BID login flow
    A=eyJjZXJ0aWZpY2F0ZXMiOlsiZXlKaGJHY2...some_real_assertion_ekdNeGhGM05CM3diUXV6UzC
    ldapwhoami -Y BROWSER-ID -I -X $A -U 'localhost:8001'

### Sample client and server ###
If you've compiled SASL's sample/client and sample/server programs...

    sudo cp configs/sample.conf /usr/lib/sasl2
    cd ${SASL_SRC}
    ./sample/server -p 8089 -s testing -m BROWSER-ID
    ./sample/client -p 8089 -s testing -m BROWSER-ID localhost


## License ##
plugins/plugin_common.c and plugins/plugin_common.h are copied from [CMU's Cyrus SASL distribution](http://ftp.andrew.cmu.edu/pub/cyrus-mail/).
They are copywrite CMU and licensed per file. See files for details.

The rest of this codebase is original and Copywrite Mozilla Corporation 2011.
A License is TBD.

We'll pick a license that works well with Cyrus SASL distributions and balances other factors.
