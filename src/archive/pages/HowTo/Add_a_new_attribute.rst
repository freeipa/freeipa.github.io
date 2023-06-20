Add_a_new_attribute
===================

This is a patch from Martin Kosek, annotated to explain how to add an
LDAP attribute to IPA. You can find the change as commit
`5af2e17 <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=5af2e1779ae1a0eca785493c8ed2eb044c8e282a>`__
in the IPA source tree.

| ``   From: Martin Kosek <mkosek@redhat.com>``
| ``   Date: Tue, 23 Apr 2013 09:59:24 +0200``
| ``   Subject: [PATCH] Add userClass attribute for hosts``
| ``   ``
| ``   This new freeform host attribute will allow provisioning systems``
| ``   to add custom tags for host objects which can be later used for``
| ``   in automember rules or for additional local interpretation.``
| ``   ``
| ``   Design page: ``\ ```http://www.freeipa.org/page/V3/Integration_with_a_provisioning_systems`` <http://www.freeipa.org/page/V3/Integration_with_a_provisioning_systems>`__
| ``   Ticket: ``\ ```https://fedorahosted.org/freeipa/ticket/3583`` <https://fedorahosted.org/freeipa/ticket/3583>`__
| ``   ``
| ``   ---``
| ``   API.txt                               |  9 ++++++---``
| ``   VERSION                               |  2 +-``
| ``   install/share/60basev2.ldif           |  2 +-``
| ``   install/updates/10-60basev3.update    |  1 +``
| ``   ipalib/plugins/host.py                |  7 +++++++``
| ``   tests/test_xmlrpc/test_host_plugin.py | 23 +++++++++++++++++++++++``
| ``   6 files changed, 39 insertions(+), 5 deletions(-)``

First, we need to add the attribute to the LDAP schema, so it is
available for new installs. Locate the appropriate objectclass
definition in ``install/share/``, and add the desired attribute.

| ``   diff --git a/install/share/60basev2.ldif b/install/share/60basev2.ldif``
| ``   index 3b05e370147f6cace12913e695e02eb6550c6010..8e7174c10ddf73194bfbe634ff34c8c3fd25e264 100644``
| ``   --- a/install/share/60basev2.ldif``
| ``   +++ b/install/share/60basev2.ldif``
| ``   @@ -13,7 +13,7 @@ attributeTypes: (2.16.840.1.113730.3.8.3.24 NAME 'ipaEntitlementId' DESC 'Entitl``
| ``    # ipaKrbAuthzData added here. Even though it is a v3 attribute it is updating``
| ``    # a v2 objectClass so needs to be here.``
| ``    attributeTypes: (2.16.840.1.113730.3.8.11.37 NAME 'ipaKrbAuthzData' DESC 'type of PAC preferred by a service' EQUALITY caseExactMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v3' )``
| ``   -objectClasses: (2.16.840.1.113730.3.8.4.1 NAME 'ipaHost' AUXILIARY MUST ( fqdn ) MAY ( userPassword $ ipaClientVersion $ enrolledBy $ memberOf) X-ORIGIN 'IPA v2' )``
| ``   +objectClasses: (2.16.840.1.113730.3.8.4.1 NAME 'ipaHost' AUXILIARY MUST ( fqdn ) MAY ( userPassword $ ipaClientVersion $ enrolledBy $ memberOf $ userClass ) X-ORIGIN 'IPA v2' )``
| ``    objectClasses: (2.16.840.1.113730.3.8.4.12 NAME 'ipaObject' DESC 'IPA objectclass' AUXILIARY MUST ( ipaUniqueId ) X-ORIGIN 'IPA v2' )``
| ``    objectClasses: (2.16.840.1.113730.3.8.4.14 NAME 'ipaEntitlement' DESC 'IPA Entitlement object' AUXILIARY MUST ( ipaEntitlementId ) MAY ( userPKCS12 $ userCertificate ) X-ORIGIN 'IPA v2' )``
| ``    objectClasses: (2.16.840.1.113730.3.8.4.15 NAME 'ipaPermission' DESC 'IPA Permission objectclass' AUXILIARY MAY ( ipaPermissionType ) X-ORIGIN 'IPA v2' )``

The next step is to add a parameter definition to the appropriate
framework object. In addition to the big ``takes_params`` list, there
may also be other lists of parameters (such as ``default_attributes``
whose members are shown by ``*-find`` and ``*-show`` commands by
default). Update these lists as appropriate.

::

    | ``   diff --git a/ipalib/plugins/host.py b/ipalib/plugins/host.py``
    | ``   index c79b9e212feec640a34ac0905d46adacee54060f..e615259174722af645bdad72802d8fca9783f6d3 100644``
    | ``   --- a/ipalib/plugins/host.py``
    | ``   +++ b/ipalib/plugins/host.py``
    | ``   @@ -230,6 +230,7 @@ class host(LDAPObject):``
    | ``            'fqdn', 'description', 'l', 'nshostlocation', 'krbprincipalname',``
    | ``            'nshardwareplatform', 'nsosversion', 'usercertificate', 'memberof',``
    | ``            'managedby', 'memberindirect', 'memberofindirect', 'macaddress',``
    | ``   +        'userclass'``
    | ``        ]``
    | ``        uuid_attribute = 'ipauniqueid'``
    | ``        attribute_members = {``
    | ``   @@ -323,6 +324,12 @@ class host(LDAPObject):``
    | ``                csv=True,``
    | ``                flags=['no_search'],``
    | ``            ),``
    | ``   +        Str('userclass*',``
    | ``   +            cli_name='class',``
    | ``   +            label=_('Class'),``
    | ``   +            doc=_('Host category (semantics placed on this attribute are for '``
    | ``   +                  'local interpretation)'),``
    | ``   +        ),``
    | ``        ) + ticket_flags_params``
    | ``   ``
    | ``        def get_dn(self, *keys, **options):``

