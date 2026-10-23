# iOS Glass Dark — thème Mailspring

Thème d'interface (`"theme": "ui"`) pour Mailspring, inspiré d'iOS 26 (Liquid Glass), en mode sombre.
Uniquement du Less : pas de JavaScript, pas de build, pas de dépôt git.

## Fichiers

| Fichier | Rôle |
|---|---|
| `package.json` | Métadonnées du thème. Monter `version` à chaque modification. |
| `styles/ui-variables.less` | Palette (`@ios-*`) et surcharge des variables Mailspring (`@background-primary`, `@text-color`…). N'émet aucun CSS. |
| `styles/ios-glass.less` | Toutes les règles de l'interface. |
| `styles/email-frame.less` | Règles injectées **dans l'iframe** de chaque email (corps du message). |

## Règles de chargement (important)

- Mailspring (`app/src/package.ts`, `loadStylesheets`) : si `styles/` contient un fichier `index.*`, **seul ce fichier est chargé**. Sinon, tous les fichiers `*.less` / `*.css` du dossier sont chargés.
  → Ne jamais créer de `styles/index.less`, sinon `email-frame.less` n'est plus chargé.
- `email-frame.less` : Mailspring (`email-frame-styles-store.ts`) récupère toute feuille dont le chemin contient `email-frame.less`, retire le préfixe `.ignore-in-parent-frame` et l'injecte dans l'iframe. Toute règle de ce fichier doit donc être sous `.ignore-in-parent-frame { … }` (sans effet dans la fenêtre principale).
- `@import "ui-variables";` résout le `ui-variables.less` du thème, qui importe lui-même `base/ui-variables` (variables par défaut de Mailspring).

## Contraintes du compilateur Less

Mailspring utilise un Less ancien. Une erreur de compilation affiche une fenêtre « Error compiling Less stylesheet » et le fichier entier est ignoré.

- Interdit : drapeau `i` dans les sélecteurs d'attribut (`[style*="x" i]` → `expected ']' got 'i'`).
- Éviter `:is()` / `:where()` dans les sélecteurs. `:has()` passe (utilisé dans `ios-glass.less`).
- Vérifier avant de livrer, avec un stub pour `base/ui-variables` :

```sh
S=$(mktemp -d); mkdir -p $S/base && : > $S/base/ui-variables.less
for f in ios-glass email-frame ui-variables; do
  npx -y less@2.7.3 --no-color styles/$f.less --include-path=$S > /dev/null && echo "$f OK"
done
```

## Installation et test

