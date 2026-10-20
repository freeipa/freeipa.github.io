Web_UI_Integration_Tests
========================

Overview
--------

Purpose of integration tests is to help developers with avoiding
regressions. New tests should be written along with new Web UI
functionality. It means that tests will be written by developers and
therefore the test framework has to be developer friendly. `Selenium
WebDriver <http://docs.seleniumhq.org/projects/webdriver/>`__ browser
automation technology was chosen for this task. It offers wide range of
language binding and doesn't enforce any specific testing framework.
These features allow to implement tests using python and python nose
framework, a framework which is used by server unit/integration tests.

Combination of Web UI design patterns and generic programming language
allowed to abstract clicks and key presses to common Web UI tasks. New
tests can be written very easily and quickly by using these tasks.
Another benefit is that it makes the tests robust. A change in one Web
UI widget doesn't cause many changes all around the tests but it only
requires to fix the code which drives the particular widget.

Infrastructure
--------------

Test infrastructure consists of parts as follows:

FreeIPA server
   Test requires FreeIPA server installed.
Selenium server
   A machine with installed browsers and selenium-server Java
   application. Browsers may also require a webdriver. Webdriver is
   mediator between selenium-server and a browser. Starting Selenium 3.0
   Firefox requires its own webdriver called
   `Geckodriver <https://developer.mozilla.org/en-US/docs/Mozilla/QA/Marionette/WebDriver>`__.
Test runner
   Test runner executes python nose tests. They can be executed either
   in-tree or from a test package.

All parts can run on their own machine or they can share one (not
tested).



FreeIPA Server Configuration
----------------------------

Web UI differs on various FreeIPA server configuration. For purpose of
integration tests, as basic configuration is consider a default FreeIPA
installation with DNS support. The differences from the basic
configuration will be called capabilities and currently they are defined
as follows:

no_dns
   IPA server is installed without DNS: ``--setup-dns`` option is
   missing
no_ca
   Dogtag is not configured. Can be done by defining:
   ``--http-cert-file`` ``--http-pin`` ``--dirsrv-cert-file``
   ``--dirsrv-pin`` More in `CA-less
   documentation <V3/CA-less_install>`__.
has_trusts
   Trust support is enabled: ``ipa-adtrust-install`` was run.



Selenium Server Configuration
-----------------------------

The most important part is to run the selenium server. If your
distribution doesn't have it packaged, download it at
`1 <https://code.google.com/p/selenium/downloads/list>`__.

Run:

.. code:: bash

   java -jar selenium-server-standalone.jar

Firefox
----------------------------------------------------------------------------------------------

Starting Selenium version 3.0 Firefox requires its own
`Geckodriver <https://developer.mozilla.org/en-US/docs/Mozilla/QA/Marionette/WebDriver>`__.
It might be downloaded as tar archive
`here <https://github.com/mozilla/geckodriver/releases>`__. After
downloading it has to be unpacked and then put on $PATH or in directory
which already is in $PATH to be runnable.

Geckodriver automatically stores logs into 'geckodriver.log' file placed
in directory from which tests are run. It is useful to set
'geckodriver_log_path' variable in ui_test.conf file and set there file
where user has write access. Otherwise tests might fail with Permission
denied error because they are unable to write to log file.

Chrome/Chromium
----------------------------------------------------------------------------------------------

#. download repo file and install browser
#. download chromedriver according to platform at
   `2 <http://chromedriver.storage.googleapis.com/index.html>`__
#. put chrome driver in directory included in ``$PATH``



Internet Explorer
----------------------------------------------------------------------------------------------

#. download ``IEDriverServer.exe``. Note: 32 version seems to work
   better even on 64bit Windows.
#. put ``IEDriverServer.exe`` in directory included in ``$PATH``
#. disable Protected Mode in IE (Internet Options/Security)
   `3 <http://code.google.com/p/selenium/wiki/InternetExplorerDriver>`__
#. download FreeIPA server CA cert and put it into root authorities.



X virtual framebuffer
----------------------------------------------------------------------------------------------

