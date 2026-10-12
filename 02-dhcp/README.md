# TP2 — Automatiser l'adressage IP avec DHCP (Packet Tracer)

[← Lab précédent](../01-reseau-lan-simple) · [Sommaire](../README.md) · [Lab suivant →](../03-acl)

| Fichier | Contenu |
|---|---|
| `tp02.pkt` | Fichier Packet Tracer du TP |
| `configs/` | Configurations des équipements (texte) |
| `images/` | Captures d'écran |

📝 Article LinkedIn : [FR](#) · [EN](#)

**Niveau :** Débutant
**Prérequis :** TP1 (réseau LAN avec 2 sous-réseaux et routage inter-réseaux)
**Objectif :** Remplacer l'adressage IP statique des 4 PC par une attribution automatique via DHCP configuré sur le routeur.
**Durée estimée :** 40–50 min

---

## 1. Rappel de la topologie (identique au TP1)

```
        [PC0]                                   [PC2]
          |                                        |
        [Switch0] ---- [Routeur0] ---- [Switch1]
          |                                        |
        [PC1]                                   [PC3]
```

- Réseau 1 (côté Switch0) : 192.168.1.0/24 — passerelle 192.168.1.1
- Réseau 2 (côté Switch1) : 192.168.2.0/24 — passerelle 192.168.2.1

Si tu repars d'un nouveau fichier Packet Tracer, reconstruis la topologie du TP1.

---

## 2. Pourquoi le DHCP ?

Configurer chaque PC à la main fonctionne pour 4 machines. Ça ne passe pas à l'échelle pour 200 postes, et ça multiplie les erreurs humaines (doublons d'IP, oublis de passerelle...). Le **DHCP (Dynamic Host Configuration Protocol)** centralise cette attribution : le routeur (ou un serveur dédié) distribue automatiquement IP, masque, passerelle et DNS à chaque PC qui en fait la demande.

**Règle simple :** tout ce qui *distribue* ou *route* (routeur, serveurs) garde une IP fixe ; tout ce qui *consomme* (PC, laptops, téléphones) passe en DHCP.

---

## 3. Étape 1 — Adresser les interfaces du routeur (prérequis indispensable)

Un pool DHCP ne sert à rien si le routeur n'a pas lui-même une adresse dans le réseau qu'il doit servir.

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
```

Vérifie avec `show ip interface brief` : Gi0/0 et Gi0/1 doivent afficher leur IP et être en **up / up**.

---

## 4. Étape 2 — Configurer les deux pools DHCP

On crée **deux pools distincts**, un par sous-réseau, car chaque réseau a sa propre passerelle.

```
! --- Pool pour le réseau 192.168.1.0/24 ---
ip dhcp excluded-address 192.168.1.1 192.168.1.9
ip dhcp pool POOL_RESEAU1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
 exit

! --- Pool pour le réseau 192.168.2.0/24 ---
ip dhcp excluded-address 192.168.2.1 192.168.2.9
ip dhcp pool POOL_RESEAU2
 network 192.168.2.0 255.255.255.0
 default-router 192.168.2.1
 dns-server 8.8.8.8
 exit

end
write memory
```

**Pourquoi `ip dhcp excluded-address` ?**
Le pool couvre tout le sous-réseau (de .1 à .254), et le routeur ne sait pas quelles adresses tu utilises déjà en statique. Sans exclusion, il pourrait attribuer à un PC une adresse déjà prise (par exemple la passerelle 192.168.1.1), ce qui crée un conflit d'IP.

On exclut donc :
- l'adresse du routeur (la passerelle) ;
- les équipements à IP fixe (serveurs, imprimantes, switches managés, points d'accès) ;
- éventuellement une plage réservée pour de futurs équipements fixes.

Ici, on exclut .1 à .9 : les PC recevront des adresses à partir de .10.

> **Précision :** Cisco IOS teste souvent une adresse (ping) avant de l'attribuer, et saute celle qui répond. On ne peut pas s'y fier : un équipement qui ne répond pas au ping passe à travers. L'exclusion reste la bonne pratique.
>
> **Sur l'ordre des commandes :** ce qui compte, c'est que l'exclusion soit en place **avant que les PC demandent une adresse**. Un PC qui a déjà reçu une adresse la garde jusqu'à la fin de son bail.

---

## 5. Étape 3 — Configurer les PC

Sur chaque PC (PC0 à PC3) : **Desktop → IP Configuration → sélectionner "DHCP"** (au lieu de "Static"). Ne saisis aucune IP, aucun masque, aucune passerelle.

Packet Tracer envoie alors une requête DHCP ; le PC récupère automatiquement IP, masque, passerelle et DNS.

---

## 6. Vérifications

Sur chaque PC, dans **IP Configuration**, tu dois voir une IP attribuée automatiquement (192.168.1.x ou 192.168.2.x selon le switch).

Sur le routeur :
```
show ip dhcp pool
show ip dhcp binding
```
- `show ip dhcp pool` : liste les pools, le nombre d'adresses louées et exclues.
- `show ip dhcp binding` : liste les baux actifs (IP, adresse MAC du PC, expiration). Tu dois voir 4 lignes.

Test de connectivité :
```
ping <IP de PC2>     (depuis PC0)
```
Doit réussir : le DHCP ne change rien au routage, seulement à la méthode d'attribution.

---

## 7. Le piège classique : « les pools sont bons mais personne n'obtient d'IP »

Si `show ip dhcp binding` est vide et que `show ip dhcp pool` affiche `Leased addresses : 0`, aucune requête n'est arrivée jusqu'au routeur. Vérifie dans l'ordre :

1. **`show ip interface brief`** : les interfaces Gi0/0 et Gi0/1 ont-elles leur IP ? Si elles affichent **"unassigned"**, le routeur n'a aucune adresse dans les réseaux à servir et reste sourd aux demandes DHCP. C'est la cause la plus fréquente quand on repart d'un nouveau fichier.
2. Les interfaces sont-elles en **up / up** ? Sinon : `no shutdown` oublié, ou problème de câble.
3. Les PC sont-ils bien en mode **DHCP** ? Repasse en Static puis de nouveau en DHCP pour relancer la requête.
4. Les liens sont-ils verts ? S'ils sont orange, attends 30 secondes ou utilise "Fast Forward Time".

---

## 8. Ce qu'il faut retenir

- Le DHCP fonctionne par **bail** (lease) : l'IP est prêtée pour une durée donnée.
- Un pool DHCP n'a d'effet que si le routeur a une interface dans le réseau correspondant.
- Les adresses fixes doivent être **exclues** pour éviter les conflits.
- Un routeur Cisco peut gérer plusieurs pools, un par sous-réseau.

---

## 9. Pour aller plus loin (TP3)

Les **listes de contrôle d'accès (ACL)** : restreindre qui peut communiquer avec qui entre les deux réseaux.

---

## Captures d'écran

<!-- Ajoute tes captures dans le dossier images/ puis décommente les lignes ci-dessous :
![Topologie](images/topologie.png)
![Vérification](images/verification.png)
-->
