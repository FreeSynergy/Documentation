# fs-auth

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-auth` ist der Authentifizierungs-Service für FreeSynergy.
Er definiert 4 Protokoll-Traits und implementiert sie via Kanidm (Adapter Pattern).
Consumer-Code arbeitet ausschließlich gegen die Traits — nie gegen Kanidm direkt.

---

## Architektur

```
OAuthProvider-Trait   ← Strategy Pattern
ScimProvider-Trait    ← Strategy Pattern
SsoProvider-Trait     ← Strategy Pattern
PamProvider-Trait     ← Strategy Pattern
       ↑
KanidmBackend         ← Adapter Pattern: implementiert alle 4 Traits
```

---

## Protokoll-Traits

### OAuthProvider

| Methode          | Beschreibung                           |
|------------------|----------------------------------------|
| `authorize`      | Authorization-URL generieren           |
| `exchange_code`  | Authorization Code → Token-Paar        |
| `refresh_token`  | Access Token erneuern                  |
| `revoke_token`   | Token widerrufen                       |

### ScimProvider

| Methode        | Beschreibung                    |
|----------------|---------------------------------|
| `create_user`  | Nutzer anlegen (SCIM 2.0)       |
| `update_user`  | Nutzer aktualisieren            |
| `delete_user`  | Nutzer löschen                  |
| `list_users`   | Alle Nutzer abfragen            |

### SsoProvider

| Methode            | Beschreibung                 |
|--------------------|------------------------------|
| `sso_login`        | SSO-Login-URL generieren     |
| `validate_session` | Session validieren           |
| `logout`           | Session beenden              |

### PamProvider

| Methode           | Beschreibung                    |
|-------------------|---------------------------------|
| `pam_authenticate`| PAM-Authentifizierung           |
| `pam_authorize`   | PAM-Autorisierung               |

---

## KanidmSetupWizard (Phase 5.1)

`fs-manager-auth` enthält einen **State Machine Wizard** zur Erstkonfiguration von Kanidm nach dem Install.

**Schritte:**
```
Domain → AdminAccount → OidcClients → Confirm → Done
```

| Schritt       | Eingabe                             |
|---------------|-------------------------------------|
| Domain        | Kanidm-Domain, z. B. `idm.example.com` |
| AdminAccount  | Admin-Benutzername + Passwort (≥ 8 Zeichen) |
| OidcClients   | OIDC-Clients (ID, Anzeigename, Redirect-URI) — optional, wiederholbar |
| Confirm       | Prüfen + Anwenden                   |
| Done          | `KanidmConfig` bereit               |

**Besonderheiten:**
- `skip_oidc()` — OidcClients-Schritt überspringen (Clients später im Manager hinzufügen)
- `KanidmConfig` ist das Output-Objekt; serialisierbar via fs-config
- i18n: alle Texte in `fs-i18n/locales/{lang}/auth-setup.ftl`

---

## Repo

- Lokal: `/home/kal/Server/fs-auth/`
- GitHub: `git@github.com:FreeSynergy/fs-auth.git`
- Manager: `/home/kal/Server/fs-managers/auth/` (`fs-manager-auth`)

---

Weiter: [← Index](../INDEX.md)
