Introduction
------------

Identity Management (IdM) needs change as an organization grows in size.
For an example, I'll describe a fictional company, and take it from the
smallest to largest stages. While, to some degree, the industry of this
firm really doesn't matter, I am going to use a small import business
started by a single individual and scale it up to a multinational
corporation. As the organization grows in size, the technical needs will
drive the scope and scale of the identity management solutions required.

Scenario
~~~~~~~~

Ms. N. Zappa has just come back from a masters degree program in
Jakarta. While living in Indonesia, she established connections with
several furniture producers, and she starts a small import/export
business in the Boston area. She has contacts at a handful of stores
already from previous experience. She moves back in with her folks, who
have agreed that she can use a section of the garage for storage, and
she outsources all technical issues to a small web services firm. They
register the domain name ZappasImports.com for her, set up an account a
large cloud provider who provide web and email services. Her information
management needs are for tracking orders, suppliers, and customers,
which she keeps in a set of spreadsheets that she backs up to the cloud
on a nightly basis.

When there is just one person in Zappas, the IdM needs are purely
between N Zappa and the cloud provider. Her Internet services contractor
has set up an account with the Cloud provider. The Zappa account has two
users associated with it: nzappa and the contractor. This authentication
is done against the Cloud Providers Authentication Database.

.. _initial_setup:

Initial Setup
~~~~~~~~~~~~~

The contractor sets up three applications for Zappas. A static web
server, remote backups, and email. The email server is a managed
service, and runs on the same email server as many of the other clients
of the Cloud Provider. At this point, the Cloud provider is really just
an Internet Service Provider. The website has a simple WebDAV interface
and runs as a virtual host on shared web server. Remote backups are done
via WebDAV. The Cloud provider manages all these services in order to
scale up the number of customers it can handle without overloading its
own infrastructure. Because the web applications are run on shared
servers, they are responsive. If the overall load increases, the Cloud
provider can scale up by allocating more virtual machines.

.. _contractor_with_multiple_clients:

Contractor with multiple Clients
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Zappas is not the only company the web services company manages. They
have a portfolio of customers. The cloud deployment approach matches
their development methods and skill set. They do not want to have a
seperate account with the cloud provider for each of their clients.
Instead, the user accounts are associated with the client accounts. In
addition, the web services firm is itself a client of the cloud
provider, with systems that are somewhat comparable to those that they
are setting up for Zappas.

.. _initial_growth:

Initial Growth
--------------

There are two potential next steps. One is that the company adds
personel. The other is when Zappas wants to start tracking and
authenticating people outside of her company. These are two fairly
different directions. They can happen in either order. Lets look at the
external issue first.

.. _external_identity_management:

External Identity Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After a few months of operations, she realizes that she is having a much
lower rate of return customers than she anticipated. She wants to
communicate better with the people that have bought from here before to
encourage repeat business, as well as to gauge the relative degree of
satisfaction her customers have had with their purchases.

Zappa's sets up a single external application, a customer order portal.
This runs on a dedicated virtual machine, and uses a software package
that the cloud provider has pre-canned. It uses an internal database,
either PostgreSQL or MySQL with a table for user accounts.
Authentication is done via Basic-Auth. The system is accessed via Web
over SSL. This requires that the Web server have a publicly routed
static IP address. The Cloud Provider runs a certificate authority, and
issues a certificate for the web service to Zappa's Imports. The
Customer database is pre-populated with email addresses from a
spreadsheet. Passwords are set to expired, which allows the users to
reset them using their email addresses as verification. She emails her
customers about the system and they start using it.

She also starts a blog, to describe the process she goes through to find
new furniture, to explain the history behind some of the more
interesting designs, and to introduce people to the culture of
Indonesia. Again, this starts off as a managed service by the Cloud
service provider: a Worpress instance with another dedicated back-end
database. At first she enables public commenting, but the high degree of
spam dictates that she somehow authenticate the comments. Instead, she
decides that the appropriate thing for her business is to limit comments
to people that have either purchased or are thinking of purchasing
furniture from her. She could choose to use OpenID or a different Web
Single Sign on solution, but that provides her no way to cross reference
posts with her customer database. She wants to integrate the two
systems. She also is not really happy with the order tracking system,
and thinks she might want to switch to a different software package.
While all three software packages she has been looking at provide
embedded Identity management, they are all incompatible with one
another. However, all of them support external authentication. She
decides to extract the IdM out into a dedicated system in order to
federate her customer focused applications more easily.

