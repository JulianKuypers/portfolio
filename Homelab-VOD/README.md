# 🎓 Architecture Réseau Campus Sécurisée (Cisco)

**Objectif du projet :** Concevoir et simuler une infrastructure réseau campus évolutive, sécurisée et redondante visant à unifier les communications de multiples départements (Health, Business, Engineering, etc.) tout en centralisant la gestion du réseau sans-fil.

```mermaid
graph TD
    %% Définition des acteurs externes
    subgraph WAN [Zone WAN / Internet]
        ISP((Réseau Public<br/>ISP))
    end

    %% Périmètre Réseau HQ
    subgraph HQ [Main Campus - HeadQuarter]
        ASA_HQ[Pare-feu Périmétrique<br/>Cisco ASA]
        
        subgraph DMZ [Server Farm]
            Servers[(DHCP, DNS, WLC)]
        end
        
        Core[Couche Core<br/>Switchs L3 + EtherChannel]
        Dist[Couche Distribution<br/>Routage Inter-VLAN + HSRP]
        Acc[Couche Accès<br/>Segmentation VLANs 10-50]
        
        ASA_HQ --- Core
        Core --- Dist
        Dist --- Acc
        Core --- Servers
    end

    %% Périmètre Réseau Branch
    subgraph Branch [Branch Campus - Filiale]
        ASA_BR[Pare-feu Périmétrique<br/>Cisco ASA]
        Dist_BR[Couche Distribution / Routage]
        Acc_BR[Couche Accès<br/>Segmentation VLANs 60-90]

        ASA_BR --- Dist_BR
        Dist_BR --- Acc_BR
    end

    %% Tunnels et Connexions
    ASA_HQ <-->|Tunnel VPN IPsec chiffré| ISP
    ASA_BR <-->|Tunnel VPN IPsec chiffré| ISP


## 🏗️ 1. Infrastructure et Virtualisation
La base de l'infrastructure repose sur une séparation stricte des services pour garantir stabilité et évolutivité.

* **Hyperviseur (Proxmox VE) :** Hébergement et gestion centralisée des ressources matérielles.
* **Conteneurisation (LXC / Docker) :** Chaque micro-service est isolé dans son propre conteneur (LXC pour les services lourds, Docker pour les applications spécifiques). Cela facilite les mises à jour et les redémarrages indépendants.

## 🌐 2. Réseau, Sécurité et Accès Externe
L'infrastructure est accessible depuis l'extérieur de manière sécurisée, sans exposer directement le réseau local.

* **Nom de domaine personnalisé :** Point d'entrée professionnel et propre.
* **Cloudflare (Proxy & Tunnels) :** Gestion de la zone DNS et mise en place d'un tunnel Cloudflare. 
  * *Avantages :* Sécurisation des accès, masquage de l'IP publique (sécurité anti-DDoS) et gestion automatique des certificats SSL (HTTPS) pour un chiffrement de bout en bout.

## 💻 3. Frontend (Interfaces Utilisateurs)
L'expérience utilisateur a été pensée pour être aussi intuitive qu'une plateforme commerciale de VOD.

* **Serveur de diffusion central (Jellyfin) :** Gère l'affichage du catalogue, le transcodage vidéo en temps réel selon la bande passante du client, et intègre l'ajout dynamique de métadonnées (sous-titres, synopsis).
* **Portail web de requêtes (Jellyseerr) :** Connecté au moteur métier, il permet aux utilisateurs d'effectuer des recherches et de demander des médias via une interface web fluide, sans nécessiter d'accès à la configuration technique.

## ⚙️ 4. Backend (Pipeline d'Automatisation CI/CD)
C'est le cœur du système. Aucune intervention humaine n'est requise après une requête utilisateur.

* **Orchestrateur (Radarr) :** Connecté au portail de requêtes, il interroge les sources de données, sélectionne le meilleur paquet selon des critères stricts (voir *Logique Métier*), gère les limites d'ingestion de données (GiB/heure), puis standardise (renommage) et déplace les fichiers vers le serveur de diffusion.
* **Gestionnaire d'indexeurs (Prowlarr) :** Maintient à jour et synchronise de multiples sources de données distantes directement avec l'orchestrateur.
* **Agent de synchronisation (qBittorrent) :** Service de traitement réseau en arrière-plan. Reçoit les ordres de l'orchestrateur, gère la file d'attente, télécharge les paquets de données et signale la fin de l'acquisition pour déclencher le post-traitement.

## 🧠 5. Logique Métier (Filtrage Intelligent et Localisation)
Pour garantir une qualité optimale, des règles de priorisation strictes (*Custom Formats*) ont été configurées dans l'orchestrateur :

1. **Priorité Absolue (+100 points) :** Fichiers contenant des flux audio localisés (Garantit l'audio francophone).
2. **Plan B (+50 points) :** Fichiers taggés avec des métadonnées de sous-titrage natif si la version localisée n'est pas disponible.
3. **Par Défaut (0 point) :** Version originale acceptée temporairement. 
   * *Feature :* Une boucle de vérification (*Upgrade*) tourne en arrière-plan pour remplacer automatiquement ce fichier dès qu'une version répondant aux critères de priorité supérieure est publiée.
