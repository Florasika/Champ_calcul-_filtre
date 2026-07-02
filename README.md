# 🧮 Jour 2 / 10 — Tableau : Champs Calculés & Filtres

> **Série : 10 Days of Tableau** · Jour 2/10  
> Concepts : Champs calculés · Filtres de dimensions/mesures · Filtres contextuels · Paramètres

---

## 📁 Fichiers du projet

```
day-02-calculated-fields/
│
├── ventes_tableau_j2.csv    ← 188 lignes (+ Coût Unitaire et Coût Total vs Jour 1)
└── README.md
```

---

## 🚀 Connexion des données

```
1. Ouvrir Tableau
2. Fichier → Se connecter → Fichier texte → ventes_tableau_j2.csv
3. Vérifier les types : Date = calendrier, textes = Abc, chiffres = #
4. Cliquer "Feuille 1"
```

---

## 🔑 PARTIE 1 — Champs Calculés

Un champ calculé est une **formule que tu crées** dans Tableau — comme une colonne Excel calculée, mais elle vit dans Tableau, pas dans le CSV.

### Créer un champ calculé

```
Analyse → Créer un champ calculé...
    OU
Clic droit dans le panneau Données → Créer un champ calculé
```

---

### Calcul 1 — Marge Brute (€)

```
Nom : Marge Brute
Formule :
    SUM([Montant]) - SUM([Coût Total])
```
Affiche la marge en euros : ce que la vente rapporte après le coût des produits.

---

### Calcul 2 — Taux de Marge (%)

```
Nom : Taux de Marge
Formule :
    (SUM([Montant]) - SUM([Coût Total])) / SUM([Montant])
```
Mettre en format % : clic droit sur le champ → Format → Pourcentage, 1 décimale.

---

### Calcul 3 — Atteinte Objectif (%)

```
Nom : Atteinte Objectif
Formule :
    SUM([Montant]) / SUM([Objectif])
```
Indique si le vendeur/produit est au-dessus ou en-dessous de son objectif.

---

### Calcul 4 — Statut Objectif (logique IF)

```
Nom : Statut Objectif
Formule :
    IF SUM([Montant]) >= SUM([Objectif])
    THEN "✓ Atteint"
    ELSE "✗ Non atteint"
    END
```
Crée une dimension calculée — utilisable pour colorer ou filtrer.

---

### Calcul 5 — Panier Moyen

```
Nom : Panier Moyen
Formule :
    SUM([Montant]) / COUNT([Montant])
```
Valeur moyenne par transaction — utile pour comparer les segments clients.

---

### Calcul 6 — Variation vs Objectif (€)

```
Nom : Écart Objectif
Formule :
    SUM([Montant]) - SUM([Objectif])
```
Positif = au-dessus, négatif = en dessous. Afficher en couleur rouge/vert avec une MFC.

---

## 🔑 PARTIE 2 — Filtres

### Types de filtres dans Tableau (ordre d'exécution)

```
1. Filtres d'extraits (Extract Filters)     ← agissent sur la source
2. Filtres de source de données             ← avant tout calcul
3. Filtres contextuels                      ← référence pour les autres filtres
4. Filtres de dimensions                    ← filtrent les membres (texte, date)
5. Filtres de mesures                       ← filtrent sur les valeurs agrégées
6. Filtres de calculs de table              ← après les calculs de table
```

---

### Filtre de dimension — par Catégorie

```
Méthode :
    Glisser "Catégorie" → zone Filtres
    Cocher les catégories voulues → OK
    
    Clic droit sur le filtre → "Afficher le filtre"
    → Un contrôle interactif apparaît sur la vue
```

---

### Filtre de mesure — par Montant minimum

```
Méthode :
    Glisser "Montant" → zone Filtres
    Choisir : Somme → Plage de valeurs
    Définir un minimum (ex. 5 000€)
    → Seuls les produits/vendeurs au-dessus du seuil apparaissent
```

