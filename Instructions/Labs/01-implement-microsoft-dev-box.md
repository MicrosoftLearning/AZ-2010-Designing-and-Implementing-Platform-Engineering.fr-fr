---
lab:
  title: Implémenter Microsoft Dev Box
  module: Implement Developer Self-Service
---

# Labo 01 : implémenter Microsoft Dev Box

## Durée estimée : 60 minutes

## Prérequis

- Un locataire Microsoft Entra avec trois comptes d’utilisateur précréés (et éventuellement trois groupes Microsoft Entra précréés) représentant trois rôles différents impliqués dans des déploiements Microsoft Dev Box. Par souci de clarté, les noms d’utilisateurs et de groupes dans les instructions du labo correspondent aux informations contenues dans le tableau suivant :

| Utilisateur              | Groupe                        | Rôle                  |
| ----------------- | ---------------------------- | --------------------- |
| platformegineer01 | DevCenter_Platform_Engineers | Ingénieur plateforme     |
| devlead01         | DevCenter_Dev_Leads          | Responsable de l’équipe de développement (Development team lead) |
| devuser01         | DevCenter_Dev_Users          | Développeur             |

- Un abonnement Azure pour héberger les ressources Microsoft Dev Box associées au locataire Microsoft Entra hébergeant les comptes d’utilisateur et de groupe
- Un abonnement Microsoft Intune associé au même locataire Microsoft Entra que l’abonnement Azure
- Une licence Microsoft Intune affectée aux trois comptes d’utilisateur Microsoft Entra précréés
- Vous aurez besoin d’un compte et d’un référentiel GitHub.
  - Pour créer un compte GitHub, suivez les instructions de l’article [Créer un compte sur GitHub](https://docs.github.com/get-started/quickstart/creating-an-account-on-github).
  - Pour créer un référentiel GitHub, suivez les instructions de l’article [Création d’un référentiel](https://docs.github.com/repositories/creating-and-managing-repositories/creating-a-new-repository).
- Un dépôt GitHub créé en tant que fourche de https://github.com/microsoft/devcenter-catalog
- Un dépôt GitHub créé en tant que fourche de https://github.com/MicrosoftLearning/contoso-co-eShop

## Objectifs

À la fin de cet atelier, vous serez en mesure de :

- Implémenter un environnement Microsoft Dev Box
- Personnaliser un environnement Microsoft Dev Box

## Exercice 1 : implémenter un environnement Microsoft Dev Box

Dans cet exercice, vous allez tirer parti d’un ensemble de fonctionnalités fournies par Microsoft pour implémenter un environnement Microsoft Dev Box. Cette approche se concentre sur la réduction de l’effort impliqué dans la création d’une solution fonctionnelle en libre service de développeur.

L’exercice se compose des tâches suivantes :

- Tâche 1 : créer un centre de développement
- Tâche 2 : passer en revue les paramètres du centre de développement
- Tâche 3 : créer une définition de dev box
- Tâche 4 : créer un projet
- Tâche 5 : créer un pool de dev box
- Tâche 6 : configurer les autorisations
- Tâche 7 : évaluer une dev box

### Tâche 1 : créer un centre de développement

Dans cette tâche, vous allez créer un centre de développement Azure qui sera utilisé tout au long du premier exercice de ce labo. Un centre de développement est un service d’ingénierie de plateforme qui centralise la création et la gestion d’environnements de développement et de déploiement évolutifs préconfigurés, en optimisant la collaboration et l’utilisation des ressources pour les équipes de développement logiciel.

1. Démarrez un navigateur web et accédez au portail Azure à l’adresse `https://portal.azure.com`.
1. Lorsque vous êtes invité à vous authentifier, connectez-vous avec votre compte d’utilisateur Microsoft Entra.
1. Dans le Portail Microsoft Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **`Dev centers`**.
1. Dans la page des **centres de développement**, sélectionnez **+Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un centre de développement**, spécifiez les paramètres suivants et sélectionnez **Suivant : Paramètres** :

   | Paramètre                                                                 | Valeur                                                        |
   | ----------------------------------------------------------------------- | ------------------------------------------------------------ |
   | Abonnement                                                            | Nom de l’abonnement Azure que vous utilisez dans ce labo |
   | Resource group                                                          | Le nom d’un **nouveau** groupe de ressources **rg-devcenter-01**     |
   | Nom                                                                    | **devcenter-01**                                             |
   | Emplacement                                                                | **(États-Unis) USA Est**                                             |
   | Attacher un catalogue de démarrage rapide - Définitions d’environnement de déploiement Azure | Activé(e)                                                      |
   | Attacher un catalogue de démarrage rapide - Tâches de personnalisation de dev box              | Désactivé                                                     |

   > **Note :** vous allez configurer un catalogue avec des tâches de personnalisation activées dans le deuxième exercice de ce labo.

1. Sous l’onglet **Paramètres** de la page **Créer un centre de développement**, spécifiez les paramètres suivants et sélectionnez **Examiner et créer** :

   | Paramètre                                    | Valeur   |
   | ------------------------------------------ | ------- |
   | Activer les catalogues par projet                | Activé(e) |
   | Autoriser le réseau hébergé par Microsoft dans les projets | Activé(e) |
   | Autoriser l’installation de l’agent Azure Monitor    | Activé(e) |

   > **Note :** par conception, les ressources des catalogues attachés au centre de développement sont disponibles pour tous les projets qu’il contient. Le paramètre **Activer les catalogues par projet** permet également d’attacher des catalogues supplémentaires à des projets arbitrairement sélectionnés.

   > **Note :** les dev box peuvent être connectées à un réseau virtuel dans votre propre abonnement Azure ou à un abonnement Microsoft hébergé par Microsoft, selon qu’elles doivent communiquer avec les ressources de votre environnement. Si ce n’est pas nécessaire, en activant le paramètre **Autoriser le réseau hébergé par Microsoft dans les projets**, vous introduisez l’option permettant de connecter des dev box à un réseau hébergé par Microsoft, ce qui réduit efficacement la surcharge de gestion et de configuration.

   > **Note :** le paramètre **Activer l’installation de l’agent Azure Monitor** déclenche automatiquement l’installation de l’agent Azure Monitor sur toutes les dev box du centre de développement.

1. Sous l’onglet **Examiner et créer**, attendez que le processus de validation se termine, puis sélectionnez **Créer**.

   > **Note :** attendez que le projet soit provisionné. La création de projet peut durer environ 1 minute.

1. Dans la page **Déploiement terminé**, sélectionnez **Accéder à la ressource**.

### Tâche 2 : passer en revue les paramètres du centre de développement

Dans cette tâche, vous allez passer en revue le paramètre de configuration de base du centre de développement que vous avez créé dans la tâche précédente.

1. Dans le navigateur web affichant le Portail Azure, dans la page **devcenter-01**, dans le menu de navigation vertical situé à gauche, développez la section **Configuration de l’environnement** et sélectionnez **Catalogues**.
1. Dans la page **Catalogues devcenter-01 \|**, notez que le centre de développement est configuré avec le catalogue **quickstart-environment-definitions**, qui pointe vers le référentiel GitHub `https://github.com/microsoft/devcenter-catalog.git`.
1. Vérifiez que la colonne **État** contient l’entrée de **synchronisation réussie**. Si ce n’est pas le cas, utilisez la séquence d’étapes suivante pour recréer le catalogue :

   1. Cochez la case en regard de l’entrée de catalogue générée automatiquement, **quickstart-environment-definitions**, puis, dans la barre d’outils, sélectionnez **Supprimer**.
   1. Dans la page **Catalogues devcenter-01 \|**, sélectionnez **+Ajouter**.
   1. Dans le volet **Ajouter un catalogue**, dans la zone de texte **Nom**, saisissez **`quickstart-environment-definitions-fix`**. Dans la section **Emplacement du catalogue**, sélectionnez **GitHub**. Dans le **type d’authentification**, sélectionnez **Application GitHub**, laissez la case à cocher **Synchroniser automatiquement ce catalogue** activée, puis sélectionnez **Se connecter avec GitHub**.
   1. Dans la fenêtre **Se connecter avec GitHub**, entrez les informations d’identification GitHub, puis sélectionnez **Se connecter**.

      > **Note :** ces informations d’identification GitHub vous permettent d’accéder à un référentiel GitHub créé en tant que duplication de <https://github.com/microsoft/devcenter-catalog>.

   1. Lorsque vous y êtes invité, dans la fenêtre **Autoriser Microsoft DevCenter**, sélectionnez **Autoriser Microsoft DevCenter**.
   1. De retour au volet **Ajouter un catalogue**, dans la liste déroulante **Référentiel**, sélectionnez **devcenter-catalog**. Dans la liste déroulante **Branche**, acceptez l’entrée **Banche par défaut**. Dans le **chemin du dossier**, entrez**`Environment-Definitions`**, puis sélectionnez **Ajouter**.
   1. De retour à la page **Catalogues devcenter-01 \|**, vérifiez que la synchronisation se termine correctement en surveillant l’entrée dans la colonne **État**.

1. Dans la page **Catalogues devcenter-01 \|**, sélectionnez l’entrée **quickstart-environment-definitions-fix**.
1. Dans la page **quickstart-environment-definitions-fix**, passez en revue la liste des définitions d’environnement prédéfinies.

   > **Note :** chaque entrée représente une définition d’un environnement de déploiement Azure défini dans un sous-dossier respectif du dossier **Environment-Definitions** du référentiel GitHub `https://github.com/microsoft/devcenter-catalog.git`.

   > **Note :** un environnement de déploiement est une collection de ressources Azure définies dans un modèle appelé définition d’environnement. Les développeurs peuvent utiliser ces définitions pour déployer l’infrastructure qui servira à héberger leurs solutions. Pour plus d’informations sur les environnements de déploiement Azure, consultez l’article Microsoft Learn [Qu’est-ce que les environnements de déploiement Azure ?](https://learn.microsoft.com/azure/deployment-environments/overview-what-is-azure-deployment-environments)

### Tâche 3 : créer une définition de dev box

Dans cette tâche, vous allez créer une définition de dev box. Son objectif est de définir le système d’exploitation, les outils, les paramètres et les ressources qui servent de blueprint pour créer des environnements de développement cohérents et personnalisés (appelés dev box).

1. Dans le navigateur web affichant le Portail Azure, dans la page **devcenter-01**, dans le menu de navigation vertical situé à gauche, développez la section **Configuration de la dev box** et sélectionnez **Définitions de la dev box**.
1. Dans la page **Définitions de la dev box devcenter-01 \|**, sélectionnez **+Créer**.
1. Dans la fenêtre **Créer une définition de dev box**, spécifiez les paramètres suivants, puis sélectionnez **Créer** :

   | Paramètre            | Valeur                                                                                |
   | ------------------ | ------------------------------------------------------------------------------------ | ----------------------- |
   | Nom               | **devbox-definition-01**                                                             |
   | Image              | \*\*Visual Studio 2022 Enterprise sur Windows 11 Entreprise + Applications Microsoft 365 24H2 | Prise en charge de la mise en veille prolongée\*\* |
   | Version d’image      | **La plus récente**                                                                           |
   | Compute            | **8 processeurs virtuels, 32 Go de RAM**                                                                |
   | Stockage            | **SSD de 256 Go**                                                                       |
   | Activer la mise en veille prolongée | Activé(e)                                                                              |

   > **Note :** attendez la création de la définition de dev box. Cela devrait prendre moins d’une minute.

### Tâche 4 : créer un projet de centre de développement

Dans cette tâche, vous allez créer un projet de centre de développement. Un projet de centre de développement correspond généralement à un projet de développement au sein de votre organisation. Par exemple, vous pouvez créer un projet pour le développement d’une application métier, et un autre projet pour le développement du site web de l’entreprise. Tous les projets d’un centre de développement partagent les mêmes définitions de dev box, la même connexion réseau, les mêmes catalogues et les mêmes galeries de calcul. Vous pouvez envisager de créer plusieurs projets de centre de développement si vous avez plusieurs projets de développement qui ont des administrateurs de projet et des exigences d’autorisations d’accès qui diffèrent.

1. Dans le navigateur web affichant le Portail Azure, dans la page **devcenter-01**, menu de navigation vertical situé à gauche, développez la section **Gérer** et sélectionnez **Projets**.
1. Dans la page **Projets devcenter-01 \|**, sélectionnez **+Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un projet**, spécifiez les paramètres suivants et sélectionnez **Suivant : Gestion de dev box** :

   | Paramètre        | Valeur                                                        |
   | -------------- | ------------------------------------------------------------ |
   | Abonnement   | Nom de l’abonnement Azure que vous utilisez dans ce labo |
   | Resource group | **rg-devcenter-01**                                          |
   | Centre de développement     | **devcenter-01**                                             |
   | Nom           | **devcenter-project-01**                                     |
   | Description    | **devcenter-project-01**                                     |

1. Sous l’onglet **Gestion de dev box** de la page **Créer un projet**, spécifiez les paramètres suivants, puis sélectionnez **Suivant : Catalogues** :

   | Paramètre                 | Valeur |
   | ----------------------- | ----- |
   | Activer les limites de dev box   | Oui   |
   | Dev box par développeur | **2** |

1. Sous l’onglet **Catalogue** de la page **Créer un projet**, spécifiez les paramètres suivants, puis sélectionnez **Examiner et créer** :

   | Paramètre                            | Valeur   |
   | ---------------------------------- | ------- |
   | Définitions d’environnement de déploiement | Activé(e) |
   | Définitions d’image                  | Activé(e) |

   > **Notes :** étant donné que vous avez activé les catalogues au niveau du projet lors de la création du centre de développement, pour les catalogues attachés à un projet de développement, ces paramètres déterminent quels éléments de catalogue sont importés dans le projet lors de la synchronisation.

1. Sous l’onglet **Examiner et créer** de la page **Créer un centre de développement**, sélectionnez **Créer**.

   > **Note :** attendez que le projet soit créé. Cela devrait prendre moins d’une minute.

1. Dans la page **Déploiement terminé**, sélectionnez **Accéder à la ressource**.

### Tâche 5 : créer un pool de dev box

Dans cette tâche, vous allez créer un pool de dev box dans le projet du centre de développement que vous avez créé dans la tâche précédente. Les utilisateurs de dev box créent des dev box à l’aide des pools de dev box. Un pool de dev box lie une définition de dev box à une connexion réseau. En général, vous avez le choix entre des connexions hébergées par Microsoft ou vos propres connexions réseau Azure. La connexion réseau détermine l’emplacement de l’hébergement d’une dev box et son accès à d’autres ressources cloud et locales. Vous pouvez créer un pool de dev box avec une connexion réseau au plus près des utilisateurs de dev box. En outre, si vous souhaitez réduire le coût d’exécution des dev box, vous pouvez les configurer dans un pool pour qu’elles s’arrêtent quotidiennement à une heure prédéfinie.

1. Dans le navigateur web affichant le Portail Azure, dans la page **devcenter-project-01**, dans le menu de navigation vertical situé à gauche, développez la section **Gérer** et sélectionnez **Pools de dev box**.
1. Dans la page **Pools de dev box devcenter-project-01 \|**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un pool de dev box**, spécifiez les paramètres suivants et sélectionnez **Créer** :

   | Paramètre                                                                                                           | Valeur                                    |
   | ----------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
   | Nom                                                                                                              | **devcenter-project-01-devbox-pool-01**  |
   | Définition                                                                                                        | **devbox-definition-01**                 |
   | Connexion réseau                                                                                                | **Déployer sur un réseau hébergé par Microsoft** |
   | Région                                                                                                            | **(États-Unis) USA Est**                         |
   | Activer l'authentification unique                                                                                             | Activé(e)                                  |
   | Privilèges de créateur de dev box                                                                                        | **Administrateur local**                  |
   | Activer l’arrêt automatique selon la planification                                                                                      | Activé(e)                                  |
   | Heure d’arrêt                                                                                                         | **19 h**                             |
   | Fuseau horaire                                                                                                         | Votre fuseau horaire actuel                   |
   | Activer la mise en veille prolongée lors de la déconnexion                                                                                    | Activé(e)                                  |
   | Période de grâce en minutes                                                                                           | **60**                                   |
   | Je confirme que mon organisation dispose de licences Azure Hybrid Benefits, qui s’appliqueront à toutes les dev box de ce pool | Activé(e)                                  |

   > **Note :** attendez la création du pool de dev box. Cela peut prendre environ 2 minutes.

### Tâche 6 : configurer les autorisations

Dans cette tâche, vous allez attribuer des autorisations appropriées relatives aux dev box Microsoft aux trois principaux de sécurité Microsoft Entra qui ont été provisionnés dans votre environnement de labo. Ces principaux de sécurité correspondent aux rôles classiques dans les scénarios d’ingénierie de plateforme :

| Utilisateur              | Groupe                        | Rôle                  |
| ----------------- | ---------------------------- | --------------------- |
| platformegineer01 | DevCenter_Platform_Engineers | Ingénieur plateforme     |
| devlead01         | DevCenter_Dev_Leads          | Responsable de l’équipe de développement (Development team lead) |
| devuser01         | DevCenter_Dev_Users          | Développeur             |

Microsoft Dev Box utilise le contrôle d’accès en fonction du rôle (Azure RBAC) pour octroyer l’accès aux fonctionnalités au niveau du projet : Les ingénieurs de plateforme doivent avoir un contrôle total pour créer et gérer des centres de développement, leurs catalogues et leurs projets. Cela nécessite effectivement le rôle propriétaire ou collaborateur, selon le besoin de déléguer des autorisations à d’autres personnes. Les leads de l’équipe de développement doivent être affectés au rôle d’administrateur de projet du centre de développement, ce qui permet d’effectuer des tâches administratives sur les projets Microsoft Dev Box. Les utilisateurs de la zone de développement doivent pouvoir créer et gérer leurs propres dev box, qui sont associées au rôle utilisateur de dev box.

> **Note :** vous allez commencer par attribuer des autorisations au groupe Microsoft Entra destiné à contenir des comptes d’utilisateur d’ingénieur de plateforme.

1. Dans le navigateur web affichant le Portail Azure, accédez à la page **devcenter-01** et, dans le menu de navigation vertical situé à gauche, sélectionnez **Contrôle d’accès (IAM)**.
1. Dans la page **Contrôle d’accès (IAM) devcenter-01 \|**, sélectionnez **+ Ajouter** et, dans la liste déroulante, sélectionnez **Ajouter une attribution de rôle**.
1. Sous l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, sélectionnez **Rôles d’administrateur privilégiés**. Dans la liste des rôles, sélectionnez **Propriétaire**, puis **Suivant**.
1. Sous l’onglet **Membres** de la page **Ajouter une attribution de rôle**, vérifiez que l’option **Utilisateur, groupe ou principal de service** est sélectionnée, puis cliquez sur **+ Sélectionner des membres**.
1. Dans le volet **Sélectionner des membres**, recherchez et sélectionnez **`DevCenter_Platform_Engineers`**, puis cliquez sur **Sélectionner**.
1. Dans l’onglet **Membres** de la page **Ajouter une attribution de rôle**, cliquez sur **Suivant**.
1. Sous l’onglet **Conditions** de la page **Ajouter une attribution de rôle**, dans la section **Ce que l’utilisateur peut faire**, sélectionnez l’option **Autoriser l’utilisateur à attribuer tous les rôles (hautement privilégié),** puis sélectionnez **Suivant**.
1. Dans l’onglet **Vérifier + attribuer** de la page **Ajouter une attribution de rôle**, sélectionnez **Vérifier + attribuer**.

   > **Note :** vous allez ensuite attribuer des autorisations au groupe Microsoft Entra destiné à contenir des comptes d’utilisateur du lead de l’équipe de développement.

1. De retour à la page **Contrôle d’accès (IAM) devcenter-01 \|**, dans le menu de navigation vertical situé à gauche, développez la section **Gérer**, sélectionnez **Projets**, puis, dans la liste des projets, sélectionnez **devcenter-project-01**.
1. Dans la page **devcenter-project-01**, dans le menu vertical de gauche, sélectionnez **Contrôle d’accès (IAM)**.
1. Dans la page **Contrôle d’accès (IAM) evcenter-project-01 \|**, sélectionnez **+Ajouter** et, dans la liste déroulante, sélectionnez **Ajouter une attribution de rôle**.
1. Dans l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, vérifiez que l’onglet **Rôles** de fonction de tâche est sélectionné. Dans la liste des rôles, sélectionnez **Administrateur de projet DevCenter**, puis sélectionnez **Suivant**.
1. Sous l’onglet **Membres** de la page **Ajouter une attribution de rôle**, vérifiez que l’option **Utilisateur, groupe ou principal de service** est sélectionnée, puis cliquez sur **+ Sélectionner des membres**.
1. Dans le volet **Sélectionner des membres**, recherchez et sélectionnez **`DevCenter_Dev_Leads`**, puis cliquez sur **Sélectionner**.
1. Dans l’onglet **Membres** de la page **Ajouter une attribution de rôle**, cliquez sur **Suivant**.
1. Dans l’onglet **Vérifier + attribuer** de la page **Ajouter une attribution de rôle**, sélectionnez **Vérifier + attribuer**.

   > **Note :** enfin, vous allez attribuer des autorisations au groupe Microsoft Entra destiné à contenir des comptes d’utilisateur de développeur.

1. De retour à la page **Contrôle d’accès (IAM) devcenter-project-01 \|**, sélectionnez **+ Ajouter** puis, dans le menu déroulant, sélectionnez **Ajouter une attribution de rôle**.
1. Dans l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, vérifiez que l’onglet **Rôles de fonction de tâche** est sélectionné. Dans la liste des rôles, sélectionnez **Utilisateur DevCenter Dev Box**, puis sélectionnez **Suivant**.
1. Sous l’onglet **Membres** de la page **Ajouter une attribution de rôle**, vérifiez que l’option **Utilisateur, groupe ou principal de service** est sélectionnée, puis cliquez sur **+ Sélectionner des membres**.
1. Dans le volet **Sélectionner des membres**, recherchez et sélectionnez **`DevCenter_Dev_Users`**, puis cliquez sur **Sélectionner**.
1. Dans l’onglet **Membres** de la page **Ajouter une attribution de rôle**, cliquez sur **Suivant**.
1. Dans l’onglet **Vérifier + attribuer** de la page **Ajouter une attribution de rôle**, sélectionnez **Vérifier + attribuer**.

### Tâche 7 : évaluer une dev box

Dans cette tâche, vous allez évaluer une fonctionnalité de dev box à l’aide d’un compte d’utilisateur développeur Microsoft Entra.

1. Démarrez un navigateur web incognito/privé et accédez au portail des développeurs Microsoft Dev Box à l’adresse `https://aka.ms/devbox-portal`.
1. Lorsque vous êtes invité à vous connecter, fournissez les informations d’identification du compte d’utilisateur **devuser01**.
1. Dans la page **Bienvenue, devuser01** du portail des développeurs Microsoft Dev Box, sélectionnez **+Nouvelle dev box**.
1. Dans le volet **Ajouter une dev box**, dans la zone de texte **Nom**, entrez **`devuser01box01`**.
1. Passez en revue d’autres informations présentées dans le volet **Ajouter une dev box**, notamment le nom du projet, les spécifications du pool de dev box, l’état de prise en charge de la mise en veille prolongée et la durée d’arrêt planifiée. En outre, notez l’option permettant d’appliquer des personnalisations et la notification indiquant que la création d’une dev box peut prendre jusqu’à 65 minutes.

   > **Note :** les noms de dev box doivent être uniques dans le projet.

1. Dans le volet **Ajouter une dev box**, sélectionnez **Créer**.

   > **Note :** n’attendez pas que la zone de développement soit créée. Au lieu de cela, passez directement à l’exercice suivant de ce labo. Toutefois, envisagez de revenir à cet exercice et de parcourir le reste de cette tâche une fois que vous avez terminé l’exercice suivant.

1. Une fois que la zone de développement est entièrement approvisionnée et en cours d’exécution, connectez-vous à celle-ci en sélectionnant l’option **Se connecter via l’application**.

   > **Note :** la connectivité à une dev box peut être établie à l’aide d’une application Windows Bureau à distance, d’un client Bureau à distance (mstsc.exe) ou directement dans une fenêtre de navigateur web.

1. Dans la fenêtre indépendante **Ce site tente d’ouvrir le Centre de connexions Bureau à distance Microsoft**, sélectionnez **Ouvrir**. Cette opération lance automatiquement une session Bureau à distance dans la dev box.
1. Lorsque vous êtes invité à entrer des informations d’identification, authentifiez-vous en fournissant le nom d’utilisateur et le mot de passe du compte **devuser01**.
1. Dans la session Bureau à distance de la dev box, vérifiez que sa configuration inclut une installation des applications Visual Studio 2022 et Microsoft 365.

   > **Note :** vous pouvez arrêter la zone de développement directement à partir du portail des développeurs Microsoft Dev Box en tant qu’utilisateur de développement en sélectionnant d’abord le symbole de points de suspension dans l’interface **Votre dev box**, puis en sélectionnant **Arrêter** dans le menu en cascade. En tant qu’ingénieur plateforme, vous pouvez également contrôler le cycle de vie de la zone de développement à partir de la section **Pools dev box** du projet du centre de développement correspondant.

## Exercice 2 : Personnaliser un environnement Microsoft Dev Box

Dans cet exercice, vous allez personnaliser les fonctionnalités de l’environnement Microsoft Dev Box. Cette approche se concentre sur l’étendue des modifications que vous pouvez appliquer lors de l’implémentation d’une solution en libre-service de développeur personnalisée.

L’exercice se compose des tâches suivantes :

- Tâche 1 : créer une Azure Compute Gallery et l’attacher au centre de développement
- Tâche 2 : configurer l’authentification et l’autorisation pour Azure Image Builder
- Tâche 3 : créer une image personnalisée à l’aide d’Azure Image Builder
- Tâche 4 : créer une connexion réseau du centre de développement Azure
- Tâche 5 : ajouter des définitions d’images à un projet du centre de développement Azure
- Tâche 6 : créer un pool de dev box personnalisé
- Tâche 7 : évaluer une dev box personnalisée

### Tâche 1 : créer une Azure Compute Gallery et l’attacher au centre de développement

Dans cette tâche, vous allez créer une Azure Compute Gallery et l’attacher au centre de développement que vous avez créé dans l’exercice précédent de ce labo. Une galerie est un référentiel stocké dans votre abonnement Azure, qui vous aide à créer une structure et une organisation autour de vos ressources d’image. Après avoir attaché une galerie de calcul à un centre de développement dans Microsoft Dev Box, vous pouvez créer des définitions de Dev Box basées sur des images stockées dans la galerie de calcul.

1. Démarrez un navigateur web et accédez au portail Azure à l’adresse `https://portal.azure.com`.
1. Lorsque vous êtes invité à vous authentifier, connectez-vous avec votre compte Microsoft.
1. Dans le Portail Microsoft Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **`Azure compute galleries`**.
1. Dans la page **Azure compute galleries**, sélectionnez **+Créer**.
1. Dans l’onglet **Informations de base** de la page **Créer une galerie Azure Compute**, spécifiez les paramètres suivants, puis sélectionnez **Suivant : Méthode de partage** :

   | Paramètre        | Valeur                                                        |
   | -------------- | ------------------------------------------------------------ |
   | Abonnement   | Nom de l’abonnement Azure que vous utilisez dans ce labo |
   | Resource group | **rg-devcenter-01**                                          |
   | Nom           | **compute_gallery_01**                                       |
   | Région         | **(États-Unis) USA Est**                                             |

1. Dans l’onglet **Partage de la méthode** de la page **Créer une Azure Compute Gallery**, sélectionnez l’option **Contrôle d’accès en fonction du rôle (RBAC)**, puis sélectionnez **Examiner et créer**.
1. Sous l’onglet **Examiner et créer**, attendez que le processus de validation se termine, puis sélectionnez **Créer**.

   > **Note :** attendez que le projet soit provisionné. La création de la Azure Compute Gallery doit prendre moins de 1 minute.

1. Dans le Portail Azure, recherchez et sélectionnez **`Dev centers`** et, dans la page **Centres de développement**, sélectionnez **devcenter-01**.
1. Dans la page **devcenter-01**, dans le menu de navigation vertical situé à gauche, développez la section **Configuration de la dev box** et sélectionnez **Azure compute galleries**.
1. Dans la page **Azure Compute Galleries devcenter-01 \|**, sélectionnez **+ Ajouter**.
1. Dans le volet **Ajouter une Azure Compute Gallery**, dans la liste déroulante **Galerie**, sélectionnez **compute_gallery_01**, puis **Ajouter**.

   > **Note :** si vous recevez un message d’erreur : «_ Ce centre de développement n’a pas d’identité affectée par le système ou par l’utilisateur. Les galeries ne peuvent pas être ajoutées tant qu’une identité n’a pas été affectée._  », vous devez affecter une identité affectée par le système au centre de développement.
   > Pour ce faire, dans le Portail Azure, dans la page **devcenter-01**, dans le menu de navigation vertical situé à gauche, sélectionnez **Identité** sous Paramètres. Sous l’onglet **Affecté par le système**, définissez le commutateur **État** sur **activé**, puis sélectionnez **Enregistrer**.

### Tâche 2 : configurer l’authentification et l’autorisation

Dans cette tâche, vous allez créer une identité managée affectée par l’utilisateur qui sera utilisée par Azure Image Builder pour ajouter des images à la galerie de calcul Azure que vous avez créée dans la tâche précédente. Vous allez également configurer les autorisations requises en créant un rôle de contrôle d’accès en fonction du rôle personnalisé (RBAC) et en l’affectant à l’identité managée. Cela vous permet d’utiliser Azure Image Builder dans la tâche suivante pour générer une image personnalisée.

1. Dans le Portail Azure, sélectionnez l’icône de barre d’outils **Cloud Shell** pour ouvrir le volet Cloud Shell et, si nécessaire. Sélectionnez **Basculer vers PowerShell** pour démarrer une session PowerShell et, dans la boîte de dialogue **Basculer vers PowerShell dans Cloud Shell**. Sélectionnez **Confirmer**.

   > **Note :** si c’est la première fois que vous ouvrez Cloud Shell, dans la **boîte de dialogue Bienvenue dans Azure Cloud Shell**, sélectionnez **PowerShell**. Dans le volet **Prise en main**, sélectionnez l’option **Aucun compte de stockage requis**. Dans la liste déroulante **Abonnement**, sélectionnez le nom de l’abonnement Azure que vous utilisez dans ce labo.

1. Dans la session PowerShell du volet Cloud Shell, exécutez les commandes suivantes pour vous assurer que tous les fournisseurs de ressources requis sont inscrits :

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
   Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
   Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
   Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
   Register-AzResourceProvider -ProviderNamespace Microsoft.Network
   ```

1. Exécutez la commande suivante pour installer les modules PowerShell requis (lorsque vous y êtes invité, tapez **A** et appuyez sur la **touche Entrée**) :

   ```powershell
   'Az.ImageBuilder', 'Az.ManagedServiceIdentity' | ForEach-Object {Install-Module -Name $_ -AllowPrerelease}
   ```

1. Exécutez les commandes suivantes pour configurer des variables qui seront référencées tout au long du processus de génération d’image :

   ```powershell
   $currentAzContext = Get-AzContext
   # the target Azure subscription ID
   $subscriptionID=$currentAzContext.Subscription.Id
   # the target Azure resource group name
   $imageResourceGroup='rg-devcenter-01'
   # the target Azure region
   $location='eastus'
   # the reference name assigned to the image created by using the Azure Image Builder service
   $runOutputName="aibWinImg01"
   # image template name
   $imageTemplateName="templateWinVSCode01"
   # the Azure compute gallery name
   $computeGallery = 'compute_gallery_01'
   ```

1. Exécutez les commandes suivantes pour créer une identité managée affectée par l’utilisateur (VM Image Builder utilise l’identité utilisateur que vous fournissez pour stocker des images dans la Azure Compute Gallery cible) :

   ```powershell
   # Install the Azure PowerShell module to support AzUserAssignedIdentity
   Install-Module -Name Az.ManagedServiceIdentity
   # Generate a pseudo-random integer to be used for resource names
   $timeInt=$(get-date -UFormat "%s")

   # Create an identity
   $identityName='identityAIB' + $timeInt
   New-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName -Location $location
   $identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
   $identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId
   ```

1. Exécutez les commandes suivantes pour accorder à l’identité managée affectée par l’utilisateur nouvellement créée les autorisations nécessaires pour stocker des images dans le groupe de ressources **rg-devcenter-01** :

   ```powershell
   # Set variables
   $imageRoleDefName = 'Custom Azure Image Builder Image Def' + $timeInt
   $aibRoleImageCreationUrl = 'https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json'
   $aibRoleImageCreationPath = 'aibRoleImageCreation.json'

   # Customize the role definition file
   Invoke-WebRequest -Uri $aibRoleImageCreationUrl -OutFile $aibRoleImageCreationPath -UseBasicParsing
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace '<subscriptionID>', $subscriptionID) | Set-Content -Path $aibRoleImageCreationPath
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace '<rgName>', $imageResourceGroup) | Set-Content -Path $aibRoleImageCreationPath
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName) | Set-Content -Path $aibRoleImageCreationPath

   # Create a role definition
   New-AzRoleDefinition -InputFile  ./aibRoleImageCreation.json

   # Assign the role to the VM Image Builder user-assigned managed identity within the scope of the **rg-devcenter-01** resource group
   New-AzRoleAssignment -ObjectId $identityNamePrincipalId -RoleDefinitionName $imageRoleDefName -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"
   ```

### Tâche 3 : créer une image personnalisée à l’aide d’Azure Image Builder

Dans cette tâche, vous allez utiliser Azure Image Builder pour créer une image personnalisée basée sur un modèle Azure Resource Manager (ARM) existant qui définit une image Windows 11 Entreprise avec Choco et Visual Studio Code installés automatiquement. Azure VM Image Builder simplifie considérablement le processus de définition et d’approvisionnement d’images de machine virtuelle. Il s’appuie sur une configuration d’image que vous spécifiez pour configurer un pipeline d’imagerie automatisée. Par la suite, les développeurs pourront utiliser ces images pour approvisionner leurs dev box.

1. Dans la session PowerShell du volet Cloud Shell, exécutez les commandes suivantes pour créer une définition d’image à ajouter à la Azure Compute Gallery que vous avez créée dans la première tâche de cet exercice :

   ```powershell
   # ensure that the image definition security type property is set to 'TrustedLaunch'
   $securityType = @{Name='SecurityType';Value='TrustedLaunch'}
   $features = @($securityType)
   # Image definition name
   $imageDefName = 'imageDefDevBoxVSCode01'

   # Create the image definition
   New-AzGalleryImageDefinition -GalleryName $computeGallery -ResourceGroupName $imageResourceGroup -Location $location -Name $imageDefName -OsState generalized -OsType Windows -Publisher 'Contoso' -Offer 'vscodedevbox' -Sku '1-0-0' -Feature $features -HyperVGeneration 'V2'
   ```

   > **Note :** une image de dev box doit satisfaire à un certain nombre de conditions, notamment l’utilisation de Generation 2, Hyper-V v2 et Windows 10 ou 11 Entreprise version 20H2 ou ultérieure. Pour obtenir la liste complète, reportez-vous à l’article [Configurer Azure Compute Gallery pour Microsoft Dev Box](https://learn.microsoft.com/en-us/azure/dev-box/how-to-configure-azure-compute-gallery).

1. Exécutez les commandes suivantes pour créer un fichier vide nommé template.json qui contiendra un modèle ARM définissant une image Windows 11 Entreprise avec choco et Visual Studio Code installés automatiquement :

   ```powershell
   Set-Location -Path ~
   $templateFile = 'template.json'
   Set-Content -Path $templateFile -Value ''
   ```

1. Dans la session PowerShell de Cloud Shell, utilisez l’éditeur de texte nano pour ajouter le contenu suivant au fichier nouvellement créé :

   > **Note :** Exécutez la commande `nano ./template.json` pour ouvrir l’éditeur de texte nano. Pour enregistrer les modifications et quitter l’éditeur de texte nano, appuyez sur **Ctrl+X**, puis sur **Y**, puis sur **Entrée**.

   ```json
   {
     "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "imageTemplateName": {
         "type": "string"
       },
       "api-version": {
         "type": "string"
       },
       "svclocation": {
         "type": "string"
       }
     },
     "variables": {},
     "resources": [
       {
         "name": "[parameters('imageTemplateName')]",
         "type": "Microsoft.VirtualMachineImages/imageTemplates",
         "apiVersion": "[parameters('api-version')]",
         "location": "[parameters('svclocation')]",
         "dependsOn": [],
         "tags": {
           "imagebuilderTemplate": "win11multi",
           "userIdentity": "enabled"
         },
         "identity": {
           "type": "UserAssigned",
           "userAssignedIdentities": {
             "<imgBuilderId>": {}
           }
         },
         "properties": {
           "buildTimeoutInMinutes": 100,
           "vmProfile": {
             "vmSize": "Standard_DS2_v2",
             "osDiskSizeGB": 127
           },
           "source": {
             "type": "PlatformImage",
             "publisher": "MicrosoftWindowsDesktop",
             "offer": "Windows-11",
             "sku": "win11-21h2-ent",
             "version": "latest"
           },
           "customize": [
             {
               "type": "PowerShell",
               "name": "Install Choco and Vscode",
               "inline": [
                 "Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))",
                 "choco install -y vscode"
               ]
             }
           ],
           "distribute": [
             {
               "type": "SharedImage",
               "galleryImageId": "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<sharedImageGalName>/images/<imageDefName>",
               "runOutputName": "<runOutputName>",
               "artifactTags": {
                 "source": "azureVmImageBuilder",
                 "baseosimg": "win11multi"
               },
               "replicationRegions": ["<region1>", "<region2>"]
             }
           ]
         }
       }
     ]
   }
   ```

1. Exécutez les commandes suivantes pour remplacer les espaces réservés dans l’template.json par les valeurs spécifiques à votre environnement Azure :

   ```powershell
   $replRegion2 = 'eastus2'
   $templateFilePath = '.\template.json'
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<subscriptionID>', $subscriptionID | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<rgName>', $imageResourceGroup | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<runOutputName>', $runOutputName | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<imageDefName>', $imageDefName | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<sharedImageGalName>', $computeGallery | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<region1>', $location | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<region2>', $replRegion2 | Set-Content -Path $templateFilePath
   ((Get-Content -Path $templateFilePath -Raw) -Replace '<imgBuilderId>', $identityNameResourceId) | Set-Content -Path $templateFilePath
   ```

1. Exécutez la commande suivante pour envoyer le modèle au service Azure Image Builder (le service traite le modèle soumis en téléchargeant tous les artefacts dépendants, tels que des scripts, et en les stockant dans un groupe de ressources intermédiaire, dont le nom inclut le préfixe **IT\_**, pour créer une image de machine virtuelle personnalisée :

   ```powershell
   New-AzResourceGroupDeployment -ResourceGroupName $imageResourceGroup -TemplateFile $templateFilePath -Api-Version "2020-02-14" -imageTemplateName $imageTemplateName -svclocation $location
   ```

1. Exécutez la commande suivante pour invoquer le processus de création d’image :

   ```powershell
   Invoke-AzResourceAction -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2020-02-14" -Action Run -Force
   ```

1. Exécutez la commande suivante pour déterminer l’état d’approvisionnement d’images :

   ```powershell
   Get-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup | Select-Object -Property Name, LastRunStatusRunState, LastRunStatusMessage, ProvisioningState
   ```

   > **Note :** la sortie suivante indique que le processus de génération s’est terminé correctement :

   ```powershell
   Name                LastRunStatusRunState LastRunStatusMessage ProvisioningState
   ----                --------------------- -------------------- -----------------
   templateWinVSCode01 Succeeded                                  Succeeded
   ```

1. Vous pouvez également surveiller la progression de la build en procédant comme suit :

   1. Dans le portail Azure, recherchez et sélectionnez **`Image templates`**.
   1. Dans la page **Modèles d’images**, sélectionnez **templateWinVSCode01**.
   1. Dans la page **templateWinVSCode01**, dans la section **Essentials**, notez la valeur de l’entrée **État d’exécution de build**.

   > **Note :** le processus de génération peut prendre environ 30 minutes. N’attendez pas que l’opération se termine, mais passez à la tâche suivante de l’exercice.

1. Une fois la build terminée, dans le Portail Azure, recherchez et sélectionnez **`Azure compute galleries`**.
1. Dans la page **Azure Compute Galleries**, sélectionnez **compute_gallery_01**.
1. Dans la page **compute_gallery_01**, vérifiez que l’onglet **Définitions** est sélectionné et, dans la liste des définitions, sélectionnez **imageDefDevBoxVSCode**.
1. Dans la page **imageDefDevBoxVSCode**, sélectionnez l’onglet **Versions** et vérifiez que l’entrée **1.0.0 (dernière version)** apparaît dans la liste avec l’**état d’approvisionnement** défini sur **Réussite**.
1. Sélectionnez l’entrée **1.0.0 (dernière version)**.
1. Dans la page **1.0.0 (compute_gallery_01/imageDefDevBoxVSCode/1.0.0)**, passez en revue les paramètres de version de l’image de machine virtuelle.

### Tâche 4 : créer une connexion réseau du centre de développement Azure

Dans cette tâche, vous allez configurer la mise en réseau du centre de développement Azure à utiliser dans un scénario nécessitant une connectivité privée aux ressources hébergées dans un réseau virtuel Azure. Contrairement au réseau hébergé par Microsoft que vous avez utilisé dans le premier exercice de ce labo, les connexions de réseau virtuel prennent également en charge les scénarios hybrides (fournissant une connectivité aux ressources locales) et la jonction hybride Microsoft Entra des zones de développement Azure (en plus de la prise en charge de la jonction Microsoft Entra).

1. Dans le navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **`Virtual networks`**.
1. Dans la page **Réseaux virtuels**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** du panneau **Créer un réseau virtuel**, spécifier les paramètres suivants et sélectionner **Suivant** :

   | Paramètre        | Valeur                                                        |
   | -------------- | ------------------------------------------------------------ |
   | Abonnement   | Nom de l’abonnement Azure que vous utilisez dans ce labo |
   | Resource group | **rg-devcenter-01**                                          |
   | Nom           | **vnet-01**                                                  |
   | Emplacement       | **(États-Unis) USA Est**                                             |

1. Sous l’onglet **Sécurité** de la page **Créer un réseau virtuel**, acceptez les paramètres par défaut et sélectionnez **Suivant**.
1. Sous l’onglet **Adresses IP** de la page **Créer un réseau virtuel**, passez en revue les paramètres existants sans modifier leurs valeurs par défaut, puis sélectionnez **Examiner et créer**.
1. Dans l’onglet **Examiner et créer** de la page **Créer une machine virtuelle**, sélectionnez **Créer**.

   > **Note :** attendez que le réseau virtuel soit créé. Cela devrait prendre moins d’une minute.

1. Dans le Portail Microsoft Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **`Network connections`**.
1. Dans la page **Connexions réseau**, sélectionnez **+Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer une connexion réseau**, spécifiez les paramètres suivants et sélectionnez **Examiner et créer** :

   | Paramètre         | Valeur                                                        |
   | --------------- | ------------------------------------------------------------ |
   | Abonnement    | Nom de l’abonnement Azure que vous utilisez dans ce labo |
   | Resource group  | **rg-devcenter-01**                                          |
   | Nom            | **network-connection-vnet-01**                               |
   | Réseau virtuel | **vnet-01**                                                  |
   | Sous-réseau          | **default**                                                  |

1. Dans l’onglet **Examiner et créer** de la page **Créer une machine virtuelle**, sélectionnez **Créer**.

   > **Note :** attendez la création de la connexion réseau. Cela peut prendre environ 1 minute.

1. Dans le Portail Azure, recherchez et sélectionnez **`Dev centers`** et, dans la page **Centres de développement**, sélectionnez **devcenter-01**.
1. Dans la page **devcenter-01**, dans le menu de navigation vertical situé à gauche, développez la section **Configuration de la dev box** et sélectionnez **Mise en réseau**.
1. Dans la page **Mise en réseau devcenter-01 \|**, sélectionnez **+Ajouter**.
1. Dans le volet **Ajouter une connexion réseau**, dans la liste déroulante **Connexion réseau**, sélectionnez **network-connection-vnet-01**, puis sélectionnez **Ajouter**.

   > **Note :** n’attendez pas que la connexion réseau soit créée, mais passez à la tâche suivante. L’ajout d’une connexion réseau peut prendre environ 1 minute.

### Tâche 5 : ajouter des définitions d’images à un projet du centre de développement Azure

Dans cette tâche, vous allez ajouter des définitions d’images à un projet du centre de développement Azure que vous avez créé dans le premier exercice de ce labo. Les définitions d’images combinent une Place de marché Azure ou une image personnalisée avec des tâches configurables qui définissent des modifications supplémentaires à appliquer à l’image sous-jacente. Une définition d’image peut être utilisée pour générer une nouvelle image (contenant toutes les modifications, y compris celles appliquées par des tâches) ou pour créer des pools de dev box directement. La création d’une image réutilisable réduit le temps nécessaire pour l’approvisionnement de dev box.

Pour configurer l’imagerie pour les personnalisations de l’équipe Microsoft Dev Box, les catalogues au niveau du projet (que vous avez déjà créés dans le premier exercice de ce labo) doivent être activés. Dans cette tâche, vous allez configurer les paramètres de synchronisation du catalogue pour le projet. Cela implique l’attachement d’un catalogue qui contient des fichiers de définition d’image.

1. Dans le navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **`Dev centers`**.
1. Dans la page **Centres de développement**, sélectionnez **devcenter-01**.
1. Dans la page **devcenter-01**, dans le menu de navigation vertical situé à gauche, dans la section **Gérer**, sélectionnez **Projets**.
1. Dans la **Projets devcenter-01 \|**, dans la liste des projets, sélectionnez **devcenter-project-01**.
1. Dans la page **devcenter-project-01**, dans le menu de navigation vertical situé à gauche, développez la section **Paramètres** et sélectionnez **Catalogues**.
1. Dans la page **Catalogues devcenter-project-01 \|**, sélectionnez **+Ajouter**.
1. Dans le volet Ajouter un **catalogue**, dans la zone de texte **Nom**, entrez **`image-definitions-01`**. Dans la section **Source du catalogue**, sélectionnez **GitHub**. Dans le **type d’authentification**, sélectionnez l’**application GitHub**, laissez la case à cocher **Synchroniser automatiquement ce catalogue** activée, puis sélectionnez **Se connecter avec GitHub**.
1. Si vous y êtes invité, dans la fenêtre **Se connecter avec GitHub**, entrez vos informations d’identification GitHub et sélectionnez **Se connecter**.

   > **Note :** vous devez dupliquer le référentiel https://github.com/MicrosoftLearning/contoso-co-eShop sur votre compte GitHub avant de pouvoir effectuer cette étape.

1. Si vous y êtes invité, dans la fenêtre **Autoriser Microsoft DevCenter**, sélectionnez **Autoriser Microsoft DevCenter**.
1. De retour au volet **Ajouter un catalogue**, dans la liste déroulante **Référentiel**, sélectionnez **contoso-co-eShop**. Dans la liste déroulante **Branche**, acceptez l’**entrée de branche par défaut**. Dans le **chemin du dossier**, entrez **`.devcenter/catalog/image-definitions`**, puis sélectionnez **Ajouter**.
1. De retour à la page **Catalogues devcenter-project-01 \|**, vérifiez que la synchronisation se termine correctement en surveillant l’entrée dans la colonne **État**.
1. Sélectionnez le lien **Synchronisation réussie** dans la colonne **État**, passez en revue le volet de notification résultant, vérifiez que 3 éléments ont été ajoutés au catalogue, puis fermez le volet en sélectionnant le symbole **x** dans le coin supérieur droit.
1. De retour à la page **Catalogues devcenter-project-01 \|**, sélectionnez **image-definitions-01** et vérifiez qu’elle contient trois entrées nommées **ContosoBaseImageDefinition**, **backend-eng** et **frontend-eng**.

   > **Note :** dans le cadre de ce labo, vous allez tester les fonctionnalités de la définition d’image **frontend-eng**, qui a le contenu suivant :

   ```yaml
   $schema: "1.0"
   name: "frontend-eng"
   # Using the "Windows 11 Enterprise 24H2" image as the base
   image: microsoftwindowsdesktop_windows-ent-cpc_win11-24H2-ent-cpc
   description: "This definition is for the eShop frontend engineering environment"

   tasks:
     - name: ~/winget
       description: Install Visual Studio Code
       parameters:
         package: Microsoft.VisualStudioCode
         runAsUser: true
   ```

1. Dans le Portail Azure, revenez à la page **devcenter-project-01**. Dans le menu de navigation vertical situé à gauche, développez la section **Gérer**, sélectionnez **Définitions d’images** et vérifiez que la page affiche les 3 mêmes définitions d’images que vous avez identifiées précédemment dans cette tâche.

### Tâche 6 : créer un pool de dev box personnalisé

Dans cette tâche, vous allez utiliser les définitions d’images nouvellement approvisionnées pour créer un pool de dev box. Le pool utilise également la connexion réseau que vous avez configurée précédemment dans cet exercice.

1. Dans le Portail Azure affichant la page **Définitions d’images devcenter-project-01 \|**, dans le menu de navigation verticale situé à gauche, dans la section **Gérer**, sélectionnez **Pools de dev box**.
1. Dans la page **Pools de dev box devcenter-project-01 \|**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un pool de dev box**, spécifiez les paramètres suivants et sélectionnez **Créer** :

   | Paramètre                                                                                                           | Valeur                                                 |
   | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
   | Nom                                                                                                              | **devcenter-project-01-devbox-pool-02**               |
   | Définition                                                                                                        | **frontend-eng**                                      |
   | Compute                                                                                                           | **8 processeurs virtuels, 32 Go de RAM**                                 |
   | Stockage                                                                                                           | **SSD de 256 Go**                                        |
   | Connexion réseau                                                                                                | **Déployer sur une connexion réseau dans mon organisation** |
   | Nom de la connexion réseau                                                                                           | **network-connection-vnet-01**                        |
   | Activer l'authentification unique                                                                                             | Activé(e)                                               |
   | Privilèges de créateur de dev box                                                                                        | **Administrateur local**                               |
   | Activer l’arrêt automatique selon la planification                                                                                      | Activé(e)                                               |
   | Heure d’arrêt                                                                                                         | **19 h**                                          |
   | Fuseau horaire                                                                                                         | Votre fuseau horaire actuel                                |
   | Activer la mise en veille prolongée lors de la déconnexion                                                                                    | Activé(e)                                               |
   | Période de grâce en minutes                                                                                           | **60**                                                |
   | Je confirme que mon organisation dispose de licences Azure Hybrid Benefits, qui s’appliqueront à toutes les dev box de ce pool | Activé(e)                                               |

   > **Note :** attendez la création du pool de dev box. Cela peut prendre environ 2 minutes.

### Tâche 7 : évaluer une dev box personnalisée

Dans cette tâche, vous allez évaluer les fonctionnalités d’une dev box personnalisée à l’aide d’un compte d’utilisateur développeur Microsoft Entra.

> **Note :** compte tenu du temps supplémentaire nécessaire pour effectuer cette tâche, son achèvement est facultatif.

1. Démarrez un navigateur web incognito/privé et accédez au portail des développeurs Microsoft Dev Box à l’adresse `https://aka.ms/devbox-portal`.
1. Lorsque vous êtes invité à vous connecter, fournissez les informations d’identification du compte d’utilisateur **devuser01**.
1. Dans la page **Bienvenue, devuser01** du portail des développeurs Microsoft Dev Box, sélectionnez **+Nouvelle dev box**.
1. Dans le volet **Ajouter une dev box**, dans la zone de texte **Nom**, entrez **`devuser01box02`**.
1. Dans la liste déroulante du **pool de dev box**, sélectionnez **devcenter-project-01-devbox-pool-02**.
1. Passez en revue d’autres informations présentées dans le volet **Ajouter une dev box**, notamment le nom du projet, les spécifications du pool de dev box, l’état de prise en charge de la mise en veille prolongée et la durée d’arrêt planifiée. En outre, notez l’option permettant d’appliquer des personnalisations et la notification indiquant que la création d’une dev box peut prendre jusqu’à 65 minutes.
1. Dans le volet **Ajouter une dev box**, sélectionnez **Créer**.
1. Une fois que la zone de développement est entièrement approvisionnée et en cours d’exécution, connectez-vous à celle-ci en sélectionnant l’option **Se connecter via l’application**.
1. Dans la fenêtre indépendante **Ce site tente d’ouvrir le Centre de connexions Bureau à distance Microsoft**, sélectionnez **Ouvrir**. Cette opération lance automatiquement une session Bureau à distance dans la dev box.
1. Lorsque vous êtes invité à entrer des informations d’identification, authentifiez-vous en fournissant le nom d’utilisateur et le mot de passe du compte **devuser01**.
1. Dans la session Bureau à distance de la dev box, vérifiez que sa configuration inclut une installation de Visual Studio Code.

   > **Note :** vous pouvez également valider le résultat de la personnalisation en sélectionnant le symbole de sélection dans l’interface **Votre dev box**, puis en sélectionnant **Personnalisations** dans le menu en cascade.
