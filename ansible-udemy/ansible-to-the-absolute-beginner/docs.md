# Ansible to the absolute beginner
## 1. Inventory

- Inventory Parameters
    + `ansible_host`: ip or hostname
    + `ansible_connection`: ssh/winrm/localhost
    + `ansible_port`: 22/5986
    + `ansible_user`: root/admin
    + `ansible_ssh_pass` cho linux - `ansible_password` cho window

```
[all_servers:children]
web_servers
db_servers

[web_servers]
web1
web2
web3

[db_servers]
db1
```

```
web_node1 ansible_host=web01.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node2 ansible_host=web02.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node3 ansible_host=web03.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass

sql_db1 ansible_host=sql01.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass
sql_db2 ansible_host=sql02.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass

[db_nodes]
sql_db1
sql_db2

[web_nodes]
web_node1
web_node2
web_node3

[boston_nodes]
sql_db1
web_node1

[dallas_nodes]
sql_db2
web_node2
web_node3

[us_nodes:children]
boston_nodes
dallas_nodes
```

## 2. YAML
- Key/Value
- Array/Lists
- Dictionary/Map

```
employees:
    -
        name: john
        gender: male
        age: 24
    
    - 
        name: sarah
        gender: female
        age: 28
```

```
employee:
    name: john
    gender: male
    age: 24
    address:
        city: edison
        state: 'new jersey'
        country: 'united states'
    payslips:
        - 
            month: june
            amount: 1400
        - 
            month: july
            amount: 2400
        - 
            month: august
            amount: 3400
```

```
---
car:
  color: blue
  price: "$20,000"
  wheels:
  - 
    model: KDJ39848T
    location: front-right
  - 
    model: MDJ39485DK
    location: front-left
  - 
    model: KCMDD3435K
    location: rear-right
  - 
    model: JJDH34234KK
    location: rear-left
bus:
  color: white
  price: "$120,000"
  wheels: 
  - 
    model: BBBB39848T
    location: front-right
  - 
    model: BBBB9485DK
    location: front-left
  - 
    model: BBBB3435K
    location: rear-right
  - 
    model: BBBB4234KK
    location: rear-left
```

## 3. Playbook
- ansible command:
    + `ansible <hosts> -a <command>`
    + `ansible all -a '/sbin/reboot'`
    + `ansible <hosts> -m <module>`
    + `ansible target1 -m ping`

- ansible plabook
    + `ansible-playbook <playbook-name>`

- `playbook-pingtest.yaml`
    ```
    - 
      name: Test connectivity to target servers
      hosts: all
      tasks:
        - name: Ping test
          ping:
    ```

- `playbook-copyfile.yaml`
    ```
    - 
      name: Copy file to target servers
      hosts: all
      tasks:
        - name: Copy file
          copy:
            src: /tmp/test.txt
            dest: /tmp/test.txt
    ```

## 4. Ansible modules
- command
- file
- script
- service: Manage Services - Start, Stop, Restart
- lineinfile: Search for a line in file and replace it or add it if it doesn't exist

```
-
    name: Execute a script on all web server nodes
    hosts: web_nodes
    tasks: 
        - 
            name: 'Execute a script on all web server nodes'
            script: /tmp/install_script.sh
```

```
-
    name: 'Execute a script on all web server nodes'
    hosts: web_nodes
    tasks:
        -
            name: 'Execute a script on all web server nodes'
            script: /tmp/install_script.sh
        
        -
            name: 'start httpd services on all web nodes'
            service:
                name: httpd
                state: started
```

```
-
    name: 'Execute a script on all web server nodes'
    hosts: web_nodes
    tasks:
        - 
            name: 'Add an entry into /etc/resolv.conf file for hosts'
            lineinfile:
                path: /etc/resolv.conf
                line: nameserver 10.1.250.10
            
        -
            name: 'Execute a script'
            script: /tmp/install_script.sh
        -
            name: 'Start httpd service'
            service:
                name: httpd
                state: present
```

```
-
    name: 'Execute a script on all web server nodes and start httpd service'
    hosts: web_nodes
    tasks:
        -
            name: 'Update entry into /etc/resolv.conf'
            lineinfile:
                path: /etc/resolv.conf
                line: 'nameserver 10.1.250.10'
                
        -
            name: 'Create a new web user'
            user:
                name: web_user
                uid: 1040
                group: developers
            
        -
            name: 'Execute a script'
            script: /tmp/install_script.sh
        -
            name: 'Start httpd service'
            service:
                name: httpd
                state: present
```

## 5. Ansible variables

```
# playbook file
-
    name: 'Update nameserver entry into resolv.conf file on localhost'
    hosts: localhost
    tasks:
        -
            name: 'Update nameserver entry into resolv.conf file'
            lineinfile:
                path: /etc/resolv.conf
                line: 'nameserver {{ nameserver_ip }}'


# Sample Inventory File
localhost ansible_connection=localhost nameserver_ip=10.1.250.10
```

```
# playbook file
-
    name: 'Update nameserver entry into resolv.conf file on localhost'
    hosts: localhost
    tasks:
        -
            name: 'Update nameserver entry into resolv.conf file'
            lineinfile:
                path: /etc/resolv.conf
                line: 'nameserver {{ nameserver_ip }}'
        -
            name: 'Disable SNMP Port'
            firewalld:
                port: '{{ snmp_port }}'
                permanent: true
                state: disabled

# Sample Inventory File

localhost ansible_connection=localhost nameserver_ip=10.1.250.10 snmp_port=160-161
```

```
-
    name: 'Update nameserver entry into resolv.conf file on localhost'
    hosts: localhost
    vars:
        car_model: BMW M3
        country_name: USA
        title: Systems Engineer
    tasks:
        -
            command: 'echo "My car''s model is {{ car_model }}"'
        -
            command: 'echo "I live in the {{ country_name }}"'
        -
            name: 'Print my title'
            command: 'echo "I work as a {{ title }}"'
```

## 6. Conditionals

```
-
    name: 'Execute a script on all web server nodes'
    hosts: all_servers
    tasks:
        -
            service: 'name=mysql state=started'
            when: 'ansible_host=="server4.company.com"'
```

```
-
    name: 'Am I an Adult or a Child?'
    hosts: localhost
    vars:
        age: 25
    tasks:
        -
            command: 'echo "I am a Child"'
            when: 'age<18'
        -
            command: 'echo "I am an Adult"'
            when: 'age>=18'
```
```
-
    name: 'Add name server entry if not already entered'
    hosts: localhost
    tasks:
        -
            shell: 'cat /etc/resolv.conf'
            register: command_output
        -
            shell: 'echo "nameserver 10.0.250.10" >> /etc/resolv.conf'
            when: 'command_output.stdout.find("10.0.250.10") == -1'
```

## 7 Loops
```
-
    name: 'Print list of fruits'
    hosts: localhost
    vars:
        fruits:
            - Apple
            - Banana
            - Grapes
            - Orange
    tasks:
        -
            command: 'echo "{{ item }}"'
            with_items: '{{ fruits }}'
```

```
-
    name: 'Install required packages'
    hosts: localhost
    vars:
        packages:
            - httpd
            - binutils
            - glibc
            - ksh
            - libaio
            - libXext
            - gcc
            - make
            - sysstat
            - unixODBC
            - mongodb
            - nodejs
            - grunt
    tasks:
        -
            yum: 
                name: "{{ item }}" 
                state: 'present'
            with_items: '{{ packages }}'
```

