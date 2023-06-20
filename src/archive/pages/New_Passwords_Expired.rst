New_Passwords_Expired
=====================

As an Identity Management store FreeIPA manages user passwords. One of
the features we decided to embed in FreeIPA is that when a password is
first set or when a password is later reset we mark this password as
immediately expired and require the owner to perform a password change.
The only exception is for `password synchronization
agents <PasswordSynchronization>`__.



Password Exclusivity
====================

Sometimes this behavior is unexpected and not understood. So why do we
do that?

A password has only one real requirement, it needs to be secret, and it
needs to be known only by the entity authorized to use it. When the
password is the only authentication factor in use it essentially
comprises your entire identity.

When an administrator first sets a password for a user this requirement
is not satisfied. The password is secret, but 2 (or more) people now
know it. This means this password does not identify a single entity. It
is therefore important to remedy this situation as soon as possible.



Password Distribution
=====================

There is another factor that comes into play, *password distribution*.
When an administrator resets a password, not only he gets to know it,
but he also needs to transmit it to the final user. Common means to
transmit a new password are by phone or by email, or on paper. All of
these methods pose a significant threat to the security of the password,
and leave ample margins for an attacker to steal fresh new credentials.
We can reduce the threat that a stolen password is abused and the abuse
to go unnoticed by forcing a password reset.

If an attacker get access to the initial password during transmission,
he has a very small period of time to (ab)use it. The first thing anyone
is requested to do to use the password is to change it. Now, if the
legit user does it first, the attacker is left with a useless secret. If
the attacker does it first, the user will notice immediately the first
time he tries to authenticate himself. He will not have access, and he
will notify the administrator. This allow the administrator to promptly
take action and verify what's going on.

Remediation
===========

By making a new password expired by default we basically force the user
to remedy these situations as the first thing, so that the threat
against impersonation is reduced.

Of course this is not to be considered a bullet proof protection and an
excuse to avoid protection of the credentials during transmission. There
are many other attack vectors that this scheme does not address at all.
This it is just an additional measure to make life a bit more difficult
for an attacker.