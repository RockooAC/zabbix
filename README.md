Dokumentacja wewnętrzna Zabbixa:
https://confluence.redge.com/display/DSW/Zabbix

użycie playbooka dla agentów:
ansible-playbook -i hosts install_zabbix_agent.yml

użycie playbooka dla proxy:
ansible-playbook -i hosts install_zabbix_proxy.yml

opcjonalnie, jeśli potrzebny dostęp do proxy:
-e proxy_auth={token}

