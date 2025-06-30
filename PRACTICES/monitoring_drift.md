### Détection Avancée de la Dérive avec Prometheus, Grafana & Loki

La première étape du cycle ConCon, "Détecter", peut être grandement améliorée en passant d'une détection manuelle (`terraform plan`) ou périodique (`cron`) à un système de monitoring en temps réel. Cette approche transforme la détection de dérive en une pratique de sécurité et d'observabilité continue.

L'objectif est de capturer les **événements de modification** (API calls) qui ne proviennent pas de vos pipelines d'automatisation approuvés.

#### Stratégie Générale

1. **Centraliser les journaux d'audit** : Collecter les logs d'audit de votre infrastructure (Kubernetes API, AWS CloudTrail, etc.) dans Loki.

2. **Transformer les logs en métriques** : Utiliser le langage de requête de Loki (LogQL) pour extraire des informations des logs et les exposer comme des métriques Prometheus.

3. **Alerter sur les métriques** : Configurer Prometheus Alertmanager pour déclencher des alertes lorsque des modifications manuelles ou suspectes sont détectées.

4. **Visualiser la dérive** : Créer un dashboard Grafana pour visualiser les tendances, investiguer les événements et identifier les sources de dérive.

#### Mise en œuvre pour Kubernetes

1. **Activer les Audit Logs de l'API Server** : C'est la source de vérité pour toute modification sur le cluster. Configurez votre API Server avec une politique d'audit qui enregistre au minimum les requêtes de modification (`create`, `update`, `patch`, `delete`).

2. **Ingérer les logs avec Promtail/Loki** : Déployez Promtail (l'agent de Loki) pour qu'il lise les fichiers d'audit logs et les envoie à Loki. Une étape cruciale est le **parsing** pour extraire les labels pertinents :

   * `user`: Le nom de l'utilisateur ou du service account.

   * `verb`: L'action effectuée (`create`, `update`, etc.).

   * `kind`: Le type de ressource (`Deployment`, `ConfigMap`, etc.).

   * `namespace`: Le namespace de la ressource.

   * `name`: Le nom de la ressource.

3. **Créer des Métriques avec LogQL** : Dans la configuration de Loki, utilisez la fonctionnalité `metrics` pour créer un compteur Prometheus. Le secret est de **filtrer le bruit** pour isoler les actions manuelles.

   _Exemple de requête LogQL pour une métrique :_

   ```
   sum(rate({job="kubernetes/audit"} | json | verb=~"create|update|patch|delete" | user!~"system:serviceaccount:.*|cicd-service-account" [1m])) by (user, verb, kind, namespace)
   ```

   Cette requête :

   * Sélectionne les logs du job d'audit.

   * Parse le JSON.

   * Filtre sur les verbes de modification.

   * **Exclut (`!~`)** les service accounts connus (ceux de `kube-system` et votre service account de CI/CD).

   * Crée une métrique `..._total` avec les labels `user`, `verb`, `kind`, `namespace`.

4. **Configurer l'Alerte Prometheus** : Prometheus scrape cette nouvelle métrique depuis Loki. Vous pouvez alors créer une règle d'alerte.

   _Exemple de règle d'alerte :_

   ```
   - alert: ManualKubernetesChangeDetected
     expr: increase(kubernetes_audit_manual_changes_total[5m]) > 0
     for: 1m
     labels:
       severity: warning
     annotations:
       summary: "Modification manuelle détectée sur le cluster Kubernetes"
       description: "L'utilisateur '{{ $labels.user }}' a effectué l'action '{{ $labels.verb }}' sur la ressource '{{ $labels.kind }}' dans le namespace '{{ $labels.namespace }}'."
   ```

   Cette alerte se déclenchera dès qu'une nouvelle modification manuelle est détectée et enverra une notification (Slack, PagerDuty...) avec tous les détails.

5. **Construire le Dashboard Grafana "ConCon"** :

   * **Panneau d'Alertes** : Affiche les alertes `ManualKubernetesChangeDetected` actives.

   * **Tableau des Événements** : Une table alimentée par une requête Loki, montrant les derniers logs d'audit manuels, permettant de voir en un coup d'œil "qui a fait quoi".

   * **Graphique de Tendance** : Un graphique montrant le nombre de modifications manuelles par heure/jour, pour identifier les tendances.

   * **Top 5 des "Drifters"** : Des diagrammes montrant les utilisateurs et les types de ressources les plus fréquemment modifiés manuellement.

En suivant ces étapes, vous transformez l'étape "Détecter" en un système d'observabilité puissant, rendant l'invisible visible et engageant proactivement le cycle ConCon dès qu'une dérive se produit.
