V4_slapi_nis_locking
====================

Overview
--------

Schema compat plugin can create deadlocks like
`1435663 <https://bugzilla.redhat.com/show_bug.cgi?id=1435663>`__. The
two operations in that BZ have to be managed by Schema compat so it is
not possible to workaround that deadlock scenario by ignoring one of
them. The root cause of the deadlock is that Schema compat maps are
acting like a backend (with its own lock). Let an operation (SRCH)
accessing the map then needing access to main backend pages (ACI
evaluation). An other operation updating the same main backend pages
then with TXN POST operations access the map. Those two threads are in a
scenario of deadlock because acquiring locks in the opposite order.

The purpose of that design is to describe the constraints to fix this
problem and describe a solution



Use case
--------

A admin is adding a IPA user to a GROUP_A, this triggers an update of
the user (to update memberof attribute) that will update the related
Schema Compat entry. At the same time a SRCH of a Schema Compat entry
need to evaluate GROUP_A to grant access to attributes present in the
search filter.

Design
------

Just to give an example of a typical scenario of the deadlock here is
how deadlocking threads are looking like

::

   | ``   Thread 34 (Thread 0x7f2f4f7fe700 (LWP 22764)):``
   | ``   #0  0x00007f2f99f776d5 in pthread_cond_wait@@GLIBC_2.3.2 ()``
   | ``   ..``
   | ``   /*``
   | ``    * The page owning the group is held by an other TXN``
   | ``    */``
   | ``   #5  0x00007f2f936a8132 in __db_lget ()``
   | ``   ..``
   | ``   #12 0x00007f2f8e583fba in _entryrdn_index_read ()``
   | ``   ...``
   | ``   /*``
   | ``    * It need to evaluate a group in the main DB``
   | ``    */``
   | ``   #19 0x00007f2f9c3fdf5e in search_internal_callback_pb ()``
   | ``   #20 0x00007f2f91d1900f in acllas__user_ismember_of_group ()``
   | ``   ..``
   | ``   #27 0x00007f2f91d249f7 in acl_access_allowed_main ()``
   | ``   ..``
   | ``   /* A search in the scope of Schema Compat, needs to``
   | ``    * evaluate if the requestor as access right to an entry``
   | ``    * This search holds the SC map lock``
   | ``    */``
   | ``   #31 0x00007f2f9c3c50da in slapi_filter_test_ext ()``
   | ``   #32 0x00007f2f8c7bfcda in backend_search_set_cb ()``
   | ``   #33 0x00007f2f8c7d103f in map_data_foreach_map ()``
   | ``   #34 0x00007f2f8c7bfa46 in backend_search_group_cb ()``
   | ``   #35 0x00007f2f8c7d0f80 in map_data_foreach_domain ()``
   | ``   #36 0x00007f2f8c7bf101 in backend_search_cb ()``
   | ``   #37 0x00007f2f9c3f8db8 in plugin_call_func ()``
   | ``   #38 0x00007f2f9c3f9043 in plugin_call_plugins ()``
   | ``   #39 0x00007f2f9c3ecfdb in op_shared_search ()``
   | ``   #40 0x00007f2f9c8e511e in do_search ()``
   | ``   ``
   | ``   ===============================``
   | ``   Thread 11 (Thread 0x7f2f43fe7700 (LWP 22787)):``
   | ``   /*``
   | ``    * The MOD needs to update the SC map and acquire its lock``
   | ``    */``
   | ``   #0  0x00007f2f99f7703e in pthread_rwlock_wrlock ()``
   | ``   #1  0x00007f2f8c7c49d5 in backend_shr_modify_cb.part.20 ()``
   | ``   #2  0x00007f2f8c7c5171 in backend_shr_betxn_post_modify_cb ()``
   | ``   ``
   | ``   /*``
   | ``    * We are in BETXN_POST, so DB pages impacted by the MOD are hold``
   | ``    */``
   | ``   #3  0x00007f2f9c3f8db8 in plugin_call_func ()``
   | ``   #4  0x00007f2f9c3f9043 in plugin_call_plugins ()``
   | ``   #5  0x00007f2f8e58c9d1 in ldbm_back_modify ()``
   | ``   #6  0x00007f2f9c3e655b in op_shared_modify ()``
   | ``   #7  0x00007f2f9c3e791f in do_modify ()``

The basic of the deadlock is two locks taken in the opposite order. 3
ideas came to my mind to fix it

