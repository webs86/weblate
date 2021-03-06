.. _deployments:

Weblate deployments
===================

Weblate comes with support for deployment using several technologies. This
section brings overview of them.

.. _appliance:

SUSE Studio appliance
---------------------

Weblate appliance provides preconfigured Weblate running with MySQL database as
backend and Apache as web server. It is provided in many formats suitable for
any form of virtualization, cloud or hardware installation.

It comes with standard set of passwords you will want to change:

======== ======== ======= ==================================================
Username Password Scope   Description
======== ======== ======= ==================================================
root     linux    System  Administrator account, use for local or SSH login
root              MySQL   MySQL administrator
weblate  weblate  MySQL   Account in MySQL database for storing Weblate data
admin    admin    Weblate Weblate/Django admin user
======== ======== ======= ==================================================

The appliance is built using SUSE Studio and is based on openSUSE 12.3.

You should also adjust some settings to match your environment, namely:

* :ref:`production-debug`
* :ref:`production-site`
* :ref:`production-email`

.. _docker:

Weblate and Docker
------------------

With dockerized weblate deployment you can get your personal weblate instance
up an running in seconds. All of Weblate's dependencies are already included.
PostgreSQL is configured as default database.

Deployment
++++++++++

Following examples assume you have working Docker environment, with
docker-compose installed. Please check Docker documentation for instructions on
this.

1. Clone weblate-docker repo:

.. code-block:: sh

    git clone https://github.com/WeblateOrg/docker.git weblate-docker
    cd weblate-docker

2. Create a :file:`docker-compose.override.yml` file with your settings.
   See :ref:`docker-environment` full list of environment vars

.. code-block:: yaml

    version: '2'
    services:
      weblate:
        ports:
          - "80:8000"
        environment:
          - WEBLATE_EMAIL_HOST=email.com
          - WEBLATE_EMAIL_HOST_USER=user
          - WEBLATE_EMAIL_HOST_PASSWORD=pass
          - WEBLATE_SECRET_KEY=something more secret
          - WEBLATE_ALLOWED_HOSTS=your hosts

4. Build Weblate containers:

.. code-block:: sh

    docker-compose build

5. Create Weblate database:

.. code-block:: sh

    docker-compose run --rm weblate migrate

6. Prepare static files:

.. code-block:: sh

    docker-compose run --rm weblate collectstatic --noinput

7. Create admin user (this command will print your random password for admin, you can change it later, see :djadmin:`createadmin`):

.. code-block:: sh

    docker-compose run --rm weblate createadmin

.. note::

    Remember to write down the admin password, you will need it to manage Weblate later.

8. Start Weblate containers:

.. code-block:: sh

    docker-compose up

Enjoy your Weblate deployment, it's accessible on port 8000 of the container.

.. seealso:: :ref:`invoke-manage`

Maintenance tasks
+++++++++++++++++

There are some cron jobs to run. You should set :envvar:`WEBLATE_OFFLOAD_INDEXING` to ``1`` when these are setup

.. code-block:: crontab

    */5 * * * * cd /usr/share/weblate/; docker-compose run --rm weblate update_index
    @daily cd /usr/share/weblate/; docker-compose run --rm weblate cleanuptrans
    @hourly cd /usr/share/weblate-docker/; docker-compose run --rm weblate commit_pending --all --age=96

.. _docker-environment:

Docker environment variables
++++++++++++++++++++++++++++

Many of Weblate :ref:`config` can be set in Docker container using environment variables:

Generic settings
~~~~~~~~~~~~~~~~

.. envvar:: WEBLATE_DEBUG

    Configures Django debug mode, see :ref:`production-debug`.

.. envvar:: WEBLATE_LOGLEVEL

    Configures verbosity of logging.


.. envvar:: WEBLATE_SITE_TITLE

    Configures site title, see :ref:`production-site`.

.. envvar:: WEBLATE_ADMIN_NAME
.. envvar:: WEBLATE_ADMIN_EMAIL

    Configures site admins name and email, see :ref:`production-admins`.

.. envvar:: WEBLATE_SERVER_EMAIL
.. envvar:: WEBLATE_DEFAULT_FROM_EMAIL

    Configures address for outgoing mails, see :ref:`production-email`.

.. envvar:: WEBLATE_ALLOWED_HOSTS

    Configures allowed HTTP hostnames, see :ref:`production-hosts`

.. envvar:: WEBLATE_SECRET_KEY

    Configures secret for cookies signing, see :ref:`production-secret`.

.. envvar:: WEBLATE_REGISTRATION_OPEN

    Configures whether registrations are open, see :std:setting:`REGISTRATION_OPEN`.

.. envvar:: WEBLATE_TIME_ZONE

    Configures used time zone.

.. envvar:: WEBLATE_OFFLOAD_INDEXING

    Configures offloaded indexing, see :ref:`production-indexing`.

.. envvar:: WEBLATE_ENABLE_HTTPS

    Configures when use https in email and API links, see :ref:`production-site`.