.. _internal_identity_management:

Internal Identity Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Her business grows and she gets an office manager. She rents space at a
local warehouse and a small office nearby. They set up accounts with a
handful of delivery companies and need to be able to integrate the
tracking of packages. The international nature of the business dictates
that they get an account that specializes in international trade. Zappas
has sufficient business going direct to end consumers that they set up a
web catalog and order placement system. This dictates that they have a
content management system, and that they establish a relationship with
professional photographers in the remote countries that can supply their
catalog with high quality images. They have a handful of contractors on
retainer that can deliver and assemble some of the more complex pieces.
The organization grows a little further. The company is producing enough
revenue that she has a full time sales manager. During the busy season,
she also hires seasonal help.

When Zappas hires her first employee, she needs to set up a new email
account. This is likely done in the Cloud provider's IdM system. She
also needs to add the new employees desktop to the back up system. While
the new employee is nominally an office manager, in practical terms she
is helping with all aspects of the business, which means she needs to be
able to perform administrative activities on the systems hosted in the
cloud. The IdM solution they've chose supports this by adding the user
to a group specific to each of the applications.

Analysis
--------

"There is nothing more practical than a good theory." --Kurt Lewin

There are patterns that show up in all implementations of IdM systems.
Recognizing them will help in implementation of future systems.

Delegation
~~~~~~~~~~

Her organization may look simple, but Zappa's Imports has most of the
identity management needs of a large organization. She has distinct
external contacts that she needs to segregate from each other. Each has
a different set of permissible activities based on role. , but there are
certain activities that the partners can do that the the office manager
can't. With the addition of the sales manager the organization has grown
to the point that she can (and must) delegate management
responsibilities: this is the critical point for an identity management
system. The sales manager needs to be able to hire a temporary worker,
give that worker limited access, and then revoke it after the busy
period. Perhaps the temp can take orders, but is not allowed to handle
returns. In addition, the sales manager needs to be able to schedule and
track a contractor that is heading to a customer site to do an assembly.

The IdM system manages three types of relationships. First, it manages
the relationship between The cloud provider and Zappa's imports. Second,
it manages relationships between the Zappa's imports employees. Finally,
it manages the relationships between Zappa's Imports and the people that
do business with Zappa's. From Zappa's perspective, adding a new
application and adding an additional employee are related tasks. At
first, both are handled by Ms. Zappa. By the end, are performed by
employees who have been delegated the authority to perform these tasks.

Ownership
~~~~~~~~~

Probably the largest question in cloud IdM integration is who owns the
information. The Zappas customer list and suppliers list are probably
the most valuable assets of the company. If the cloud provider wants to
maintain its own customers, they need to provide sufficient reason to
trust their Id solution. On the other hand, they need to protect their
own assets. In their organization, Ms. Zappa is a client, and she can
add and remove servers based on her contract and quality of service
agreement with the cloud provider. As the organization grows, this is
certainly one of the actions she will want to delegate to the CTO, who
will later delegate it to the Operations Manager when Zappa's Imports
grows to the size to hire one.

.. _avoiding_lock_in:

Avoiding Lock In
~~~~~~~~~~~~~~~~

Although companies like Zappa's are not going to be jumping from one
cloud provider to another each week, they still want to avoid vendor
lock in. Zappa's applications should have no dependencies on elements
internal to the cloud provider that cannot be redirected to a different
cloud provider. This is going to be more and more important as the
company scales up. A Cloud provider can prove to be a poor fit for an
organization for multiple reasons, and not all of those can be foreseen
upfront. In the case of Zappa's, technology is an enabler for business,
but it is not the company's forte. By contracting with a decent internet
services firm, she has offloaded some of the decisions. She shouldn't be
locked in indefinitely to a technology decision made today that is
secondary to the core of the business.

