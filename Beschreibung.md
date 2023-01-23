VM vertraut werden

Wir finden die VM über folgenden link, brauchen da aber nicht raufkucken: https://portal.azure.com/#@brightstraining.com/resource/subscriptions/afc373bb-3a48-411a-a448-2cd0314f6fc6/resourceGroups/DevOps2022_Jenkins_Machines/overviewsiehe 

vm "frosty" details:

snowman
20.218.111.156
eB1aA##nT9nsM1

1. ssh login in der command line in vs code: 
"ssh username@IP" und dann fragt er nach dem passwort

2. einloggen, file anlegen  (touch) und files von anderen team membern finden

3. mit Ansible file auf VM schreiben.

    a. Jenkinscontainer starten
    - hostfile anlegen: darin die IP Adresse der VM, die ssh Connection und den username definieren, 

```bash

[vm]
20.218.111.156

[all:vars]
ansible_connection=ssh //erlaubt einen Username/Password zu benutzen. dafault würde ein Key sein.
ansible_user=snowman

```


    - playbook.yml anlegen: Arbeitsanweisung für Ansible 
    - Jenkinsfile in github (rep: HELLO-WORLD_ULRIKE) anlegen: in der pipeline auf hostfile und playbook verweisen!
    - In Jenkins: Credentials für VM anlegen (`vm_frosty`). credentials in jenkins erstellen für die VM! wichtig keine credentials in git comitten

Dazwischen immer wieder in Jenkins Build for main und an den Fehlermeldungen entlanghangeln, und alles fixen.


Tasks: cross-funktional
z.B.: Für alle Funktionen eine ssh-Verbinung erstellen
      Für alle Funktionen das Passwort und der Username weiterreichen


Assassment:

- Verschlüsselung, Paket mit Zertifikat, aber Key passt nicht - was kann passiert sein?

- Metrik erzeugen mit nxingx, prmetheus abgeholt, grafan abgeholt: Fragen rund um das Thema Metriken. Grob verstehen, dass es einen Metrikendpunkt gibt, wo Metriken abgeholt werden, Prometheus holt ab. Szenario: man sieht den Metrik-Endpunkt aber es kommen auf Grafana keine Meztriken an - was könnte das Problem sein?
Oder: das generelles Setup beschreiben, dann Schnipsel zeigen und Frage: wo steckt der Fehler? (Prometheus)
Grafana: Source bei Grafana angeben, dann Fehler einbauen, z.B. Localhost eingeben und dann kucken, wo der Fehler ist.
Üben: Vom gesunden Setup aus starten, dann einzelne Dinge verändern und die Fehlermeldung verstehen lernen.
Metric-Pfad fehlt, Quelle liegt nicht auf Localhost sondern auf nginx

- Aufgabe ist zu groß, wie geht ihr im agilen Setup weiter vor (z.b. Backlog ist zu groß)

- Ansible: Automatisierung, die auf der VM Docker installieren kann. Nach Java: Eine Automatisierung erstellen, die auf VM Jenkins installieren kann

Backlog Aufgabe:

## Create another playbook that installs docker on the team vm.

translate the install description for docker into ansible stept.
https://docs.docker.com/engine/install/ubuntu/

BE VERY SLOW AND MAKE SURE YOUR FEEDBACK LOOP IS TIGHT

find a way to make tiny steps and verify your tiny steps

Here are some modules you will need:
apt, apt_key, apt_repository, service, user

https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_key_module.html
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_repository_module.html
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html



Examples on how to check some of the aspects of the installation:

apt-key adv --list-public-keys --with-fingerprint --with-colons | grep docker

### How to install dependencies with ansible.builtin.apt

Final result:





1.) Start your playbook with the reference to the host like in our first playbook if you haven't done already:

- hosts: all

Info: "all" or name of host in your hostfile



2.) Add activate privilege escalation for tasks

  become: true

Info: Tasks will be executed as root. Otherwise, ansible can not install dependencies and outputs an "permission denied" error.

you can activate privilege escalation for all tasks or just for specific tasks.



