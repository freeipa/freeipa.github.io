Time-Based_Account_Policies
===========================

Overview
========

FreeIPA is currently missing any temporal settings in the HBAC rules.
However, handling access to a host in repeating time periods might be a
desirable feature. The administrator of a certain environment should be
able to set the time a host should be accessed in either the host local
time, a certain time zone time or in UTC. Host-local-time policies would
allow to adapt the time a host can be accessed to the host's movement
along different time zones. A time bound to a certain time zone is more
transparent than local time as it doesn't change with the host
travelling. Sometimes, it may also be appropriate to set time in UTC.
This is rather strict setting that does not reflect daylight saving
time.



Use Cases
=========

A host or service should only be accessed at certain times given by a
certain time zone. This access is repeated in a way, such as three times
a week the same time every other month.

Design
======

The time based account policies are an extension mostly to the current
HBAC rule plugin. As the time rules may find use somewhere else in the
system (Sudo rules, for example), they should be designed with that in
mind.



Time Scenarios
--------------

This extension is based on the `iCalendar
format <http://tools.ietf.org/html/rfc5545>`__. It therefore understands
time in three different views. These are: host local time, time at a
certain time zone, and UTC.



Host Local Time
----------------------------------------------------------------------------------------------

Host local time approach is meant for those hosts that are most likely
to be found across different time zones and for some reason it's
important that the time they can be accessed reflects their current
position. This helps creating only a single HBAC rule instead of
multiple when only time zone or UTC rules would apply. The time of a
host is counted using the ``/etc/localtime`` information of the certain
host. Testing such a rule (in HBAC Test) requires the tester to specify
a certain time zone the rule would be tested against. It's important to
note that this type of policy may bring some unexpected behavior as host
may appear in different time zones, or in case of a hostgroup there
could be hosts from multiple different time zones, and administrator
should be very sure they want to use this.



Time Zones
----------------------------------------------------------------------------------------------

In this approach, the time is thought of as of a time at a certain time
zone. This might be appropriate when the time settings should reflect a
certain time zone, e.g. the host or the users connecting to it are to be
found in that certain time zone. The time conversion between host and
user time zones is calculated by the *libical* library. Daylight saving
time is taken into account.

UTC
----------------------------------------------------------------------------------------------

Sometimes the rules should apply for a certain time that is the same for
the whole globe throughout the year. That's why UTC is also supported.

Implementation
==============



Time Policies Storage
---------------------

It should be possible to create time rule objects so that the same time
rule could be referenced by other LDAP objects (at least by HBAC Rule
objects for a start). Therefore, a new container for these objects with
its own permissions should be created. The default permissions to access
this container should be set to the administrators of the objects that
will have access to the time rules.

The time rule object is a simple object with *cn*, *description*,
*accesstime* and *memberof* attributes. In *cn*, the name of the object
is stored, the **single-valued** *accesstime* stores the iCalendar
string that describes the time periods the rule applies to, and
*memberof* is the common attribute for handling n <-> n relationships.

The change to the schema looks like this:

``objectClasses: (2.16.840.1.113730.3.8.12.37 NAME 'ipaTimeRule' SUP top STRUCTURAL MUST ( cn $ accessTime ) MAY ( memberOf $ description ) X-ORIGIN 'IPA v4.4')``

Also, there need to be changes to the directory structure:

| ``dn: cn=timerules,$SUFFIX``
| ``changetype: add``
| ``objectClass: top``
| ``objectClass: nsContainer``
| ``cn: timerules``



HBAC Rule LDAP object changes
-----------------------------

