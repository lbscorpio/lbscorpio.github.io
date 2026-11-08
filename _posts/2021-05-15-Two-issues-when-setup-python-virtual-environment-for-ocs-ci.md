---
title:  Two issues when setup python virtual environment for ocs-ci
date: 2021-05-15 11:50:18
categories:
  - Troubleshoot
tags:
  - pd
  - ocs-ci
  - python
  - pip
---

Our project want to use ocs-ci framework for functions testing, I tried to setup the python virtual environment but met issues at the beginning.

## Environment

- macOS Big Sur 11.3.1

## Issue#1: clang version is old for Big Sur

- Ran command `pip install -r requirements.txt`, failed to build some python modules(e.g. greenlet, ovirt-engine-sdk-python), with below errors:

      Building wheel for greenlet (setup.py) ... error
       ERROR: Command errored out with exit status 1:
        command: /usr/local/opt/python@3.9/bin/python3.9 -u -c 'import io, os, sys, setuptools, tokenize; sys.argv[0] = '"'"'/private/tmp/pip-install-om3is4hw/greenlet_3cb1f77a7af24b5e8574173cfa87ab23/setup.py'"'"'; __file__='"'"'/private/tmp/pip-install-om3is4hw/greenlet_3cb1f77a7af24b5e8574173cfa87ab23/setup.py'"'"';f = getattr(tokenize, '"'"'open'"'"', open)(__file__) if os.path.exists(__file__) else io.StringIO('"'"'from setuptools import setup; setup()'"'"');code = f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' bdist_wheel -d /private/tmp/pip-wheel-yb365j4a
            cwd: /private/tmp/pip-install-om3is4hw/greenlet_3cb1f77a7af24b5e8574173cfa87ab23/
       Complete output (9 lines):
       running bdist_wheel
       running build
       running build_ext
       building 'greenlet' extension
       creating build
       creating build/temp.macosx-11-x86_64-3.9
       clang -Wno-unused-result -Wsign-compare -Wunreachable-code -fno-common -dynamic -DNDEBUG -g -fwrapv -O3 -Wall -isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk -I/usr/local/include -I/usr/local/opt/openssl@1.1/include -I/usr/local/opt/sqlite/include -I/usr/local/opt/python@3.9/Frameworks/Python.framework/Versions/3.9/include/python3.9 -c greenlet.c -o build/temp.macosx-11-x86_64-3.9/greenlet.o
       clang: error: invalid version number in 'MACOSX_DEPLOYMENT_TARGET=11'
       error: command '/usr/bin/clang' failed with exit code 1
       ----------------------------------------
       ERROR: Failed building wheel for greenlet
       Running setup.py clean for greenlet
       Building wheel for netifaces (setup.py) ... error
       ERROR: Command errored out with exit status 1:
        command: /usr/local/opt/python@3.9/bin/python3.9 -u -c 'import io, os, sys, setuptools, tokenize; sys.argv[0] = '"'"'/private/tmp/pip-install-om3is4hw/netifaces_ae9eb33f0f2442dda1f59ff905103134/setup.py'"'"'; __file__='"'"'/private/tmp/pip-install-om3is4hw/netifaces_ae9eb33f0f2442dda1f59ff905103134/setup.py'"'"';f = getattr(tokenize, '"'"'open'"'"', open)(__file__) if os.path.exists(__file__) else io.StringIO('"'"'from setuptools import setup; setup()'"'"');code = f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' bdist_wheel -d /private/tmp/pip-wheel-98dlfjs8
            cwd: /private/tmp/pip-install-om3is4hw/netifaces_ae9eb33f0f2442dda1f59ff905103134/
       Complete output (16 lines):
       running bdist_wheel
       running build
       running build_ext
       checking for getifaddrs...not found.
       checking for getnameinfo...not found.
       checking for socket IOCTLs...not found.
       checking for optional header files...none found.
       checking whether struct sockaddr has a length field...no.
       checking which sockaddr_xxx structs are defined...none!
       checking for routing socket support...no.
       checking for sysctl(CTL_NET...) support...no.
       checking for netlink support...no.
       building 'netifaces' extension
       clang -Wno-unused-result -Wsign-compare -Wunreachable-code -fno-common -dynamic -DNDEBUG -g -fwrapv -O3 -Wall -isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk -DNETIFACES_VERSION=0.10.9 -I/usr/local/include -I/usr/local/opt/openssl@1.1/include -I/usr/local/opt/sqlite/include -I/usr/local/opt/python@3.9/Frameworks/Python.framework/Versions/3.9/include/python3.9 -c netifaces.c -o build/temp.macosx-11-x86_64-3.9/netifaces.o
       clang: error: invalid version number in 'MACOSX_DEPLOYMENT_TARGET=11'
       error: command '/usr/bin/clang' failed with exit code 1
       ----------------------------------------
   or

        ERROR: Failed building wheel for ovirt-engine-sdk-python
        Running setup.py clean for ovirt-engine-sdk-python
      Failed to build ovirt-engine-sdk-python
      Installing collected packages: ovirt-engine-sdk-python
          Running setup.py install for ovirt-engine-sdk-python ... error
          ERROR: Command errored out with exit status 1:
           command: /Users/mars/workspace/ocs-ci/venv/bin/python3 -u -c 'import io, os, sys, setuptools, tokenize; sys.argv[0] = '"'"'/private/var/folders/hq/km332y3d1079j7l3hxq2v_cr0000gn/T/pip-install-fop84xho/ovirt-engine-sdk-python_399016e0f364412ab2b48e2f53c47070/setup.py'"'"'; __file__='"'"'/private/var/folders/hq/km332y3d1079j7l3hxq2v_cr0000gn/T/pip-install-fop84xho/ovirt-engine-sdk-python_399016e0f364412ab2b48e2f53c47070/setup.py'"'"';f = getattr(tokenize, '"'"'open'"'"', open)(__file__) if os.path.exists(__file__) else io.StringIO('"'"'from setuptools import setup; setup()'"'"');code = f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /private/var/folders/hq/km332y3d1079j7l3hxq2v_cr0000gn/T/pip-record-grrncx03/install-record.txt --single-version-externally-managed --compile --install-headers /Users/mars/workspace/ocs-ci/venv/include/site/python3.9/ovirt-engine-sdk-python
               cwd: /private/var/folders/hq/km332y3d1079j7l3hxq2v_cr0000gn/T/pip-install-fop84xho/ovirt-engine-sdk-python_399016e0f364412ab2b48e2f53c47070/
          Complete output (28 lines):
          4.4.12
          running install
          running build
          running build_py
          creating build
          creating build/lib.macosx-10.9-x86_64-3.9
          creating build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          copying lib/ovirtsdk4/service.py -> build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          copying lib/ovirtsdk4/services.py -> build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          copying lib/ovirtsdk4/readers.py -> build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          copying lib/ovirtsdk4/version.py -> build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          copying lib/ovirtsdk4/__init__.py -> build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          copying lib/ovirtsdk4/types.py -> build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          copying lib/ovirtsdk4/writers.py -> build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          copying lib/ovirtsdk4/reader.py -> build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          copying lib/ovirtsdk4/http.py -> build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          copying lib/ovirtsdk4/writer.py -> build/lib.macosx-10.9-x86_64-3.9/ovirtsdk4
          running build_ext
          building 'ovirtsdk4.xml' extension
          creating build/temp.macosx-10.9-x86_64-3.9
          creating build/temp.macosx-10.9-x86_64-3.9/ext
          gcc -Wno-unused-result -Wsign-compare -Wunreachable-code -fno-common -dynamic -DNDEBUG -g -fwrapv -O3 -Wall -arch x86_64 -g -I/usr/include/libxml2 -I/Users/mars/workspace/ocs-ci/venv/include -I/Library/Frameworks/Python.framework/Versions/3.9/include/python3.9 -c ext/ov_xml_module.c -o build/temp.macosx-10.9-x86_64-3.9/ext/ov_xml_module.o
          gcc -Wno-unused-result -Wsign-compare -Wunreachable-code -fno-common -dynamic -DNDEBUG -g -fwrapv -O3 -Wall -arch x86_64 -g -I/usr/include/libxml2 -I/Users/mars/workspace/ocs-ci/venv/include -I/Library/Frameworks/Python.framework/Versions/3.9/include/python3.9 -c ext/ov_xml_reader.c -o build/temp.macosx-10.9-x86_64-3.9/ext/ov_xml_reader.o
          ext/ov_xml_reader.c:20:10: fatal error: 'libxml/xmlreader.h' file not found
          #include <libxml/xmlreader.h>
                   ^~~~~~~~~~~~~~~~~~~~
          1 error generated.
          error: command '/usr/bin/gcc' failed with exit code 1
          ----------------------------------------
      ERROR: Command errored out with exit status 1: /Users/mars/workspace/ocs-ci/venv/bin/python3 -u -c 'import io, os, sys, setuptools, tokenize; sys.argv[0] = '"'"'/private/var/folders/hq/km332y3d1079j7l3hxq2v_cr0000gn/T/pip-install-fop84xho/ovirt-engine-sdk-python_399016e0f364412ab2b48e2f53c47070/setup.py'"'"'; __file__='"'"'/private/var/folders/hq/km332y3d1079j7l3hxq2v_cr0000gn/T/pip-install-fop84xho/ovirt-engine-sdk-python_399016e0f364412ab2b48e2f53c47070/setup.py'"'"';f = getattr(tokenize, '"'"'open'"'"', open)(__file__) if os.path.exists(__file__) else io.StringIO('"'"'from setuptools import setup; setup()'"'"');code = f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /private/var/folders/hq/km332y3d1079j7l3hxq2v_cr0000gn/T/pip-record-grrncx03/install-record.txt --single-version-externally-managed --compile --install-headers /Users/mars/workspace/ocs-ci/venv/include/site/python3.9/ovirt-engine-sdk-python Check the logs for full command output.
  
