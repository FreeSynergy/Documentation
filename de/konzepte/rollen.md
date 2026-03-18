# Rollen-System

[← Zurück zum Index](../INDEX.md) | [Architektur](../architektur/uebersicht.md)

---

## Warum Rollen statt Typen?

**Alt (Typen):** Ein Service hat EINEN Typ: `iam/kanidm`. Das impliziert eine Hierarchie (iam → kanidm) und schränkt ein. Was wenn Kanidm auch RADIUS kann? Dann ist `iam/kanidm` zu eng.

**Neu (Rollen):** Ein Service hat EINEN Namen: `kanidm`. Und er deklariert beliebig viele Rollen die er erfüllen kann:

```toml
[package]
id = "kanidm"
name = "Kanidm"

[roles]
# Hauptrollen
iam = true                    # Ist ein Identity & Access Management
oidc-provider = true          # Kann OIDC bereitstellen
scim-server = true            # Kann SCIM bereitstellen
mfa = true                    # Kann Multi-Faktor-Auth
webauthn = true               # Kann WebAuthn
radius = true                 # Kann RADIUS

# Rollen die es NICHT kann
ldap = false                  # Kein LDAP
saml = false                  # Kein SAML
```

**Warum ist das besser?**

1. Ein Service kann VIELE Rollen haben, nicht nur eine
2. Man kann nach Rollen suchen: "Zeig mir alle Services die `oidc-provider` können"
3. Rollen haben Ober- und Unter-Rollen: `database` ist die Oberrolle, `database.postgres` ist die Unterrolle
4. Bei der Variablen-Analyse ist die Differenzierung wichtig: Manchmal reicht "irgendeine DB", manchmal MUSS es Postgres sein

## Rollen-Hierarchie

```
database
├── database.postgres
├── database.mysql
├── database.mariadb
├── database.mongodb
├── database.sqlite
└── database.cockroachdb

cache
├── cache.redis
├── cache.dragonfly
├── cache.keydb
├── cache.valkey
└── cache.memcached

iam
├── iam.oidc-provider
├── iam.scim-server
├── iam.saml
├── iam.ldap
├── iam.mfa
├── iam.webauthn
└── iam.radius

smtp
├── smtp.sender
├── smtp.receiver
└── smtp.relay

wiki
├── wiki.create-page
├── wiki.search
├── wiki.api
└── wiki.export

git
├── git.hosting
├── git.api
├── git.ci-cd
└── git.mirror

chat
├── chat.rooms
├── chat.direct
├── chat.threads
├── chat.encryption
└── chat.bridges

map
├── map.points
├── map.layers
├── map.geojson
└── map.api

proxy
├── proxy.reverse
├── proxy.tls-termination
├── proxy.tcp-forward
└── proxy.load-balancer

monitoring
├── monitoring.logs
├── monitoring.metrics
├── monitoring.traces
└── monitoring.alerts

tasks
├── tasks.personal
├── tasks.projects
├── tasks.kanban
└── tasks.api

tickets
├── tickets.events
├── tickets.sales
└── tickets.api

collab
├── collab.realtime
├── collab.encrypted
├── collab.documents
└── collab.spreadsheets
```

## Wie Rollen in der Variablen-Analyse genutzt werden

Wenn der [Container App Manager](../programme/container_app/README.md) eine YAML analysiert und eine Umgebungsvariable `REDIS_URL` findet:

1. Erkennung: `*REDIS*` → Wahrscheinlich Rolle `cache`
2. Differenzierung: Ist es wirklich Redis, oder ein Redis-kompatibler Service (Dragonfly, KeyDB, Valkey)?
3. Unterrolle: `cache.redis` ODER `cache.dragonfly` — je nach Image-Name
4. Im Container App Manager kann der Benutzer dann wählen: "Welcher Cache-Service soll hier eingesetzt werden?" → Dropdown mit allen installierten Services die die Rolle `cache` haben

Dasselbe bei Datenbanken:
- `DATABASE_URL=postgres://...` → Rolle `database.postgres` (MUSS Postgres sein)
- `DB_HOST=...` → Rolle `database` (könnte jede DB sein)

---

## Rollen in der Suche

Im [Store](../programme/store/README.md) kann man nach Rollen suchen:

```
"Welche Services bieten oidc-provider?"
→ Kanidm, KeyCloak, Authentik, Rauthy

"Welche Services bieten database.postgres?"
→ PostgreSQL (offiziell), Supabase, CockroachDB (kompatibel)

"Welche Services bieten cache UND sind Redis-kompatibel?"
→ Redis, Dragonfly, KeyDB, Valkey
```

---

## Rollen in Lenses

[Lenses](../programme/lenses/README.md) nutzen Rollen um Daten zusammenzustellen:

```
Lens "Meine Gruppe":
  → Suche alle Services mit Rolle "wiki" → hole Artikel
  → Suche alle Services mit Rolle "map" → hole Kartenausschnitt
  → Suche alle Services mit Rolle "chat" → hole Nachrichten
  → Suche alle Services mit Rolle "tasks" → hole Aufgaben
```

---

Weiter: [Rechte-System](rechte.md) | [Container App Manager](../programme/container_app/README.md)
