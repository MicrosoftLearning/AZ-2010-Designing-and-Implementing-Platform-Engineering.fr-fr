---
lab:
  title: "Implémentation de la surveillance en temps réel avec Azure\_Monitor"
  module: Observability and Continuous Improvement
---

# Labo 02 : implémentation de la surveillance en temps réel avec Azure Monitor

## Durée estimée : 20 minutes

## Prérequis

- Abonnement Azure : si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/).
- Connaissance générale des concepts de surveillance et d’observabilité.
- Connaissance générale des services Azure.

## Objectifs

- Créer un exemple d’application web.
- Activer et configurer Azure Monitor et Application Insights.
- Créer des tableaux de bord personnalisés pour visualiser les indicateurs de performance de l’application.
- Configurer des alertes pour informer les parties prenantes des anomalies de performances.
- Analyser les données de performances pour obtenir des améliorations potentielles.

## Exercice 1 : créer un exemple d’application web

En tant qu’ingénieur de plateforme, vous devez vous assurer que les applications s’exécutant sur votre plateforme sont observables et surveillées en permanence. Dans ce labo, vous allez créer un exemple d’application web sur Azure, configurer Azure Monitor et Application Insights, configurer des tableaux de bord personnalisés et créer des alertes pour suivre les performances de l’application.

### Tâche 1 : créer une application web Azure

1. Ouvrez le portail Azure.
1. Dans la barre de recherche, tapez App Services et sélectionnez ce choix.
1. Cliquez sur +Créer, puis sélectionnez Application web.
1. Sous l’onglet Informations de base :
   - Abonnement: Sélectionnez votre abonnement Azure.
   - Groupe de ressources : cliquez sur Créer, entrez **`monitoringlab-rg`**, puis cliquez sur OK.
   - Nom : entrez un nom unique, par exemple **`monitoringlab-webapp`**.
   - Publier : sélectionnez Code.
   - Pour la pile d’exécution, choisissez .NET 8 (LTS).
   - Région : sélectionnez une région proche de vous.
1. Cliquez sur Examiner et créer, puis sur Créer.
1. Attendez la fin du déploiement, puis cliquez sur Accéder à la ressource.
1. Sous l’onglet Vue d’ensemble, cliquez sur l’URL pour vérifier que l’application web est en cours d’exécution.

### Tâche 2 : vérifier Application Insights

1. Dans la ressource Application web, faites défiler jusqu’à la section Surveillance.
1. Cliquez sur Application Insights.
1. Application Insights est déjà activé pour cette application web. Cliquez sur le lien pour ouvrir la ressource Application Insights.
1. Dans la ressource Application Insights, cliquez dans le tableau de bord d’application pour afficher les données de performances fournies par le tableau de bord par défaut.

## Exercice 2 : configurer Azure Monitor et les tableaux de bord

### Tâche 1 : accéder à Azure Monitor

1. Sur le portail Azure, recherchez Monitor et cliquez dessus.
1. Dans le volet gauche, cliquez sur Métriques.
1. Dans la section Étendue, sélectionnez l’application web sous l’abonnement et le groupe de ressources où vous avez déployé l’application web.
1. Cliquez sur Appliquer et observez les métriques disponibles pour l’application web.

### Tâche 2 : ajouter des métriques clés au tableau de bord

1. Dans la section Étendue, sélectionnez votre application web (monitoringlab-webapp).
1. Sous Métrique, choisissez Temps de réponse.
1. Définissez l’agrégation sur Moyenne, puis cliquez sur +Ajouter une métrique.
1. Répétez la procédure pour les métriques supplémentaires.
   - Temps processeur (nombre)
   - Demandes (moyenne)
1. Cliquez sur le tableau de bord et épinglez-le.
1. Sélectionnez Type Partagé et sélectionnez l’abonnement et le tableau de bord monitoringlab-webapp.
1. Cliquez sur Épingler au tableau de bord.
1. Cliquez sur l’icône du tableau de bord dans le volet gauche pour afficher le tableau de bord.
1. Vérifiez que les métriques sont affichées sur le tableau de bord et mises à jour en temps réel.

## Exercice 3 : créer des alertes

### Tâche 1 : définir des conditions et des actions d’alerte

1. Dans Azure Monitor, cliquez sur Alertes.
1. Sélectionnez +Créer, puis Règle d’alerte.
1. Sous Étendue, sélectionnez votre application web (monitoringlab-webapp), puis cliquez sur Appliquer.
1. Sous Condition, cliquez dans le champ Nom du signal et sélectionnez Temps de réponse.
1. Configurez la règle d’alerte :
   - Type de seuil : dynamique
   - Type d’agrégation : Moyenne
   - La valeur est : supérieure à ou inférieure à
   - Sensibilité du seuil : élevée
   - Quand évaluer : vérifiez toutes les minutes et contrôlez dans 5 minutes.
1. Sous Actions, sélectionnez Utiliser des actions rapides.
1. Entrez :
   - Nom du groupe d’actions : WebAppMonitoringAlerts
   - Nom complet : WebAlert
   - E-mail : entrez votre adresse e-mail.
1. Cliquez sur Enregistrer.
1. Cliquez sur Suivant : Détails.
1. Entrez un nom `WebAppResponseTimeAlert` et sélectionnez un niveau de gravité détaillé.
1. Cliquez sur Vérifier + créer, puis sur Créer.

   > **Note :** votre règle d’alerte est maintenant créée et déclenche une notification par e-mail lorsque le temps de réponse dépasse le seuil. Vous pouvez forcer le déclenchement de l’alerte en envoyant un grand nombre de requêtes à l’application web. Par exemple, vous pouvez utiliser des tests de charge Azure ou un outil comme Apache JMeter.

1. Retournez à Azure Monitor > Alertes.
1. Cliquez sur Règles d’alerte pour voir la règle d’alerte que vous avez créée.

## Exercice 4 : analyser les données de performances

### Tâche 1 : passer en revue les métriques collectées

1. Dans le portail Azure, sélectionnez Application Insights.
1. Cliquez sur le tableau de bord d’application.
1. Cliquez sur la vignette Performances pour analyser les temps de réponse et du serveur et la charge. Vous pouvez également afficher le nombre de demandes et les demandes ayant échoué.
1. Cliquez sur Analyser avec les classeurs, puis sélectionnez Analyse du compteur de performances.
1. Cliquez sur Classeurs.
1. Sélectionnez l’analyse des performances du classeur sous Performances.
1. Vous pouvez voir les données de performances de l’application web.

> **Note :** vous pouvez personnaliser le classeur pour inclure des métriques et des filtres supplémentaires.
