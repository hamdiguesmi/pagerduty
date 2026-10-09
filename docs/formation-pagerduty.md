# Formation complète PagerDuty — Fonctionnalités et préparation de l'appel d'offres

Oct 8, 2026 · @Hamdi

## Objectifs de la formation

Cette formation donne à un architecte d'exploitation une maîtrise fonctionnelle complète de PagerDuty et les éléments nécessaires pour rédiger les exigences d'un appel d'offres. Deux constats structurent tout le reste : l'offre commerciale a été refondue autour d'une nouvelle plateforme (PD Reliability Platform), et l'hébergement se limite aux régions US et EU.

**Public visé :** architectes d'exploitation, responsables NOC/centre d'opérations, SRE, gestionnaires d'incidents ITIL, acheteurs TI.

**À la fin, vous saurez :**

- expliquer le modèle de données (événement → alerte → incident) et le cycle de vie d'un incident;
- concevoir horaires, rotations et politiques d'escalade;
- configurer l'orchestration d'événements et la réduction du bruit;
- structurer la réponse aux incidents majeurs (rôles, workflows, communication);
- situer l'automatisation et les agents IA dans l'offre;
- distinguer ce qui est inclus, en option ou réservé à certains forfaits;
- formuler des exigences mesurables et les questions à poser au fournisseur.

**Plan :** 14 modules, du fonctionnel (modules 1 à 10) au contractuel (modules 11 à 13), puis l'implantation (module 14). Durée indicative : 2 jours en salle, ou 6 séances de 2 h.

## Module 1 — Présentation de PagerDuty

PagerDuty est une plateforme SaaS de gestion des opérations numériques : elle reçoit les signaux des outils de supervision, réveille la bonne personne, coordonne la réponse et mesure l'amélioration. Ce n'est pas un outil de supervision : il ne collecte ni métriques ni journaux, il les reçoit de Datadog, Dynatrace, Splunk, Azure Monitor, Prometheus, etc.

### Les grandes familles de produits

