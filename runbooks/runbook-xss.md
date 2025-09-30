# Runbook: Cross-Site Scripting (XSS)
**Objetivo:** Detectar e mitigar injeção de scripts e exfiltração via XSS.

## 1) Detecção
**Sinais**
- Requisições com `<script>`, `onerror=`, `javascript:` em parâmetros.  
- Relatos de usuários sobre scripts aparecendo em interface.  
- WAF bloqueando payloads com pattern `<script>`.

**Query exemplo**
- Nginx/Node logs: `message:("*<script>*" OR "*onerror=*" OR "*javascript:*")`

## 2) Contenção
- Bloquear IP de origem (temporário).  
- Ativar regras WAF de bloqueio para payloads XSS.  
- Se ponto de persistência (ex.: comentário), remover conteúdo do DB imediatamente (colocar em quarentena).

## 3) Análise
- Identificar se XSS é refletido (reflected), armazenado (stored) ou DOM-based.  
- Reproduzir em ambiente de teste.  
- Verificar se houve roubo de cookies/sessões (logs de acessos com cookies anômalos).

## 4) Erradicação
- Para reflected/DOM XSS: sanitizar saída (escape) e implementar CSP (Content-Security-Policy).  
- Para stored XSS: limpar os campos afetados no DB, adicionar filtros de input e regras de validação no backend.  
- Adicionar escaping em templates (ex.: use funções de template engine com escaping por padrão).

## 5) Recuperação
- Remover conteúdo malicioso do DB e restaurar de backup se necessário.  
- Forçar renovação de cookies/sessões para usuários afetados (logout global) se houver risco.

## 6) Lições Aprendidas
- Incluir testes de XSS no CI (OWASP ZAP/Scanner básico).  
- Atualizar regras WAF e monitoramento para XSS patterns.

## Detalhes práticos
- CSP header exemplo: `Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none';`
- Escapar saídas: nunca inserir dados do usuário sem escape.
