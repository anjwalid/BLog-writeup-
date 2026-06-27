---
title: Pourquoi STRIDE seul ne suffit pas pour sécuriser les systèmes d’IA
date: 2026-06-27 00:05:00 +0000
author: walid
categories:
  - AISEC
tags:
  - stride
  - threat-modeling
  - ai-security
  - mitre-atlas
  - owasp-llm
  - llm-security
  - data-poisoning
  - prompt-injection
image:
  path: https://bishopfox.com/blog/taking-maestro-in-stride-ai-threat-modeling-frameworks
pin: true
toc: true
---


## title: "Pourquoi STRIDE seul ne suffit pas pour sécuriser les systèmes d’IA"
date: 2026-06-27 10:00:00 +0000
categories: \[Cybersecurity, AI Security]
tags: \[stride, ai-security, threat-modeling, mitre-atlas, owasp-llm, llm-security, data-poisoning, prompt-injection]
description: "Comprendre pourquoi STRIDE doit être adapté aux systèmes d’IA et comment l’enrichir avec MITRE ATLAS et OWASP LLM Top 10."
toc: true
mermaid: true

## Introduction

L’intelligence artificielle transforme la manière dont les organisations conçoivent leurs applications, automatisent leurs processus et prennent des décisions. Aujourd’hui, les systèmes d’IA ne se limitent plus à produire de simples prédictions. Ils peuvent analyser des documents, répondre à des utilisateurs, interroger des bases de données, utiliser des outils, exécuter du code ou encore générer des rapports.

Cette évolution apporte de nouvelles opportunités, mais elle introduit aussi de nouveaux risques de sécurité.

Dans une application traditionnelle, les efforts de sécurité se concentrent généralement sur le code, les serveurs, les API, les bases de données, les comptes utilisateurs et les dépendances logicielles. Avec l’IA, le périmètre à protéger devient plus large. Il faut aussi prendre en compte les données d’entraînement, les modèles, les prompts, les embeddings, les bases vectorielles, les pipelines RAG et les outils connectés au modèle.

