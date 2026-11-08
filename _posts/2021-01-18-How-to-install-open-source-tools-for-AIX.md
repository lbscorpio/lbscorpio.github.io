---
title:  How to install open source tools for AIX
date: 2021-01-18 18:57:50
categories:
  - Tools
  - Procedure
tags:
  - aix
  - gnu
  - yum
  - tar
  - gtar
---

AIX tar failed to extract some compressed file, you need to use GNU tar utility (aka gtar here) refer to <a href="https://www.ibm.com/support/pages/aix-tar-command-not-able-uncompress-ci0i7mlop601-media-package" target="_blank">this KB</a>.

There is an AIX Toolbox for Linux Applications, which contains a collection of open source and GNU software built for AIX IBM Systems.

## Steps to install AIX Toolbox for Linux Applications

- Download YUM installation script <a href="http://ftp.software.ibm.com/aix/freeSoftware/aixtoolbox/ezinstall/ppc/yum.sh" target="_blank">yum.sh</a> directly, then upload it to your AIX.

- Issuing below commands to install rpm.rte and all the packages needed for yum

      # chmod +x yum.sh
      # ./yum.sh
      # yum update             // Update packages to the latest level

## Install the required packages via YUM command

- Issuing the yum command like Linux world

      # yum install -y tar
      # gtar --help

## References

<a href="https://www.ibm.com/support/pages/aix-toolbox-linux-applications-downloads-alpha" target="_blank">AIX Toolbox for Linux Applications</a>  