There is a slight change to how HBAC rules are stored. To be able to
handle backward compatibility more easily, a new object called
*ipaHBACRulev2* is introduced. This object has the same attributes as
the old *ipaHBACRule* object with the addition of a *memberTimeRule*
attribute. The creation of *ipaHBACRulev2* should be helpful for
additional changes to the HBAC rule LDAP object in near future (e.g.
http://www.freeipa.org/page/V4/URI-based_HBAC). It will also cause that
the **older clients won't list these new objects** as HBAC rules which
means these rules automatically don't apply for these older clients.

Upon addition of a time rule to an HBAC rule, the HBAC rule's
*objectClass* switches to the new *ipaHBACRulev2*, when there are no
time rules present it switches back.

The new schema for the new HBAC Rule type:

| ``attributeTypes: (2.16.840.1.113730.3.8.11.76 NAME 'ipaMemberTimeRule' DESC 'Reference to a time rule describing some period of time' SUP distinguishedName EQUALITY distinguishedNameMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 X-ORIGIN 'IPA v4.4' )``
| ``objectClasses: (2.16.840.1.113730.3.8.12.38 NAME 'ipaHBACRuleV2' SUP ipaAssociation STRUCTURAL MAY ( serviceCategory $ memberService $ externalHost  $ ipaMemberTimeRule ) X-ORIGIN 'IPA v4.4' )``



iCalendar strings restrictions
------------------------------

The iCalendar standard format is very rich in features and allows
extensibility. As such, the available parsers (*python-icalendar*,
*libical*) don't do much of a strict parsing and it seems neither sane
nor advisable to try and introduce strict parsing for any iCalendar
string in FreeIPA/SSSD.

Instead of controlling the whole string, the server-side should do
strict parsing of at least the *VCALENDAR* and *VEVENT* iCalendar
components for the input iCalendar files, ignoring format correctness of
the rest. The client side should just obtain the *VEVENT* components
from a received iCalendar string, get values of the properties required
for proper evaluation and check whether these meet the requirements for
correct evaluation.

Furthermore, for the **first version**, we make additional restrictions.
The iCalendar strings stored on server side should strictly only contain
**one** *VEVENT* component in the root *VCALENDAR* component. The **only
supported properties** of this *VEVENT* component are:

-  *DTSTART*
-  *DTEND*
-  *DURATION*
-  *RDATE*
-  *RRULE*

These restrictions exist for the feasibility of setting the certain time
policies from either CLI or WebUI.



Server side
-----------

The server side should be able to receive iCalendar files and strings
and validate them according to the above restrictions. It should also
give means to generating iCalendar strings based on user input from the
CLI and WebUI. This should be performed using the options at addition
and modification commands.

**New dependency:** *python-icalendar* will be used for parsing and
creating the iCalendar strings.



SSSD side
---------

SSSD will be enforcing the time rules. To do that, it will need to
handle parsing the iCalendar strings - *libical* C library is used for
that. SSSD evaluator should go through the *VEVENT* components and check
if the current time falls into the time span defined by these
*VEVENT*\ s.

For the **first version** only systems that offer means for
non-complicated programmable current time zone retrieval will be
supported. This means Red Hat and Debian based systems.

**New dependency:** *libical* library will be used to handle parsing of
the iCalendar strings. It will also be used to generate recurrence of
*RRULE* property of iCalendar strings to help evaluate the comparison
against the current time.



Feature Management
==================

There are multiple ways how to associate an iCalendar string with a time
rule object:

#. Use options of the addition/modification commands (preferred way)
#. Add it using an escaped iCalendar string
#. Use a file generated by an external tool

The time rules should also offer a way to test whether they apply for a
given time similarly to what *hbactest* module does. The *hbactest*
module should also be extended to allow testing whether an HBAC rule
applies at a given time.

These possibilities should be reflected both in the WebUI and CLI.

UI

A new page will need to be created for listing and creation of time rule
objects. The creation page should allow creation/modification of a time
rule using the parameters of the according addition/modification
commands. It should also allow upload of an iCalendar file or direct
iCalendar string addition. For the modification of the *RRULE* iCalendar
property some code of this 3rd party solution might be helpful:
http://jkbrzt.github.io/rrule/.

The WebUI should show warning of some kind when modifying a time rule
that belongs to one or more HBAC rules.

The UI of HBAC rules needs changing as well. It should now include a new
section for addition of time rules, similar to the user, host and
service sections. User should be able to add more time policies for an
HBAC rule by the name of the policy.

CLI

CLI will need to introduce new commands for the addition of the time
rules as well as adding these newly created rules to HBAC rules.

+--------------------------+------------------------------------------+
| Command                  | Options                                  |
+==========================+==========================================+
| timerule-add             | NAME [ --icalfile=file.ics \|            |
|                          | --time=escaped_icalstring \| OPTS ]      |
+--------------------------+------------------------------------------+
| timerule-mod             | NAME [ --icalfile=file.ics \|            |
|                          | --time=escaped_icalstring \| OPTS ]      |
+--------------------------+------------------------------------------+
| timerule-del             | NAME                                     |
+--------------------------+------------------------------------------+
| timerule-show            | NAME                                     |
+--------------------------+------------------------------------------+
| timerule-find            | [NAME]                                   |
+--------------------------+------------------------------------------+
| timerule-test            | --time=DTIME                             |
+--------------------------+------------------------------------------+
| hbacrule-add-timerule    | NAME --timerule=RULE_NAME                |
+--------------------------+------------------------------------------+
| hbacrule-remove-timerule | NAME --timerule=RULE_NAME                |
+--------------------------+------------------------------------------+
| hbactest                 | --time=DTIME                             |
+--------------------------+------------------------------------------+

where
``OPTS = [--``\ ```start`` <https://tools.ietf.org/html/rfc5545#section-3.8.2.4>`__\ ``=TIME] [--``\ ```end`` <https://tools.ietf.org/html/rfc5545#section-3.8.2.2>`__\ ``=TIME] | --``\ ```duration`` <https://tools.ietf.org/html/rfc5545#section-3.8.2.5>`__\ ``=DUR] [--``\ ```dates`` <https://tools.ietf.org/html/rfc5545#section-3.8.5.2>`__\ ``=DTLIST] [--``\ ```rrule`` <https://tools.ietf.org/html/rfc5545#section-3.8.5.3>`__\ ``=RRULE]``.
``TIME``, ``DUR``, ``DTLIST``, ``RRULE`` should be values formatted
according to `RFC5545 <http://tools.ietf.org/html/rfc5545>`__ for the
given iCalendar components. The RFC5545 value type (e.g.
``DATE, DATE-TIME``) is recognized automatically from the value format.

