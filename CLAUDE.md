# Jacob — app pessoal de controlo de gastos

App **só para acompanhar os gastos do mês** (e as entradas). Sem transferências (o utilizador pediu explicitamente). Há saldos por conta só para comparar com o banco. PWA (web app instalável) para o **iPhone** do Henrique, instalada pelo Safari (Partilhar → Adicionar ao ecrã principal). Também usada pela namorada num **Android** (instalada pelo Chrome, ⋮ → Instalar app); cada telemóvel tem os seus próprios dados. Não fazer nada específico do Henrique sem ser configurável. Publicada em **https://henriq02.github.io/Jacob/** via GitHub Pages (branch `main`, pasta raiz). Responder ao utilizador em **português de Portugal**.

## Regras do projeto
- **Manter a app leve**: sem frameworks, sem bibliotecas externas, sem CDNs, sem imagens pesadas. Tudo em `index.html` (HTML + CSS + JS inline, vanilla JS).
- **Dados só no telemóvel**: `localStorage`, nunca enviar dados para servidores. O repositório é público mas só contém código.
- **Funciona offline**: qualquer ficheiro novo tem de entrar na lista `FILES` do `sw.js`. Ao alterar a app, **incrementar `CACHE` em `sw.js`** (ex.: `jacob-v7` → `jacob-v8`).
- **Nunca partir dados existentes**: alterações ao modelo de dados passam pela função `migrate()` (vê `d.version` antes de mudar) (campos novos com valor por defeito). A chave `localStorage` continua `gastos.v1`.
- Valores monetários guardados em **cêntimos (inteiros)**. Formatação `pt-PT`, EUR, datas `YYYY-MM-DD` internamente.
- Textos da interface em PT-PT.
- Testar em viewport móvel (390×844), modo claro e escuro, antes de publicar.
- **iPhone**: campos de texto com `font-size` ≥ 16px (abaixo disso o iOS faz zoom ao tocar). A app instalada tem armazenamento separado do Safari/Brave: mudar de browser obriga a exportar e restaurar a cópia JSON. O Brave no iOS não consegue instalar; tem de ser o Safari.

## Ficheiros
- `index.html` — app inteira
- `manifest.json` — nome "Jacob", ícones, atalhos (`./?v=add`, `./?v=inc`, `./?v=sum`; só funcionam no Android, não no iPhone)
- `sw.js` — service worker (rede primeiro, cache como fallback offline)
- `icon-192.png`, `icon-512.png`, `icon-maskable.png` (talão a 74 %, para recorte), `apple-touch-icon.png` (180×180, iPhone). Ícone: talão branco inclinado sobre degradê violeta→azul (`#7b6cff`→`#2f6fde`), escolhido pelo utilizador
- `.nojekyll` — evita processamento Jekyll no Pages

## Modelo de dados (`localStorage["gastos.v1"]`, versão 3)
```
{
  version: 3,
  cats:      [{id, emoji, name, type:"D"|"R", budget /*cêntimos, 0 = sem*/}],
  methods:   ["BPI","Revolut","Trade Republic","Dinheiro"], // contas reais (campo Conta)
  bases:     {"BPI": {cents, date:"YYYY-MM-DD", at:timestamp}}, // saldo base por conta
  tx:        [{id, date, cents, type:"D"|"R"|"T", cat, method, to /*só T*/, note, source /*fonte, só R*/}],
  recurring: [{id, type, cents, cat, method, note, source, day, from:"YYYY-MM", last:"YYYY-MM", active}],
  pending:   [{id, recId, ...campos de tx}],     // recorrentes à espera de confirmação
  quick:     [{id, label, cents, cat, method, note}],   // despesas habituais fixas (Início)
  goals:     [{id, name, emoji, target, saved}],        // metas de poupança (não afetam contas)
  debts:     [{id, person, cents, dir:"owed"|"owe", note, date}],
  budget:    cêntimos,                           // orçamento do mês (0 = sem)
  hiddenHab: [habKey, ...],                      // sugestões de habituais escondidas
  recapSeen: "YYYY-MM",                         // último "fecho do mês" fechado
  accInit:   {...},                              // antigo, já não usado (substituído por bases)
  profile:   {name, done},                       // nome (inicial no canto); done=false mostra as boas-vindas
  lastBackup: timestamp|null, since: timestamp
}
```
Tema guardado à parte em `localStorage["gastos.theme"]` = `light` | `dark` | `auto`.

