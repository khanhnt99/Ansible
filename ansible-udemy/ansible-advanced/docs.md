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

## 4. Asynchronous actions
- Run a process and check on it later
- Run multiple processes at once and check on them later
- Run processes and forget
- `async`: how long to run?
- `poll`: how frequently to check ? (default 10 seconds)
- `registrer: webapp_result`

-  Monitor the web application for 6 minutes to ensure its running OK. But dont want to hold the SSH connection. 

  ```
  -
    name: Monitor Web Application for 6 Minutes
    hosts: web_server
    command: /opt/monitor_webapp.py
    async: 360
  ```

- `https://medium.com/@knoldus/ansible-asynchronous-actions-and-polling-9e8ede5c3032`

```
-
  name: Deploy a mysql DB
  hosts: db_server
  roles:
    - python
    - mysql_db

-
  name: Deploy a Web Server
  hosts: web_server
  roles:
    - python
    - flask_web

-
  name: Monitor Web Application for 6 Minutes
  hosts: web_server
  command: /opt/monitor_webapp.py
  async: 360
  poll: 0

-
  name: Monitor Database for 6 Minutes
  hosts: db_server
  command: /opt/monitor_database.py
  async: 360
  poll: 0
```

- `https://www.middlewareinventory.com/blog/ansible-async/`


