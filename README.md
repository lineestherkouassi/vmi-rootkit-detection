# vmi-rootkit-detection
AI-powered Virtual Machine Introspection framework for malware and rootkit detection in OpenStack cloud environments./Framework d'introspection de machines virtuelles basé sur l'intelligence artificielle pour la détection des malwares et des rootkits dans les environnements cloud OpenStack.

## Présentation

Ce projet présente un framework conçu pour détecter les malwares et rootkits dans des environnements cloud virtualisés sans installer d'agent au sein des machines virtuelles.

Le projet combine la **Virtual Machine Introspection (VMI)**, l'analyse mémoire avec **Volatility 3** et un modèle de **Machine Learning non supervisé** afin d'identifier les comportements anormaux à partir d'artefacts mémoire.

L'approche proposée repose sur une surveillance **sans agent (agentless)**, permettant d'observer le comportement des machines virtuelles depuis l'hyperviseur, sans installer de logiciel de sécurité dans le système invité.

Le framework combine plusieurs modules :

- Acquisition de la mémoire via la Virtual Machine Introspection (VMI)
- Analyse des images mémoire avec Volatility 3
- Extraction automatique de caractéristiques comportementales
- Détection d'anomalies à l'aide d'algorithmes d'apprentissage automatique

---

# Objectifs

- Développer une solution de détection de malwares sans agent pour les environnements cloud virtualisés.
- Collecter les artefacts mémoire des machines virtuelles grâce à la Virtual Machine Introspection.
- Analyser les images mémoire avec Volatility 3.
- Extraire automatiquement des caractéristiques issues des espaces **Kernel** et **User**.
- Détecter les comportements anormaux associés aux malwares et aux rootkits.
- Améliorer la détection de menaces inconnues grâce à l'apprentissage automatique.

---

# Architecture du framework

```text
Machine virtuelle
        │
        ▼
Virtual Machine Introspection (VMI)
        │
        ▼
Acquisition de la mémoire
        │
        ▼
Analyse mémoire (Volatility 3)
        │
        ▼
Extraction des caractéristiques
        │
        ▼
Module de Machine Learning
(One-Class SVM)
        │
        ▼
Détection des malwares et rootkits
```

---

# Fonctionnalités

- Surveillance sans agent des machines virtuelles
- Acquisition d'images mémoire via la Virtual Machine Introspection
- Analyse forensique des images mémoire avec Volatility 3
- Extraction automatique de caractéristiques comportementales
- Analyse des artefacts des espaces Kernel et User
- Détection des anomalies par apprentissage automatique avec OCSVM
- Architecture modulaire facilitant l'évolution du framework

---

# Technologies utilisées

- Python
- OpenStack
- KVM / QEMU
- Virtual Machine Introspection (VMI)
- Volatility 3
- Scikit-learn
- Pandas
- NumPy
- Linux

---

# Structure du projet

```text
CloudGuard/
│
├── introspection/
├── volatility/
├── feature_extraction/
├── ml_module/
├── dataset/
├── documentation/
├── experiments/
└── README.md
```

---

# Contributions

- Détection de malwares sans agent grâce à la Virtual Machine Introspection.
- Analyse forensique de la mémoire pour les environnements cloud.
- Extraction de caractéristiques comportementales issues des espaces Kernel et User.
- Évaluation comparative de plusieurs algorithmes de détection d'anomalies.
- Détection de comportements malveillants, y compris pour des menaces inconnues.

---

# Résultats

Le framework démontre que l'association de la **Virtual Machine Introspection**, de l'analyse mémoire et de l'apprentissage automatique permet de détecter efficacement des comportements malveillants dans des infrastructures cloud virtualisées.

Les expérimentations mettent en évidence les performances du modèle **One-Class SVM**, qui obtient les meilleurs résultats parmi les algorithmes évalués tout en maintenant un très faible taux de faux positifs.

---

# Contexte académique

Ce projet a été réalisé dans le cadre d'un **Projet de Fin d'Études (PFE)** en cybersécurité. Il porte sur la conception d'un framework intelligent de détection des malwares et des rootkits dans les environnements cloud virtualisés à l'aide de la Virtual Machine Introspection et de l'apprentissage automatique.

---

# Auteur

**Moaye Line Esther Kouassi**

Ingénieure Cybersécurité • Cloud Security 
