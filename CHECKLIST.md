# CHECKLIST RÁPIDO - LojaZeta (80/20)

## Instalação / Quick Wins (30 dias)
- [ ] Instalar agente de logs (Filebeat / Wazuh) nas VMs (web, app, db).
  - Filebeat example: `filebeat modules enable nginx` + configurar outputs.
- [ ] Deploy Graylog ou Wazuh (1 VM inicial): configurar índices e parsers.
- [ ] Ativar ModSecurity (OWASP CRS) no Nginx.
- [ ] Implementar rate-limiting no /login (Nginx).
- [ ] Configurar fail2ban para bloqueio de IPs com falhas repetidas.
- [ ] Forçar MFA para contas administrativas.
- [ ] Teste de restauração de backup em ambiente isolado (documentar o procedimento).
- [ ] Criar alertas iniciais no SIEM: brute-force, SQLi, XSS, spike 5xx.

## Médio prazo (30–90 dias)
- [ ] Harden hosts (CIS basics), desabilitar root SSH, usar key-only.
- [ ] Segmentar redes (web/app/db) e aplicar security groups.
- [ ] Instalar/ativar EDR leve (Wazuh + integracao).
- [ ] Criar 12 regras de correlação no SIEM e dashboards.

## Longo prazo (90–180 dias)
- [ ] Pentest / code review + CI security scans.
- [ ] Automatizar testes de restore e verificações periódicas.
- [ ] Considerar SOAR básico para playbooks automatizados.

## Comandos úteis
- Teste Nginx config: `sudo nginx -t`
- Restart Nginx: `sudo systemctl restart nginx`
- Tail logs: `tail -f /var/log/nginx/access.log`
