# Appware — guia para o agente

O Appware trata software como **estrutura validável**, não como texto. A fonte da
verdade é o arquivo `project.appware.json`. O código é **gerado** a partir dele e
nunca editado à mão. Seu trabalho como agente é produzir/editar esse JSON; o
Appware valida e gera o código.

## Loop de trabalho (sempre)

1. Entenda a intenção do usuário (telas, campos, ações).
2. Edite o `project.appware.json` (ou use os comandos `appware` do modo estrito).
3. Rode `appware validate <arquivo>` — corrija TODOS os erros antes de seguir.
4. Rode `appware generate nextjs <arquivo> --out <dir>`.
5. `cd <dir> && npm install && npm run dev`.

Nunca pule a validação. O Appware rejeita estrutura inválida com mensagem clara —
itere até ficar verde. Não invente tipos: use só o vocabulário abaixo.

## As 4 camadas

- **Core**: conceitos universais e neutros (Context, Element, Container,
  ValueCollector, IntentTrigger, Event, Effect, State, Constraint, Resource,
  Relation, Flow). Não conhece "tela" ou "botão".
- **Kit (app)**: vocabulário de domínio `app.*` mapeado para o Core.
- **Generator (nextjs)**: transforma o modelo em um app Next.js.
- **Project**: o `project.appware.json`.

## Formato do project.appware.json

```jsonc
{
  "appware": "0.1",
  "project": "meu-app",
  "kits": ["app"],
  "targets": ["nextjs"],
  "resources": [ { "id": "res_auth", "name": "auth", "kind": "api" } ],
  "nodes": [
    {
      "id": "tela_login", "coreType": "Context", "kitType": "app.Screen", "name": "Login",
      "children": [
        { "id": "form", "coreType": "Container", "kitType": "app.Form", "name": "LoginForm",
          "children": [
            { "id": "email", "coreType": "ValueCollector", "kitType": "app.Input", "props": { "type": "email", "required": true } },
            { "id": "entrar", "coreType": "IntentTrigger", "kitType": "app.Button", "name": "Entrar",
              "events": [ { "name": "onClick", "flow": [ /* efeitos, ver abaixo */ ] } ] }
          ] }
      ],
      "states": ["loading", "error"],
      "constraints": ["token must use secure storage"]
    }
  ]
}
```

Cada nó carrega `coreType` (universal) **e** `kitType` (domínio). Eles devem ser
coerentes — use a tabela de mapeamento.

## Vocabulário do App Kit → Core (use exatamente estes)

Estruturais (viram nós na árvore):

| kitType | coreType | regras |
|---|---|---|
| app.Screen / app.Page | Context | contém Form, Input, Button; vira uma rota |
| app.Form | Container | contém só app.Input e app.Button |
| app.Input | ValueCollector | folha; exige prop `type` (text,email,password,number,tel,url,date) |
| app.Button | IntentTrigger | folha; aceita só o evento onClick |

Efeitos (vivem em `events[].flow`, resolvidos no Core):

| efeito | vira | uso |
|---|---|---|
| app.Link | TransitionEffect | navegação; `targetRef` = id de um Screen/Page |
| app.ApiCall | IOEffect (resource api) | `resourceRef` = id; `as` nomeia o resultado |
| app.Storage | IOEffect (resource storage) | `resourceRef` = id |
| app.Modal / app.Alert | FeedbackEffect | `params.message` |
| Condition | if/else | `when:{state,negate?}`, `then:[...]`, `else?:[...]` |
| StateEffect | define um State | `state`, `fromResult`, `resultPath`, `negate?` |
| GenericEffect | efeito nomeado | ex.: `{ "kind":"GenericEffect","name":"validateForm" }` |

Um efeito no flow é um objeto `{ "kind": "...", "name": "...", ... }`. Tipos de
`kind`: TransitionEffect, FeedbackEffect, IOEffect, GenericEffect, Condition, StateEffect.

### Exemplo de flow com lógica real (login)

```jsonc
"flow": [
  { "kind": "GenericEffect", "name": "validateForm" },
  { "kind": "IOEffect", "name": "login", "resourceRef": "res_auth", "as": "auth" },
  { "kind": "StateEffect", "name": "deriveAuth", "state": "authenticated", "fromResult": "auth", "resultPath": "ok" },
  { "kind": "Condition", "name": "branch", "when": { "state": "authenticated" },
    "then": [
      { "kind": "IOEffect", "name": "saveToken", "resourceRef": "res_store" },
      { "kind": "TransitionEffect", "name": "navigate", "targetRef": "tela_home" }
    ],
    "else": [
      { "kind": "FeedbackEffect", "name": "modal", "params": { "message": "Falha no login" } }
    ] }
]
```

## Modo estrito (CLI) — alternativa a editar o JSON

```bash
appware init project.appware.json --project meu-app
appware add app.Screen tela_login --name Login
appware add app.Form form --parent tela_login
appware add app.Input email --parent form --prop type=email --prop required=true
appware add app.Button entrar --parent form --name Entrar
appware event  entrar onClick
appware effect entrar onClick generic  --name validateForm
appware effect entrar onClick apicall  --resource res_auth --name login --as auth
appware effect entrar onClick setstate --state authenticated --from auth --path ok
appware effect entrar onClick link     --to tela_home
appware validate project.appware.json
appware generate nextjs project.appware.json --out ./meu-app
```

Cada comando é validado (Kit → Core). Condicionais aninhados (`then/else`) são
escritos direto no JSON, não pelo CLI.

## Regras de ouro

- Modelo é a verdade: nunca edite o código gerado; regenere.
- Só use `kitType`/`coreType` e `kind` desta lista. Tipo desconhecido = rejeitado.
- `app.Form` só contém `app.Input`/`app.Button`; `app.Input` exige `type`.
- Sempre rode `appware validate` e corrija até passar antes de gerar.
- Não implemente auth/DB reais no modelo — `app.ApiCall`/`app.Storage` são stubs.

## Limites atuais

Só App Kit + generator Next.js. Sem UI online, sem DB/auth reais, `then/else`
só via JSON. Tudo o que não está nesta lista ainda não é suportado.
