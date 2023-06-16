On the Local Replay issue, we do not really need to use locks (which
increase contentions on servers), but I think we can simply use a
delete/add operation that removes by specific value in order to update
the HWM/counter. If 2 auth attempts race on updating the HWM/counter,
then one of them will fail the delete as the value has already been
updated by the other thread. Thus we can have local lockless coherency.
-sss

In the Replication Counter Race section it is repeated multiple times
that the HOTP counter must be replicated. I do not think that is
inherently necessary. We can avoid constant replication by adopting 2
techniques. Daily or other interval based replication (like when the
counter is incremented by 10) and/or leap of faith authentication on
usurpation, where even if the counter is much in the future compoared to
what is stored loclly we still accpet authentication, a larger window is
used during usurpation to check for counter-skew. This allows
replication to be performed only occasionally (and maybe only to a more
restricted set of designated HOTP master servers).
