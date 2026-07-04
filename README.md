# Gestão de Cobranças

App de uso pessoal para controle de cobranças/empréstimos com juros. Funciona no
celular e no PC como uma página web única (PWA).

## Onde fica hospedado

- **Link do app (celular/PC):** https://magalhaeseduardo71.github.io/Gestao-cobrancas/
- **Repositório (GitHub Pages):** github.com/magalhaeseduardo71/Gestao-cobrancas
  - O arquivo servido é o `index.html` na raiz do repositório.

## Tecnologias

- HTML/CSS/JS tudo dentro de um único `index.html` (sem build, sem instalação).
- **Firebase Realtime Database** — guarda os clientes e as configurações.
- **Firebase Authentication (e-mail/senha)** — protege o acesso aos dados.
- Chart.js (dashboard) e html2canvas (gerar recibos), carregados por CDN.

## Firebase (projeto `magalhaes-70905`)

- Banco: `https://magalhaes-70905-default-rtdb.firebaseio.com/`
- Login exige usuário autenticado. As **regras** do Realtime Database são:

  ```json
  {
    "rules": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
  ```

- O app pede e-mail/senha do Firebase apenas na primeira vez em cada aparelho
  (a sessão fica salva). O PIN da tela inicial é um bloqueio local rápido.

## Como fazer alterações com segurança (passo a passo)

A regra de ouro: **nunca editar direto o que está no ar. Testar antes.**

1. Trabalhe numa cópia (ex.: `index-novo.html`), não no `index.html` publicado.
2. Abra `index-novo.html` no navegador do PC e teste tudo (login, salvar,
   excluir, dashboard, recibo).
3. Deu certo? Suba a nova versão para o GitHub:
   - No repositório, **Add file → Upload files**.
   - Arraste o arquivo já renomeado para `index.html`.
   - **Commit changes** (o GitHub guarda a versão anterior no histórico).
4. Aguarde ~1 min e recarregue o app no celular/PC.

Se algo quebrar, dá para voltar para qualquer versão anterior pelo histórico de
commits do GitHub (ou pelo git local nesta pasta).

## Backup local

Esta pasta é um repositório git. A versão original (antes das melhorias de
segurança) está salva no primeiro commit, então nada se perde.
