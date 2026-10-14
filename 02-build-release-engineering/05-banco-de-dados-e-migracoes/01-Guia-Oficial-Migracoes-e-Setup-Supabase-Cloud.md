# Guia Oficial de Migrações, Schemas e Integração Supabase Cloud
**Projeto:** Totalcad Viewer (RustDesk Core)  
**Autor:** Equipe de Engenharia de Software & Dados  
**Data:** 27/08/2026  
**Status:** HOMOLOGADO  

---

## 1. Visão Geral da Infraestrutura

O **Totalcad Viewer** utiliza a nuvem do **Supabase** como backend relacional, autenticação gerenciada e mensageria em tempo real.

- **Projeto Oficial:** `jyjqzwiothwpbfsvtjmv`
- **URL Base:** `https://jyjqzwiothwpbfsvtjmv.supabase.co`
- **Engine:** PostgreSQL 17.6
- **Gateway:** PostgREST 14.17 + GoTrue + Realtime WebSockets

---

## 2. Estrutura de Tabelas e Governança de Dados

```mermaid
erDiagram
    profiles ||--o{ support_sessions : "atende"
    support_sessions ||--o{ session_audit_logs : "possui"
    support_sessions ||--o{ session_chat_messages : "contem"
    profiles ||--o{ admin_audit_logs : "executa"
    profiles ||--o{ governance_policies : "atualiza"

    profiles {
        uuid id PK
        text username
        text display_name
        varchar title
        text avatar_url
        text role
        boolean is_active
        timestamptz created_at
    }

    support_sessions {
        uuid id PK
        text client_id
        text erp_code
        text client_name
        text ticket_number
        uuid technician_id FK
        text technician_name
        text service_scope
        varchar session_token
        varchar client_rustdesk_id
        varchar client_hostname
        text status
        uuid assumed_by_id FK
        boolean consent_granted
        int nps_score
        timestamptz scheduled_at
    }

    session_audit_logs {
        uuid id PK
        uuid session_id FK
        text action
        uuid performed_by_id FK
        text details
        timestamptz created_at
    }

    session_chat_messages {
        uuid id PK
        uuid session_id FK
        text sender_id
        text message
        boolean is_internal
        timestamptz sent_at
    }

    governance_policies {
        text id PK
        boolean chat_logging_enabled
        boolean zero_trust_consent_required
        boolean active_alerts_enabled
        int inactivity_timeout_minutes
        boolean nps_survey_enabled
    }

    admin_audit_logs {
        uuid id PK
        uuid admin_id FK
        text action
        text details
        timestamptz created_at
    }
```

---

## 3. Higiene de Credenciais e Segurança (.env)

1. **Nunca comitar segredos:** As chaves `service_role` e o token de gerenciamento `sbp_...` devem residir exclusivamente no `.env` local.
2. **Gitignore Obrigatório:** Os arquivos `.env`, `.env*.local` e `*.env` devem constar no `.gitignore`.
3. **Template .env.example:** Sempre manter o `.env.example` atualizado na raiz e subpastas de apps.

---

## 4. Comandos de Automação MCP e Agent Skills

```bash
# Adicionar Servidor MCP
claude mcp add --scope project --transport http supabase "https://mcp.supabase.com/mcp?project_ref=jyjqzwiothwpbfsvtjmv&features=docs%2Caccount%2Cdatabase%2Cdebugging%2Cdevelopment%2Cfunctions%2Cbranching"

# Instalar Agent Skills
npx skills add supabase/agent-skills
```
