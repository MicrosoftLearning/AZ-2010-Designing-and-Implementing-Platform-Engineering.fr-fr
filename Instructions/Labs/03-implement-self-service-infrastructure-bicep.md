---
lab:
  title: Implémentation d’une infrastructure en libre service avec Bicep
  module: Strategic Platform Road Mapping
---

# Lab 03 : implémentation d’une infrastructure en libre service avec Bicep

## Durée estimée : 20 minutes

## Prérequis

- Abonnement Azure : si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/).
- Connaissance générale d’Azure services et d’Azure CLI.
- Visual Studio Code installé avec l’extension Bicep.
- Azure CLI installé et configuré sur votre ordinateur local.
- Connaissance des concepts d’infrastructure en tant que code (IaC).

## Objectifs

- Configurer votre environnement Bicep.
- Créer un modèle Bicep pour définir des ressources Azure.
- Déployer un Azure App Service avec un backend SQL Database.
- Appliquer la gouvernance à l’aide de balises et de stratégies.
- Implémenter la mise à l’échelle automatique avec Bicep.

## Exercice 1 : créer un modèle Bicep pour déployer des ressources Azure

Dans un environnement d’ingénierie de plateforme, les développeurs ont besoin d’un moyen de provisionner l’infrastructure de manière cohérente et efficace. Ce labo vous guidera dans l’utilisation de Bicep, un outil d’infrastructure en tant que code (IaC), pour déployer et gérer des ressources Azure en libre service. Vous allez créer un modèle Bicep pour provisionner un groupe de ressources, un Azure App Service, une base de données Azure SQL et un compte de stockage tout en appliquant la gouvernance avec le balisage et les stratégies.

### Tâche 1 : installer la CLI Bicep

1. Ouvrez votre terminal local.
1. Pour vérifier que Bicep est installé, exécutez :

   ```bash
   az bicep version
   ```

   Si Bicep n’est pas installé, installez-le avec :

   ```bash
   az bicep install
   ```

1. Confirmez l’installation en exécutant :

   ```bash
   az bicep version
   ```

   Vous devez voir la sortie CLI Bicep version X.Y.Z.

   > **NOTE :** assurez-vous que [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) est installé.

### Tâche 2 : créer un modèle Bicep

1. Dans votre terminal local, créez un fichier :

   ```bash
   code main.bicep
   ```

1. Copiez et collez le code Bicep suivant dans le fichier :

   ```bicep
   param location string = 'eastus2'
   param appServicePlanName string = 'bicepAppPlan'
   param webAppName string = 'bicep-webapp'
   param storageAccountName string = 'biceplabstorage'
   param sqlServerName string = 'bicep-sqlserver'
   param sqlDatabaseName string = 'bicepdb'
   param sqlAdminUser string
   @secure()
   param sqlAdminPassword string

   targetScope = 'resourceGroup'

   resource storage 'Microsoft.Storage/storageAccounts@2021-09-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'StorageV2'
   }

   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'F1'
       tier: 'Free'
     }
   }

   resource webApp 'Microsoft.Web/sites@2021-02-01' = {
     name: webAppName
     location: location
     properties: {
       serverFarmId: appPlan.id
     }
   }

   resource sqlServer 'Microsoft.Sql/servers@2022-05-01-preview' = {
     name: sqlServerName
     location: location
     properties: {
       administratorLogin: sqlAdminUser
       administratorLoginPassword: sqlAdminPassword
       version: '12.0'
     }
   }

   resource sqlDb 'Microsoft.Sql/servers/databases@2022-05-01-preview' = {
     name: sqlDatabaseName
     location: location
     parent: sqlServer
     properties: {
       collation: 'SQL_Latin1_General_CP1_CI_AS'
       maxSizeBytes: 2147483648
     }
   }
   ```

   > **NOTE :** vous pouvez installer l’extension Bicep dans Visual Studio Code pour obtenir la mise en surbrillance de la syntaxe et IntelliSense pour les fichiers Bicep.

   > **IMPORTANT :** vérifiez que storageAccountName, webAppName et sqlServerName sont globalement uniques. Si le déploiement échoue en raison de conflits de noms, modifiez les noms en conséquence.

   > **NOTE :** si vous rencontrez une erreur liée à la région, essayez de déployer dans une autre région en modifiant la valeur du paramètre `location`.

