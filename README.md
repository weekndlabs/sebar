Sebar
==========

**sebar.deploy** are ansible roles to easily manage the deployment process for scripting applications such as Nodejs, Golang, PHP, Python and Ruby etc.

Installation
------------

sebar.sebar is an ansible role distributed globally using [Ansible Galaxy](https://galaxy.ansible.com/). In order to install sebar role you can use the following command.

```
$ ansible-galaxy install fajarhide.sebar
```

Update
------

```
$ ansible-galaxy install --force fajarhide.sebar
```

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

License
-------

MIT
