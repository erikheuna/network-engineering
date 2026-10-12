# TP3 — Contrôler qui parle à qui avec les ACL (Packet Tracer)

[← Lab précédent](../02-dhcp) · [Sommaire](../README.md)

| Fichier | Contenu |
|---|---|
| `tp03.pkt` | Fichier Packet Tracer du TP |
| `configs/` | Configurations des équipements (texte) |
| `images/` | Captures d'écran |

📝 Article LinkedIn : [FR](#) · [EN](#)

**Niveau :** Débutant → intermédiaire
**Prérequis :** TP1 (routage inter-réseaux) et TP2 (DHCP)
**Objectif :** Utiliser les listes de contrôle d'accès (ACL) du routeur pour filtrer le trafic entre les deux réseaux : d'abord une ACL standard, puis une ACL étendue.
**Durée estimée :** 60–75 min

---

## 1. Topologie : celle du TP2 + un serveur

```
                          [Server0]
                              |
        [PC0]                 |                  [PC2]
          |                   |                     |
        [Switch0] ------ [Routeur0] ------ [Switch1]
          |                                         |
        [PC1]                                    [PC3]
```

Ajoute **un serveur** (Server-PT) branché sur **Switch0** (réseau 1) :

| Équipement | Réseau | Adresse IP | Configuration |
|---|---|---|---|
| Server0 | 192.168.1.0/24 | **192.168.1.5** (statique) | masque 255.255.255.0, passerelle 192.168.1.1 |

> **Lien avec le TP2 :** 192.168.1.5 fait partie de la plage exclue du DHCP (.1 à .9). C'est exactement le rôle des exclusions : réserver des adresses pour les équipements à IP fixe.

Le service **HTTP** du serveur est activé par défaut dans Packet Tracer (onglet *Services → HTTP*). Vérifie qu'il est bien sur « On ».

**Avant de commencer :** vérifie que tout fonctionne (les 4 PC ont une IP par DHCP, PC0 peut pinguer PC2 et le serveur). On ne filtre jamais un réseau qu'on n'a pas testé avant.

---

## 2. C'est quoi une ACL ?

Une ACL est une liste de règles « autoriser / refuser » que le routeur lit **de haut en bas** pour chaque paquet, sur une interface et dans un sens donné (`in` = entrant, `out` = sortant).

Trois règles à retenir :
1. **Première règle qui correspond = gagnante.** Le routeur s'arrête là. L'ordre est donc crucial.
2. **Refus implicite à la fin.** Si aucune règle ne correspond, le paquet est rejeté (un `deny any` invisible termine toute ACL).
3. **Une ACL ne fait rien tant qu'elle n'est pas appliquée** sur une interface avec `ip access-group`.

| Type | Filtre sur | Où la placer |
|---|---|---|
| **Standard** (numéros 1–99) | l'adresse IP **source** uniquement | **près de la destination** |
| **Étendue** (numéros 100–199, ou nommée) | source, destination, protocole, port | **près de la source** |

**Le masque inversé (wildcard) :** dans une ACL, on n'écrit pas 255.255.255.0 mais son inverse, 0.0.0.255. Un `0` signifie « ce bit doit correspondre », un `255` signifie « peu importe ». Donc `192.168.2.0 0.0.0.255` = tout le réseau 192.168.2.x, et `host 192.168.2.11` = uniquement cette machine.

---

## 3. Partie A — ACL standard : bloquer une machine

**Scénario :** PC3 (192.168.2.11) ne doit plus pouvoir atteindre le réseau 1. Les autres machines du réseau 2 gardent l'accès.

Une ACL standard ne regarde que la source, donc on la place **près de la destination** : sur Gi0/0, dans le sens **sortant** (vers le réseau 1).

```
enable
configure terminal

access-list 10 deny host 192.168.2.11
access-list 10 permit any

interface GigabitEthernet0/0
 ip access-group 10 out
 exit

end
```

La ligne `permit any` est indispensable : sans elle, le refus implicite bloquerait **tout** le monde.

### Vérifications
- Depuis **PC3** : `ping 192.168.1.10` (ou l'IP d'un PC du réseau 1) → doit **échouer**.
- Depuis **PC2** : même ping → doit **réussir**.
- Sur le routeur : `show access-lists` → chaque règle affiche un compteur de paquets (`matches`). C'est ainsi qu'on vérifie qu'une règle fonctionne vraiment.
- `show ip interface GigabitEthernet0/0` → ligne « Outgoing access list is 10 ».

### Nettoyage avant la partie B
```
configure terminal
interface GigabitEthernet0/0
 no ip access-group 10 out
 exit
no access-list 10
end
```

---

## 4. Partie B — ACL étendue : n'autoriser que le web vers le serveur

**Scénario :** les machines du réseau 2 (invités) ne peuvent accéder qu'au **service web (HTTP) du serveur** 192.168.1.5. Tout le reste vers le réseau 1 est interdit.

On filtre près de la source : sur Gi0/1, dans le sens **entrant** (le trafic du réseau 2 qui arrive au routeur).

```
configure terminal

ip access-list extended FILTRE_INVITES
 permit udp any eq 68 any eq 67
 permit tcp 192.168.2.0 0.0.0.255 host 192.168.1.5 eq 80
 deny ip any any
 exit

interface GigabitEthernet0/1
 ip access-group FILTRE_INVITES in
 exit

end
write memory
```

### Pourquoi la première ligne (`permit udp any eq 68 any eq 67`) ?
C'est la requête **DHCP** des clients (port source 68, port destination 67). Sans elle, l'ACL bloquerait les demandes DHCP du réseau 2 : un PC qui renouvelle son bail ne recevrait plus d'adresse. Ça illustre un principe important : **une ACL peut casser des services auxquels on ne pensait pas.**

### Vérifications
- Depuis **PC2**, onglet *Desktop → Web Browser* : `http://192.168.1.5` → la page du serveur **s'affiche**.
- Depuis **PC2** : `ping 192.168.1.5` → **échoue** (ICMP n'est pas autorisé).
- Depuis **PC2** : `ping 192.168.1.10` (PC0) → **échoue**.
- `show access-lists` → les compteurs de `permit tcp` et `deny ip any any` augmentent selon les tests.

> Selon la version de Packet Tracer, un paquet refusé s'affiche par « Request timed out » ou « Destination host unreachable ». Dans les deux cas, c'est bien l'ACL qui bloque.

---

## 5. Les pièges classiques

1. **ACL créée mais pas appliquée.** Sans `ip access-group`, rien ne se passe.
2. **Mauvais sens (`in` / `out`) ou mauvaise interface.** Réfléchis toujours : « d'où vient le paquet, et à quel endroit je l'intercepte ? ».
3. **Ordre des règles.** Si un `permit` large est placé avant un `deny` précis, le `deny` n'est jamais atteint.
4. **Le refus implicite.** Une ACL qui ne contient que des `deny` bloque tout. Pense au `permit` final.
5. **Les ACL sont sans état (stateless).** Dans la partie B, PC0 (réseau 1) ne peut plus pinguer PC2 : la requête part bien, mais la **réponse** de PC2 revient par Gi0/1 en entrée, et l'ACL la refuse. Une ACL classique ne « se souvient » pas qu'une conversation a été initiée de l'autre côté. C'est la limite que les pare-feu à états (comme **FortiGate**, qu'on verra plus tard dans la série) viennent résoudre.
6. **Modifier une ACL numérotée** est pénible : on ne peut pas insérer une ligne au milieu facilement. Les ACL **nommées** (comme `FILTRE_INVITES`) sont plus faciles à maintenir.

---

## 6. Ce qu'il faut retenir

- Une ACL = liste de règles lue de haut en bas, **première correspondance gagnante**, **refus implicite** à la fin.
- Standard = filtre sur la source, à placer près de la destination. Étendue = filtre fin (protocole, ports), à placer près de la source.
- Une ACL doit être **appliquée** sur une interface, dans un sens précis.
- On teste toujours **avant** et **après**, et on lit `show access-lists`.

---

## 7. Pour aller plus loin (TP4)

Les **VLAN** : segmenter un réseau directement au niveau des switches, avant même le routeur.

---

## Captures d'écran

<!-- Ajoute tes captures dans le dossier images/ puis décommente les lignes ci-dessous :
![Topologie](images/topologie.png)
![Vérification](images/verification.png)
-->
