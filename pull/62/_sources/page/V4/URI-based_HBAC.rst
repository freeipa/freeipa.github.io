URI-based_HBAC
==============

Overview
--------

A feature allowing Host Based Access Control to decide based not only on
triplet (user,service,hostname), but also on URI of the requested
resource.



Use Cases
---------

In some services, it may be useful to base authorization decision on
what resource within the the service is being accessed.

For example, it is very common in web applications that different parts
of the web (with different URIs) are meant to be accessed by different
users: there is some public part, some user-only part, some part that
should be accessible to administrator only...

Consider a webserver with the following URIs:

::

   http://webserver/application/public-content
   http://webserver/application/login-only/all-users
   http://webserver/application/login-only/user/user1
   http://webserver/application/login-only/admin

Each of these parts needs different access rights; so far, HBAC only
supported deciding whether an authorized user is allowed to access the
whole non-public part of the web. Obviously, it is possible for the web
application to handle access rights of the users. It is, however, a good
idea to also handle this on level of service (Apache) for consistency,
centralization and security on multiple levels.

This feature is a RFE for ticket #5030.

Design
------

For this feature, we need to add the URI to HBAC rule in FreeIPA, make
SSSD able to determine authorization based on it, and give services some
way to communicate with SSSD in terms of URI-based HBAC.

Schema
----------------------------------------------------------------------------------------------

