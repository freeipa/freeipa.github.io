Kibana
======

Introduction
------------

This page covers my personal findings when trying to get FreeIPA logs
sent through to our central log consolidation infrastructure.

Requirements
------------

These IPA servers are running RHEL 6.6. I have a pair of Logstash
servers to receive and parse logs and a cluster of 5 ElasticSearch
database servers. Kibana is used as the interface to view log data from
the ES cluster.



Base Log Sending
----------------

The basic rsyslog configuration is modified by adding the file
/etc/rsyslog.d/logstash.conf

``*.* @logserver.example.com:5544``

This will get rsyslog sending everything it receives to logserver via
UDP on port 5544.



Advanced Log Sending
--------------------

Whilst this will work perfectly well for sending basic logs via UDP, for
bonus points I'm actually doing the following on our Dev IPA servers.
These servers have the basic rsyslog package replaced by rsyslog7 which
is rsyslog version 7.4.10. This allows us to do some more complex
logging to assist the central log servers. I create the
/etc/rsyslog.d/logstash.conf file, which looks as follows:

::

    template(name="ls_json" type="list" option.json="on")
      { constant(value="{")
        constant(value="\"@timestamp\":\"")         property(name="timegenerated" dateFormat="rfc3339")
        constant(value="\",\"@version\":\"1")
        constant(value="\",\"message\":\"")         property(name="msg")
        constant(value="\",\"host\":\"")            property(name="fromhost")
        constant(value="\",\"host_ip\":\"")         property(name="fromhost-ip")
        constant(value="\",\"my_environment\":\"Development")
        constant(value="\",\"my_project\":\"IPA")
        constant(value="\",\"my_use\":\"Auth")
        constant(value="\",\"logsource\":\"")       property(name="fromhost")
        constant(value="\",\"severity_label\":\"")  property(name="syslogseverity-text")
        constant(value="\",\"severity\":\"")        property(name="syslogseverity")
        constant(value="\",\"facility_label\":\"")  property(name="syslogfacility-text")
        constant(value="\",\"facility\":\"")        property(name="syslogfacility")
        constant(value="\",\"program\":\"")         property(name="programname")
        constant(value="\",\"pid\":\"")             property(name="procid")
        constant(value="\",\"rawmsg\":\"")          property(name="rawmsg")
        constant(value="\",\"syslogtag\":\"")       property(name="syslogtag")
        constant(value="\"}\n")
      } 
    
    *.* @@logstash01.example.com:5500;ls_json
    $ActionExecOnlyWhenPreviousIsSuspended on
    & @@logstash02.example.com:5500;ls_json
    & /var/log/localbuffer
    $ActionExecOnlyWhenPreviousIsSuspended off

This lets us send the log in JSON format, whilst adding the extra fields
my_environment, my_project, and my_use. You don't need these, but they
will help separate out different log types from the log-viewing UI. Our
network has, for example, 2 IPA servers in Development, with a further 8
in Production. These extra fields allow us to pull out only the logs
that match "my_environment=Dev" AND "my_project=IPA".

The other rsyslog properties that are sent are my current best-guess at
what I need when reviewing the logs through the Kibana UI.



Getting dirsrv Logs Sending
---------------------------

The best way I have so far of doing this is to add another file in the
/etc/rsyslog.d directory which configures rsyslog to look at external
log files. /etc/rsyslog.d/dirsrv.conf contains the following:

| ``module(load="imfile" PollingInterval="2")``
| ``input(type="imfile"``
| ``      File="/var/log/dirsrv/slapd-EXAMPLE-COM/access"``
| ``      Tag="dirsrv"``
| ``      StateFile="statedirsrv"``
| ``      Facility="local0")``
| ``input(type="imfile"``
| ``      File="/var/log/dirsrv/slapd-EXAMPLE-COM/errors"``
| ``      Tag="dirsrv"``
| ``      StateFile="statedirsrverr"``
| ``      Severity="error"``
| ``      Facility="local0")``

This uses the imfile module in rsyslog to monitor changes in the dirsrv
access and errors log files. The log files are polled every 2 seconds
(although I know dirsrv also buffers, so I should change this) and the
new log data is pulled in by rsyslog and then sent on to the Logstash
server(s) via the logstash.conf file above.



Logstash Configuration
----------------------

Logstash 1.4.2 is installed via RPM and configured to accept log data.
Create file /etc/logstash/conf.d/syslog.conf with the following
contents:

| ``input {``
| ``  syslog {``
| ``    type => syslog``
| ``    port => 5544``
| ``  }``
| ``  tcp {``
| ``    type => syslogjson``
| ``    port => 5500``
| ``    codec => "json"``
| ``  }``
| ``}``
| ``filter {``
| ``  # This replaces the host field (UDP source) with the host that generated the message (sysloghost)``
| ``  if [sysloghost] {``
| ``    mutate {``
| ``      replace => [ "host", "%{sysloghost}" ]``
| ``      remove_field => "sysloghost" # prune the field after successfully replacing "host"``
| ``    }``
| ``  }``
| ``  if [type] == "syslog" {``
| ``    grok {``
| ``      patterns_dir => "/opt/logstash/patterns"``
| ``      match => { "message" => "%{FWGROK}" }``
| ``      match => { "message" => "%{AUDITAVC}" }``
| ``    }``
| ``  }``
| ``  if [type] == "syslogjson" {``
| ``    grok {``
| ``      patterns_dir => "/opt/logstash/patterns"``
| ``      match => { "message" => "%{FWGROK}" }``
| ``      match => { "message" => "%{AUDITAVC}" }``
| ``      match => { "message" => "%{COMMONAPACHELOG}" }``
| ``      tag_on_failure => []``
| ``    }``
| ``  }``
| ``  # This filter populates the @timestamp field with the timestamp that's in the actual message``
| ``  # dirsrv logs are currently pulled in every 2 minutes, so @timestamp is wrong``
| ``  if [syslogtag] == "dirsrv" {``
| ``    mutate {``
| ``      remove_field => [ 'rawmsg' ]``
| ``    }``
| ``    grok {``
| ``      match => [ "message", "%{HTTPDATE:log_timestamp}" ]``
| ``    }``
| ``    date {``
| ``      match => [ "log_timestamp", "dd/MMM/YYY:HH:mm:ss Z"]``
| ``      locale => "en"``
| ``      remove_field => [ "log_timestamp" ]``
| ``    }``
| ``  }``
| ``}``
| ``output {``
| ``  elasticsearch {``
| ``    protocol => node``
| ``    node_name => "Indexer01"``
| ``  }``
| ``}``

This instructs Logstash to listen on port 5544 for basic log data, and
also on port 5500 for JSON formatted data. The FWGROK and AUDITAVC lines
force Logstash to run 2 bespoke grok filters on the data to get iptables
and auditavc lines into better shape.

The section for "dirsrv" is there to force Logstash to replace the
incoming timestamp for dirsrv data (which will be based on when rsyslog
first saw the data - and is therefore next to useless) with the
timestamp that appears in the actual log line. This is an improvement,
but will only be to the resolution of 1 second.

Issues
------

#. dirsrv logs are timestamped with a resolution that allows dozens of
   log lines to share the same timestamp. Increased resolution of
   timestamp from dirsrv would help fix this.
#. An unwanted side-effect at the moment is that the dirsrv logs are
   written to /var/log/messages as well. This needs fixing, but the main
   aim here has been to get the logs onto a remote server.
#. Needs further thought with regards the rsyslog properties that are
   passed in the JSON template.
#. Failure of both logstash servers will result in logs writing to
   /var/log/localbuffer, where they will simply remain. This is
   sub-optimal.