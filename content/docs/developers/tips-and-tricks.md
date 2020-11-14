---
title: Tips and tricks for development
weight: 190
---

This sections collects a few tips and tricks for development of {{% gls App %}}
on UBOS that we have found useful.

## Rapid create/test cycle for UBOS packages

It's important to be able to test UBOS packages in development quickly. Here's
the setup we use:

* As recommended, use an Arch Linux development host with the UBOS Arch tools
  installed (see {{% pageref install-arch.md %}})

* Run UBOS in a container started from the development host (see
  {{% pageref "/docs/users/installation/x86_container.md" %}}). It may be
  advantageous to bind your home directory into the container, for example by
  adding ``--bind /home/joe`` to the ``systemd-nspawn`` command that starts
  the container.

* Determine the container's IP address and point a friendly hostname to it.
  In the container:

  ```
  % ip addr
  ```

  On the host, add a line like this:

  ```
  10.0.0.2 testhost
  ```

  to your ``/etc/hosts`` file, assuming the container has IP address
  ``10.0.0.2``. You can also use systemd's built-in local DNS server
  for named containers.

* Install your in-development package in the container. If you have used the
  ``--bind`` option described above, in the container:

  ```
  % sudo pacman -U --noconfirm /home/joe/path/to/your/package.pkg.xz
  ```

* Create a {{% gl Site %}} with the hostname you picked, in the container
  that runs your {{% gl App %}}. In the container:

  ```
  % sudo ubos-admin createsite
  ```

  Specify the test host you picked above, and the name of your package.

* Use a browser on your host to access your {{% gl App %}}, e.g. at ``http://testhost/``.

* When you make changes to your package on the host, update that installed {{% gl App %}}
  in the container by repackaging, and deploying. On the host:

  ```
  % makepkg -f
  ```

  In the container:

  ```
  % sudo ubos-admin update --pkg /home/joe/path/to/your/package.pkg.xz
  ```

  Alternatively, you can use ``ubos-push`` (part of the UBOS tools for Arch) if
  you set up ssh access for the ``shepherd`` account in the container. Then, on the
  host:

  ```
  % ubos-push -h testhost package.pkg.xz
  ```

## Quickly setting up a shepherd account in a UBOS container