- Dossier de travail : `~/Downloads/ios-glass-mailspring/ios-glass-dark`
- Dossier installé (celui que Mailspring lit) : `~/Library/Application Support/Mailspring/packages/ui-ios-glass-dark`
- Après chaque modification, copier vers le dossier installé (ne pas toucher aux fichiers qui n'existent que là-bas, ex. `aaa`) :

```sh
P=~/Library/Application\ Support/Mailspring/packages/ui-ios-glass-dark
cp package.json "$P/" && cp styles/*.less "$P/styles/"
diff -rq --exclude=.DS_Store --exclude=aaa . "$P"
```

- Recharger : menu Developer > Reload, ou quitter (Cmd+Q) et relancer. Sur une erreur de compilation, cliquer « Continuer », pas « Reset Theme and Restart » (qui réinitialise les thèmes).
- Inspecter : Developer > Toggle Developer Tools.
- Réglages utilisateur : `~/Library/Application Support/Mailspring/config.json`. Base locale (lecture seule, `sqlite3 -readonly`) : `edgehill.db`, table `MessageBody` pour le HTML des emails.

## Conventions du code

- Commentaires et titres de section en français.
- Le fichier est organisé en sections versionnées (`// v1.1 — …`, `// v1.2 — …`). Une correction s'ajoute en nouvelle section en fin de fichier, ou modifie en place la règle concernée si elle est simple à localiser.
- Les styles de Mailspring sont souvent plus spécifiques : surcharger avec `!important` et un sélecteur au moins aussi spécifique.
- Couleurs uniquement via les variables `@ios-*` de `ui-variables.less`, pas de valeurs en dur dans les règles (sauf `#fff` sur l'accent).
- Mixins disponibles : `.ios-glass()` (surface en verre flou), `.ios-capsule()` (rayon 999px).

## Palette actuelle

| Zone | Variable | Valeur |
|---|---|---|
| Barre latérale | `@ios-panel-sidebar` | `#242424` |
| Liste + lecture | `@ios-panel` | `#1B1B1B` |
| Séparateur liste / lecture | `@ios-panel-separator` | `#313131` (aucun trait entre barre latérale et liste) |
| Cartes de message, « Écrire une réponse » | `@ios-card` / `@ios-card-soft` | `#242424` / `#202020` |
| Accent | `@ios-accent` | `#0A84FF` |

Préférences exprimées par l'utilisateur : pas de coins arrondis sur les barres et la fenêtre de rédaction, pas de contour bleu dans l'éditeur, panneaux à plat.

## Pièges déjà rencontrés

- **Bouton Envoyer (`.button-dropdown.btn-emphasis`)** : il est en `font-size: 0` (pour coller les deux moitiés), donc `.secondary-items` doit rétablir la taille. Le menu intérieur `.menu` ne doit pas recevoir le style verre (double calque). Mailspring place le menu en `top: -100%` : surchargé pour s'ouvrir au-dessus du bouton.
- **Anneau de focus** : la règle globale `*:focus-visible` (anneau bleu) touchait l'éditeur ; neutralisée sur `.RichEditor-*`, `[contenteditable]` et le composer. Le mixin `.btn()` de Mailspring ajoute aussi un `outline` au focus.
- **Fond noir derrière le texte d'un email** : l'utilisateur a `core.reading.emailRenderMode: "dark"`. Ce mode peint `#inbox-html-wrapper` en blanc puis applique `filter: invert(100%) hue-rotate(180deg)` au `body`, ce qui donne un bloc noir. Mailspring ajoute aussi `.has-background` (fond blanc) à des emails sans fond. Corrigé dans `email-frame.less` en rendant `#inbox-html-wrapper` transparent : seul le texte est inversé.
- **`color-scheme`** : un email qui déclare `color-scheme: dark` alors que l'app est en `normal` fait peindre à Chromium un fond opaque dans l'iframe. `email-frame.less` force `color-scheme: light` sur `html`.
- **Arrondis** : Mailspring arrondit `.composer-action-bar-wrap` ; avec un `border-top`, le trait se courbe aux extrémités.

## Documentation et sources de référence

Mailspring n'a pas de documentation de thème détaillée : la référence est le code source.

- Dépôt : https://github.com/Foundry376/Mailspring
- Variables par défaut : `app/static/style/base/ui-variables.less` (celles qu'importe `base/ui-variables`)
- Styles de base : `app/static/style/` (`workspace.less`, `buttons.less`, `email-frame.less`, `components/button-dropdown.less`, `components/menu.less`)
- Composer : `app/internal_packages/composer/styles/composer.less`, `lib/composer-view.tsx`, `lib/send-action-button.tsx`
- Iframe des emails : `app/internal_packages/message-list/lib/email-frame.tsx`, `email-frame-styles-store.ts`
- Chargement des paquets : `app/src/package.ts` ; thèmes : `app/src/theme-manager.ts`
- Thèmes intégrés à imiter : `app/internal_packages/ui-dark`, `ui-taiga` (exemple d'`email-frame.less`)
- Fichiers bruts : `https://raw.githubusercontent.com/Foundry376/Mailspring/master/<chemin>`
- Guide officiel de création de thème : https://github.com/Foundry376/Mailspring-Theme-Starter (le dossier `packages/theme-starter` installé localement en est une copie)
