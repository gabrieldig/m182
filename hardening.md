# Hardening

Offene Ports, Dienste und Benutzerkonten:

### Ports
| Port | Service | Status |
| :--- | :------ | :----- |
| 22   | SSh     | Offen  |

### Dienste
| Dienst    | Status |
| :-------- | :----- |
| /bin/bash | offen  |
| sshd      | offen  |
| ps        | aux    |

### Benutzerkonten
| Username         | Password | UID   | GID   | Comment/GECOS                      | Home Directory  | Login Shell       |
| :--------------- | :------- | :---- | :---- | :--------------------------------- | --------------- | ----------------- |
| root             | x        | 0     | 0     | root                               | /root           | /bin/bash         |
| daemon           | x        | 1     | 1     | daemon                             | /usr/sbin       | /usr/sbin/nologin |
| bin              | x        | 2     | 2     | bin                                | /bin            | /usr/sbin/nologin |
| sys              | x        | 3     | 3     | sys                                | /dev            | /usr/sbin/nologin |
| sync             | x        | 4     | 65534 | sync                               | /bin            | /bin/sync         |
| games            | x        | 5     | 60    | games                              | /usr/games      | /usr/sbin/nologin |
| man              | x        | 6     | 12    | man                                | /var/cache/man  | /usr/sbin/nologin |
| lp               | x        | 7     | 7     | lp                                 | /var/spool/lpd  | /usr/sbin/nologin |
| mail             | x        | 8     | 8     | mail                               | /var/mail       | /usr/sbin/nologin |
| news             | x        | 9     | 9     | news                               | /var/spool/news | /usr/sbin/nologin |
| uucp             | x        | 10    | 10    | uucp                               | /var/spool/uucp | /usr/sbin/nologin |
| proxy            | x        | 13    | 13    | proxy                              | /bin            | /usr/sbin/nologin |
| www-data         | x        | 33    | 33    | www-data                           | /var/www        | /usr/sbin/nologin |
| backup           | x        | 34    | 34    | backup                             | /var/backups    | /usr/sbin/nologin |
| list             | x        | 38    | 38    | Mailing List Manager               | /var/list       | /usr/sbin/nologin |
| irc              | x        | 39    | 39    | ircd                               | /run/ircd       | /usr/sbin/nologin |
| gnats            | x        | 41    | 41    | Gnats Bug-Reporting System (admin) | /var/lib/gnats  | /usr/sbin/nologin |
| nobody           | x        | 65534 | 65534 | nobody                             | /nonexistent    | /usr/sbin/nologin |
| _apt             | x        | 100   | 65534 |                                    | /nonexistent    | /usr/sbin/nologin |
| systemd-network  | x        | 101   | 102   | systemd Network Management,,,      | /run/systemd    | /usr/sbin/nologin |
| systemd-resolve  | x        | 102   | 103   | systemd Resolver,,,                | /run/systemd    | /usr/sbin/nologin |
| messagebus       | x        | 103   | 104   |                                    | /nonexistent    | /usr/sbin/nologin |
| systemd-timesync | x        | 104   | 105   | systemd Time Synchronization,,,    | /run/systemd    | /usr/sbin/nologin |
| sshd             | x        | 105   | 65534 |                                    | /run/sshd       | /usr/sbin/nologin |

## Härtung

| Massnahme                                  | Grund                                                                                                          |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| 1. SSH-Root-Login deaktivieren             | Der root benutzer gibt es überall und hat rechte auf alles, deswegen ist es good-practice ihn zu deaktivieren. |
| 2. Starkes Passwort setzen                 | das passwort wird oft default gelassen, was sicherheit beeinträchtigt                                          |
| 3. Neuen Benutzer anlegen                  | Man sollte möglichst immer nach dem "least privilege" Prinzip vorgehen.                                        |
| 4. Nur SSH starten, keine weiteren Dienste | Führt dazu, dass weniger Angriffsflächen existieren                                                            |
| 5. UFW installieren und aktivieren         | Ist eine einfache Firewall, die zur Steuerung des Netzwerkverkehrs verwendet werden kann.                      |
| 6. TCP- und UDP-Ports prüfen               | Dient zur Überprüfung der offenen Ports und kann helfen, potenzielle Sicherheitslücken zu identifizieren.      |
| 7. Deaktivieren von IPv6 (optional)        | Wenn ungenutzt, kann IPv6 deaktiviert werden, um die Sicherheit zu erhöhen.                                    |
| 8. Unnötige Software entfernen             | Man sollte unnötige Software entfernen, um das System sicherer zu machen.                                      |

## Speichern
Wenn man ein Docker Container so konfiguriert hat, dass er den Sicherheitsanforderungen entspricht, sollte man ein Image daraus erstellen, um die Konfiguration zu speichern.
Z.B. ``docker commit hardeninglab hardend-ubuntu``.

Und dannach kann man ihn wieder mit ``docker run -it hardend-ubuntu bash`` starten.


# Reflektion

- Dienste

Ich finde das disablen des root Logins eine der wichtigsten Massnahmen ist, da der root Benutzer überall existiert und somit ein beliebtes Ziel für Angriffe ist. Falls ein root Benutzer notwenig ist, sollte man am besten ein neuen Benutzer anlegen und diesem die nötigen Rechte geben.

Ein gehärtetes Image ist ein guter Ausgangspunkt, um ein sicheres System zu erstellen. Im Vergleich zu einem Vanilla Image, kann man sie direkt ohne weitere Konfiguration starten und hat somit eine höhere Sicherheit.