| Produit | Rôle | Remarque |
| --- | --- | --- |
| [Incident Management](https://www.pagerduty.com/platform/incident-management/) | Astreinte, escalade, notifications, réponse, post-mortem | Cœur historique |
| [AIOps](https://www.pagerduty.com/platform/aiops/) (Signal Intelligence) | Orchestration avancée, regroupement, suppression du bruit, analyse de cause | Add-on, ou inclus dans PD Reliability Platform |
| [Automation](https://www.pagerduty.com/platform/automation/) | Runbook Automation (issu de Rundeck), actions d'automatisation | Add-on |
| [PagerDuty Advance](https://www.pagerduty.com/platform/generative-ai/) | IA générative et agents (SRE, Scribe, Shift, Insights) | Add-on, ou « AI Actions » dans PD Reliability Platform |
| [Status Pages](https://www.pagerduty.com/platform/business-ops/status-pages/) | Pages de statut internes, externes, privées | Selon forfait |
| [Customer Service Ops](https://www.pagerduty.com/platform/business-ops/) | Pont entre soutien à la clientèle et TI | Intégrations Zendesk, Salesforce, ServiceNow CSM |
| PD Reliability Platform | Nouvelle offre phare regroupant tout ce qui précède | Tarification par plateforme et crédits |

### Les quatre étapes du cycle opérationnel

PagerDuty organise ses fonctionnalités selon quatre étapes, reprises dans sa grille tarifaire :

1. **Détecter et trier** (Detect and Triage) — ingestion, déduplication, routage, sévérité.
2. **Mobiliser et automatiser** (Mobilize and Automate) — astreinte, workflows, rôles, chat.
3. **Communiquer** (Communicate) — parties prenantes, pages de statut, gabarits.
4. **Apprendre** (Learn) — post-incident reviews, analytique, maturité opérationnelle.

### Positionnement par rapport à l'ITSM

PagerDuty ne remplace pas ServiceNow ou Jira Service Management. L'ITSM reste le système d'enregistrement (incident ITIL, problème, changement); PagerDuty est le système d'engagement (qui est réveillé, quand, et comment la réponse se coordonne). L'intégration bidirectionnelle entre les deux est un point clé de l'appel d'offres (module 10).

## Module 2 — Modèle de données et cycle de vie

Tout dans PagerDuty part d'un **événement** reçu sur une **intégration**, transformé en **alerte**, puis regroupé dans un **incident** rattaché à un **service**. Bien concevoir les services est la décision d'architecture la plus structurante.

### Objets fondamentaux

| Objet | Définition | Point d'attention |
| --- | --- | --- |
| Événement (event) | Message reçu par l'Events API v2, un courriel ou une intégration | Champs clés : `routing_key`, `event_action` (trigger, acknowledge, resolve), `dedup_key`, `severity`, `payload` |
| Changement (change event) | Événement informatif issu du CI/CD (déploiement, commit) | Ne crée pas d'incident; sert à la corrélation |
| Alerte | Événement accepté et normalisé | Porte la sévérité (critical, error, warning, info) |
| Incident | Unité de travail humain, regroupe une ou plusieurs alertes | Statuts : Triggered → Acknowledged → Resolved |
| Service technique | Composant dont une équipe est responsable | Porte la politique d'escalade et les règles d'urgence |
| Service d'affaires | Capacité métier (ex. « Déclaration en ligne ») | Dépend de services techniques; utile aux parties prenantes |
| Intégration | Point d'entrée d'un outil de supervision | Clé d'intégration par service, ou clé globale (Global Integration Key) |
| Équipe (team) | Regroupement d'utilisateurs et d'objets | Base des permissions avancées |

### Cycle de vie d'un incident

1. **Triggered** — l'incident est créé; la politique d'escalade notifie le premier niveau.
2. **Acknowledged** — un intervenant accuse réception; l'escalade s'arrête. Un délai de « ré-déclenchement » peut s'appliquer si rien ne bouge.
3. **Resolved** — résolution manuelle, par API, ou automatique quand l'outil source envoie un `resolve` avec la même `dedup_key`.

### Urgence, sévérité et priorité : trois notions distinctes

- **Sévérité** — attribut de l'alerte, fournie par l'outil source ou par une règle.
- **Urgence** (haute ou basse) — attribut de l'incident qui détermine *comment* on notifie (règles de notification de chaque utilisateur). Configurable par service, selon l'heure (heures ouvrables) ou dynamiquement selon la sévérité.
- **Priorité** (P1 à P5 par défaut, personnalisable) — classement d'affaires de l'incident, aligné sur la matrice impact × urgence ITIL. Disponible selon le forfait (voir module 12).

### Déduplication

La `dedup_key` est le mécanisme qui évite les tempêtes de notifications : tant qu'un incident est ouvert avec cette clé, les nouveaux déclenchements s'ajoutent à l'alerte existante. Exiger que chaque intégration source fournisse une clé stable est une bonne pratique d'architecture.

### Fenêtres de maintenance

Une fenêtre de maintenance empêche la création d'incidents sur un ou plusieurs services pendant une période planifiée. Elle doit être reliée au calendrier des changements (ServiceNow Change, par exemple).

## Module 3 — Gestion des astreintes

L'astreinte repose sur trois objets emboîtés : l'**horaire** dit qui est de garde, la **politique d'escalade** dit dans quel ordre on appelle, et les **règles de notification** de chaque utilisateur disent par quel canal on le joint.

### Horaires (schedules)

- **Couches (layers)** — un horaire empile des couches; la couche la plus haute l'emporte. Exemple : couche 1 = rotation hebdomadaire 24/7, couche 2 = relève de jour en semaine.
- **Rotations** — quotidienne, hebdomadaire ou personnalisée, avec heure de passation et restrictions (heures ouvrables, nuits, fins de semaine).
- **Horaires par quart (shift-based schedules)** — nouveau modèle qui remplace progressivement les horaires « legacy »; à valider pendant la preuve de concept.
- **Dérogations (overrides)** — remplacement ponctuel d'une personne, créé sur le web, le mobile ou depuis le chat.
- **Fuseaux horaires** — chaque horaire a son fuseau; essentiel pour une équipe répartie.
- **Exports** — flux iCal/WebCal vers Outlook ou Google Agenda.

### Politiques d'escalade

- Plusieurs **niveaux**; chaque niveau cible des utilisateurs et/ou des horaires.
- **Délai d'escalade** par niveau (ex. 15 min sans accusé de réception → niveau suivant).
- **Répétition** de la politique jusqu'à N fois si personne ne répond.
- **Round robin** — répartit les incidents à tour de rôle entre les membres d'un niveau (selon forfait).
- Nombre maximal d'utilisateurs par niveau : 5 en forfait Free et Professional, 50 dans PD Reliability Platform.

### Notifications

| Canal | Remarque |
| --- | --- |
| Notification push (application mobile iOS/Android) | Canal privilégié; contourne le mode silencieux en haute urgence si configuré |
| SMS | Inclus dans les forfaits payants; 100/mois en Free |
| Appel vocal | Inclus dans les forfaits payants; permet l'accusé de réception par touche |
| Courriel | Basse urgence surtout |
| WhatsApp | Disponible selon les pays pris en charge |
| Slack / Microsoft Teams | Notification et action dans le fil |

Chaque utilisateur définit des **règles de notification** distinctes pour la haute et la basse urgence (ex. push immédiat, appel à +5 min, SMS à +10 min). L'administrateur peut imposer un minimum via des rapports de préparation (On-call Readiness).

### Live Call Routing

Numéro de téléphone dédié qui achemine un appel entrant vers la personne de garde, selon l'horaire et l'escalade. Utile pour un centre de services qui doit joindre l'astreinte sans connaître son nom. Add-on en forfait Professional; 1 à 3 lignes incluses dans PD Reliability Platform.

### Agent Shift

L'agent IA **Shift** synchronise horaires et congés du calendrier, détecte les conflits et propose un remplaçant directement dans le chat (module 7).

## Module 4 — Event Orchestration et AIOps

L'orchestration d'événements décide, avant toute notification, où va un événement et sous quelle forme. La version de base est incluse partout; la version avancée et la réduction du bruit par apprentissage machine exigent l'add-on AIOps ou la PD Reliability Platform (appelée alors **Signal Intelligence**).

### Les deux niveaux d'orchestration

| Niveau | Où | Rôle |
| --- | --- | --- |
| Orchestration globale | Clé d'intégration globale | Routage vers le bon service, enrichissement commun, suppression transverse |
| Orchestration de service | Chaque service | Règles propres à l'équipe : sévérité, urgence, priorité, pause, automatisation |

Les règles s'écrivent avec des conditions ET/OU, des expressions régulières et des blocs SI/SINON. L'ancien mécanisme **Rulesets** est en voie de remplacement; tout nouveau déploiement doit utiliser Event Orchestration.

### Capacités incluses dans tous les forfaits

- clé d'intégration globale, routage vers un service;
- déduplication à l'ingestion;
- règles de service, conditions de base, SI/SINON;
- enrichissement de base : priorité, sévérité, `event_action`.

### Capacités AIOps / Signal Intelligence

**Réduction du bruit**

- **Suppression d'alertes** — l'alerte est conservée mais ne notifie personne.
- **Seuils** — n'alerter qu'au-delà de N occurrences en X minutes.
- **Regroupement par fenêtre de temps** (time-based).
- **Regroupement par contenu** (content-based) — sur des champs choisis (hôte, application).
- **Regroupement intelligent** (intelligent) — modèle ML entraîné sur l'historique de chaque service.
- **Regroupement unifié** — ML + règles de contenu combinés.
- **Regroupement global** — entre plusieurs services techniques.
- **Pause des notifications** et **pause automatique** — laisser le temps à une alerte instable (flapping) de se résoudre seule.

**Orchestration avancée**

- extraction de variables depuis la charge utile et écriture dans des champs personnalisés;
- variables de cache (mémoriser des données d'un événement à l'autre);
- routage dynamique vers un service ou une politique d'escalade selon le contenu;
- déclenchement d'actions d'autoréparation;
- **AI Orchestrations** — règles assistées par IA (module 7).

**Analyse de cause**

- **Incidents passés** et **incidents connexes** (ML, entre équipes);
- **Incidents atypiques** (outlier);
- **Origine probable** (probable origin) — où regarder en premier;
- **Corrélation des changements** — lien entre un déploiement et un incident.

**Visibilité**

- **Operations Console** — console partagée de type NOC, filtrable, pour piloter les incidents critiques.

### Indicateur à exiger

Demander au fournisseur un taux de compression mesuré sur vos propres données pendant la preuve de concept (événements reçus ÷ incidents créés), plutôt qu'un chiffre marketing.

## Module 4 bis — Cas pratiques AIOps avec AppDynamics, ServiceNow et autres sources

Le modèle cible : toutes les sources (Splunk AppDynamics, infrastructure, journaux, cloud) envoient leurs événements à **une clé d'intégration globale**; l'orchestration globale normalise et route vers le bon service; l'AIOps regroupe et supprime le bruit; seul l'incident qualifié est synchronisé avec ServiceNow. Les noms d'applications, de services et de groupes ci-dessous sont des exemples à remplacer par les vôtres.

### Flux de bout en bout

1. **Sources** — AppDynamics (violations de health rules), outils d'infrastructure et de journaux, cloud, CI/CD (événements de changement), ServiceNow Change (changements approuvés).
2. **Clé globale** — un seul point d'entrée Events API v2 (`events.pagerduty.com/v2/enqueue`).
3. **Orchestration globale** — normalisation des champs (application, environnement, CI), suppression des environnements hors production, routage vers le service technique.
4. **Orchestration de service** — sévérité → urgence et priorité, seuils, pause, actions d'automatisation.
5. **Regroupement AIOps** — intelligent, par contenu ou global entre services.
6. **Incident** — notification de l'astreinte, analyse de cause (origine probable, changements récents).
7. **ServiceNow** — création ou mise à jour de l'incident ITSM, affectation au groupe, synchronisation des états et notes.

### Étape 1 — Brancher AppDynamics sur l'Events API v2

Le [guide officiel](https://www.pagerduty.com/docs/guides/appdynamics-integration-guide/) utilise encore l'ancien point d'accès v1 (`generic/2010-04-15/create_event.json`). Pour bénéficier de l'orchestration et de l'AIOps, créer plutôt dans AppDynamics (*Alert & Respond → HTTP Request Templates*) un gabarit **v2** : méthode POST, type `application/json`, option « One Request Per Event », variables `pd_routing_key` et `pd_event_action` (`trigger` ou `resolve`).

```json
{
  "routing_key": "${pd_routing_key}",
  "event_action": "${pd_event_action}",
  "dedup_key": "appd-${latestEvent.application.name}-${latestEvent.tier.name}-${latestEvent.healthRule.name}",
  "payload": {
    "summary": "${latestEvent.displayName} — ${latestEvent.application.name} / ${latestEvent.tier.name}",
    "source": "${latestEvent.node.name}",
    "severity": "${latestEvent.severity}",
    "component": "${latestEvent.tier.name}",
    "group": "${latestEvent.application.name}",
    "class": "${latestEvent.eventType}",
    "custom_details": {
      "outil": "appdynamics",
      "application": "${latestEvent.application.name}",
      "tier": "${latestEvent.tier.name}",
      "health_rule": "${latestEvent.healthRule.name}",
      "message": "${latestEvent.summaryMessage}"
    }
  },
  "links": [{ "href": "${latestEvent.deepLink}", "text": "Ouvrir dans AppDynamics" }]
}
```

Créer deux actions sur le même gabarit (« PD Trigger » et « PD Resolve »), puis deux politiques : *Health Rule Violation Started* → Trigger, *Health Rule Violation Ended* → Resolve. La `dedup_key` construite sur application + tier + health rule garantit que le `resolve` ferme le bon incident. À valider : la sévérité AppDynamics (`CRITICAL`, `WARNING`, `INFO`) doit être convertie en `critical`, `warning`, `info` (règle d'orchestration ci-dessous) et le nom exact des variables de gabarit selon votre version du contrôleur.

### Étape 2 — Orchestration globale : normaliser et router

Exemple en Terraform (fournisseur `PagerDuty/pagerduty`), à adapter :

```hcl
# Routage par application AppDynamics vers le service technique
resource "pagerduty_event_orchestration_router" "global" {
  event_orchestration = pagerduty_event_orchestration.operations.id
  set {
    id = "start"
    rule {
      label = "AppD - Portail citoyen"
      condition { expression = "event.custom_details.application matches part 'PORTAIL'" }
      actions { route_to = pagerduty_service.portail_web.id }
    }
    rule {
      label = "AppD - Paiements"
      condition { expression = "event.custom_details.application matches part 'PAIEMENT'" }
      actions { route_to = pagerduty_service.paiements_api.id }
    }
  }
  catch_all { actions { route_to = pagerduty_service.noc_triage.id } }
}
```

Règles globales typiques :

| Règle | Condition | Action |
| --- | --- | --- |
| Hors production | `custom_details.application` contient « DEV », « QA » ou « UAT » | Supprimer (conservé, sans notification) |
| Sévérité AppD | `severity` = `WARNING` | Fixer sévérité `warning`, urgence basse |
| Sévérité AppD | `severity` = `CRITICAL` | Fixer sévérité `critical` |
| Extraction | Regex sur `summary` (ex. nom de base de données) | Écrire dans un champ personnalisé `bd_cible` |
| Filet de sécurité | Aucune règle ne correspond | Router vers un service « NOC – triage » |

### Étape 3 — Scénarios AIOps concrets

| No | Situation | Sans AIOps | Configuration PagerDuty | Résultat attendu |
| --- | --- | --- | --- | --- |
| 1 | Une base Oracle ralentit; AppDynamics lève des violations sur 6 tiers qui l'appellent | 6 incidents, 6 équipes réveillées | Regroupement **global** entre services, par contenu sur `bd_cible`, fenêtre 10 min | 1 incident, affecté à l'équipe BD; les autres voient l'incident connexe |
| 2 | Health rule « temps de réponse » instable qui s'ouvre et se ferme toutes les 3 minutes | Dizaines de notifications nocturnes | **Pause des notifications** 5 min sur ce service, ou pause automatique ML | Notification seulement si la violation persiste |
| 3 | Le même problème est vu par AppDynamics (application) et par l'outil d'infrastructure (CPU du serveur) | 2 incidents sur 2 services | Regroupement **intelligent** + **unifié** sur `source` (nom d'hôte) | 1 incident avec 2 alertes; contexte complet |
| 4 | Pic d'erreurs juste après un déploiement | L'équipe cherche la cause | **Événements de changement** du pipeline CI/CD et des changements ServiceNow; **corrélation des changements** | L'incident affiche « changement récent sur ce service » en tête |
| 5 | Fenêtre de maintenance planifiée dans ServiceNow | Alertes pendant l'intervention | Fenêtre de maintenance PagerDuty créée par workflow ou API à partir du changement approuvé | Aucune notification pendant la fenêtre |
| 6 | 50 violations « Warning » de capacité disque par jour | Fatigue d'alerte | **Seuil** : notifier seulement à 5 occurrences en 30 min; sinon basse urgence | Le bruit va dans un rapport, pas dans un téléphone |
| 7 | Incident récurrent connu (pool applicatif saturé) | Diagnostic manuel à chaque fois | Action d'automatisation déclenchée par la règle de service (diagnostic Runbook Automation) | Résultat du diagnostic dans la chronologie avant que l'intervenant ouvre l'incident |
| 8 | Incident inhabituel jamais vu | Escalade lente | **Incidents atypiques**, **origine probable**, **incidents passés**; agent SRE si acheté | Piste de diagnostic dès l'ouverture |

### Étape 4 — Boucle avec ServiceNow

| Élément | Recommandation |
| --- | --- |
| Sens de création | PagerDuty crée l'incident ServiceNow une fois l'incident qualifié (après regroupement), pas à chaque événement |
| Correspondance | Service PagerDuty ↔ CI / service d'affaires de la CMDB; politique d'escalade ↔ groupe d'affectation |
| Priorité | Priorité PagerDuty (P1–P5) ↔ priorité ITIL ServiceNow (impact × urgence) |
| États | Accusé de réception, résolution et notes synchronisés dans les deux sens |
| Champs personnalisés | `application`, `bd_cible`, lien AppDynamics copiés dans l'incident ServiceNow (forfait Plus ou Ultimate) |
| Changements | Demandes de changement ServiceNow envoyées comme événements de changement pour la corrélation |
| Problèmes | Actions de suivi des post-incident reviews converties en problèmes ITIL |

### Indicateurs à mesurer pendant la preuve de concept

| Indicateur | Calcul | Source |
| --- | --- | --- |
| Taux de compression | Événements reçus ÷ incidents créés | Event Analytics |
| Taux de suppression | Alertes supprimées ÷ alertes totales | Event Analytics |
| Incidents ServiceNow évités | Incidents ITSM avant ÷ après | ServiceNow |
| Interruptions hors heures | Notifications de nuit par intervenant par semaine | Insights |
| MTTA et MTTR | Avant et après, sur les mêmes services | Insights |

Conseil : rejouer 2 à 4 semaines d'événements AppDynamics réels dans un compte d'essai avant la décision; c'est le seul moyen de valider les gains de regroupement promis.

## Module 5 — Réponse aux incidents

Pour un incident majeur, PagerDuty fournit la structure (types, rôles, tâches), l'automatisation du rituel (Incident Workflows) et le lieu de travail (Slack, Teams, pont de conférence). La plupart de ces capacités sont plafonnées en forfait Professional et complètes dans PD Reliability Platform Plus et Ultimate.

### Structurer l'incident

| Capacité | Description | Disponibilité |
| --- | --- | --- |
| Types d'incidents (Incident Types) | Catégories (ex. sécurité, majeur, données) avec champs et workflows propres | 3 prédéfinis en Professional; 100 personnalisés en Plus/Ultimate |
| Rôles d'incident | Commandant d'incident, scribe, etc. | 2 prédéfinis en Professional; jusqu'à 10 personnalisés en Plus/Ultimate |
| Tâches d'incident | Liste d'actions assignées (aviser la direction, ouvrir le billet ITSM) | Plus/Ultimate |
| Champs personnalisés d'incident | Données d'affaires sur l'incident | Toutes les offres PD Reliability Platform |
| Priorité d'incident | P1–P5 personnalisables | PD Reliability Platform |
| Ajout dynamique d'intervenants | Mobiliser une autre équipe en cours d'incident | PD Reliability Platform |
| Quick Declare | Déclaration rapide d'un incident majeur | À valider en démonstration |
| Champs requis à la résolution | Forcer la saisie (cause, catégorie) avant de fermer | À valider en démonstration |

### Incident Workflows

Constructeur sans code ou à faible code qui exécute une suite d'actions quand une condition est remplie (ex. priorité P1 sur un service critique).

- **Déclencheurs** : conditionnel (création, changement de priorité), manuel (bouton ou commande chat), API, champ personnalisé (Plus/Ultimate).
- **Actions courantes** : créer et lier un canal Slack ou Teams, lancer une réunion Zoom/Teams, ajouter des intervenants, envoyer une mise à jour de statut, créer un billet Jira ou ServiceNow, archiver le canal après l'incident.
- **Logique** : conditions, boucles et délais (Plus/Ultimate).
- **Actions premium** : diagnostics, redémarrage de service, redéploiement de conteneur, demande d'approbation, sans acheter Runbook Automation (PD Reliability Platform).
- **Limite** : 1 workflow en forfait Professional — un point bloquant pour un centre d'opérations.

### Expérience chat (ChatOps)

Gestion de l'incident de bout en bout depuis Slack ou Microsoft Teams : commandes slash, accusé de réception, escalade, invocation des workflows et de l'assistant IA. L'ingestion automatique du canal dans la post-incident review est réservée à Slack. Pour une organisation sur Microsoft 365, faire démontrer la parité fonctionnelle Teams.

### Pont de conférence

Zoom, Microsoft Teams, Google Meet; « One Touch to Join » pour rejoindre en un geste (PD Reliability Platform). L'agent **Scribe** transcrit l'appel Zoom et publie des résumés dans le canal.

### Communication aux parties prenantes

- **Licences de parties prenantes** (stakeholders) — utilisateurs en lecture qui reçoivent les mises à jour sans être intervenants; incluses dans les sièges PD Reliability Platform.
- **Abonnement aux services d'affaires** — un gestionnaire s'abonne à « Paiement en ligne » et reçoit les mises à jour.
- **Gabarits de mise à jour** par public (direction, soutien, clientèle).
- **Tableaux de bord de statut** web et mobile pour les employés.

### Pages de statut

| Type | Public | Disponibilité |
| --- | --- | --- |
| Interne | Employés | PD Reliability Platform |
| Externe | Public | 250 abonnés en Professional; inclus dans PD Reliability Platform |
| Privée | Utilisateurs authentifiés (partenaires) | PD Reliability Platform |

## Module 6 — Automatisation

PagerDuty propose trois étages d'automatisation, du plus simple au plus puissant; seul Runbook Automation exécute des tâches arbitraires dans votre infrastructure, et il reste un add-on dans toutes les offres.

| Étage | Ce qu'il fait | Où ça s'exécute | Disponibilité |
| --- | --- | --- | --- |
| Custom Incident Actions | Bouton sur l'incident qui appelle une URL externe (webhook) | Système externe | Tous les forfaits |
| Actions premium des Incident Workflows | Diagnostics, redémarrage, redéploiement, approbation | Connecteurs PagerDuty | PD Reliability Platform |
| [Runbook Automation](https://www.pagerduty.com/platform/automation/runbook/) | Tâches et workflows complets (scripts, Ansible, PowerShell, API cloud) | Runners dans votre réseau | Add-on partout |

### Runbook Automation (issu de Rundeck)

- **Jobs** — séquences d'étapes (commandes, scripts, appels API, modules Ansible) avec options saisies par l'utilisateur.
- **Nœuds** — inventaire des serveurs cibles (sources : AWS, Azure, VMware, fichiers, CMDB).
- **Runners** — agents installés dans votre réseau, qui initient la connexion sortante vers le SaaS; aucun port entrant à ouvrir. Point clé pour la sécurité réseau.
- **Contrôle d'accès** — politiques ACL par projet, par job, par nœud; journal d'exécution complet.
- **Gestion des secrets** — magasin de clés intégré ou coffre externe (HashiCorp Vault, CyberArk, Azure Key Vault : à confirmer par le fournisseur).
- **Versions** — SaaS (Runbook Automation) ou auto-hébergé (Runbook Automation Self-Hosted / Rundeck Enterprise). L'auto-hébergé permet de garder l'exécution entièrement sur site.

### Automation Actions

Lien entre un incident et un job de Runbook Automation : l'intervenant (ou une règle d'orchestration) lance un diagnostic depuis l'incident, et le résultat s'affiche dans la chronologie. Avec PagerDuty Advance, un résumé IA des journaux d'exécution et des suggestions d'étapes suivantes sont produits.

### Cas d'usage typiques

1. Diagnostic automatique à la création de l'incident (état du service, derniers journaux, espace disque).
2. Autoréparation d'incidents bien connus (redémarrer un pool IIS, vider une file, relancer un pod).
3. Délégation sécurisée : un niveau 1 lance une action approuvée sans accès administrateur au serveur.
4. Tâches planifiées d'exploitation (purges, rotation de certificats).

### Workflow Automation

PagerDuty a acquis Catalytic; l'offre **Workflow Automation** couvre les processus d'affaires sans code au-delà de l'incident. À clarifier avec le fournisseur si elle entre dans le périmètre de l'appel d'offres.

## Module 7 — IA générative et agents IA

L'IA de PagerDuty se vend sous le nom **PagerDuty Advance** (add-on du forfait Professional) ou est incluse comme « AI Actions » dans la PD Reliability Platform. Elle comprend des fonctions d'assistance et quatre agents, dont **Paige**, l'agent SRE qui peut exécuter des remédiations après approbation.

### Fonctions d'assistance (IA générative)

| Fonction | Ce qu'elle produit |
| --- | --- |
| Assistant IA dans Slack et Teams | Réponses en langage naturel : résumé, contexte, état de l'incident |
| Mises à jour de statut | Brouillon adapté au public (direction, clientèle) |
| Post-incident review | Résumé exécutif et premier jet du récit et des actions de suivi |
| Actions d'automatisation | Synthèse des résultats d'un job (journaux, prochaines étapes) |
| Runbooks générés par IA (accès anticipé) | Job d'automatisation à partir d'une consigne en français ou en anglais |
| Analytique Advance | Temps gagné grâce aux fonctions IA, avant/après |

### Les quatre agents

| Agent | Rôle | Autonomie |
| --- | --- | --- |
| [Paige, SRE Agent](https://www.pagerduty.com/platform/ai-agents/sre/) | Classe l'incident, cherche les incidents similaires, consulte observabilité et base de connaissances, recommande puis exécute la remédiation; génère des runbooks qui se mettent à jour | Exécution seulement après approbation humaine; peut agir comme « intervenant virtuel » de premier niveau |
| Scribe Agent | Transcrit les appels Zoom et conversations, publie des résumés structurés dans Slack ou Teams | Lecture et synthèse |
| Shift Agent | Détecte les conflits entre horaire et congés, trouve un remplaçant, crée la dérogation | Modifie les horaires depuis le chat |
| Insights Agent | Répond aux questions sur l'analytique, propose des améliorations proactives | Lecture et recommandation |

Paige s'appuie sur des **connecteurs, outils et compétences** (skills) pour joindre vos outils d'observabilité et vos environnements cloud. Le périmètre exact des connecteurs disponibles est à faire préciser.

### Serveur MCP

PagerDuty expose un serveur **Model Context Protocol** (MCP) distant : des assistants IA tiers ou des IDE peuvent lire et agir sur incidents, services et horaires via l'API. Inclus dès le forfait Professional. À encadrer par des jetons à portée restreinte (Scoped OAuth Apps).

### Questions de gouvernance de l'IA à poser

- Quels grands modèles de langage (LLM) sont utilisés, chez quel fournisseur, dans quelle région?
- Les données de l'organisation servent-elles à entraîner des modèles? Peut-on le refuser contractuellement?
- Peut-on activer les fonctions IA service par service ou équipe par équipe?
- Quelles actions un agent peut-il exécuter sans approbation, et comment sont-elles journalisées?
- Comment se consomment et se mesurent les « AI Actions » dans PD Reliability Platform?
- Les transcriptions de Scribe sont-elles conservées, où et combien de temps?

## Module 8 — Service Graph, services d'affaires et Customer Service Ops

Le graphe de services relie les composants techniques aux capacités métier, ce qui permet de dire en cours d'incident « quels services aux citoyens ou aux clients sont touchés ». C'est le pont entre l'exploitation et la direction.

### Composants

- **Service Directory** — catalogue de tous les services techniques, avec propriétaire, politique d'escalade, intégrations.
- **Profil de service** — onglet Impact montrant dépendances amont et aval.
- **Dépendances** — technique → technique et technique → affaires; gérables par API et Terraform.
- **Dynamic Service Graph** — visualisation interactive de l'écosystème et de l'impact en temps réel.
- **Champs personnalisés de service** — criticité, propriétaire d'affaires, CI ServiceNow associé (PD Reliability Platform).
- **Service Standards** — règles de qualité de configuration (description, escalade multi-niveaux, intégration présente) avec score par équipe.

### Modélisation recommandée

1. Un service technique = un composant déployable avec une seule équipe responsable.
2. Un service d'affaires = une capacité visible par l'usager (ex. portail, prestation, paiement).
3. Aligner les noms sur les CI de la CMDB pour la synchronisation ITSM.
4. Synchroniser le graphe depuis la CMDB plutôt que de le maintenir à la main (API, Terraform ou intégration ServiceNow).

### Customer Service Operations

Permet aux agents du centre de contact de voir dans Zendesk, Salesforce Service Cloud ou ServiceNow CSM qu'un incident technique est en cours, d'y lier des demandes clients et de recevoir les mises à jour, sans être intervenants PagerDuty. Pertinent si un centre d'appels doit savoir rapidement qu'une panne est connue.

## Module 9 — Analytique, post-incident reviews et amélioration continue

PagerDuty mesure la performance de la réponse (MTTA, MTTR, interruptions) et la charge humaine de l'astreinte. L'historique de données accessible est de 3 mois en Free, 1 an en Professional, et complet dans PD Reliability Platform : un critère à fixer contractuellement.

### Indicateurs disponibles

| Indicateur | Ce qu'il mesure |
| --- | --- |
| MTTA | Délai moyen d'accusé de réception |
| MTTR | Délai moyen de résolution |
| Interruptions | Notifications reçues, dont hors heures et pendant le sommeil |
| Escalades | Incidents qui ont dépassé le premier niveau |
| Incidents par service et par équipe | Concentration de la charge |
| Préparation à l'astreinte | Utilisateurs sans méthode de contact, horaires avec trous de couverture |

### Rapports et tableaux de bord

- **Analytics Dashboard** — visualisations interactives des tendances.
- **Insights** — activité des incidents, performance des services, intervenants, équipes, politiques d'escalade, impact d'affaires.
- **Operational Reviews** — revues hebdomadaires ou mensuelles prêtes à présenter; revue de passation d'astreinte (PD Reliability Platform).
- **On-Call Readiness Reports** — trous de couverture et conformité des règles de notification.
- **Event Analytics** — volumes d'événements et efficacité de la réduction du bruit.
- **Operational Maturity** — score de maturité et recommandations.
- **Audit Trail** — historique des modifications de configuration sur 1 an (PD Reliability Platform).
- **Analytics API et export** — alimenter Power BI ou un entrepôt de données.

### Post-incident reviews (Jeli)

PagerDuty a intégré Jeli pour les revues post-incident :

- gabarits personnalisés (1 en Professional, illimités dans PD Reliability Platform);
- **Narrative Builder** — chronologie construite à partir de Slack, de PagerDuty et des transcriptions Scribe;
- collaboration en direct, commentaires, pièces jointes;
- actions de suivi assignées et suivies, exportables vers Jira;
- premier jet généré par IA (avec Advance ou PD Reliability Platform);
- synchronisation vers l'agent SRE pour qu'il apprenne des incidents passés.

### Boucle d'amélioration

1. Revue post-incident sans recherche de coupable pour chaque P1/P2.
2. Actions de suivi en problèmes ITIL dans l'ITSM.
3. Revue opérationnelle mensuelle par équipe (interruptions, bruit, MTTR).
4. Ajustement des règles d'orchestration et des runbooks d'automatisation.

## Module 10 — Intégrations, API et infrastructure en tant que code

PagerDuty revendique plus de 750 intégrations prêtes à l'emploi. Le point critique pour un grand organisme est l'intégration ServiceNow : la version avancée n'existe que dans PD Reliability Platform, et la synchronisation bidirectionnelle des champs personnalisés seulement à partir du forfait Plus.

### Familles d'intégrations

| Famille | Exemples | Sens |
| --- | --- | --- |
| Supervision et observabilité | Datadog, Dynatrace, Splunk, New Relic, Prometheus/Alertmanager, Zabbix, Nagios, SolarWinds | Entrant |
| Cloud | Amazon CloudWatch, EventBridge, GuardDuty, Security Hub; Azure Monitor; Google Cloud | Entrant |
| CI/CD et code | GitHub, GitLab, Jenkins, Bitbucket | Événements de changement |
| ITSM | ServiceNow ITSM, ServiceNow CSM, Jira Service Management, BMC Remedy, Cherwell | Bidirectionnel |
| Billetterie | Jira Software, Jira Cloud et Server, Zendesk | Bidirectionnel |
| Collaboration | Slack, Microsoft Teams, Zoom, Google Meet | Bidirectionnel |
| Portail développeur | Backstage | Lecture |
| Courriel | Intégration par courriel avec filtres | Entrant (dernier recours) |

### ServiceNow : niveaux d'intégration

| Niveau | Contenu | Offre |
| --- | --- | --- |
| Free et Professional | Aucune intégration ITSM avancée (ServiceNow, Remedy, Cherwell); Jira et Zendesk seulement | — |
| ITSM avancé | Synchronisation incident PagerDuty ↔ incident ServiceNow, provisioning des services et groupes, OAuth | PD Reliability Platform (tous les forfaits) |
| Champs personnalisés bidirectionnels et actions avancées | Mappage de champs, types d'incidents synchronisés, réaffectation de service | Plus et Ultimate |
| Demandes de changement | Changements ServiceNow comme événements de changement | À confirmer |

### API

- **Events API v2** — envoi d'événements (`trigger`, `acknowledge`, `resolve`) et d'événements de changement. Format commun PD-CEF.
- **REST API v2** — gestion complète des objets (services, horaires, utilisateurs, incidents). Limites de débit documentées.
- **Webhooks v3** — abonnement sortant aux événements d'incident, signés.
- **Analytics API** — extraction des indicateurs.
- **Jeli API** — post-incident reviews.
- **Scoped OAuth Apps** — jetons à portée limitée, préférables aux clés API globales.
- **Liste d'adresses IP sortantes** (safelist) pour les pare-feu, et **liste d'IP autorisées** pour les sessions utilisateur.
- **PagerDuty Agent** — petit agent local qui met les événements en file d'attente si le lien Internet tombe.

### Infrastructure en tant que code

- **Fournisseur Terraform officiel** (`PagerDuty/pagerduty`) — services, horaires, escalades, orchestrations, équipes, champs personnalisés.
- Bibliothèques clientes (Python, Go, Ruby, Node.js).
- Recommandation : toute configuration de production versionnée dans Git, appliquée par pipeline; l'interface web réservée aux dérogations d'astreinte.

## Module 11 — Sécurité, conformité et résidence des données

PagerDuty n'offre que deux régions d'hébergement, États-Unis et Union européenne, toutes deux sur AWS; il n'existe pas de région canadienne. Pour un organisme public québécois soumis à la Loi 25 et aux exigences d'évaluation des facteurs relatifs à la vie privée (EFVP) pour les communications hors Québec, c'est le risque contractuel numéro un.

### Régions d'hébergement

| Région | Centres de données | URL de l'application | API |
| --- | --- | --- | --- |
| États-Unis | AWS US West (N. California), US West (Oregon), US East (Ohio) | app.pagerduty.com | api.pagerduty.com, events.pagerduty.com |
| Union européenne | AWS EU Central (Francfort), EU West (Irlande) | app.eu.pagerduty.com | api.eu.pagerduty.com, events.eu.pagerduty.com |

Selon la [documentation officielle](https://support.pagerduty.com/main/docs/service-regions) :

- un compte n'existe que dans une seule région; on la choisit à l'inscription;
- une migration de région passe par les services professionnels, généralement payants;
- certaines données sont traitées **hors de la région choisie** : journaux applicatifs, analytique d'usage agrégée, soutien technique, facturation, profils utilisateurs globaux;
- les données envoyées à une intégration tierce peuvent aussi sortir de la région;
- les fournisseurs de téléphonie, SMS et courriel dépendent de la région.

### Données à minimiser

Concevoir les charges utiles d'événements pour qu'elles ne contiennent **aucun renseignement personnel ni fiscal** : identifiants techniques seulement (hôte, service, code d'erreur), liens vers les outils internes pour le détail. Les règles d'orchestration peuvent retirer ou masquer des champs à l'ingestion; à vérifier en preuve de concept.

### Identité et accès

| Capacité | Disponibilité |
| --- | --- |
| SSO SAML (Entra ID, Okta, ADFS, OneLogin) et Google | Professional et plus |
| Provisionnement SCIM (Entra ID, Okta, OneLogin) | À confirmer selon forfait |
| Rôles de base (propriétaire, administrateur, gestionnaire, intervenant, observateur, utilisateur limité, partie prenante) | Tous les forfaits |
| Permissions avancées par équipe, équipes et services privés | Professional et plus |
| Délai d'expiration des sessions | Configurable |
| Liste d'IP autorisées pour les sessions | À confirmer selon forfait |
| Verrouillage par NIP de l'application mobile | PD Reliability Platform |
| Microsoft Intune (gestion des applications mobiles) | Plus et Ultimate |
| Journal d'audit sur 1 an | PD Reliability Platform |

### Certifications à exiger

Des sources tierces indiquent SOC 2 Type II, ISO 27001 et FedRAMP Low; aucun engagement HIPAA n'est affiché. Exiger dans l'appel d'offres les rapports à jour (SOC 2 Type II de moins de 12 mois, certificat ISO 27001 et déclaration d'applicabilité), ainsi que la liste des sous-traitants ultérieurs (sub-processors) et leur pays.

### Disponibilité du service

PagerDuty est lui-même un service critique : s'il tombe, personne n'est réveillé. Exiger :

- un SLA de disponibilité chiffré avec crédits de service;
- les mécanismes de notification de panne PagerDuty (page [status.pagerduty.com](https://status.pagerduty.com/), notifications de panne);
- un plan de contingence interne (liste d'astreinte exportée, numéros de secours).

## Module 12 — Licences, forfaits et add-ons

La [grille officielle](https://www.pagerduty.com/pricing/) présente désormais deux familles : Incident Management (Free et Professional, facturés par utilisateur) et la nouvelle **PD Reliability Platform** (frais de plateforme annuels incluant sièges, événements et AI Actions, complétés par des crédits). Les anciens forfaits Business et Enterprise n'y figurent plus; si votre contrat actuel les utilise, la migration est le sujet central de la négociation. Prix publics en dollars américains, avant négociation.

### Incident Management (par utilisateur)

| Forfait | Prix (USD) | Contenu essentiel | Limites notables |
| --- | --- | --- | --- |
| Free | 0 $ | Astreinte et réponse de base | 5 utilisateurs, 1 horaire, 1 politique d'escalade, 100 SMS/appels par mois, 3 mois d'historique |
| Professional | 21 $/utilisateur/mois (annuel) ou 25 $ (mensuel) | SSO, chat amélioré, post-incident reviews, rapports, MCP | 2 équipes, 1 Incident Workflow, 5 utilisateurs par niveau d'escalade, 1 an d'historique, pas d'ITSM avancé, AIOps et Advance en add-on |

### PD Reliability Platform (par plateforme)

| Forfait | Frais annuels (USD) | Sièges | Événements Signal Intelligence | Lignes Live Call Routing |
| --- | --- | --- | --- | --- |
| Starter | 2 800 $ | 10 | 5 000 | 1 |
| Essential | 12 000 $ | 20 | 40 000 | 2 |
| Plus | 48 000 $ | 50 | 150 000 | 3 |
| Ultimate | Sur devis | 500 | 2 000 000 | 3 |

Toutes les offres PD Reliability Platform incluent Signal Intelligence (ex-AIOps), les fonctions IA et agents, l'ITSM avancé, les champs personnalisés, la priorité, les pages de statut et l'historique complet. **Plus et Ultimate** ajoutent ce qu'un centre d'opérations d'envergure exige : workflows avec conditions, boucles et délais, tâches d'incident, 100 types d'incidents, 10 rôles personnalisés, ServiceNow bidirectionnel avec champs personnalisés, Intune.

### Règles de consommation à comprendre

- **Siège** — inclut les licences de parties prenantes; calculé sur le **pic d'usage du mois précédent**.
- **Événement** — tout objet accepté par l'Events API (y compris via intégrations et courriel); les événements rejetés ou limités ne comptent pas.
- **AI Actions** — consommation des fonctions IA; unité exacte à faire préciser.
- **Crédits** — génériques (sièges, événements ou AI Actions), payés d'avance, **expirent annuellement**, non réaffectables une fois consommés.
- Les quantités incluses dans un forfait ne s'échangent pas entre capacités.
- Paiement exigé d'avance.

### Add-ons et services

| Élément | Disponibilité |
| --- | --- |
| AIOps | Add-on Professional; SKU maintenu à la vente et au renouvellement |
| PagerDuty Advance | Add-on Professional; SKU maintenu |
| Runbook Automation | Add-on dans toutes les offres |
| Status Pages | Add-on Professional (250 abonnés externes inclus) |
| Live Call Routing | Add-on Professional |
| Gold Services (soutien premium) | Add-on |
| Expert Services | **Obligatoire** avec Essential, Plus et Ultimate |
| Formations en direct et certifications | Add-on; PagerDuty University gratuit |

Des sources tierces citent des prix d'entrée d'environ 699 $/mois pour AIOps et 415 $/mois pour Advance (annuel), non confirmés sur la page officielle : à faire coter.

### Pistes de négociation

1. Comparer le coût total sur 3 ans : Professional + add-ons contre Plus/Ultimate + crédits.
2. Mesurer vos volumes réels : nombre d'intervenants, de parties prenantes, d'événements par mois (pic et moyenne).
3. Plafonner la hausse au renouvellement (ex. ≤ 5 %/an) et fixer le prix unitaire des crédits.
4. Négocier le report ou la non-expiration des crédits non utilisés.
5. Faire inclure Expert Services dans le prix plutôt qu'en sus.
6. Prévoir un prix pour une éventuelle région canadienne ou une clause de sortie si la résidence des données devient obligatoire.

## Module 13 — Grille d'exigences et questions au fournisseur

La grille ci-dessous sert de base au devis technique : chaque exigence est vérifiable en démonstration ou en preuve de concept. « O » = obligatoire (éliminatoire), « S » = souhaitable (pondérée).

### Grille d'exigences

| No | Domaine | Exigence | Type | Vérification |
| --- | --- | --- | --- | --- |
| E-01 | Résidence des données | Indiquer la région d'hébergement, la liste des données traitées hors région et les sous-traitants ultérieurs par pays | O | Documentation + EFVP |
| E-02 | Résidence des données | Engagement et échéancier pour une région canadienne, ou mesures compensatoires | S | Lettre d'engagement |
| E-03 | Sécurité | Rapport SOC 2 Type II de moins de 12 mois et certificat ISO 27001 | O | Documents |
| E-04 | Identité | SSO SAML avec Microsoft Entra ID et provisionnement SCIM | O | Démonstration |
| E-05 | Identité | Permissions par équipe, équipes et services privés | O | Démonstration |
| E-06 | Audit | Journal d'audit des configurations conservé au moins 12 mois, exportable | O | Démonstration |
| E-07 | Disponibilité | SLA de disponibilité ≥ 99,9 % avec crédits de service | O | Contrat |
| E-08 | Astreinte | Horaires multicouches, rotations, dérogations, fuseaux horaires, export iCal | O | Démonstration |
| E-09 | Escalade | Au moins 3 niveaux, délais configurables, répétition, round robin | O | Démonstration |
| E-10 | Notification | Push, SMS, appel vocal au Canada; accusé de réception par appel | O | Preuve de concept |
| E-11 | Langue | Interface, application mobile et notifications en français | O | Démonstration |
| E-12 | Ingestion | API d'événements avec déduplication et événements de changement | O | Preuve de concept |
| E-13 | Réduction du bruit | Regroupement intelligent, par contenu et global; suppression; seuils | O | Preuve de concept sur données réelles |
| E-14 | Orchestration | Routage dynamique, extraction de variables, champs personnalisés | O | Preuve de concept |
| E-15 | ITSM | Intégration bidirectionnelle ServiceNow avec mappage de champs personnalisés | O | Preuve de concept |
| E-16 | ITSM | Fenêtres de maintenance liées aux changements ServiceNow | S | Démonstration |
| E-17 | Collaboration | Gestion complète de l'incident depuis Microsoft Teams, parité avec Slack | O | Démonstration |
| E-18 | Incident majeur | Types d'incidents, rôles, tâches et workflows conditionnels | O | Démonstration |
| E-19 | Communication | Licences de parties prenantes, abonnement aux services d'affaires, page de statut interne | O | Démonstration |
| E-20 | Automatisation | Exécution de runbooks par agent sortant, sans port entrant, avec approbation | S | Preuve de concept |
| E-21 | IA | Activation granulaire, non-utilisation des données pour l'entraînement, journalisation des actions d'agents | O | Contrat + démonstration |
| E-22 | Analytique | MTTA, MTTR, interruptions hors heures, API d'export vers Power BI | O | Démonstration |
| E-23 | Rétention | Historique complet des incidents pendant toute la durée du contrat | O | Contrat |
| E-24 | IaC | Fournisseur Terraform couvrant services, horaires, escalades, orchestrations | O | Preuve de concept |
| E-25 | Mobile | Application iOS/Android, NIP, compatibilité Intune | S | Démonstration |
| E-26 | Sortie | Export complet des données et de la configuration en fin de contrat | O | Contrat |
| E-27 | Soutien | Soutien en français, heures de couverture, délais de réponse par sévérité | O | Contrat |

### Questions à poser au fournisseur

**Offre et prix**

1. Quelle est la trajectoire de migration de notre forfait actuel vers PD Reliability Platform, et à quel coût?
2. Comment se calcule un siège : intervenant, partie prenante, compte de service?
3. Quelle est l'unité de mesure des AI Actions et le coût unitaire des crédits?
4. Que se passe-t-il en cas de dépassement des événements inclus : blocage, facturation, alerte?
5. Quel est le contenu d'Expert Services, obligatoire avec Essential, Plus et Ultimate?

**Données et conformité**

6. Quelles données précises quittent la région choisie, vers quels pays?
7. Les fournisseurs de SMS et téléphonie pour des numéros canadiens sont-ils situés au Canada?
8. Pouvez-vous signer une entente de traitement des données conforme à la Loi 25?
9. Une région canadienne est-elle sur la feuille de route, avec quelle date?

**Technique**

10. Quelles fonctions sont en accès anticipé (Early Access) plutôt qu'en disponibilité générale?
11. Quel est le plan de fin de vie des horaires « legacy » et des Rulesets?
12. Quelles limites de débit s'appliquent à l'Events API et à la REST API?
13. Quels connecteurs l'agent SRE prend-il en charge (Azure, Dynatrace, Splunk, etc.)?

## Module 14 — Implantation, gouvernance et validation

Une implantation réussie se joue sur le modèle de services et la discipline de configuration, pas sur le nombre de fonctionnalités activées.

### Bonnes pratiques d'architecture

1. **Service ownership** — chaque service a une équipe responsable et une politique d'escalade d'au moins 2 niveaux.
2. **Une intégration par source et par service**, ou une clé globale avec orchestration de routage.
3. **Haute urgence = action humaine immédiate requise.** Tout le reste en basse urgence ou supprimé.
4. **Configuration en Terraform** dans Git; revue par les pairs avant application.
5. **Nomenclature commune** avec la CMDB (services, équipes, CI).
6. **Zéro donnée personnelle** dans les charges utiles d'événements.
7. **Service Standards** activés et suivis par équipe.

### Gouvernance

| Rôle | Responsabilités |
| --- | --- |
| Propriétaire de la plateforme | Licences, SSO/SCIM, normes, feuille de route |
| Administrateurs | Orchestration globale, intégrations, gabarits |
| Responsables d'équipe | Horaires, escalades, règles de service de leur équipe |
| Gestionnaire des incidents majeurs | Types, rôles, workflows, communications |
| Sécurité / conformité | Revue des accès, journaux d'audit, EFVP |

### Feuille de route type

1. **Fondations (mois 1–2)** — SSO, SCIM, équipes, services pilotes, Terraform.
2. **Astreinte (mois 2–3)** — horaires, escalades, règles de notification, application mobile.
3. **Intégrations (mois 3–4)** — supervision, ServiceNow, Teams.
4. **Réduction du bruit (mois 4–6)** — orchestration, regroupement, mesure de la compression.
5. **Incident majeur (mois 5–6)** — types, rôles, workflows, parties prenantes, page de statut.
6. **Automatisation et IA (mois 6+)** — runbooks, agents, après accord de gouvernance.

### Quiz de validation

1. Quelle différence entre sévérité, urgence et priorité?
2. À quoi sert la `dedup_key`?
3. Quelle couche d'un horaire l'emporte en cas de chevauchement?
4. Que se passe-t-il quand un intervenant accuse réception?
5. Citez trois méthodes de regroupement d'alertes et l'offre qui les inclut.
6. Pourquoi les runners de Runbook Automation simplifient-ils la sécurité réseau?
7. Quelle action de Paige exige une approbation humaine?
8. Quelles régions d'hébergement existent, et lesquelles des données sortent de la région?
9. Comment se calcule un siège dans PD Reliability Platform?
10. Quel forfait minimal permet la synchronisation bidirectionnelle des champs personnalisés ServiceNow?

**Corrigé :** 1) sévérité = alerte, urgence = mode de notification, priorité = classement d'affaires · 2) regrouper les déclenchements répétés dans une même alerte · 3) la couche la plus haute · 4) l'escalade s'arrête · 5) temps, contenu, intelligent (ML), unifié, global — AIOps ou PD Reliability Platform · 6) connexion sortante, aucun port entrant · 7) l'exécution d'une remédiation · 8) US et EU; journaux, analytique agrégée, soutien, facturation, profils globaux · 9) pic d'usage du mois précédent, parties prenantes comprises · 10) Plus.

## Sources

Prix et fonctionnalités relevés le 8 octobre 2026; à reconfirmer auprès du fournisseur avant publication de l'appel d'offres.

- [PagerDuty — Plans & Pricing (grille complète des fonctionnalités)](https://www.pagerduty.com/pricing/)
- [PagerDuty Knowledge Base — Service Regions](https://support.pagerduty.com/main/docs/service-regions)
- [PagerDuty — What's New (agents IA, MCP)](https://www.pagerduty.com/whats-new/)
- [PagerDuty Blog — SRE Agent, virtual responder (mars 2026)](https://www.pagerduty.com/fr/blog/ai/meet-your-virtual-responder-pagerdutys-sre-agent-for-ai-driven-reliability/)
- [ADMIN Magazine — suite d'agents IA, version Fall '25](https://www.admin-magazine.com/index.php/News/PagerDuty-Launches-End-to-End-AI-Agent-Suite)
- [FrontDesk Review — prix et certifications (source tierce)](https://frontdeskreview.com/software/incident-management/pagerduty/)
- [DevTune — prix des add-ons (source tierce)](https://devtune.ai/verticals/incident-management-and-on-call/pagerduty/pricing)
- [PagerDuty University](https://university.pagerduty.com) · [Incident Response Docs](https://response.pagerduty.com)
