---
name: Identidade Visual (Branding)
description: Página admin /branding para personalizar nome, logo, favicon e cores (primary/accent) globalmente via app_settings
type: feature
---
A página `/branding` (admin-only) permite white-label da plataforma:
- **Nome da plataforma** (sidebar + document.title) e **título da tela de login**
- **Logo** e **Favicon** (upload via bucket `message-media/branding/`)
- **Cores Primary e Accent** (HSL) com presets prontos e color picker (hex↔hsl)

Persistência: tabela `app_settings` com `key='branding_config'` e `value` JSON serializado.
Provider `BrandingProvider` em `src/hooks/useBranding.tsx` carrega config no boot, aplica via CSS variables (`--primary`, `--ring`, `--accent`, `--sidebar-primary`) e injeta favicon/title no document.
Consumido por `AppSidebar` (logo + nome) e `Auth` (logo + título).