-  Avoid to acquire one of the lock. For example in Thread 32 to not
   call the ACI evaluation. This option is not possible else this would
   mean CVE (anonymous could lookup entry even if they are not allowed
   to do so)
-  Implement a finer grain lock. It does not exist design that describes
   what is protected by map lock. Maps themselves are MT-Safe but the
   meta data of the btree need to be protected (various counters and
   pointers). As I said it is difficult to evaluate all fields that need
   to be protected and those who need are quite spread in the code. So
   redesigning the locking of slapi nis was not my favorite option.
-  Reorder the way locks are taken, so that a write operation first
   acquire the SC map lock before acquiring the DB pages/locks. This is
   the option described in that document

The rest of this document is describing the third option : **Reorder
locking mechanism**

General comments:

-  The deadlock is easily reproducible with the `reproducible test
   case <https://bugzilla.redhat.com/attachment.cgi?id=1315173>`__. Now
   the conditions are rare

   -  search/write need to access the same DB page
   -  the data must not have been cached (entry cache, group cache) to
      trigger DB get
   -  Search/write run at the same time

-  Because the impact of this reordering is difficult to anticipate
   (performance, others scenario of deadlocks, crash...), I would
   recommend to **activate it on demand** (config toggle) and **not
   enabled it by default**.
-  It is not possible for performance reason to serialize SRCH requests,
   so this is WRITE operations that need to acquire SC map lock before
   updating DB pages.
-  Schema compat is a **BE_TXN_POSTOP** for write operations
-  Schema compat scopes only parts of the DIT. But even if a direct
   update is out of the scope of SC, it can trigger internal updates
   (from others postop plugin) that are in the scope of SC. So **any
   update should acquire SC map lock**
-  if a write operation (that can potentially impact SC) prevents others
   reads/write threads, a read operation should allow others reads and
   block writes until it completes. So **the lock should be a read write
   lock**
-  it is unpredicatable (because of plugins) how much time the lock will
   be acquired by a same thread. The **lock should be reentrant**.

The basic idea to reorder locking mechanism is that a write thread
acquires the SC lock in PREOP and release it in POSTOP.



First try
----------------------------------------------------------------------------------------------

A first
`patch <https://bugzilla.redhat.com/attachment.cgi?id=1305923>`__ was

-  Using the already existing BETXN_PREOP (*backend_write_cb*) to
   acquire the lock in write and already existing BETXN_POSTOP
   (*backend_shr_add_cb backend_shr_modify_cb..*) to release it.
-  Using per thread variables to make the lock reentrant
-  Using the SC map lock (*map_unlock, map_rdlock and map_wrlock*)
-  Update *map_unlock, map_rdlock and map_wrlock* to make it reentrant

This **first patch** failed because:

-  Because the order plugin are called does not guaranty that the write
   lock is release **after the last postop** plugin callback is called
-  PREOP was releasing the write lock when the update was out of the
   scope of the SC



Second patch
----------------------------------------------------------------------------------------------

Because we can not guaranty the order the plugins are called, the
acquisition/release of the lock should be done a step above:
**BE_PREOP** / **BE_POST**.

The lock is more a plugin lock that is specifically protecting the maps.
It grants multiple reader to proceed in SC plugin but only one writer at
a time and no reader when the writer is proceeding. So instead of using
map lock, a new RW **plugin_lock** is used.

A second patch was

-  Registering new BE_PREOP and BE_POSTOP callback to acquire/release
   **plugin_lock**
-  BE_PREOP acquire **plugin_lock** whatever is the scope of the
   operation
-  Using per thread variables to make the lock reentrant
-  Using a new **plugin_lock** (initialized in plugin init function)
-  Update *map_unlock, map_rdlock and map_wrlock* so that they
   acquire/release **plugin_lock**.

   -  in a write thread, map_rdlock and map_wrlock should never acquire
      **plugin_lock** as this is the job of BE_PREOP to actually acquire
      it.
   -  similarly in a write thead, *map_unlock* should never release
      **plugin_lock** because it is the job of BE_POSTOP
   -  only a read thread can acquire/release **plugin_lock** (in read)

