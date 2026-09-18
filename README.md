# Gestão de Cobranças

App de uso pessoal para controle de cobranças/empréstimos com juros. Funciona no
celular e no PC como uma página web única (PWA).

## Onde fica hospedado

- **Link do app (celular/PC):** https://magalhaeseduardo71.github.io/Gestao-cobrancas/
- **Repositório (GitHub Pages):** github.com/magalhaeseduardo71/Gestao-cobrancas
  - O arquivo servido é o `index.html` na raiz do repositório.

## Tecnologias

- HTML/CSS/JS tudo dentro de um único `index.html` (sem build, sem instalação).
- **Firebase Realtime Database** — guarda as cobranças (`clientes`), os clientes
  cadastrados (`clientesCadastrados`), os investidores (`investidores`) e as
  configurações (`config`).
- **Firebase Authentication (e-mail/senha)** — protege o acesso aos dados.
- Chart.js (dashboard) e html2canvas (gerar recibos e extratos), carregados por CDN.

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

## Investidores (empréstimo com dinheiro de terceiros)

Serve para controlar empréstimos bancados por outra pessoa, separando **o que é
seu** do **que é dela**.

### A regra do dinheiro

O investidor entra com um **aporte** e recebe de volta o aporte mais uma
**porcentagem sobre ele**. Todo o resto do lucro é seu.

```
lucro do investidor  = aporte × (% dele / 100)
devolver a ele       = aporte + lucro do investidor
SEU lucro            = lucro do empréstimo − lucro do investidor
```

Exemplo (o caso padrão): Carlos aporta **R$ 100**, você empresta a **50%** e a
parte dele é **20%**.

| Item                       | Valor      |
| -------------------------- | ---------- |
| Aporte do Carlos           | R$ 100,00  |
| Valor final do empréstimo  | R$ 150,00  |
| Lucro do empréstimo        | R$ 50,00   |
| Lucro do Carlos (20%)      | R$ 20,00   |
| **Devolver ao Carlos**     | **R$ 120,00** |
| **Seu lucro**              | **R$ 30,00**  |

**Os juros de atraso (5% ao dia) são 100% seus.** A parte do investidor é fixa no
percentual combinado sobre o aporte e não cresce com o atraso.

### Cadastrar um investidor

Aba **👥 Cadastro** → botão **💼 Investidores**. Informe o nome e a **% padrão**
dele (o app já sugere 20). Essa % é só o padrão: ao criar o empréstimo dá para
mudar caso a caso.

Os investidores ficam no nó `investidores` do Firebase — separado dos clientes.

### Marcar um empréstimo como de investidor

Na aba **➕ Nova**, embaixo do formulário, ligue **💼 Dinheiro de investidor
(lucro dividido)**. Vale nos dois modos (Com Juros e Valor Fixo).

- Ao escolher o investidor, a **%** dele vem preenchida do cadastro.
- O **aporte** vem preenchido com o Valor Solicitado (é o caso normal), mas pode
  ser alterado se ele bancou só uma parte.
- Uma caixa verde mostra na hora **quanto é dele e quanto é seu**.

Quando há investidor, a linha *Seu Lucro* da prévia passa a se chamar **Lucro do
Empréstimo** (é o valor bruto) — o seu de verdade aparece na caixa da divisão.

Dá para ligar/desligar isso depois em qualquer cobrança pelo botão **✏️**
(Editar). Ao **🔄 Renovar**, o investidor é mantido automaticamente.

### Como isso aparece na aba Clientes

**Emblema com a sigla do investidor, antes do nome do cliente.** Um retângulo
pequeno e colorido com as iniciais: *Roberto Carlos* vira **RC**, *Carlos* vira
**CA**. Cada investidor tem **sua própria cor**, sempre a mesma (é sorteada pela
chave dele no Firebase, não muda sozinha), e o mesmo emblema aparece na lista da
aba Cadastro — bate na hora de olhar. Passando o mouse, mostra o nome completo.

