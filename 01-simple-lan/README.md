# TP1 — Mon premier réseau LAN (Packet Tracer)

[Sommaire](../README.md) · [Lab suivant →](../02-dhcp)

| Fichier | Contenu |
|---|---|
| `tp01.pkt` | Fichier Packet Tracer du TP |
| `configs/` | Configurations des équipements (texte) |
| `images/` | Captures d'écran |

📝 Article LinkedIn : [FR](#) · [EN](#)

**Niveau :** Débutant total
**Objectif :** Construire un petit réseau local avec 2 switches et 1 routeur, adresser les postes en IPv4, et vérifier la connectivité de bout en bout.
**Durée estimée :** 45–60 min

---

## 1. Topologie à construire

```
        [PC0]                                   [PC2]
          |                                        |
        [Switch0] ---- [Routeur0] ---- [Switch1]
          |                                        |
        [PC1]                                   [PC3]
```

Matériel Packet Tracer à glisser sur le plan de travail :
- 1 routeur : **Router 2911** (ou équivalent)
- 2 switches : **Switch 2960**
- 4 PC (génériques)
- Câbles : **Copper Straight-Through** entre PC↔Switch et Switch↔Routeur

> Astuce Packet Tracer : au survol d'un câble mal choisi, une icône orange apparaît. Utilise le mode "Auto" de câblage au début si tu hésites sur le type de câble.

---

## 2. Plan d'adressage IP

On utilise deux sous-réseaux distincts, un par switch, reliés par le routeur (routage inter-réseaux).

| Équipement | Interface | Adresse IP | Masque |
|---|---|---|---|
| Routeur0 | Gi0/0 (côté Switch0) | 192.168.1.1 | 255.255.255.0 |
| Routeur0 | Gi0/1 (côté Switch1) | 192.168.2.1 | 255.255.255.0 |
| PC0 | NIC | 192.168.1.10 | 255.255.255.0 (passerelle 192.168.1.1) |
| PC1 | NIC | 192.168.1.11 | 255.255.255.0 (passerelle 192.168.1.1) |
| PC2 | NIC | 192.168.2.10 | 255.255.255.0 (passerelle 192.168.2.1) |
| PC3 | NIC | 192.168.2.11 | 255.255.255.0 (passerelle 192.168.2.1) |

---

## 3. Configuration des PC

Sur chaque PC : onglet **Desktop → IP Configuration**, saisir IP / masque / passerelle selon le tableau ci-dessus (Static, pas DHCP pour ce TP).

---

## 4. Configuration du routeur (CLI)

Clique sur Routeur0 → onglet **CLI** :

```
enable
configure terminal

interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit

interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
 exit

end
write memory
```

`no shutdown` est l'étape que presque tout débutant oublie — sans elle, l'interface reste administrativement désactivée même si tout le reste est correct.

---

## 5. Vérifications

Sur le routeur :
```
show ip interface brief
```
→ Les deux interfaces doivent apparaître **up / up**.

Depuis PC0 (invite de commandes) :
```
ping 192.168.1.1      → doit réussir (passerelle locale)
ping 192.168.2.1      → doit réussir (routeur, autre interface)
ping 192.168.2.10     → doit réussir (PC2, routage inter-réseaux)
```

Si le dernier ping échoue alors que les deux premiers passent : vérifie que Routeur0 a bien les deux interfaces "up" et que les PC2/PC3 ont la bonne passerelle configurée (erreur la plus fréquente à ce stade).

---

## 6. Ce qu'il faut retenir

- Un switch ne fait que commuter des trames dans un même réseau (même sous-réseau IP)
- Pour qu'un PC du réseau 192.168.1.0/24 parle à un PC du réseau 192.168.2.0/24, il faut un routeur : c'est le **routage inter-réseaux**
- La passerelle par défaut est l'adresse que le PC contacte dès qu'il veut sortir de son propre sous-réseau

---

## 7. Pour aller plus loin (TP2, semaine prochaine)

Ajouter le DHCP sur le routeur pour que PC0–PC3 reçoivent leur IP automatiquement, au lieu de la configuration statique faite ici.

---

## Captures d'écran

<!-- Ajoute tes captures dans le dossier images/ puis décommente les lignes ci-dessous :
![Topologie](images/topologie.png)
![Vérification](images/verification.png)
-->
