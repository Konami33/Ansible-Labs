# Deploy different websites in remote server using nginx webserver

## Directory Structure

```
.
├── ansible
│   ├── deploy_nginx_websites.yml
│   ├── hosts.ini
│   └── vars
│       └── websites.yml
├── images
├── webpage1
│   └── index.html
├── webpage2
    └── index.html
```

### Step 1: Inventory File (`ansible/hosts.ini`)

```ini
[webservers]
webserver1 ansible_host=192.168.1.10
webserver2 ansible_host=192.168.1.11
```

### Step 2: Variables File (`ansible/vars/websites.yml`)

```yaml
websites:
  - name: webserver1
    src: ../../webpage1/index.html
    dest: /var/www/html/index.html
  - name: webserver2
    src: ../../webpage2/index.html
    dest: /var/www/html/index.html
```

### Step 3: Ansible Playbook (`ansible/deploy_nginx_websites.yml`)

```yaml
---
- name: Deploy two different websites on Nginx
  hosts: webservers
  become: yes
  vars_files:
    - vars/websites.yml
  tasks:
    - name: Ensure Nginx is installed
      apt:
        name: nginx
        state: present
      tags: nginx

    - name: Copy website files
      copy:
        src: "{{ item.src }}"
        dest: "{{ item.dest }}"
      when: inventory_hostname == item.name
      with_items: "{{ websites }}"
      notify: restart nginx

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

### Step 4: HTML Content

Ensure you have your HTML files with the embedded CSS:

**Webpage 1 (`webpage1/index.html`):**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Website 1</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            color: #333;
            text-align: center;
            padding: 50px;
        }
        h1 {
            color: #007BFF;
        }
    </style>
</head>
<body>
    <h1>Welcome to Website 1</h1>
    <p>This is a simple website hosted on webserver1.</p>
</body>
</html>
```

**Webpage 2 (`webpage2/index.html`):**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Website 2</title>
    <style>
        body {
            font-family: 'Trebuchet MS', sans-serif;
            background-color: #e0f7fa;
            color: #333;
            text-align: center;
            padding: 50px;
        }
        h1 {
            color: #00796B;
        }
    </style>
</head>
<body>
    <h1>Welcome to Website 2</h1>
    <p>This is a simple website hosted on webserver2.</p>
</body>
</html>
```

### Step 5: Run the Playbook

Navigate to the `ansible` directory and execute the playbook using the following command:

```bash
cd ansible
ansible-playbook -i hosts.ini deploy_nginx_websites.yml
```

This setup will ensure Nginx is installed on both servers, copy the website files from the specified paths, and restart Nginx if the files are updated. The paths in `websites.yml` are relative to the playbook's location, so the `../../` ensures it correctly references the files in `webpage1` and `webpage2`.