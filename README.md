Not maintained
==============

This image is no longer used by the author, and is no longer maintained.


centos-owncloud
===============

The ownCloud docker image on top of CentOS 7.

Pull
----

    docker pull dockingbay/centos-owncloud:latest

Usually you'll want to pull some specific tag instead of `latest`. See
[available centos-owncloud tags](https://registry.hub.docker.com/u/dockingbay/centos-owncloud/tags/manage/).

Run
---

* Prepare a place for ownCloud volumes on the host machine.

        mkdir -p /var/lib/owncloud/{config,data,postgresql}

* Label the directories to make SELinux happy.

        chcon -R -t svirt_sandbox_file_t /var/lib/owncloud/data
        chcon -R -t svirt_sandbox_file_t /var/lib/owncloud/config
        chcon -R -t docker_var_lib_t     /var/lib/owncloud/pgsql

  On a production system you'll want to run this instead, to make the
  labels permanent:

        semanage fcontext -a -t svirt_sandbox_file_t /var/lib/owncloud/data(/.*)?
        semanage fcontext -a -t svirt_sandbox_file_t /var/lib/owncloud/config(/.*)?
        semanage fcontext -a -t docker_var_lib_t /var/lib/owncloud/pgsql(/.*)?
        restorecon -Rv /var/lib/owncloud

* Run ownCloud docker container.

        docker run -p 8000:80 \
            -v /var/lib/owncloud/config:/var/www/html/owncloud/config
            -v /var/lib/owncloud/data:/var/www/html/owncloud/data
            -v /var/lib/owncloud/pgsql:/var/lib/pgsql
            dockingbay/centos-owncloud:latest

  This will bind ownCloud to port 8000. This is ok for testing, but
  for real use you'll want to put a reverse proxy in front of it and
  make the reverse proxy accessible only through HTTPS.

* To increase security for real use, the ownCloud installation wizard
  will not be used (someone else could use the wizard before you
  would). Instead, ownCloud will autoinstall itself. Give it a moment
  to do so. The DB and admin passwords will be autogenerated, and you
  will find them in `/var/lib/owncloud/config/initrc`. Move that file
  out of that directory and put it somewhere safe.

* Point your browser to host (and port) where you're running
  ownCloud. If you're just testing it on your workstation and
  following these steps, the URL is `http://localhost:8000`.

* Log into ownCloud with the admin password that was generated to the
  `initrc` file. You can now change the admin password to something
  else. You're running ownCloud now, congrats!

Notes
-----

* Because the `apps` directory contains apps both from the ownCloud
  RPM and apps that can be downloaded after installation, there's not
  really a straightforward solution to allow persistence of the
  downloaded apps and updating the apps that come with the RPM (= apps
  that come with the ownCloud docker image). Currently the `apps`
  directory is not exported as a volume, which means that all changes
  there are lost between updates of the docker image. If you just use
  the ownCloud apps which come with the RPM, you don't need to worry
  about this.

* The image currently doesn't support serving HTTPS straight from the
  Docker container. Always use a reverse proxy.

* Backup the exported volume directories between image updates.
