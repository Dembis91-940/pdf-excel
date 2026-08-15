# PDF2Données

Micro-outil web : **extraire les données d'un PDF (factures, relevés, tableaux) en format exploitable (CSV / tableau)**, directement dans le navigateur, **sans envoyer le fichier sur un serveur** (privacy-first, RGPD).

Marché visé : comptables, indépendants, artisans, équipes RH — tous ceux qui reçoivent des PDF et retapent les données à la main.

## Livrables

| Fichier | Rôle |
|---|---|
| `index.html` | Landing page (accueil, arguments, offres, formulaire EmailJS, FAQ) |
| `outil.html` | L'outil d'extraction — fonctionne réellement dans le navigateur |
| `README.md` | Ce document |

## L'outil (`outil.html`) — comment ça marche

1. **Dépôt** : glisser-déposer (ou sélection) d'un PDF dans la zone prévue.
2. **Extraction locale** : le PDF est lu par la bibliothèque [pdf.js](https://mozilla.github.io/pdf.js/) (CDN), directement dans le navigateur. **Aucun fichier n'est envoyé sur un serveur.**
3. **Détection de tableaux par motifs** : les lignes de texte sont regroupées, puis un histogramme des séparateurs de colonnes (milieux entre tokens adjacents) identifie les colonnes régulières retrouvées sur au moins deux lignes. Chaque ligne est ensuite assignée aux colonnes les plus proches et les rangées consécutives sont regroupées en blocs = tableaux.
4. **Export réel** : boutons « Copier CSV » (presse-papiers) et « Télécharger CSV » (Blob, encodage UTF-8 avec BOM, séparateur `;` compatible Excel). L'export Excel natif (`.xlsx`) est disponible sur l'offre Business (SheetJS chargé à la demande).
5. **Quota** : compteur de pages en `localStorage` (3 pages/mois offertes, 100 pour Pro, illimité pour Business), réinitialisation automatique le 1er du mois, message d'upgrade quand le quota est atteint.
6. **Historique local** des extractions sur les offres Pro et Business.

## Offres

| Offre | Prix | Pages/mois | Fonctionnalités clés |
|---|---|---|---|
| Gratuit | 0 € | 3 | Extraction basique (1 tableau), aperçu, copie CSV |
| Pro | 9 €/mois | 100 | Tous les tableaux, export CSV complet, historique local |
| Business | 19 €/mois | Illimité | Tableaux multiples, export Excel (.xlsx), historique illimité |

Commande et contact via le formulaire EmailJS de la landing page (confirmation par email ; paiement par virement ou message privé, sans engagement).

## Limites assumées (honnêteté produit)

- **L'extraction est basée sur des motifs** : elle gère très bien les **tableaux simples aux colonnes régulières** (factures générées par logiciel, relevés, devis, bordereaux). Elle ne gère pas les mises en page complexes (tableaux imbriqués, cellules fusionnées, colonnes irrégulières, PDF multi-colonnes type articles de presse).
- **Les PDF scannés ou photographiés ne sont pas extractibles** : ce sont des images, il n'y a pas de texte à lire. Ils nécessitent une **reconnaissance OCR**, prévue sur les offres payantes (Pro/Business) et non incluse dans cette version. L'outil détecte les pages sans texte et l'indique clairement à l'utilisateur — il ne prétend jamais avoir extrait ce qu'il n'a pas pu lire.
- **PDF protégés par mot de passe** : non pris en charge dans cette version (message explicite).
- **Le quota et le plan sont gérés côté client** (localStorage) : c'est un MVP. La monétisation réelle (paiement, activation des plans) se fait par commande manuelle via le formulaire EmailJS — virement ou message privé — puis activation du plan côté utilisateur.
- **Confidentialité** : l'extraction étant 100 % locale, aucune donnée ne transite. C'est la garantie RGPD du produit, mais cela implique qu'aucune donnée n'est récupérable en cas de perte du navigateur (l'historique est local).

## Technique

- Pure HTML/CSS/JS, aucun framework ni build.
- pdf.js 3.11.174 (CDN cdnjs) pour la lecture des PDF.
- SheetJS (xlsx) chargé à la demande pour l'export Excel (Business).
- EmailJS (`@emailjs/browser@4`, chargé à la demande) pour le formulaire de commande : `serviceId = service_cy1ytdb`, `templateId = template_xpo58cv`, `publicKey = 8Pui4ZEqxW2jRVF7h`, payload `{ site, name, email, question }`.
- Serveur requis au premier chargement (CDN) : servir le dossier via n'importe quel serveur statique (`python3 -m http.server`, GitHub Pages, Netlify…) — le fichier peut aussi être ouvert en local, mais le chargement de pdf.js et du worker nécessite Internet au premier affichage.

## Tester

```bash
cd ~/Documents/livrables/pdf-excel
python3 -m http.server 8080
# puis ouvrir http://localhost:8080/outil.html
```

Le sélecteur de plan dans l'outil permet de simuler les trois offres et de vérifier les messages d'upgrade.
