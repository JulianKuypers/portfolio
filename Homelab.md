# 🏠 Infrastructure Réseau & Virtualisation (Homelab)

**Objectif du projet :** Concevoir, déployer et administrer une infrastructure réseau domestique de niveau entreprise, axée sur la sécurité périmétrique, la virtualisation des services, l'auto-hébergement et la segmentation avancée du trafic.

```mermaid
graph TD
    %% 1. Zone Serveurs et Services
    subgraph DMZ [Services Auto-hébergés]
        Docker[(Conteneurs LXC/Docker<br/>Infrastructure VOD, Tunnels)]
    end

    %% 2. Cœur de réseau virtualisé
    subgraph Hypervisor [Hyperviseur Bare-Metal - Proxmox VE]
        OPN[Pare-feu Périmétrique L3<br/>OPNsense VM]
    end

    %% 3. Zone WAN / Internet
    subgraph WAN [Zone WAN / Internet]
        ISP((Réseau Public<br/>Modem FAI))
    end

    %% 4. Couche Accès Physique
    subgraph Access [Couche Distribution & Accès - VLANs 1, 20, 30]
        Switch[Switch Administrable<br/>TP-Link]
        AP[Points d'Accès Wi-Fi<br/>Omada]
        CPL[Infrastructure CPL<br/>Devolo vers Terminaux]

        Switch --- AP
        Switch --- CPL
    end

    %% 5. Connexions finales
    OPN --- Docker
    OPN <-->|Passerelle WAN / DHCP| ISP
    OPN <-->|Liaison Trunk 802.1Q| Switch

1. Virtualisation et Serveurs (Proxmox VE & Docker)

L'infrastructure repose sur un hyperviseur bare-metal permettant une gestion granulaire des ressources et une isolation des environnements.

    Hyperviseur Type 1 : Déploiement et administration d'un environnement complet sous Proxmox VE, hébergeant le routeur principal (VM) ainsi que les divers conteneurs (LXC).

    Conteneurisation (Micro-services) : Gestion de conteneurs Docker pour le déploiement de services auto-hébergés, incluant la conception d'un pipeline VOD complet (Jellyfin, Radarr, Prowlarr) optimisant l'allocation des ressources matérielles.

🔒 2. Routage Avancé et Sécurité Périmétrique (OPNsense)

La sécurité périmétrique et le routage de niveau 3 (L3) sont centralisés au sein d'une appliance de sécurité virtualisée.

    Pare-feu Virtualisé : Configuration d'OPNsense en tant que passerelle principale pour filtrer le trafic entrant et sortant, gérer les traductions d'adresses (NAT) et assurer la distribution DHCP.

    Contrôle d'Accès & Tunnels : Élaboration de politiques de pare-feu (Firewall Rules) appliquant le principe du moindre privilège pour bloquer le trafic inter-réseaux non sollicité, et intégration de tunnels sécurisés (Cloudflare) pour masquer l'infrastructure publique.

📡 3. Segmentation Réseau et Topologie (VLANs & Switch)

Le réseau physique est segmenté de manière logique pour isoler les flux de données et confiner les équipements potentiellement vulnérables.

    Isolation Logique (802.1Q) : Création et routage de sous-réseaux étanches (LAN Privé, VLAN IoT 20, VLAN Invités 30) protégeant les terminaux principaux des appareils connectés moins sécurisés.

    Distribution Physique : Interconnexion de l'architecture via un switch administrable TP-Link, distribuant les réseaux virtuels de la machine Proxmox vers l'infrastructure CPL (Devolo) et les terminaux finaux.

⚙️ 4. Gestion Centralisée du Sans-Fil et Maintenance (Omada)

Le laboratoire est pensé pour être évolutif et maintenu avec des standards professionnels de documentation et de gestion.

    Écosystème Wi-Fi Professionnel : Déploiement et intégration de points d'accès TP-Link Omada pour une couverture sans-fil optimale, cartographiant directement les SSID virtuels sur leurs VLANs respectifs.

    Administration Continue : Modélisation et documentation exhaustive de l'architecture physique et logique via Draw.io, facilitant grandement les opérations de maintenance et le dépannage (troubleshooting) du réseau.