This second patch is fixing the `reproducible test
case <https://bugzilla.redhat.com/attachment.cgi?id=1315173>`__.
**\\o/** Indeed, the reproducible test case was reproducing almost
systematically the problem ( > 3 times out of 4). The fix makes it
successful almost more than 9 times out of 10. The last time it hit an
other issue.



Case not fixed
^^^^^^^^^^^^^^

An other issue is related to plugins or tasks **starting transaction**
and accessing DB pages (under the txn) before doing internal updates.
Indeed, the txn will lock the DB pages (in read) that can not be access
(even in read) by others threads.

The deadlock is then looking like

::

   | ``  thread 11 read a DB page (index) under a txn, then adds an entry ``
   | ``          dnaHostname=``\ ``+dnaPortNum=389,cn=posix-ids,cn=dna,cn=ipa,cn=etc,``
   | ``  ``
   | ``  ``
   | ``  Thread 11 (Thread 0x7f8861820700 (LWP 66191)):                                                                                               ``
   | ``  #0  0x00007f888d0a88e4 in futex_abstimed_wait ``
   | ``  #1  __pthread_rwlock_wrlock_full (abstime=0x0, rwlock=0x5559b577bc40) at pthread_rwlock_common.c:803``
   | ``  #2  __GI___pthread_rwlock_wrlock (rwlock=0x5559b577bc40) at pthread_rwlock_wrlock.c:27``
   | ``  #3  0x00007f887f826bed in backend_be_pre_write_cb ()``
   | ``  #4  0x00007f888f733aba in plugin_call_func ``
   | ``  #5  0x00007f888f733d44 in plugin_call_list ``
   | ``  #6  plugin_call_plugins ``
   | ``  #7  0x00007f88817e67c7 in ldbm_back_add ``
   | ``  #8  0x00007f888f6d7b12 in op_shared_add ``
   | ``  #9  0x00007f888f6d8333 in add_internal_pb ``
   | ``  #10 0x00007f888f6d905e in slapi_add_internal_pb ``
   | ``  #11 0x00007f8883cdd3fc in dna_update_shared_config ``
   | ``      < here a txn is started>``
   | ``  #12 0x00007f8883ce0252 in dna_update_config_event ``
   | ``  #13 0x00007f888f6faa5c in eq_call_all ``
   | ``  #14 eq_loop (arg=``\ ``) ``
   | ``  #15 0x00007f888d70708b in _pt_root ``
   | ``  #16 0x00007f888d0a336d in start_thread ``
   | ``  #17 0x00007f888cb92bbf in clone () ``
   | ``  ``
   | ``  ===============================================``
   | ``  This thread acquires SC map lock but evaluate aci that need DB access``
   | ``  ``
   | ``  Thread 56 (Thread 0x7f884a7f2700 (LWP 66237)):``
   | ``  #0  0x00007f888d0a990b in futex_wait_cancelable (private=``\ ``, expected=0, futex_word=0x7f887a380a08) at ../sysdeps/unix/sysv/linux/futex-internal.h:88``
   | ``  ...``
   | ``  #14 0x00007f88858adef5 in __dbc_get ``
   | ``  ...``
   | ``  #25 0x00007f888f73884e in search_internal_callback_pb ``
   | ``  ..``
   | ``  #27 0x00007f8884b803da in acllas__user_ismember_of_group ``
   | ``  #41 0x00007f888f70052d in slapi_filter_test_ext ``
   | ``  #42 0x00007f887f829932 in backend_search_set_cb ()``
   | ``  #43 0x00007f887f839e4f in map_data_foreach_map ()``
   | ``  #44 0x00007f887f829616 in backend_search_group_cb ()``
   | ``  #45 0x00007f887f839d90 in map_data_foreach_domain ()``
   | ``  #46 0x00007f887f828cd8 in backend_search_cb ()``
   | ``  #47 0x00007f888f733aba in plugin_call_func ``
   | ``  #48 0x00007f888f733d44 in plugin_call_list ``
   | ``  #49 plugin_call_plugins ``
   | ``  #50 0x00007f888f728410 in op_shared_search ``
   | ``  #51 0x00005559b29ef577 in do_search ``

The db_stat output of the locked pages is showing Thread 11 transaction
(80005604) that is blocking many readers (in the way reproducible test
case runs)

| ``  80005604 READ          7 HELD    userRoot/entryrdn.db      page          3``
| ``       426 READ          1 WAIT    userRoot/entryrdn.db      page          3``
| ``       403 READ          1 WAIT    userRoot/entryrdn.db      page          3``
| ``       442 READ          1 WAIT    userRoot/entryrdn.db      page          3``
| ``       443 READ          1 WAIT    userRoot/entryrdn.db      page          3``
| ``        ......``