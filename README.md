# ansible

### Ansible Directory Layout (with Role)

```
ansible/
├── inventory.ini
├── playbook.yml
└── roles/
    └── webserver/
        ├── tasks/
        │   └── main.yml
        └── files/
            └── index.html
```

### inventory.ini
Assuming container is accessible via Docker or SSH:
```
[web]
<container_name_or_ip> ansible_connection=docker
```

If you use SSH inside the container:
```
[web]
<container_ip> ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### playbook.yml
```
- name: Provision web server inside Docker
  hosts: web
  become: yes
  roles:
    - webserver
```

### roles/webserver/tasks/main.yml
```
- name: Update apt cache
  apt:
    update_cache: yes

- name: Install NGINX
  apt:
    name: nginx
    state: present

- name: Deploy index.html
  copy:
    src: index.html
    dest: /var/www/html/index.html
    mode: '0644'

- name: Ensure NGINX is running
  service:
    name: nginx
    state: started
    enabled: true
```


### roles/webserver/files/index.html
```
<!DOCTYPE html>
<html>
  <head><title>Ansible FTW</title></head>
  <body>
    <h1>Provisioned by Ansible!</h1>
  </body>
</html>
```

## Role Example: webserver
This role will:
Install NGINX
Copy a custom index.html
Start and enable the NGINX service

### Full Role Structure:
```
roles/
└── webserver/
    ├── tasks/
    │   └── main.yml
    ├── files/
    │   └── index.html
    └── handlers/
        └── main.yml
```
We'll now use a handler to restart NGINX only when needed (e.g., if index.html changes)


### roles/webserver/tasks/main.yml
```
- name: Update apt cache
  apt:
    update_cache: yes

- name: Install NGINX
  apt:
    name: nginx
    state: present

- name: Copy custom index.html
  copy:
    src: index.html
    dest: /var/www/html/index.html
    mode: '0644'
  notify: Restart NGINX  # ← this triggers a handler if the file changes

- name: Ensure NGINX is enabled and running
  service:
    name: nginx
    state: started
    enabled: true
```

### roles/webserver/files/index.html
```
<!DOCTYPE html>
<html>
  <head><title>Provisioned by Ansible</title></head>
  <body>
    <h1>This page was configured inside a Docker container using Ansible!</h1>
  </body>
</html>
```

### roles/webserver/handlers/main.yml
```
- name: Restart NGINX
  service:
    name: nginx
    state: restarted
```

## Main Playbook that Uses the Role (playbook.yml)
```
- name: Configure Docker container as a web server
  hosts: web
  become: yes
  roles:
    - webserver
```

Running the Playbook
```
ansible-playbook -i inventory.ini playbook.yml
```
