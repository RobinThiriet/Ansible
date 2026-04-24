# Projet Ansible 

Un mini projet pour monter en compétence sur Ansible en restant sur un cas concret.

## Objectif

Le playbook configure une machine Linux locale en :

- installant `nginx`, `curl` et `git`
- créant une page HTML générée depuis un template Jinja2
- démarrant et activant le service `nginx`

Le projet est pensé pour être lu, modifié puis rejoué afin de comprendre l'idempotence d'Ansible.

## Structure

```text
.
├── ansible.cfg
├── group_vars/
│   └── all.yml
├── inventory/
│   └── hosts.yml
├── playbooks/
│   └── site.yml
└── roles/
    └── web_demo/
        ├── defaults/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── tasks/
        │   └── main.yml
        └── templates/
            └── index.html.j2
```

## Prérequis

- Ubuntu ou Debian
- Ansible installé sur la machine de contrôle
- `sudo` disponible sur la machine cible

Exemple d'installation sur Ubuntu :

```bash
sudo apt update
sudo apt install -y ansible
```

## Lancer le projet

Depuis la racine du dépôt :

```bash
ansible-playbook playbooks/site.yml --ask-become-pass
```

Comme l'inventaire cible `localhost`, le playbook s'exécute sur la machine locale.

## Vérifications utiles

```bash
ansible-inventory --graph
ansible-playbook playbooks/site.yml --check --diff --ask-become-pass
curl http://localhost
systemctl status nginx
```

## Ce qu'il faut observer

1. Le premier lancement applique des changements.
2. Le second lancement doit afficher peu ou pas de changements.
3. Si tu modifies les variables dans `group_vars/all.yml`, Ansible met à jour uniquement ce qui est nécessaire.

## Exercices pour progresser

1. Change `demo_site_title` et `demo_owner`.
2. Ajoute un nouveau package dans `web_demo_packages`.
3. Crée une deuxième page avec un autre template.
4. Remplace l'inventaire local par une VM distante.
5. Découpe le rôle en plusieurs fichiers avec `include_tasks`.

## Fichiers clés

- [ansible.cfg](/root/Ansible/ansible.cfg)
- [inventory/hosts.yml](/root/Ansible/inventory/hosts.yml)
- [group_vars/all.yml](/root/Ansible/group_vars/all.yml)
- [playbooks/site.yml](/root/Ansible/playbooks/site.yml)
- [roles/web_demo/tasks/main.yml](/root/Ansible/roles/web_demo/tasks/main.yml)

## Idées pour aller plus loin

- ajouter un deuxième rôle pour sécuriser SSH
- gérer plusieurs environnements (`dev`, `prod`)
- utiliser `ansible-vault` pour des secrets
- ajouter `molecule` pour tester le rôle
