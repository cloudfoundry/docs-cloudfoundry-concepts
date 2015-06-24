---
title: Stacks
---

A stack is a prebuilt root filesystem (rootfs) which works in tandem with a buildpack and is used to support running applications.

<p class="note"><strong>Note</strong>: You must ensure compatibility for buildpacks on the stacks they are running against.</p>

##Building Stacks

DEAs can support multiple stacks. The scripts for building the available Cloud Foundry stacks (and later for building other stacks) reside in the [stacks](http://github.com/cloudfoundry/stacks) project.

##Available Stacks

###The cflinuxfs2 Stack

The cflinuxfs2 stack is derived from Ubuntu Trusty 14.04.

##CLI Commands

Use the `cf stacks` command to list all the stacks available in a deployment.

```
$ cf stacks
Getting stacks in org My-Org / space development as developer@sample-app.com...
OK

name         description
cflinuxfs2   Cloud Foundry Linux-based filesystem
```

Use the `cf push APPNAME -s STACKNAME` to change your stack and restage your application.

```
$ cf push testappmx1 -s cflinuxfs2
Using stack cflinuxfs2...
OK
Creating app testappmx1 in org pivotal-pubtools / space test-acceptance as developer@sample-app.com...
OK

Creating route testappmx1.cfapps.io...
OK

Binding testappmx1.cfapps.io to testappmx1...
OK

Uploading testappmx1...
Uploading app files from: /Users/pivotal/workspace/app_files
Uploading 13.4M, 544 files
Done uploading
OK

Starting app testappmx1 in org pivotal-application-tools / space test-acceptance as developer@sample-app.com...
-----> Downloaded app package (21M)
-------> Buildpack version 1.3.0
-----> Compiling Ruby/Rack
-----> Using Ruby version: ruby-2.0.0
-----> Installing dependencies using 1.7.12
      Running: bundle install --without development:test --path vendor/bundle --binstubs vendor/bundle/bin -j4 --deployment
      Fetching gem metadata from http://rubygems.org/..........
      Installing rack-rewrite 1.5.0
      Installing rack 1.5.2
      Using bundler 1.7.12
      Installing ref 1.0.5
      Installing libv8 3.16.14.3
      Installing therubyracer 0.12.1
      Your bundle is complete!
      Gems in the groups development and test were not installed.
      It was installed into ./vendor/bundle
      Bundle completed (16.63s)
      Cleaning up the bundler cache.
###### WARNING:
      No Procfile detected, using the default web server (webrick)
       https://devcenter.heroku.com/articles/ruby-default-web-server

-----> Uploading droplet (45M)

1 of 1 instances running

App started


OK

App testappmx1 was started using this command `bundle exec rackup config.ru -p $PORT`

Showing health and status for app testappmx1 in org pivotal-application-tools / space test-acceptance as developer@sample-app.com...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: testappmx1.cfapps.io
last uploaded: Wed Apr 8 23:40:57 UTC 2015

    state    since                    cpu    memory        disk
#0  running  2015-04-08 04:41:54 PM  0.0%  57.3M of 1G  128.8M of 1G
```

For API information, review the Stacks section of the [Cloud Foundry API Documentation](http://apidocs.cloudfoundry.org).
