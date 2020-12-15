![dp100 image](img/dp100.png)

# brief-formateurs

Superviser la gestion des coûts de sa promotion dans le cadre de la préparation à la dp100.

La documentation pour ce projet se trouve [ici](https://github.com/jtobelem-simplon/dp100-brief-init/blob/master/doc/dp100.pdf?raw=true).


## Configuration des utilisateurs
- une souscription dont le nom est celui de la promotion : ecole-ia-XXX
- un groupe (dans azure active directory) qui contient les membres (rôle:utilisateur) de la promo : ecole-ia-XXX-pXX (le numéro de promo)
- un groupe (dans azure AD) qui contient les formateurs (rôle:utilsateur) : ecole-ia-XXX-pXX-formateurs
- depuis le contrôle des accès (IAM), ajouter les groupes (rôle:contributeur) dans la souscription

## Configuration des ressources
- un groupe de ressource pour toute la promo par projet (par exemple prepadp100) afin de supprimer le groupe après passage de la certif
- un workspace pour la promo
- envoyer une [demande](https://docs.microsoft.com/fr-fr/azure/azure-portal/supportability/regional-quota-requests) (par mail + auto) au support pour augmenter le quota par défaut de 10 coeurs
- une instance de clacul de type ds11-v2 par personne
- un cluster de calcul (min = 0, max = 4) pour toute la promo

## Création d'alerte
- une alerte tous les 100€ pour contrôler que le budget mensuel ne déborde pas (il devrait être d'environ 400€ pour la dp100) 


## Script d'arret des machines pour une souscription
- à lancer de temps en temps (en rappelant à l'ordre les personnes qui n'aurait pas éteint leur machine)

```python
import os
from azureml.core.compute import ComputeTarget, AmlCompute, ComputeInstance
from azureml.core.compute_target import ComputeTargetException
from azureml.core import Workspace

subscription_id = os.environ.get("SUBSCRIPTION_ID", "XXX") #ID soubscription Azure
resource_group = os.environ.get("RESOURCE_GROUP", "dp100-resources") #Resource group

for ws_name in Workspace.list(subscription_id, resource_group=resource_group):
    ws = Workspace(subscription_id=subscription_id, resource_group=resource_group, workspace_name=ws_name)

    for compute in ComputeTarget.list(ws):
        print(">>>>", compute.name, "in", ws_name)
        if type(compute) is ComputeInstance and compute.get_status().state != 'Stopped':
            print('try to stop compute', compute.name)
            compute.stop(show_output=True)
```

## Identification des éléments qui pesent le plus sur le coût total
- afficher le journal d'activité de la souscription
![activity azure](img/azure-activity.png)

- télécharger au format csv pour identifier les ressources les plus utilisées
![activity](img/activity.png)


## Gestion des policies

[Azure Policy](https://docs.microsoft.com/fr-fr/azure/governance/policy/overview#azure-policy-objects) aide à appliquer les normes organisationnelles et à évaluer la conformité à l’échelle. Avec son tableau de bord de conformité, il fournit une vue agrégée permettant d’évaluer l’état général de l’environnement, avec la possibilité d’explorer au niveau de chaque ressource et stratégie. Il vous aide également à mettre vos ressources en conformité par le biais de la correction en bloc pour les ressources existantes et de la correction automatique pour les nouvelles ressources. 

Dans Azure Policy, nous proposons plusieurs stratégies intégrées qui sont disponibles par défaut. Par exemple : 
 
- Références SKU de compte de stockage autorisées (Refuser) : Détermine si un compte de stockage en cours de déploiement se trouve dans un ensemble de tailles de référence SKU. Son effet consiste à refuser tous les comptes de stockage dont la taille ne fait pas partie de l’ensemble de tailles de référence SKU définies. 
- Type de ressource autorisé (Refuser) : Définit les types de ressources que vous pouvez déployer. Son effet consiste à refuser toutes les ressources qui ne font pas partie de cette liste définie. 
- Emplacements autorisés (Refuser) : Restreint les emplacements disponibles pour les nouvelles ressources. Son effet permet d’appliquer vos exigences de conformité géographique. 
- Références SKU de machine virtuelle autorisées (Refuser) : Spécifie un ensemble de références SKU de machine virtuelle que vous pouvez déployer. 
- Ajouter une étiquette aux ressources (Modifier) : Applique une balise requise et sa valeur par défaut si elle n’est pas spécifiée par la requête de déploiement. 
- Types de ressources non autorisés (Refuser) : Empêche une liste de types de ressources d’être déployés. 

Pour implémenter ces définitions de stratégie (définitions intégrées et personnalisées), vous devez les affecter. Vous pouvez affecter l’une de ces stratégies par le biais du portail Azure, de PowerShell ou d’Azure CLI. Voici un tutoriel pour commencer: https://docs.microsoft.com/fr-fr/azure/governance/policy/tutorials/create-and-manage#implement-a-new-custom-policy 

## briefs apprenants

- [installer l'environnement azure pour suivre le parcours d'apprentissage MS](https://github.com/jtobelem-simplon/dp100-brief-init.git)
- [installer l'environnement local pour suivre le parcours d'apprentissage MS](https://github.com/jtobelem-simplon/dp100-brief-init-expert.git)
- [cadre pour transposer les exercices du parcours d'apprentissage MS sur l'exemple du titanic](https://github.com/jtobelem-simplon/dp100-brief-titanic.git)