- The output of running yum.sh

      # ./yum.sh
      Attempting download of rpm.rte & yum_bundle.tar ...
      Installing rpm.rte at the latest version ...
      This may take several minutes depending on the number of rpms installed...
      +-----------------------------------------------------------------------------+
                          Pre-installation Verification...
      +-----------------------------------------------------------------------------+
      Verifying selections...done
      Verifying requisites...done
      Results...
      
      SUCCESSES
      ---------
        Filesets listed in this section passed pre-installation verification
        and will be installed.
      
        Selected Filesets
        -----------------
        rpm.rte 4.15.1.1                            # RPM Package Manager
      
        << End of Success Section >>
      
      +-----------------------------------------------------------------------------+
                         BUILDDATE Verification ...
      +-----------------------------------------------------------------------------+
      Verifying build dates...done
      FILESET STATISTICS
      ------------------
          1  Selected to be installed, of which:
              1  Passed pre-installation verification
        ----
          1  Total to be installed
      
      +-----------------------------------------------------------------------------+
                               Installing Software...
      +-----------------------------------------------------------------------------+
      
      installp:  APPLYING software for:
              rpm.rte 4.15.1.1
      
      Checking if rpm package is already owning
        some of the libraries and binaries..
      
      . . . . . << Copyright notice for rpm.rte >> . . . . . . .
       Licensed Materials - Property of IBM
      
       5765H4000
         Copyright International Business Machines Corp. 2014, 2020.
         Copyright "Cowboy" Ben Alman, 2010.
         Copyright Steven Benner (http://stevenbenner.com/), 2013.
         Copyright The Regents of the University of California, 1990, 1993, 1994.
         Copyright Ralf Corsepius <corsepiu@fedoraproject.org>, 2007.
         Copyright Vasil Dinkov, Vadikom Web Ltd., 2016.
         Copyright The Dojo Foundation, 2011.
         Copyright Ulrich Drepper <drepper@gnu.ai.mit.edu>, 1995-1997, 2000-2006.
         Copyright Tom Dyas <tdyas@eden.rutgers.edu>, 1998.
         Copyright W. L. Estes <wlestes@uncg.edu>, 1998.
         Copyright Free Software Foundation, Inc., 1989, 1991-2018.
         Copyright Dave Furfero, 2011-2014.
         Copyright Neal Gompa <ngompa13@gmail.com>, 2015.
         Copyright Dimitri van Heesch, 1997-2017.
         Copyright The Internet Society (1998).
         Copyright Per Oeyvind Karlsen <proyvind@moondrake.org>, 2010.
         Copyright Dmitry V. Levin <ldv@altlinux.org>, 2019.
         Copyright Dan Nicholson <dbn.lists@gmail.com>, 2012-2015.
         Copyright Red Hat, Inc., 1998-2003, 2005, 2007, 2009-2011, 2016-2017.
         Copyright Red Hat Software, 1995-2002.
         Copyright Scott James Remnant <scott@netsplit.com>, 2004.
         Copyright John Resig, 2011.
         Copyright Enrico Scholz <enrico.scholz@informatik.tu-chemnitz.de>, 2004.
         Copyright System Design and Research, 2003.
         Copyright Julian Seward et al., 2002-2015.
         Copyright Mark J. Wielaard <mark@klomp.org>, 2016, 2019.
         Copyright X Consortium, 1994.
      
       All rights reserved.
       US Government Users Restricted Rights - Use, duplication or disclosure
       restricted by GSA ADP Schedule Contract with IBM Corp.
      . . . . . << End of copyright notice for rpm.rte >>. . . .
      
      ##################################################################
        This version of rpm installs files under the path /usr/opt/rpm.
        More detailed information on new changes can be found in
        /usr/opt/rpm/README file.
      ##################################################################
      
      Please wait...
      Successfully updated the Kernel Authorization Table.
      Successfully updated the Kernel Role Table.
      Successfully updated the Kernel Command Table.
      Successfully updated the Kernel Device Table.
      Successfully updated the Kernel Object Domain Table.
      Successfully updated the Kernel Domains Table.
      Successfully updated the Kernel RBAC log level.
      Successfully updated the Kernel Authorization Table.
      Successfully updated the Kernel Role Table.
      Successfully updated the Kernel Command Table.
      Successfully updated the Kernel Device Table.
      Successfully updated the Kernel Object Domain Table.
      Successfully updated the Kernel Domains Table.
      Successfully updated the Kernel RBAC log level.
      Finished processing all filesets.  (Total time:  58 secs).
      
      +-----------------------------------------------------------------------------+
                                      Summaries:
      +-----------------------------------------------------------------------------+
      
      Installation Summary
      --------------------
      Name                        Level           Part        Event       Result
      -------------------------------------------------------------------------------
      rpm.rte                     4.15.1.1        USR         APPLY       SUCCESS
      rpm.rte                     4.15.1.1        ROOT        APPLY       SUCCESS
      
      Extracting yum_bundle.tar ...
      x ca-certificates-2016.10.7-2.aix6.1.ppc.rpm, 214726 bytes, 420 media blocks.
      x curl-7.52.1-1.aix6.1.ppc.rpm, 533288 bytes, 1042 media blocks.
      x db-4.8.24-3.aix6.1.ppc.rpm, 2897799 bytes, 5660 media blocks.
      x gdbm-1.8.3-5.aix5.2.ppc.rpm, 56991 bytes, 112 media blocks.
      x gettext-0.19.7-1.aix6.1.ppc.rpm, 4036762 bytes, 7885 media blocks.
      x glib2-2.14.6-2.aix5.2.ppc.rpm, 1686134 bytes, 3294 media blocks.
      x pysqlite-1.1.7-2.aix6.1.ppc.rpm, 51749 bytes, 102 media blocks.
      x python-2.7.10-1.aix6.1.ppc.rpm, 23333701 bytes, 45574 media blocks.
      x python-devel-2.7.10-1.aix6.1.ppc.rpm, 15366474 bytes, 30013 media blocks.
      x python-iniparse-0.4-1.aix6.1.noarch.rpm, 37912 bytes, 75 media blocks.
      x python-pycurl-7.19.3-1.aix6.1.ppc.rpm, 162093 bytes, 317 media blocks.
      x python-tools-2.7.10-1.aix6.1.ppc.rpm, 830446 bytes, 1622 media blocks.
      x python-urlgrabber-3.10.1-1.aix6.1.noarch.rpm, 158584 bytes, 310 media blocks.
      x readline-6.1-2.aix6.1.ppc.rpm, 489547 bytes, 957 media blocks.
      x sqlite-3.15.2-1.aix6.1.ppc.rpm, 3570302 bytes, 6974 media blocks.
      x yum-3.4.3-8.aix6.1.noarch.rpm, 1385622 bytes, 2707 media blocks.
      x yum-metadata-parser-1.1.4-2.aix6.1.ppc.rpm, 62283 bytes, 122 media blocks.
      
      
      Checking whether any of the rpms from yum_bundle are already installed ...
      
      ca-certificates-2016.10.7-2 will be installed ...
      curl-7.52.1-1 will be installed ...
      db-4.8.24-3 will be installed ...
      Skipping gdbm-1.8.3-5 as higher version is already installed.
      Please make sure these packages are from the Toolbox as there is no guarantee that
      third party packages are compatible with Toolbox packages.
      
      gettext-0.19.7-1 will be installed ...
      glib2-2.14.6-2 will be installed ...
      pysqlite-1.1.7-2 will be installed ...
      Skipping python-2.7.10-1 as higher version is already installed.
      Please make sure these packages are from the Toolbox as there is no guarantee that
      third party packages are compatible with Toolbox packages.
      
      Skipping python-devel-2.7.10-1 as higher version is already installed.
      Please make sure these packages are from the Toolbox as there is no guarantee that
      third party packages are compatible with Toolbox packages.
      
      python-iniparse-0.4-1 will be installed ...
      Package python-pycurl-7.19.3-1 is already installed
      Skipping python-tools-2.7.10-1 as higher version is already installed.
      Please make sure these packages are from the Toolbox as there is no guarantee that
      third party packages are compatible with Toolbox packages.
      
      Package python-urlgrabber-3.10.1-1 is already installed
      Skipping readline-6.1-2 as higher version is already installed.
      Please make sure these packages are from the Toolbox as there is no guarantee that
      third party packages are compatible with Toolbox packages.
      
      sqlite-3.15.2-1 will be installed ...
      yum-3.4.3-8 will be installed ...
      yum-metadata-parser-1.1.4-2 will be installed ...
      
      Installing the packages...
      
      Verifying...                          ################################# [100%]
      Preparing...                          ################################# [100%]
      Updating / installing...
         1:sqlite-3.15.2-1                  ################################# [ 10%]
         2:pysqlite-1.1.7-2                 ################################# [ 20%]
         3:python-iniparse-0.4-1            ################################# [ 30%]
      /
         4:gettext-0.19.7-1                 ################################# [ 40%]
      add libintl.so.1 (32bits) shared member to /opt/freeware/lib/libintl.a
      add libintl.so.1 (64bits) shared member to  /opt/freeware/lib/libintl.a
      /
         5:glib2-2.14.6-2                   ################################# [ 50%]
         6:yum-metadata-parser-1.1.4-2      ################################# [ 60%]
         7:ca-certificates-2016.10.7-2      ################################# [ 70%]
         8:curl-7.52.1-1                    ################################# [ 80%]
         9:yum-3.4.3-8                      ################################# [ 90%]
        10:db-4.8.24-3                      ################################# [100%]
      
      Yum installed successfully.
      Please run 'yum update' to update packages to the latest level.

- The output of installing GNU tar utility

      # yum install tar
      Setting up Install Process
      Resolving Dependencies
      --> Running transaction check
      ---> Package tar.ppc 0:1.32-1 will be installed
      --> Finished Dependency Resolution
      
      Dependencies Resolved
      
      ========================================================================================================================================================================================================================================
       Package                                              Arch                                                 Version                                                   Repository                                                    Size
      ========================================================================================================================================================================================================================================
      Installing:
       tar                                                  ppc                                                  1.32-1                                                    AIX_Toolbox                                                  1.5 M
      
      Transaction Summary
      ========================================================================================================================================================================================================================================
      Install       1 Package
      
      Total download size: 1.5 M
      Installed size: 1.5 M
      Is this ok [y/N]: y
      Downloading Packages:
      tar-1.32-1.aix6.1.ppc.rpm                                                                                                                                                                                        | 1.5 MB  00:00:01
      Running Transaction Check
      Running Transaction Test
      Transaction Test Succeeded
      Running Transaction
        Installing : tar-1.32-1.ppc                                                                                                                                                                                                       1/1
      
      Installed:
        tar.ppc 0:1.32-1
      
      Complete!
      
- The output of the gtar help

      # gtar --help
      Usage: gtar [OPTION...] [FILE]...
      GNU 'tar' saves many files together into a single tape or disk archive, and can
      restore individual files from the archive.
      
      Examples:
        tar -cf archive.tar foo bar  # Create archive.tar from files foo and bar.
        tar -tvf archive.tar         # List all files in archive.tar verbosely.
        tar -xf archive.tar          # Extract all files from archive.tar.
      
       Local file name selection:
      
            --add-file=FILE        add given FILE to the archive (useful if its name
                                   starts with a dash)
        -C, --directory=DIR        change to directory DIR
            --exclude=PATTERN      exclude files, given as a PATTERN
            --exclude-backups      exclude backup and lock files
            --exclude-caches       exclude contents of directories containing
                                   CACHEDIR.TAG, except for the tag file itself
            --exclude-caches-all   exclude directories containing CACHEDIR.TAG
            --exclude-caches-under exclude everything under directories containing
                                   CACHEDIR.TAG
            --exclude-ignore=FILE  read exclude patterns for each directory from
                                   FILE, if it exists
            --exclude-ignore-recursive=FILE
                                   read exclude patterns for each directory and its
                                   subdirectories from FILE, if it exists
            --exclude-tag=FILE     exclude contents of directories containing FILE,
                                   except for FILE itself
            --exclude-tag-all=FILE exclude directories containing FILE
            --exclude-tag-under=FILE   exclude everything under directories
                                   containing FILE
            --exclude-vcs          exclude version control system directories
            --exclude-vcs-ignores  read exclude patterns from the VCS ignore files
            --no-null              disable the effect of the previous --null option
            --no-recursion         avoid descending automatically in directories
            --no-unquote           do not unquote input file or member names
            --no-verbatim-files-from   -T treats file names starting with dash as
                                   options (default)
            --null                 -T reads null-terminated names; implies
                                   --verbatim-files-from
            --recursion            recurse into directories (default)
        -T, --files-from=FILE      get names to extract or create from FILE
            --unquote              unquote input file or member names (default)
            --verbatim-files-from  -T reads file names verbatim (no escape or option
                                   handling)
        -X, --exclude-from=FILE    exclude patterns listed in FILE
      
       File name matching options (affect both exclude and include patterns):
      
            --anchored             patterns match file name start
            --ignore-case          ignore case
            --no-anchored          patterns match after any '/' (default for
                                   exclusion)
            --no-ignore-case       case sensitive matching (default)
            --no-wildcards         verbatim string matching
            --no-wildcards-match-slash   wildcards do not match '/'
            --wildcards            use wildcards (default for exclusion)
            --wildcards-match-slash   wildcards match '/' (default for exclusion)
      
       Main operation mode:
      
        -A, --catenate, --concatenate   append tar files to an archive
        -c, --create               create a new archive
            --delete               delete from the archive (not on mag tapes!)
        -d, --diff, --compare      find differences between archive and file system
        -r, --append               append files to the end of an archive
        -t, --list                 list the contents of an archive
            --test-label           test the archive volume label and exit
        -u, --update               only append files newer than copy in archive
        -x, --extract, --get       extract files from an archive
      
       Operation modifiers:
      
            --check-device         check device numbers when creating incremental
                                   archives (default)
        -g, --listed-incremental=FILE   handle new GNU-format incremental backup
        -G, --incremental          handle old GNU-format incremental backup
            --hole-detection=TYPE  technique to detect holes
            --ignore-failed-read   do not exit with nonzero on unreadable files
            --level=NUMBER         dump level for created listed-incremental archive
            --no-check-device      do not check device numbers when creating
                                   incremental archives
        -n, --seek                 archive is seekable
            --no-seek              archive is not seekable
            --occurrence[=NUMBER]  process only the NUMBERth occurrence of each file
                                   in the archive; this option is valid only in
                                   conjunction with one of the subcommands --delete,
                                   --diff, --extract or --list and when a list of
                                   files is given either on the command line or via
                                   the -T option; NUMBER defaults to 1
            --sparse-version=MAJOR[.MINOR]
                                   set version of the sparse format to use (implies
                                   --sparse)
        -S, --sparse               handle sparse files efficiently
      
       Overwrite control:
      
            --keep-directory-symlink   preserve existing symlinks to directories when
                                   extracting
        -k, --keep-old-files       don't replace existing files when extracting,
                                   treat them as errors
            --keep-newer-files     don't replace existing files that are newer than
                                   their archive copies
            --no-overwrite-dir     preserve metadata of existing directories
            --one-top-level[=DIR]  create a subdirectory to avoid having loose files
                                   extracted
            --overwrite            overwrite existing files when extracting
            --overwrite-dir        overwrite metadata of existing directories when
                                   extracting (default)
            --recursive-unlink     empty hierarchies prior to extracting directory
            --remove-files         remove files after adding them to the archive
            --skip-old-files       don't replace existing files when extracting,
                                   silently skip over them
        -U, --unlink-first         remove each file prior to extracting over it
        -W, --verify               attempt to verify the archive after writing it
      
       Select output stream:
      
            --ignore-command-error ignore exit codes of children
            --no-ignore-command-error   treat non-zero exit codes of children as
                                   error
        -O, --to-stdout            extract files to standard output
            --to-command=COMMAND   pipe extracted files to another program
      
       Handling of file attributes:
      
            --atime-preserve[=METHOD]   preserve access times on dumped files, either
                                   by restoring the times after reading
                                   (METHOD='replace'; default) or by not setting the
                                   times in the first place (METHOD='system')
            --clamp-mtime          only set time when the file is more recent than
                                   what was given with --mtime
            --delay-directory-restore   delay setting modification times and
                                   permissions of extracted directories until the end
                                   of extraction
            --group=NAME           force NAME as group for added files
            --group-map=FILE       use FILE to map file owner GIDs and names
            --mode=CHANGES         force (symbolic) mode CHANGES for added files
            --mtime=DATE-OR-FILE   set mtime for added files from DATE-OR-FILE
        -m, --touch                don't extract file modified time
            --no-delay-directory-restore
                                   cancel the effect of --delay-directory-restore
                                   option
            --no-same-owner        extract files as yourself (default for ordinary
                                   users)
            --no-same-permissions  apply the user's umask when extracting permissions
                                   from the archive (default for ordinary users)
            --numeric-owner        always use numbers for user/group names
            --owner=NAME           force NAME as owner for added files
            --owner-map=FILE       use FILE to map file owner UIDs and names
        -p, --preserve-permissions, --same-permissions
                                   extract information about file permissions
                                   (default for superuser)
            --same-owner           try extracting files with the same ownership as
                                   exists in the archive (default for superuser)
        -s, --preserve-order, --same-order
                                   member arguments are listed in the same order as
                                   the files in the archive
            --sort=ORDER           directory sorting order: none (default), name or
                                   inode
      
       Handling of extended file attributes:
      
            --acls                 Enable the POSIX ACLs support
            --no-acls              Disable the POSIX ACLs support
            --no-selinux           Disable the SELinux context support
            --no-xattrs            Disable extended attributes support
            --selinux              Enable the SELinux context support
            --xattrs               Enable extended attributes support
            --xattrs-exclude=MASK  specify the exclude pattern for xattr keys
            --xattrs-include=MASK  specify the include pattern for xattr keys
      
       Device selection and switching:
      
        -f, --file=ARCHIVE         use archive file or device ARCHIVE
            --force-local          archive file is local even if it has a colon
        -F, --info-script=NAME, --new-volume-script=NAME
                                   run script at end of each tape (implies -M)
        -L, --tape-length=NUMBER   change tape after writing NUMBER x 1024 bytes
        -M, --multi-volume         create/list/extract multi-volume archive
            --rmt-command=COMMAND  use given rmt COMMAND instead of rmt
            --rsh-command=COMMAND  use remote COMMAND instead of rsh
            --volno-file=FILE      use/update the volume number in FILE
      
       Device blocking:
      
        -b, --blocking-factor=BLOCKS   BLOCKS x 512 bytes per record
        -B, --read-full-records    reblock as we read (for 4.2BSD pipes)
        -i, --ignore-zeros         ignore zeroed blocks in archive (means EOF)
            --record-size=NUMBER   NUMBER of bytes per record, multiple of 512
      
       Archive format selection:
      
        -H, --format=FORMAT        create archive of the given format
      
       FORMAT is one of the following:
      
          gnu                      GNU tar 1.13.x format
          oldgnu                   GNU format as per tar <= 1.12
          pax                      POSIX 1003.1-2001 (pax) format
          posix                    same as pax
          ustar                    POSIX 1003.1-1988 (ustar) format
          v7                       old V7 tar format
      
            --old-archive, --portability
                                   same as --format=v7
            --pax-option=keyword[[:]=value][,keyword[[:]=value]]...
                                   control pax keywords
            --posix                same as --format=posix
        -V, --label=TEXT           create archive with volume name TEXT; at
                                   list/extract time, use TEXT as a globbing pattern
                                   for volume name
      
       Compression options:
      
        -a, --auto-compress        use archive suffix to determine the compression
                                   program
        -I, --use-compress-program=PROG
                                   filter through PROG (must accept -d)
        -j, --bzip2                filter the archive through bzip2
        -J, --xz                   filter the archive through xz
            --lzip                 filter the archive through lzip
            --lzma                 filter the archive through lzma
            --lzop                 filter the archive through lzop
            --no-auto-compress     do not use archive suffix to determine the
                                   compression program
        -z, --gzip, --gunzip, --ungzip   filter the archive through gzip
            --zstd                 filter the archive through zstd
        -Z, --compress, --uncompress   filter the archive through compress
      
       Local file selection:
      
            --backup[=CONTROL]     backup before removal, choose version CONTROL
            --hard-dereference     follow hard links; archive and dump the files they
                                   refer to
        -h, --dereference          follow symlinks; archive and dump the files they
                                   point to
        -K, --starting-file=MEMBER-NAME
                                   begin at member MEMBER-NAME when reading the
                                   archive
            --newer-mtime=DATE     compare date and time when data changed only
        -N, --newer=DATE-OR-FILE, --after-date=DATE-OR-FILE
                                   only store files newer than DATE-OR-FILE
            --one-file-system      stay in local file system when creating archive
        -P, --absolute-names       don't strip leading '/'s from file names
            --suffix=STRING        backup before removal, override usual suffix ('~'
                                   unless overridden by environment variable
                                   SIMPLE_BACKUP_SUFFIX)
      
       File name transformations:
      
            --strip-components=NUMBER   strip NUMBER leading components from file
                                   names on extraction
            --transform=EXPRESSION, --xform=EXPRESSION
                                   use sed replace EXPRESSION to transform file
                                   names
      
       Informative output:
      
            --checkpoint[=NUMBER]  display progress messages every NUMBERth record
                                   (default 10)
            --checkpoint-action=ACTION   execute ACTION on each checkpoint
            --full-time            print file time to its full resolution
            --index-file=FILE      send verbose output to FILE
        -l, --check-links          print a message if not all links are dumped
            --no-quote-chars=STRING   disable quoting for characters from STRING
            --quote-chars=STRING   additionally quote characters from STRING
            --quoting-style=STYLE  set name quoting style; see below for valid STYLE
                                   values
        -R, --block-number         show block number within archive with each message
      
            --show-defaults        show tar defaults
            --show-omitted-dirs    when listing or extracting, list each directory
                                   that does not match search criteria
            --show-snapshot-field-ranges
                                   show valid ranges for snapshot-file fields
            --show-transformed-names, --show-stored-names
                                   show file or archive names after transformation
            --totals[=SIGNAL]      print total bytes after processing the archive;
                                   with an argument - print total bytes when this
                                   SIGNAL is delivered; Allowed signals are: SIGHUP,
                                   SIGQUIT, SIGINT, SIGUSR1 and SIGUSR2; the names
                                   without SIG prefix are also accepted
            --utc                  print file modification times in UTC
        -v, --verbose              verbosely list files processed
            --warning=KEYWORD      warning control
        -w, --interactive, --confirmation
                                   ask for confirmation for every action
      
       Compatibility options:
      
        -o                         when creating, same as --old-archive; when
                                   extracting, same as --no-same-owner
      
       Other options:
      
        -?, --help                 give this help list
            --restrict             disable use of some potentially harmful options
            --usage                give a short usage message
            --version              print program version
      
      Mandatory or optional arguments to long options are also mandatory or optional
      for any corresponding short options.
      
      The backup suffix is '~', unless set with --suffix or SIMPLE_BACKUP_SUFFIX.
      The version control may be set with --backup or VERSION_CONTROL, values are:
      
        none, off       never make backups
        t, numbered     make numbered backups
        nil, existing   numbered if numbered backups exist, simple otherwise
        never, simple   always make simple backups
      
      Valid arguments for the --quoting-style option are:
      
        literal
        shell
        shell-always
        shell-escape
        shell-escape-always
        c
        c-maybe
        escape
        locale
        clocale
      
      *This* tar defaults to:
      --format=gnu -f- -b20 --quoting-style=escape
      --rmt-command=/opt/freeware/libexec/rmt --rsh-command=/usr/bin/remsh

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.