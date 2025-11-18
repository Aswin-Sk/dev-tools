## Note on Running Ansible Inside WSL

If you are running Ansible inside WSL, avoid placing your Ansible project inside Windows-mounted paths such as:

```
/mnt/c/Users/...
```

WSL mounts the Windows filesystem with permissive default permissions (`0777`).  
From Linux's perspective, this makes the directory **world-writable**, and Ansible will *not* trust or load any `ansible.cfg` file located in an insecure path.

Because of this, settings such as custom `roles_path`, inventory paths, callbacks, and other configuration options may be ignored, and you may see warnings like:


### Recommended Fix

Move your Ansible files into the WSL Linux filesystem, for example:

```
/home/<user>/ansible/
```

This directory uses real POSIX permissions, is not world-writable, and Ansible will correctly load your `ansible.cfg`.


