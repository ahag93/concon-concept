### ConCon avec Terraform

Terraform, étant basé sur un fichier d'état, possède un mécanisme d'importation natif qui est au cœur de la pratique ConCon pour l'IaC.

#### Le Cycle ConCon pour Terraform

1. **Détecter** : Une commande `terraform plan` sur une infrastructure existante révèle une dérive ("changes outside of Terraform"). Ou, vous savez qu'une ressource a été créée manuellement (ex: un bucket S3) et n'est pas du tout gérée par Terraform.

2. **Exporter (Importer)** : On utilise `terraform import`. Cette commande ne fait que lier une ressource existante à une adresse dans le code Terraform, en mettant à jour le fichier d'état (`terraform.tfstate`). **Elle ne génère pas le code HCL.**

   ```
   # Syntaxe : terraform import <ADRESSE_CODE> <ID_REEL>
   terraform import aws_s3_bucket.data_archive my-manual-s3-bucket-123
   ```

3. **Transformer** : C'est ici l'étape manuelle (ou semi-automatisée) :

   * **Écrivez le code** : Vous devez créer le bloc `resource "aws_s3_bucket" "data_archive" {}` dans vos fichiers `.tf`.

   * **Récupérez les attributs** : Vous devez consulter la console de votre fournisseur cloud ou utiliser son CLI pour connaître les attributs de la ressource (policy, versioning, etc.) et les ajouter à votre code HCL.

   * **Automatisation** : Des outils comme `terraformer` ou `aztfy` peuvent automatiser cette génération de code, s'intégrant parfaitement dans une démarche ConCon avancée.

4. **Intégrer** : Validez votre travail en lançant `terraform plan`. Si le résultat est **"No changes. Your infrastructure matches the configuration."**, la convergence est réussie. Vous pouvez alors commiter votre code en toute confiance.
