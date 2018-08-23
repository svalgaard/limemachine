Samba with TimeMachine support on Centos/Red Hat
================================================

New versions of SAMBA supports FULLSYNC which is a requirement for
Apple TimeMachine to work out-of-the-box for new versions of OS X:

   https://github.com/samba-team/samba/pull/64

Unfortunately Centos 7 and Red Hat 7 does not support FULLSYNC using
the standard SAMBA install. This show the steps required to fix this.


Requirements
------------

* Centos 7 / Red Hat 7
* samba-fullsync.patch


Resources
---------

* https://blog.packagecloud.io/eng/2015/04/20/working-with-source-rpms/


Step 1: Fetch Samba sources + dependencies
------------------------------------------

First, install yumdownloader + samba dependencies

    sudo yum install yum-utils rpm-build
    sudo yum-builddep samba


Get the sambda source rpm and save it in /tmp

    cd /tmp
    yumdownloader --source samba

Install the files inside the rpm in ~/rpmbuild. Replace
samba-4.7.1-9.el7_5.src.rpm with the filename of the actual file
downloaded in the previous step

    rpm -ivh samba-4.7.1-9.el7_5.src.rpm

Check for extra dependencies

    cd ~/rpmbuild/SPECS/
    rpmbuild -bp samba.spec


Step 2: Fetch + fix patch file
------------------------------

The original patch from github does not match cleanly with our version of SAMBA.

Fetch modified patch

    PURL=https://raw.githubusercontent.com/svalgaard/limemachine/master/patches/samba-fullsync.patch
    wget "$PURL" -O ~/rpmbuild/SOURCES/samba-fullsync.patch

Check that the patch applies

    cd ~/rpmbuild/BUILD/samba-4.7.1
    patch --dry-run -p1 < ~/rpmbuild/SOURCES/samba-fullsync.patch

It should say something along the line of this

    checking file docs-xml/manpages/vfs_fruit.8.xml
    checking file libcli/smb/smb2_create_ctx.h
    checking file source3/modules/vfs_fruit.c
    Hunk #1 succeeded at 140 (offset 1 line).
    Hunk #2 succeeded at 1890 (offset 340 lines).
    Hunk #3 succeeded at 2550 (offset 340 lines).
    Hunk #4 succeeded at 2974 with fuzz 1 (offset 329 lines).
    checking file source3/smbd/avahi_register.c
    Hunk #3 succeeded at 110 (offset 5 lines).
    Hunk #4 succeeded at 139 (offset 4 lines).
    Hunk #5 succeeded at 194 (offset 3 lines).
    Hunk #6 succeeded at 235 (offset 7 lines).

If it fails due to other changes in the files, fix them by editing the
patch file before proceeding.


Step 3: Update the spec file
----------------------------

We need to update the spec file to also use the extra patch file.  Use
your favourite editor and edit samba.spec

    cd ~/rpmbuild/SPECS/
    emacs -nw samba.spec

At the top of the file, increment the release version slightly to
ensure that yum prefers your samba version over the default (add .1 to
the version number)

    %define main_release 9.1

Add a line like this next to the other patch files (approximately line
131). You may need to renumber this:

    Patch13:   samba-fullsync.patch


Finally, build the new rpms

    cd ~/rpmbuild/SPECS/
    rpmbuild -ba samba.spec

RPMs are now located in ~/rpmbuild/RPMS/x86_64 and
~/rpmbuild/RPMS/noarch


Step 4: Install samba
---------------------

Go to the right directory and do a localinstall of your new files

    cd ~/rpmbuild/RPMS/x86_64
    sudo yum localinstall samba-4.7* samba-client-4.7* samba-client-libs-4.7* ../noarch/samba-common-4.7* samba-common-libs-4.7* samba-common-tools-4.7* samba-libs-4.7* libsmbclient-4.7* libwbclient-4.7*


You could also instead simply install all the rpm files

    sudo yum localinstall *.rpm ../noarch/*.rpm


Step 5: Setup samba
-------------------

This is not covered in this file.
