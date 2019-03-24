# Deploying Node.js application with ansible

### Before Running the script
Run this commands in the remote server
```
$ sudo mkdir /var/www/nodejs
$ sudo chown ubuntu:ubuntu /var/www/nodejs
```

1. clone the node.js application in ```releases``` folder
2. In ```deploy.yml``` file, replace the git repository with the repository of you node application which deployed on the server

Final ansible playbook will look like this
```
---
- hosts: prod
  vars:
    project_path: /var/www/nodejs
  tasks:
    - name: Set some variable
      set_fact:
        release_path: "{{ project_path }}/releases/{{ lookup('pipe','date +%Y%m%d%H%M%S') }}"
        current_path: "{{ project_path }}/current"
    - name: Retrieve current release folder
      command: readlink -f current
      register: current_release_path
      ignore_errors: yes
      args:
        chdir: "{{ project_path }}"
    - name: Create new folder
      file:
        dest={{ release_path }}
        mode=0755
        recurse=yes
        state=directory
    - name: Clone the repository
      git:
        repo: git@github.com:USERNAME/REPO.git
        dest: "{{ release_path }}"
    - name: Update npm
      npm:
        path={{ release_path }}
    - name: Update symlink
      file:
        src={{ release_path }}
        dest={{ current_path }}
        state=link
    - name: Delete old pm2 process
      command: pm2 delete ws-node
      ignore_errors: yes
    - name: Start pm2
      command: pm2 start {{ current_path }}/server.js --name node-app
    - name: Delete old dir
      shell: rm -rf {{ current_release_path.stdout }}/
      when: current_release_path.stdout != current_path
```

3. Run the command ``` ansible-playbook deploy/deploy.yml ``` to deploy to server
