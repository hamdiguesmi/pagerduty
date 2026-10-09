# Gouvernance PagerDuty — Plan et pages SharePoint

Version : 2026-10-08 · Auteur : Hamdi

## Partie A — Plan de gouvernance

L'équipe PagerDuty devient le centre d'expertise (CoE) propriétaire de la plateforme : elle fixe les normes, approuve les adhésions et garde le contrôle des licences, tandis que chaque direction reste responsable de ses propres services et horaires d'astreinte.

### Objectifs

1. Uniformiser la configuration (nommage, équipes, politiques d'escalade) pour toutes les directions.
2. Réduire le bruit d'alertes et le délai de prise en charge des incidents (MTTA, MTTR).
3. Encadrer l'attribution des licences et les coûts, en appui à l'appel d'offres.
4. Faciliter l'adhésion des directions grâce à un parcours clair et documenté.
5. Protéger l'information : aucun renseignement confidentiel ou personnel dans les alertes.

### Rôles et responsabilités (RACI)

| Activité | Équipe PagerDuty (CoE) | Responsable de direction | Gestionnaire d'équipe PD | Intervenant d'astreinte | Sécurité de l'information |
| --- | --- | --- | --- | --- | --- |
| Normes et conventions | R/A | C | I | I | C |
| Approbation d'une adhésion | A | R | C | I | I |
| Attribution des licences | R/A | C | I | I | I |
| Création des équipes et services | A | I | R | I | I |
| Horaires et politiques d'escalade | C | A | R | I | I |
| Intégrations (ServiceNow, AppDynamics) | R/A | I | C | I | C |
| Réponse aux incidents | I | I | A | R | I |
| Revue des accès (trimestrielle) | R | A | C | I | C |
| Revue post-incident | C | A | R | R | I |

### Modèle d'organisation dans PagerDuty

- **Rôles de base** : Administrateurs de compte limités à l'équipe CoE ; Gestionnaires d'équipe désignés par chaque direction ; Intervenants (Responder) pour l'astreinte ; Observateurs (Stakeholder) pour la consultation.
- **Hiérarchie** : Direction → Équipe PagerDuty → Services techniques → Services d'affaires (Business Services) pour la vue de haut niveau.
- **Authentification** : SSO obligatoire, provisionnement des comptes par groupes d'annuaire, aucun compte local sauf compte d'urgence géré par le CoE.
- **Licences** : attribuées sur demande approuvée ; revue trimestrielle des comptes inactifs depuis 90 jours.

### Processus encadrés

| Processus | Déclencheur | Livrable | Responsable |
| --- | --- | --- | --- |
| Adhésion d'une direction | Formulaire de demande | Équipe, services et escalades configurés | CoE |
| Ajout ou retrait d'utilisateur | Arrivée, départ, mutation | Compte créé ou désactivé | Gestionnaire d'équipe |
| Nouvelle intégration | Demande de la direction | Intégration testée et documentée | CoE |
| Changement de norme | Proposition au comité | Norme publiée, versionnée | CoE |
| Revue des accès | Trimestrielle | Rapport et correctifs | CoE |
| Revue de la qualité des alertes | Mensuelle | Liste des alertes bruyantes à corriger | Gestionnaire d'équipe |

### Comité de gouvernance

Un comité se réunit chaque trimestre : CoE PagerDuty (présidence), un représentant par direction adhérente, Sécurité de l'information, Gestion des services (ServiceNow). Il approuve les normes, suit les indicateurs et arbitre les demandes de licences.

### Indicateurs de suivi

| Indicateur | Définition | Cible proposée |
| --- | --- | --- |
| MTTA | Délai moyen d'accusé de réception | À fixer par direction après 3 mois de mesure |
| MTTR | Délai moyen de résolution | À fixer par direction |
| Taux d'alertes non actionnables | Alertes résolues sans action / total | Baisse continue, revue mensuelle |
| Incidents hors heures par intervenant | Interruptions de nuit et de fin de semaine | Suivi pour la charge d'astreinte |
| Services conformes aux normes | Services respectant nommage et escalade | 100 % |
| Licences utilisées | Comptes actifs / licences achetées | Donnée d'entrée pour l'appel d'offres |

### Feuille de route

1. **Fondation** : normes, rôles, configuration SSO, nettoyage de l'existant.
2. **Publication** : site SharePoint, formulaire d'adhésion, séance d'information aux directions.
3. **Adhésion pilote** : une ou deux directions volontaires, ajustement du guide.
4. **Déploiement** : ouverture à toutes les directions, revues mensuelles de la qualité des alertes.
5. **Amélioration continue** : comité trimestriel, rapport d'utilisation intégré à l'appel d'offres.

Question ouverte : dates et cibles chiffrées à confirmer avec la direction.

## Partie B — Architecture du site SharePoint

Un site de communication unique, « PagerDuty — Gestion des incidents et de l'astreinte », regroupe sept pages et deux bibliothèques ; l'équipe CoE en est propriétaire, toutes les directions en sont lectrices.

| Page | Fichier SharePoint | Public cible | Disposition suggérée |
| --- | --- | --- | --- |
| 1. Accueil | Accueil.aspx | Tous | Bannière héros, 4 tuiles d'accès rapide, Actualités |
| 2. Gouvernance et rôles | Gouvernance.aspx | Gestionnaires, directions | Texte + tableau RACI |
| 3. Normes et nommage | Normes.aspx | Gestionnaires d'équipe | Texte + tableaux de conventions |
| 4. Bonnes pratiques | Bonnes-pratiques.aspx | Intervenants, gestionnaires | Sections repliables |
| 5. Guide d'adhésion | Adhesion.aspx | Directions | Étapes numérotées + bouton « Faire une demande » |
| 6. Intégrations | Integrations.aspx | Équipes techniques | Tableau du catalogue |
| 7. FAQ et soutien | FAQ.aspx | Tous | Composant FAQ ou sections repliables |

**Bibliothèques et listes**

- Bibliothèque « Documents de référence » : normes versionnées (PDF), gabarits, présentations de formation.
- Liste « Demandes d'adhésion » ou formulaire Microsoft Forms relié à Power Automate, ou demande ServiceNow si un article de catalogue existe.

**Navigation (menu horizontal)** : Accueil · Gouvernance · Normes · Bonnes pratiques · Adhérer · Intégrations · FAQ.

**Permissions** : Propriétaires = équipe CoE ; Membres = aucun (contenu contrôlé) ; Visiteurs = tous les employés. Chaque page affiche un pied « Propriétaire : équipe PagerDuty — Dernière révision : AAAA-MM-JJ ».

**Mise en ligne** : copier chaque page ci-dessous dans un composant Texte SharePoint ; les tableaux se collent directement depuis le document exporté en Word.

## Page 1 — Accueil PagerDuty

### Bannière

**PagerDuty : la bonne personne, alertée au bon moment.**
La plateforme organisationnelle de gestion des alertes, de l'astreinte et de la réponse aux incidents.

### Qu'est-ce que PagerDuty ?

PagerDuty reçoit les alertes de nos outils de surveillance (AppDynamics et autres), les regroupe en incidents et avise automatiquement l'intervenant d'astreinte par application mobile, texto, appel ou courriel. Si personne ne répond, l'incident est escaladé selon des règles définies d'avance. Les incidents sont synchronisés avec ServiceNow pour assurer la traçabilité.

### Pourquoi l'utiliser ?

- **Moins de temps perdu** : l'alerte va directement à la personne d'astreinte, sans chaîne téléphonique.
- **Aucune alerte oubliée** : l'escalade automatique garantit une prise en charge.
- **Horaires d'astreinte clairs** : rotations, remplacements et congés gérés dans un seul outil.
- **Moins de bruit** : regroupement et suppression des alertes en double.
- **Une vue commune** : état des services et incidents en cours visibles par les parties prenantes.

### Accès rapide (tuiles)

| Tuile | Lien vers |
| --- | --- |
| Adhérer au service | Page 5 — Guide d'adhésion |
| Normes et nommage | Page 3 |
| Bonnes pratiques | Page 4 |
| Obtenir de l'aide | Page 7 — FAQ et soutien |

### Qui sommes-nous ?

L'équipe PagerDuty (architecture d'exploitation) administre la plateforme, définit les normes et accompagne les directions dans leur adhésion. Pour nous joindre : [boîte courriel de l'équipe] ou une demande dans ServiceNow, groupe [nom du groupe d'affectation].

