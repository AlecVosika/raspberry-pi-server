# **Raspberry PI 4**

OS: Ubuntu Server 20.04

## **NginX Info**

- _/etc/nginx:_ The Nginx configuration directory. All of the Nginx configuration files reside here.
- _/etc/nginx/nginx.conf:_ The main Nginx configuration file. This can be modified to make changes to the Nginx global configuration.
- _/etc/nginx/sites-available/:_ The directory where per-site server blocks can be stored. Nginx will not use the configuration files found in this directory unless they are linked to the sites-enabled directory. Typically, all server block configuration is done in this directory, and then enabled by linking to the other directory.
- _/etc/nginx/sites-enabled/:_ The directory where enabled per-site server blocks are stored. Typically, these are created by linking to configuration files found in the sites-available directory.
- _/etc/nginx/snippets:_ This directory contains configuration fragments that can be included elsewhere in the Nginx configuration. Potentially repeatable configuration segments are good candidates for refactoring into snippets.

## **Server Logs**

- _/var/log/nginx/access.log:_ Every request to your web server is recorded in this log file unless Nginx is configured to do otherwise.

- _/var/log/nginx/error.log:_ Any Nginx errors will be recorded in this log.
