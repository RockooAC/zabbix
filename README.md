Dokumentacja wewnętrzna Zabbixa:
https://confluence.redge.com/display/DSW/Zabbix

## Co jest w tym repozytorium
- **Agent:** rola `zabbix_agent3` (playbook: `install_zabbix_agent.yml`) instalująca Zabbix Agent 2 w wersji 7.0 z PSK oraz metadanymi autorejestracji. Domyślnie ustawia `HostMetadata=groups=Linux passive`, więc każda nowa maszyna pojawi się w akcji autorejestracji jako członek tej grupy (możesz podmienić listę grup w zmiennej `zabbix_agent_autoreg_groups`).
- **Proxy:** rola `zabbix_proxy2` (playbook: `install_zabbix_proxy.yml`) instalująca Zabbix Proxy 7.0 na SQLite z PSK oraz unikalnym `TLSPSKIdentity` per host. Po uruchomieniu host musi istnieć na serwerze Zabbix (ręcznie lub przez API) z tą samą tożsamością i kluczem PSK z faktu `zabbix_proxy_tls_psk_value`.
- **Brak roli serwera:** repozytorium nie zawiera roli ani playbooka do instalacji/konfiguracji serwera Zabbix; serwer trzeba przygotować osobno.

użycie playbooka dla proxy:
ansible-playbook -i hosts install_zabbix_proxy.yml

opcjonalnie, jeśli potrzebny dostęp do proxy:
-e proxy_auth={token}

