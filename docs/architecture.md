# Architecture du projet

Ce document décrit l'architecture fonctionnelle du dépôt et le chemin suivi par Ansible pendant une exécution du playbook principal.

## Vue d'ensemble

Le projet repose sur une architecture simple :

- une machine de contrôle exécute Ansible
- l'inventaire cible `localhost`
- un playbook unique appelle un rôle unique
- les variables globales alimentent le rôle
- le rôle installe les paquets, déploie la page et gère le service `nginx`

Ce modèle est volontairement minimal pour rendre visibles les mécanismes fondamentaux d'Ansible sans dépendre d'une infrastructure distante.

## Schéma d'architecture

```mermaid
flowchart TD
    A[Machine de contrôle<br/>Ansible CLI] --> B[ansible.cfg]
    B --> C[inventory/hosts.yml]
    C --> D[Playbook<br/>playbooks/site.yml]
    E[group_vars/all.yml] --> F[Role<br/>web_demo]
    G[defaults/main.yml] --> F
    D --> F

    F --> H[Assert OS family]
    F --> I[APT cache update]
    F --> J[Package installation]
    F --> K[Template deployment]
    F --> L[Service management]
    K -. notify .-> M[Handler restart nginx]

    H --> N[localhost]
    I --> N
    J --> N
    K --> N
    L --> N
    M --> N

    N --> O[/var/www/html/index.html]
    N --> P[Service nginx]
    P --> Q[HTTP sur localhost:80]
```

## Composants

### 1. Configuration Ansible

[ansible.cfg](/root/Ansible/ansible.cfg:1) centralise la configuration locale du projet.

Points importants :

- `inventory = inventory/hosts.yml`
- `roles_path = roles`
- `stdout_callback = yaml`
- `become = True`

Cette configuration réduit la quantité d'options à passer en ligne de commande et rend le dépôt plus autonome.

### 2. Inventaire

[inventory/hosts.yml](/root/Ansible/inventory/hosts.yml:1) définit un groupe `local` contenant `localhost` avec `ansible_connection: local`.

Conséquence :

- la machine de contrôle et la machine gérée sont ici la même machine
- aucun accès SSH distant n'est nécessaire pour ce scénario

### 3. Playbook principal

[playbooks/site.yml](/root/Ansible/playbooks/site.yml:1) est le point d'entrée.

Responsabilités :

- cibler le groupe `local`
- activer l'élévation de privilèges
- appeler le rôle `web_demo`

### 4. Variables globales

[group_vars/all.yml](/root/Ansible/group_vars/all.yml:1) contient les variables métier du laboratoire :

- titre de page
- message affiché
- propriétaire du labo
- liste des paquets à installer

Cette séparation entre logique et données suit une bonne pratique classique d'Ansible : garder les valeurs personnalisables hors des tâches.

### 5. Rôle `web_demo`

Le rôle se trouve dans `roles/web_demo/` et suit la structure standard documentée par Ansible.

Fichiers clés :

- [roles/web_demo/tasks/main.yml](/root/Ansible/roles/web_demo/tasks/main.yml:1)
- [roles/web_demo/defaults/main.yml](/root/Ansible/roles/web_demo/defaults/main.yml:1)
- [roles/web_demo/handlers/main.yml](/root/Ansible/roles/web_demo/handlers/main.yml:1)
- [roles/web_demo/templates/index.html.j2](/root/Ansible/roles/web_demo/templates/index.html.j2:1)

## Flux d'exécution

Lorsqu'on lance :

```bash
ansible-playbook playbooks/site.yml
```

Ansible suit ce chemin :

1. charge `ansible.cfg`
2. charge l'inventaire `inventory/hosts.yml`
3. sélectionne le groupe `local`
4. exécute le rôle `web_demo`
5. vérifie que `ansible_os_family == "Debian"`
6. met à jour le cache `apt`
7. installe les paquets demandés
8. génère le template HTML dans `/var/www/html/index.html`
9. redémarre `nginx` seulement si le template a changé
10. garantit que le service `nginx` est activé et démarré

## Idempotence

Le projet illustre un principe central d'Ansible : l'idempotence.

Dans ce dépôt :

- l'installation des paquets ne réinstalle pas ce qui est déjà présent
- le handler ne redémarre `nginx` que si le template change
- une seconde exécution doit produire peu ou pas de changements

Cela rend le playbook prévisible et sûr à rejouer.

## Points d'extension

Le dépôt peut évoluer facilement vers une architecture plus riche :

- ajouter des groupes `dev`, `test`, `prod` dans l'inventaire
- créer un second rôle pour la sécurité ou la supervision
- déplacer certaines variables vers des fichiers par environnement
- ajouter `ansible-vault` pour les secrets
- appliquer le rôle à une VM distante ou à plusieurs hôtes

## Références

- Ansible community documentation: https://docs.ansible.com/
- Introduction to Ansible: https://docs.ansible.com/projects/ansible/latest/getting_started/introduction.html
- Using roles in playbooks: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_reuse_roles.html