3.) Add the task and the ansible.builtin.apt module to install dependencies for docker:

  tasks:
    - name: install dependencies  #Just the name of the task
      ansible.builtin.apt:        #Manages apt packages
        name:                     #A list of package names. aliases: package, pgk
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        state: latest             #ensures that the latest version is installed: default="present"
        update_cache: true        #runs the equivalent of apt-get update before the operation



4.) How to check if the task is setup correct and all dependencies are installed:

push to repo and build and start in jenkins.

    - name: check if dependencies are installed
      ansible.builtin.shell: dpkg -l 'ca-certificates' && dpkg -l 'curl' &&  dpkg -l 'gnupg' && dpkg -l 'lsb-release'
      register: certificates
      failed_when: certificates.stderr != ''
      changed_when: false



To 1.): ca-certificates is a package that contains root certificates that are used to establish trust for secure connections to servers. It allows software like curl and gnupg to verify the authenticity of SSL/TLS certificates when establishing a secure connection to a server.

curl is a command-line tool for transferring data using various protocols, including HTTP, HTTPS, and FTP. It can be used to download and upload files, as well as to send and receive data from servers.

gnupg is the GNU Privacy Guard, a free software implementation of the OpenPGP standard for encrypting and signing data. It can be used to securely transmit data over the internet, verify the authenticity of signed documents, and generate and manage cryptographic keys.

lsb-release is a utility that displays information about the Linux Standard Base (LSB) and the distribution of Linux that you are using. It can be used to determine the version and release number of your distribution, as well as the vendor and vendor version.



See documentation for further information about docker dependencies here and about the ansible.builtin.apt module here and in the comm

## add docker repo with ansible.builtin.apt_repository

Add Docker Repository

    - name: add Docker Repository
      ansible.builtin.apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu bionic stable



Check if Repo is added

This command in your VM shows the Repository list, if your ansible playbook entry was successful

cat  /etc/apt/sources.list.d/download_docker_com_linux_ubuntu.list

Else your VM will show ‘No such file or directory’



oder ein weiteres Task in das playbook schreiben mit dem ansible shell skript:

    - name: test docker rep susseccfull
      ansible.builtin.shell: sudo apt-cache policy | grep docker

    - name: check added docker repository
      ansible.builtin.shell: cat "/etc/apt/sources.list.d/Docker.list"
      register: dockerrepo
      failed_when: dockerrepo.stderr != ''
      changed_when: false


### install docker with ansible.builtin.apt

Wieso? -> ¿ Später wollen wir Jenkins in einem Docker-Container betreiben...

Es gibt mehrere Gründe, warum es sinnvoll sein könnte, Docker über Ansible auf eine VM zu installieren:

Automatisierung: Ansible ermöglicht es, die Installation von Docker auf der VM in einem Skript zu automatisieren. Dies kann besonders nützlich sein, wenn Sie Docker auf mehreren VMs installieren müssen.

Wiederholbarkeit: indem Sie die Installation von Docker in einem Ansible-Skript festhalten, können Sie sicherstellen, dass die Installation jedes Mal gleich ist und reproduzierbar ist.

Überwachung: Ansible bietet die Möglichkeit, den Fortschritt der Installation von Docker zu verfolgen und zu überwachen. Sie können auch Benachrichtigungen einrichten, um über den Fortschritt der Installation informiert zu werden.

Fehlerbehebung: Wenn bei der Installation von Docker auf der VM ein Fehler auftritt, können Sie das Ansible-Skript verwenden, um den Fehler zu diagnostizieren und zu beheben.

Zentralisierung: indem Sie Ansible verwenden, um Docker auf der VM zu installieren, können Sie die Installation von Docker an einem zentralen Ort verwalten und verfolgen.

Packete

Docker CE (Community Edition) ist eine freie, open-source Container-Orchestrierungsplattform, die es ermöglicht, Anwendungen in Container zu packen und diese Container auf jedem Host mit Docker-Unterstützung auszuführen.