1. Enregistrez et fermez le fichier.

### Tâche 3 : déployer le modèle avec Azure CLI

1. Exécutez la commande suivante pour créer le groupe de ressources Azure :

   ```bash
   az group create --name BicepLab-RG --location centralus

   ```

   > **NOTE :** vous devrez peut-être vous connecter à votre compte Azure si vous n’êtes pas déjà authentifié. Pour obtenir des instructions détaillées, consultez [S’authentifier auprès d’Azure à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli).

2. Déployez le modèle dans le groupe de ressources :

   ```bash
   az deployment group create --resource-group BicepLab-RG --template-file main.bicep --parameters sqlAdminUser='azureuser' sqlAdminPassword='YourSecurePassword123!'

   ```

   > **NOTE :** remplacez YourSecurePassword123! par un mot de passe fort et azureuser par un nom d’utilisateur valide.

3. Attendez la fin du déploiement. Un message de confirmation doit s’afficher.
4. Dans le Portail Azure, accédez à Groupes de ressources.
5. Recherchez BicepLab-RG, puis cliquez dessus.
6. Vérifiez que la ressource a été correctement créée.

Vous avez déployé avec succès un Azure App Service avec un back-end SQL Database à l’aide d’un modèle Bicep. Vous pouvez désormais gérer votre infrastructure en tant que code et déployer des ressources de manière cohérente et efficace. L’étape suivante consiste à intégrer ce modèle dans votre pipeline CI/CD pour les déploiements automatisés.

## Exercice 2 : appliquer la gouvernance avec des balises et des stratégies

Dans un environnement d’infrastructure en libre service, il est essentiel d’appliquer la gouvernance pour garantir la conformité et le contrôle des coûts. Les balises et les stratégies sont deux mécanismes clés pour y parvenir.

### Tâche 1 : ajouter des balises aux ressources

1. Exécutez la commande suivante pour ajouter une balise au groupe de ressources :

   ```bash
   az group update --name BicepLab-RG --set tags.Owner='PlatformEngineering'
   ```

1. Vérifiez que la balise a été ajoutée en exécutant :

   ```bash
   az group show --name BicepLab-RG --query tags
   ```

### Tâche 2 : créer une stratégie pour appliquer le balisage

1. Créez un fichier de stratégie JSON :

   ```bash
   code tagging-policy.json
   ```

1. Copiez et collez la définition de stratégie suivante dans le fichier :

   ```json
   {
     "if": {
       "not": {
         "field": "tags[Owner]",
         "exists": "true"
       }
     },
     "then": {
       "effect": "deny"
     }
   }
   ```

1. Créez la définition de stratégie à l’aide du fichierJSON :

   ```bash
    az policy definition create --name 'tagging-policy' --display-name 'Enforce tagging' --rules @tagging-policy.json --mode All
   ```

1. Attribuez la stratégie au groupe de ressources :

   ```bash
   az policy assignment create --name 'tagging-policy-assignment' --display-name 'Enforce tagging' --policy "/subscriptions/<subscription-id>/providers/Microsoft.Authorization/policyDefinitions/tagging-policy" --scope "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG"

   ```

   > **IMPORTANT :** remplacez les deux <subscription-id> par votre ID d’abonnement Azure.

1. Vérifiez que la stratégie a été affectée en exécutant :

   ```bash
   az policy assignment list --output table
   ```

### Tâche 3 : tester l’application de la stratégie

1. Pour supprimer la balise du groupe de ressources, exécutez la commande suivante :

   ```bash
   az tag update --resource-id /subscriptions/<subscription-id>/resourceGroups/BicepLab-RG --operation delete --tags Owner
   ```

   > **IMPORTANT :** remplacez <subscription-id> par votre ID d’abonnement Azure.

