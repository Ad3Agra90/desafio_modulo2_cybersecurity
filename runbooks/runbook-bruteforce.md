# Runbook: Brute-force / Ataque em /login
**Objetivo:** Detectar, conter e mitigar tentativas de força bruta em endpoints de autenticação.

## 1) Detecção
**Sinais**
- Múltiplas tentativas de login falhas de um mesmo IP ou mesma conta (>5 falhas/5min).  
- Picos de requisições POST em /login.

**Query exemplo**
- Nginx access: `method:POST AND path:"/login" AND status:401 | stats count() by src_ip | where count > 5`

## 2) Contenção (imediato)
- Implementar rate limiting no Nginx: `limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;` e aplicar no location /login.  
- Configurar fail2ban com filtro para Nginx logs (bloqueio IP por X minutos após N falhas).  
- Forçar lockout temporário de conta após 5 falhas.

## 3) Análise
- Identificar IPs, user-agents e se há pattern de proxy (TOR).  
- Verificar se credenciais comprometidas foram usadas com sucesso (last login, novos devices).

## 4) Erradicação
- Se contas comprometidas — forçar reset de senha e MFA, invalidar tokens.  
- Remover listas de IPs maliciosos da whitelist e bloquear via firewall ou WAF.  
- Considerar CAPTCHA ou step-up authentication para /login.

## 5) Recuperação
- Monitorar sucesso de logins pós-contenção e revisar logs por 72h.  
- Notificar usuários afetados (se aplicável).

## 6) Lições Aprendidas
- Automatizar thresholds no SIEM e ajustar conforme false positives.  
- Considerar uso de IdP/SSO com políticas de proteção avançadas.

## Comandos úteis
- fail2ban local tail: `sudo tail -f /var/log/fail2ban.log`
- Nginx rate limit snippet (exemplo):
  ```
  limit_req_zone $binary_remote_addr zone=one:10m rate=5r/m;
  server {
    location /login {
      limit_req zone=one burst=7 nodelay;
      proxy_pass http://app;
    }
  }
  ```
