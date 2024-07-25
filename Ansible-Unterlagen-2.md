# Variablen

* Möglichkeit, dynamische Werte zu speichern, um diese wiederzuverwenden
* Variablennamen beginnen immer mit einem Buchstaben und enthalten nur Buchstaben, Zahlen und Unterstriche
* Die Verwendung der Variable erfolgt über `{{ variable_name }}`
* Beginnt der Wert eines Schlüssel-Wert Paars mit einer Variable, so müssen Anführungszeichen `"` verwendet werden!

## Definition von Variablen
Es gibt verschiedene Orte, an denen Variablen je nach Geltungsbereich definiert werden können. Diese sind in aufsteigender Priorität:

* Variablen können im Inventory als Gruppen- oder Hostvariablen hinterlegt werden
* Es besteht die Möglichkeit Gruppen- bzw. Hostvariablen in eine Datei auszulagern (Dateiname äquivalent zu den Hosts/Hostgruppen im Verzeichnis `group_vars` oder `host_vars` unter dem Ablageort des Inventory)
* Im Playbook können Variablen auf Play-Ebene im `vars`-Block definiert sein oder in einer Datei `vars_files` ausgelagert werden
* Im Playbook können Task-spezifische Variaben definiert werden
* Variablen können über die Befehlszeile übergeben werden

s. auch [Ansible Dokumentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)

## Variaben in Playbooks
```yaml
vars:
  user: guethb
tasks:
  - name: Create user {{ user }}
    ansible.builtin.user:
      name: "{{ user }}"
```
Achtung: Wenn eine Variable als erstes Element in einem Schlüssel-Wert-Paar verwendet wird, muss sie in Anführungszeichen gesetzt werden.

