# Foreman/Katello mit Ansible automatisieren

---

## `$ whoami`

Evgeni Golov

Software Engineer at Red Hat

ex-Consultant at Red Hat

Debian and Grml Developer

♥ FOSS ♥

♥ automation ♥

---

## Agenda

* Motivation / WTF
* Warum nicht `X`?!
* Foreman Ansible Modules
* Selber Module schreiben!

---

# Motivation / WTF

---

## Was ist Foreman?

* Tool zur Verwaltung von physikalischen und virtuellen Servern
* Power Management, Installation, Konfiguration
* Bare-Metal, VMware, RHV, OpenStack, GCE, Azure, etc
* Erweiterbar durch Plugins (zB Katello, Monitoring, Ansible)

---

## Was ist Katello?

* Plugin fuer Foreman
* Content Management (RPM, DEB, Puppet, Containers, Files)
* Content kann grupiert und gefiltert an Clients ausgeliefert werden
* Erlaubt Snapshots von Content zur Versionierung
* Patch Management

---

## Was ist Ansible?

* "radically simple IT automation engine"
* bringt eine enorme Zahl an Modulen fuer unterschiedliche Einsatzzwecke mit
* kann leicht durch eigene Module erweitert werden
* laesst sich gut mit REST APIs integrieren

---

## Warum automatisieren?

* Jeder kann mitmachen
* Peer Review der Änderungen
* Rollback bei Problemen
* Reproducibility
* Daten lesbar speichern und versionieren

Note:
PR ist einfacher als in der UI runklicken

---

## Wie automatisieren?

### Foreman hat eine WebUI!

* Kein Review der Änderungen möglich
* Zurückspringen zu älteren Einstellungen aufwändig
* Reproducibility ist eher nicht gegeben

---

## Wie automatisieren?

### Foreman hat eine CLI!

* Ich hab da mal schnell was mit `sed` und `awk` gebaut. Nein!
* Kann auch CSV/JSON Output und `jq`/`jo` sind toll, aber immer noch nein!
* Ansible `command`/`shell` Module <span class="emoji">🙄</span>

---

## Wie automatisieren?

### Foreman hat eine API!

* <span class="emoji">😻</span>
* Daten (Zustand) in einer Datenstruktur (YAML)
* Ein API Client übernimmt die Arbeit
* API Client selber schreiben? Später!
* Ansible!

---

# Warum nicht `X`?!

---

## `foreman` und `katello` Module

* Ansible Upstream seit 2.3 (2016)
* Deprecated seit 2.8
* Wird in 2.12 entfernt
* Ein Modul für alles, dadurch kompliziert zu bedienen
* Benutzt die `nailgun` Bibliothek

Note:
Nailgun ist eigentlich fuer Satellite geschrieben, versions spezifisch und hat Probleme mit non-Katello Installationen

---

```yaml
- name: Create CI Organization
  foreman:
    username: admin
    password: admin
    server_url: https://foreman.example.com
    entity: organization
    params:
      name: My Cool New Organization
```

---

```yaml
- name: Enable RHEL Product
  katello:
    username: admin
    password: admin
    server_url: https://katello.example.com
    entity: repository_set
    params:
      name: Red Hat Enterprise Linux 7 Server (RPMs)
      product: Red Hat Enterprise Linux Server
      organization: Default Organization
      basearch: x86_64
      releasever: 7Server
```

---

## `ansible-foreman-modules`

* Seit 2015 gibt es auch [`ansible-foreman-modules`](https://github.com/Nosmoht/ansible-module-foreman) von Thomas Krahn ([@Nosmoht](https://github.com/Nosmoht))
* Gut geplegt
* Benutzt eine eigene Bibliothek um mit der API zu sprechen
  * Benutzt nicht Foremans `apidoc.json`
  * Braucht Anpassungen fuer Plugins
* Aktuell kein Katello Support

---

# Foreman Ansible Modules

---

## Foreman Ansible Modules

* Seit Juni 2017
* Versucht `foreman`/`katello` aufzuraeumen
* Zunaechst durch Aufsplittung in einzelne Module pro Objekt
* Dann durch Aufbau eines Frameworks um Module schlank zu halten
* Tests!

---

## Foreman Ansible Modules

* Die Nutzung von `nailgun` wurde irgendwann anstrengend
  * Welche Version von `nailgun`?
  * Was ist mit Plugins?
* `apypie` Bibliothek als Ersatz fuer `nailgun`
  * Liest die `apidoc.json` von Foreman
  * alle Plugins!
* Durch existierendes Framework und Tests ist die Migration einfach

---

```yaml
- name: create example.org domain
  foreman_domain:
    name: "example.org"
    description: "Example Domain"
    server_url: "https://foreman.example.com"
    username: "admin"
    password: "secret"
    state: present
```

---

```yaml
- name: "Enable RHEL 7 RPMs repositories"
  katello_repository_set:
    username: "admin"
    password: "changeme"
    server_url: "https://foreman.example.com"
    name: "Red Hat Enterprise Linux 7 Server (RPMs)"
    organization: "Default Organization"
    product: "Red Hat Enterprise Linux Server"
    repositories:
    - releasever: "7Server"
      basearch: "x86_64"
    state: enabled
```

---

# Selber Module schreiben!

---

```python
from ansible.module_utils.foreman_helper import
  ForemanEntityApypieAnsibleModule

name_map = { 'name': 'name' }
module = ForemanEntityApypieAnsibleModule(
 argument_spec=dict(name=dict(required=True)))
entity_dict = module.clean_params()
module.connect()

entity = module.find_resource_by_name('architectures',
  name=entity_dict['name'], failsafe=True)
changed = module.ensure_resource_state('architectures',
  entity_dict, entity, module.state, name_map)
module.exit_json(changed=changed)
```

---

# Thanks!

<i class="fa fa-envelope" aria-hidden="true"></i> [evgeni@golov.de](mailto:evgeni@golov.de)

<i class="fa fa-globe" aria-hidden="true"></i> [die-welt.net](https://www.die-welt.net)

<i class="fa fa-twitter" aria-hidden="true"></i> [@zhenech](https://twitter.com/zhenech)

<i class="fa fa-mastodon" aria-hidden="true"></i> [@zhenech@chaos.social](https://chais.social/@zhenech)

<i class="fa fa-github" aria-hidden="true"></i> [@evgeni](https://github.com/evgeni)

<i class="fa fa-stack-exchange" aria-hidden="true"></i> [zhenech](https://stackexchange.com/users/1107433/zhenech)