If one wants to run test without classical X-system started, one can use
xvbf.

Installation on Fedora:

.. code:: bash

   yum install xorg-x11-server-Xvfb

Run:

.. code:: bash

   export DISPLAY=:99
   /usr/bin/Xvfb $DISPLAY -ac -noreset -screen 0 1400x1200x8 > /dev/null &



Make it quick
----------------------------------------------------------------------------------------------

To avoid doing all the configuration by hand, `attached
script <#Attachments>`__ configures Fedora server for you.

It:

#. prepares repo files for Chrome and Chromium
#. installs Firefox, Chrome, Chromium and xvfb
#. downloads and installs selenium-server
#. downloads and install chrome driver



Test Runner Configuration
-------------------------

Selenium client library for Python is required to run the tests. All
tests are skipped when the library is not installed.

Installation on Fedora:

.. code:: bash

   pip install selenium

\`pip\` is required instead of \`dnf install python-selenium\` until
Fedora has more recent version(3+) of python-selenium.

Test runner requires to be configured. There two ways:

#. configuration file
#. environmental variables

They can be combined but either one is sufficient. Configuration file is
loaded first, then then configuration is overwritten by that in
environmental variables.



Configuration file
----------------------------------------------------------------------------------------------

Is located in ``$HOME/.ipa/ui_test.conf``. It's a
`YAML <http://www.yaml.org/>`__ file, therefore it requires to have YAML
Python library installed. Configuration file is not used if the library
is not installed.

Install yaml on Fedora:

.. code:: bash

   dnf install PyYAML

Example of configuration file:

.. code:: yaml

   # Current FreeIPA server configuration
   # ====================================
   ipa_admin: admin
   ipa_password: Secret123

   ipa_server: DEV.EXAMPLE.COM
   ipa_ip: 10.10.10.10
   ipa_domain: example.com
   ipa_realm: EXAMPLE.COM

   # Uncomment when IPA is installed without CA
   #no_ca: True

   # Uncomment when IPA is installed without DNS server
   #no_dns: True

   # Uncomment when IPA is installed with trust support
   #has_trusts: True

   # Active Directory configuration
   ad_domain: addomain.test
   ad_dc: dc.addomain.test
   ad_admin: Administrator
   ad_password: Secret123
   ad_dc_ip: 10.10.20.10
   trust_secret: Secret123

   # certificates
   host_csr_path: /home/username/.ipa/test.csr
   service_csr_path: /home/username/.ipa/test_srvc.csr
   # Geckodriver setup:
   # =================
   # log file has to be somewhere, where user has rights to write into file
   geckodriver_log_path: /home/me/.ipa/geckodriver.log

   # Web driver setup:
   # =================
   # Selenium server is on localhost or remote machine.
   # Allowed: ['local', 'remote']
   type: remote

   # Browser to test with
   # Allowed: ['chrome', 'chromium', 'firefox', 'ie']
   browser: chrome

   # host needed when type == 'remote'
   # Allowed: hostname or IP address
   host: testrunner.mydomain.test

   # Screenshots
   # ===========
   save_screenshots: True
   # directory where screenshots should be saved
   screenshot_dir: /home/me/tests



Environmental variables
----------------------------------------------------------------------------------------------

Environmental variables are mapped to configuration options according to
following table. Environmental variable names are designed to be similar
with the ones in `server integration tests <V3/Integration_testing>`__.

====================== ====================
Environmental variable Configuration option
====================== ====================
MASTER                 ipa_server
ADMINID                ipa_admin
ADMINPW                ipa_password
DOMAIN                 ipa_domain
IPA_REALM              ipa_realm
IPA_IP                 ipa_ip
IPA_NO_CA              no_ca
IPA_NO_DNS             no_dns
IPA_HAS_TRUSTS         has_trusts
IPA_HOST_CSR_PATH      host_csr_path
IPA_SERVICE_CSR_PATH   service_csr_path
AD_DOMAIN              ad_domain
AD_DC                  ad_dc
AD_ADMIN               ad_admin
AD_PASSWORD            ad_password
AD_DC_IP               ad_dc_ip
TRUST_SECRET           trust_secret
SEL_TYPE               type
SEL_BROWSER            browser
SEL_HOST               host
FF_PROFILE             ff_profile
====================== ====================