Since the API was changed, run the ``make-api`` script located at the
root of te source tree. This will re-generate API.txt, our safeguard
against unintentional API changes.

| ``   diff --git a/API.txt b/API.txt``
| ``   index 3e01fdc3611b5bc71e1a4ee185af63f7c4b07c06..c2400e901345a70e0236d1c02979220c19ece9a5 100644``
| ``   --- a/API.txt``
| ``   +++ b/API.txt``
| ``   @@ -1723,7 +1723,7 @@ output: Output('summary', (<type 'unicode'>, <type 'NoneType'>), None)``
| ``    output: Output('value', <type 'bool'>, None)``
| ``    output: Output('warning', (<type 'list'>, <type 'tuple'>, <type 'NoneType'>), None)``
| ``    command: host_add``
| ``   -args: 1,20,3``
| ``   +args: 1,21,3``
| ``    arg: Str('fqdn', attribute=True, cli_name='hostname', multivalue=False, primary_key=True, required=True)``
| ``    option: Str('addattr*', cli_name='addattr', exclude='webui')``
| ``    option: Flag('all', autofill=True, cli_name='all', default=False, exclude='webui')``
| ``   @@ -1743,6 +1743,7 @@ option: Flag('random', attribute=False, autofill=True, cli_name='random', defaul``
| ``    option: Flag('raw', autofill=True, cli_name='raw', default=False, exclude='webui')``
| ``    option: Str('setattr*', cli_name='setattr', exclude='webui')``
| ``    option: Bytes('usercertificate', attribute=True, cli_name='certificate', multivalue=False, required=False)``
| ``   +option: Str('userclass', attribute=True, cli_name='class', multivalue=True, required=False)``
| ``    option: Str('userpassword', attribute=True, cli_name='password', multivalue=False, required=False)``
| ``    option: Str('version?', exclude='webui')``
| ``    output: Entry('result', <type 'dict'>, Gettext('A dictionary representing an LDAP entry', domain='ipa', localedir=None))``
| ``   @@ -1774,7 +1775,7 @@ output: Output('result', <type 'bool'>, None)``
| ``    output: Output('summary', (<type 'unicode'>, <type 'NoneType'>), None)``
| ``    output: Output('value', <type 'unicode'>, None)``
| ``    command: host_find``
| ``   -args: 1,31,4``
| ``   +args: 1,32,4``
| ``    arg: Str('criteria?', noextrawhitespace=False)``
| ``    option: Flag('all', autofill=True, cli_name='all', default=False, exclude='webui')``
| ``    option: Str('description', attribute=True, autofill=False, cli_name='desc', multivalue=False, query=True, required=False)``
| ``   @@ -1805,6 +1806,7 @@ option: Flag('raw', autofill=True, cli_name='raw', default=False, exclude='webui``
| ``    option: Int('sizelimit?', autofill=False, minvalue=0)``
| ``    option: Int('timelimit?', autofill=False, minvalue=0)``
| ``    option: Bytes('usercertificate', attribute=True, autofill=False, cli_name='certificate', multivalue=False, query=True, required=False)``
| ``   +option: Str('userclass', attribute=True, autofill=False, cli_name='class', multivalue=True, query=True, required=False)``
| ``    option: Str('userpassword', attribute=True, autofill=False, cli_name='password', multivalue=False, query=True, required=False)``
| ``    option: Str('version?', exclude='webui')``
| ``    output: Output('count', <type 'int'>, None)``
| ``   @@ -1812,7 +1814,7 @@ output: ListOfEntries('result', (<type 'list'>, <type 'tuple'>), Gettext('A list``
| ``    output: Output('summary', (<type 'unicode'>, <type 'NoneType'>), None)``
| ``    output: Output('truncated', <type 'bool'>, None)``
| ``    command: host_mod``
| ``   -args: 1,21,3``
| ``   +args: 1,22,3``
| ``    arg: Str('fqdn', attribute=True, cli_name='hostname', multivalue=False, primary_key=True, query=True, required=True)``
| ``    option: Str('addattr*', cli_name='addattr', exclude='webui')``
| ``    option: Flag('all', autofill=True, cli_name='all', default=False, exclude='webui')``
| ``   @@ -1833,6 +1835,7 @@ option: Flag('rights', autofill=True, default=False)``
| ``    option: Str('setattr*', cli_name='setattr', exclude='webui')``
| ``    option: Flag('updatedns?', autofill=True, default=False)``
| ``    option: Bytes('usercertificate', attribute=True, autofill=False, cli_name='certificate', multivalue=False, required=False)``
| ``   +option: Str('userclass', attribute=True, autofill=False, cli_name='class', multivalue=True, required=False)``
| ``    option: Str('userpassword', attribute=True, autofill=False, cli_name='password', multivalue=False, required=False)``
| ``    option: Str('version?', exclude='webui')``
| ``    output: Entry('result', <type 'dict'>, Gettext('A dictionary representing an LDAP entry', domain='ipa', localedir=None))``

With every update of the API, you must bump the API version. When adding
parameters calls, only bump the minor version number.

| ``   diff --git a/VERSION b/VERSION``
| ``   index 9208237cbedf23d71c5c579fcc10207380cc9712..4bee01b981d818de21f0be1b16d5668a7f453baf 100644``
| ``   --- a/VERSION``
| ``   +++ b/VERSION``
| ``   @@ -89,4 +89,4 @@ IPA_DATA_VERSION=20100614120000``
| ``    #                                                      #``
| ``    ########################################################``
| ``    IPA_API_VERSION_MAJOR=2``
| ``   -IPA_API_VERSION_MINOR=57``
| ``   +IPA_API_VERSION_MINOR=58``

And of course, every code change should be accompanied by a test.

| ``   diff --git a/tests/test_xmlrpc/test_host_plugin.py b/tests/test_xmlrpc/test_host_plugin.py``
| ``   index f788dc6bc6d55f46856ada4b816997bfb517d8c4..07faf77607284b2193716854b287208f563d9472 100644``
| ``   --- a/tests/test_xmlrpc/test_host_plugin.py``
| ``   +++ b/tests/test_xmlrpc/test_host_plugin.py``
| ``   @@ -700,6 +700,7 @@ class test_host(Declarative):``
| ``                    dict(``
| ``                        description=u'Test host 2',``
| ``                        l=u'Undisclosed location 2',``
| ``   +                    userclass=[u'webserver', u'mailserver'],``
| ``                        force=True,``
| ``                    ),``
| ``                ),``
| ``   @@ -715,6 +716,7 @@ class test_host(Declarative):``
| ``                        objectclass=objectclasses.host,``
| ``                        ipauniqueid=[fuzzy_uuid],``
| ``                        managedby_host=[fqdn2],``
| ``   +                    userclass=[u'webserver', u'mailserver'],``
| ``                        has_keytab=False,``
| ``                        has_password=False,``
| ``                    ),``
| ``   @@ -722,6 +724,27 @@ class test_host(Declarative):``
| ``            ),``
| ``   ``
| ``   ``
| ``   +        dict(``
| ``   +            desc='Retrieve %r' % fqdn2,``
| ``   +            command=('host_show', [fqdn2], {}),``
| ``   +            expected=dict(``
| ``   +                value=fqdn2,``
| ``   +                summary=None,``
| ``   +                result=dict(``
| ``   +                    dn=dn2,``
| ``   +                    fqdn=[fqdn2],``
| ``   +                    description=[u'Test host 2'],``
| ``   +                    l=[u'Undisclosed location 2'],``
| ``   +                    krbprincipalname=[u'host/%s@%s' % (fqdn2, api.env.realm)],``
| ``   +                    has_keytab=False,``
| ``   +                    has_password=False,``
| ``   +                    managedby_host=[fqdn2],``
| ``   +                    userclass=[u'webserver', u'mailserver'],``
| ``   +                ),``
| ``   +            ),``
| ``   +        ),``
| ``   +``
| ``   +``
| ``            # This test will only succeed when running against lite-server.py``
| ``            # on same box as IPA install.``
| ``            dict(``
| ``   -- ``
| ``   1.8.1.4``