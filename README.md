Logentries
=========

[![Build Status](https://travis-ci.org/inovia-team/ansible-logentries.svg?branch=master)](https://travis-ci.org/inovia-team/ansible-logentries)

---

A role to add configuration of logentries and allow to format log

Requirements
------------

An account [logentries](https://logentries.com/)

Role Variables
--------------

    logentries_account_key: "your-logentries-account-key"
    logentries_stage: production
    logentries_logs:
      config_filename1:
        - {
            logentries_project: "syslog",
            logentries_files_on_server: "/var/log/syslog",
            logentries_destination_on_logentries: "project-dev/monolith-syslog"
        }
      config_filename2:
        - {
            logentries_project: "section_name",
            logentries_files_on_server: "logfile1",
            logentries_destination_on_logentries: "log_name_on_logentries"
        }
        - {
            logentries_project: "section_name2",
            logentries_files_on_server: "logfile2",
            logentries_destination_on_logentries: "log_name_on_logentries"
        }
    logentries_formatters:
      format:
        - {
          method_name: "format_syslog_log",
          regexp: "regex",
          output_line: "'%s %s %s %s %s'%(self.token, self.identity, self.hostname, self.log_name, line)"
        }
      formatters:
        - {
          logentries_project: "syslog",
          method_name: "format_syslog_log",
        }

`method_name` in section `logentries_formatters.formatters` *MUST* be the same as
`method_name` in `logentries_formatters.format` it's the link between the logs and how to
format it. Same for `logentries_project` which *MUST* be the same as
`logentries_project` in section `logentries_logs`

The format of output line should be formatted with mandatory parts,
`output_line` must have at least

    "%s %s %s %s"%(self.token, self.identity, self.hostname, self.log_name)

The result of parsing line with regexp is stored in variable `pl`

    (?P<firstpart>^[^{]*)(?P<secondpart>{.*})(?P<lastpart>.*)

All parts of this regexp will be stored as an array and will be accessible:

    pl['firstpart']
    pl['secondpart']
    pl['lastpart']

Example Playbook
----------------

Add variables in inventory

    logentries_account_key: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    logentries_stage: {{ansible_hostname}}
    logentries_logs:
      syslog:
        - {
            logentries_project: "syslog",
            logentries_files_on_server: "/var/log/syslog",
            logentries_destination_on_logentries: "myhostname/project-syslog"
        }
      project:
        - {
            logentries_project: "project-default-log",
            logentries_files_on_server: "/var/www/project/stage/log/filename.log",
            logentries_destination_on_logentries: "myhostname/project-filename"
        }
        - {
            logentries_project: "project-webserver-error.log",
            logentries_files_on_server: "/var/log/webserver/error.log",
            logentries_destination_on_logentries: "myhostname/webserver/error.log"
        }
    logentries_formatters:
      format:
        - {
          method_name: "format_webserver_log",
          regexp: "regexp",
          output_line: "'%s %s %s %s %s'%(self.identity, self.hostname, self.log_name, self.token, line)"
        }
        - {
          method_name: "format_syslog_log",
          regexp: "(?P<firstpart>^[^{]*)(?P<secondpart>{.*})(?P<lastpart>.*)",
          output_line: "'%s{\"identity\":\"%s\", \"hostname\":\"%s\", \"appname\":\"%s\", \"data\": %s, \"metadata\":\"%s\", \"lastpart\":\"%s\"}'%(self.token, self.identity, self.hostname, self.log_name, pl['secondpart'], pl['firstpart'], pl['lastpart'])"
        }
      formatters:
        - {
          logentries_project: "syslog",
          method_name: "format_syslog_log",
        }
        - {
          logentries_project: "project-webserver-error.log",
          method_name: "format_webserver_log",
        }

This configuration will generate 2 files in `/etc/le/conf.d`:

`syslog.conf` file will contains:

    [stage-syslog]
    path = /var/log/syslog
    destination = project-dev/monolith-syslog

`project.conf` file will contains:

    [stage-project-default-log]
    path = /var/www/project/stage/log/filename.log
    destination = myhostname/project-filename

    [stage-project-webserver-error.log]
    path = /var/log/webserver/error.log
    destination = myhostname/webserver/error.log

If you add formatting part this will add 2 others files `__init__.py` and
`formatters.py` in folder `/etc/le/le_formatters`

In `formatters.py` you will have a list of methods:

    import re

    class Form(object):
        """Formats lines based of pattern given."""

        def __init__(self, identity, hostname, log_name, token):
            self.identity = identity
            self.hostname = hostname
            self.log_name = log_name
            self.token = token

        def format_syslog_log(self, line):
            line = line.rstrip()
            m = re.match(r"(?P<firstpart>^[^{]*)(?P<secondpart>{.*})(?P<lastpart>.*)", line)
            if m:
                pl = m.groupdict()
                return '%s{"identity":"%s", "hostname":"%s", "appname":"%s", "data": %s, "metadata":"%s", "lastpart":"%s"}'%(self.token, self.identity, self.hostname, self.log_name, pl['secondpart'], pl['firstpart'], pl['lastpart'])
            else:
                return line

        def format_webserver_log(self, line):
            line = line.rstrip()
            m = re.match(r"regexp", line)
            if m:
                pl = m.groupdict()
                return '%s %s %s %s %s'%(self.identity, self.hostname, self.log_name, self.token, line)
            else:
                return line

    formatters = {
        'development-syslog': lambda hostname, log_name, token: Form('development-syslog', hostname, log_name, token).format_syslog_log,
        'development-mobilities-default': lambda hostname, log_name, token: Form('development-mobilities-default', hostname, log_name, token).format_webserver_log,
    }


Use role in playbook
--------------------

    - name: logentries
      hosts: all
      roles:
        - ansible-logentries
      tags:
        - logentries

License
-------

BSD

