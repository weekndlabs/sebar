Sebar
==========

**sebar.deploy** are Ansible roles to easily manage the deployment process for scripting applications such as Nodejs, Golang, PHP, Python and Ruby etc.

- [Requirements](#requirements)
- [Installation](#installation)
- [Update](#update)
- [Features](#features)
- [Main workflow](#main-workflow)
- [Role Variables](#role-variables)
- [Deploying](#deploying)
- [Rolling back](#rolling-back)
- [Multistage environment (devel, preprod, prod, etc.)](#multistage-environment-devel-preprod-prod-etc)
- [Hooks: Custom tasks](#hooks-custom-tasks)
- [Variables in custom tasks](#variables-in-custom-tasks)
- [Pruning old releases](#pruning-old-releases)
- [Example Playbook](#example-playbook)
- [Sample projects](#sample-projects)


Requirements
------------

In order to deploy your apps with sebar, you will need:

* Ansible in your deployer machine
* `rsync` on the target machine if you are using either the `rsync` or `git` deployment strategy or if you are using `sebar_current_via = rsync`

Installation
------------

sebar is an Ansible role distributed globally using [Ansible Galaxy](https://galaxy.ansible.com/). In order to install sebar role you can use the following command.

```
$ ansible-galaxy install sebar.deploy
```

Update
------

If you want to update the role, you need to pass **--force** parameter when installing. Please, check the following command:

```
$ ansible-galaxy install --force sebar.deploy
```

Features
--------

* Customize your deployment with hooks before and after critical steps
* Save disk space keeping a maximum fixed releases in your hosts
* Choose between SCP, RSYNC, GIT, SVN, HG, HTTP Download or S3 GET deployment strategies (optional unarchive step included)

Main workflow
-------------

sebar deploys applications following the Capistrano flow.

* Setup phase: Creates the folder structure to hold your releases
* Code update phase: Puts the new release into your hosts
* Symlink phase: After deploying the new release into your hosts, this step changes the `current` softlink to new the release
* Cleanup phase: Removes any old version based in the `sebar_keep_releases` parameter (see "Role Variables")

![sebar Flow](https://raw.githubusercontent.com/sebar/deploy/master/docs/sebar-flow.png)

Role Variables
--------------

```yaml
vars:
  sebar_deploy_from: "{{ playbook_dir }}" # Where my local project is (relative or absolute path)
  sebar_deploy_to: "/var/www/my-app" # Base path to deploy to.
  sebar_version_dir: "releases" # Releases folder name
  sebar_current_dir: "current" # Softlink name. You should rarely changed it.
  sebar_current_via: "symlink" # Deployment strategy who code should be deployed to current path. Options are symlink or rsync
  sebar_keep_releases: 0 # Releases to keep after a new deployment. See "Pruning old releases".

  # Arrays of directories and files to be shared.
  # The following arrays of directories and files will be symlinked to the current release directory after the 'update-code' step and its callbacks
  # Notes:
  # * Paths are relative to the /shared directory (no starting /)
  # * If your items are in a subdirectory, write the entire path to each shared directory
  #
  # Example:
  # sebar_shared_paths:
  #   - path/to/first-dir
  #   - path/next-dir
  # sebar_shared_files:
  #   - my-file.txt
  #   - path/to/file.txt
  sebar_shared_paths: []
  sebar_shared_files: []


  # Shared paths and basedir shared files creation.
  # By default the shared paths directories and base directories for shared files are created automatically if not exists. But in some scenarios those paths could be symlinks to another directories in the filesystem, and the deployment process would fails. With these variables you can disable the involved tasks. If you have two or three shared paths, and don't need creation only for some of them, you always could disable the automatic creation and add a custom task in a hook.
  sebar_ensure_shared_paths_exist: yes
  sebar_ensure_basedirs_shared_files_exist: yes

  sebar_deploy_via: "rsync" # Method used to deliver the code to the server. Options are copy, rsync, git, svn, s3 or download. Copy, download and s3 have an optional step to unarchive the downloaded file which can be used by adding _unarchive. You can check all the options inside tasks/update-code folder!
  sebar_allow_anonymous_stats: yes

  # Variables used in the rsync deployment strategy
  sebar_rsync_extra_params: "" # Extra parameters to use when deploying with rsync in a single string. Although Ansible allows an array this can cause problems if we try to add multiple --include args as it was reported in https://github.com/sebar/deploy/commit/e98942dc969d4e620313f00f003a7ea2eab67e86
  sebar_rsync_set_remote_user: yes # See [ansible synchronize module](http://docs.ansible.com/ansible/synchronize_module.html). Options are yes, no.
  sebar_rsync_path: "" # See [ansible synchronize module](http://docs.ansible.com/ansible/synchronize_module.html). By default is "sudo rsync", it can be overwriten with (example): "sudo -u user rsync".
  sebar_rsync_use_ssh_args: no # See [ansible synchronize module](http://docs.ansible.com/ansible/synchronize_module.html). If set yes, use the ssh_args specified in ansible.cfg.

  # Variables used in the Git deployment strategy
  sebar_git_repo: git@github.com:USERNAME/REPO.git # Location of the git repository
  sebar_git_branch: master # What version of the repository to check out. This can be the full 40-character SHA-1 hash, the literal string HEAD, a branch name, or a tag name
  sebar_git_repo_tree: "" # If specified the subtree of the repository to deploy
  sebar_git_identity_key_path: "" # If specified this file is copied over and used as the identity key for the git commands, path is relative to the playbook in which it is used
  sebar_git_identity_key_remote_path: "" # If specified this file on the remote server is used as the identity key for the git commands, remote path is absolute
  # Optional variables, omitted by default
  sebar_git_refspec: ADDITIONAL_GIT_REFSPEC # Additional refspec to be used by the 'git' module. Uses the same syntax as the 'git fetch' command.
  sebar_git_ssh_opts: "-o StrictHostKeyChecking=no" # Additional ssh options to be used in Git
  sebar_git_depth: 1 # Additional history truncated to the specified number or revisions

  # Variables used in the SVN deployment strategy
  # Please note there was a bug in the subversion module in Ansible 1.8.x series (https://github.com/ansible/ansible-modules-core/issues/370) so it is only supported from Ansible 1.9
  sebar_svn_repo: "https://svn.company.com/project" # Location of the svn repository
  sebar_svn_branch: "trunk" # What branch from the repository to check out.
  sebar_svn_revision: "HEAD" # What revision from the repository to check out.
  sebar_svn_username: "user" # SVN authentication username
  sebar_svn_password: "Pa$$word" # SVN authentication password
  sebar_svn_environment: {} # Dict with environment variables for svn tasks (https://docs.ansible.com/ansible/playbooks_environment.html)

  # Variables used in the HG deployment strategy
  sebar_hg_repo: "https://USERNAME@bitbucket.org/USERNAME/REPO" # Location of the hg repo
  sebar_hg_branch: "default" # Any branch identifier that works with hg -r, so named branch, bookmark, commit hash...

  # Variables used in the download deployment strategy
  sebar_get_url: https://github.com/someproject/somearchive.tar.gz
  sebar_download_force_basic_auth: false # no default as this is only supported from Ansible 2.0
  sebar_download_headers: "" # no default as this is only supported from Ansible 2.0

  # Variables used in the S3 deployment strategy
  sebar_s3_bucket: s3bucket
  sebar_s3_object: s3object.tgz # Add the _unarchive suffix to the sebar_deploy_via if your object is a package (ie: s3_unarchive)
  sebar_s3_region: eu-west-1
  sebar_s3_rgw: false # must be Ansible >= 2.2. use Ceph RGW (when set true, ignore sebar_s3_region)
  sebar_s3_url: http://rgw.example.com # when use Ceph RGW, set url
  # Optional variables, omitted by default
  sebar_s3_aws_access_key: YOUR_AWS_ACCESS_KEY
  sebar_s3_aws_secret_key: YOUR_AWS_SECRET_KEY
  sebar_s3_ignore_nonexistent_bucket: false

  # Variables used in the GCS deployment strategy
  sebar_gcs_bucket: gcsbucket
  sebar_gcs_object: gcsobject.tgz # Add the _unarchive suffix to the sebar_deploy_via if your object is a package (ie: s3_unarchive)
  sebar_gcs_region: eu-west-1 # https://cloud.google.com/storage/docs/bucket-locations
  # Optional variables, omitted by default
  sebar_gcs_access_key: YOUR_GCS_ACCESS_KEY # navigate to Cloud console > Storage > Settings > Interoperability
  sebar_gcs_secret_key: YOUR_GCS_SECRET_KEY

  # Hooks: custom tasks if you need them
  sebar_before_setup_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-before-setup-tasks.yml"
  sebar_after_setup_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-after-setup-tasks.yml"
  sebar_before_update_code_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-before-update-code-tasks.yml"
  sebar_after_update_code_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-after-update-code-tasks.yml"
  sebar_before_symlink_shared_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-before-symlink-shared-tasks.yml"
  sebar_after_symlink_shared_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-after-symlink-shared-tasks.yml"
  sebar_before_symlink_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-before-symlink-tasks.yml"
  sebar_after_symlink_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-after-symlink-tasks.yml"
  sebar_before_cleanup_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-before-cleanup-tasks.yml"
  sebar_after_cleanup_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-after-cleanup-tasks.yml"
```

`{{ playbook_dir }}` is an Ansible variable that holds the path to the current playbook.

Deploying
---------

In order to deploy with sebar, you need to perform some steps:

* Create a new `hosts` file. Check [ansible inventory documentation](http://docs.ansible.com/intro_inventory.html) if you need help. This file will identify all the hosts where to deploy to. For multistage environments check [Multistage environments](#multistage-environment-devel-preprod-prod-etc).
* Create a new playbook for deploying your app, for example, `deploy.yml`
* Set up role variables (see [Role Variables](#role-variables))
* Include the `sebar.deploy` role as part of a play
* Run the deployment playbook

```ansible-playbook -i hosts deploy.yml```

If everything has been set up properly, this command will create the following approximate directory structure on your server. Check how the hosts folder structure would look like after one, two and three deployments.

```
-- /var/www/my-app.com
|-- current -> /var/www/my-app.com/releases/20100509145325
|-- releases
|   |-- 20100509145325
|-- shared
```

```
-- /var/www/my-app.com
|-- current -> /var/www/my-app.com/releases/20100509150741
|-- releases
|   |-- 20100509150741
|   |-- 20100509145325
|-- shared
```

```
-- /var/www/my-app.com
|-- current -> /var/www/my-app.com/releases/20100512131539
|-- releases
|   |-- 20100512131539
|   |-- 20100509150741
|   |-- 20100509145325
|-- shared
```

### Serial deployments

To prevent different timestamps when deploying to several servers using the [`serial`](http://docs.ansible.com/playbooks_delegation.html#rolling-update-batch-size) option, you should set the `sebar_release_version` variable.

```ansible-playbook -i hosts -e "sebar_release_version=`date -u +%Y%m%d%H%M%SZ`" deploy.yml```


Rolling back
-----------

In order to rollback with sebar, you need to set up the deployment and run the rollback playbook.

```ansible-playbook -i hosts rollback.yml```

If you try to rollback with zero or one releases deployed, an error will be raised and no actions performed.

Variables you can tune in rollback role are less than in deploy one:

```yaml
vars:
  sebar_deploy_to: "/var/www/my-app" # Base path to deploy to.
  sebar_version_dir: "releases" # Releases folder name
  sebar_current_dir: "current" # Softlink name. You should rarely changed it.

  # Hooks: custom tasks if you need them
  sebar_before_symlink_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-before-symlink-tasks.yml"
  sebar_after_symlink_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-after-symlink-tasks.yml"
  sebar_before_cleanup_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-before-cleanup-tasks.yml"
  sebar_after_cleanup_tasks_file: "{{ playbook_dir }}/<your-deployment-config>/my-after-cleanup-tasks.yml"
```

Multistage environment (devel, preprod, prod, etc.)
---------------------------------------------------

If you want to deploy to different environments such as devel, preprod and prod, it's recommended to create different hosts files. When done, you can specify a different host file when running the deployment playbook using the **-i** parameter. On every host file, you can specify different users, password, connection parameters, etc.

```ansible-playbook -i hosts_devel deploy.yml```

```ansible-playbook -i hosts_preprod deploy.yml```

```ansible-playbook -i hosts_prod deploy.yml```

Hooks: Custom tasks
-------------------

You will typically need to reload your webserver after the `Symlink` step, or download your dependencies before `Code update` or even do it in production before the `Symlink`. So, in order to perform your custom tasks you have some hooks that sebar will execute before and after each of the main 3 steps. **This is the main benefit against other similar deployment roles.**

```
-- /my-local-machine/my-app.com
|-- hosts
|-- deploy.yml
|-- my-custom-tasks
|   |-- before-code-update.yml
|   |-- after-code-update.yml
|   |-- before-symlink.yml
|   |-- after-symlink.yml
|   |-- before-cleanup.yml
|   |-- after-cleanup.yml
```

For example, in order to restart apache after `Symlink` step, we'll add in the `after-symlink.yml`

```
- name: Restart Apache
  service: name=httpd state=reloaded
```

* **Q: Where would you add sending email notification after a deployment?**
* **Q: (for PHP and Symfony developers) Where would you clean the cache?**

You can specify a custom tasks file for before and after every step using `sebar_before_*_tasks_file` and `sebar_after_*_tasks_file` role variables. See "Role Variables" for more information.

Variables in custom tasks
-------------------------

When writing your custom tasks files you may need some variables that sebar makes available to you:

* ```{{ sebar_release_path.stdout }}```: Path to current deployment release (probably the one you are going to use the most)
* ```{{ sebar_releases_path }}```: Path to releases folder
* ```{{ sebar_shared_path }}```: Path to shared folder (where common releases assets can be stored)
* ```{{ sebar_release_version }}```: Relative directory name for the release (by default equals to the current timestamp in UTC timezone)

Pruning old releases
--------------------

In continuous delivery environments, you will possibly have a high number of releases in production. Maybe you have tons of space and you don't mind, but it's common practice to keep just a custom number of releases.

After the deployment, if you want to remove old releases just set the `sebar_keep_releases` variable to the total number of releases you want to keep.

Let's see three deployments with an `sebar_keep_releases: 2` configuration:

```
-- /var/www/my-app.com
|-- current -> /var/www/my-app.com/releases/20100509145325
|-- releases
|   |-- 20100509145325
|-- shared
```

```
-- /var/www/my-app.com
|-- current -> /var/www/my-app.com/releases/20100509150741
|-- releases
|   |-- 20100509150741
|   |-- 20100509145325
|-- shared
```

```
-- /var/www/my-app.com
|-- current -> /var/www/my-app.com/releases/20100512131539
|-- releases
|   |-- 20100512131539
|   |-- 20100509150741
|-- shared
```

See how the release `20100509145325` has been removed.

Example Playbook
----------------

In the folder, `example` you can check an example project that shows how to deploy a small application with sebar.

In order to run it, you will need to have Vagrant and the sebar roles installed. Please check https://www.vagrantup.com for more information about Vagrant and our Installation section.

```
$ cd example/my-playbook
$ vagrant up
$ ansible-playbook -i hosts deploy.yml
```

And after running these commands, the index.html located in the `my-app` folder will be deployed to both vagrant boxes

In order to test the rollback playbook, you will need to run deploy.yml at least twice (so that there is something to rollback to). And once this is done, you only need to run

```
$ ansible-playbook -i hosts rollback.yml
```

You can check more advanced examples inside the test folder which are run against Travis-CI

Sample projects
---------------

We have added sebar support for other projects we are working on.

* LastWishes: Domain-Driven Design PHP Sample App: https://github.com/dddinphp/last-wishes

As an example, see the execution log of the LastWishes deployment:

```
PLAY [Deploy last wishes app to my server] ************************************

GATHERING FACTS ***************************************************************
ok: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Ensure deployment base path exists] ***
ok: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Ensure releases folder exists] ***
ok: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Ensure shared elements folder exists] ***
ok: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Get release timestamp] ***********
changed: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Get release path] ****************
changed: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Get releases path] ***************
changed: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Get shared path (in rsync case)] ***
changed: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Rsync application files to remote shared copy (in rsync case)] ***
changed: [quepimquepam.com -> 127.0.0.1]

TASK: [carlosbuenosvinos.sebar-deploy | Deploy existing code to servers] ***
changed: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Deploy existing code to remote servers] ***
skipping: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Update remote repository] ********
skipping: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Export a copy of the repo] *******
skipping: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Deploy code from to servers] *****
skipping: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Copy release version into REVISION file] ***
changed: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Touches up the release code] *****
changed: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Change softlink to new release] ***
changed: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Reload Apache] *******************
changed: [quepimquepam.com]

TASK: [carlosbuenosvinos.sebar-deploy | Clean up releases] ***************
skipping: [quepimquepam.com]

PLAY RECAP ********************************************************************
quepimquepam.com           : ok=14   changed=10   unreachable=0    failed=0
```

License
-------

MIT
