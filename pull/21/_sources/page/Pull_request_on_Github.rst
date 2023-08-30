Pull_request_on_Github
======================

At first you have to create your own fork of FreeIPA on
`github.com <https://github.com/freeipa/freeipa>`__. Please read how to
`here <https://help.github.com/articles/fork-a-repo/>`__.

Add your github fork to git remote branches in your local freeipa
repository (cloned from fedorahosted)

| ``$ git clone ``\ ```https://pagure.io/freeipa.git`` <https://pagure.io/freeipa.git>`__
| ``$ cd freeipa/``
| ``$ git remote add myfork git@github.com:``\ ``/freeipa.git``

Create a new branch (from master)

``$ git checkout -b fix-for-ticket-1234``

Put changes there, commit and do rebase. We do not allow merges so
commits must be applied on the top of current master branch.

``$ git rebase master fix-for-ticket-1234``

Push your commits to your Github fork and `create pull
request <https://help.github.com/articles/creating-a-pull-request/>`__
against **freeipa/freeipa**. Please add ticket to pull request (if
exists) and short summary.

``$ git push myfork``

If you need to make any modifications in pull request, edit the commits
locally and use force push to your branch in your Github fork. Changes
will be applied automatically to pull request.

``$ git push myfork --force``

When pull request is created, FreeIPA developers are automatically
notified. All review will happen on Github.

Please put link to your pull request to ticket field *Patch link* (if
fixes any ticket).