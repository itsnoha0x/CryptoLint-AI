# 🔐 CryptoLint AI — Audit Cryptographique Android

Outil d'analyse statique des mauvaises pratiques cryptographiques dans les applications Android, combinant **règles locales** et **IA Qwen2.5-Coder** via Featherless AI.

---

## 📁 Structure du projet

```
cryptolint-ai/
├── backend/
│   ├── app.py              # API Flask — analyse statique + appels IA
│   └── requirements.txt    # Dépendances Python
├── frontend/
│   └── index.html          # Interface single-page (aucun build requis)
├── sample/
│   └── InsecureCryptoExample.java   # Fichier de test avec vulnérabilités
└── README.md
```

---

## ⚙️ Installation

### 1. Backend (Python 3.9+)

```bash
cd backend

# Créer un venv (recommandé)
python -m venv venv
venv\Scripts\activate

# Installer les dépendances
pip install -r requirements.txt

# Configurer les variables (Recommandé via .env)
# Créez un fichier .env dans le dossier backend/ :
# FEATHERLESS_API_KEY=votre_cle
# JAVA_HOME=C:\Program Files\Java\jdk-21.x.x
# JADX_PATH=D:\Chemin\vers\jadx.bat

# Lancer le serveur
python app.py
```

Le backend tourne sur `http://localhost:5000`

### 2. Frontend

Ouvrir simplement `frontend/index.html` dans un navigateur.

> ⚠️ Si vous avez des problèmes CORS, lancez un serveur local :
> ```bash
> cd frontend
> python -m http.server 8080
> # Puis ouvrir http://localhost:8080
> ```

---

## 🔑 Configuration Featherless AI

1. Créer un compte sur [featherless.ai](https://featherless.ai)
2. Générer une API key dans le dashboard
3. L'exporter comme variable d'environnement ou la remplacer dans `app.py` ligne :
   ```python
   FEATHERLESS_API_KEY = os.environ.get("FEATHERLESS_API_KEY", "VOTRE_CLE_ICI")
   ```

**Modèle utilisé :** `Qwen/Qwen2.5-Coder-32B-Instruct`

---

## 🔍 Fonctionnalités

### Analyse statique (locale, 16 règles)

| ID | Sévérité | Description |
|---|---|---|
| HASH_MD5 | 🔴 Critique | Détection de MD5 |
| HASH_SHA1 | 🔴 Critique | Détection de SHA-1 |
| AES_ECB | 🔴 Critique | AES sans mode (ECB par défaut) |
| AES_CBC_NOAUTH | 🟠 Majeur | AES-CBC sans authentification |
| DES_USAGE | 🔴 Critique | DES/3DES obsolète |
| HARDCODED_KEY | 🔴 Critique | Clé hardcodée dans le code |
| STATIC_IV | 🔴 Critique | IV/Nonce statique |
| WEAK_RNG_RANDOM | 🟠 Majeur | java.util.Random pour crypto |
| MATH_RANDOM | 🟠 Majeur | Math.random() pour crypto |
| SHAREDPREFS_KEY | 🔴 Critique | Secret en SharedPreferences |
| INTERNAL_STORAGE_KEY | 🟠 Majeur | Fichier world-readable |
| SSL_ALL_HOSTS | 🔴 Critique | Hostname verification désactivé |
| TRUST_ALL_CERTS | 🔴 Critique | TrustManager acceptant tout |
| HTTP_CLEAR_TEXT | 🟠 Majeur | URL HTTP en clair |
| PKCS5_PADDING | 🟡 Mineur | Padding PKCS5/7 dangereux |
| RSA_SMALL_KEY | 🟠 Majeur | Clé RSA < 2048 bits |

### Analyse IA (Qwen2.5-Coder)

- **Risk Score** (0-100) avec classification risque
- **Résumé exécutif** de la posture de sécurité
- **Scénarios d'attaque** identifiés
- **Impact conformité** (OWASP, GDPR, PCI-DSS)
- **Patch détaillé** par vulnérabilité (sur demande)

---

## 🧪 Test rapide

Utilisez le fichier `sample/InsecureCryptoExample.java` qui contient intentionnellement toutes les vulnérabilités couvertes par CryptoLint AI.

```bash
# Via curl
curl -X POST http://localhost:5000/api/analyze \
  -F "file=@sample/InsecureCryptoExample.java"

# Ou via l'interface web en collant le contenu du fichier
```

---

## 🛠️ API REST

### `POST /api/analyze`
**Fichier :**
```
Content-Type: multipart/form-data
file: <fichier .apk/.java/.kt>
```
**Code JSON :**
```json
{ "code": "...", "filename": "MyClass.java" }
```
**Réponse :**
```json
{
  "success": true,
  "filename": "MyClass.java",
  "stats": { "total": 12, "critique": 7, "majeur": 4, "mineur": 1 },
  "findings": [ { "rule_id": "HASH_MD5", "severity": "critique", ... } ],
  "ai_analysis": { "global_risk_score": 85, "global_summary": "..." }
}
```

### `POST /api/patch/<rule_id>`
Génère un patch IA détaillé pour une vulnérabilité.

### `GET /api/rules`
Liste toutes les règles de détection.

### `GET /api/health`
Statut du serveur.

---

## 🏗️ Architecture

```
[Browser] ──POST /api/analyze──▶ [Flask Backend]
                                      │
                                      ├── Analyse statique (regex rules)
                                      │
                                      └── Featherless AI API
                                              │
                                         Qwen2.5-Coder-32B
                                              │
                                         Risk Reasoning
                                         Patch Generation
```

---

## 🔒 Recommandations générales

| Mauvaise pratique | Bonne pratique |
|---|---|
| MD5 / SHA-1 | SHA-256, SHA-3 |
| AES-ECB | AES-GCM (AEAD) |
| IV statique | `SecureRandom().nextBytes(iv)` |
| `new Random()` | `new SecureRandom()` |
| Clé hardcodée | Android Keystore System |
| SharedPreferences | EncryptedSharedPreferences |
| Trust-all certs | Certificate Pinning (OkHttp) |
| HTTP | HTTPS + network_security_config |
| RSA-1024 | RSA-3072+ ou ECDSA P-256 |

---

## 📚 Références

- [OWASP Mobile Security Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android Security Best Practices](https://developer.android.com/topic/security/best-practices)
- [NIST SP 800-175B — Cryptographic Standards](https://csrc.nist.gov/publications/detail/sp/800-175b/rev-1/final)
- [CWE Crypto Weaknesses](https://cwe.mitre.org/data/definitions/310.html)