.. envvar:: WEBLATE_REQUIRE_LOGIN

    Configures login required for whole Weblate using :setting:`LOGIN_REQUIRED_URLS`.

.. envvar:: WEBLATE_GOOGLE_ANALYTICS_ID

    Configures ID for Google Analytics, see :setting:`GOOGLE_ANALYTICS_ID`.


Machine translation settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. envvar:: WEBLATE_MT_GOOGLE_KEY

    Enables Google machine translation and sets :setting:`MT_GOOGLE_KEY`


Authentication settings
~~~~~~~~~~~~~~~~~~~~~~~

.. envvar:: WEBLATE_SOCIAL_AUTH_GITHUB_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_GITHUB_SECRET

    Enables :ref:`google_auth`.

.. envvar:: WEBLATE_SOCIAL_AUTH_BITBUCKET_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_BITBUCKET_SECRET

    Enables :ref:`bitbucket_auth`.

.. envvar:: WEBLATE_SOCIAL_AUTH_FACEBOOK_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_FACEBOOK_SECRET

    Enables :ref:`facebook_auth`.

.. envvar:: WEBLATE_SOCIAL_AUTH_GOOGLE_OAUTH2_KEY
.. envvar:: WEBLATE_SOCIAL_AUTH_GOOGLE_OAUTH2_SECRET

    Enables :ref:`google_auth`.

PostgreSQL databse setup
~~~~~~~~~~~~~~~~~~~~~~~~

The databse is created by :file:`docker-compose.yml`, so this settings affects
both Weblate and PostgreSQL containers.

.. envvar:: POSTGRES_PASSWORD

    PostgreSQL password.

.. envvar:: POSTGRES_USER

    PostgreSQL username.

.. envvar:: POSTGRES_DATABASE

    PostgreSQL databse name.

Email server setup
~~~~~~~~~~~~~~~~~~

To make outgoing email work, you need to provide mail server.

.. envvar:: WEBLATE_EMAIL_HOST

    Mail server, the server has to listen on port 587 and understand TLS.

.. envvar:: WEBLATE_EMAIL_USER

    Email authentication user, do NOT use quotes here.

.. envvar:: WEBLATE_EMAIL_PASSWORD

    Email authentication password, do NOT use quotes here.



Select your machine - local or cloud providers
++++++++++++++++++++++++++++++++++++++++++++++

With docker-machine you can create your Weblate deployment either on your local
machine or on any large number of cloud-based deployments on e.g. Amazon AWS,
Digitalocean and many more providers.

.. _openshift:

Weblate on OpenShift
--------------------

