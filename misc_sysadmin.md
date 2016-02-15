
ajout utilisateur ansible : 
	useradd -u 4545 -G users ansible-user -s /bin/bash

ou avec adduser: 
	
	adduser --disabled-login --gecos "ansible-user" --uid 4545 ansible-user
	usermod -aG sudo ansible-user

dans /home/ansible-user:
	.ssh/authorized_keys:

ansible.cfg et ssh.config : regarder dans Workspace/Ansible

avant de pouvoir appliquer le playbook, on doit faire le provisionning "bootstrap"

#procédure: on se connecte au "bastion" à l'aide de la config ssh
ssh Bastion -F ssh.config

#pour se connecter à tadaam-pprod
ssh tadaam-pprod -F ssh.config

pour pouvoir être "sudoer sans password":

au début on ajoute dans /etc/sudoer la ligne:

	ansible-user ALL=(ALL) NOPASSWD:ALL

à terme, remplacer le dernier champ par une liste de commande autorisée ?

on a donc un ansible-user sudoer

on modifie ensuite le daemon ssh pour forcer l'authentification par clef :
PubkeyAuthentication yes
AuthorizedKeysFile	%h/.ssh/authorized_keys

#initialisation de ansible sur tadaam-pprod
ansible all -m ping -i hosts.ini

#lancement d'un playbook : 
#depuis le répertoire ansible/

ansible-playbook -s playbooks/practice1.playbook -i hosts.ini