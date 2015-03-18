---
layout: default
title: How To Migrate
---

### {{ page.title }}
***

#### Step by Step migration from **0.7.10 version to v1.0.x**
***

<div class="alert alert-warning" role="alert">
  <b>Important !</b> Before migrating to the new 1.0 you <b>MUST</b> migrate first to the <b>v0.7.10</b> (see below). It includes a Rake task that prepare the migration to 1.0, so <b>don't miss that step!</b>
</div>

<div class="alert alert-warning" role="alert">Before update the plugin don't forget to backup your database and stop Redmine!</div>

First install new dependencies :

    root# apt-get install libssh2-1 libssh2-1-dev cmake libgpg-error-dev


Be sure to have locales properly set ([#343](https://github.com/jbox-web/redmine_git_hosting/issues/343)) :

    root# apt-get install locales
    root# dpkg-reconfigure locales


It will open a dialog box to select the locales you want to generate then an other one to choose your default locale.

When you are in **v0.7.10**, you first need to launch this Rake task to delete SSH Keys in Gitolite (we'll recreate them later) :

    redmine$ cd REDMINE_ROOT
    redmine$ bundle exec rake redmine_git_hosting:prepare_migration_to_v1 RAILS_ENV=production


Then you can switch to the **1.0.2** version and launch the migration task :

    # Update Redmine Gitolite Hosting
    redmine$ cd REDMINE_ROOT/plugins/redmine_git_hosting
    redmine$ git checkout 1.0.2

    # Update Bootstrap Kit
    redmine$ cd REDMINE_ROOT/plugins/redmine_bootstrap_kit
    redmine$ git checkout 0.2.2

    # Install gems and do the migration
    redmine$ cd REDMINE_ROOT
    redmine$ bundle install --without development test
    redmine$ bundle exec rake redmine_git_hosting:migrate_to_v1 RAILS_ENV=production

Now you must start Redmine to check everything is working.

Go to *Administration -> Redmine Git Hosting -> Config Checks* and check that everything is green.

[Let me know if it works ! (or not)](https://github.com/jbox-web/redmine_git_hosting/issues/339)

***

<div class="alert alert-danger" role="alert"><strong>Warning!&nbsp;During the migration some repositories paths/url may become inconsistent !</strong></div>

The main case is sub-projects's main repository who changes from

```repository/mainproject/subrepo.git``` to

```repository/mainproject/subproject/subrepo.git```.

This is due to [https://github.com/jbox-web/redmine_git_hosting/issues/199](https://github.com/jbox-web/redmine_git_hosting/issues/199).

Unfortunately those repositories cannot be handled by the migration script.

You will have to :

* move them manually in gitolite to their new location (overwriting the empty new .git directory if exists)
* adjust manually the paths in the ```gitolite.conf``` file (check for duplicate entries)
* correct the database ```repositories.url``` and ```repositories.root_url``` fields to point to the correct location
* inform your users about the change

I'm really sorry for the disadvantages.

***

#### Step by Step migration from **0.6 version (or older) to v0.7.10**
***

If you're upgrading from **0.6** version (or older) you should follow these steps :

<div class="alert alert-warning" role="alert">Before update the plugin don't forget to backup your database and stop Redmine!</div>

    # Switch user
    root# su - redmine

    # Remove old plugin
    redmine$ cd REDMINE_ROOT/plugins
    redmine$ rm -rf redmine_git_hosting

    # First git clone Bootstrap Kit
    redmine$ cd REDMINE_ROOT/plugins
    redmine$ git clone https://github.com/jbox-web/redmine_bootstrap_kit.git
    redmine$ cd redmine_bootstrap_kit/
    redmine$ git checkout 0.1.0

    # Then git clone Redmine Git Hosting
    redmine$ cd REDMINE_ROOT/plugins
    redmine$ git clone https://github.com/jbox-web/redmine_git_hosting.git
    redmine$ cd redmine_git_hosting/
    redmine$ git checkout 0.7.10

    # Finally install gems and migrate database
    redmine$ cd REDMINE_ROOT
    redmine$ bundle install --without development test
    redmine$ bundle exec rake redmine:plugins:migrate RAILS_ENV=production NAME=redmine_git_hosting

Now you must start Redmine to check everything is working.

Go to *Administration -> Redmine Git Hosting -> Config Checks* and check that everything is green.

If not, check your configuration.

Then you must update SSH keys by running the following command :

    root# su - redmine
    redmine$ cd REDMINE_ROOT
    redmine$ bundle exec rake redmine_git_hosting:rename_ssh_keys RAILS_ENV=production


<div id="toc">
</div>