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
- Async được kích hoạt khi chạy ansible nền (daemon) và có thể được kiểm tra lại sau đó.
- Giá trị async là thời gian tối đa mà ansible sẽ đợi cho 1 job đó.
- `async`: how long to run? -> cho biết job được chạy trong bao lâu trước khi Ansible từ bỏ nó -> Bằng cách nào ansible có thể kiểm tra trạng thái của các job này trong daemon -> poll.
- `poll`: how frequently to check ? (default 10 seconds) -> Tần suất kiểm tra trạng thái job đó.
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

## 7. Strategy
- Strategy define how a playbook is executed in Ansible.
- Default: linear (các task thực thi lần lượt trên tất cả các server thành công rồi mới đến task tiếp theo)
- Free
- serial: 3 (chỉ thực hiện chạy trên 3 server từng thời điểm)

## 8. Error Handling
### Task Failure
- Sử dụng khi muốn stop ansible playbook khi 1 trong số các server fail (bình thường playbook sẽ vẫn chạy và skip tại các server fail) -> Sử dụng option `any_errors_fatal: true`.

### Ignore errors, failed_when
- Không quan tâm task sẽ được hoàn thành hay không -> `ingore_errors: yes`

```
- command: cat /var/log/server.log
  register: command_output
  failed_when: "'ERROR'" in command_output.stdout"
```

## 9. Jinja2

```
-
  name: Test Jinja2 Templating
  hosts: localhost
  vars:
    first_name: james
    last_name: bond
  tasks:
  - debug:
      msg: 'The name is {{ last_name }}! {{ first_name }} {{ last_name }}!'
```

```
-
  name: Test Jinja2 Templating
  hosts: localhost
  vars:
    first_name: james
    last_name: bond
  tasks:
  - debug:
      msg: 'The name is {{ last_name | title }}! {{ first_name | title }} {{ last_name | title }}!'
```

```
-
  name: Test Jinja2 Templating
  hosts: localhost
  vars:
    array_of_numbers:
      - 12
      - 34
      - 06
      - 34
  tasks:
  - debug:
      msg: 'Lowest = {{ array_of_numbers | min }} '
```

- Hợp nhất của 2 list 
```
-
  name: Install Dependencies
  hosts: localhost
  vars:
    web_dependencies:
         - python
         - python-setuptools
         - python-dev
         - build-essential
         - python-pip
         - python-mysqldb
    sql_dependencies:
         - python
         - python-mysqldb
  tasks:
  - name: Install dependencies
    apt: name='{{ item }}' state=present
    with_items: '{{  web_dependencies | union(sql_dependencies) }}'
```
- Tạo các file có dang random_file_0,1,...Random từ 1 -> 1000

```
-
  name: Generate random file name
  hosts: localhost
  tasks:
  - name: Install dependencies
    file:
      path: /tmp/random_file"{{ 1000 | random }}"
      state: touch
```

- Return true nếu IP address có tồn tại. Return fail nếu IP address không tồn tại.

```
-
  name: Test valid IP Address
  hosts: localhost
  vars:
    ip_address: 192.168.1.6
  tasks:
  - name: Test IP Address
    debug:
      msg: IP Address = {{ ip_address | ipaddr }}
```

## 10. Lookups
- csv file
```
# credentials.csv
# Credentials File
Hostname,Password
web_server,Passw0rd
db_server,Passw0rd
```

```
-
  name: Test Connectivity
  hosts: web_server
  vars:
    ansible_ssh_pass: "{{ lookup('csvfile', 'web_server file=credentials.csv delimiter=,') }}"
  tasks:
  - name: Ping target host
    ping:
           data: "Test"
```
- ini file
```
# credentials.ini

# Credentials File

[web_server]
password=Passw0rd

[db_server]
password=Passw0rd
```

```
-
  name: Test Connectivity
  hosts: web_server
  vars:
    ansible_ssh_pass: "{{ lookup('ini', 'password section=web_server file=credentials.ini') }}"
  tasks:
  - name: Ping target host
    ping:
           data: "Test"
```
## 11. Vault
## 12. Dynamic inventory