If you run UBOS in a container with ``systemd-nspawn``, it may be a good
idea to make your home directory on the development machine available in
the container, so it's easy to move files around. As mentioned above,
that can be accomplished by adding ``--bind /home/joe`` to the
``systemd-nspawn` invocation, assuming your home directory is indeed at
``/home/joe``.

If so, adding a shepherd account to the container becomes really simple,
using your existing public key pair in ``~/.ssh``. In the container,
execute:

```
% sudo ubos-admin setup-shepherd --add-key "$(cat /home/joe/.ssh/id_rsa.pub)"
```

## Debugging a Java/Tomcat App

If you test your Java/Tomcat web {{% gl App %}} by running it in a UBOS container, the
following setup has proven to be useful:

1. In the container, have ``systemd`` start Tomcat with the debug flags on. To do
   so, say:

   ```
   % sudo systemctl edit tomcat8
   ```

   and enter the following content:

   ```
   [Service]
   Environment='CATALINA_OPTS=-Xdebug -Xrunjdwp:transport=dt_socket,address=8888,server=y,suspend=n'
   ```

   Note the quotes.

   Then invoke ``systemctl restart tomcat8``. This will restart Tomcat and your {{% gl App %}},
   but instead of running normally, it will wait for your IDE's debugger to connect on
   port 8888 before proceeding.

2. In the container, open port 8888 in the firewall so the debugger running on the
   host can connect to Tomcat:

   ```
   % sudo vi /etc/iptables/iptables.rules
   ```

   Add the following line where similar lines are:

   ```
   -A OPEN-PORTS -p tcp --dport 8888 -j ACCEPT
   ```

   Restart the firewall: ``systemctl restart iptables``. Note that this setting
   will be overridden as soon as you invoke ``ubos-admin setnetconfig``, but that
   should not be an issue in a debug scenario.

   Alternatively you can create a file named, say ``/etc/ubos/open-ports.d/java-debug``
   with content ``tcp/8080`` and ``ubos-admin setnetconfig`` will always keep
   that port open.

3. On your host, attach your debugger to the container's port 8888. In NetBeans,
   for example, select "Debug / Attach Debugger", select "JDPA", "SocketAttach",
   "dt_socket", enter the IP address of your container and port 8888. For
   good measure, increase the timeout to 60000msec.

## Using up a local Depot

Usually, a UBOS installation pulls software packages from ``http://depot.ubos.net/``.
However, during development and testing, it may be advantageous to run a local
{{% gl Depot %}}  on a build machine.

### Setting up a depot container

To set this up, follow these steps:

1. Go to the ``ubos-buildconfig`` directory.

1. Create an ssh keypair you will use to upload new packages to the depot, e.g.:

   ```
   % mkdir local.ssh
   % ssh-keygen
   ```

   Enter a filename such as ``local.ssh/id_rsa`` and no passphrase.

1. Create a systemd service file that will start the ``depot`` container correctly.
   Depending on your needs, you may use different values. Here is an example that
   uses the host's ``/home/buildmaster/UBOS-STAFF-DEPOT`` as the container's UBOS Staff, so
   you can log in via ssh afterwards. We save it as
   ``/etc/systemd/system/systemd-nspawn@depot.service``:

   ```
   # systemd .service file for starting a UBOS depot container, modify as needed
   # compare with /usr/lib/systemd/system/systemd-nspawn@.service

   [Unit]
   Description=Local UBOS depot
   Documentation=man:systemd-nspawn(1)
   PartOf=machines.target
   Before=machines.target
   After=network.target

   [Service]
   ExecStart=/usr/bin/systemd-nspawn --quiet --keep-unit --boot \
           --link-journal=try-guest --network-veth --machine=%I \
           --bind /home/buildmaster/UBOS-STAFF-DEPOT:/UBOS-STAFF
   KillMode=mixed
   Type=notify
   RestartForceExitStatus=133
   SuccessExitStatus=133
   Slice=machine.slice
   Delegate=yes

   # Enforce a strict device policy, similar to the one nspawn configures
   # when it allocates its own scope unit. Make sure to keep these
   # policies in sync if you change them!
   DevicePolicy=strict
   DeviceAllow=/dev/null rwm
   DeviceAllow=/dev/zero rwm
   DeviceAllow=/dev/full rwm
   DeviceAllow=/dev/random rwm
   DeviceAllow=/dev/urandom rwm
   DeviceAllow=/dev/tty rwm
   DeviceAllow=/dev/net/tun rwm
   DeviceAllow=/dev/pts/ptmx rw
   DeviceAllow=char-pts rw

   [Install]
   WantedBy=machines.target
   ```

1. Make sure the ``/home/buildmaster/UBOS-STAFF-DEPOT`` directory exists (if you chose the
   above configuration) and contains the following information:

   ```
   % mkdir -p /home/buildmaster/UBOS-STAFF-DEPOT/shepherd/ssh
   % ssh-keygen
   ```

   Specify ``/home/buildmaster/UBOS-STAFF-DEPOT/shepherd/ssh/id_rsa`` as the filename,
   and no password. You could reuse the above keypair, too, if you'd like to, but the
   ``id_rsa.pub`` file needs to be in that directory, so UBOS can configure the
   ``shepherd`` account correctly. (The private key doesn't need to be there.)

1. Boot a UBOS container that will become the local {{% gl Depot %}}. This requires that
   you have a UBOS container tarball available that you have downloaded. Let's assume we use
   ``ubos_dev_container-pc_LATEST.tar``:

   ```
   % sudo machinectl import-tar ubos_dev_container-pc_LATEST.tar depot
   % sudo machinectl start depot
   ```

1. Login as shepherd with the private key of the keypair whose public key ended up
   in the ``UBOS-STAFF`` directory:

   ```
   % ssh shepherd@depot -i /home/buildmaster/UBOS-STAFF-DEPOT/shepherd/ssh/id_rsa
   ```

   and install a locally built ``ubos-depot`` package, unless you want the default from
   the default UBOS depot at ``http://depot.ubos.net/``. You may want a locally built version
   if the container you are booting uses an image you built yourself; otherwise version
   inconsistencies between standard UBOS and your build may occur.

   You can copy the package file from the host to the container with ``scp``, or
   ``machinectl copy-to``. Then, in the container:

   ```
   % sudo pacman -U --noconfirm ...path...to.../ubos-repo...pkg.tar.xz
   ```

1. Set up the depot website:

   ```
   % sudo ubos-admin createsite
   ```

   Enter ``ubos-repo`` as the name of the {{% gl App %}}, ``depot`` as the hostname, and
   paste the content of the host's ``local.ssh/id_rsa.pub`` (that you created earlier) into the
   field where it asks for a public upload ssh key. Pick whatever admin account information,
   it does not matter in this case.

1. You should now be able to reach ``http://depot/`` from the host. (Note: by default, the front
   page redirects to ``http://ubos.net/``) If you cannot reach it, check your container setup.
   On the host, as ``root``:

   ```
   # echo 0 > /proc/sys/net/ipv4/ip_forward
   # echo 1 > /proc/sys/net/ipv4/ip_forward
   ```

   and make sure ``/etc/nsswitch.conf`` contains ``mymachines`` in the ``hosts`` section.

### Uploading built packages to the local depot

On your Arch build machine, go back to the ``ubos-buildconfig`` directory. Edit (or create)
the ``local.mk`` file, so it has these lines:

```
UPLOADDEST=ubos-repo@depot:
UPLOADSSHKEY=local.ssh/id_rsa
```

This will instruct make's ``upload`` target to upload packages and images to the host
``depot`` (i.e. the container you created above), using ``ubos-repo`` as the username, and
and the ssh key you created earlier. User ``ubos-repo`` was automatically created when you
installed package ``ubos-repo`` on the ``depot`` container. The upload will be performed
using ``rsync`` over ``ssh``; hence the syntax for ``UPLOADDEST``.