### Actualités

Composant Actualités du site : annonces de nouvelles fonctionnalités, séances de formation, changements de normes.

## Page 2 — Gouvernance et rôles

PagerDuty est un service organisationnel administré par l'équipe PagerDuty. Chaque direction qui y adhère reste responsable de ses services, de ses horaires d'astreinte et de la qualité de ses alertes.

### Principes directeurs

1. **Un propriétaire pour chaque service** : tout service PagerDuty a une équipe et un gestionnaire nommés.
2. **Aucune alerte sans escalade** : chaque service est relié à une politique d'escalade d'au moins deux niveaux.
3. **Une alerte doit être actionnable** : si personne n'a d'action à poser, elle ne doit pas réveiller quelqu'un.
4. **Les normes s'appliquent à tous** : nommage, rôles et intégrations suivent les normes publiées.
5. **Information protégée** : aucun renseignement personnel ou confidentiel dans le titre ou le contenu d'une alerte.
6. **Accès au besoin** : les droits sont accordés selon le rôle et revus chaque trimestre.

### Qui fait quoi

| Rôle | Qui | Responsabilités principales |
| --- | --- | --- |
| Équipe PagerDuty (CoE) | Architecture d'exploitation | Administration du compte, normes, licences, intégrations, SSO, accompagnement, revue des accès |
| Responsable de direction | Gestionnaire désigné par la direction | Approuve l'adhésion et les licences de sa direction, assume la charge d'astreinte |
| Gestionnaire d'équipe PagerDuty | Un ou deux par équipe | Configure services, horaires et escalades ; tient les membres à jour ; revue mensuelle des alertes |
| Intervenant d'astreinte | Membres des équipes | Accuse réception, traite ou escalade les incidents ; met à jour ses coordonnées |
| Partie prenante (observateur) | Gestionnaires, communications | Consulte l'état des services et reçoit les mises à jour d'incidents |

