# Release

- [Release](#release)
  - [Pre-release repository changes](#pre-release-repository-changes)
    - [Translations](#translations)
    - [Update Contributors.txt](#update-contributorstxt)
    - [Create release notes](#create-release-notes)
  - [Tagging the tree](#tagging-the-tree)
    - [Process](#process)
    - [Creating a new branch](#creating-a-new-branch)
  - [Building the sources](#building-the-sources)
    - [Create tarball](#create-tarball)
      - [IPA 4.5+](#ipa-45)
      - [IPA 4.4 and older](#ipa-44-and-older)
    - [Sign and upload tarball](#sign-and-upload-tarball)
    - [Re-enable git snapshot versioning](#re-enable-git-snapshot-versioning)
  - [Updating the COPR repository](#updating-the-copr-repository)
  - [Releasing to Fedora](#releasing-to-fedora)
    - [Spec file changes](#spec-file-changes)
    - [Closing Bugzillas](#closing-bugzillas)
  - [Release to PyPI](#release-to-pypi)
    - [Test PyPI](#test-pypi)
    - [PyPI](#pypi)
  - [Doing a Major Release](#doing-a-major-release)
  - [Tickets administravia](#tickets-administravia)



Pre-release repository changes
------------------------------

### Translations

Translations are handled by Weblate platform at [https://translate.fedoraproject.org/projects/freeipa/](https://translate.fedoraproject.org/projects/freeipa/) Weblate periodically pushes updates as pull requests to Github mirror of FreeIPA. However, \`\`ipa.pot\`\` template needs to be updated beforehand so that Weblate would see new untranslated strings.

Update ipa.pot file:

**IPA 4.5+**

```
$ autoreconf -i
$ ./configure
$ make -C po ipa.pot-update
\`\`ipa.pot\`\` is always generated during build, there is no \`\`ipa.pot\`\` in git tree in versions before 4.8.8. For newer versions we commit \`\`ipa.pot\`\` due to integration with Weblate.
```

**IPA 4.4 and lower**

```
$ make -C install/po update-pot
**!!!** Don't forgot to put these changes to commit with translations.
```

The changes to \`\`ipa.pot\`\` have to be submitted as a separate pull request upstream. Note that currently we have several issues with Weblate ordering of translations in the generated PO files, so pull requests from Weblate and \`\`ipa.pot\`\` file changes are a bit "noisy".

Remove empty translations from \*.po files

**IPA 4.5+**
```
$ make -C po strip-po
$ make -C po strip-pot
```

**IPA 4.4 and older**
```
$ make -C install/po update-po
```

Push (send patch) with new translations
```
$ git add
$ git commit
$ git push
```
### Update Contributors.txt

Make sure that [Contributors.txt](https://github.com/freeipa/freeipa/blob/master/Contributors.txt) file is updated with the Developer or other contributors. [update-contributors.py](https://github.com/freeipa/freeipa-tools/blob/master/update-contributors.py) can be used for automating this task.

  

### Create release notes

As first create draft and send it on review on freeipa-devel list.

Use script from [FreeIPA release tools](https://github.com/freeipa/freeipa-tools/tree/master/release) against a clean HTTP clone of [https://github.com/freeipa/freeipa](https://github.com/freeipa/freeipa).

Example:

```
python3 release/release-notes.py 4.9.7 2021-08-19 4.9.6 4.9   release-4-9-6..origin/ipa-4-9 "FreeIPA 4.9"  --links --token-file ~/.ipa/pagure.token  --nomilestones --repo=/home/USER/src/freeipa-clean/ > release\_notes-4-9-7
```

Use options _\--links_ to generate wiki format links for wiki as well.

See the TODO in the output. Remove extraneous tickets and fill the Enhancements and Known Issues sections out.

The generated release notes are in mediawiki format. You can use pandoc to
convert them to RST format:

```
pandoc -f mediawiki -t rst release_notes-o release_notes.rst
```

Make sure to include a title page as well at the top of the file:

```
FreeIPA X.Y.Z
=============

<release notes content>
```

You can use pandoc to convert the generated release notes to asciidoc format.

```
pandoc -f mediawiki -t asciidoc release-notes.mediawiki -o release-notes.txt
```

And then use sed to convert it to adapt it for email use:

```
sed -e "/^::/d;/ ;;/d;/^'''''/d;/^\\\[\\\[.\*\\$/d;" -i release-notes.txt
```

You can send this announcement to freeipa-users and freeipa-devel mailing lists.

Tagging the tree
----------------

The IPA master source is on pagure. Do all git work directly against a clean pull from there, named freeipa-writable to differenciate it from your devel pull.

Examples for our current tagging scheme:

*   alpha\_1-4-0-1 - first alpha of 4.0.1
*   beta\_3-4-0-1 - third beta of 4.0.1
*   rc\_2-4-0-1 - second release candidate of 4.0.1
*   release-4-0-1 - 4.0.1 final

### Process

Once we've confirmed that the tree is frozen, that all bugs we want fixed are fixed, we create a new tag:

```
$ git checkout master
$ git fetch
$ git rebase origin/master
```

**FixMe: checkout the proper branch instead. Do not tag master!**

Bump FreeIPA in VERSION\[.m4\], make sure that git snapshot versioning is disabled (package from tagged commit and tarball must have the same version) and commit:

```
$ vim VERSION.m4
  \* set versions
  \* set IPA\_VERSION\_IS\_GIT\_SNAPSHOT to  no
```
```
$ git commit -a -m "Become IPA X.Y.Z"
```

Make a _signed_ tag:

```
$ git tag -s -m "tagging IPAv4 X.Y.Z" release-x-y-z
$ git verify-tag release-x-y-z
```
Note: this step requires you to have properly [configured git for signing](http://blog.tomaskrizek.com/2017/03/11/code-signing-in-git/).

If you are signing remotely you may need to set environment variable `export GPG_TTY=$(tty)` and be sure to have the pinentry package installed.

Check the pushed changes (version update commit + tags) and push when OK:

```
$ git push origin <remote-branch> --tags --dry-run
```

### Creating a new branch

We create branches on new X or Y releases. Use:
```
$ git log master
```
and note down the commit id of the 'Become IPA x.y' commit, then create the branch locally:
```
$ git branch ipa-x-y <commit-id>
```
updating branching information for [fastcheck](/web/20221001164305/https://freeipa.org/page/Testing#Fast_test "Testing"):

```
$ vim VERSION.m4
 \* change IPA\_GIT\_BRANCH to define(IPA\_GIT\_BRANCH, ipa-IPA\_VERSION\_MAJOR-IPA\_VERSION\_MINOR)
$ git commit ...
```

finally push the new branch upstream:

```
$ git push origin ipa-x-y
```

**Weblate**: Please create new component in Weblate for the new branch.

Building the sources
--------------------

If doing a pre release pull a fresh tree to do the builds into a separate directory:

```
$ git clone [https://pagure.io/freeipa.git](https://pagure.io/freeipa.git) freeipa-x-y-z
$ cd freeipa-x-y-z
```

Make absolutely sure I have the right code by switching to the tag:

```
$ git checkout release-x-y-z
```

### Create tarball

#### IPA 4.5+

Create tarball

```
$ cat VERSION.m4
   _IPA\_VERSION\_IS\_GIT\_SNAPSHOT_ must be  **no**
```

If _IPA\_VERSION\_IS\_GIT\_SNAPSHOT_ == **yes** go back to previous section, set **no** and tag again! Tarball must be the same as tagged version

```
$ ./autogen.sh
$ make dist
```

#### IPA 4.4 and older

Make RPMs:

```
$ make IPA\_VERSION\_IS\_GIT\_SNAPSHOT=no rpms
```

### Sign and upload tarball

Create a detached signature:

```
$ gpg2 --default-key <IPA MASTER KEYID> --armor --detach-sign freeipa-x.y.z.tar.gz
$ gpg2 --verify freeipa-x.y.z.tar.gz.asc
```

Upload tarball and signature to pagure.io: [https://pagure.io/freeipa/releases](https://pagure.io/freeipa/releases)

If you signing key is not in the [Verify Release Signature](/web/20221001164305/https://freeipa.org/page/Verify_Release_Signature "Verify Release Signature") guide, append it to the list and gpg verify command.

### Re-enable git snapshot versioning

After you have created and signed the tarball, you must change _IPA\_VERSION\_IS\_GIT\_SNAPSHOT_ in VERSION.m4 back to **yes**.

Updating the COPR repository
----------------------------

When a new version is released, it has to be added into the corresponding [COPR repository](https://copr.fedorainfracloud.org/groups/g/freeipa/coprs/). The dependencies also need to be checked:

*   note the Fedora versions for the release, listed in the \*Description\* of the COPR repository (for instance ipa-4-6 branch is for Fedora 26 and Fedora 27)
*   for each Fedora version found above, install the freeipa-\* packages into a system which has been updated with packages coming exclusively from fedora, fedora-updates and the COPR repository
*   check that the installed version is the one that was just released into COPR. If an older version is picked, it means that some dependencies are missing. You can run _sudo dnf update freeipa-server_ to find the list of missing dependencies, for instance:

```
nothing provides pki-ca >= 10.5.1-2 needed by freeipa-server-4.6.2-0.fc26.x86\_64
```
*   if some dependencies are missing, you will have to add them to the copr repository

Releasing to Fedora
-------------------

All FreeIPA releases (except some internal Alpha or Beta versions) are released also to Fedora. See guidelines and help regarding the release process in [Fedoraproject.org HOWTO](http://fedoraproject.org/wiki/Package_update_HOWTO).

### Spec file changes

Fedora downstream spec file is usually being rebased to upstream version when a new Fedora is out. Then only a diff between versions is being applied (assuming you are in a directory with FreeIPA repo which is checked out on the new tag):

```
$ git diff release-x-y-z freeipa.spec.in > /tmp/spec.patch  # release-x-y-z is last released version
$ sed -i "s/freeipa.spec.in/freeipa.spec/g" /tmp/spec.patch
$ cd /path/to/fedora/freeipa/repo
$ patch -p1 < /tmp/spec.patch
```

Then, you just need to manually merge any conflicting changes (changelog entries are not merged) and you are done.

Create a changelog entry with a link to the Release Notes page on the wiki. For a minor release, list the main bugs fixed in this release, and the major updated dependencies

### Closing Bugzillas

When a [Fedora update system](https://admin.fedoraproject.org/updates/) entry is being created, add all related _POST_ Red Hat Bugzilla bugs of the released Fedora version to the list of fixed bugs ([link to query](https://bugzilla.redhat.com/buglist.cgi?bug_status=POST&classification=Fedora&columnlist=product%2Cversion%2Ccomponent%2Cassigned_to%2Cbug_status%2Cresolution%2Cshort_desc%2Cchangeddate&component=freeipa&list_id=1669877&product=Fedora)) to the list of fixed Bugzillas so that they are automatically closed when the package is released to Fedora repos.

Release to PyPI
---------------

FixMe: probably obsolete.

### Test PyPI

First try to upload FreeIPA to [testing instance of PyPI](https://test.pypi.org/) . Your account must be have permissions to upload to the PyPI packages freeipa, ipa, ipaclient, ipalib, ipapython, ipaplatform, and ipaserver on both PyPI and Test PyPI. The instances have separate user databases.

Create '~/.pypirc" config file as following

```
\[distutils\]
index-servers=
    pypi
    testpypi

\[testpypi\]
repository=[https://test.pypi.org/legacy/](https://test.pypi.org/legacy/)
username = <your user name goes here>
password = <your test password goes here>

\[pypi\]
repository=[https://upload.pypi.org/legacy/](https://upload.pypi.org/legacy/)
username = <your user name goes here>
password = <your live password optionally goes here>
```

First start by checking out the release tag and creating a clean build environment:

```
git checkout release-x-y-z
./autogen.sh
make clean
```

Create PyPI packages (both regular wheels and placeholder packages). The wheels are placed in 'dist/pypi':

```
make pypi\_packages
```

Upload wheels using _twine_ ([https://wiki.python.org/moin/TestPyPI](https://wiki.python.org/moin/TestPyPI)), replace 4.8.0 with correct version. It should be **eight** wheels in total.

```
twine upload --sign -r testpypi ./dist/pypi/\*-4.8.0-\*.whl
```

Test if Test PyPI works:

```
 python3 -m venv build/testenv3
 build/testenv3/bin/pip install --extra-index-url=[https://testpypi.python.org/pypi](https://testpypi.python.org/pypi) ipaclient==4.8.0
```

The pip command should download 4.6.1 from TestPyPI, all dependencies from main PyPI.

### PyPI

When Test PyPI was sucesfull, execute the same steps production version of [PyPI](https://pypi.python.org/pypi).

```
twine upload --sign ./dist/pypi/\*-4.8.0-\*.whl
```

Doing a Major Release
---------------------

When doing a major release, consider following extra steps:

*   [Downloads](/web/20221001164305/https://freeipa.org/page/Downloads "Downloads") - update list of active COPR repositories
*   [Roadmap](/web/20221001164305/https://freeipa.org/page/Roadmap "Roadmap") - consider updating information about planned or new releases
*   [Documentation](/web/20221001164305/https://freeipa.org/page/Documentation "Documentation") - update links and version numbers
*   [Build](/web/20221001164305/https://freeipa.org/page/Build "Build") - update COPR repositories if new ones are introduced or old abandoned

Tickets administravia
---------------------

FixMe: is this being done for 4.9.x?

*   [Create a milestone](https://fedorahosted.org/freeipa/milestone?action=new) for next FreeIPA version in form FreeIPA x.y.z.
*   Create a new milestone in [Pagure project settings](https://pagure.io/freeipa/settings) (if doesn't exist).
*   Move all [opened tickets](https://pagure.io/freeipa/roadmap?status=Open&milestone=FreeIPA+4.5.1&all_stones=True&no_stones=) from the released version milestone to the new milestone. (Following script can be used, use [move-milestone.py --help](https://raw.githubusercontent.com/freeipa/freeipa-tools/master/release/move-milestone.py) to see usage)
*   Close the old milestone in the [Roadmap](https://fedorahosted.org/freeipa/roadmap).