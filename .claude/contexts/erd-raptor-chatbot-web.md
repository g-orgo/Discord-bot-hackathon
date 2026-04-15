# ERD — raptor-chatbot-web

**Date:** 2026-04-13
**Stack:** React 19 · Vite · Zustand (ou Context)
**Port:** 5173

---

## Persistência

Apenas `sessionStorage` — sem banco de dados próprio.

```mermaid
classDiagram
    class SessionStorage {
        +String token   "JWT retornado pelo /auth/login"
        +Object user    "{ _id, email, displayName, discordUsername }"
    }
```

---

## APIs consumidas

### raptor-chatbot-server (3001)

| Método | Rota | Descrição |
|--------|------|-----------|
| POST | `/auth/login` | Login → armazena token + user no sessionStorage |
| POST | `/auth/register` | Cadastro |
| GET | `/auth/me` | Valida sessão ativa |
| GET | `/auth/history` | Carrega histórico do usuário |
| POST | `/auth/history` | Salva mensagem enviada via web |
| DELETE | `/auth/history` | Apaga histórico |
| GET | `/auth/history/stream` | SSE — recebe atualizações em tempo real |

### raptor-chatbot-llm (8000)

| Método | Rota | Descrição |
|--------|------|-----------|
| POST | `/api/chat` | Envia mensagem ao LLM |
| GET | `/api/system-prompt` | Lê system prompt atual |
| PUT | `/api/system-prompt` | Atualiza system prompt |

---

## Decisões de design

- **sessionStorage como fonte de verdade local:** token e user são lidos do storage; componentes não chamam API diretamente.
- **SSE para histórico ao vivo:** `GET /auth/history/stream` mantém o histórico sincronizado quando o bot Discord responde.
- **Store como intermediário:** chamadas de API são feitas pelo store, não pelos componentes.