This repository contains a configuration for the OpenShift platform as a
service product, which facilitates easy installation of Weblate on OpenShift
Online (https://www.openshift.com/), OpenShift Enterprise
(https://enterprise.openshift.com/) and OpenShift Origin
(https://www.openshift.org/).

Prerequisites
+++++++++++++

1. OpenShift Account

   You need an account for OpenShift Online (https://www.openshift.com/) or
   another OpenShift installation you have access to.

   You can register a free account on OpenShift Online, which allows you to
   host up to 3 applications free of charge.

2. OpenShift Client Tools

   In order to follow the examples given in this documentation you need to have
   the OpenShift Client Tools (RHC) installed:
   https://developers.openshift.com/en/managing-client-tools.html

   While there are other possibilities to create and configure OpenShift
   applications, this documentation is based on the OpenShift Client Tools
   (RHC) because they provide a consistent interface for all described
   operations.

Installation
++++++++++++

You can install Weblate on OpenShift directly from Weblate's github repository
with the following command:

.. code-block:: sh

    rhc -aweblate app create -t python-2.7 --from-code https://github.com/nijel/weblate.git --no-git

The ``-a`` option defines the name of your weblate installation, ``weblate`` in
this instance. You are free to specify a different name.

Optionally you can specify tag identifier right of the ``#`` sign to identify
the version of Weblate to install (for example specify
``https://github.com/nijel/weblate.git#weblate-2.0`` to install Weblate 2.0).
For a list of available versions see here:
https://github.com/nijel/weblate/tags. Please note that only version 2.0 and
newer can be installed on OpenShift, as older versions don't include the
necessary configuration files. The ``--no-git`` option skips the creation of a
local git repository.

You can also specify which database you want to use:

.. code-block:: sh

    # For MySQL
    rhc -aweblate app create -t python-2.7 -t mysql-5.5 --from-code https://github.com/nijel/weblate.git --no-git

    # For PostgreSQL
    rhc -aweblate app create -t python-2.7 -t postgresql-9.2 --from-code https://github.com/nijel/weblate.git --no-git

Default Configuration
+++++++++++++++++++++

After installation on OpenShift Weblate is ready to use and preconfigured as follows:

* SQLite embedded database (DATABASES)
* Random admin password
* Random Django secret key (SECRET_KEY)
* Indexing offloading if the cron cartridge is installed (:setting:`OFFLOAD_INDEXING`)
* Committing of pending changes if the cron cartridge is installed (:djadmin:`commit_pending`)
* Weblate machine translations for suggestions bases on previous translations (:setting:`MACHINE_TRANSLATION_SERVICES`)
* Weblate directories (STATIC_ROOT, :setting:`DATA_DIR`, :setting:`TTF_PATH`, Avatar cache) set according to OpenShift requirements/conventions
* Django site name and ALLOWED_HOSTS set to DNS name of your OpenShift application
* Email sender addresses set to no-reply@<OPENSHIFT_CLOUD_DOMAIN>, where <OPENSHIFT_CLOUD_DOMAIN> is the domain OpenShift runs under. In case of OpenShift Online it's rhcloud.com.

.. seealso::

   :ref:`customize_config`

Retrieve Admin Password
~~~~~~~~~~~~~~~~~~~~~~~

You can retrieve the generated admin password with the following command:

.. code-block:: sh

    rhc -aweblate ssh credentials

Indexing Offloading
~~~~~~~~~~~~~~~~~~~

To enable the preconfigured indexing offloading you need to add the cron cartridge to your application and restart it:

.. code-block:: sh

    rhc -aweblate add-cartridge cron
    rhc -aweblate app stop
    rhc -aweblate app start

The fulltext search index will then be updated every 5 minutes.
Restarting with ``rhc restart`` instead will not enable indexing offloading in Weblate.
You can verify that indexing offloading is indeed enabled by visiting the URL ``/admin/performance/`` of your application.

Pending Changes
~~~~~~~~~~~~~~~

Weblate's OpenShift configuration contains a cron job which periodically commits pending changes older than a certain age (24h by default).
To enable the cron job you need to add the cron cartridge and restart Weblate as described in the previous section. You can change the age
parameter by setting the environment variable WEBLATE_PENDING_AGE to the desired number of hours, e.g.:

.. code-block:: sh

    rhc -aweblate env set WEBLATE_PENDING_AGE=48

.. _customize_config:

Customize Weblate Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can customize the configuration of your Weblate installation on OpenShift
through environment variables.  Override any of Weblate's setting documented
under :ref:`config` using ``rhc env set`` by prepending the settings name with
``WEBLATE_``. The variable content is put verbatim to the configuration file,
so it is parsed as Python string, after replacing environment variables in it
(eg. ``$PATH``). To put literal ``$`` you need to escape it as ``$$``.

For example override the ``ADMINS`` setting like this:

.. code-block:: sh

    rhc -aweblate env set WEBLATE_ADMINS='(("John Doe", "jdoe@example.org"),)'

To change site title, do not forget to include additional quotes:

.. code-block:: sh

    rhc -aweblate env set WEBLATE_SITE_TITLE='"Custom Title"'

New settings will only take effect after restarting Weblate:

.. code-block:: sh

    rhc -aweblate app stop
    rhc -aweblate app start

Restarting using ``rhc -aweblate app restart`` does not work. For security reasons only constant expressions are allowed as values.
With the exception of environment variables which can be referenced using ``${ENV_VAR}``. For example:

.. code-block:: sh

    rhc -aweblate env set WEBLATE_PRE_COMMIT_SCRIPTS='("${OPENSHIFT_DATA_DIR}/examples/hook-generate-mo",)'

You can check the effective settings Weblate is using by running:

.. code-block:: sh

    rhc -aweblate ssh settings

This will also print syntax errors in your expressions.
To reset a setting to its preconfigured value just delete the corresponding environment variable:

.. code-block:: sh

   rhc -aweblate env unset WEBLATE_ADMINS

.. seealso::

   :ref:`config`

Updating
++++++++

It is recommended that you try updates on a clone of your Weblate installation before running the actual update.
To create such a clone run:

.. code-block:: sh

    rhc -aweblate2 app create --from-app weblate

Visit the newly given URL with a browser and wait for the install/update page to disappear.

You can update your Weblate installation on OpenShift directly from Weblate's github repository by executing:

.. code-block:: sh

    rhc -aweblate2 ssh update https://github.com/nijel/weblate.git

The identifier right of the ``#`` sign identifies the version of Weblate to install.
For a list of available versions see here: https://github.com/nijel/weblate/tags.
Please note that the update process will not work if you modified the git repository of you weblate installation.
You can force an update by specifying the ``--force`` option to the update script. However any changes you made to the
git repository of your installation will be discarded:

.. code-block:: sh

   rhc -aweblate2 ssh update --force https://github.com/nijel/weblate.git

The ``--force`` option is also needed when downgrading to an older version.
Please note that only version 2.0 and newer can be installed on OpenShift,
as older versions don't include the necessary configuration files.

The update script takes care of the following update steps as described under :ref:`generic-upgrade-instructions`.

* Install any new requirements
* manage.py migrate
* manage.py setupgroups --move
* manage.py setuplang
* manage.py rebuild_index --all
* manage.py collectstatic --noinput


Bitnami Weblate stack
---------------------

Bitnami provides Weblate stack for many platforms at
<https://bitnami.com/stack/weblate>. The setup will be adjusted during
installation, see <https://bitnami.com/stack/weblate/README.txt> for more
documentation.
