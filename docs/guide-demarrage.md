# Guide de démarrage

Ce guide permet de lancer le laboratoire Ansible localement et de comprendre ce qui se passe pendant l'exécution.

## Objectif

Après ce guide, vous devez pouvoir :

- exécuter le playbook principal
- vérifier le résultat dans le navigateur ou avec `curl`
- modifier les variables du projet
- rejouer le playbook pour observer l'idempotence

## Pré requis

Le projet cible une machine Debian ou Ubuntu.

Installez Ansible :

```bash
sudo apt update
sudo apt install -y ansible
```

Vérifiez l'installation :

```bash
ansible --version
ansible-playbook --version
```

## Comprendre le point d'entrée

La commande principale du projet est :

```bash
ansible-playbook playbooks/site.yml
```

Pourquoi cette commande suffit :

- [ansible.cfg](/root/Ansible/ansible.cfg:1) déclare déjà l'inventaire
- le playbook [playbooks/site.yml](/root/Ansible/playbooks/site.yml:1) appelle directement le rôle `web_demo`
- l'inventaire [inventory/hosts.yml](/root/Ansible/inventory/hosts.yml:1) cible `localhost`

## Première exécution

Depuis la racine du dépôt :

```bash
ansible-playbook playbooks/site.yml
```

Ce que le playbook fait :

1. vérifie que l'OS appartient à la famille Debian
2. met à jour le cache `apt`
3. installe `nginx`, `curl` et `git`
4. génère une page HTML depuis un template Jinja2
5. active et démarre le service `nginx`

## Vérifier le résultat

Test HTTP :

```bash
curl http://localhost
```

État du service :

```bash
systemctl status nginx
```

Visualiser l'inventaire :

```bash
ansible-inventory --graph
```

## Tester sans appliquer

Pour simuler une exécution :

```bash
ansible-playbook playbooks/site.yml --check --diff
```

Cette commande est utile pour comprendre ce qu'Ansible modifierait sans toucher au système.

## Personnaliser le contenu

Modifiez [group_vars/all.yml](/root/Ansible/group_vars/all.yml:1).

Variables principales :

- `demo_site_title`
- `demo_site_message`
- `demo_owner`
- `web_demo_packages`

Après modification, relancez :

```bash
ansible-playbook playbooks/site.yml
```

## Observer l'idempotence

Une bonne séquence d'apprentissage :

1. lancer une première fois le playbook
2. relancer immédiatement une deuxième fois
3. observer qu'il y a peu ou pas de changements
4. modifier une variable
5. relancer et voir qu'Ansible n'applique que ce qui a changé

## Fichiers à connaître

- [ansible.cfg](/root/Ansible/ansible.cfg:1) : configuration du projet
- [inventory/hosts.yml](/root/Ansible/inventory/hosts.yml:1) : inventaire local
- [playbooks/site.yml](/root/Ansible/playbooks/site.yml:1) : point d'entrée
- [group_vars/all.yml](/root/Ansible/group_vars/all.yml:1) : variables personnalisables
- [roles/web_demo/tasks/main.yml](/root/Ansible/roles/web_demo/tasks/main.yml:1) : logique du rôle
- [roles/web_demo/templates/index.html.j2](/root/Ansible/roles/web_demo/templates/index.html.j2:1) : page HTML générée

## Dépannage rapide

Si `nginx` n'est pas accessible :

- vérifiez que le playbook s'est terminé sans erreur
- vérifiez `systemctl status nginx`
- vérifiez qu'aucun autre service n'occupe le port 80

Si le playbook échoue sur le test d'OS :

- le projet n'est prévu que pour Debian ou Ubuntu

Si le contenu de la page ne change pas :

- vérifiez vos variables dans `group_vars/all.yml`
- relancez le playbook
- consultez le HTML généré dans `/var/www/html/index.html`

## Étapes suivantes

Pour faire évoluer ce labo :

- remplacer `localhost` par une VM distante
- ajouter un deuxième rôle
- structurer les variables par environnement
- ajouter des tests avec `molecule`
- introduire `ansible-vault` pour gérer des secrets

## Références

- Ansible getting started: https://docs.ansible.com/projects/ansible/latest/getting_started/introduction.html
- Ansible docs home: https://docs.ansible.com/