### Rôles dans l'outil

| Rôle PagerDuty | Attribué à | Remarque |
| --- | --- | --- |
| Administrateur de compte | Équipe CoE seulement | Nombre restreint, revu chaque trimestre |
| Gestionnaire (Manager) d'équipe | Gestionnaire d'équipe PD | Droits limités à son équipe |
| Intervenant (Responder) | Personnel d'astreinte | Licence complète |
| Partie prenante (Stakeholder) | Observateurs | Licence de partie prenante, lecture seule |

### Comité de gouvernance

Réuni chaque trimestre, il approuve les normes, suit les indicateurs (MTTA, MTTR, bruit d'alertes, licences) et arbitre les demandes. Les procès-verbaux sont déposés dans la bibliothèque « Documents de référence ».

### Dérogations

Toute exception à une norme est demandée par écrit à l'équipe PagerDuty, avec justification et durée. Les dérogations approuvées sont consignées et revues au comité.

## Page 3 — Normes et conventions de nommage

Tous les objets PagerDuty suivent le format `[DIRECTION]-[EQUIPE]-[objet]` en majuscules pour les codes, sans accents ni espaces ; ce format permet de filtrer, de produire des rapports par direction et de faire correspondre les objets avec ServiceNow.

### Conventions par objet

| Objet | Format | Exemple |
| --- | --- | --- |
| Équipe | `DIR-EQUIPE` | `DTI-INFRA-INFONUAGIQUE` |
| Service technique | `DIR-EQUIPE-Application-Composant` | `DTI-INFRA-PortailClient-API` |
| Service d'affaires | Nom compris par les clients internes | `Portail citoyen` |
| Horaire d'astreinte | `DIR-EQUIPE-Astreinte-[Primaire/Secondaire]` | `DTI-INFRA-Astreinte-Primaire` |
| Politique d'escalade | `DIR-EQUIPE-EP-[Criticité]` | `DTI-INFRA-EP-Critique` |
| Intégration | `[Outil]-[Environnement]` | `AppDynamics-PROD` |
| Ensemble de règles (Event Orchestration) | `DIR-EQUIPE-Orchestration` | `DTI-INFRA-Orchestration` |

Les codes de direction et d'équipe proviennent de la liste maintenue par l'équipe PagerDuty dans la bibliothèque « Documents de référence ».

### Correspondance avec ServiceNow

- Chaque service technique PagerDuty correspond à un service ou élément de configuration (CI) dans la CMDB ServiceNow.
- Chaque équipe PagerDuty correspond à un groupe d'affectation ServiceNow.
- Le nom ou l'identifiant ServiceNow est inscrit dans la description du service PagerDuty.

### Niveaux d'urgence et de priorité

| Priorité | Description | Urgence PagerDuty | Notification |
| --- | --- | --- | --- |
| P1 — Critique | Service essentiel indisponible pour la clientèle | Haute | Immédiate, 24/7 |
| P2 — Majeure | Dégradation importante, contournement difficile | Haute | Immédiate, 24/7 |
| P3 — Modérée | Impact limité, contournement possible | Basse | Heures ouvrables |
| P4 — Mineure | Aucun impact client | Basse | Heures ouvrables, sans appel |

Cette grille doit être alignée sur la matrice de priorité ServiceNow existante (à confirmer).

### Normes obligatoires

- [ ] Chaque équipe compte au moins 2 gestionnaires et 3 intervenants.
- [ ] Chaque service a une description, un propriétaire et un lien vers son guide d'intervention (runbook).
- [ ] Chaque politique d'escalade a au moins 2 niveaux ; le dernier niveau est un gestionnaire.
- [ ] Le délai d'escalade ne dépasse pas 30 minutes pour un service critique.
- [ ] Aucun horaire ne contient de période sans couverture pour un service 24/7.
- [ ] Les comptes utilisent le SSO ; les coordonnées comprennent au moins l'application mobile et un numéro de téléphone.
- [ ] Les alertes ne contiennent aucun renseignement personnel, fiscal ou confidentiel.

Les valeurs chiffrées ci-dessus sont des propositions à valider par le comité de gouvernance.

## Page 4 — Bonnes pratiques

Une bonne configuration PagerDuty repose sur trois règles : n'envoyer que des alertes actionnables, toujours prévoir une escalade, et partager équitablement la charge d'astreinte.

### Alertes : réduire le bruit

- Ne réveillez quelqu'un que si une action humaine est requise maintenant. Le reste passe en urgence basse ou en simple notification.
- Activez le regroupement intelligent (Intelligent Alert Grouping) ou basé sur le contenu pour éviter 50 incidents pour une même panne.
- Utilisez l'orchestration d'événements pour supprimer les alertes de test, les doublons et les maintenances planifiées.
- Planifiez les fenêtres de maintenance dans PagerDuty avant chaque changement.
- Rédigez des titres clairs : quoi, où, gravité. Exemple : « PortailClient-API PROD — temps de réponse > 5 s ».
- Revoyez chaque mois les 10 alertes les plus fréquentes : corriger le seuil, la cause ou supprimer l'alerte.

### Horaires d'astreinte

- Prévoyez un niveau primaire et un niveau secondaire.
- Rotation hebdomadaire recommandée, avec relève en semaine (ex. mardi 9 h) plutôt que le lundi.
- Saisissez congés et remplacements (overrides) dans l'outil, pas par courriel.
- Visez au moins 4 à 5 personnes par rotation pour limiter la fatigue.
- Respectez les conventions collectives et politiques internes de rémunération d'astreinte.

### Politiques d'escalade

- Niveau 1 : intervenant primaire, escalade après 15 minutes sans accusé de réception.
- Niveau 2 : intervenant secondaire, escalade après 15 minutes.
- Niveau 3 : gestionnaire de l'équipe.
- Répétez la politique (2 ou 3 cycles) pour les services critiques.

### Réponse aux incidents

1. **Accuser réception** rapidement, même si l'analyse commence plus tard.
2. **Évaluer** l'impact et ajuster la priorité.
3. **Mobiliser** au besoin : ajouter des intervenants ou d'autres équipes.
4. **Communiquer** : mises à jour d'état aux parties prenantes à intervalles réguliers.
5. **Résoudre** et documenter les actions dans la note de l'incident.
6. **Revue post-incident** pour tout P1 et P2, dans les 5 jours ouvrables.

### Guides d'intervention (runbooks)

Chaque service critique doit avoir un guide lié : description du service, vérifications initiales, actions de rétablissement, contacts et escalade. Le lien est inscrit dans la description du service.

### Sécurité et confidentialité

- Aucun renseignement personnel, fiscal ou confidentiel dans les alertes, notes ou titres.
- Ne partagez jamais votre compte ; utilisez le SSO.
- Activez l'authentification sur l'application mobile (NIP ou biométrie).
- Signalez tout départ ou changement de poste à votre gestionnaire d'équipe PagerDuty.

Les délais indiqués sont des valeurs de départ à ajuster par chaque direction selon la criticité de ses services.

## Page 5 — Guide d'adhésion pour les directions

Votre direction veut utiliser PagerDuty ? L'équipe PagerDuty vous accompagne de la demande à la mise en service, puis pendant les 30 premiers jours.

```mermaid
flowchart LR
    A[Demande d'adhésion] --> B[Analyse par le CoE]
    B --> C{Prérequis OK ?}
    C -- non : compléter la demande --> A
    C -- oui --> D[Atelier de démarrage]
    D --> E[Configuration, tests]
    E --> F[[Mise en service]]
    F --> G[Suivi de 30 jours]
```

Une demande incomplète revient à la direction ; une fois les prérequis réunis, le CoE planifie l'atelier de démarrage.

### PagerDuty est-il pour vous ?

PagerDuty convient si votre équipe :

- exploite des services qui doivent être rétablis rapidement, en heures ouvrables ou 24/7 ;
- assure ou doit assurer une astreinte ;
- reçoit des alertes d'outils de surveillance (AppDynamics, journaux, infonuagique, etc.) ;
- veut réduire le bruit d'alertes et mieux suivre ses incidents.

### Prérequis

- [ ] Un responsable de direction qui approuve l'adhésion et les licences.
- [ ] Un ou deux gestionnaires d'équipe PagerDuty désignés.
- [ ] La liste des membres et de leur rôle (intervenant ou observateur).
- [ ] La liste des services à surveiller, avec leur criticité et leur correspondance ServiceNow.
- [ ] Les sources d'alertes à intégrer.
- [ ] Un horaire d'astreinte cible (heures ouvrables ou 24/7).

### Étapes

1. **Faire une demande** : remplir le formulaire d'adhésion (bouton ci-dessous).
2. **Analyse** : l'équipe PagerDuty valide les prérequis et la disponibilité des licences.
3. **Atelier de démarrage** (environ 1 h 30) : revue des normes, conception des services, horaires et escalades.
4. **Configuration et tests** : création de l'équipe et des objets selon les normes, branchement des intégrations, test d'alerte de bout en bout, installation de l'application mobile par chaque intervenant.
5. **Mise en service** : activation des notifications réelles.
6. **Suivi de 30 jours** : revue du bruit d'alertes et ajustements avec le CoE.

### Bouton d'action

**[ Faire une demande d'adhésion ]** → lien vers le formulaire ou l'article du catalogue ServiceNow.

### Formation

- Séance d'introduction pour les intervenants (45 min).
- Séance pour les gestionnaires d'équipe (1 h 30) : horaires, escalades, remplacements, rapports.
- Ressources : PagerDuty University et la bibliothèque « Documents de référence ».

## Page 6 — Intégrations

Toutes les intégrations sont configurées ou approuvées par l'équipe PagerDuty, afin d'assurer la sécurité des clés, la cohérence du routage et la synchronisation avec ServiceNow.

### Catalogue des intégrations

| Outil | Rôle | Sens | Statut | Remarques |
| --- | --- | --- | --- | --- |
| ServiceNow | Gestion des incidents (ITSM), CMDB | Bidirectionnel | Disponible | Création et mise à jour des incidents ServiceNow à partir de PagerDuty ; correspondance services ↔ CI |
| AppDynamics | Surveillance de la performance applicative | Vers PagerDuty | Disponible | Une intégration par application ou environnement |
| Courriel | Source d'alertes générique | Vers PagerDuty | Sur approbation | À éviter si une intégration native existe |
| Microsoft Teams | Collaboration pendant l'incident | Bidirectionnel | À confirmer | Notifications et actions depuis Teams |
| Autres outils (Azure Monitor, journaux, etc.) | Surveillance | Vers PagerDuty | Sur demande | Analyse par le CoE |

Le statut de chaque ligne est à valider avec l'état réel de la plateforme.

### Règles d'intégration

- Une clé d'intégration par source et par environnement ; aucune clé partagée entre équipes.
- Les clés sont stockées dans le coffre de secrets approuvé, jamais dans le code ou un courriel.
- Les environnements hors production n'envoient pas d'alertes de haute urgence.
- Les événements passent par l'orchestration d'événements (Event Orchestration) pour le routage, la priorité et la suppression.
- Chaque nouvelle intégration est testée de bout en bout avant la mise en service.
- Le contenu des alertes est filtré : aucun renseignement personnel ou confidentiel.

### Demander une nouvelle intégration

Remplissez le formulaire de demande en précisant l'outil source, les services visés, l'environnement et le volume d'alertes estimé. Le CoE évalue la faisabilité et la sécurité, puis planifie la configuration avec votre gestionnaire d'équipe.

## Page 7 — FAQ et soutien

### Questions fréquentes

**Combien coûte l'adhésion pour ma direction ?**
Les licences sont gérées centralement par l'équipe PagerDuty. Le modèle de répartition des coûts est précisé lors de l'analyse de votre demande.

**Quelle est la différence entre un intervenant et une partie prenante ?**
L'intervenant reçoit les alertes et traite les incidents. La partie prenante consulte l'état des services et reçoit des mises à jour, sans être appelée.

**Je pars en congé, comment me faire remplacer ?**
Créez un remplacement (override) dans l'horaire ou demandez-le à votre gestionnaire d'équipe PagerDuty.

**Je reçois trop d'alertes. Que faire ?**
Signalez-le à votre gestionnaire d'équipe. Consultez la section « Alertes : réduire le bruit » de la page Bonnes pratiques ; le CoE peut vous aider à ajuster l'orchestration.

**PagerDuty remplace-t-il ServiceNow ?**
Non. PagerDuty gère l'alerte, l'astreinte et la mobilisation. ServiceNow demeure le système de référence pour les incidents, les problèmes et les changements.

**Puis-je créer moi-même une équipe ou un service ?**
Les gestionnaires d'équipe peuvent créer des services dans leur équipe en respectant les normes. La création d'une nouvelle équipe passe par l'équipe PagerDuty.

**Comment configurer mes notifications ?**
Installez l'application mobile, connectez-vous par SSO et définissez au moins deux méthodes (notification push et appel téléphonique) pour les incidents de haute urgence.

**Que faire si je quitte l'équipe ou l'organisation ?**
Votre gestionnaire d'équipe retire votre compte des horaires ; l'accès est désactivé selon le processus de départ.

### Obtenir de l'aide

| Besoin | Canal | Délai visé |
| --- | --- | --- |
| Question générale | [Boîte courriel de l'équipe] ou canal Teams | À définir |
| Problème d'accès ou de configuration | Demande ServiceNow, groupe [à préciser] | À définir |
| Panne de la plateforme PagerDuty | Incident ServiceNow, priorité selon l'impact | Selon la priorité |
| Nouvelle adhésion ou intégration | Formulaire d'adhésion | À définir |

### Liens utiles

- État de la plateforme PagerDuty : status.pagerduty.com
- Documentation officielle : support.pagerduty.com
- Formation en ligne : PagerDuty University
- Bibliothèque « Documents de référence » du site

## Annexes

### A. Champs du formulaire de demande d'adhésion

| Champ | Type | Obligatoire |
| --- | --- | --- |
| Direction et équipe | Liste | Oui |
| Responsable de direction (approbateur) | Personne | Oui |
| Gestionnaires d'équipe PagerDuty (1 ou 2) | Personne | Oui |
| Nombre d'intervenants et d'observateurs | Nombre | Oui |
| Services à surveiller et criticité | Texte | Oui |
| Correspondance ServiceNow (groupe d'affectation, CI) | Texte | Oui |
| Sources d'alertes à intégrer | Choix multiples | Oui |
| Couverture requise (heures ouvrables ou 24/7) | Choix | Oui |
| Date de mise en service souhaitée | Date | Non |
| Commentaires | Texte | Non |

### B. Liste de contrôle de mise en service

- [ ] Équipe créée selon la convention de nommage
- [ ] Membres ajoutés avec le bon rôle et connexion SSO validée
- [ ] Application mobile installée et notifications testées par chaque intervenant
- [ ] Horaires primaire et secondaire sans trou de couverture
- [ ] Politique d'escalade d'au moins 2 niveaux reliée à chaque service
- [ ] Services décrits, avec propriétaire et lien vers le guide d'intervention
- [ ] Intégrations branchées et alerte de test reçue de bout en bout
- [ ] Synchronisation ServiceNow vérifiée
- [ ] Règles de suppression et fenêtres de maintenance configurées
- [ ] Revue de suivi planifiée à 30 jours

### C. Gabarit de communication aux directions

**Objet : PagerDuty — un service offert à votre direction pour la gestion des alertes et de l'astreinte**

Bonjour,

L'équipe PagerDuty met à la disposition des directions une plateforme organisationnelle de gestion des alertes, de l'astreinte et de la réponse aux incidents, intégrée à ServiceNow et à nos outils de surveillance.

PagerDuty permet d'aviser automatiquement la bonne personne, d'escalader sans délai et de réduire le bruit d'alertes. Notre équipe vous accompagne de la demande à la mise en service.

Pour en savoir plus et faire une demande, consultez notre site : [lien SharePoint].

Une séance d'information est prévue le [date]. Inscription : [lien].

L'équipe PagerDuty

### D. Points à confirmer avant publication

- Codes officiels des directions et des équipes pour le nommage.
- Matrice de priorité ServiceNow à aligner.
- Modèle de répartition des coûts des licences.
- Canaux de soutien et délais visés.
- Statut réel de chaque intégration du catalogue.
- Approbation du comité de gouvernance pour les normes chiffrées.