### Diagnostic Steps

- Found current clang version is old

      $ clang --version
      Apple clang version 11.0.3 (clang-1103.0.32.62)
      Target: x86_64-apple-darwin20.4.0
      Thread model: posix
      InstalledDir: /Library/Developer/CommandLineTools/usr/bin

      $ ls /Library/Developer/CommandLineTools/SDKs
      MacOSX.sdk      MacOSX10.14.sdk MacOSX10.15.sdk

### Resolution

- Remove the old SDK, install the latest Xcode CLI.

      $ sudo rm -rf /Library/Developer/CommandLineTools
      $ sudo xcode-select --install

- Verify new clang version installed.

      $ clang --version
      Apple clang version 12.0.5 (clang-1205.0.22.9)
      Target: x86_64-apple-darwin20.4.0
      Thread model: posix
      InstalledDir: /Library/Developer/CommandLineTools/usr/bin

## Issue#2: Some folders have invalid permission

- After issue#1 was resolved, re-ran the same pip command again, failed with new errors:

      Running setup.py develop for ocs-ci
        ERROR: Command errored out with exit status 1:
         command: /Users/lbosh/Projects/PuDong/ocs-ci/venv/bin/python3.9 -c 'import io, os, sys, setuptools, tokenize; sys.argv[0] = '"'"'/Users/lbosh/Projects/PuDong/ocs-ci/setup.py'"'"'; __file__='"'"'/Users/lbosh/Projects/PuDong/ocs-ci/setup.py'"'"';f = getattr(tokenize, '"'"'open'"'"', open)(__file__) if os.path.exists(__file__) else io.StringIO('"'"'from setuptools import setup; setup()'"'"');code = f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' develop --no-deps
             cwd: /Users/lbosh/Projects/PuDong/ocs-ci/
        Complete output (3 lines):
        running develop
        running egg_info
        error: [Errno 13] Permission denied
        ----------------------------------------
      ERROR: Command errored out with exit status 1: /Users/lbosh/Projects/PuDong/ocs-ci/venv/bin/python3.9 -c 'import io, os, sys, setuptools, tokenize; sys.argv[0] = '"'"'/Users/lbosh/Projects/PuDong/ocs-ci/setup.py'"'"'; __file__='"'"'/Users/lbosh/Projects/PuDong/ocs-ci/setup.py'"'"';f = getattr(tokenize, '"'"'open'"'"', open)(__file__) if os.path.exists(__file__) else io.StringIO('"'"'from setuptools import setup; setup()'"'"');code = f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' develop --no-deps Check the logs for full command output.

