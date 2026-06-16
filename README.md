# Appware — Agent Skills

Skills que ensinam qualquer agente de IA a construir software como **estrutura
validável** com o Appware (modelo `project.appware.json` → validação → código).

## Instalar

```bash
npx skills add humbertocruz/appware-skills
```

A CLI [`npx skills`](https://github.com/vercel-labs/skills) detecta seus agentes
(Claude Code, Cursor, Codex, Copilot, Gemini, Windsurf…) e instala em todos.

Instalar só esta skill, ou só num agente:

```bash
npx skills add humbertocruz/appware-skills --skill appware-app-builder
npx skills add humbertocruz/appware-skills --skill appware-app-builder -a claude-code
```

## Skills

- **appware-app-builder** — cria/edita apps Appware via `project.appware.json`, valida e gera Next.js.

Gerado por `appware skill`. Não edite à mão: regenere a partir do CLI do Appware.
