# Ansible advanced
## 1. Web application
- Step
    + Python
    + Install and configure Mysql 
    + Install Flask
    + Download source code from repository

- Step Ansible
    + Install dependencies
    + Install Mysql Database
    + Start Mysql service
    + Create application database
    + Create application database user
    + Install Python flask dependencies
    + Copy web server code
    + Run web-server

```
-
  name: Deploy a web application
  hosts: db_and_web_server
  vars:
    db_name: 'employee_db'
    db_user: 'db_user'
    db_password: 'Passw0rd'
  tasks:
    - name: Install dependencies
      apt: name={{ item }} state=present
      with_items:
       - python
       - python-setuptools
       - python-dev
       - build-essential
       - python-pip
       - python-mysqldb

    - name: Install MySQL database
      apt:
        name: "{{ item }}"
        state:  present
      with_items:
       - mysql-server
       - mysql-client

    - name: Start Mysql Service
      service:
        name: mysql
        state: started
        enabled: yes

    - name: Create Application Database
      mysql_db: name={{ db_name }} state=present


    - name: Create Application DB User
      mysql_user: name={{ db_user }} password={{ db_password }} priv='*.*:ALL' host='%' state='present'

    - name: Install Python Flask dependencies
      pip:
        name: '{{ item }}'
        state: present
      with_items:
       - flask
       - flask-mysql

    - name: Copy web-server code
      copy: src=app.py dest=/opt/app.py

    - name: Start web-application
      shell: FLASK_APP=/opt/app.py nohup flask run --host=0.0.0.0 &
      
```

## 2. File Separation
- Folder
    - group_vars
    - tasks

## 3. Roles
- `ansible-galaxy init webserver`
- **`webserver`**
  + `tests`
  + `tasks`
  + `handlers`
  + `vars`
  + `defaults`
  + `meta`
- `ansible-galaxy import webserver`
- `ansible-galaxy init mysql_db`
- `ansible-galaxy init flask_web`

```
- 
  name: Deploy database
  hosts: db_server
  roles:
  - mysql_db

- 
  name: Deploy web application
  hosts: web_server
  roles: 
  - flask_web
```
