## Kubernetes operator

* Kubernetes can manage the complete lifecycle of stateless applications in a fully automated way, because these applications don’t have business logic for deployment.
So basically, once you deployed the application, you don’t have to sit there and control that your application is running properly.

### StateFUL applications WITHOUT Operator 

* For stateful applications, like databases, the whole process isn’t as straightforward.
  They need more "hand-holding" during its whole lifecycle, because the replicas of stateful apps aren't identical.

* Operator solves this problem and basically replaces this "human" operator with a "software" operator.
 At its core it has the same control loop mechanism that Kubernetes has, that watches for changes in the application state.
It also uses CRDs, which is basically a custom K8s component. So, it takes the basic Kubernetes resources and its controller concept as a foundation to build upon, and on top of that includes application-specific knowledge to automate the entire life cycle of the application it "operates".

managing the complete apps for stateful application via ** kubernetes operator **

example : prometheus operator.

## Let's say you have 2 servers with different ports and username then how can ansible handle different machines for applying and running the same runbook.

```
[webservers]
server1 ansible_host=192.168.1.1 ansible_port=22 ansible_user=ec2-user
server2 ansible_host=192.168.1.2 ansible_port=22 ansible_user=ec2-user

```
```
- name: Example playbook
  hosts: webservers
  tasks:
    - name: Ensure Apache is installed
      apt:
        name: apache2
        state: present
      become: yes
```

``` ansible-playbook -i inventory.ini playbook.yml```

In a ansible playbook, we have a section which could fail on certian nodes but we don't want the playbook to stop or exit beacuse of this. Is it possible to ignore if this part of playbook fails?

```
# playbook.yml
- name: Example playbook with error handling
  hosts: webservers
  tasks:
    - name: This task will succeed
      command: echo "This is a successful command"

    - name: This task might fail but will be ignored
      command: /bin/false
      ignore_errors: yes

    - name: This task will also run even if the previous task fails
      command: echo "This command will run regardless of previous failures"
```
* The second task executes a command (/bin/false) that will fail. By using ignore_errors: yes, we instruct Ansible to ignore the failure and continue with the subsequent tasks.

* Third Task: The third task will run regardless of the outcome of the second task. This demonstrates that the playbook continues executing even after a task fails.
  
## How can we handles secrets in ansible? Let's say we have playbook which needs to login to premises server with login name ans password.

* Create a Vault File: Store your secrets in an encrypted file.
* Encrypt the Vault File: Use Ansible Vault to encrypt the file. ``` ansible-vault encrypt vault.yml ```

```
# playbook.yml
- name: Example playbook using vault secrets
  hosts: webservers
  vars_files:
    - vault.yml

  tasks:
    - name: Ensure a user is logged in
      ansible.builtin.shell: echo "Logged in as {{ ansible_user }}"
      register: login_result

    - name: Print login result
      ansible.builtin.debug:
        msg: "{{ login_result.stdout }}"
```


