# Laudo Técnico — Mercadão do Cariri

Data: 2026-09-18  
Responsável: Pedro & Opencode (MiMo v2.5 Free)

---

## Problema 1 — Total do pedido

**Causa raiz:**  
Em `loja/precos.py:32-33`, o desconto do cupom era aplicado no total (que já incluía frete), não só ao subtotal. Regra diz: cupom desconta só sobre produtos, nunca sobre frete. Se cupom > subtotal, desconto para no subtotal (sem troco).

**Solução:**  
Mudança em `loja/precos.py:32-33`:
- Antes: `desconto = min(cupom, total)` + `total = total - desconto - estornado`
- Depois: `desconto = min(cupom, subtotal)` + `total = (subtotal - desconto) + frete - estornado`

Garantindo que:
1. Frete não é afetado por cupom
2. Se cupom > subtotal, desconto = subtotal (não negativo)
3. Fórmula: `total = (subtotal - desconto_no_subtotal) + frete - estornos`

---

## Problema 2 — Relatório de clientes

**Causa raiz:**  
Em `loja/relatorios.py:16`, a condição `WHERE p.feito_em >= '2026-01-01'` filtrava pedidos antes do LEFT JOIN, excluindo clientes sem pedidos da lista final. Cliente novo (sem pedidos) e antigo (só pedidos antes de 2026) sumiam.

**Solução:**  
Mudança em `loja/relatorios.py:16`: mover condição de data para o ON do JOIN:
- Antes: `LEFT JOIN pedidos p ON p.cliente_id = c.id` + `WHERE p.feito_em >= '2026-01-01'`
- Depois: `LEFT JOIN pedidos p ON p.cliente_id = c.id AND p.feito_em >= '2026-01-01'`

Garante que:
1. Todos os clientes aparecem (LEFT JOIN preserva os clientes da tabela esquerda)
2. Contagem inclui só pedidos de 2026 em diante
3. Clientes sem pedidos aparecem com `qtd_pedidos = 0`

---

## Problema 3 — Busca por e-mail

**Causa raiz:**  
Em `loja/clientes.py:6`, a query usava formatação de string Python (`'%s' % email`), causando dois problemas:
1. **SQL Injection**: e-mail forjado poderia executar SQL arbitrário
2. **Erro com apóstrofo**: e-mail com `'` quebrava a sintaxe SQL

**Solução:**  
Mudança em `loja/clientes.py:6`: usar parâmetros de query do sqlite3:
- Antes: `sql = "SELECT id, nome, cidade FROM clientes WHERE email = '%s'" % email`
- Depois: `sql = "SELECT id, nome, cidade FROM clientes WHERE email = ?"` + `conn.execute(sql, (email,))`

Garante que:
1. Entrada tratada como não confiável (parâmetros escapam automaticamente)
2. E-mails com apóstrofo funcionam normalmente
3. Proteção contra SQL injection

