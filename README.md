Sebar
==========

**sebar ((deploy))** merupakan ansible role untuk mempermudah mengelola proses rilis version untuk aplikasi scripting seperti Nodejs, Golang, PHP, Python dan Ruby dll.

Installation
------------

Untuk menginstal role **sebar**, Anda dapat menggunakan perintah berikut dengan **ansible-galaxy**

```
$ ansible-galaxy install fajarhide.sebar
```

Update
------

```
$ ansible-galaxy install --force fajarhide.sebar
```

Variables
---------

```yaml
vars:
  sebar_deploy_from: "{{ playbook_dir }}" # Dimana proyek local dibuat
  sebar_deploy_to: "/var/www/my-app" # Path tujuan kode dirilis
  sebar_keep_releases: 10 # Untuk menentukan jumlah release yang akan di pertahankan

  # Custom task jika dibutuhkan
  sebar_before_setup_tasks_file: "{{ playbook_dir }}/<your-config>/my-before-setup-tasks.yml"
  sebar_after_setup_tasks_file: "{{ playbook_dir }}/<your-config>/my-after-setup-tasks.yml"
  sebar_before_update_code_tasks_file: "{{ playbook_dir }}/<your-config>/my-before-update-code-tasks.yml"
  sebar_after_update_code_tasks_file: "{{ playbook_dir }}/<your-config>/my-after-update-code-tasks.yml"
  sebar_before_symlink_shared_tasks_file: "{{ playbook_dir }}/<your-config>/my-before-symlink-shared-tasks.yml"
  sebar_after_symlink_shared_tasks_file: "{{ playbook_dir }}/<your-config>/my-after-symlink-shared-tasks.yml"
  sebar_before_symlink_tasks_file: "{{ playbook_dir }}/<your-config>/my-before-symlink-tasks.yml"
  sebar_after_symlink_tasks_file: "{{ playbook_dir }}/<your-config>/my-after-symlink-tasks.yml"
  sebar_before_cleanup_tasks_file: "{{ playbook_dir }}/<your-config>/my-before-cleanup-tasks.yml"
  sebar_after_cleanup_tasks_file: "{{ playbook_dir }}/<your-config>/my-after-cleanup-tasks.yml"
```

License
-------

MIT
