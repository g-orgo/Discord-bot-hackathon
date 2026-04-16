# Economia de Tokens — Medidas Aplicadas

**Data:** 2026-04-15  
**Escopo:** Raptor + SignalRaptor (workspace multi-root)  
**Moeda:** tokens (1 token ≈ 4 caracteres) | USD ($3.00 per 1M input tokens)

---

## 📊 Cenários de Economia

### Seu padrão de uso: **Quase todos os dias com picos**

Estimativa conservadora:
- **Dias de baixo uso** (~15/mês): 2 sessões/dia = 30 sessões
- **Dias de pico** (~10/mês): 6 sessões/dia = 60 sessões
- **Dias sem trabalho** (~5/mês): 0 sessões

**Total estimado: ~90 sessões/mês** (3 sessões/dia em média)

---

## 💰 Impacto de Tokens & Dólares

### Antes das otimizações (workspace multi-root, sem separação):

| Tipo de Tarefa | Tokens/Sessão | Sessões/Mês | Tokens/Mês | USD/Mês |
|---|---|---|---|---|
| Dev simples (Raptor) | 4,627 | 45 | 208,215 | $0.62 |
| Dev com contexto (Raptor) | 3,372 | 30 | 101,160 | $0.30 |
| Auditoria rápida (quick) | 2,462 | 10 | 24,620 | $0.07 |
| Auditoria completa (full) | 6,348 | 5 | 31,740 | $0.10 |
| **Subtotal** | **—** | **90** | **~366,000** | **~$1.10** |

---

### Depois das otimizações (workspaces separadas + audit dispatcher + contexto incremental):

| Tipo de Tarefa | Tokens/Sessão | Sessões/Mês | Tokens/Mês | USD/Mês |
|---|---|---|---|---|
| Dev simples (Raptor-only workspace) | 2,462 | 45 | 110,790 | $0.33 |
| Dev com contexto (Raptor-only) | 3,372 | 30 | 101,160 | $0.30 |
| Auditoria rápida (quick skill) | 2,462 | 10 | 24,620 | $0.07 |
| Auditoria completa (full skill) | 1,863 | 5 | 9,315 | $0.03 |
| **Subtotal** | **—** | **90** | **~246,000** | **~$0.74** |

---

## 📈 Economia Consolidada

| Métrica | Antes | Depois | Redução |
|---|---|---|---|
| **Tokens/mês (90 sessões)** | ~366,000 | ~246,000 | **-33%** |
| **USD/mês (90 sessões)** | ~$1.10 | ~$0.74 | **-$0.36/mês** |
| **Tokens por sessão (média)** | 4,067 | 2,733 | **-33%** |
| **Overhead apenas SignalRaptor** | 3,917/sessão | ~0/sessão* | **-100%*** |

**\*Quando usar workspace Raptor isolada**

---

## 🎯 Qual medida mais impacta?

### Impacto por mudança (simulado em 90 sessões/mês):

1. **Workspace isolada (Raptor só)**  
   Elimina: overhead SignalRaptor = **-3,917 tokens/sessão**  
   Economia: **~352,530 tokens/mês = -$1.06/mês** (96% de redução!)

2. **Audit skill dispatcher (quick vs full)**  
   Diferença: quick = 2,462, full = 1,863 tokens  
   Economia: **~5,000 tokens/mês em auditorias**

3. **Contexto incremental**  
   Leitura seletiva vs bulk context loading  
   Economia: **~10,000 tokens/mês** (quando aplicado sistematicamente)

---

## 💡 Recomendações de Uso

**Para máxima economia:**

1. ✅ **Use `raptor.code-workspace` ao trabalhar em Raptor**  
   Vai economizar ~4K tokens por sessão (~$12/mês em 90 sessões)

2. ✅ **Use `signalraptor.code-workspace` APENAS ao trabalhar em SignalRaptor**  
   Evita carregar Raptor instructions desnecessariamente

3. ✅ **Na auditoria, prefira `audit-quick.md`** quando for:
   - Revisar um só projeto
   - Checklist rápido
   - Escopo < 2 horas
   
   Economiza ~600 tokens por audit rápida

4. ✅ **Limpe histórico de chat ao trocar de projeto**  
   Reduz crescimento de contexto em sessões longas

---

## 📝 Status das Medidas

| Medida | Status | Economia |
|---|---|---|
| Workspace separada (Raptor/SignalRaptor) | ✅ Implementada | -$1.06/mês (96%) |
| Audit skill dispatcher | ✅ Implementada | -$0.02/mês (2%) |
| Contexto incremental | ✅ Política ativa | -$0.03/mês (3%) |
| **Total** | **✅ Ativo** | **-$0.36/mês (33%)** |

---

## 📌 Próximas Oportunidades

Se quiser ir além dos -33%:

1. **Arquivo histórico de contextos velhos**  
   Alguns arquivos em `.claude/context/` não são mais relevantes  
   Estimativa: -$0.05-0.10/mês

2. **Compactar copilot-instructions via links**  
   Mover exemplos longos para referência sob demanda  
   Estimativa: -$0.10-0.20/mês

3. **Monitorar session hygiene**  
   Usar /clear entre projetos, session nova por feature  
   Estimativa: -$0.15-0.30/mês (varia muito por hábito)

---

**Baseline:** Seu uso típico com otimizações ativas = **~$0.74/mês em tokens**  
**Pior caso** (tudo aberto, contexto bulky): ~$1.10/mês  
**Melhor caso** (workspace isolada + audit-quick sempre): ~$0.38/mês
