# LAB-17 — Cracker OWASP UnCrackable Android Level 3

## Objectif

Ce lab consiste à analyser l’application **OWASP UnCrackable Android Level 3** afin de retrouver le secret caché.  
L’analyse combine la partie Java avec JADX et la partie native avec Ghidra.

---

## Étape 1 — Analyse Java

L’analyse avec JADX montre que la vérification principale n’est pas faite directement dans le code Java.

La classe `MainActivity` contient une clé statique appelée `xorkey` :

`pizzapizzapizzapizzapizz`

Cette clé est transmise à une fonction native via la méthode `init()`.

<img width="1002" height="500" alt="image" src="https://github.com/user-attachments/assets/43f6db08-56c1-4777-8ae7-c07a9ef746b0" />


---

## Étape 2 — Analyse de la bibliothèque native

La bibliothèque `libfoo.so` a été extraite puis ouverte dans Ghidra.

L’analyse montre que la fonction JNI :

`Java_sg_vantagepoint_uncrackable3_CodeCheck_bar`

contient la logique de validation du secret.
<img width="1003" height="504" alt="image" src="https://github.com/user-attachments/assets/c70de621-ba40-4f6b-bf9a-aaf810e63921" />


<img width="998" height="493" alt="image" src="https://github.com/user-attachments/assets/2aacf818-ed0d-4335-87e2-a6ec362eb642" />


---

## Étape 3 — Identification du payload

La fonction `FUN_001012c0` remplit un buffer local utilisé pour la vérification.

Même si le code est obfusqué, la fin de la fonction permet d’identifier le vrai payload chiffré.

<img width="989" height="264" alt="image" src="https://github.com/user-attachments/assets/6e9632fa-1596-4f3e-941c-f789f0b48b1e" />


---

## Étape 4 — Correction de l’Endianness

L’émulateur Android utilise une architecture ARM en **Little Endian**.  
Les valeurs extraites depuis Ghidra doivent donc être inversées par octets.

```text
0x1549170f1311081d -> 1d0811130f174915
0x15131d5a1903000d -> 0d0003195a1d1315
0x14130817005a0e08 -> 080e5a0017081314
```

Payload final :

```text
1d0811130f1749150d0003195a1d1315080e5a0017081314
```

## Étape 5 — Déchiffrement du secret

Le payload corrigé a été déchiffré avec CyberChef.

Les opérations utilisées sont :

* From Hex
* XOR avec la clé "pizzapizzapizzapizzapizz"

Résultat obtenu :

```text
making owasp great again
```

<img width="1006" height="462" alt="image" src="https://github.com/user-attachments/assets/88457506-ea51-43fa-b0fa-9d7c670bc20b" />


## Étape 6 — Validation

Le secret obtenu a été testé dans l’application Android.
L’application accepte le code, ce qui confirme que l’analyse est correcte.

<img width="379" height="760" alt="image" src="https://github.com/user-attachments/assets/bda90271-f333-428b-9f38-f595ca1e7254" />


## Conclusion

Ce lab montre comment combiner l’analyse Java et l’analyse native pour retrouver un secret protégé dans une application Android.

L’utilisation de JADX, Ghidra et CyberChef a permis d’identifier la clé, de reconstruire le payload et de déchiffrer le flag final.