Docker CE CLI (Command Line Interface) ist ein Befehlszeilenwerkzeug, das verwendet wird, um Docker-Befehle auszuführen. Es kann verwendet werden, um Container zu verwalten, Images zu bauen und zu veröffentlichen, und vieles mehr.

Containerd_io ist ein Container-Orchestrierungsdienst, der in Docker CE enthalten ist. Es ist verantwortlich für die Ausführung und Verwaltung von Containern auf dem Host.

Docker Compose Plugin ist ein Plugin für Docker, das es ermöglicht, mehrere Docker-Container mithilfe von YAML-basierten Definitionen zu verwalten. Es kann verwendet werden, um komplexere Anwendungen, die aus mehreren Containern bestehen, zu definieren und zu verwalten.

Diese vier Pakete werden gemeinsam installiert, um Docker CE auf dem Ziel-Host zu installieren und zu betreiben.

Ausblick ?

Es kann eine gute Idee sein, Docker mithilfe von Ansible auf Kubernetes-Pods zu installieren. Ansible ist ein leistungsfähiges Werkzeug für die Automatisierung von IT-Infrastrukturen und -Prozessen, das es Ihnen ermöglicht, Docker auf einer Vielzahl von Hosts zu installieren und zu verwalten. Mit Ansible können Sie Docker auf Kubernetes-Pods installieren, indem Sie ein Playbook erstellen, das die erforderlichen Aufgaben definiert und ausführt. So können Sie die Installation von Docker auf Kubernetes-Pods schnell und zuverlässig durchführen.



Add The Task in your playboo. yml file :

  tasks:
    - name: install Docker
      ansible.builtin.apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-compose-plugin
        state: latest
        update_cache: true





How to check if docker installed:

sign in your VM and type in termional : docker --version

your will see as reply like this- Docker version 20.10.22, build 3a2c30b

    - name: check if docker is installed
      ansible.builtin.shell: dpkg -l 'docker-ce' && dpkg -l 'docker-ce-cli' && dpkg -l 'containerd.io' && dpkg -l 'docker-compose-plugin'
      register: docker
      failed_when: docker.stderr != ''    #fails if stderr is NOT empty
      changed_when: false


### check if docker is running: ansible.builtin.servicehttps://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html

playbook.yaml unter docker installation unter Tasks:

name: Name des Tasksansible.builtin.servicename: Name des services (z.B. Docker)Parameter: state

´´´

hosts: alltasks:

name: Is docker runningansible.builtin.service:name: dockerstate: started´´´how to check if it works???

Docker should already running after installation. This checks also if the docker service is already running.

    - name: check if docker is running
      ansible.builtin.service:
        name: docker
        state: started

### add user to docker group ansible.builtin.user

- name: Add the user 'Schultes' with a specific uid and a primary group of 'admin'
   ansible.builtin.user:
      name: Schultes
      comment: Felix Schultes
      uid: 1041
      group: admin



    - name: add the VM user to docker group
      ansible.builtin.user:
        name: snowman       #depends on your VM
        groups: docker      #we want to add our VM user to the docker group
        uid: 1000       #not needed bcs user on you VM already has uid:1000
        append: true

test a hello-world container with ansible.builtin.docker_container

Das passende Ansible Modul für Docker Containers findet mann nicht unter ansible.builtin aber hier:
https://docs.ansible.com/ansible/latest/collections/community/docker/docker_container_module.html

To check whether the Hello-World Container (s) is running: enter
sudo docker ps -a in the VM command line.

Run hello-world container

    - name: run hello-world container
      ansible.builtin.docker_container:
        name: hello-world           #container name
        image: hello-world          #image
        state: started

Check if hello-world container was started

    - name: check if hello-world container was started
      ansible.builtin.shell: docker ps | grep hello-world
      register: container
      failed_when: container.stderr != ''     #failes when stderr is NOT empty/fails when there is a error in stderr
      changed_when: false

Print result in jenkins output console if hello-world container is running

    - name: print result
      ansible.builtin.debug:
        msg: "Hello-world container is running."


