Logentries
=========

A short role to add configuration of logentries

Requirements
------------

An account logentries

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

This configuration will generate 2 files in /etc/le/conf.d:

syslog.conf file will contains:

    [stage-syslog]
    path = /var/log/syslog
    destination = project-dev/monolith-syslog

project.conf file will contains:

    [stage-project-default-log]
    path = /var/www/project/stage/log/filename.log
    destination = myhostname/project-filename

    [stage-project-webserver-error.log]
    path = /var/log/webserver/error.log
    destination = myhostname/webserver/error.log

Use role in playbook

    - name: logentries
      hosts: all
      roles:
        - logentries
      tags:
        - logentries

License
-------

BSD