Running tests
-------------

Test can be run either in-tree or from test package. Selenium is quite
chatty so it's recommended to run the test with less verbose debug level
like ``--logging-level=INFO``



In tree
----------------------------------------------------------------------------------------------

-  All Web UI tests:

.. code:: bash

   ./make-test --logging-level=INFO ipatests/test_webui/

-  Particular module:

.. code:: bash

   ./make-test --logging-level=INFO ipatests/test_webui/test_module.py

-  Particular test:

.. code:: bash

   ./make-test --logging-level=INFO ipatests/test_webui/test_module.py::class_name::method_name



Test package
----------------------------------------------------------------------------------------------

-  All Web UI tests:

.. code:: bash

   ipa-run-tests --logging-level=INFO test_webui

-  Particular module:

.. code:: bash

   ipa-run-tests --logging-level=INFO test_webui/test_module.py

-  Particular test:

.. code:: bash

   ipa-run-tests --logging-level=INFO test_webui/test_module.py::class_name::method_name



Writing tests
-------------

-  tests are located in-tree in ``ipatests/test_webui`` directory
-  common Web UI tasks and assertions are located in
   ``ipatests.test_webui.ui_driver.UI_driver`` class
-  task usually contains assertions to ensure that all is happening as
   it should

Simple test example:

.. code:: python

   from ipatests.test_webui.ui_driver import UI_driver
   import ipatests.test_webui.data_user as user

   class test_example(UI_driver):

       def test_find(self):
           """
           Test search on user search facet
           """
           # navigate to app and log in
           self.init_app()

           # common assertions are already included in UI_driver methods
           self.add_record(user.ENTITY, user.DATA, navigate=False)
           self.find_record(user.ENTITY, user.DATA)

Attachments
-----------

Selenium server environment preparation script:

