V3PRD
=====

-  Add support for certificate authentication and provisioning for users
-  Switch IPA's DNS schema to use syntax for attributes that is now
   available in latest upstream version of DS.
-  Central management of the disk encryption keys
-  Integration with LUKS/Anaconda for key escrow
-  Integration with Anaconda for enrolling a machine in an IPA domain at
   install time
-  NTP and SSSD. The main problem here btw is insuring that the source
   of the time is trusted. Microsoft modified NTP to add their custom
   signature so that clients can trust that NTP packets are coming from
   a trusted server and not a forgery to convince our machine to change
   time. We probably need to investigate if we have a way to use GSSAPI
   or some other key mechanism to do trusted NTP as well. NTP already
   has some sort of signing method.
-  NTP and IPA v3. There is a patch for NTP server that allows Samba to
   serve Windows clients with the NTP requests the Windows clients
   expect. We need to explore integration between Samba's way of using
   NTP and NTP provided by IPA in the scope of IPA v3 project. The
   Windows clients can be configured to use normal NTP so it might be a
   lower priority, however tha concern is that it would require some
   configuration changes on the Windows client.
-  Add RADIUS support
-  Add the capability to associate a full ACI with each command plugin
   to implement access control for commands that don't modify or query
   LDAP. For v2, will will have only the ability to required a
   particular group for each command (like must be in "admin" group).
-  The access control to who can perform certificate operations should
   be improved. In case of v2 it is group based where group name is keep
   in a special configuration entry
-  In future releases we might want to add the capability to define
   rules about how the input of different fields must be validated. For
   example login name must match some combination of letters from field
   and last name.
-  Use case: "We have several different servers that are going to
   require different users to have different shells on each. It appears
   I can only define one shell per user. Is there a way to either create
   some sort of policy with server groups that will dynamically change
   the shell the user is using depending on the group they are in?"

   It would be possible to do something like this by modifying the
   classOfService plugin so that it would modify the return value of the
   user shell based on the bind identity of the LDAP connection. This
   would not work in IPAv1 because all users bind as themselves, but in
   IPAv2, through the SSSD, the LDAP connections will be bound as the
   machine connection. This would make it possible to have "group of
   machine" responses to individual attributes.

-  Consider storing SUDO information in DS and service it via SSSD.
   Integrate SUDO with SSSD on the client. This is an alternative to
   just distributing files. The reason is that for customers the SUDO is
   more identity management than configuration management. In our case
   we said that SUDO file is distributed via config management solution
   but realistically it makes sense to do it centrally and store in DS.
-  Allow one host to provision keytabs for another host.