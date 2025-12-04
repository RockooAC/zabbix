Dokumentacja wewnętrzna Zabbixa:
https://confluence.redge.com/display/DSW/Zabbix

## Co jest w tym repozytorium
- **Agent:** rola `zabbix_agent3` (playbook: `install_zabbix_agent.yml`) instalująca Zabbix Agent 2 w wersji 7.0 z PSK oraz metadanymi autorejestracji. Domyślnie ustawia `HostMetadata=groups=Linux passive`, więc każda nowa maszyna pojawi się w akcji autorejestracji jako członek tej grupy (możesz podmienić listę grup w zmiennej `zabbix_agent_autoreg_groups`).
- **Proxy:** rola `zabbix_proxy2` (playbook: `install_zabbix_proxy.yml`) instalująca Zabbix Proxy 7.0 na SQLite z PSK oraz unikalnym `TLSPSKIdentity` per host. Po uruchomieniu host musi istnieć na serwerze Zabbix (ręcznie lub przez API) z tą samą tożsamością i kluczem PSK z faktu `zabbix_proxy_tls_psk_value`.
- **Brak roli serwera:** repozytorium nie zawiera roli ani playbooka do instalacji/konfiguracji serwera Zabbix; serwer trzeba przygotować osobno.

## Autorejestracja nowych hostów
- **Agenty:** działają automatycznie po ustawieniu akcji autorejestracji na serwerze Zabbix, która filtruje `HostMetadata` zawierające `groups=` i przypisuje host do odpowiednich grup oraz szablonów. Domyślne PSK (`tLSPSKIdentity: zabbix-autoregistration`) jest generowane automatycznie na hoście.
- **Proxy:** Zabbix nie posiada mechanizmu autorejestracji proxy. Po zainstalowaniu nowego proxy playbookiem dodaj host na serwerze z tym samym `Hostname`, `TLSPSKIdentity` i PSK. PSK jest zapisywane w `/etc/zabbix/zabbix-proxy.key` oraz dostępne jako fakt `zabbix_proxy_tls_psk_value` do wykorzystania w API/GUI.

## Użycie playbooków
- Agenty: `ansible-playbook -i hosts install_zabbix_agent.yml`
- Proxy: `ansible-playbook -i hosts install_zabbix_proxy.yml`
  - opcjonalnie: `-e proxy_auth={token}` jeśli wymagane jest wyjście przez proxy HTTP/HTTPS

