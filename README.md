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

## Regras de cobrança e mensagens de WhatsApp

### Juros de atraso — 5% ao dia, compostos, sem limite

Passado o vencimento, a dívida é corrigida em **5% ao dia sobre o valor já
corrigido** (juros compostos). Não existe teto de dias nem de valor: os juros
continuam acumulando **até a cobrança ser quitada** (marcada como *Pago*).

```
valor atualizado = valorFinal × 1,05 ^ (dias de atraso)
```

Exemplo com uma dívida de R$ 100,00:

| Dias de atraso | Valor atualizado |
| -------------- | ---------------- |
| 1 dia          | R$ 105,00        |
| 2 dias         | R$ 110,25        |
| 3 dias         | R$ 115,76        |
| 7 dias         | R$ 140,71        |
| 30 dias        | R$ 432,19        |
| 50 dias        | R$ 1.146,74      |

O cálculo fica centralizado na função `calcAtraso()` do `index.html`, usada pela
tabela, pelo alerta do topo e pela mensagem de cobrança — todas mostram sempre o
mesmo número. A taxa está na constante `JUROS_DIA` (0.05); é só ali que se mexe
para mudar o percentual.

### Botões de mensagem na tabela

Enquanto a cobrança **não estiver paga**, sempre há um botão para gerar a
mensagem — o texto é copiado para a área de transferência, é só colar no WhatsApp:

| Situação                | Botão (coluna Ações) |
| ----------------------- | -------------------- |
| Vence amanhã            | 📋 Amanhã            |
| Vence hoje              | 📋 Hoje              |
| Vencido (qualquer dia)  | 📋 Cobrar `N`d       |

O botão **📋 Cobrar** não tem limite de dias: aparece a partir do 1º dia de
atraso e continua aparecendo com 7, 50, 200 dias — some apenas quando a cobrança
é marcada como *Pago*. O badge da coluna *Status* (⚠ Atrasado `N`d) também é
clicável e copia a mesma mensagem.

A mensagem de atraso é detalhada: mostra a data do vencimento, os dias de atraso,
o valor original, os juros acumulados, o total atualizado do dia e a chave PIX.
A saudação (Bom dia / Boa tarde / Boa noite) acompanha o horário do envio.

## Como fazer alterações com segurança (passo a passo)

A regra de ouro: **testar no PC antes de publicar.**

1. Edite o `index.html` nesta pasta (`D:\Claude_VS_Code\Gestao-cobrancas-main`).
2. Abra o arquivo no navegador do PC e teste tudo (login, salvar, excluir,
   dashboard, recibo, botões de mensagem).
3. Deu certo? Publique:

   ```bash
   git add index.html
   git commit -m "descrição da mudança"
   git push
   ```

4. Aguarde ~1 min e recarregue o app no celular/PC.

Enquanto o `git push` não é feito, a alteração existe só no PC — o que está no ar
continua sendo a versão anterior.

Se algo quebrar, dá para voltar para qualquer versão anterior pelo histórico de
commits do GitHub (ou pelo git local nesta pasta).

## Backup local e estrutura do git

Esta pasta é um repositório git. A versão original (antes das melhorias de
segurança) está salva no primeiro commit, então nada se perde.

- `remote origin` → github.com/magalhaeseduardo71/Gestao-cobrancas
- Branch local **`main`** → branch remota **`main`** (é a que o GitHub Pages
  serve). Mesmo nome dos dois lados, então `git push` e `git pull` funcionam
  sozinhos, sem argumentos.
- `.claude/settings.local.json` fica no `.gitignore` — config local, não é
  publicada.

### Se aparecer erro de branch no push

Até setembro/2026 o branch local se chamava `master` e o remoto `main`. Com
nomes diferentes, o `git push` sozinho falhava com *"The upstream branch of your
current branch does not match the name of your current branch"* e era preciso
digitar `git push origin master:main`. Isso foi resolvido renomeando o branch
local para `main`.

Se um dia o erro voltar (por exemplo numa cópia antiga do repositório em outro
PC), confira em que branch você está e conserte com:

```bash
git branch --show-current          # se responder "master", é esse o caso
git branch -m master main          # renomeia o branch local
git branch -u origin/main main     # aponta para o branch remoto certo
```
