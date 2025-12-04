Dokumentacja wewnętrzna Zabbixa:
https://confluence.redge.com/display/DSW/Zabbix

## Co jest w tym repozytorium
- **Agent:** rola `zabbix_agent3` (playbook: `install_zabbix_agent.yml`) instalująca Zabbix Agent 2 w wersji 7.0 z PSK oraz metadanymi autorejestracji. Domyślnie ustawia `HostMetadata=groups=Linux passive`, więc każda nowa maszyna pojawi się w akcji autorejestracji jako członek tej grupy (możesz podmienić listę grup w zmiennej `zabbix_agent_autoreg_groups`). Rola jest przeznaczona dla zwykłych hostów.
- **Proxy:** rola `zabbix_proxy2` (playbook: `install_zabbix_proxy.yml`) instalująca Zabbix Proxy 7.0 na SQLite z PSK oraz unikalnym `TLSPSKIdentity` per host. W ramach tej samej roli instalowany i konfigurowany jest także Zabbix Agent 2 (domyślnie z metadanymi `groups=Linux servers,Zabbix proxies`). Po uruchomieniu host musi istnieć na serwerze Zabbix (ręcznie lub przez API) z tą samą tożsamością i kluczem PSK z faktu `zabbix_proxy_tls_psk_value`.
- **Brak roli serwera:** repozytorium nie zawiera roli ani playbooka do instalacji/konfiguracji serwera Zabbix; serwer trzeba przygotować osobno.

użycie playbooka dla proxy:
ansible-playbook -i hosts install_zabbix_proxy.yml

## Autorejestracja nowych hostów
- **Agenty:** akcja autorejestracji jest tworzona przez rolę `zabbix_server` i filtruje `HostMetadata` zawierające `groups=`. Domyślne PSK (`tLSPSKIdentity: zabbix-autoregistration`) jest generowane automatycznie na hoście, a grupy i szablony są narzucane przez serwerową akcję.
- **Proxy:** rola `zabbix_server` potrafi zarejestrować proxy przez API, pobierając PSK z faktu `zabbix_proxy_tls_psk_value` (wygenerowanego na proxy). Jeśli nie chcesz automatu, możesz zarejestrować proxy ręcznie/API z tym samym `Hostname`, `TLSPSKIdentity` i PSK zapisanym w `/etc/zabbix/zabbix-proxy.key`.

## Użycie playbooków
- Serwer (konfiguracja API, grupy, akcja autorejestracji, rejestracja proxy): `ansible-playbook -i hosts install_zabbix_server.yml -e zabbix_server_url=https://twoj.zabbix/ui -e zabbix_server_login_user=Admin -e zabbix_server_login_password=haslo`
  - Zmienna `zabbix_server_proxy_psk_variable` wskazuje, z którego faktu/zmiennej w inwentarzu pobierać PSK wygenerowane przez role proxy (`zabbix_proxy_tls_psk_value` domyślnie).
- Wymagana kolekcja `community.zabbix` na nodzie kontrolnym Ansible.
- Agenty: `ansible-playbook -i hosts install_zabbix_agent.yml`
- Proxy: `ansible-playbook -i hosts install_zabbix_proxy.yml`
  - opcjonalnie: `-e proxy_auth={token}` jeśli wymagane jest wyjście przez proxy HTTP/HTTPS

