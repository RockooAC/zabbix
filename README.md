Dokumentacja wewnętrzna Zabbixa:
https://confluence.redge.com/display/DSW/Zabbix

## Co jest w tym repozytorium
- **Server:** rola `zabbix_server` (playbook: `install_zabbix_server.yml`) zarządzająca konfiguracją serwera Zabbix poprzez API: tworzy grupy hostów, akcję autorejestracji dla agentów (w tym opcjonalne linkowanie szablonów) oraz rejestruje proxy z PSK pobranym z inwentarza.
- **Agent:** rola `zabbix_agent` (playbook: `install_zabbix_agent.yml`) instalująca Zabbix Agent 2 w wersji 7.0 z PSK oraz metadanymi autorejestracji. Domyślnie ustawia `HostMetadata=groups=Linux passive`, więc każda nowa maszyna pojawi się w akcji autorejestracji jako członek tej grupy (możesz podmienić listę grup w zmiennej `zabbix_agent_autoreg_groups`).
- **Proxy:** rola `zabbix_proxy` (playbook: `install_zabbix_proxy.yml`) instalująca Zabbix Proxy 7.0 na SQLite z PSK oraz unikalnym `TLSPSKIdentity` per host. Po uruchomieniu host jest automatycznie rejestrowany przez serwerową rolę (lub możesz zrobić to ręcznie/API) z tą samą tożsamością i kluczem PSK z faktu `zabbix_proxy_tls_psk_value`.

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

## Jak dodać nowego hosta
### Nowy Zabbix Proxy (pełna automatyzacja z PSK i rejestracją)
1. **Dodaj host do inwentarza** w grupie `zabbix_proxies` (np. `nowy-proxy ansible_host=10.0.0.10`).
2. Opcjonalnie zdefiniuj hostname i serwer Zabbixa dla proxy: w `host_vars/nowy-proxy.yml` ustaw `zabbix_proxy_hostname`, `zabbix_proxy_server`, ewentualnie inne parametry z `roles/zabbix_proxy/defaults/main.yml`.
3. **Zainstaluj proxy**: `ansible-playbook -i hosts install_zabbix_proxy.yml -l nowy-proxy`.
   - Rola wygeneruje PSK (`/etc/zabbix/zabbix-proxy.key`) i zapisze fakt `zabbix_proxy_tls_psk_value` jako cache’owalny.
4. **Zarządzaj serwerem**: `ansible-playbook -i hosts install_zabbix_server.yml -l zabbix_server -e zabbix_server_url=https://twoj.zabbix/ui -e zabbix_server_login_user=Admin -e zabbix_server_login_password=haslo`.
   - Rola odczyta PSK z hostvars (wymaga włączonego cache faktów lub statycznej zmiennej w inwentarzu `zabbix_proxy_tls_psk_value`) i zarejestruje proxy z odpowiednią tożsamością.

### Nowy Zabbix Agent
1. Dodaj host do grupy objętej playbookiem agentów (np. `zabbix_agents_hosts`).
2. (Opcjonalnie) Zmień grupy autorejestracji: `zabbix_agent_autoreg_groups` w `host_vars/<host>.yml` lub na poziomie grupy.
3. Uruchom: `ansible-playbook -i hosts install_zabbix_agent.yml -l <host>`.
4. Agenta przechwyci akcja autorejestracji utworzona przez rolę serwerową – host trafi do grup z `HostMetadata` (`groups=...`) i (jeśli wskazane) zostaną podpięte szablony.

