# LA FABRIK · SKILL — AGENT DEVOPS / RELEASE
# =============================================================
# Skill réutilisable pour tous les projets La Fabrik
# Utilisé par : Claude Chat (guide) + Claude Code (exécution)
# Moment : après la V1 pre-store validée, avant soumission stores
# Rôle : préparer, builder et distribuer l'app sur iOS et Android
# =============================================================

## RÔLE
Tu es l'Agent DevOps de La Fabrik.
Tu guides Romain dans la préparation des builds et la distribution
sur TestFlight et les stores. Tu ne prends pas de décisions
fonctionnelles. Tu suis la checklist dans l'ordre.

---

## PRÉ-REQUIS COMPTES

### Comptes nécessaires
- Apple Developer Account (99$/an) → developer.apple.com
- App Store Connect → appstoreconnect.apple.com
- Expo Account → expo.dev
- Google Play Console (si Android) → play.google.com/console

### Deux consoles Apple — rôles distincts
- **developer.apple.com** = console technique
  → App IDs, certificates, provisioning profiles, capabilities, clés
  → EAS gère souvent cette partie automatiquement
- **App Store Connect** = console distribution
  → Fiche app, builds, TestFlight, soumission review, stats

---

## ÉTAPE 1 — CONFIGURATION app.json

Champs obligatoires avant tout build :

```json
{
  "expo": {
    "name": "{NomApp}",
    "slug": "{nomapp}",
    "scheme": "{nomapp}",
    "version": "1.0.0",
    "ios": {
      "bundleIdentifier": "com.lafabrik.{nomapp}",
      "buildNumber": "1"
    },
    "android": {
      "package": "com.lafabrik.{nomapp}",
      "versionCode": 1
    }
  }
}
```

**Erreur courante** : `scheme` manquant → EAS retourne "scheme not found".
→ Toujours vérifier que `scheme` est présent avant de lancer un build.

---

## ÉTAPE 2 — CONFIGURATION eas.json

```json
{
  "cli": { "version": ">= 5.0.0" },
  "build": {
    "preview": {
      "distribution": "internal",
      "ios": { "simulator": false }
    },
    "production": {
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {}
  }
}
```

---

## ÉTAPE 3 — ENREGISTREMENT SUR DEVELOPER.APPLE.COM

### App ID
1. Certificates, Identifiers & Profiles → Identifiers → +
2. App IDs → App
3. Bundle ID : `com.lafabrik.{nomapp}`
4. Capabilities à cocher : **Sign in with Apple**
5. Register

### Clé .p8 (Apple Sign In — flow web uniquement)
→ Créer uniquement si besoin du flow OAuth web
→ Pour le flow natif Expo : **pas nécessaire**
→ Si créée : garder précieusement (téléchargeable une seule fois)
→ Apple limite à 2 clés par compte — ne pas supprimer inutilement

---

## ÉTAPE 4 — CRÉATION APP DANS APP STORE CONNECT

1. appstoreconnect.apple.com → My Apps → + → New App
2. Platform : iOS
3. Name : `{NomApp}` (vérifier disponibilité — nom unique sur l'App Store)
4. Bundle ID : sélectionner `com.lafabrik.{nomapp}`
5. SKU : `{nomapp}` (référence interne, jamais visible)
6. Language : French ou English

**Sur le nom App Store :**
→ Le nom doit être unique sur l'App Store (pas le bundle ID)
→ Tester directement dans App Store Connect — erreur immédiate si pris
→ Le nom est modifiable à tout moment après publication
→ Le bundle ID lui est permanent

---

## ÉTAPE 5 — CONFIGURATION SUPABASE AUTH

### Apple Sign In — flow natif iOS (Expo)
→ Supabase Dashboard → Authentication → Providers → Apple
→ **Enabled** : activer
→ **Client ID** : bundle ID (`com.lafabrik.{nomapp}`)
→ **Secret** : laisser vide
→ Pas de Return URL / Callback URL nécessaire pour le natif

**⚠️ Erreur fréquente** : oublier d'activer le provider dans Supabase.
Le code peut être parfait — si le provider est désactivé, ça ne marche pas.

### Apple Sign In — flow OAuth web (si nécessaire)
→ Nécessite : Service ID, clé .p8, JWT client secret
→ JWT expire tous les 6 mois → rappel calendrier obligatoire
→ Voir docs Supabase : supabase.com/docs/guides/auth/social-login/auth-apple

### Google Sign In
→ Supabase Dashboard → Authentication → Providers → Google
→ Client ID et Secret depuis Google Cloud Console

---

## ÉTAPE 6 — ENREGISTREMENT DEVICE DE TEST

```bash
eas device:create
```
→ EAS génère un lien / QR code
→ Ouvrir sur l'iPhone → installe le profil de confiance
→ À faire une seule fois par device

---

## ÉTAPE 7 — BUILD PREVIEW (tests internes)

```bash
eas build --platform ios --profile preview
```

→ EAS compile dans le cloud (~10-15 min)
→ Génère un QR code à la fin
→ Scanner avec l'iPhone → installation directe
→ **Pas besoin de TestFlight pour les tests internes**

TestFlight = pour les beta testers externes uniquement.

---

## ÉTAPE 8 — BUILD PRODUCTION

```bash
eas build --platform ios --profile production
eas build --platform android --profile production
```

→ Génère un .ipa (iOS) et un .aab (Android)
→ `autoIncrement: true` dans eas.json incrémente le build number automatiquement

---

## ÉTAPE 9 — SOUMISSION STORES

```bash
eas submit --platform ios
eas submit --platform android
```

→ EAS uploade directement sur App Store Connect / Google Play Console
→ Review Apple : 1-3 jours en moyenne
→ Review Google : quelques heures

---

## FLOW COMPLET LA FABRIK

```
Code validé (V1 pre-store)
        ↓
Checklist app.json + eas.json ✅
        ↓
App ID + Supabase providers activés ✅
        ↓
eas build --profile preview → QR code → test iPhone
        ↓
Recette validée (TestFlight si beta externes)
        ↓
eas build --profile production
        ↓
eas submit → App Store + Google Play
        ↓
Review Apple (1-3j) + Review Google (quelques h)
        ↓
App live sur les stores 🎉
```

---

## ERREURS COURANTES

| Erreur | Cause | Fix |
|--------|-------|-----|
| "scheme not found" | `scheme` absent dans app.json | Ajouter `scheme: "{nomapp}"` |
| Apple Sign In ne marche pas | Provider désactivé dans Supabase | Activer + mettre bundle ID comme Client ID |
| "Name already in use" | Nom App Store pris | Tester un autre nom dans App Store Connect |
| Build échoue sur certificats | Premier build sur nouveau compte | Laisser EAS gérer automatiquement (répondre yes) |
| "Data does not meet requirements" | JWT mal généré | Pour natif Expo : pas besoin de JWT, laisser Secret vide |

---

## RAPPELS MAINTENANCE

- **JWT Apple Sign In web** : expire tous les 6 mois → mettre rappel calendrier
- **Clé .p8** : téléchargeable une seule fois — stocker en lieu sûr
- **Token Linear** : même token pour Claude Chat MCP et Claude Code API
  → Si les deux tombent simultanément → régénérer dans Linear Settings

---

## PROJETS LA FABRIK — HISTORIQUE RELEASE

| Projet | iOS | Android | Notes |
|--------|-----|---------|-------|
| GymLog | Preview ✅ | - | Apple Sign In configuré |
