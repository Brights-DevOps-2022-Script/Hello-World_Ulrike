# helpful snippets

## run docker for local debug

```bash
docker run -v "$(pwd):/workspace" -w /workspace -it  cytopia/ansible
```

and inside the container run

```bash
   export "ANSIBLE_KEY=<snowman password>"
   export "ANSIBLE_CONFIG=etc/ansible/ansible.cfg"
   apk add --update --no-cache openssh sshpass
   ansible-playbook -vvv -i hostfile playbook.yml -e ansible_ssh_pass='{{ lookup("env", "ANSIBLE_KEY") }}'
```