Cobrança com **capital próprio não tem emblema nenhum** — a ausência já diz que
o lucro é todo seu.

**Setinha ▶** — só nas linhas com emblema. Clique e abre embaixo a divisão
daquele empréstimo: investidor, aporte, lucro do empréstimo, lucro do investidor,
quanto devolver, seu lucro e (se estiver atrasado) os juros de atraso que são só
seus.

**Filtro de capital** — ao lado de *Todos / Em Aberto / Pagos*:
💰 Todo capital • 👤 Meu capital • 💼 De investidor.

Para trocar as cores ou acrescentar mais, mexa na constante `CORES_INVESTIDOR`
do `index.html`. As funções são `siglaInvestidor()`, `corInvestidor()` e
`emblemaInvestidor()`.

### Recibo do investidor (🧾)

Clique na **setinha ▶** da cobrança e, na linha que abre, no botão verde
**🧾 Recibo do investidor**. Baixa um PNG (`recibo-investidor-nome.png`), do
mesmo jeito que o recibo do cliente.

É o mesmo desenho do recibo do cliente, mas **em verde** e com os números dele:

| Campo               | O que mostra                                        |
| ------------------- | --------------------------------------------------- |
| Título              | **CONFIRMAÇÃO DE INVESTIMENTO** (faixa verde clara)  |
| Investidor          | o nome do investidor, não o do cliente               |
| Data da Solicitação | a mesma data do empréstimo                           |
| Valor Investido     | o **aporte** dele                                    |
| Valor Final         | **aporte + o lucro dele** — R$ 100 a 20% sai R$ 120  |
| Data de Pagamento   | o vencimento do empréstimo                           |
| Rodapé              | *"Agradeço pela confiança e pela parceria!"* — no lugar do aviso de 5% ao dia |

O recibo do investidor **não mostra nada da sua margem**: nem o lucro do
empréstimo, nem a sua porcentagem, nem o nome do cliente que pegou o dinheiro.

Código: `gerarReciboInvestidor()` e `desenharReciboInvestidor()`.

### O que mudou nos cálculos

O dinheiro do investidor **não conta como seu**. Onde aparecia "Lucro" agora
aparece o **lucro líquido**, já descontada a parte dele:

| Onde                        | Antes          | Agora                                        |
| --------------------------- | -------------- | -------------------------------------------- |
| Coluna da tabela            | Lucro          | **Lucro (Meu)** — só o seu, limpo; o bruto fica na linha da setinha |
| Card na aba Clientes        | Lucro Recebido | **Meu Lucro Recebido** + card *Lucro dos Investidores* |
| Painel *A receber em*       | Lucro Previsto | **Meu Lucro Previsto** + *Volta p/ Investidores* |
| Resumo do cliente filtrado  | Lucro Total    | **Meu Lucro** + *Lucro do Investidor*        |
| Dashboard                   | Lucro Total    | **Meu Lucro Total** + *Lucro dos Investidores* + *Capital de Terceiros na Rua* |
| Gráficos de barra e linha   | Lucro          | **Meu Lucro** + série *Lucro Investidores*   |

Os totais **brutos** continuam brutos de propósito — *Em Aberto*, *Recebido* e
*Faturamento* são o dinheiro que entra ou está na rua, independente de quem
bancou. Quanto desse dinheiro tem dono aparece nos cards *Lucro dos Investidores*
(aba Clientes) e *Capital de Terceiros na Rua* (Dashboard), e caso a caso na
linha da setinha.

O **CSV** ganhou as colunas *Capital, Investidor, Aporte, % Investidor, Lucro
Investidor, Devolver ao Investidor* e *Meu Lucro*.

### Cada um vê só o que é dele

- **Recibo e extrato do cliente** continuam exatamente como eram: sem investidor,
  sem porcentagem, sem lucro.
- **Recibo do investidor** mostra o aporte dele e o que ele recebe de volta — e
  nada da sua margem nem do cliente.

A divisão completa (lucro do empréstimo, sua parte, a dele) é controle interno e
só existe dentro do app.

