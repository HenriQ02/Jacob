# Jacob — app pessoal de controlo de gastos

PWA (web app instalável) para Android do Henrique. Publicada em **https://henriq02.github.io/Jacob/** via GitHub Pages (branch `main`, pasta raiz). Responder ao utilizador em **português de Portugal**.

## Regras do projeto
- **Manter a app leve**: sem frameworks, sem bibliotecas externas, sem CDNs, sem imagens pesadas. Tudo em `index.html` (HTML + CSS + JS inline, vanilla JS).
- **Dados só no telemóvel**: `localStorage`, nunca enviar dados para servidores. O repositório é público mas só contém código.
- **Funciona offline**: qualquer ficheiro novo tem de entrar na lista `FILES` do `sw.js`. Ao alterar a app, **incrementar `CACHE` em `sw.js`** (ex.: `gastos-v4` → `gastos-v5`).
- **Nunca partir dados existentes**: alterações ao modelo de dados passam pela função `migrate()` (campos novos com valor por defeito). A chave `localStorage` continua `gastos.v1`.
- Valores monetários guardados em **cêntimos (inteiros)**. Formatação `pt-PT`, EUR, datas `YYYY-MM-DD` internamente.
- Textos da interface em PT-PT.
- Testar em viewport móvel (390×844), modo claro e escuro, antes de publicar.

## Ficheiros
- `index.html` — app inteira
- `manifest.json` — nome "Jacob", ícones, atalhos Android (`./?v=add`, `./?v=inc`, `./?v=sum`)
- `sw.js` — service worker (rede primeiro, cache como fallback offline)
- `icon-192.png`, `icon-512.png`, `icon-maskable.png`
- `.nojekyll` — evita processamento Jekyll no Pages

## Modelo de dados (`localStorage["gastos.v1"]`, versão 2)
```
{
  version: 2,
  cats:      [{id, emoji, name, type:"D"|"R", budget /*cêntimos, 0 = sem*/}],
  methods:   ["Cartão de débito", ...],          // também servem de "contas"
  tx:        [{id, date, cents, type:"D"|"R"|"T", cat, method, to /*só T*/, note, source /*fonte, só R*/}],
  recurring: [{id, type, cents, cat, method, note, source, day, from:"YYYY-MM", last:"YYYY-MM", active}],
  pending:   [{id, recId, ...campos de tx}],     // recorrentes à espera de confirmação
  quick:     [{id, label, cents, cat, method, note}],   // atalhos de despesa
  goals:     [{id, name, emoji, target, saved}],        // metas de poupança (não afetam contas)
  debts:     [{id, person, cents, dir:"owed"|"owe", note, date}],
  accInit:   {"<método>": cêntimos},             // saldos iniciais das contas
  lastBackup: timestamp|null, since: timestamp
}
```
Tema guardado à parte em `localStorage["gastos.theme"]` = `light` | `dark` | `auto`.

## Ecrãs (navegação inferior) e funcionalidades
- **Despesa** (`v-add`): valor, categoria (chips), data, conta/método, descrição. Atalhos no topo (1 toque regista, com "Anular"). Links "☆ Criar atalho" e "🔁 Recorrente". Alerta de orçamento a 80 % e ao exceder (`budgetAlert`).
- **Entradas** (`v-inc`): valor, **fonte** (texto com sugestões das fontes usadas), tipo, data, onde entrou. Resumo do mês por fonte.
- **Movimentos** (`v-list`): lista do mês agrupada por dia, pesquisa, tocar para editar (despesa/entrada/transferência).
- **Resumo** (`v-sum`): KPIs com comparação vs mês anterior, previsão de fim de mês (linear + recorrentes por vir), donut por categoria (conic-gradient), categorias vs orçamento com delta %, histórico 6 meses, saldo acumulado no ano (SVG).
- **Carteira** (`v-wal`): saldos por conta (saldo inicial + entradas − despesas ± transferências), transferências entre contas (tipo `T`, não contam como despesa/entrada), metas de poupança, dívidas (devem-me / eu devo).
- **Definições** (`v-set`, botão ⚙️ no cabeçalho): aparência (claro/escuro/auto), recorrentes, atalhos, categorias e orçamentos, métodos/contas, cópias de segurança (JSON completo e CSV), apagar tudo.
- Banners no topo: recorrentes por confirmar (✓/✕) e lembrete de cópia de segurança (30 dias sem exportar, ≥5 movimentos).
- `ask(title, fields, opt)` — diálogo genérico reutilizável (tipos: text, money, select, date, number, emoji).

## Visual
- Claro: variáveis em `:root`. Escuro **"Grafite"** (escolhido pelo utilizador): `html[data-theme="dark"]` — fundo `#121417`, cartões `#1c1f24` com contorno `#2a2f36`, destaque azul `#4c8dff`, vermelho `#ff6161` despesas, verde `#38c98a` entradas.
- `applyTheme()` corre no `<head>` para evitar flash.

## CSV (compatível com a folha Excel do utilizador)
Separador `;`, BOM UTF-8, decimais com vírgula. Colunas: `Data;Descrição;Categoria;Tipo;Método;Valor (€);Fonte`. Tipo = `Despesa` | `Receita` | `Transferência` (nas transferências, Fonte = `→ <conta destino>`).

## Publicar
Commit + push para `main`; o GitHub Pages atualiza em ~1 min. Lembrar de incrementar `CACHE` no `sw.js`.

## Ideias ainda não feitas
Importar extrato bancário (CSV do banco) com classificação automática; notificações push de orçamento; widget no ecrã inicial.
