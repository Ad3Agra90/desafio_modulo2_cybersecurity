# Runbook: Indisponibilidade / Queda de Serviço
**Objetivo:** Restaurar rapidamente serviço e identificar causa raiz.

## 1) Detecção
**Sinais**
- Alertas de healthcheck do load balancer falhando.  
- Picos de latência e erros 5xx nos logs.  
- Usuários relatando indisponibilidade.

**Query exemplo**
- `index=nginx_access status:500 OR status:502 OR status:504 | stats count() by status`

## 2) Contenção
- Redirecionar tráfego para instâncias saudáveis (drain/replace instance no LB).  
- Colocar manutenção se o impacto for grande e correção demorar (>15 min).

## 3) Análise
- Verificar logs da aplicação (stack traces), uso de CPU/mem, I/O e conexões de DB.  
  - `top`, `dmesg`, `journalctl -u nodeapp`, `ss -tnp | grep node`  
- Verificar DB (locks, long queries): `SELECT pid, now()-pg_stat_activity.query_start AS duration, query FROM pg_stat_activity WHERE state='active' AND (now()-pg_stat_activity.query_start) > interval '5 seconds';`

## 4) Erradicação
- Se loop infinito ou leak: reiniciar serviço controladamente (`systemctl restart nodeapp`), escalar para rollback de deploy se início do problema coincide com deploy recente.  
- If DB overloaded: kill long queries, increase pool temporarily, or promote replica.

## 5) Recuperação
- Testes de saúde após correção; gradualmente recolocar instâncias no LB.  
- Executar restore em staging para reproduzir e corrigir.

## 6) Lições Aprendidas
- Implementar healthchecks, circuit-breakers e deploys canary.  
- Monitorar métricas de recursos e alertas proativos.

## Comandos Rápidos
- Systemd: `sudo journalctl -u nodeapp -n 200 --no-pager`
- PostgreSQL: `sudo -u postgres psql -c "SELECT * FROM pg_stat_activity LIMIT 20;"`
