### ConCon avec Helm

Appliquer la pratique ConCon avec Helm consiste à transformer des ressources Kubernetes existantes, souvent créées manuellement, en un Chart Helm structuré et maintenable. Cela permet de passer d'une gestion impérative (`kubectl apply/edit`) à une gestion déclarative et versionnée.

L'outil principal pour la phase d'**Exportation** est [`helm-dump`](https://github.com/redhat-developer/helm-dump) 

#### Le Cycle ConCon pour Helm

1. **Détecter** : Vous constatez qu'un déploiement dans le namespace `production` a été modifié manuellement (`kubectl scale`, `kubectl edit`, etc.) et ne correspond plus au Chart Helm dans votre dépôt Git. L'outil `helm diff` peut être utilisé pour visualiser précisément cette dérive avant même d'appliquer un `helm upgrade`.

2. **Exporter** : Utilisez `helm-dump` pour capturer l'état de toutes les ressources du namespace :

   ```
   helm dump -n production --chart-name my-app-chart --output-dir ./charts
   ```

   Cette commande génère un répertoire `charts/my-app-chart` contenant les manifestes YAML bruts de vos ressources.

3. **Transformer** : C'est l'étape la plus importante. Ouvrez le Chart généré et :

   * **Nettoyez les manifestes** en supprimant les champs de statut ou les métadonnées inutiles.

   * **Identifiez les variables** : le nombre de réplicas, la version de l'image, les noms de ConfigMaps, les ressources CPU/mémoire, etc.

   * **Paramétrez les templates** en remplaçant les valeurs fixes par des expressions Helm (`{{ .Values.replicaCount }}`).

   * **Remplissez `values.yaml`** avec les valeurs par défaut que vous venez d'extraire, en ajoutant des commentaires explicatifs.

4. **Intégrer** : Une fois votre Chart Helm proprement "templaté", ouvrez une Pull Request pour fusionner ces changements dans votre branche principale. Le prochain déploiement de cette application devra se faire via `helm install` ou `helm upgrade`.
