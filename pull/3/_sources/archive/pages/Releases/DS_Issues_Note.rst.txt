An update to 389-ds-base has been released in Fedora 17 which works
correctly with IPA. You can safely update to
389-ds-base-1.2.11.5-1.fc17.

Don't forget to remove 389-ds-base from excludes in yum.conf and/or use
yum versionlock delete 389-ds-base{,-devel,-libs}