> **Idée clé :** STRIDE reste une base solide pour la modélisation des menaces, mais il doit être adapté au contexte de l’IA et complété avec des frameworks spécialisés comme [MITRE ATLAS](https://atlas.mitre.org/) et [OWASP LLM Top 10](https://genai.owasp.org/llm-top-10/).
> {: .prompt-info }

Dans cet article, nous allons voir pourquoi STRIDE seul ne suffit pas pour sécuriser les systèmes d’IA, comment l’adapter au contexte des modèles IA, et comment l’enrichir avec [MITRE ATLAS](https://atlas.mitre.org/) et [OWASP LLM Top 10](https://genai.owasp.org/llm-top-10/) pour obtenir une analyse de risques plus complète.

## 1. Avant STRIDE : comprendre ce qui change avec l’IA

Avant de parler de STRIDE, il faut comprendre pourquoi les systèmes d’IA changent la manière de penser la sécurité.

Dans une architecture traditionnelle, les composants critiques sont souvent bien identifiés :

* frontend ;
* backend ;
* API ;
* base de données ;
* système d’authentification ;
* infrastructure cloud ;
* dépendances logicielles.

Avec l’IA, l’attaque peut se produire à plusieurs niveaux supplémentaires. Un attaquant peut par exemple :

* empoisonner les données d’entraînement ;
* injecter de fausses informations dans une base documentaire utilisée par un système RAG ;
* manipuler un prompt pour contourner les règles du modèle ;
* extraire des informations sensibles à partir des réponses du modèle ;
* exploiter les outils connectés au LLM ;
* faire exploser les coûts d’inférence avec des requêtes très coûteuses.

Le danger est que certaines attaques IA ne produisent pas d’erreur immédiate. Une donnée empoisonnée peut rester invisible pendant plusieurs semaines, jusqu’à ce que le modèle soit entraîné, validé puis déployé en production.

> Contrairement à une dépendance logicielle compromise que l’on peut corriger rapidement, un modèle empoisonné peut nécessiter un réentraînement complet.
> {: .prompt-warning }

## 2. La chaîne d’approvisionnement des données en IA

Les applications classiques ont une chaîne d’approvisionnement logicielle : dépendances, bibliothèques, packages, images de conteneurs, plugins, etc.

Les systèmes d’IA héritent de ces risques, mais ajoutent une autre chaîne d’approvisionnement : **la chaîne d’approvisionnement des données**.

Chaque étape devient un point potentiel de compromission.

### 2.1 Collecte des données

Les données d’entraînement peuvent venir de plusieurs sources :

* web scraping ;
* bases de données internes ;
* fournisseurs tiers ;
* contenus générés par les utilisateurs ;
* jeux de données achetés.

Si un attaquant peut influencer l’une de ces sources, il peut introduire des données malveillantes dès le début du processus.

### 2.2 Nettoyage et labellisation

Les données sont ensuite nettoyées, filtrées et parfois labellisées. Cette étape peut être réalisée par des équipes internes, des prestataires externes ou des outils automatiques.

Si les labels sont compromis, le modèle peut apprendre de mauvaises associations. Le jeu de données ne paraît pas forcément corrompu, mais il enseigne au modèle un mauvais comportement.

### 2.3 Entraînement du modèle

Pendant l’entraînement, le modèle apprend à partir des données préparées. Si des données empoisonnées ont survécu aux étapes précédentes, elles peuvent être intégrées dans les poids du modèle.

Contrairement à une bibliothèque logicielle compromise que l’on peut remplacer rapidement, un modèle empoisonné peut nécessiter un réentraînement complet.

### 2.4 Validation et packaging

Une fois entraîné, le modèle est évalué, versionné et stocké dans un registre de modèles. Si ce registre est compromis, un attaquant peut remplacer un modèle légitime par un modèle backdooré.

Le modèle peut réussir les tests classiques si les déclencheurs malveillants ne sont pas présents dans le jeu de validation.

### 2.5 Inférence

Enfin, le modèle est utilisé en production. Pour les systèmes basés sur des LLM, cette étape peut inclure :

* un endpoint d’inférence ;
* un pipeline RAG ;
* une base vectorielle ;
* des documents externes ;
* des outils connectés ;
* des API internes.

Cela crée de nouveaux points d’injection qui n’existent pas toujours dans les applications classiques.

## 3. C’est quoi STRIDE ?

[STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats) est un framework de modélisation des menaces largement utilisé en cybersécurité. Il permet de classer les menaces selon six catégories principales.

Dans une application classique, l’approche consiste à prendre chaque composant du système, puis à se poser des questions comme :

* un utilisateur peut-il usurper l’identité d’un autre utilisateur ?
* une donnée peut-elle être modifiée sans autorisation ?
* une action critique est-elle bien journalisée ?
* une information sensible peut-elle être exposée ?
* le service peut-il être rendu indisponible ?
* un utilisateur peut-il obtenir plus de privilèges que prévu ?

Cette méthode reste très utile. Le problème est que, dans les systèmes d’IA, les réponses deviennent beaucoup plus complexes.

## 4. Pourquoi STRIDE seul ne suffit pas pour l’IA

STRIDE reste une base solide, mais il ne couvre pas toujours correctement les menaces propres à l’IA.

### 4.1 L’intégrité des données devient plus difficile à protéger

Dans STRIDE, la modification non autorisée des données correspond au **Tampering**. Cela fonctionne bien pour une base de données ou un fichier de configuration.

Mais dans l’IA, modifier des données d’entraînement est différent. Les effets sont souvent :

* retardés ;
* diffus ;
* difficiles à observer ;
* intégrés dans le comportement du modèle.

Un jeu de données empoisonné ne génère pas nécessairement une erreur. Il produit simplement un modèle qui se comporte mal dans certains cas.

C’est le principe du **data poisoning**.

### 4.2 Les attaques peuvent viser le comportement du modèle

Un attaquant peut créer des entrées spécialement conçues pour faire échouer le modèle. Cela peut provoquer :

* une mauvaise classification ;
* une hallucination ;
* un contournement de filtre ;
* une réponse non autorisée ;
* une action dangereuse via un outil connecté.

Ce type d’attaque peut relever à la fois du **Tampering**, du **Spoofing** ou de l’**Elevation of Privilege**, selon le contexte.

STRIDE n’a pas été conçu initialement pour des menaces aussi hybrides.

### 4.3 La notion de privilège change avec les LLM

Dans une application classique, une élévation de privilèges signifie souvent qu’un utilisateur obtient des droits administrateur.

Dans un système LLM, le problème peut être différent. Si un chatbot a accès à une base de données, une messagerie, un outil de génération de fichiers ou un moteur d’exécution de code, alors un jailbreak peut transformer ce chatbot en interface d’attaque.

L’attaquant n’obtient pas forcément un accès root sur un serveur. Mais il obtient indirectement les capacités du modèle et de ses outils.

### 4.4 La fuite d’informations peut concerner le modèle lui-même

Dans une application classique, l’**Information Disclosure** concerne souvent une fuite de données sensibles.

Dans l’IA, l’actif volé peut être le modèle lui-même. Par exemple, un attaquant peut interroger une API de manière répétée pour reconstruire une copie fonctionnelle du modèle.

C’est ce qu’on appelle le **model extraction** ou **model stealing**.

## 5. Adaptation de STRIDE aux systèmes d’IA

Pour utiliser STRIDE dans un contexte IA, il faut réinterpréter chaque catégorie.

> STRIDE n’est donc pas à abandonner. Il faut plutôt le **réadapter** au contexte de l’IA.
> {: .prompt-tip }

## 6. Exemple : MegaCorp et son système IA

Prenons l’exemple d’une entreprise fictive appelée **MegaCorp**.

MegaCorp utilise plusieurs systèmes d’IA :

* un chatbot client basé sur un LLM ;
* un pipeline RAG connecté à une base documentaire interne ;
* un système de détection de fraude réentraîné chaque mois ;
* un moteur de recommandation accessible via API ;
* des outils connectés au chatbot pour consulter certaines données clients.

Avec STRIDE-AI, on peut identifier plusieurs risques.

### Spoofing

Si un attaquant injecte de faux documents dans la base de connaissances RAG, le chatbot peut répondre avec des informations fausses mais crédibles.

### Tampering

Si un attaquant injecte des transactions malveillantes dans les données utilisées pour réentraîner le modèle de fraude, le modèle peut progressivement apprendre à ne plus détecter certains schémas frauduleux.

### Repudiation

Si MegaCorp ne journalise pas les versions de modèles, les entrées, les sorties et le contexte RAG utilisé, il devient difficile d’expliquer une décision du modèle après coup.

### Information Disclosure

Si un concurrent interroge massivement l’API du moteur de recommandation, il peut reconstruire un modèle similaire sans accéder directement aux poids du modèle.

### Denial of Service

Si un attaquant envoie des milliers de prompts très longs au chatbot, le service peut rester disponible, mais les coûts d’inférence peuvent exploser.

### Elevation of Privilege

Si le chatbot est jailbreaké et dispose d’outils mal limités, l’attaquant peut l’utiliser pour interroger des données sensibles.

## 7. MITRE ATLAS : enrichir STRIDE avec des techniques concrètes

STRIDE permet de répondre à la question :

> **Quel type de menace peut exister ?**

[MITRE ATLAS](https://atlas.mitre.org/) permet de répondre à une autre question :

> **Comment l’attaquant peut-il concrètement réaliser cette attaque ?**

MITRE ATLAS, pour **Adversarial Threat Landscape for Artificial-Intelligence Systems**, est une base de connaissances dédiée aux tactiques et techniques adverses visant les systèmes d’IA et de machine learning.

On peut le voir comme l’équivalent de [MITRE ATT&CK](https://attack.mitre.org/), mais pour l’intelligence artificielle.

Quelques techniques importantes :

L’intérêt d’ATLAS est de transformer une menace générale en scénario concret.

Au lieu de dire simplement :

> Il existe un risque de Tampering.

On peut dire :

> Le système est exposé à du **Data Poisoning — AML.T0020**, avec un risque d’injection de données malveillantes dans le pipeline d’entraînement.

Cela permet d’obtenir une analyse plus précise, avec un identifiant technique, des scénarios d’attaque et des pistes de mitigation.

## 8. OWASP LLM Top 10 : relier les risques aux composants

STRIDE classe les menaces.
MITRE ATLAS décrit les techniques d’attaque.
[OWASP LLM Top 10](https://genai.owasp.org/llm-top-10/) permet de relier les risques aux composants d’une architecture LLM.

L’OWASP Top 10 for LLM Applications identifie les risques majeurs pour les applications basées sur des grands modèles de langage.

Cette approche est très utile pour analyser une architecture.

Par exemple, si une organisation utilise une base vectorielle pour son système RAG, elle doit immédiatement penser à :

* **LLM01** : prompt injection indirect ;
* **LLM08** : faiblesses liées aux embeddings ;
* **LLM09** : désinformation provenant de documents obsolètes ou incorrects.

## 9. Comment combiner STRIDE, MITRE ATLAS et OWASP

Ces trois frameworks ne sont pas concurrents. Ils sont complémentaires.

On peut les voir comme trois niveaux d’analyse :

STRIDE donne une vue générale des types de menaces.
MITRE ATLAS donne le détail technique des attaques.
OWASP LLM Top 10 indique les composants les plus exposés et aide à prioriser les actions de sécurité.

## 10. Mesures de sécurité recommandées

Pour sécuriser un système d’IA, il ne suffit pas de protéger uniquement l’API ou le serveur. Il faut sécuriser toute la chaîne.

### 10.1 Sécuriser les données

Il faut contrôler la provenance des données, vérifier leur qualité, détecter les anomalies et surveiller les modifications inhabituelles.

Mesures recommandées :

* validation des sources de données ;
* suivi de la provenance des données ;
* détection d’anomalies dans les jeux de données ;
* séparation entre données fiables et données non vérifiées ;
* contrôle des labels utilisés pour l’entraînement.

### 10.2 Sécuriser le modèle

Le modèle doit être traité comme un actif critique.

Mesures recommandées :

* versionner les modèles ;
* contrôler leur intégrité ;
* protéger le registre de modèles ;
* éviter les modèles non vérifiés provenant de sources inconnues ;
* surveiller les changements de performance après déploiement.

### 10.3 Sécuriser les prompts

Les prompts système ne doivent pas contenir de secrets, de mots de passe ou de clés API.

Mesures recommandées :

* ne jamais mettre de secrets dans le system prompt ;
* séparer clairement les instructions système du contenu utilisateur ;
* détecter les tentatives de prompt injection ;
* journaliser les entrées et sorties importantes ;
* limiter les informations sensibles exposées au modèle.

### 10.4 Sécuriser le RAG

Les systèmes RAG introduisent des risques spécifiques, car le modèle utilise des documents externes comme contexte.

Mesures recommandées :

* valider les documents avant indexation ;
* contrôler les accès à la base vectorielle ;
* surveiller la fraîcheur des documents ;
* supprimer les documents obsolètes ;
* empêcher l’indexation de contenu non fiable.

### 10.5 Sécuriser les outils connectés

Les outils accessibles au LLM doivent suivre le principe du moindre privilège.

Mesures recommandées :

* limiter les permissions des outils ;
* valider toutes les actions avant exécution ;
* imposer une confirmation humaine pour les actions sensibles ;
* journaliser les appels d’outils ;
* séparer les outils critiques des outils simples.

### 10.6 Surveiller l’inférence

Les endpoints LLM peuvent être exposés à des attaques de coût, comme le **denial of wallet**.

Mesures recommandées :

* rate limiting ;
* limites de tokens ;
* quotas par utilisateur ;
* surveillance des coûts ;
* détection des prompts anormalement longs ;
* blocage des usages abusifs.

## Conclusion

STRIDE reste une base solide pour la modélisation des menaces. Cependant, les systèmes d’IA introduisent de nouveaux actifs, de nouvelles chaînes d’approvisionnement et de nouvelles formes d’attaques.

Utilisé seul, STRIDE ne permet pas toujours de capturer toute la complexité des risques liés à l’IA. C’est pourquoi il doit être adapté et enrichi.

Une approche efficace consiste à combiner :

* **STRIDE-AI** pour identifier les catégories de menaces ;
* **MITRE ATLAS** pour comprendre les techniques d’attaque spécifiques à l’IA ;
* **OWASP LLM Top 10** pour relier les risques aux composants d’une architecture LLM.

En combinant ces trois approches, on obtient une méthode plus complète, plus concrète et plus adaptée à la sécurité des systèmes d’intelligence artificielle modernes.

> **Résumé final :** STRIDE donne la structure, MITRE ATLAS donne le détail technique, et OWASP LLM Top 10 indique où se trouvent les risques dans l’architecture.
> {: .prompt-tip }

## Références

* [Microsoft — Threat Modeling Tool threats / STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats)
* [Microsoft — Threat Modeling Tool overview](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool)
* [MITRE ATLAS — Adversarial Threat Landscape for Artificial-Intelligence Systems](https://atlas.mitre.org/)
* [MITRE ATT&CK — Knowledge base of adversary tactics and techniques](https://attack.mitre.org/)
* [OWASP GenAI Security Project — LLM Top 10](https://genai.owasp.org/llm-top-10/)
* [OWASP Project Page — Top 10 for Large Language Model Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

CatégorieSignificationPropriété de sécurité concernée**S — Spoofing**Usurpation d’identitéAuthenticité**T — Tampering**Modification non autoriséeIntégrité**R — Repudiation**Déni d’une action effectuéeNon-répudiation**I — Information Disclosure**Divulgation d’informationsConfidentialité**D — Denial of Service**Indisponibilité du serviceDisponibilité**E — Elevation of Privilege**Obtention de droits non autorisésAutorisationSTRIDEManifestation principale en IAExemple**Spoofing**Usurpation de source de donnéesInjection de faux documents dans une base RAG**Tampering**Data poisoningDonnées d’entraînement manipulées pour influencer le modèle**Repudiation**Absence de traçabilité des décisionsImpossible d’expliquer pourquoi le modèle a pris une décision**Information Disclosure**Model extractionReconstruction d’un modèle via son API**Denial of Service**Denial of walletRequêtes coûteuses qui font exploser la facture cloud**Elevation of Privilege**Jailbreak / guardrail bypassContournement des règles du LLM pour accéder à des capacités interditesTechnique ATLASIdentifiantDescriptionCorrespondance STRIDE[Data Poisoning](https://atlas.mitre.org/techniques/AML.T0020/)AML.T0020Injection de données malveillantes dans le pipeline d’entraînementTampering[Model Extraction](https://atlas.mitre.org/techniques/AML.T0024/)AML.T0024Reconstruction d’un modèle via son APIInformation Disclosure[Evade ML Model](https://atlas.mitre.org/techniques/AML.T0015/)AML.T0015Entrées adversariales pour tromper le modèleTampering / Spoofing / Elevation of Privilege[LLM Prompt Injection](https://atlas.mitre.org/techniques/AML.T0051/)AML.T0051Manipulation du comportement du LLM par injection d’instructionsTampering[Backdoor ML Model](https://atlas.mitre.org/techniques/AML.T0018/)AML.T0018Déclencheur caché intégré dans le modèleTamperingRisque OWASPDescriptionComposants concernés**LLM01 Prompt Injection**Manipulation du comportement du modèleEndpoint LLM, pipeline RAG, base vectorielle**LLM02 Sensitive Information Disclosure**Fuite de données sensiblesModèle, prompt système, données d’entraînement**LLM03 Supply Chain**Dépendances, modèles ou datasets compromisPipeline d’entraînement, registre de modèles, plugins**LLM04 Data and Model Poisoning**Données ou modèle corrompusTraining pipeline, model registry**LLM05 Improper Output Handling**Sorties non validéesFrontend, API gateway, services aval**LLM06 Excessive Agency**Trop de permissions accordées au modèleOutils, agents, API**LLM07 System Prompt Leakage**Fuite du prompt systèmeConfiguration LLM**LLM08 Vector and Embedding Weaknesses**Faiblesses des embeddings et bases vectoriellesVector database, RAG**LLM09 Misinformation**Réponses fausses mais crédiblesLLM, sources RAG**LLM10 Unbounded Consumption**Consommation excessive de ressourcesEndpoint LLM, API gatewayFrameworkRôleQuestion principale**STRIDE-AI**Classer les menacesQu’est-ce qui peut mal tourner ?**MITRE ATLAS**Décrire les techniques d’attaqueComment l’attaquant peut-il procéder ?**OWASP LLM Top 10**Relier les risques aux composantsOù se situe le risque dans l’architecture ?
