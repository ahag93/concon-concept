### ConCon avec Ansible

Avec Ansible, qui est de nature procédurale et sans état, la pratique ConCon s'apparente à une ingénierie inverse disciplinée.

#### Le Cycle ConCon pour Ansible

1. **Détecter** : Lancez un playbook en mode "check" et "diff" : `ansible-playbook my_playbook.yml --check --diff`. Ansible vous montrera en jaune tous les changements qu'il _voudrait_ appliquer. Ces "diffs" sont vos dérives.

2. **Exporter (Inspecter)** : Il n'y a pas de commande d'export direct. Vous devez utiliser des modules Ansible pour lire l'état actuel de la configuration sur le nœud cible.

   * Utilisez le module `setup` pour collecter un maximum de "facts".

   * Utilisez le module `slurp` pour récupérer le contenu d'un fichier de configuration.

   * Utilisez `command` ou `shell` pour exécuter une commande qui affiche un paramètre système.

3. **Transformer** : Analysez les configurations récupérées et mettez à jour vos artefacts Ansible pour qu'ils correspondent à la réalité observée :

   * Modifiez les variables dans `group_vars/` ou `host_vars/`.

   * Ajustez les templates Jinja2 (`.j2`) pour qu'ils génèrent un fichier de configuration identique à celui récupéré.

4. **Intégrer** : Ré-exécutez le playbook en mode `--check --diff`. S'il ne montre plus de changements ("ok" ou "skipped" mais pas "changed"), votre code est synchronisé avec la réalité. Commitez vos modifications.