1. Un message d’erreur doit s’afficher, indiquant que l’opération a été refusée en raison de l’application de la stratégie. La balise ne doit pas être supprimée. Cette commande confirme que la stratégie fonctionne comme prévu. Pour résoudre l’erreur, supprimez temporairement l’attribution de stratégie et réexécutez la commande.

   ```bash
   az policy exemption create --name 'tagging-policy-exemption' --policy-assignment "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG/providers/Microsoft.Authorization/policyAssignments/tagging-policy-assignment" --scope "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG" --display-name "Temporary exemption to remove tag" --exemption-category Waiver
   ```

   > **IMPORTANT :** remplacez les deux <subscription-id> par votre ID d’abonnement Azure.

Vous avez appliqué la gouvernance à l’aide de balises et de stratégies. Les développeurs seront désormais tenus de baliser les ressources avec la balise Propriétaire, en garantissant la propriété et la responsabilité appropriées. Vous pouvez créer des stratégies supplémentaires pour appliquer d’autres exigences de gouvernance, telles que des conventions d’affectation de noms de ressources, des types de ressources et des emplacements de ressources.

## Exercice 3 : implémenter la mise à l’échelle automatique avec Bicep

Dans un environnement d’ingénierie de plateforme, il est essentiel de s’assurer que les applications peuvent être mises à l’échelle efficacement. La mise à l’échelle automatique améliore les performances, optimise l’utilisation des ressources et réduit les coûts. Dans cet exercice, vous allez implémenter la mise à l’échelle automatique pour un Azure App Service à l’aide de Bicep.

### Tâche 1 : définir une stratégie de mise à l’échelle automatique dans Bicep

1. Ouvrez le fichier main.bicep et recherchez la section Plan App Service existante :

   ```bicep
   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'F1'
       tier: 'Free'
     }
   }
   ```

1. Modifiez la ressource appPlan de sorte à utiliser le SKU S1 (qui prend en charge la mise à l’échelle automatique) :

   ```bicep
   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'S1'
       tier: 'Standard'
     }
     properties: {
       perSiteScaling: false
       maximumElasticWorkerCount: 10
     }
   }
   ```

1. Immédiatement après cette ressource, ajoutez la configuration autoscaleSetting :

   ```bicep
   resource autoscaleSetting 'Microsoft.Insights/autoscaleSettings@2024-01-01-preview' = {
   name: 'autoscale-rule'
   location: location
   properties: {
    profiles: [
      {
        name: 'defaultProfile'
        capacity: {
          minimum: '1'
          maximum: '5'
          default: '1'
        }
        rules: [
          {
            metricTrigger: {
              metricName: 'CpuPercentage'
              metricResourceUri: resourceId('Microsoft.Web/serverfarms', appServicePlanName)
              operator: 'GreaterThan'
              threshold: 75
              timeAggregation: 'Average'
              statistic: 'Average'
              timeWindow: 'PT5M'
              timeGrain: 'PT1M'
            }
            scaleAction: {
              direction: 'Increase'
              type: 'ChangeCount'
              value: '1'
              cooldown: 'PT1M'
            }
          }
        ]
      }
    ]
   }
   }
   ```

1. Enregistrez le fichier.
1. Validez le fichier Bicep mis à jour :

   ```bash
   az bicep build --file main.bicep
   ```

1. Déployez le modèle mis à jour :

   ```bash
   az deployment group create --resource-group BicepLab-RG --template-file main.bicep --parameters sqlAdminUser='azureuser' sqlAdminPassword='YourSecurePassword123!'

   ```

1. Vérifiez que la stratégie de mise à l’échelle automatique a été appliquée au plan App Service. Accédez au Portail Azure, accédez au plan App Service, sélectionnez le panneau Scale-out (plan App Service) et vérifiez que la règle de mise à l’échelle automatique est configurée.

   > **NOTE :** si vous souhaitez tester le comportement de mise à l’échelle automatique, vous pouvez simuler une utilisation élevée de l’UC sur App Service en exécutant un test de charge ou en générant du trafic. App Service doit effectuer un scale-out automatiquement en fonction de la règle définie.

Vous avez correctement implémenté la mise à l’échelle automatique d’azure App Service à l’aide de Bicep. Cela garantit que vos applications peuvent gérer efficacement le trafic et la demande accrus, améliorer les performances et réduire les coûts.
