# Audit de « Les Chroniques d'Al-Qamar » — ébauche Acte I (par Vibe Code)

Objet : `index.html` (605 lignes, moteur complet) + `questions-acte1.js` / `Banque_de_questions_Acte_I.json` (12 épreuves × 3 questions × 2 versions, cohérents et valides).
Méthode : lecture intégrale du code, `node --check` (syntaxe OK), vérification scriptée de la banque (IDs uniques, indices de réponse dans les bornes, explications présentes, clés d'illustrations résolues — aucune erreur détectée sur ces points).

---

## 1. Comment fonctionne le jeu en l'état

- **Accueil** : Mode Quête (jouable), Grimoires, Code de sauvegarde, option chronomètre, nouvelle partie. *Entraînement libre* et *Versus* sont des boutons désactivés (itération 5 non commencée).
- **Quête** : le joueur actif (Badr/Othman, alternance automatique) se déplace sur une carte 7×5 (clic ou flèches), ramasse trésors (or, sagesse, potions, parchemins), découvre des décors (+3 XP + anecdote), puis l'épreuve courante se déclenche automatiquement dès qu'il passe à côté du PNJ.
- **Épreuve** : dialogue narratif, puis 3 questions réussies obligatoires (chaque échec est rejoué jusqu'à réussite, la question ratée revient en priorité). Médaille or/argent/bronze selon le nombre d'erreurs, gains d'or, objets. Une « Leçon de l'Encre » récapitule les erreurs avant de continuer.
- **Épreuves spéciales** : T03 = duel tour par tour contre le Pion d'Encre (3 cœurs joueur / 2 cœurs ennemi, actions attaquer/défendre/esquiver + potion) ; T07 = concours chronométré (6 questions, 60 s) ; T12 = conseil du feu de camp avec question libre, puis écran de fin de l'Acte I.
- **Méta** : XP/niveaux (palier 30 XP, titres Écuyer→Légende), Grimoire par joueur (chaque question posée, réponse donnée, bonne réponse, explication — exigence pédagogique respectée), défis secondaires, sac, échanges or→indice/potion, sauvegarde `localStorage` + code recopiable en base64.

---

## 2. Bugs avérés

1. **Trésors définitivement perdus sur citadelle, atlas et pont** (`sceneCarte`, index.html:523 ; `courante`, :488). La carte affichée dépend uniquement de l'étape *non faite* la plus ancienne. Ces trois scènes n'ont qu'une seule épreuve : dès qu'elle est terminée, le héros est téléporté sur la scène suivante sans retour possible. Les trésors/parchemins non ramassés y sont inaccessibles pour toujours, et le défi « lire 4 parchemins » peut devenir mathématiquement infaisable (les parchemins 0, 1 et 2 sont sur ces scènes).
2. **Déclenchement involontaire des épreuves** (`aller`, :496-503). `proche(e)` est testé à *chaque pas* : tout clic (même vers un trésor à l'opposé) fait marcher le héros qui déclenche l'épreuve dès qu'il frôle le PNJ. Aucun moyen de refuser, annuler ou contourner — l'enfant ne contrôle pas vraiment son trajet.
3. **Chronomètre du concours qui coule pendant la lecture** (`lancerConcours`, :266-276 ; `auto:1100`). Le décompte continue pendant l'affichage de la correction/explication (1,1 s par question) : le temps de lecture réel est amputé d'environ 20 %, pénalisant surtout Othman (8 ans).
4. **Répétition espacée fragilisée par les doublons de Grimoire** (`noter`, :210-212). Chaque ratage rejoué ajoute *une nouvelle page* à chaque tentative ; `revoir()` (:309) s'en sort, mais le compteur « X pages » et la complétion en % sont gonflés artificiellement par les multi-tentatives (une même question ratée 3 fois compte 3 fois).
5. **Duel sans re-tentative** (`lancerDuel`, :279-300). Une erreur = action ratée, puis manche suivante : contredit la règle « une erreur entraîne une nouvelle tentative sous une autre forme ». Le duel est de plus presque impossible à perdre (3 cœurs contre 2, riposte annulée par esquive réussie) — aucune tension.
6. **Commentaire mensonger** (`lancerEpreuve`, :239) : « 3 réussites tirées d'une réserve de 6 questions » — les épreuves n'ont que 3 questions. Le mécanisme décrit n'existe plus.
7. **`son -E.potions` dans le duel** : boire une potion consomme une manche (re-render `manche()` sans incrémenter `m` mais le Pion ne riposte pas non plus) — comportement cohérent mais non documenté à l'écran ; un enfant de 8 ans ne peut pas savoir que la potion « coûte » un temps d'action.

## 3. Lacunes vs cahier des charges

- **Les 5 caractéristiques** (attaque, défense, agilité, sagesse, endurance) n'existent pas : sections `COMBAT` (:302) et `ÉCONOMIE` (:304) sont vides ; le duel n'a que 3 actions génériques et aucun chiffre visible.
- **Entraînement libre** (filtre par notion) et **Versus 1 c 1** : non implémentés (boutons désactivés) — itération 5.
- **Boutique du marchand ambulant** : réduite à deux boutons d'échange dans le Sac (:439) ; ni scène, ni catalogue, ni équipement achetable.
- **Niveaux « illimités »** : le titre plafonne à « Légende d'Al-Qamar » dès le niveau 7 (`TITRES[min(3,…)]`) — acceptable, mais rien n'indique ensuite une progression (pas de palier narratif au-delà).
- **Aucun test automatisé** ni garde-fou sur la banque (un JSON malformé est silencieusement remplacé par la banque de repli à 1 épreuve, avec seulement un avertissement discret à l'accueil).

## 4. Redondances

- **Trois copies des questions** : `Banque_de_questions_Acte_I.json`, `questions-acte1.js` (identiques octet près aujourd'hui, vérifié) et `QUESTIONS_EMBED` en dur dans index.html (:127). Triple risque de divergence ; l'embed contredit la règle « jamais de questions en dur dans le code » (repli assumé, mais à assumer *une seule fois*).
- **CSS en couches de correctifs** : `.choix button.bon/.mauvais` et `.retour.ko` définis deux fois avec des couleurs différentes (:46-47 vs :57-58), `.case:hover` surchargé avec `!important` (:63), `.deco` défini deux fois (:60, :71). Le style fonctionne par cascade de rustines d'itérations successives.
- **Logique de médailles dupliquée** : mapping `{or:🥇,argent:🥈,bronze:🥉}` recodé 3 fois (:311, :557, :576).
- **Génération de la piste Boussole** (:527) réimplémente un A* naïf inline, non factorisé.

## 5. Ergonomie / UX

- **Grimoire inaccessible pendant les questions** : `hud(1)` masque les boutons Quête/Sac/Grimoire — on ne peut pas relire une leçon au moment où elle servirait.
- **Aucun bouton « retour à l'accueil » pendant une question** ni moyen d'abandonner proprement une épreuve (seul le rechargement de page en sort).
- **Indicateur de tour** : présent (bandeau + écran « C'est ton tour »), mais le concours change `E.actif` sans écran de transition — l'enfant peut ne pas remarquer que la question s'adresse à l'autre.
- **`confirm()` natif** pour la nouvelle partie : jure avec le style enluminure.
- **Lecture vocale** (`parler`) : bouton présent sur les questions, mais pas sur les dialogues du PNJ avant l'épreuve ni sur la Leçon de l'Encre — incohérent pour un lecteur débutant.
- **Accessibilité** : `aria-label` sur un seul bouton, focus visible correct, `prefers-reduced-motion` bien géré (bon point), mais les contrastes grenat/or sur bleu outremer sont limites sur certains petits textes (`.pagination`).

## 6. Architecture / modulabilité

- **État global mutable** (`E`, `hero`, `verrou`, `sceneActive`, `enCours`, `RETOUR`, `sansFondu`) + fonctions globales + `onclick` inline dans les templates : testable seulement à la main, aucun isolement.
- **Rendu par ré-écriture complète** (`ecran` = `innerHTML`) à chaque pas de 140 ms : fonctionne à cette échelle, mais interviendra mal avec des animations de transition de case.
- **Navigation « RETOUR » implicite** : `RETOUR` global réassigné à la main dans chaque écran (5 endroits) — une pile de navigation (`RETOUR.push/pop`) éliminerait ce risque d'écran de retour cassé.
- **Données de scène dispersées** : `DECORS`, `TEX`, `ETAPES`, `FAIT`, `ANEC`, `ICO`, `OBJ` — la « scène » est éclatée en six structures à maintenir en phase ; une seule structure `SCENES = {citadelle:{…}}` réduirait les désynchronisations.
- **Templates HTML en chaînes** : interpolation directe des contenus du JSON (`${v.enonce}`) sans échappement — sans risque en local, mais toute question contenant `<` ou `&` casserait l'affichage (aucune des 36 n'en contient aujourd'hui, vérifié).
- **Bon points à conserver** : sections commentées respectées, hors-ligne total (aucun CDN, `onerror` silencieux :119), sauvegarde robuste (`try/catch` + `compl()` de migration), banque effectivement débranchée du moteur, feedback bienveillant (sons doux, jamais de rouge agressif, Encre Noire responsable des échecs), typographie 20 px adaptée.

## 7. Feuille de route suggérée (une itération à la fois, par ordre de dépendance)

1. **Fix trésors perdus** : autoriser le retour sur les scènes terminées (sélecteur de scène ou « rester sur place » après une épreuve) — corrige aussi le défi parchemins.
2. **Fix déclenchement** : déclencher l'épreuve seulement quand le héros *s'arrête* volontairement à côté du PNJ (clic sur le PNJ ou confirmation).
3. **Chronomètre en pause** pendant l'affichage des corrections du concours.
4. **Navigation** : pile de retour + bouton accueil permanent ; accès Grimoire/Sac pendant les questions.
5. **Duel** : implémenter les 5 caractéristiques visibles (attaque/défense/agilité/sagesse/endurance) et la re-tentative après échec.
6. **Boutique du marchand** en scène dédiée (indice, potion, équipement).
7. **Itération 5** : Entraînement libre (filtre par notion depuis `BANQUE`) puis Versus.
8. **Hygiène** : dédupliquer la banque (JSON seul + fetch local OU js seul, pas les trois), nettoyer le CSS en couches, corriger le commentaire « réserve de 6 questions », échapper les contenus du JSON à l'affichage.

---

*Audit read-only : aucun fichier du jeu n'a été modifié.*
