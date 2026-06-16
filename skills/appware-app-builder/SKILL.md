---
name: appware-app-builder
description: Build software as validated structure with Appware instead of writing code by hand. Use when the user wants to create or edit an app, screen, form, or flow via a project.appware.json model, mentions Appware, or references app.Screen / app.Form / app.Input / app.Button, or asks to generate a Next.js app from a structural model.
---

# Appware app builder

Construa apps como **estrutura validável** (`project.appware.json`), não como
código. O Appware valida o modelo e gera o app Next.js.

## Quando usar

O usuário quer criar/alterar um app, tela, formulário, navegação ou fluxo, e o
projeto usa Appware (há um `project.appware.json`, ou ele cita Appware/app.*).

## Como agir (sempre este loop)

1. Leia `reference.md` (vocabulário e formato completos) antes de editar.
2. Traduza a intenção do usuário em nós/efeitos do `project.appware.json`.
3. `appware validate <arquivo>` — corrija todos os erros antes de seguir.
4. `appware generate nextjs <arquivo> --out <dir>` e then `npm install && npm run dev`.

Nunca invente tipos fora do vocabulário em `reference.md`; a validação rejeita o
inválido — itere até passar. Nunca edite o código gerado; regenere a partir do modelo.

## Arquivos

- `reference.md` — guia completo do padrão Appware (vocabulário, mapeamentos, efeitos, comandos).
- `appware.schema.json` — JSON Schema do `project.appware.json` (autocomplete/validação de forma).
- `example.project.appware.json` — um projeto mínimo e válido como referência.
