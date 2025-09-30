# PROPOSTA – Opção 2 (Consultoria)
**Cliente:** LojaZeta · **Data:** 2025-09-29

---

## 1. Sumário Executivo
**Riscos-chave**
- App web exposto com tentativas confirmadas de **SQL Injection (SQLi)**, **Cross-Site Scripting (XSS)** e **brute-force** em `/login`.  
- Logs dispersos nas instâncias → visibilidade reduzida e MTTD elevado.  
- Backups existentes mas **não testados** → risco de restore.  
- Time pequeno (2 devs, 1 ops) e orçamento limitado → priorizar ações de maior impacto/baixo custo.

**Visão da solução**
- **Defesa em camadas**: Perímetro (WAF/CRS), Rede (LB/segmentation), Host (hardening/EDR), App (input validation/WAF), Dados (privilégios/backups), Identidade (MFA).  
- **SIEM MVP**: centralização de logs e correlações com stack open-source (Wazuh + ELK/Graylog) para reduzir MTTD.  
- **Plano IR (NIST)** com runbooks simples e acionáveis para SQLi, XSS, brute-force e indisponibilidade.  
- **80/20**: quick wins em 30 dias para reduzir risco imediato.

**Ganhos esperados**
- Redução do MTTD/MTTR, bloqueio de ataques automáticos, validação de backups, e melhoria contínua do ciclo de defesa.

---

## 2. Escopo e Metodologia
**Cobertura**
- Camadas: perímetro, rede, host, aplicação, dados, identidade.  
- Monitoramento: centralização de logs, regras de correlação, alertas acionáveis.  
- IR: runbooks NIST (detecção → contenção → erradicação → recuperação → lições).

**Não incluído**
- Reescrita completa do código, pentest completo (pode ser proposta futura), SIEM comercial pago (podemos propor).

**Metodologia / Fases**
1. Inventário & levantamento (0–3d)  
2. Quick wins (3–30d) — 80/20  
3. SIEM MVP e regras (30–60d)  
4. Harden & automações (60–120d)  
5. Treino e melhoria contínua (contínuo)

---

## 3. Arquitetura de Defesa (Camadas)
**Perímetro**
- WAF (Nginx + ModSecurity + OWASP CRS) à frente do Load Balancer. Rate limiting no /login.  
**Rede / LB**
- Load Balancer (Nginx ou cloud LB) com redirecionamento para app pool. Segmentar subnets (web/app/db).  
**Host**
- CIS baseline, SSH key-only, desabilitar root login, agents para logs (Filebeat/Wazuh).  
**Aplicação**
- Prepared statements/ORM, validação e sanitização, Content Security Policy (CSP), headers (HSTS, X-Frame-Options).  
**Dados**
- Usuário DB com privilégios mínimos; backups automatizados e testes de restauração.  
**Identidade**
- MFA obrigatório para admins; lockout e rate-limits; uso de IdP se possível.

(Detalhes técnicos e configurações mínimas nas próximas seções do documento e no diagrama.mmd)

---

## 4. Monitoramento & SIEM (Plano mínimo viável)
**Fontes de log**
- Nginx (access/error), Node.js (stdout + logs estruturados), PostgreSQL (query & auth logs), syslog/journald, WAF/ModSecurity, alertas IDS (Se houver), autenticação/IdP logs.

**Arquitetura SIEM MVP**
- Coleta: Filebeat/Wazuh agent → Logstash/Graylog ingest → Elasticsearch → Kibana/Graylog UI.  
- Alternativa leve: Graylog + Elasticsearch + Mongo(for configs) + Beats.

**Use-cases (correlações & alertas iniciais)**
1. **SQLi pattern**: requests com `UNION SELECT`, `--`, `OR 1=1` → score alto.  
2. **XSS candidate**: payloads `<script>`, `onerror=`, `innerHTML` → detect.  
3. **Brute-force**: >5 falhas de login por IP em 5 minutos → bloqueio + alert.  
4. **Aumento de erros 5xx**: aumento >200% nas últimas 5 min.  
5. **Acesso anômalo ao DB**: queries longas ou SELECTs em tabelas sensíveis por usuários não esperados.  
6. **Alertas de integridade**: hash de arquivos críticos alterado (Wazuh).

**KPIs**
- MTTD (target 1–4h), MTTR (target 4–24h), Número de tentativas de ataque bloqueadas, % cobertura de logs (objetivo >90 das fontes críticas).

**Sample alert (ELK/KQL-like)**  
- Brute force: `index=nginx_access status:401 | stats count() by src_ip | where count > 5`

---

## 5. Resposta a Incidentes (NIST IR)
Estrutura comum: **Detecção → Contenção → Erradicação → Recuperação → Lições Aprendidas**.

**Runbooks** incluídos (SQLi, XSS, brute-force, indisponibilidade) com steps acionáveis, queries SIEM e comandos.

---

## 6. Recomendações (80/20) e Roadmap
**Quick wins (30 dias)**
1. Centralizar logs (Filebeat → Graylog/Wazuh).  
2. ModSecurity/OWASP CRS no Nginx.  
3. Rate limit + fail2ban para /login.  
4. Forçar MFA para admins.  
5. Teste de restore de backup.

**Médio prazo (90 dias)**
- SIEM configurado com 12 use-cases, host hardening, segmentação de rede, EDR leve.

**Longo prazo (180 dias)**
- Processos maturados, pen-test, automatizações SOAR simples, política de retenção e TI/Segurança formal.

---

## 7. Riscos, Custos e Assunções
**Riscos**
- Limitações do time e janela para mudanças em produção.  
- Falta de documentação da infra atual.

**Custos aproximados**
- Open-source: infra VM para SIEM (custo de infra), tempo de ops/dev.  
- Comercial: estimativa separada se desejar.

**Assunções**
- Acesso para instalar agentes e modificar Nginx/DB; consentimento para testes em ambiente de staging.

---

## 8. Conclusão e Próximos Passos
- Aprovar quick wins e autorizar instalação de um servidor de logs/SIEM MVP.  
- Entregar runbooks e treinar time (tabletop).  
- Métricas de sucesso: redução de MTTD/MTTR e bloqueio efetivo de vetores conhecidos.

---
