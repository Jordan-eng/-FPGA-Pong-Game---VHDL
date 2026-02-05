# 🏓 FPGA Pong Game - VHDL

Une implémentation matérielle classique du jeu **Pong** entièrement écrite en **VHDL**. Ce projet génère un signal vidéo VGA (640x480) et gère la logique du jeu, les collisions et les entrées joueurs directement sur un FPGA.

*(N'oubliez pas d'ajouter une photo ou un GIF de votre écran en action ici !)*

## 📋 Fonctionnalités

* **Affichage VGA :** Résolution 640x480 @ 60Hz.
* **Multijoueur Local :** Contrôle de deux raquettes via des boutons poussoirs.
* **Physique de Balle :**
* Déplacement fluide.
* Rebonds sur les raquettes.
* Rebonds sur les murs (Haut/Bas).
* *Mode Entraînement/Infini :* La balle rebondit sur les murs gauche et droit au lieu de compter des points (Code actuel).


* **Gestion des Collisions :** Algorithme AABB (Axis-Aligned Bounding Box) pour des rebonds précis.

## 🛠 Matériel Requis

* **Carte FPGA :** (Ex: Digilent Basys 3, Nexys A7, ou toute carte avec un port VGA).
<img width="328" height="276" alt="image" src="https://github.com/user-attachments/assets/db4eed8b-d13d-4819-8cf2-591272702c35" />
<img width="328" height="276" alt="image" src="https://github.com/user-attachments/assets/db4eed8b-d13d-4819-8cf2-591272702c35" />

* *Horloge :* 100 MHz.


* **Écran :** Moniteur avec entrée VGA.
* **Câble :** Câble VGA standard.

## 📂 Structure du Projet

* `Pong_Game.vhd` : Le module **Top Level**. Il contient :
* La machine à états du jeu.
* La logique de mouvement de la balle et des raquettes.
* Le générateur de pixels (dessin des rectangles).


* `VGA_Controller.vhd` : Module générant les signaux de synchronisation `HSYNC` et `VSYNC` ainsi que les coordonnées actuelles du pixel (`h_addr`, `v_addr`).
* `Constraints.xdc` : Fichier de contraintes liant les ports VHDL aux broches physiques du FPGA.

## ⚙️ Paramètres Techniques

| Paramètre | Valeur | Description |
| --- | --- | --- |
| **Clock Input** | 100 MHz | Horloge système de base. |
| **Game Clock** | ~40 Hz | Diviseur de fréquence pour la logique du jeu (vitesse de la balle). |
| **Résolution** | 640 x 480 | Standard VGA. |
| **Taille Balle** | 8x8 pixels | Carré bleu. |
| **Taille Raquette** | 8x60 pixels | Rectangles blancs. |

## 🚀 Installation et Utilisation

1. **Cloner le repo :**
```bash
git clone https://github.com/votre-username/fpga-pong-vhdl.git

```


2. **Ouvrir le projet :** Lancez Vivado (ou votre outil FPGA préféré) et créez un nouveau projet.
3. **Importer les sources :** Ajoutez `Pong_Game.vhd` et `VGA_Controller.vhd`.
4. **Configurer les contraintes :** Ajoutez le fichier `.xdc` correspondant à votre carte (assurez-vous que les broches `CLK100MHZ`, `VGA`, et les boutons sont corrects).
5. **Générer le Bitstream :** Lancez la synthèse, l'implémentation et la génération du bitstream.
6. **Programmer :** Connectez votre carte via USB et programmez-la.

---

## ⚠️ Difficultés Majeures et Solutions

Ce projet a présenté plusieurs défis techniques spécifiques à la conception matérielle (Hardware Design) par rapport à la programmation logicielle classique.

### 1. Le "Tunneling" (Balle traversant les murs)

* **Problème :** À certaines vitesses, la balle traversait le mur du haut ou les raquettes sans rebondir. Cela est dû au fait que la position de la balle est mise à jour de manière discrète (par "sauts" de plusieurs pixels). Si la balle se trouve à `y=2` et se déplace de `-4`, elle atterrit à `y=-2`, sautant par-dessus la condition `y=0`.
* **Solution :**
* Réduction de la vitesse de déplacement (`ball_dy`) à 1 ou 2 pixels par cycle.
* Ajustement du `GAME_SPEED_DIVIDER` pour trouver l'équilibre entre fluidité et détection précise.



### 2. Le "Sticky Ball" (Balle collée au mur/raquette)

* **Problème :** Parfois, la balle restait coincée à l'intérieur d'une raquette ou d'un mur, vibrant indéfiniment. Cela arrive si la direction est inversée mais que la balle est encore physiquement dans la zone de collision au cycle suivant.
* **Solution :** Implémentation d'une logique de "rejet" (Push logic). Lors d'une collision, on ne se contente pas d'inverser la direction, on force aussi la coordonnée de la balle à sortir de la zone de collision (ex: `ball_y <= 0 + 1`).

### 3. Erreurs de Placement (IO/Clock Placement Failure)

* **Problème :** Erreur critique lors de l'implémentation (`DRC Error`) indiquant que l'horloge n'était pas sur une broche dédiée.
* **Solution :** Compréhension de l'architecture FPGA : les horloges rapides doivent utiliser des broches **CCIO** (Clock Capable IO) et des buffers globaux (`BUFG`). Modification du fichier de contraintes (`.xdc`) pour assigner l'horloge à la bonne broche physique de la carte.

### 4. Affichage du Score (Font ROM)

* **Problème :** Afficher du texte sur un écran VGA sans processeur graphique est complexe. Il faut définir chaque pixel de chaque chiffre manuellement via une mémoire morte (ROM).
* **Décision :** Pour cette version, la fonctionnalité de score a été retirée pour se concentrer sur la fluidité du gameplay et la physique des rebonds, transformant le jeu en un mode "échange infini".

---

## 🔮 Améliorations Futures

* [ ] Réintégrer le système de score avec une Font ROM.
* [ ] Ajouter un écran "Game Over" et un état "Reset".
* [ ] Augmenter la vitesse de la balle progressivement au fil des échanges.
* [ ] Ajouter des couleurs ou des motifs de fond.

---

**Auteur :** Jordan TOE
**Date :** 2025