[Lab 3](https://github.com/guethb/ansible-labs/tree/main/lab-3)

## Variablen in Inventories

* Es besteht die Möglichkeit, die Variablen direkt in die Inventories zu schreiben (unüblich, veraltet)
* Gruppenvariablen werden in einer Datei deren Dateiname äquivalent zu der betroffenen Gruppe ist im Verzeichnis `group_vars` unter dem Ablageort des Inventory oder des Playbook als Schlüssel-Wert-Paar gehalten
* Hostvariablen werden in einer Datei deren Dateiname äquivalent zum Hosteintrag ist im Verzeichnis `host_vars` unter dem Ablageort des Inventory oder des Playbook als Schlüssel-Wert-Paar gehalten

[Lab 4](https://github.com/guethb/ansible-labs/tree/main/lab-4)

## Register

* Das Ergebnis der Ausführung eines Moduls kann in einer Variable gespeichert werden
* Dafür wird auf Ebene des Tasks das Schlüsselwort `register` verwendet und erhält als Wert den Namen der Variable, in der das Ergebnis gespeichert wird

[Lab 5](https://github.com/guethb/ansible-labs/tree/main/lab-5)

# Facts
* Fakten sind spezielle Variablen, die während der Ausführung eines Playbook automatisch auf den Managed Hosts ermittelt werden
* Es handelt sich um systemspezifische Kennwerte, wie Hostnamen, Kernelversion, Anzahl der CPUs, verfügbarer oder freier Arbeitsspeicher, etc.
* Die Facts sind im Dictionary `ansible_facts` gespeichert
* Der Wert zum Schlüssel `key` aus einem Dictionary `dict` kann entweder mit `dict['key']` oder `dict.key` ausgelesen werden
* In älteren Ansible-Versionen wurden die Fakten in Variablen injiziert und hießen z.B. `ansible_hostname` oder `ansible_dns['nameservers]`

## Übung 3: Facts

1. Lesen Sie die Dokumentation des Moduls [ansible.builtin.debug](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html#ansible-collections-ansible-builtin-debug-module) 

2. Schreiben Sie ein Playbook, welches das Dictionary `ansible_facts` ausgibt

3. Schauen Sie sich die zurückgegebenen Werte an

4. Diskutieren Sie mögliche Einsatz-Szenarien.

## Übung 4: Konfiguration

1. Wo können Sie die Ansible Konfiguration ablegen?
2. Erzeugen Sie eine Ansible Konfiguration in Ihrem Home-Verzeichnis, die auf die Inventory-Datei mit dem Namen `~/inventory` verweist.
3. Erzeugen Sie unter `~/inventory` ein Inventory mit folgendem Inhalt:

```
host0.lab.drv.internal
host1.lab.drv.internal
host2.lab.drv.internal
host3.lab.drv.internal
```

4. Welche Dateien aus Übung 3 müssen Sie löschen, um die neue Konfiguration zu verwenden? Welchen Vorteil sehen Sie in dem Vorgehen? Tun Sie dies!
5. Führen Sie Übung 3 erneut aus.

# Schleifen

* Eine Schleife iteriert über eine Liste und führt einen Task für jedes Listenelement aus
* Das aktuelle Listenelement findet sich in der intrinsisch definierten Task-Variable `item`
* Listenelemente können aus Objekten bestehen
* Als Listenelemente kann auch eine Variable, Gruppenvariable oder Hostvariable 
* Um eine Schleife für einen Task zu erzeugen, wird dass Schlüsselwort `loop` verwendet und diesem die Werte als Liste übergeben

Beispiel für eine einfache Liste:
```yaml
tasks:
  - name: List Demo
    ansible.builtin.debug:
      msg: {{ item }}
    loop:
      - value1
      - value2
```

[Lab 6](https://github.com/guethb/ansible-labs/tree/main/lab-6)

## Übung 5: Schleifen
Verwenden Sie die Dateien aus [Lab 6](https://github.com/guethb/ansible-labs/tree/main/lab-6). Unter der Verwendung von Host-Variablen, erzeugen Sie folgendes Szenario:

1. Auf `host0` gibt es einen Benutzer mit der Kennung `mueller` in der Gruppe `wheel`
1. Auf `host0` gibt es einen Benutzer mit der Kennung `mayer` in der Gruppe `root`
1. Auf `host1` gibt es die Benutzer mit den Kennungen `fischer` und `schmidt`, jeweils in der Gruppe `root`

# Bedingte Ausführung

* Die Ausführung eines Tasks kann man an eine Bedingung knüpfen
* Eine Bedingung ist ein Ausdruck, der sich zu `true` oder `false` evaluieren lässt
* Unterschied zwischen `false` und `"false"` lässt sich durch einen Typecast lösen (`variable | bool`)
* Der Ausdruck wird im Schlüssel `when` des Tasks aufgeführt
* Ausdrücke können mit `and` und `or` verknüpft werden
* Wird eine Liste von Ausdrücken übergeben, ist die Verknüpfung implizit eine Und-Verknüpfung
* Es lassen sich Schleifen und Bedingungen verknüpfen

|Syntax|Beschreibung|
|---|---|
|gleich (string)|`string == "hello"`|
|gleich (numerisch)|`number == 123`|
|kleiner als|`number < 10`|
|größer als|`number > 10`|
|kleiner oder gleich|`number <= 10`|
|größer oder gleich|`number >= 10`|
|ungleich|`number != 10`|
|Variable existiert|`var is defined`|
|Variable existiert nicht|`var is not defined`|
|Boolsche Variable ist `true`|`var`|
|Boolsche Variable ist `false`|`not var`|
|Wert in Liste enhalten|`var in list`|

Ein Beispiel für eine bedingte Ausführung
```yaml
tasks:
  - name: Execute if value is greater than 3
    ansible.builtin.debug:
      msg: You see, because Value {{ value }} is greater than 3
    when: "{{value}}" > 3 
```

Was ist die Ausgabe dieses Tasks?
```yaml
tasks:
  - name: print out numbers
    ansible.builtin.debug:
      var: item
    loop: [ 0, 2, 4, 6, 8, 10 ]
    when: item  > 5
```
[Lab 7](https://github.com/guethb/ansible-labs/tree/main/lab-7)


## Übung 6: Bedingungen
Ein IT Betrieb legt folgende Regeln für die Größe des Swap-Speichers fest:

* Verfügt der Server über mehr als 16GB RAM, soll die Swap-Partition 16GB groß sein
* Verfügt der Server über bis zu 16GB RAM, soll die Swap-Partition genauso groß sein, wie der Arbeitsspeicher des Servers

Erstellen Sie ein Playbook, welches für jeden Host die Soll-Größe des Swap-Speichers angibt.

## Blöcke
* Blöcke gruppieren mehrere Tasks
* Ein Block wird wie ein Task behandelt
* Ein Block kann eigene `when` oder `loop` Schlüsselwörter enthalten, die sich auf den gesamten Block anwenden
* Es wird ein eigener Name für den Block vergeben
* Die Gruppierung erfolgt über das `block` Schlüsselwort

```yaml
tasks:
  - name: Block of multiple tasks
    block:
    - name: Task 1 of block
      ansible.builtin.shell:
        ...
    - name: Task 2 of block
      ansible.builtin.shell:
        ...
      loop:
        - loop_once
        - loop_twice 
    when: ansible_facts['distribution'] == "RedHat"
```

# Handler

* Handler eignen sich dazu, Abhängigkeiten zwischen Aufgaben zu definieren
* Handler werden nur ausgeführt, wenn eine Änderung durch die auslösende Aufgabe stattgefunden hat
* Häufiger Anwendungsfall sind der Restart von Services nach Installation von Paketen oder Konfigurationsänderungen
* Handler regeagieren auf Benachrichtigungen (engl.: Notifications)
* Handler werden am Ende eines Plays aufgerufen und werden nur einmalig ausgeführt
* Handler werden immer in der Reihenfolge ausgeführt, in der sie definiert sind
* Um mit einem Task einen Handler zu benachrichtigen, wird das Schlüsselwort `notify` verwendet und als Wert der Name des Handlers übergeben
* Handler werden in einem eigenen Obekt unterhalb des Plays verwendet für den sie definiert sind

```yaml
tasks:
  - name: do something
    notify: restart httpd
    ...
handlers:
  - name: restart httpd
    ansible.builtin.service:
      name: httpd
      state: restarted
```
## Übung 7: Handler
1. Schreiben Sie ein Playbook, welches auf `host0` die Pakete `mysql-server` und `httpd` installiert
2. Schreiben Sie einen Handler, der bei erfolgreicher Installation von `httpd` den Service `http` an der Firewall freischaltet
3. Schreiben Sie einen Handler, der den `httpd` Service bei erfolgreicher Installation rebootfest startet
4. Schreiben Sie einen Handler, der bei erfolgreicher Installation von `mysql-server` den Service `mysql` an der Firewall freischaltet
5. Schreiben Sie einen Handler, der den `mysqld` Service bei erfolgreicher Installation rebootfest startet

# Fehlerbehandlung

* Ansible bricht die Ausführung des Playbooks bei fehlgeschlagenen Tasks ab
* Das Verhalten kann mit dem Schlüssel `ignore_errors` (bool) geändert werden
* Für einen Block kann mit dem Schlüsselwort `rescue` ein Rescue-Block definiert werden, der bei Fehlschlagen eines Tasks ausgeführt wird
* Ebenso kann mit den Schlüsselwort `always` ein Block vereinbart werden, der immer ausgeführt wird, auch wenn ein Task fehlschlägt

```yaml
tasks:
  -name: Name of block
  block:
    - name: Task to upgrade database
      ...
  rescue:
    - name: rollback upgrade
      ...
  always:
    - name: restart database
```

## Fehler zurückmelden
* Es ist möglich, für einen Task Bedingungen zu definieren, die dazu führen, dass der Task als fehlgeschlagen behandelt wird
* Das Schlüsselwort hierfür ist `failed_when` und enthält die Bedingung in der gleichen Syntax eines `when` Blocks


## Übung 8: Fehlerbehandlung
1. Sehen Sie sich die Dokumentation der Module `ansible.builtin.get_url`, `ansible.builtin.command` und `ansible.builtin.stat` an
1. Sehen Sie sich die die [Ansible Dokumentation zur Variablenübergabe über die CLI](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#defining-variables-at-runtime) an. 
1. Führen Sie das gesamte Play als Benutzer `user` aus, verwenden Sie keine Priviledge Escalation
1. Brechen Sie den Play ab, wenn auf dem Host bereits eine Datei `/home/user/inventory` existiert
1. Ansonsten laden Sie die Datei `https://raw.githubusercontent.com/guethb/ansible-labs/main/lab-1/inventory` herunter
1. Führen Sie das Playbook mehrfach unter der Verwendung der URL https://raw.githubusercontent.com/guethb/ansible-labs/main/lab-1/inventory aus und Diskutieren Sie Ihre Beobachtung
1. Erstellen Sie einen Rescue-Block mit `ansible.builtin.command`, der die Datei umkopiert und einen Zeitstempel anhängt
1. Laden Sie in jedem Fall die Datei herunter
1. Diskutieren Sie: ist das Playbook idempotent?

## Handler und Fehler
* Normalerweise werden nach Abbrüchen auch keine Handler ausgeführt, wenn diese vorher benachrichtigt wurden
* Im Play kann mit den Schlüssel `force_handlers` die Ausführung der benachrichtigten Handler erzwungen werden
* Handler von fehlgeschlagenen Aufgaben werden nicht Benachrichtigt und deshalb auch mit `force_handlers` nicht ausgeführt

>**_Hinweis:_** Für das Debugging kann es sinnvoll sein, das Log-Level mit dem Schalter `-v` auf bis zu vier Stufen (`-vvvv`) zu erhöhen.

# Tags
[Ansible Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_tags.html)

# Jinja 2 Templates
* Eine Template Engine kann eine Schablone verwenden und diese dynamisch mit Inhalten befüllen
* Ansible verwendet Jinja2 Templates

Trotzdem gibt es noch andere Module, die gegebenenfalls einfacher zu verwenden sind:

|Modul|Beschreibung|
|---|---|
|`blockinfile`|Fügt einen Textblock ein, der von anpassbaren Markierungslinien umgeben ist, aktualiseren oder entfernen Sie ihn|
|`copy`|Kopiert eine Datei von einem Remote Host auf den Managed Host|
|`fetch`|Holt Dateien vom Managed Host auf den Control Node|
|`file`|Legt Attribute für eine Datei fest, wie z.B. Berechtigungen, SELinux-Kontexte, Zeitstempel oder erstellt Dateien und Verzeichnisse mit diesen Attributen|
|`lineinfile`|Stellt sicher, dass eine bestimmte Zeile in einer Datei enthalten ist oder ersetzt vorhandene Zeilen|
|`stat`|Ruft Statusinformationen zu Dateien ab|
|`patch`|Wendet Patches auf Dateien an, ähnlich GNU `patch`|

## Übung 9: Einfache Textoperation
1. Stellen Sie sicher, dass die Passwortauthentifizierung für SSH-Zugänge auf allen Hosts gesperrt ist

## Grundlage zu Templates
* Grundlage ist eine Textdatei, die als Schablone dient
* Übliche Dateiendung `.j2`, üblicherweise im Unterverzeichnis `templates`
* Variablen werden in der Form `{{ variable }}` im Template eingefügt 
* Das Modul `ansible.builtin.template` rollt ein Template auf den Hosts aus
* Es ist guter Stil, Konfigurationsdateien mit einem Hinweiskommentar zu beginnen, dass diese Datei von Ansible gemanagt wird
* Der Hinweis kann über die Variable {{ ansible_managed }} erzeugt werden, deren Inhalt in der `ansible.cfg` festgelegt wird (s. [Ansible Configuration Settings](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings))
* Kommentare, die nicht im Ziel enthalten sein sollen können mit {# comment #} erzeugt werden
* Kontrollstrukturen werden mit {% EXPRESSION %} erzeugt
* Es existiert eine ausführliche [Dokumentation zu Jinja2](https://jinja.palletsprojects.com/en/3.1.x/templates/)

## Übung 10: Templating Grundlagen
1. Erstellen Sie ein Template für einen Bericht über Hosts, der den Hostnamen, Zeitpunkt der Bericherstellung, installiertem Betriebssystem, Version des Betriebssystems. Anzahl der Prozessoren und Prozessorkerne und vorhandenem Speicher enthält
1. Verwenden Sie `ansible.builtin.template` um den Bericht auf dem Host zu erstellen
1. Holen Sie den fertigen Bericht vom Host ab

## Loops
* Schleifen iterieren über eine Liste

```
{% for item in list %}
{{ item }}
{% endfor %}
```

## Bedingungen
```
{% if boolean }
{{ result }}
{% endif %}
```

## Filter
* Filter können eine Variable oder das Ergebnis eines Ausdrucks umformatieren
* Die Schreibweise ist `{{ expr | FILTER }}`
* Mit dem `default(VALUE)` Filter, kann ein Default-Wert gesetzt werden, falls die Variable nicht exisitiert
* Mit `to_json` und `to_yaml` erfolgt die Ausgabe in JSON oder yaml
* Gut lesbare Ergebnisse erzeugt man mit `to_nice_json` bzw. `to_nice_yaml`
* Die Jinja2 Dokumentation enthält eine [vollständige Liste der verfügbaren Filter](https://jinja.palletsprojects.com/en/3.1.x/templates/#list-of-builtin-filters)

## Übung 11: Loops & Filter
Erweitern Sie den Bericht aus Übung 10 um Netzwerkinterfaces mit Informationen zu vorhandenen Netzwerkinterfaces, ihren MAC- und IPv4-Adressen und der Netzwerkmaske.

# Rollen

* Möglichkeit, Ansible-Code wiederzuverwenden
* Standardisierung von Systemen
* Einheitliche Verzeichnisstruktur
* Ablageort in `ansible.cfg` durch das Schlüsselwort `role_path` wählbar (s. [Ansible Referenz Konfiguration](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-roles-path))
* Es gibt drei Möglichkeiten, eine Rolle in einem Playbook auszuführen (s. [Ansible Playbook Leitfaden](https://docs.ansible.com/ansible/devel/playbook_guide/playbooks_reuse.html#dynamic-vs-static)):
    * Mit dem `roles` Schlüsselwort auf Ebene des Play wird die Rolle statisch importiert
    * Mit `ansible.builtin.import_role` kann eine Rolle statisch importiert werden
    * Mit `ansible.builtin.include_role` kann eine Rolle dynamisch importiert werden
* Es ist möglich, mehrere Rollen zu importieren
* Mit den Schlüsselworten `pre_tasks` und `post_tasks` können auf Play-Ebene Tasks definieren
* Es ist _nicht_ empfohlen

Die Verzeichnisstruktur einer Rolle:

|Unterverzeichnis|Funktion|
|---|---|
|`defaults`|Die Datei `main.yml` in diesem Verzeichnis enthält die Standardwerte der Rollenvariablen, die bei Verwendung überschrieben werden können|
|`files`|Dieses Verzeichnis enthält statische Dateien, auf die Rollenaufgaben verweisen|
|`handlers`|Die Datei `main.yml` in diesem Verzeichnis enthält die Handler-Definitionen der Rolle|
|`meta`|Die Datei `main.yml` in diesem Verzeichnis enthält Informationen über die Rolle, inklusive Autor, Lizenz, Plattformen und optionale Rollenabhängigkeiten|
|`tasks`|Die Datei `main.yml` in diesem Verzeichnis enthält die Aufgabendefinitionen der Rolle|
|`templates`|Dieses Verzeichnis enthält Jinja2-Vorlagen, auf die die Rollenaufgaben verweisen|
|`tests`|Dieses Verzeichnis kann ein Inventar und Playbook test.yml enthalten, die für die Prüfung der Rolle verwendet werden können.
|`vars`|Die `mail.yml` in diesem Verzeichnis definiert die Variablenwerte für interne Zwecke der Rolle|

[Lab 8](https://github.com/guethb/ansible-labs/tree/main/lab-8)

## Pre-Tasks und Post-Tasks

* Mit dem Schlüsselwort `pre_tasks` ist es auf Play-Ebene möglich, Tasks vor der Ausführung der Rolle auszuführen
* Analog kann `post_tasks` verwendet werden, um Tasks auszuführen, die nach der Rolle ausgeführt werden
* Es ist möglich, aber nicht empfohlen, neben den Rollen weitere (Standard-)Tasks zu definieren
* Handler, die von Pre-Tasks benachrichtigt werden, laufen vor der Ausführung der Rollen


Ein vollständiges Gerüst eines Plays mit Rollen:
```yaml
- name: Play to illustrate order of execution
  hosts: remote.example.com
  pre_tasks:
    - name: run first
      notify: play handler
      changed_when: true
      ...
  roles:
    - role: role1
  tasks:
    - name: runs after the roles
      notify: play handler
      changed_when: true
      ...
  post_tasks:
    - name: run last
      notify: play handler
      changed_when: true
      ...
  handlers:
    - name: play handler
      ...
```

>**_Achtung:_** Ein Handler kann drei mal ausgeführt werden: nach der Ausführung der Pre-Tasks, nach der Ausführung der Rollen und nach der Ausführung von Post-Tasks.

## Rollen erstellen

Initialiseren der Rolle mit

    ansible-galaxy init role role_name

Hinweise zum Erstellen von Rollen:

* Pro Rolle ein Repository
* Konfiguration der Rolle über Variablen
* Keine vertraulichen Informationen in der Rolle, wie Passwörter, Keys, Zertifikate, ...
* Verzeichnisse, die nicht benötigt werden, entfernen
* Dokumentation in der README.md
* Eine Rolle pro Zweck
* Keine Rollen für Sonderkonfiguration, sondern integration des Falls in bestehende Rollen

s. a. [Red Hat Good Practices for Ansible](https://redhat-cop.github.io/automation-good-practices/#_roles_good_practices_for_ansible)

## Übung 12: Rolle erstellen
1. Kopieren Sie die Rolle `motd` in das richtige Verzeichnis Ihres Projektes
1. Erstellen Sie eine neue Rolle mit Namen `vhost`
1. Löschen Sie die Unterverzeichnisse `test` und `vars` der `vhost` Rolle
1. Erstellen Sie eine Rollenvariable `webmaster` mit dem Standardwert `webmaster@example.com`
1. Erstellen Sie einen Handler für die, der den httpd-Service restartet.
1. Erstellen Sie folgende Rollen-Tasks in der `vhost` Rolle, die folgendes sicherstellen:
    * das Paket httpd ist in der neuesten Version installiert
    * Der httpd-Service ist gestartet und rebootfest
    * das `vhost.conf.j2` wird verwendet, um eine Konfiguration des virtuellen Hosts in `/etc/httpd/conf.d/vhost.conf` zu erzeugen
    * die Firewall ist für den Service http geöffnet
1. Stellen Sie sicher, dass das Ausrollen des Templates einen Restart des httpd-Service erzwingt
1. Die Datei `vhost.conf` in Ihrem Projektverzeichnis zeigt eine Webserver-Konfiguration für `host0` - erstellen Sie ein darauf basierendes Jinja2 Template
1. Schreiben Sie ein Playbook das
    * vor der Abarbeitung von Rollen die Debug-Nachricht "ensure configured web server" ausgibt 
    * die Rollen `vhost` und `motd` auf `host3` anwendet 
    * sicherstellt, dass sowohl die `motd`- als auch die `vhost`-Rolle die gleiche Email-Adresse <webmaster@lab.drv.internal> verwenden
    * nachdem die Rollen angewendet sind sicherstellt, dass sich die Datei `files/index.html` in dem Verzeichnis befindet, welches im Directory-Eintrag der vhost.conf referenziert wird (Achtung: Verzeichnis exisitert bis dahin nicht) und dem Benutzer und der Gruppe `apache` gehört, sowie die Rechte 0644 besitzt 
1. Führen Sie das Playbook aus und testen Sie die Erreichbarkeit des Webservers