The ``DTIME`` values are formatted as the
`DATE-TIME <https://tools.ietf.org/html/rfc5545#section-3.3.5>`__ value
data type.

As one can see from the table the addition/modification commands take
one of *icalfile*, *time* or combination of iCalendar creation options.

*timerule-mod* and *timerule-show* should display all the HBAC rules
that are using them so that the user directly sees the impact of their
actions.

*timerule-del* should prevent deletion of a time rule should this time
rule be used in any HBAC rule to prevent security issues.

*hbactest* command should be extended with a compulsory option *--time*.



How to Use
==========

#. A user creates a time rule depending on what they have available

   -  iCalfile:
      ``ipa timerule-add someday --icalfile=myical05052016.ics``
   -  iCalstring:
      ``ipa timerule-add someday --time="BEGIN:VCALENDAR\nPRODID:Internet iCal generator\nVERSION:2.0\nMETHOD:REQUEST\nBEGIN:VEVENT\nDTSTAMP:20160406T112129Z\nDTSTART;VALUE=DATE:20160505\nUID:1@darkside.com\nEND:VEVENT\nEND:VCALENDAR"``
   -  Using options: ``ipa timerule-add someday --start=20160505``

#. Then, ``ipa hbacrule-add newRule SOMEOPTIONS`` for standard HBAC Rule
   creation.
#. Add the newly created time rule to the HBAC rule:
   ``ipa hbacrule-add-timerule newRule --timerule=someday``
#. From now on, the hosts/services are only accessible at the time
   described by the iCalendar string in *someday* time rule.