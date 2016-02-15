#Author ximepa <ximepa@tadaam.me>
# Ansible pour le configuration management :
## Avantages

1. *un management simple*
Pas d'infrastructure de contrôle (serveur, base de donnée, etc).
On doit juste installer le binaire ansible (par exemple sur un laptop), et on peut ainsi contrôler une flotte de serveurs distants depuis ce point central.
Une machine contrôlée par Ansible ne laisse pas de codes ou fichiers spécifiques. Donc changer de version d'ansible n'impacte pas les flottes managées. </br>Il n'y a par ailleurs pas de "trace visible" sur les serveurs que ceux-cis répondent à des instructions d'un software distant. Pas de serveur central, pas de base de donnée de configuration à maintenir.

2. *push vs pull*
Par défaut, ansible fonctionne en "push", mais il y a aussi un mode "pull" où les clients se connectent à un serveur central régulièrement pour exécuter les tâches qui leur correspondent.
</br>Essentiellement, on a un master qui voit et orchestre toute les pieces de l'infrastructure, plutôt que d'avoir chaque pièce qui essaie de se maintenir dans un état en parlant à ses voisins.

3. *Un software simple, petit.*
Ansible, c'est seulement quelques milliers de lignes, très auditées.
Il se connecte à la flotte par SSH, et upload des petits modules python pour exécuter les commandes.

4. *Facile à maintenir:*
Ansible, c'est la démarche "Infrastructure as code",mais un code immédiatement lisible par des humains, un niveau d'abstraction assez bas, donc quasi pas de learning curve. Les scripts sont facilement maintenables (si une tâche ne marche pas, elle ne fait pas planter l'ensemble). Les scripts sont facilement debuggables. Enfin, lire un script écrit par un autre est immédiat, contrairement à d'autres languages
</br>L'idée est de ne pas passer plus de temps à débugger le code qui manage l'infrastructure que sur l'infrastructure elle-même. Dans d'autres outils (puppet, chef), l'architecture de l'outil fait qu'un changement implique beaucoup plus d'effort. Le modèle simple d'Ansible est un avantage.

5. *Historique et versionning, idempotence*
L'ensemble des tâches et configuration étant fichiers, tout est versionnable par les outils de git classique. On peut donc revenir à un état antérieur de l'Infrastucture facilement.</br>
Par ailleurs, la propriété d'idempotence implique qu'exécuter plusieurs fois un e tâche ansible sur la même machine d'impacte pas celle-ci. Pour chaque tâche élémentaire, celle-ci n'est exécutée que si le besoin existe. Par exemple : une ligne ne sera insérée dans un fichier que si 1/elle n'existe pas ou 2/elle est différente.
Changer l'infrastructure revient donc à éditer les fichiers existants et réexécuter l'ensemble.

5. *modulaire*
Ansible est petit, et contient plus de 200 modules pour gêrer les infrastructures spécifiques. L'idée est de n'utiliser que ce dont on a besoin. Chaque module suit la philosphie Unix : "do one thing, do it well"

6. *Pas de dépendances*
Ansible a seulement pour dépendances Python et SSH. Et python peut lui-même être installé par Ansible-raw. Donc n'importe quelle machine ayant SSH peut être intégrée à la l'infrastructure managée.
Il n'y a pas d'agent, ni de traces laissées après un "run".

8. *granularité*
Une infrastructure est séparée en groupes de machines.
un "playbook", ou liste de tâche, peut être exécuté sur un subset limité de machines de l'infrastructures. (exemple : tout les CoreOS, mais pas les Ubuntu, ou encore : les machines à Paris, pas celles de Maurice). </br>
Ensuite, un playbook comprend un ensemble de rôle (provisionner nginx, copier le contenu d'un disque...), chaque rôle comprenant des task. Il est possible de n'exécuter que certaines tasks, ou que certains rôles.
Il est même possible de "tagger" une partie des tâches, et de n'exécuter que les tasks taggées ainsi.

9. *Safe run*
Chaque role ou tâche peut être executé avec un flag --check. Dans ce mode d'exécution, Ansible va simuler en mémoire l'exécution de chacune des tâches, et renvoyer un résultat. Cela permet de vérifier que la "play",ou partition, s'exécute correctement sur l'ensemble du parc. Dans ce mode, Les serveurs ne prennent pas en compte les modifications.
Si le résultat est bon, on peut relancer la partition en mode normal, et mettre à niveau les serveurs de la flotte.

## Illustration
9. *un exemple de play*

	```YAML
	- name: Set docker daemon options
	  copy:
	    content: "DOCKER_OPTS=\"{{ docker_opts.rstrip('\n') }}\""
	    dest: /etc/default/docker
	    owner: root
	    group: root
	    mode: 0644
	  notify:
	    - Reload docker
	  when: docker_opts != ""

	- name: Create a group for docker_user
	  group:
	    name: "{{ docker_group }}"
	    state: present
	  when: docker_user != "" and docker_group != ""

	- name: Let a user run docker without sudo
	  become: yes
	  user:
	    name: "{{ docker_user }}"
	    groups: "{{ docker_group }}"
	    state: present
	  when: docker_user != ""

	- name: Start docker
	  service:
	    name: docker
	    state: started

	- name: login to registry
	  shell : "docker login -e {{ registry_mail }} -p {{ registry_pass }} -u {{ registry_user }} {{ registry_host }}"
	  # become_user: "{{ docker_user }}"
	  become: yes
	  tags: docker
	```
ce fichier, illustrant la lisibilité d'ansible, correspond au fichier task/main.yml du role docker-provision de notre infrastructure (voir suivant)

10. *structure de l'infrastructure*

	```text
	├── ansible.cfg
	├── filter_plugins
	├── group_vars
	├── hosts.ini
	├── host_vars
	├── initial.yml
	├── library
	├── roles
	│   ├── docker-config
	│   │   ├── files
	│   │   │   └── touch.txt
	│   │   ├── handlers
	│   │   │   └── main.yml
	│   │   ├── meta
	│   │   │   └── main.yml
	│   │   ├── tasks
	│   │   │   └── main.yml
	│   │   ├── templates
	│   │   └── vars
	│   │       └── main.yml
	│   ├── docker-provision
	│   │   ├── files
	│   │   │   └── touch.txt
	│   │   ├── handlers
	│   │   │   └── main.yml
	│   │   ├── meta
	│   │   │   └── main.yml
	│   │   ├── tasks
	│   │   │   └── main.yml
	│   │   ├── templates
	│   │   └── vars
	│   │       └── main.yml
	│   └── init-provision
	│       ├── files
	│       │   └── touch.txt
	│       ├── handlers
	│       │   └── main.yml
	│       ├── meta
	│       │   └── main.yml
	│       ├── tasks
	│       │   └── main.yml
	│       ├── templates
	│       └── vars
	│           └── main.yml
	├── site.yml
	├── ssh.config
	└── tasks
	```
	on a dans cet exemple :
	- 3 rôles,
	- une infrastructure décrite dans hosts.ini
	- le provisionnement ssh écrit dans ssh.config
	Pour chaque rôle, on a des variables et fichiers nécessaires (exemple: certificats) dans un directory spécifique.
	Cette structure est la structure optimale recommandée

