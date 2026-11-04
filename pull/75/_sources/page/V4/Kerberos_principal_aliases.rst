Kerberos_principal_aliases
==========================

reading this: "The old aliases will be kept unchanged so that the
user/host/service can continue to use old credentials for
authentication. " I am thinking ... why are you keeping the old name ?

If a rename happens the admin may (or not) want to keep the older name
as an alias, I think it should be a CLI/UI option whether to keep the
old name as an alias or not something like --retain-old-princ-as-alias
as an option to the rename command.

The default should be to retain I guess, otherwise deployed keytabs will
stop working. The CLI should emit a warning if a host is renamed (do we
allow renaming of hosts at all ?) that a keytab should be re-issued to
match the new canonical name.

Q: Shold e enhance the ipa-getkeytab utiity to know about the canonical
name and use that in the keytab by default ??