Código: bloco `INVESTIDORES — regra de divisão` no `index.html`
(`temInvestidor`, `aporteDe`, `lucroInvestidorDe`, `devolucaoInvestidorDe`,
`lucroMeuDe`, `lucroMeuAReceber`) e o emblema em `emblemaInvestidor()`.

## Ocultar valores (👁️)

O botão do olho fica na **barra azul do topo, à direita das abas** (ao lado de
*Cadastro*, embaixo do “Conectado”) — um só botão, no mesmo lugar em todas as
abas. Antes existiam dois, um na aba Clientes e outro no Dashboard.

Clicando, os valores em R$ ficam borrados (`filter: blur`) e o ícone vira 🙈;
clicando de novo, voltam. Vale para as quatro abas ao mesmo tempo, porque a
classe `valores-ocultos` é aplicada na `.main` inteira: cards de totais, coluna
de valores da tabela, resumo do cliente, painéis de vencimento, gráficos do
dashboard e a prévia de *Valor Final / Seu Lucro* da aba Nova.

É só visual, para usar o app perto de outras pessoas — nada muda no banco, e o
estado volta ao normal ao recarregar a página.

## Extrato do cliente (📄 Extrato)

Serve para lembrar um cliente de **vários vencimentos de uma vez**, em vez de
mandar um recibo por cobrança.

Como usar, na aba **👥 Clientes**:

1. Filtre por um cliente — pelo seletor *👤 Filtrar por cliente...*, pela busca,
   ou clicando no nome dele na tabela.
2. O bloco de resumo aparece acima da tabela e, nele, o botão **📄 Extrato**
   (ele **só existe quando há filtro por nome** — sem cliente selecionado, some).
3. Clique e o extrato baixa como PNG (`extrato-nome-do-cliente.png`), do mesmo
   jeito que o recibo.

O extrato leva **exatamente as cobranças que estão na tabela naquele momento** —
os filtros de status e de prazo (*📅 A receber em*) valem todos.

O **Valor Total em Aberto**, porém, soma sempre **só o que não foi pago**, mesmo
quando a lista mostra pagos junto. Ou seja: sem filtro de status, um cliente com
10 vencimentos e 5 quitados sai com as 10 linhas no extrato (cada uma com seu
status) e o rodapé dizendo *5 pendências* com a soma só dessas 5. Se a lista não
tiver nenhuma em aberto (filtro em *Pagos*), o rodapé mostra R$ 0,00 e
“Nenhuma pendência em aberto”.

Conteúdo da imagem — de propósito enxuto, porque ela vai para o cliente:

| Campo             | Observação                                            |
| ----------------- | ----------------------------------------------------- |
| Nome (título)     | cabeçalho azul, centralizado, sob “Extrato de Pendências” |
| Data Solicitação  | uma linha por cobrança                                 |
| Valor             | **valor final** (com juros do período já embutidos)    |
| Data Pagamento    | vencimento                                            |
| Status            | Pago (verde) / Atrasado `N`d (vermelho) / Vence Hoje (laranja) / Em Aberto (vermelho claro) |
| **Valor Total em Aberto** | soma **só das cobranças em aberto**, com a contagem de pendências |

**Porcentagem e lucro não aparecem** no extrato. As linhas vêm ordenadas pela
data de vencimento (mais antiga primeiro).

Se a busca estiver pegando mais de um cliente (ex.: digitar só “João” e existirem
dois), o app avisa e pede para selecionar um cliente específico — o extrato é
sempre de uma pessoa só.

Código: `gerarExtratoCliente()` e `desenharExtrato()` no `index.html`, usando
html2canvas — a mesma técnica do recibo.

## Como fazer alterações com segurança (passo a passo)

A regra de ouro: **testar no PC antes de publicar.**

1. Edite o `index.html` nesta pasta (`D:\Claude_VS_Code\Gestao-cobrancas-main`).
2. Abra o arquivo no navegador do PC e teste tudo (login, salvar, excluir,
   dashboard, recibo, extrato, botões de mensagem).
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
