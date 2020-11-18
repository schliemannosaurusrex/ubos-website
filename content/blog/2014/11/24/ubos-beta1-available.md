---
title:  UBOS Beta 1 is now available!
date:   2014-11-24 08:00:00
categories: [ front, release, beta ]
---

We're proud to announce that UBOS is now available in a first beta release.

You can run UBOS:

{{% slide-in-img-right src="/images/pc-79x100.png"   alt="[PC]" %}}
{{% slide-in-img-right src="/images/vbox-82x100.png" alt="[Virtual Box]" %}}
{{% slide-in-img-right src="/images/rpi-83x100.png"  alt="[Raspberry Pi]" %}}

 * on standard PCs, as their primary operating system
 * on standard PCs and Macs, in VirtualBox
 * on your Raspberry Pi Model B and Model B+.

{{% pageref "/quickstart.md" %}}


The following apps are available and have been tested on all platforms:

<img src="/images/owncloud-72x72.png" alt="[Owncloud]" style="float: left; width: 36px; margin: 5px 32px 0 5px">

   **Owncloud**: Your Cloud, Your Data, Your Way!<br>
   Install with `sudo ubos-admin createsite`, specify app `owncloud`

<img src="/images/known-72x72.png" alt="[Known]" style="float: left; width: 36px; margin: 5px 32px 0 5px">

   **Known**: Publishing Platform for Everyone<br>
   Install with `sudo ubos-admin createsite`, specify app `idno`

<img src="/images/wordpress-72x72.png" alt="[Wordpress]" style="float: left; width: 36px; margin: 5px 32px 0 5px">

   **Wordpress**: Blog tools, publishing platform, and CMS<br>
   Install with `sudo ubos-admin createsite`, specify app `wordpress`

<img src="/images/mediawiki-72x72.png" alt="[Mediawiki]" style="float: left; width: 36px; margin: 5px 32px 0 5px">

   **Mediawiki**: The wiki that Wikipedia runs on<br>
   Install with `sudo ubos-admin createsite`, specify app `mediawiki`

<img src="/images/selfoss-72x72.png" alt="[Selfoss]" style="float: left; width: 36px; margin: 5px 32px 0 5px">

   **Selfoss**: The new multipurpose rss reader, live stream, mashup, aggregation web application<br>
   Install with `sudo ubos-admin createsite`, specify app `selfoss`

<img src="/images/shaarli-72x72.png" alt="[Shaarli]" style="float: left; width: 36px; margin: 5px 32px 0 5px">

   **Shaarli**: Your Own URL Shortener<br>
   Install with `sudo ubos-admin createsite`, specify app `shaarli`

   and also **Jenkins**, the continuous integration app we use to build UBOS itself on UBOS.<br>
   Install with `sudo ubos-admin createsite`, specify app `jenkins`.


Some apps already have some accessories, which you may or may not want to install:

 * **Wordpress plugins**: ``wordpress-plugin-semantic-linkbacks``, ``wordpress-plugin-social`` and
  ``wordpress-plugin-webmention`` for [Indie Web](http://indiewebcamp.com/) support;

 * **Mediawiki extension**: ``mediawiki-ext-confirmaccount`` to cut down on wiki spam;

 * **Jenkins plugins**: ``jenkins-plugin-git``, ``jenkins-plugin-git-client``, and ``jenkins-plugin-scm-api``
   for Git integration.

The following features should work on all platforms:

 * **Single-command deployment** of web apps, with automatic database provisioniong,
   webserver configuration, etc.

 * **Multiple virtual hosts on the same host**. For example, you can run sites
   ``http://home.example.com/`` and ``http://personal.example.net/`` on the same host.

 * **Multiple apps on the same virtual host**. For example, you can run
   Wordpress at ``http://example.com/blog`` and Mediawiki at ``http://example.com/wiki``

 * **Multiple instances of the same app on the same host**. For example, you can run three
   instances of Wordpress on the same host, such as at ``http://example.com``,
   ``http://example.net/blog`` and ``http://example.net/news``

 * **Single-command undeployment**

 * **Single-command full system upgrade** which backs up all your data, upgrades all
   code from the operating system over middleware to applications, runs whatever
   data migrations might be necessary, and redeploys all your apps.

 * **Single-command backup and restore**.

and a few other things, see the {{% pageref "/docs/" documentation %}} section of this
site.

This is a beta, so expect bugs (~~release notes~~).
We run several production sites on it already, but we don't recommend (yet) that you do.

To try out UBOS, go to {{% pageref "/quickstart.md" %}}. We'd love to
{{% pageref "/community.md" "hear from you" %}} how it works for you.
