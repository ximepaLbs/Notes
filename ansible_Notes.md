### ssh stuff

2. gestion du bastion
liens 
syntaxe from [this blog](http://brokenbad.com/better-handling-of-public-ssh-keys-using-ansible/)
"discovery ssh" [cet article](https://juriansluiman.nl/article/151/managing-ssh-known-hosts-with-ansible)

4. ssh for ansible

on crée un fichier ssh.config contenant : 

	Host bastion
	    User                   ansible-user
	    HostName               172.16.0.250
	    ProxyCommand           none
	    IdentityFile           /home/ximepa/.ssh/Hosts_tadaam/Ansible/id_ansible_administrator_rsa
	    BatchMode              yes
	    PasswordAuthentication no
	    ForwardAgent           yes
	    LogLevel               ERROR

	Host acs-pprod
	    HostName               10.0.255.242
	    StrictHostKeyChecking  no
	    LogLevel               ERROR
	    ServerAliveInterval    60
	    TCPKeepAlive           yes
	    ProxyCommand           ssh -q -A ansible-user@172.16.0.250 nc %h %p
	    ControlMaster          auto
	    ControlPath            ~/.ssh/tmp/mux-%r@%h:%p
	    ControlPersist         8h
	    User                   ansible-user
	    IdentityFile           /home/ximepa/.ssh/Hosts_tadaam/Ansible/id_ansible_acs-pprod_rsa

on peut aussi éditer le ProxyCommand pour chacun des hosts du fichier /etc/ansible/hosts selon l'exemple suivant:
	
	Host destination.com
    	ProxyCommand ssh -A user@bastion.com -W %h:%p

8. gestion de l'utilisateur ansible-user
on met un mdp sudoer pour ansibleuser, le même partout, un authentification par clef, et un shell à /bin/false. ou alors un passwd -l
</br>pour locker un compte (donc pas d'authentification password : passwd -l <username>) l'authentification par clef fonctionnera toujours.
Possibilité de dévérrouiler : passwd -u <username>
enfin, pour vérrouiller complètement un comjpte : éditer le shell dans /etc/passwd en /bin/false.

9. pour ajouter une "authorized-keys"
dans vars:

	ssh_users:
	  - name: user1
	    key: "{{ lookup('file', 'user1.pub') }}"
	  - name: user2
	    key: "{{ lookup('file', 'user2.pub') }}"
	  - name: user3
	    key: "{{ lookup('file', 'user3.pub') }}"
	  - name: user4
	    key: "{{ lookup('file', 'user4.pub') }}"
dans tasks:

	 - name: Add ssh user keys
	   authorized_key: user={{ item.name }} key="{{ item.key }}"
	   with_items: ssh_users

## tips ansible

4. pseudo-terminal debug
si "pseudo-terminal will not be allocated because stdin is not a terminal" -> forcer avec `ssh -t -t`


5. vault : j'utilise le vault pour les pass sudoer ?
default : same password for all files

6. filtrer par distribution
[link](https://raymii.org/s/tutorials/Ansible_-_Only_if_on_specific_distribution_or_distribution_version.html)

7. gathering facts about acs-pprod:
`ansible acs-pprod -m setup -i hosts.ini -tree <path/to/facts/directory>`
on peut récupérer des valeurs spécifiques:
`ansible -i hosts.ini infra-init -m setup -a 'filter=ansible_distribution'`

2. loops
with items : to loop
[la doc](https://docs.ansible.com/playbooks_loops.html#standard-loops)
exemple :
	```yaml
	- hosts: all
	  gather_facts: false
	  tasks:
	  - action: shell echo {{item}}
	    with_items:
	    - 1
	    - 2
	    - 3
	    register: task
	  - debug: msg="{{item.item}}"
	    with_items: task.results
	    when: item.changed == True
	```



3. répertoire courant:
There's no concept of current directory in Ansible. You can specify current directory for specific task, like you did in your playbook. The only missing part was the actual command to execute. Try this:


	```
	- name: Go to the folder and execute command
  	  command: chdir=/opt/tools/temp ls
	```

10. repertoire ansible
les bonnes pratiques pour la gestion du [répertoire ansible](http://docs.ansible.com/playbooks_best_practices.html)


## Run de playbook: 
`ansible-playbook -i hosts.ini --limit acs-pprod site.yml --ask-become-pass`
Lance par défaut le site.yml

option --check de "dry-run"

option de lancement sélectif:

`ansible-playbook --start-at-task="install package" <playbook.yml>`
`ansible-playbook --tags=<tagname> ` n'exécute que les tasks taggées "tagname"
`ansible-playbook --skip-tags=<tag not executed>`

lancement d'un playbook spécifique:
`ansible-playbook -s playbooks/practice1.playbook -i hosts.ini`
### to investigate :

installer que si absent avec la commande "when"

###debug: ansible docker_images module