# Runbook: SQL Injection (SQLi)
**Objetivo:** Detectar, conter e remediar tentativa/comprovado SQLi.

## 1) Detecção
**Sinais**
- Requisições HTTP contendo payloads `UNION SELECT`, `' OR '1'='1'`, `--`, `/**/` etc.
- Erros 500 com mensagens de SQL no response/log (em staging).
- Alertas do WAF (ModSecurity -> audit log).
- SIEM rule: query nos logs Nginx + Node.js.

**Exemplo de query (Graylog/ELK)**
- Nginx access: `message:("*UNION*" OR "*SELECT*" OR "*OR 1=1*" OR "*--*")`
- Node/DB logs: `error: "syntax error at or near"`

## 2) Contenção (primeiros 15-60 min)
- Isolar fonte: identificar IP(s) de origem.  
  - Comando: `grep 'src_ip' /var/log/nginx/access.log | awk ...` (ajuste conforme formato).
- Bloquear IP no WAF/NGINX:  
  - Nginx: adicionar `deny <IP>;` no vhost temporariamente OU adicionar regra no WAF.  
- Ativar modo de proteção do WAF (ModSecurity -> `SecRuleEngine On` + OWASP CRS blocking).
- Se houver sessão suspeita (usuário autenticado), encerrar sessão e forçar reset de credenciais.

## 3) Análise
- Capturar payload completo, timestamp, user-agent, headers:  
  - `tail -n 200 /var/log/nginx/access.log | grep "<pattern>"`
- Verificar DB logs para queries anômalas:  
  - `sudo tail -n 500 /var/log/postgresql/postgresql-*.log | grep -i "SELECT\|UNION"`
- Determinar se houve exfiltração (SELECTs em tabelas sensíveis, dumps).

## 4) Erradicação
- Patch imediato na aplicação: **usar prepared statements** / parametrizar queries.  
  - Ex.: Node.js (pg): `client.query('SELECT * FROM users WHERE id = $1', [id])`
- Remover endpoints de debug e mensagens detalhadas de erro em produção.  
- Corrigir vulnerabilidade descoberta e deploy em staging primeiro.

## 5) Recuperação
- Restaurar integridade: rever logs pós-correção por 48h.  
- Validar integridade de dados (checksums) e comparar com backups.  
- Se houver suspeita de alteração de dados: restaurar de backup limpo para ambiente isolado e comparar.

## 6) Lições Aprendidas
- Atualizar checklist de code review para anti-patterns SQL.  
- Adicionar testes automatizados para injeção (fuzzing) em pipeline CI.  
- Reforçar regra no WAF e adicionar assinatura específica no SIEM.

## Observações técnicas rápidas
- WAF: ModSecurity + OWASP CRS; ajuste false positives nos primeiros 7 dias.  
- SIEM: criar regra que correlacione `modsec_audit` + `postgres stderr` + `nginx access`.