---

### Filtre de date — par Trimestre

```
Méthode :
    Glisser "Date" → zone Filtres
    Sélectionner : Plage de dates OU Années/Trimestres
    
    Astuce : choisir "Trimestre" permet de filtrer T1, T2, T3, T4
    indépendamment de l'année
```

---

### Filtre contextuel — définir un contexte de référence

```
Contexte = filtre qui s'applique en PREMIER et réduit la base
           avant que les autres filtres s'appliquent

Usage typique : filtrer sur la Région, puis calculer le TOP 5
des produits DANS cette région (pas le top 5 global)

Créer un filtre contextuel :
    Clic droit sur un filtre dans la zone Filtres
    → "Ajouter au contexte"
    → Le filtre devient gris avec une icône contexte
```

---

### Filtre TOP N — Top 5 des vendeurs

```
Méthode :
    Glisser "Vendeur" → zone Filtres
    Onglet "Top" → Par champ → Top 5 → par SUM(Montant)
    
    Si un filtre contextuel de région est actif :
    → Le Top 5 est calculé DANS la région sélectionnée
    → Sans filtre contextuel, c'est le Top 5 global (ignorant la région)
```

---

## 🔑 PARTIE 3 — Paramètres

Un paramètre est une **valeur modifiable par l'utilisateur** — utilisable dans les formules et les filtres.

### Paramètre — Seuil de marge minimum

```
Clic droit dans le panneau Données → Créer un paramètre...

    Nom : Seuil Marge Minimum
    Type de données : Flottant
    Valeur actuelle : 30
    Minimum : 0
    Maximum : 100
    Incrément : 5

Afficher le paramètre :
    Clic droit sur le paramètre → "Afficher le contrôle du paramètre"
```

### Utiliser le paramètre dans un champ calculé

```
Nom : Taux Marge OK
Formule :
    IF (SUM([Montant]) - SUM([Coût Total])) / SUM([Montant]) * 100
    >= [Seuil Marge Minimum]
    THEN "✓ Marge OK"
    ELSE "✗ Marge faible"
    END
```
Maintenant le seuil est dynamique — l'utilisateur peut le modifier via le curseur.

---

## 🚀 Construire la vue — Dashboard Marge & Performance

### Feuille 1 — Marge par Produit

```
Lignes    : Produit
Colonnes  : SUM(Montant)
Couleur   : Taux de Marge (dégradé vert-rouge)
Étiquette : Taux de Marge
Filtre    : Statut Objectif → "✓ Atteint" uniquement (optionnel)
```

### Feuille 2 — Atteinte par Vendeur

```
Lignes    : Vendeur
Colonnes  : Atteinte Objectif (en %)
Ligne de référence : Ajouter une ligne à 100%
    → Analyse → Ligne de référence → Constante → 1.0
Couleur   : Statut Objectif (vert/rouge)
```

### Feuille 3 — Évolution Marge Brute mensuelle

```
Colonnes  : Mois
Lignes    : Marge Brute
Type      : Ligne avec points
Filtre contextuel : Région (afficher le contrôle)
```

---

## 💡 Formules Tableau à connaître

| Fonction | Syntaxe | Description |
|---|---|---|
| Condition | `IF condition THEN a ELSE b END` | Logique conditionnelle |
| Multiple | `IIF(condition, vrai, faux)` | Version courte |
| Cas | `CASE [champ] WHEN val THEN res END` | Comme CASE SQL |
| Texte | `STR([Nombre])` | Convertit en texte |
| Date | `YEAR([Date])`, `MONTH([Date])` | Extraction date |
| Arrondi | `ROUND([Mesure], 2)` | 2 décimales |
| Null | `ISNULL([Champ])` | Teste le null |
| Cumul | `RUNNING_SUM(SUM([Montant]))` | Calcul de table |

---


---

⭐ **Si ce projet t'aide, mets une étoile !**
