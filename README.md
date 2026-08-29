# wrapper — Leilão · Alicerce

Página de uma linha só de propósito: carrega a `/exec` do app dentro de um
`<iframe credentialless>`, a partir de um domínio que **não** é o google.com.

## Por que existe

Com 2+ contas Google logadas no navegador, o Google reescreve a `/exec` para
`/u/0/`, `/u/1/`… chuta o índice errado e mostra *"Não foi possível abrir o
arquivo"*. O `credentialless` carrega o iframe sem os cookies de sessão do
Google — como uma aba anônima — e o roteador `/u/N/` não entra no caminho.

Não é segurança. Quem barra é o login do próprio app (`ORD-006`,
`requireSession_` em toda função de servidor). Isto resolve **roteamento de
conta**, só.

## Estado — 29/08/2026, medido

O Diretor republicou. `curl` sem credencial, no `@22` que esta no ar:

```
$ curl -sSL -o /dev/null -w "%{num_redirects} redirects, %{http_code}, %{size_download} B" "https://script.google.com/macros/s/AKfycbwSlTqM_AVNzw9dWS2u_WD5nbYbRSOIJaRbFwZakKxEVVVMfh2vObwRISL3POwD3vCUOA/exec"
0 redirects, 200, 58251 B      # <- o app de verdade: <title>Leilao · Alicerce</title>
                               # <- e NENHUM X-Frame-Options: o iframe carrega
```

Antes era `2 redirects, 905.719 B, X-Frame-Options: DENY` (a tela de login do
Google) — os dois pre-requisitos da Regra 1 agora valem:

1. **OK** — implantacao "Executar como: Eu" + "Quem pode acessar: Qualquer pessoa".
2. **OK** — `doGet()` com `setXFrameOptionsMode(ALLOWALL)` (`apps-script/Codigo.gs:33`).

> `curl -I` (HEAD) responde **403** neste endereco. Nao e erro: o Apps Script
> nao implementa HEAD. O **GET** e que vale, e ele traz o app.

## Manutencao

**A `/exec` muda quando o deployment e recriado — e ja mudou uma vez.** O `@21`
virou `@22` no mesmo dia, e o endereco antigo passou a responder **404**.
Quando acontecer de novo, troque nos **dois** lugares do `index.html` (o `src`
do iframe e o `href` do aviso); `tests/projeto.test.js` falha se os dois
divergirem.

Onde achar o endereco novo:

```bash
cd apps-script && clasp list-deployments -u secretaria
```

## Depois que estiver no ar

O link que a equipe salva é o do **Pages**, nunca a `/exec`. No dia em que o
deployment for recriado, você troca o endereço aqui dentro e o link de todo
mundo continua o mesmo — que é a razão de existir desta pasta.
