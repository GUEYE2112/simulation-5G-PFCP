# SAE Détection d'anomalies PFCP dans un réseau 5G
Simulation d'un réseau 5G et analyse d'une attaque PFCP

> Projet réalisé dans le cadre du BUT 2 Réseaux & Télécommunications - Parcours ROM
> IUT de Villetaneuse — Université Sorbonne Paris Nord  
> Encadré par M. OUAMRI

---

## Membres du projet

- Codou GUEYE
- Mame Bousso GUEYE
- Justine MOLONGO ARMOUDON

---

## Présentation

Ce projet déploie une architecture 5G simplifiée en environnement virtualisé,
et analyse le protocole **PFCP** (Packet Forwarding Control Protocol)
sur l'interface **N4** entre le SMF et l'UPF.

L'architecture repose sur trois machines virtuelles :

| VM | Rôle | Adresse IP |
|----|------|------------|
| VM1 — UERANSIM | Simulation UE + gNB | 192.168.200.130 |
| VM2 — Open5GS CP | Plan de contrôle (AMF, SMF, NRF, AUSF, UDM) | 192.168.200.128 |
| VM3 — Open5GS UPF | Plan utilisateur (UPF) | 192.168.200.129 |

---

## Architecture
UE (UERANSIM)

|   RRC/NAS simulé

▼

gNB (UERANSIM) —N2 (NGAP/SCTP:38412)—► AMF (Open5GS CP)

|                                          |

|  N3 (GTP-U/UDP:2152)               SMF ◄┘

▼                                     | N4 (PFCP/UDP:8805)

UPF (Open5GS UPF) ◄───────────────────┘

|

▼

ogstun (10.45.0.1/16) → Internet

---

## Composants principaux

| Composant | Rôle |
|-----------|------|
| **UE** | Terminal utilisateur simulé (IMSI, clés d'authentification) |
| **gNB** | Station de base 5G — relaie la signalisation vers l'AMF |
| **AMF** | Gestion de la mobilité et authentification |
| **SMF** | Gestion des sessions PDU |
| **UPF** | Transport des données utilisateur |
| **NRF** | Registre des fonctions réseau |
| **MongoDB** | Stockage des abonnés |
---

## Paramètres réseau

| Paramètre | Valeur |
|-----------|--------|
| MCC | 999 |
| MNC | 70 |
| TAC | 1 |
| SST | 1 |
| DNN | internet |
| Réseau UE | 10.45.0.0/16 |
| Passerelle UE | 10.45.0.1 |

---

## Interfaces et protocoles utilisés

| Interface | Communication | Protocole | Rôle |
|-----------|--------------|-----------|------|
| N2 | gNB ↔ AMF | NGAP / SCTP 38412 | Signalisation |
| N3 | gNB ↔ UPF | GTP-U / UDP 2152 | Données utilisateur |
| **N4** | **SMF ↔ UPF** | **PFCP / UDP 8805** | **Contrôle des sessions** |
| N6 | UPF ↔ DN | IP | Accès réseau data |

> **Point clé** : l'interface N4 est la zone critique car elle transporte
> PFCP entre le SMF et l'UPF.

---

## Analyse de l'attaque PFCP

### Attaque observée : Heartbeat Flood

L'attaque consiste à envoyer massivement des **PFCP Heartbeat Request**
depuis une source externe vers l'UPF sur le port UDP 8805.

**Observations Wireshark :**
- Nombreux Heartbeat Request depuis une IP externe
- Destination : UPF port 8805
- Répétition anormale — risque de surcharge de l'interface N4

### Impacts possibles

| Type d'attaque | Impact |
|----------------|--------|
| Heartbeat Flood | Surcharge de traitement du SMF/UPF |
| Session Flood | Création excessive de sessions |
| Session Deletion | Coupure du trafic utilisateur |
| Session Modification | Débit instable / QoS dégradée |
| PFCP malformés | Erreurs dans les journaux |

---

## Recommandations de sécurité

**Mesures réseau :**
- Limiter l'accès à N4 aux IP autorisées uniquement
- Filtrer le port UDP 8805
- Superviser le volume de messages PFCP
- Journaliser les événements SMF/UPF

**Détection et supervision :**
- Détecter les messages anormaux
- Mettre en place du rate limiting
- Surveiller les types de messages

---

## Outils utilisés

- **UERANSIM** — Simulation UE et gNB
- **Open5GS** — Cœur réseau 5G
- **MongoDB / WebUI** — Gestion des abonnés
- **Wireshark** — Analyse du trafic PFCP
- **tcpdump** — Capture réseau

---

## Structure du dépôt
simulation-5G-PFCP/
├── configs/configs/        
├── Codou GUEYE Mame B...  
├── Documentation technique
└── README.md 

## Auteurs du projet

- Codou GUEYE
- Mame Bousso GUEYE
- Justine MOLONGO ARMOUDON
- LE PABIC Ronan
- MOUTIER Amaury