.. code:: bash

   #!/bin/bash
   #
   # 1) Install browsers: Firefox, Chrome, Chromium
   # 2) Install Selenium server
   # 3) Install Selenium Chrome driver
   # 4) Install Selenium Gecko driver
   # 5) Install Xvfb

   CHROME_REPO=/etc/yum.repos.d/google-chrome.repo
   CHROMIUM_REPO=/etc/yum.repos.d/fedora-chromium-stable.repo
   CHROME_DRIVER_PATH=/usr/bin/chromedriver
   GECKO_DRIVER_ARCH=geckodriver.tar.gz
   GECKO_DRIVER_PATH=/usr/bin/geckodriver
   SELENIUM_TMP_DIR=~/selenium
   SELENIUM_SERVER_DIR=/opt/selenium

   # this must match exact filename in http://selenium-release.storage.googleapis.com/$SELENIUM_VERSION
   URL="https://selenium-release.storage.googleapis.com"
   SRC="$(curl $URL)"
   MAIN_VERSION=$(echo "$SRC" | grep -oP '[\.0-9]*(?=/selenium-server-standalone)' | awk '{max=$1;if($1>max) {max=$1};} END {print max}')
   SUBVERSION=$(echo "$SRC" | grep -oP "(?<=$MAIN_VERSION/selenium-server-standalone-$MAIN_VERSION\.)[0-9]" | awk '{max=$1;if($1>max) {max=$1};} END {print max}')
   SELENIUM_VERSION=$MAIN_VERSION.$SUBVERSION
   SELENIUM_JAR=selenium-server-standalone-${SELENIUM_VERSION}.jar

   # Chrome driver version
   CHROME_URL="http://chromedriver.storage.googleapis.com"
   CHROME_SRC="$(curl $CHROME_URL)"
   CHROME_DRIVER_ARCH=chromedriver_linux64.zip
   CHROME_MAINVERSION=$(echo "$CHROME_SRC" | grep -oP "[\.0-9]*(?=/$CHROME_DRIVER_ARCH)" | grep -oP "[0-9]*(?=\.)" | sort -h | tail -n1)
   CHROME_SUBVERSION=$(echo "$CHROME_SRC" | grep -oP "[\.0-9]*(?=/$CHROME_DRIVER_ARCH)" | grep -oP "(?<=$CHROME_MAINVERSION\.)[0-9]*" | sort -h | tail -n1)
   CHROME_DRIVER_VERSION=$CHROME_MAINVERSION.$CHROME_SUBVERSION

   # Gecko driver version
   GECKO_VERSION=`curl https://github.com/mozilla/geckodriver/releases/latest 2>/dev/null | egrep -o 'href="[^"]*"'`
   GECKO_VERSION=`echo "$GECKO_VERSION" | sed 's/href="//' | sed 's/"$//' | awk -F"/" '{print $NF}'`
   GECKO_URL="https://github.com/mozilla/geckodriver/releases/download/$GECKO_VERSION/geckodriver-$GECKO_VERSION-linux64.tar.gz"

   # Install dependencies for this script
   sudo yum install -y wget unzip

   # Chromium repo
   if [ ! -f $CHROMIUM_REPO ]
   then
       echo "Adding Chromium repo"
       sudo wget -O $CHROMIUM_REPO http://repos.fedorapeople.org/repos/spot/chromium-stable/fedora-chromium-stable.repo
   fi

   # Chrome repo
   if [ ! -f $CHROME_REPO ]
   then
       TMP=`mktemp`
       echo "Adding Chrome repo"
       cat > $TMP <<EOL
   [google-chrome]
   name=google-chrome - 64-bit
   baseurl=http://dl.google.com/linux/chrome/rpm/stable/x86_64
   enabled=1
   gpgcheck=1
   gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
   EOL
       sudo cp $TMP $CHROME_REPO
       rm $TMP
   fi

   # install browsers and virtual display
   sudo yum install xorg-x11-server-Xvfb firefox chromium google-chrome-stable -y


   mkdir -p $SELENIUM_TMP_DIR
   sudo mkdir -p $SELENIUM_SERVER_DIR

   # Download Chrome driver and selenium server
   # Chrome driver page: http://code.google.com/p/chromedriver/downloads/list
   # http://code.google.com/p/selenium/downloads/list
   # NOTE: chrome driver 2.0 works only with Chrome/Chromium version >= 27
   # NOTE: starting Selenium version 3.0 firefox requires its own driver called Geckodriver
   pushd $SELENIUM_TMP_DIR > /dev/null
       if [ ! -f $SELENIUM_JAR ]
       then
           echo "Downloading Selenium server"
           wget "$URL"/$MAIN_VERSION/$SELENIUM_JAR
       fi

       if [ ! -f $SELENIUM_SERVER_DIR/selenium-server.jar ]
       then
           echo "Installing Selenium server"
           sudo cp $SELENIUM_JAR $SELENIUM_SERVER_DIR/selenium-server.jar
       fi

       if [ ! -f $CHROME_DRIVER_ARCH ]
       then
           echo "Downloading Chromedriver"
           wget $CHROME_URL/$CHROME_DRIVER_VERSION/$CHROME_DRIVER_ARCH
           unzip $CHROME_DRIVER_ARCH
       fi

       if [ ! -f $CHROME_DRIVER_PATH ]
       then
           echo "Installing Chrome driver"
           sudo cp chromedriver $CHROME_DRIVER_PATH
           sudo chmod a+x $CHROME_DRIVER_PATH
       fi

       if [ ! -f $GECKO_DRIVER_ARCH ]
       then
           echo "Downloading Geckodriver"
           wget -q -O $GECKO_DRIVER_ARCH $GECKO_URL
           tar -xvzf $GECKO_DRIVER_ARCH
       fi

       if [ ! -f $GECKO_DRIVER_PATH ]
       then
           echo "Installing Gecko driver"
           sudo cp geckodriver $GECKO_DRIVER_PATH
           sudo chmod a+x $GECKO_DRIVER_PATH
       fi
   popd > /dev/null