## Ecrãs e navegação (estilo app de banco)
Barra de baixo com 3 separadores: **Início · Movimentos · Resumo**. Despesa, Entrada, Metas e dívidas e Definições abrem como páginas por cima, com botão de voltar e sem barra de baixo (`body.sub`). `show(v)` troca de ecrã; `openSub(v)` abre uma página e guarda em `backTo` o separador para onde voltar; `back()` volta.
- **Início** (`v-home`): cabeçalho com avatar "J" (Definições) e barra "Pesquisar" (abre Movimentos). Gastos do mês em grande (`bigMoney()`), linha com entradas e mês anterior, **orçamento do mês** (`#homeBudget`: barra, quanto resta e quanto podes gastar por dia; toque abre `editBudget()`), 3 ações redondas (Despesa, Entrada, Mais), **Habituais** (com "Editar"), banners, **fecho do mês** e as 6 transações mais recentes.
- **Boas-vindas** (`v-hello`, `renderHello`): só aparece com dados novos (`profile.done === false`). Pede nome, contas (lista de bancos PT + outras) e orçamento opcional. Dados antigos sem `profile` ficam `done:true` em `migrate()`, por isso o Henrique nunca vê este ecrã.
- **Fecho do mês** (`renderRecap`): nos primeiros 10 dias do mês, cartão "Outubro em números" (total vs mês anterior, categoria com mais gastos, maior despesa, nº de despesas e média por dia) até o utilizador o fechar (`recapSeen`).
- **Página Habituais** (`v-hab`, `renderHabPage`): fixas com subir/descer e Editar (`editHabit`); sugeridas com Fixar e Esconder; escondidas com Mostrar.
- **Habituais** (`renderHabits`, `habits()`): cartões de um toque no Início. Um toque regista a despesa hoje (toast com Anular; toques em menos de 800 ms ignorados). Fixas = `db.quick` (criadas em Mais → Nova despesa habitual, ou "Guardar como habitual" no formulário). Sugeridas = mesma descrição+valor+categoria 3 ou mais vezes nos últimos 90 dias, exceto as de `hiddenHab`. Cada cartão mostra a última vez ("há 4 dias").
- **Fundo do Início** (`.glow`): 3 manchas desfocadas a mexer devagar (só transform; parado com reduced-motion). A cor diz o ritmo de gastos face ao orçamento do mês (ou ao mês passado, se não houver orçamento) até ao mesmo dia: `body[data-mood]` calm (azul/verde-água, abaixo de 90 %), warm (violeta, 90–110 %), hot (laranja/vermelho, acima). Pedido pelo utilizador (estilo Revolut). No Início o cabeçalho é transparente por cima do fundo.
- **Mais** (`#more`, folha que sobe de baixo): nova despesa habitual, despesa/entrada recorrente, Contas e saldos, Metas e dívidas, Definições.
- **Contas e saldos** (`v-acc`): saldo base por conta (o que o banco mostra no momento). `accBalance()` = base + entradas − despesas dessa conta registadas depois da base (mesmo dia conta só se o movimento foi criado depois, via `createdAt(id)`). Botão Acertar compara com o saldo real e pode trocar a base. Lista os movimentos com nomes de conta antigos (MB Way, Cartão de débito…) para o utilizador escolher a conta; nunca mapear automaticamente sem ele escolher.
- **Despesa** (`v-add`) / **Entrada** (`v-inc`): seletor Despesa|Entrada no topo, valor grande, categoria (chips), data, conta, descrição. Guardar volta ao separador de origem. Alerta de orçamento a 80 % e ao exceder (`budgetAlert`).
- **Movimentos** (`v-list`): mês, pesquisa, chips de categoria (`listCat`), totais numa linha, lista agrupada por dia. Tocar para editar.
- **Resumo** (`v-sum`): gastos do mês em grande com comparação, previsão e orçamento; tocar numa barra dos 6 meses muda de mês; tocar numa categoria abre os Movimentos filtrados; entradas e saldo do mês, últimos 6 meses (gráfico principal), categorias (barra de partilha + linhas com orçamento), entradas por fonte (`renderSrc`), saldo acumulado no ano.
- **Metas e dívidas** (`v-wal`, página aberta em Mais): metas de poupança e dívidas (devem-me / eu devo).
- **Removido a pedido do utilizador:** transferências e o saldo total no Início. Os dados antigos (`tx` do tipo `T`, `accInit`) ficam guardados mas não aparecem; não voltar a pôr sem ele pedir.
- **Definições** (`v-set`): o teu nome, aparência, recorrentes, habituais, categorias e orçamentos, contas, cópias de segurança (JSON e CSV, ficheiros `jacob-…`), apagar tudo.
- Linhas de movimentos: `txRow(t)` com `avatar()` (círculo com a inicial na cor da categoria, `catColor()`). Datas relativas com `relDay()` (Hoje, Ontem, 6 nov).
- `ask(title, fields, opt)` — diálogo genérico reutilizável (tipos: text, money, select, date, number).
- Nome visível da app: **Jacob**. As chaves `localStorage` `gastos.v1` e `gastos.theme` ficam com o nome antigo de propósito (mudá-las perdia os dados).

## Visual
- Inspiração: Revolut (pedido do utilizador). Títulos dos separadores a 28px/700; linhas de movimentos a toda a largura do cartão com separador a começar depois do avatar; resposta visual ao toque (`:active`); ecrãs entram com fade (separadores) ou deslizam da direita (páginas); valores com separador de milhares (`useGrouping:"always"`).
- Claro: variáveis em `:root`. Escuro **"Grafite"** (escolhido pelo utilizador): `html[data-theme="dark"]` — fundo `#121417`, cartões `#1c1f24` com contorno `#2a2f36`, destaque azul `#4c8dff`, vermelho `#ff6161` despesas, verde `#38c98a` entradas.
- `applyTheme()` corre no `<head>` para evitar flash.

## CSV (compatível com a folha Excel do utilizador)
Separador `;`, BOM UTF-8, decimais com vírgula. Colunas: `Data;Descrição;Categoria;Tipo;Método;Valor (€);Fonte`. Tipo = `Despesa` | `Receita` | `Transferência` (nas transferências, Fonte = `→ <conta destino>`).

## Publicar
Commit + push para `main`; o GitHub Pages atualiza em ~1 min. Lembrar de incrementar `CACHE` no `sw.js`.

## Ideias ainda não feitas
Importar extrato bancário (CSV do banco) com classificação automática; notificações push de orçamento; widget no ecrã inicial.