It is helpful to think of the federation of Identity Management systems
as single solutions, with carefully specified integration points between
the component parts. By having two systems, you firewall off one segment
of your users from another. However, it is worth noting that there are a
couple issues this does not address. If there are two systems, and one
is deemed canonical, and information is pushed from one to the other,
they are acting as a single system, and thus the potential for elevation
of privileges is still there.

Enterprise
----------

A few years down the road, Zappa's is doing well, and has grown to a
middle sized companies. N Zappa maintains her role as CEO and President,
but is completely out of the software side of the company. Dealing with
the relationships essential to her business must be supported by
technology, not derailed by the management of technology. Zappa's has
all of the major subdivisions that one would expect to see in a middle
sized company. Marketing and Sales are now distinct operations.
Operations has a full time call center as well as full time employees
for deliveries that require assembly. IT has also grown to support the
needs of the organization. On the supply side, several employees have
been hired in remote offices to handle the purchasing in several
countries. The full time Human Resources manager has contracted to a
small number of staffing firms for filling the variations the yearly
cycle.

.. _techno_mix:

Techno Mix
~~~~~~~~~~

The information systems are now a mix of in-house, application service
provider, and cloud hosted applications. Zappa's works mainly with two
cloud providers that offer differentiated offerings due to geographic
location and international issues. Applications for supply, foreign tax
and travel, and shipping are hosted in a company in Jakarta. The
eCommerce web site and partner Business-to-Business portal is hosted
stateside. Zappa's has consolidated IdM in-house, and shares a subset of
its user database with the various technology partners.

The load on the cloud providers systems has scaled up with the company.
When orders were measured in single-digits per day, billing was likely
at a flat rate. At scale, the daily variations and fluctuation are
significant. Auditing the systems becomes a requirement for legal and
financial reasons. Different portions of Zappa's have different budgets,
and pay for the usage of the cloud out of separate accounts.

.. _authentication_tokens:

Authentication Tokens
~~~~~~~~~~~~~~~~~~~~~

For authentication of employees, Zappa's has moved from a
userid/password model to a cryptographically secure system, based on
technologies like Kerberos, hardware tokens, smart-cards and so forth.
Different systems require the ability to accept different authentication
mechanisms: the corporate blog needs a HW token in order to post an
article, but allow OpenID authentication for a customer to post a follow
up comment. Some of the suppliers and partners have advanced
technologically as well, and have their own authentication systems that
they want to use when performing business with Zappa's. The wide number
of web sites in use have dictated a single sign on solution that needs
to span across the public internet, and that works around the fact that
some of the sites only allow network traffic on ports 443 and 80.

By this point, a standardized mechanism for managing the IdM across all
of these systems has become essential. Adding in a new application
cannot force changes to the schema. Each additional application cannot
silo its own user database. Centralized management is a must;
replication to remote sites must be rapid. Elevation of privileges is a
major risk and the IdM must have rock solid protections built in against
it.

The cloud provider is an integral partner to Zappas. It has to provide
seamless integration of Identity Management. Trust and audit are two
sides of the same issue: Zappas needs to know what is going on at the
Cloud provider. Access to the systems running as virtual machines in the
cloud has to be tightly managed. At the same time, the cloud provider
finds it seld in the same relationship with many other companies, some
of which are competitors to Zappos. For legal and financial reasons, the
cloud providers needs to provide segregation between its clients. The
number of clients it is managing means that the system has to work
completely automated.

Conclusion
----------

This narrative has attempted to put the scale and scope of identity
management issues in a cloud deployment into to perspective. The
fictional company matches the needs of many real companies, as well as
many non-corporate entities. The scope of the identity management
solutions vary with the scope of the problem or number of problems that
they attempt to solve. As our fictional company grew, its needs changed,
as did the corresponding Identity Management Solution. The ability to
tune the software packages that your company uses will help determine
how quickly the company can react to new business opportunities or deal
with set backs. A larger organization has more contact points, more
needs, and thus cannot as easily rewrite the configuration of their
applications to deal with a different IdM solution.

`Category:NoLink <Category:NoLink>`__
`Category:CheckUpdate <Category:CheckUpdate>`__