### Diagnostic Steps
   
- Checking the permissions of current work directory, found that directory `ocs_ci.egg-info` using the different owner. It should be caused by previous sudo actions while trying to fix issue#1.

      $ ll
      total 104
      drwxr-xr-x   5 lbosh  staff   160 May 10 13:27 Docker_files
      -rw-r--r--   1 lbosh  staff  7337 May 10 13:27 Jenkinsfile
      -rw-r--r--   1 lbosh  staff  1057 May 10 13:27 LICENSE
      -rw-r--r--   1 lbosh  staff    79 May 10 13:27 MANIFEST.in
      -rw-r--r--   1 lbosh  staff   508 May 10 13:27 README.md
      drwxr-xr-x   3 lbosh  staff    96 May 10 13:27 bin
      drwxr-xr-x   8 lbosh  staff   256 May 10 13:27 conf
      drwxr-xr-x  26 lbosh  staff   832 May 10 13:27 docs
      drwxr-xr-x   3 lbosh  staff    96 May 10 13:27 external
      drwxr-xr-x  13 lbosh  staff   416 May 10 13:27 ocs_ci
      drwxr-xr-x   9 root   staff   288 May 10 21:12 ocs_ci.egg-info
      -rw-r--r--   1 lbosh  staff  2274 May 10 13:27 pytest.ini
      -rw-r--r--   1 lbosh  staff   111 May 10 13:27 pytest_unittests.ini
      -rw-r--r--   1 lbosh  staff    55 May 10 13:27 registry_requirement.txt
      -rw-r--r--   1 lbosh  staff    52 May 10 13:27 requirements-dev.txt
      -rw-r--r--   1 lbosh  staff   109 May 10 13:27 requirements-docs.txt
      -rw-r--r--   1 lbosh  staff    96 May 10 13:27 requirements.txt
      -rw-r--r--   1 lbosh  staff  2750 May 10 22:26 setup.py
      drwxr-xr-x   3 lbosh  staff    96 May 10 19:34 src
      drwxr-xr-x  12 lbosh  staff   384 May 10 13:27 tests
      -rw-r--r--   1 lbosh  staff  1044 May 10 13:27 tox.ini
      drwxr-xr-x  10 lbosh  staff   320 May 10 21:15 venv

      $ ll ocs_ci.egg-info
      total 112
      -rw-r--r--  1 root  staff    249 May 10 21:12 PKG-INFO
      -rw-r--r--  1 root  staff  31632 May 10 21:12 SOURCES.txt
      -rw-r--r--  1 root  staff      1 May 10 21:12 dependency_links.txt
      -rw-r--r--  1 root  staff    356 May 10 21:12 entry_points.txt
      -rw-r--r--  1 root  staff    919 May 10 21:12 requires.txt
      -rw-r--r--  1 root  staff     13 May 10 21:12 top_level.txt
      -rw-r--r--  1 root  staff      1 May 10 21:12 zip-safe

### Resolution

- Correct the folder permission then solve the problem.

      $ sudo chown -R lbosh:staff ocs_ci.egg-info

## References

<a href="https://ocs-ci.readthedocs.io/en/latest/getting_started.html" target="_blank">ocs-ci Getting Started guideline</a>  
<a href="https://emilwypych.com/2020/11/23/pyenv-problem-macosx_deployment_target11-0" target="_blank">Pyenv problem MACOSX_DEPLOYMENT_TARGET=11.0</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.