URI is a part of HBAC rule. It is interpreted as a prefix which decides
whether the HBAC rule that would otherwise match (if it wasn't for this
feature) matches (the URI attribute is a prefix of resource's URI) or it
does not (the URI attribute is not a prefix of resource's URI). FreeIPA
itself merely stores this attribute and allows its adding, deletion and
editation; it doesn't act based on it in any way. Because there are two
parts of URI with different interpretation and meaning (host, schema,
port VS path - the rest of URI), we store these two parts separately -
the first one as a case-insensitive UTF-8 string and the second as
case-sensitive UTF-8 string.

SSSD is a daemon running on service's local machine and deciding based
on HBAC rules. It is also possible to use other approaches, for example
the application can ask FreeIPA's LDAP and decide itself based on
provided data. In case of SSSD, there is already HBAC capability - we
need to ensure it becomes URI-aware and it gets important data from both
sides - the URI prefix from FreeIPA and the URI to match from the
service requesting authorization. We can achieve this using
already-existing channels improved to also transfer URI. The prefix is
an attribute of HBAC rule and can be retrieved from FreeIPA's LDAP using
LDAP provider (URI would be accessible by anyone who has access to the
HBAC rule). There are multiple ways of getting the accessed URI from the
service.

By using SSSD, it is not required for the service requsting
authorization to be able to decide based on authorization data it
obtains itself. While there are some minimal changes necessary (in case
of Apache, an authorization module must exist), the service merely needs
to ask for authorization (providing, user, hostname, service, URI) and
receive result.

There are multiple ways of communicating with SSSD, PAM and Infopipe
were considered. As there are already some applications using PAM and it
seems to be a good interface for this use case, SSSD expects the
application to use PAM, setting PAM environment variable to URI of the
requested resource. PAM responder uses system bus to communicate with
IPA provider in SSSD.

.. figure:: Sssd_pam.png
   :alt: sssd_pam.png

   sssd_pam.png

Evaluation
----------------------------------------------------------------------------------------------

There are multiple possible approaches to comparing the requested
resource's URI with URI in HBAC. In all of them, I use only the part of
URI after hostname as hostname and service are already matched as part
of selecting HBAC rules to evaluate in terms of matching URI.

-  Equal string - only match an HBAC rule if there is the same string
   (maybe case insensitive comparison?) as the requested URI; that would
   mean a lot of HBAC rules differing only in specified URI and is
   probably not a feasible approach
-  **Prefix** - match HBAC rule <=> HBAC rule URI is a prefix of
   requested URI. The problem is, in web applications, the longer URI
   usually means stricter access rules. This means we must do the
   longest-prefix matching. This is the solution I have chosen: matching
   any rule with host+scheme+port exactly equal to the requested
   scheme+host+port and having the URI's path that is the longest prefix
   of the requested URI's path. (See examples below)
-  Regular expression - the most powerful interpretation of URI in HBAC
   rule, Perl-compatible regular expression should be able to cover any
   situation admin should need. They even support lookarounds which is
   very useful, see example. PCRE library is already used in SSSD so
   this adds no dependency and is a big boost to usability of the
   feature.



Example - how it can not work
----------------------------------------------------------------------------------------------

Consider webapp that has, among others, these pages:
http://hostname.net/app/auth/user1 http://hostname.net/app/auth/user42
http://hostname.net/app/auth/admin

We want to set the following access rights:

-  Any authenticated user can access http://hostname.net/app/X for X
   being any

user’s name except admin.

-  Only admin can access http://hostname.net/app/auth/admin (so: while
   any user

can access anything with prefix http://hostname.net/app, only admin can
access http://hostname.net/app/admin).

As we see, the longer the URI, the stricter access control rules. This
leads us to a concept of prefix-matching the URI’s: whenever the URI in
rule is a prefix of the requested resource’s URI, the rule matches in
terms of URI.

The way HBAC rules are interpreted, however, is currently such that
whenever any HBAC rule matches, the access is allowed. It is a correct
behavior when only con- sidering (user,service,host), but it causes a
problem when trying to include URI as a matching parameter. As shown in
the last item of a goal list, we need to have a way to allow every URI
with certain prefix A except URI’s with certain prefix B where A is a
prefix of B. In other words, we need to exclude a subset from a set of
URI’s described by certain HBAC rule. This is not possible when matching
any one rule causes access authorization.

How the rules would look (omitted HOST any SERVICE any everywhere):

-  ALLOW any URI http://hostname.net/app/auth/
-  ALLOW admin URI http://hostname.net/app/auth/admin

This does not work ! While the sec ond rule only allows admin to access
http://hostname.net/app/auth/admin, the first rule allows any user to
access everything with prefix http://hostname.net/app/auth/ , including
http://hostname.net/app/auth/admin . We can accidentally allow access to
larger set than intended and there is no way to set exceptions from that
set. In this example, there is actually no way to set the rules
correctly so they achieve the goal, except using every possible prefix
other than the intended exception.

To solve the problem of exception from a set of allowed URI’s, we could
come up with a concept of DENY rules. The approach would mean allowing
access when any ALLOW rule matches and no DENY rule matches. A DENY rule
would otherwise be the very same rule as an ALLOW rule. That would not
be completely new for FreeIPA – at certain point in time, there actually
were both ALLOW and DENY rules.

DENY rules were, however, dropped from FreeIPA. The reason for this is
that we believe that access rules should always be described positively
– listing all accesses that are allowed, rather than listing what is not
allowed and thus risking we forget something or make a mistake that
would allow access that should not be allowed. Another reason is that
when we, for some reason, don’t evaluate an ALLOW rule, the result is
denial of service at worst, while failing to evaluate a DENY rule could
allow access that should not be allowed. It seems DENY rules are
absolutely not intended to be added again.

Furthermore, merely adding DENY rules would not be sufficient; for
example, there would be no easy way to come up with rules for our
example. We would need to deny access to
http://hostname.net/app/auth/admin to large or infinite number of users
as the access would by allowed by first rule. The rules would look
something like:

-  ALLOW any URI http://hostname.net/app/auth/
-  DENY user1 URI http://hostname.net/app/auth/admin
-  DENY user42 URI http://hostname.net/app/auth/admin

This could be solved by only matching the user-wise most specific rule
or giving the rules some order, e.g.:

-  1 ALLOW any URI http://hostname.net/app/auth/
-  2 DENY any URI http://hostname.net/app/auth/admin
-  3 ALLOW admin URI http://hostname.net/app/auth/admin

This would be a fully working solution, allowing exceptions, describing
infinite number of cases (both URI- and user- wise) in a relatively
small number of rules, and relatively readable. Still, there are
drawbacks:

-  It is not easy to determine a rule to compare which one of the rules
   is more

specific user-wise. It would also be very error-prone.

-  Adding order to rules would mean a significant change in their
   semantics which

would be hardly accepted

-  DENY rules will probably never be accepted
-  There are better and simpler solutions, described further



Example - how it works
----------------------------------------------------------------------------------------------

Using the previous notion, we would in many cases create a pair of rules
for subsets we wish to exclude some users from – an ALLOW rule allowing
access to certain subset of users, and a DENY rule which is the same
except it denies any user access to the same location (which is
necessary in case there is an ALLOW rule allowing access to some URI
which is a prefix of this location’s URI). The more specific or latter
of those (depending on which approach we would choose) two rules would
be the ALLOW rule and the result would be only allowing access to that
URI to certain users.

In previous example, this exactly happens: rule 1 allows access to
http://hostname. net/app/auth to anyone and to allow access to
http://hostname.net/app/auth/ admin to admin only, we first need to deny
everyone access there by rule 2 before allowing it again for admin only
by rule 3.

It is easy to understand why the DENY rule could be there implicitly –
when admin allows access to some resource to some user, he means that
user only and all other users should be denied. However, there is
another rule that allows access to anyone - the first one. To solve this
problem, we can state that we only want to decide based on the rule with
longest prefix match. Even if there are multiple rules matching, we are
only interested in the most specific one. This allows us not to use DENY
rules at all because when there is no ALLOW rule, access is denied
implicitly, and the more general rule allowing access to a superset of
the more specific rule would be ignored. We could use the same rules as
in the previous example, just ignore the ordering and drop the DENY
rule:

-  **ALLOW any URI**\ http://hostname.net/app/auth/
-  **ALLOW admin URI**\ http://hostname.net/app/auth/admin

The first rule allows anyone access to http://hostname.net/app/auth,
except for those URIs which have URI http://hostname/app/auth as a
proper prefix. The second rule’s URI is the first rule’s URI’s proper
prefix, thus the first rule is ignored for any URI matching URI of the
second rule, regardless whether the first rule’s URI matches or not.
This serves as implicit deny for everyone if their access does not match
rule 2, regardless whether it would match rule 1 or not. Rule 2 then
allows admin access to http://hostname.net/app/auth/admin, the implicit
DENY making this the exclusive access right for admin.

Compatibility
----------------------------------------------------------------------------------------------

If there is no URI in PAM request, we match any HBAC rule that would
match without this feature. In that case, we presume the application is
not interested in URI. This means effectively ignoring URI and matching
even rules with non-matching URI when specific URI is not requested.

This is a solution I picked because of backwards compatibility - because
we of course can not change behavior of previous versions and these
versions, not aware of URI-based HBAC, would allow any rule that matches
in terms of other attributes: user, host and service.

This solution might cause problems and is not ideal: it might be seen as
problematic that when the service does not ask for any specific URI, the
is always granted if there is any rule matching in terms of other
attributes, even if it does not match URI-wise. Also, when using old
version of pam_sss or SSSD, the same situation happens as if the
application didn't include URI in the request.

I haven't however, found any better solution that would be fully
backwards-compatible. I'd be glad for suggestions.



Feature Management
------------------

UI

There are two new fields in HBAC rule details for adding URI separated
into two parts: scheme+host+port and path

CLI

There are subcommands for "ipa" command to list and modify URI, these
are generated automatically.



How to Test
-----------

There are unit tests in git.

To test manually:

-  Have a working FreeIPA, SSSD registered as FreeIPA client, Apache,
   some web application, mod_hbacauthz_pam.
-  Set mod_hbacauthz_pam to "require pam-account " in some location in
   Apache.
-  Use some authentication method, for example Kerberos, in that
   location, be logged in (let's use example "/application/login")
-  Set HBAC rules so that without this feature, one HBAC rule would
   match
-  Set URI in that HBAC rule to some prefix matching the page's URI
   path; connect the page, notice you are authorized (e.g.
   "/application")
-  Set URI in that HBAC rule to some prefix NOT matching the page's URI
   path; connect the page, notice you are NOT authorized (e.g.
   "/whatever")



Test Plan
---------

HBAC rules can be modified properly and authorization works as it should

Questions
---------

-  For backwards compatibility, lack of URI in request means any URI is
   matched (as described above). Is it a good idea? Any other solution?
-  How about multiple URI's in one HBAC rule? Is it a good idea? How to
   interpret combinations of host+scheme+port and URI paths in that
   case?