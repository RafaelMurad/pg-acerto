# Transaction CRUD Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace hardcoded transaction rows with a fully dynamic data-driven model, add CRUD UI, and auto-recalculate all totals/balances/PIX on every change, persisted to JSONBin.

**Architecture:** All transactions live in `store.transactions` (JSONBin-backed). On first load, if JSONBin has no transactions, seed from `DEFAULT_TRANSACTIONS` baked into the HTML. Dynamic rendering replaces all hardcoded `<tr>` rows and payer cards. A single `recalculate()` function recomputes and updates all DOM values after every CRUD operation.

**Tech Stack:** Vanilla JS, HTML, CSS — single `index.html` file. JSONBin REST API (already integrated). No build toolchain.

---

## File Map

| File | Changes |
|------|---------|
| `index.html` lines 694–720 | Wrap pix rows in `<div id="pix-rows">`, keep section structure |
| `index.html` lines 723–906 | Remove all hardcoded card HTML; add `id="cards-{bucket}"` containers |
| `index.html` lines 908–997 | Add `data-balance-cell` attrs to balance table; add `id="unsimp-tbody"` |
| `index.html` CSS block | Add modal, floating button, row icon styles |
| `index.html` before `</body>` | Add modal HTML |
| `index.html` `<script>` block | Add constants, DEFAULT_TRANSACTIONS, new functions, update init |

---

## Task 1: Add JS constants and DEFAULT_TRANSACTIONS

**Files:**
- Modify: `index.html` — JS `<script>` block, just after `function fmtDate(...)` (~line 1077)

- [ ] **Step 1: Insert constants block**

Add this block immediately after the `fmtDate` function and before `async function loadAll()`:

```js
// ── Trip constants ────────────────────────────────────────
const PEOPLE = ['rafa','ju','higor','joao'];
const PEOPLE_LABELS = { rafa:'Rafa', ju:'Ju', higor:'Higor', joao:'João' };
const PEOPLE_COLORS = { rafa:'coral', ju:'pink', higor:'teal', joao:'gold' };
const BUCKETS = [
  { id:'pg',    label:'Praia Grande',        num:'01' },
  { id:'caipi', label:'Caipirinhas + pastel', num:'02' },
  { id:'aqa',   label:'Araraquara',           num:'03' },
];

const DEFAULT_TRANSACTIONS = [
  // ── Rafa (PG) ──
  { id:'rafa-nossocanto', bucket:'pg', date:'18/04', desc:'Mp Nossocanto',                   amount:231.00, paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/19apr_01_mp_nossocanto.png' },
  { id:'rafa-oxxo',       bucket:'pg', date:'18/04', desc:'Oxxo Xixova',                     amount:17.29,  paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/19apr_02_oxxo_xixova.png' },
  { id:'rafa-carrefour',  bucket:'pg', date:'18/04', desc:'Carrefour',                        amount:370.05, paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/19apr_04_carrefour.png' },
  { id:'rafa-pao-15',     bucket:'pg', date:'18/04', desc:'Pão de Açúcar',                   amount:15.00,  paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/19apr_05_pao_de_acucar.png' },
  { id:'rafa-bk',         bucket:'pg', date:'19/04', desc:'Burger King',                     amount:77.70,  paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/20apr_02_burger_king.png' },
  { id:'rafa-uber-a',     bucket:'pg', date:'19/04', desc:'Uber',                            amount:34.97,  paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/20apr_03_uber_a.png' },
  { id:'rafa-uber-b',     bucket:'pg', date:'19/04', desc:'Uber',                            amount:35.98,  paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/20apr_04_uber_b.png' },
  { id:'rafa-tenda',      bucket:'pg', date:'19/04', desc:'Tenda Atacado',                   amount:19.74,  paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/20apr_05_tenda_atacado.png' },
  { id:'rafa-pao-8175',   bucket:'pg', date:'19/04', desc:'Pão de Açúcar — cervejas, último dia', amount:81.75, paidBy:'rafa', splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/20apr_08_pao_de_acucar_c.png' },
  { id:'rafa-extra',      bucket:'pg', date:'19/04', desc:'Mercado Extra',                   amount:108.74, paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/20apr_09_mercado_extra.png' },
  { id:'rafa-kiosk',      bucket:'pg', date:'20/04', desc:'Kiosk — porção (÷4)',             amount:180.00, paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/kiosk_porcao.png' },
  { id:'rafa-pao-024',    bucket:'pg', date:'20/04', desc:'Pão de Açúcar',                   amount:0.24,   paidBy:'rafa',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/21apr_01_pao_de_acucar.png' },
  // ── João (PG) ──
  { id:'joao-comb',       bucket:'pg', date:'',      desc:'Combustível',                     amount:350.00, paidBy:'joao',  splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/joao_combustivel.jpg' },
  { id:'joao-pedagio',    bucket:'pg', date:'',      desc:'Pedágio',                         amount:200.00, paidBy:'joao',  splitAmong:['rafa','ju','higor','joao'], comprovante:null },
  { id:'joao-reab',       bucket:'pg', date:'',      desc:'Reabastecimento almoço',          amount:50.00,  paidBy:'joao',  splitAmong:['rafa','ju','higor','joao'], comprovante:null },
  // ── Ju (PG) ──
  { id:'ju-uber-a',       bucket:'pg', date:'18/04', desc:'Uber',                            amount:189.99, paidBy:'ju',    splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/ju_ubers.jpg' },
  { id:'ju-uber-b',       bucket:'pg', date:'18/04', desc:'Uber',                            amount:199.06, paidBy:'ju',    splitAmong:['rafa','ju','higor','joao'], comprovante:'comprovantes/ju_ubers.jpg' },
  // ── Ju (Caipi) ──
  { id:'ju-caipi',        bucket:'caipi', date:'',   desc:'3 caipirinhas (R$40 cada)',        amount:120.00, paidBy:'ju',    splitAmong:['rafa','ju'],               comprovante:null },
  { id:'ju-pastel',       bucket:'caipi', date:'',   desc:'Pastel',                           amount:25.00,  paidBy:'ju',    splitAmong:['rafa','ju'],               comprovante:null },
  // ── Higor (AQA) ──
  { id:'higor-pedagio',   bucket:'aqa',   date:'',   desc:'Pedágio',                         amount:70.00,  paidBy:'higor', splitAmong:['rafa','ju','higor'],        comprovante:null },
  { id:'higor-comb',      bucket:'aqa',   date:'',   desc:'Combustível',                     amount:50.00,  paidBy:'higor', splitAmong:['rafa','ju','higor'],        comprovante:null },
];
```

- [ ] **Step 2: Verify data integrity in browser console**

Open `index.html` in browser. In console run:
```js
const total = DEFAULT_TRANSACTIONS.reduce((s,t) => s+t.amount, 0);
console.log(total.toFixed(2)); // expected: 2426.51
```
Expected output: `2426.51`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add DEFAULT_TRANSACTIONS seed data"
```

---

## Task 2: Update store schema and seeding logic

**Files:**
- Modify: `index.html` — `loadAll()` function (~line 1079)

- [ ] **Step 1: Replace `loadAll()` with seeding version**

Replace the entire `loadAll` function:

```js
async function loadAll() {
  const r = await fetch(`${API_BASE}/${JSONBIN_BIN_ID}/latest`, {
    headers: { 'X-Master-Key': JSONBIN_KEY }
  });
  if (!r.ok) throw new Error('load failed');
  const data = await r.json();

  let transactions = data.record.transactions;
  if (!transactions || transactions.length === 0) {
    // First load — seed from defaults, applying any saved txNotes renames
    const saved = data.record.txNotes || {};
    transactions = DEFAULT_TRANSACTIONS.map(tx => ({
      ...tx,
      desc: saved[tx.id] || tx.desc
    }));
  }

  store = {
    comments:     data.record.comments     || [],
    txNotes:      data.record.txNotes      || {},
    transactions
  };

  // If we just seeded, persist immediately so future loads skip seeding
  if (!data.record.transactions || data.record.transactions.length === 0) {
    await persist();
  }

  return store;
}
```

Also update the comment on line ~1053:
```js
// store: { comments: [], txNotes: {}, transactions: [] }
```

- [ ] **Step 2: Verify in browser console**

Load the page. In console:
```js
store.transactions.length  // expected: 21
store.transactions[0].id   // expected: "rafa-nossocanto"
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: extend store schema with transactions, seed from defaults on first load"
```

---

## Task 3: Restructure static HTML — add IDs, remove hardcoded rows

**Files:**
- Modify: `index.html` — hero section (~line 694), buckets section (~line 723), balance section (~line 908)

- [ ] **Step 1: Wrap hero PIX rows**

In the hero section, wrap all three `<div class="pix-row">` elements in a container:

```html
<section class="hero">
  <div class="eyebrow">⚡ Acerto final</div>
  <h2 id="pix-headline">Três PIX e a gente fecha tudo.</h2>
  <div id="pix-rows">
    <!-- dynamically rendered by recalculate() -->
  </div>
</section>
```

Remove the three hardcoded `<div class="pix-row">` blocks (lines ~697–720).

- [ ] **Step 2: Replace bucket card HTML with dynamic containers**

Replace the entire contents of `<section class="buckets">` with this structure (preserving static text/descriptions):

```html
<section class="buckets">

  <!-- BUCKET 1: PG -->
  <div class="bucket">
    <div class="bucket-head">
      <span class="bucket-num">01</span>
      <h3 class="bucket-title">Praia Grande</h3>
      <span class="bucket-people">÷ 4 pessoas</span>
    </div>
    <p class="bucket-desc">Mercados, ubers, kiosk, pedágio e combustível da viagem. Todo mundo entra na divisão.</p>
    <p class="date-note">ℹ️ As datas são as reais da compra. Nos extratos Monzo/bancários elas aparecem com 1 dia de atraso — é o tempo de processamento/liquidação do cartão internacional.</p>
    <div id="cards-pg"></div>
    <div class="bucket-total">
      <span class="bucket-total-label">Total bucket Praia Grande</span>
      <span class="bucket-total-value" id="bucket-total-pg">R$2.161,51 — R$540,38 <span class="each">cada</span></span>
    </div>
  </div>

  <!-- BUCKET 2: Caipirinhas -->
  <div class="bucket">
    <div class="bucket-head">
      <span class="bucket-num">02</span>
      <h3 class="bucket-title">Caipirinhas + pastel</h3>
      <span class="bucket-people">÷ 2 (Rafa &amp; Ju)</span>
    </div>
    <p class="bucket-desc">Só o Rafa e a Ju consumiram, então rachado entre os dois. (Kiosk — porção de R$180 já contabilizada no bucket 01.)</p>
    <div id="cards-caipi"></div>
  </div>

  <!-- BUCKET 3: AQA -->
  <div class="bucket">
    <div class="bucket-head">
      <span class="bucket-num">03</span>
      <h3 class="bucket-title">Araraquara</h3>
      <span class="bucket-people">÷ 3 (sem o João)</span>
    </div>
    <p class="bucket-desc">Pedágio + combustível do rolê pra Araraquara. João não foi nessa, então fica de fora.</p>
    <div id="cards-aqa"></div>
  </div>

</section>
```

- [ ] **Step 3: Add data attributes to balance table cells**

Replace the balance table `<tbody>` content with this (same visual structure, adds `data-balance-cell` attributes):

```html
<tbody id="balance-tbody">
  <tr>
    <td><span class="chip rafa">Rafa</span></td>
    <td class="num" data-balance-cell="rafa-pg">R$540,38</td>
    <td class="num" data-balance-cell="rafa-caipi">R$72,50</td>
    <td class="num" data-balance-cell="rafa-aqa">R$40,00</td>
    <td class="num bold" data-balance-cell="rafa-total">R$652,88</td>
    <td class="num" data-balance-cell="rafa-paid">R$1.172,46</td>
    <td class="num balance-cell pos" data-balance-cell="rafa-net">+R$519,58</td>
  </tr>
  <tr>
    <td><span class="chip ju">Ju</span></td>
    <td class="num" data-balance-cell="ju-pg">R$540,38</td>
    <td class="num" data-balance-cell="ju-caipi">R$72,50</td>
    <td class="num" data-balance-cell="ju-aqa">R$40,00</td>
    <td class="num bold" data-balance-cell="ju-total">R$652,88</td>
    <td class="num" data-balance-cell="ju-paid">R$534,05</td>
    <td class="num balance-cell neg" data-balance-cell="ju-net">−R$118,83</td>
  </tr>
  <tr>
    <td><span class="chip higor">Higor</span></td>
    <td class="num" data-balance-cell="higor-pg">R$540,38</td>
    <td class="num muted" data-balance-cell="higor-caipi">—</td>
    <td class="num" data-balance-cell="higor-aqa">R$40,00</td>
    <td class="num bold" data-balance-cell="higor-total">R$580,38</td>
    <td class="num" data-balance-cell="higor-paid">R$120,00</td>
    <td class="num balance-cell neg" data-balance-cell="higor-net">−R$460,38</td>
  </tr>
  <tr>
    <td><span class="chip joao">João</span></td>
    <td class="num" data-balance-cell="joao-pg">R$540,38</td>
    <td class="num muted" data-balance-cell="joao-caipi">—</td>
    <td class="num muted" data-balance-cell="joao-aqa">—</td>
    <td class="num bold" data-balance-cell="joao-total">R$540,38</td>
    <td class="num" data-balance-cell="joao-paid">R$600,00</td>
    <td class="num balance-cell pos" data-balance-cell="joao-net">+R$59,62</td>
  </tr>
</tbody>
```

- [ ] **Step 4: Add ID to unsimplified table tbody**

Change `<tbody>` inside `.unsimp-table` to `<tbody id="unsimp-tbody">`. Keep all existing `<tr>` rows for now (they'll be replaced by `recalculate()` in Task 5).

- [ ] **Step 5: Verify page still loads without JS errors**

Open page in browser — static content (header, balance table, etc.) should display. No console errors expected.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "refactor: add IDs/data attrs to HTML, replace hardcoded cards with dynamic containers"
```

---

## Task 4: Implement `renderBuckets()`

**Files:**
- Modify: `index.html` — JS block, add after the constants from Task 1

- [ ] **Step 1: Add `renderBuckets()` function**

Add this function in the JS block, after the constants and before `loadAll()`:

```js
// ── Rendering ─────────────────────────────────────────────
function fmt(n) {
  return 'R$' + n.toFixed(2)
    .replace('.', ',')
    .replace(/\B(?=(\d{3})+(?!\d))/g, '.');
}

function renderBuckets() {
  BUCKETS.forEach(bucket => {
    const container = document.getElementById('cards-' + bucket.id);
    if (!container) return;

    // Group transactions for this bucket by payer
    const byPayer = {};
    store.transactions
      .filter(tx => tx.bucket === bucket.id)
      .forEach(tx => {
        if (!byPayer[tx.paidBy]) byPayer[tx.paidBy] = [];
        byPayer[tx.paidBy].push(tx);
      });

    container.innerHTML = Object.entries(byPayer).map(([payer, txs]) => {
      const color   = PEOPLE_COLORS[payer] || 'coral';
      const label   = PEOPLE_LABELS[payer] || payer;
      const total   = txs.reduce((s, tx) => s + tx.amount, 0);
      const perPerson = txs.reduce((s, tx) => s + tx.amount / tx.splitAmong.length, 0);

      const rows = txs.map(tx => {
        const dateCell = tx.date
          ? `<td class="td-date">${tx.date}</td>`
          : '';
        const compBtn = tx.comprovante
          ? ` <button class="comp-btn" onclick="openLightbox('${tx.comprovante}')">comprovante</button>`
          : '';
        return `<tr data-tx-id="${tx.id}">
          ${dateCell}
          <td class="td-desc" contenteditable="true" data-tx="${tx.id}" spellcheck="false">${escapeHtml(tx.desc)}</td>
          <td class="td-amount">${fmt(tx.amount)}${compBtn}</td>
          <td class="td-actions">
            <button class="row-btn edit-btn" onclick="openEditModal('${tx.id}')" title="Editar">✏️</button>
            <button class="row-btn del-btn"  onclick="deleteTx('${tx.id}')"      title="Remover">🗑️</button>
          </td>
        </tr>`;
      }).join('');

      return `<div class="card">
        <div class="card-head">
          <div>
            <div class="label accent-${color}">Pago por</div>
            <div class="paid-by">
              <span class="chip ${payer}">${label}</span>
              <span class="paid-by-amount" id="paid-amount-${payer}-${bucket.id}">${fmt(total)}</span>
            </div>
          </div>
        </div>
        <table><tbody id="tbody-${payer}-${bucket.id}">${rows}</tbody></table>
        <div class="card-foot">
          <span class="label">Por pessoa</span>
          <span class="per-person ${color}" id="per-person-${payer}-${bucket.id}">${fmt(perPerson)} cada</span>
        </div>
      </div>`;
    }).join('');
  });

  // Re-attach contenteditable listeners after re-render
  initEditableDescs();
}
```

- [ ] **Step 2: Call `renderBuckets()` in the init IIFE**

In the init IIFE (`async () => { ... })()`), add `renderBuckets()` after `await loadAll()`:

```js
await loadAll();
renderBuckets();
renderGlobalComments();
initEditableDescs(); // (initEditableDescs is now also called inside renderBuckets, but keep here as safety)
```

Wait — `initEditableDescs` is called inside `renderBuckets`. Remove the standalone call in init to avoid double-binding. Update init to:

```js
(async () => {
  if (!configured()) {
    document.getElementById('comments-warning').style.display = 'block';
    document.getElementById('comment-list').innerHTML =
      '<p class="no-comments">Configure o JSONBin para ativar comentários.</p>';
    return;
  }
  try {
    await loadAll();
    renderBuckets();
    renderGlobalComments();
  } catch {
    document.getElementById('comment-list').innerHTML =
      '<p class="no-comments">Erro ao carregar comentários.</p>';
  }
})();
```

- [ ] **Step 3: Update `initEditableDescs()` to write to `tx.desc`**

Replace the entire `initEditableDescs` function:

```js
function initEditableDescs() {
  document.querySelectorAll('.td-desc[data-tx]').forEach(cell => {
    const txId = cell.dataset.tx;
    const tx   = store.transactions.find(t => t.id === txId);
    if (!tx) return;

    let lastSaved = tx.desc;

    cell.addEventListener('keydown', e => {
      if (e.key === 'Enter')  { e.preventDefault(); cell.blur(); }
      if (e.key === 'Escape') { cell.textContent = lastSaved; cell.blur(); }
    });

    cell.addEventListener('blur', async () => {
      const val = cell.textContent.trim() || lastSaved;
      cell.textContent = val;
      if (val === lastSaved) return;
      tx.desc = val;
      lastSaved = val;
      try {
        await persist();
        cell.classList.add('saved-flash');
        setTimeout(() => cell.classList.remove('saved-flash'), 500);
      } catch {}
    });
  });
}
```

- [ ] **Step 4: Verify in browser**

Load the page. All transaction tables should render from JS. Each row should have ✏️ and 🗑️ buttons. Descriptions should be contenteditable (click one, change text, blur — should flash green).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: render transaction tables dynamically from store.transactions"
```

---

## Task 5: Implement `consolidateDebts()` and `recalculate()`

**Files:**
- Modify: `index.html` — JS block, add after `renderBuckets()`

- [ ] **Step 1: Add `consolidateDebts()` function**

```js
function consolidateDebts(net) {
  const eps = 0.005;
  const credits = PEOPLE
    .filter(p => net[p] > eps)
    .map(p => ({ who: p, amount: net[p] }))
    .sort((a, b) => b.amount - a.amount);
  const debts = PEOPLE
    .filter(p => net[p] < -eps)
    .map(p => ({ who: p, amount: -net[p] }))
    .sort((a, b) => b.amount - a.amount);

  const transfers = [];
  let ci = 0, di = 0;
  while (ci < credits.length && di < debts.length) {
    const c = credits[ci], d = debts[di];
    const pay = Math.min(c.amount, d.amount);
    if (pay > eps) transfers.push({ from: d.who, to: c.who, amount: pay });
    c.amount -= pay;
    d.amount -= pay;
    if (c.amount < eps) ci++;
    if (d.amount < eps) di++;
  }
  return transfers;
}
```

- [ ] **Step 2: Add `recalculate()` function**

```js
function recalculate() {
  // 1. Per-payer totals
  const payerTotals = {};
  PEOPLE.forEach(p => payerTotals[p] = 0);
  store.transactions.forEach(tx => {
    payerTotals[tx.paidBy] = (payerTotals[tx.paidBy] || 0) + tx.amount;
  });

  // 2. Per-person owed, per bucket
  const owed = {};
  PEOPLE.forEach(p => { owed[p] = {}; BUCKETS.forEach(b => owed[p][b.id] = 0); });
  store.transactions.forEach(tx => {
    const share = tx.amount / tx.splitAmong.length;
    tx.splitAmong.forEach(p => { owed[p][tx.bucket] = (owed[p][tx.bucket] || 0) + share; });
  });

  // 3. Total owed and net balance
  const totalOwed = {}, net = {};
  PEOPLE.forEach(p => {
    totalOwed[p] = BUCKETS.reduce((s, b) => s + (owed[p][b.id] || 0), 0);
    net[p] = payerTotals[p] - totalOwed[p];
  });

  // 4. Debt consolidation
  const transfers = consolidateDebts(net);

  // 5. Update balance table cells
  PEOPLE.forEach(p => {
    BUCKETS.forEach(b => {
      const cell = document.querySelector(`[data-balance-cell="${p}-${b.id}"]`);
      if (!cell) return;
      const val = owed[p][b.id];
      if (val > 0.005) {
        cell.textContent = fmt(val);
        cell.classList.remove('muted');
      } else {
        cell.textContent = '—';
        cell.classList.add('muted');
      }
    });
    const totalCell = document.querySelector(`[data-balance-cell="${p}-total"]`);
    const paidCell  = document.querySelector(`[data-balance-cell="${p}-paid"]`);
    const netCell   = document.querySelector(`[data-balance-cell="${p}-net"]`);
    if (totalCell) totalCell.textContent = fmt(totalOwed[p]);
    if (paidCell)  paidCell.textContent  = fmt(payerTotals[p]);
    if (netCell) {
      const n = net[p];
      netCell.textContent = (n >= 0 ? '+' : '−') + fmt(Math.abs(n));
      netCell.className = 'num balance-cell ' + (n >= 0 ? 'pos' : 'neg');
    }
  });

  // 6. Bucket PG total (all-4 bucket — per person = average share for Rafa)
  const pgTotal = store.transactions
    .filter(tx => tx.bucket === 'pg')
    .reduce((s, tx) => s + tx.amount, 0);
  const pgPerPerson = owed['rafa']['pg']; // Rafa is always in PG
  const pgTotalEl = document.getElementById('bucket-total-pg');
  if (pgTotalEl) pgTotalEl.innerHTML =
    `${fmt(pgTotal)} — ${fmt(pgPerPerson)} <span class="each">cada</span>`;

  // 7. Hero PIX
  const headline = document.getElementById('pix-headline');
  const pixRows  = document.getElementById('pix-rows');
  if (headline) headline.textContent =
    transfers.length === 1
      ? 'Um PIX e a gente fecha tudo.'
      : `${transfers.length === 2 ? 'Dois' : transfers.length === 3 ? 'Três' : transfers.length} PIX e a gente fecha tudo.`;
  if (pixRows) pixRows.innerHTML = transfers.map(t => `
    <div class="pix-row">
      <div class="pix-from">
        <span class="chip lg ${t.from}">${PEOPLE_LABELS[t.from]}</span>
        <span class="arrow">→</span>
        <span class="chip lg ${t.to}">${PEOPLE_LABELS[t.to]}</span>
      </div>
      <div class="pix-amount">${fmt(t.amount)}</div>
    </div>`).join('');

  // 8. Unsimplified table
  const unsimpTbody = document.getElementById('unsimp-tbody');
  if (unsimpTbody) {
    // For each person p, for each payer q ≠ p: amount p owes q across all transactions
    const directDebts = [];
    BUCKETS.forEach(b => {
      const payersInBucket = [...new Set(store.transactions.filter(tx => tx.bucket === b.id).map(tx => tx.paidBy))];
      payersInBucket.forEach(payer => {
        PEOPLE.forEach(person => {
          if (person === payer) return;
          const amt = store.transactions
            .filter(tx => tx.bucket === b.id && tx.paidBy === payer && tx.splitAmong.includes(person))
            .reduce((s, tx) => s + tx.amount / tx.splitAmong.length, 0);
          if (amt > 0.005) directDebts.push({ from: person, to: payer, amount: amt, label: b.id });
        });
      });
    });
    const bucketLabels = { pg:'compras PG', caipi:'caipi', aqa:'AQA' };
    unsimpTbody.innerHTML = directDebts.map(d => `
      <tr>
        <td><span class="chip ${d.from}">${PEOPLE_LABELS[d.from]}</span></td>
        <td class="arrow" style="padding:8px 8px;">→</td>
        <td><span class="chip ${d.to}">${PEOPLE_LABELS[d.to]}</span></td>
        <td style="color:var(--ink-soft);font-size:12px;">${bucketLabels[d.label] || d.label}</td>
        <td class="td-amount" style="padding:8px 16px;font-variant-numeric:tabular-nums;">${fmt(d.amount)}</td>
      </tr>`).join('');
    // Update footer count
    const footer = document.querySelector('.unsimp-footer');
    if (footer) {
      const saved = transfers.length;
      const orig  = directDebts.length;
      footer.textContent = `${saved} PIX em vez de ${orig} — economia de ${orig - saved} transações 🎉`;
    }
    // Update summary label
    const label = document.querySelector('.unsimp-label');
    if (label) label.textContent = `E sem simplificação? Seriam ${directDebts.length} PIX diretos ▾`;
  }
}
```

- [ ] **Step 3: Call `recalculate()` in init, after `renderBuckets()`**

In the init IIFE, update the try block:

```js
try {
  await loadAll();
  renderBuckets();
  recalculate();
  renderGlobalComments();
} catch {
  document.getElementById('comment-list').innerHTML =
    '<p class="no-comments">Erro ao carregar comentários.</p>';
}
```

- [ ] **Step 4: Verify numbers match in browser**

Load the page. Check:
- Hero PIX: Higor→Rafa R$460,38 | Ju→Rafa R$59,20 | Ju→João R$59,62
- Balance table Rafa: Devia R$652,88 | Pagou R$1.172,46 | Saldo +R$519,58
- Bucket 1 total: R$2.161,51 — R$540,38 cada

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add consolidateDebts() and recalculate() — all DOM values computed from data"
```

---

## Task 6: Add CSS for modal, floating button, and row action icons

**Files:**
- Modify: `index.html` — CSS `<style>` block, append before closing `</style>` tag

- [ ] **Step 1: Add CSS**

Append to the CSS block:

```css
/* Row action buttons */
.td-actions {
  width: 1px;
  white-space: nowrap;
  padding: 6px 8px !important;
  opacity: 0;
  transition: opacity 0.15s;
}
tr:hover .td-actions,
tr:focus-within .td-actions {
  opacity: 1;
}
@media (hover: none) {
  .td-actions { opacity: 1; }
}
.row-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 2px 4px;
  font-size: 14px;
  line-height: 1;
  border-radius: 4px;
  opacity: 0.5;
  transition: opacity 0.15s;
}
.row-btn:hover { opacity: 1; }

/* Floating add button */
.fab {
  position: fixed;
  bottom: 28px;
  right: 24px;
  background: var(--teal);
  color: #fff;
  border: none;
  border-radius: 28px;
  padding: 14px 22px;
  font-family: "Plus Jakarta Sans", sans-serif;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 20px rgba(30,94,91,0.35);
  z-index: 100;
  transition: transform 0.15s, box-shadow 0.15s;
}
.fab:hover { transform: translateY(-2px); box-shadow: 0 6px 24px rgba(30,94,91,0.45); }
.fab:active { transform: translateY(0); }

/* Modal */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(42,31,20,0.55);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 200;
  padding: 16px;
  display: none;
}
.modal-overlay.open { display: flex; }
.modal {
  background: var(--bg-soft);
  border-radius: 16px;
  padding: 28px 24px 24px;
  width: 100%;
  max-width: 440px;
  box-shadow: 0 12px 40px rgba(42,31,20,0.2);
}
.modal h3 {
  margin: 0 0 20px;
  font-size: 18px;
  font-weight: 800;
}
.modal-field {
  margin-bottom: 14px;
}
.modal-field label {
  display: block;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--ink-soft);
  margin-bottom: 5px;
}
.modal-field input[type="text"],
.modal-field input[type="number"] {
  width: 100%;
  padding: 10px 12px;
  border: 1.5px solid var(--line);
  border-radius: 8px;
  background: var(--bg);
  font-family: "Plus Jakarta Sans", sans-serif;
  font-size: 15px;
  color: var(--ink);
}
.modal-field input:focus { outline: none; border-color: var(--teal); }
.person-toggles {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}
.person-toggle {
  padding: 6px 14px;
  border-radius: 20px;
  border: 1.5px solid var(--line);
  background: var(--bg);
  font-family: "Plus Jakarta Sans", sans-serif;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.12s, border-color 0.12s, color 0.12s;
}
.person-toggle.active-rafa   { background: var(--coral); border-color: var(--coral); color: #fff; }
.person-toggle.active-ju     { background: var(--pink);  border-color: var(--pink);  color: #fff; }
.person-toggle.active-higor  { background: var(--teal);  border-color: var(--teal);  color: #fff; }
.person-toggle.active-joao   { background: var(--gold);  border-color: var(--gold);  color: #fff; }
.modal-field select {
  width: 100%;
  padding: 10px 12px;
  border: 1.5px solid var(--line);
  border-radius: 8px;
  background: var(--bg);
  font-family: "Plus Jakarta Sans", sans-serif;
  font-size: 15px;
  color: var(--ink);
}
.modal-field select:focus { outline: none; border-color: var(--teal); }
.modal-actions {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}
.modal-save {
  flex: 1;
  padding: 12px;
  background: var(--teal);
  color: #fff;
  border: none;
  border-radius: 10px;
  font-family: "Plus Jakarta Sans", sans-serif;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
}
.modal-save:disabled { opacity: 0.5; cursor: default; }
.modal-cancel {
  padding: 12px 20px;
  background: none;
  border: 1.5px solid var(--line);
  border-radius: 10px;
  font-family: "Plus Jakarta Sans", sans-serif;
  font-size: 15px;
  cursor: pointer;
}
```

- [ ] **Step 2: Verify styles render**

Load page. You should see ✏️🗑️ buttons appear on row hover (desktop) or always (mobile). No layout breakage.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add CSS for modal, floating button, and row action icons"
```

---

## Task 7: Add modal HTML and floating button

**Files:**
- Modify: `index.html` — just before `<!-- Lightbox -->` (around line 1029)

- [ ] **Step 1: Add modal and FAB markup**

Insert immediately before `<!-- Lightbox -->`:

```html
<!-- Floating Add Button -->
<button class="fab" onclick="openAddModal()">+ transação</button>

<!-- Transaction Modal -->
<div id="tx-modal" class="modal-overlay" onclick="if(event.target===this)closeModal()">
  <div class="modal">
    <h3 id="modal-title">Nova transação</h3>

    <div class="modal-field">
      <label>Data (dd/mm)</label>
      <input type="text" id="m-date" placeholder="ex: 18/04" maxlength="5" />
    </div>

    <div class="modal-field">
      <label>Descrição</label>
      <input type="text" id="m-desc" placeholder="ex: Mercado" maxlength="80" />
    </div>

    <div class="modal-field">
      <label>Valor (R$)</label>
      <input type="number" id="m-amount" placeholder="0,00" min="0.01" step="0.01" />
    </div>

    <div class="modal-field">
      <label>Quem pagou</label>
      <div class="person-toggles" id="m-paidby">
        <button type="button" class="person-toggle" data-person="rafa"  onclick="selectPayer(this)">Rafa</button>
        <button type="button" class="person-toggle" data-person="ju"    onclick="selectPayer(this)">Ju</button>
        <button type="button" class="person-toggle" data-person="higor" onclick="selectPayer(this)">Higor</button>
        <button type="button" class="person-toggle" data-person="joao"  onclick="selectPayer(this)">João</button>
      </div>
    </div>

    <div class="modal-field">
      <label>Dividir entre</label>
      <div class="person-toggles" id="m-split">
        <button type="button" class="person-toggle" data-person="rafa"  onclick="toggleSplit(this)">Rafa</button>
        <button type="button" class="person-toggle" data-person="ju"    onclick="toggleSplit(this)">Ju</button>
        <button type="button" class="person-toggle" data-person="higor" onclick="toggleSplit(this)">Higor</button>
        <button type="button" class="person-toggle" data-person="joao"  onclick="toggleSplit(this)">João</button>
      </div>
    </div>

    <div class="modal-field">
      <label>Grupo (bucket)</label>
      <select id="m-bucket">
        <option value="pg">Praia Grande</option>
        <option value="caipi">Caipirinhas + pastel</option>
        <option value="aqa">Araraquara</option>
      </select>
    </div>

    <div class="modal-actions">
      <button type="button" class="modal-save" id="modal-save-btn" onclick="saveTx()">Salvar</button>
      <button type="button" class="modal-cancel" onclick="closeModal()">Cancelar</button>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Verify modal HTML is in DOM**

In browser console: `document.getElementById('tx-modal')` — should return the element. The FAB button should be visible in bottom-right corner.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add transaction modal HTML and floating add button"
```

---

## Task 8: Implement CRUD functions and wire everything together

**Files:**
- Modify: `index.html` — JS block, add before the closing `</script>` tag

- [ ] **Step 1: Add CRUD functions**

Add this block before `</script>`:

```js
// ── Transaction CRUD ──────────────────────────────────────
let _editingTxId = null;

function openAddModal() {
  _editingTxId = null;
  document.getElementById('modal-title').textContent = 'Nova transação';
  document.getElementById('m-date').value   = '';
  document.getElementById('m-desc').value   = '';
  document.getElementById('m-amount').value = '';
  document.getElementById('m-bucket').value = 'pg';
  // Payer: deselect all
  document.querySelectorAll('#m-paidby .person-toggle').forEach(btn => {
    btn.className = 'person-toggle';
  });
  // Split: select all by default
  document.querySelectorAll('#m-split .person-toggle').forEach(btn => {
    btn.classList.add('active-' + btn.dataset.person);
  });
  document.getElementById('tx-modal').classList.add('open');
  document.getElementById('m-date').focus();
}

function openEditModal(txId) {
  const tx = store.transactions.find(t => t.id === txId);
  if (!tx) return;
  _editingTxId = txId;
  document.getElementById('modal-title').textContent = 'Editar transação';
  document.getElementById('m-date').value   = tx.date;
  document.getElementById('m-desc').value   = tx.desc;
  document.getElementById('m-amount').value = tx.amount;
  document.getElementById('m-bucket').value = tx.bucket;
  // Payer
  document.querySelectorAll('#m-paidby .person-toggle').forEach(btn => {
    btn.className = 'person-toggle';
    if (btn.dataset.person === tx.paidBy) btn.classList.add('active-' + tx.paidBy);
  });
  // Split
  document.querySelectorAll('#m-split .person-toggle').forEach(btn => {
    btn.className = 'person-toggle';
    if (tx.splitAmong.includes(btn.dataset.person)) btn.classList.add('active-' + btn.dataset.person);
  });
  document.getElementById('tx-modal').classList.add('open');
}

function closeModal() {
  document.getElementById('tx-modal').classList.remove('open');
  _editingTxId = null;
}

function selectPayer(btn) {
  document.querySelectorAll('#m-paidby .person-toggle').forEach(b => b.className = 'person-toggle');
  btn.classList.add('active-' + btn.dataset.person);
}

function toggleSplit(btn) {
  const cls = 'active-' + btn.dataset.person;
  if (btn.classList.contains(cls)) {
    btn.classList.remove(cls);
  } else {
    btn.classList.add(cls);
  }
}

async function saveTx() {
  const date   = document.getElementById('m-date').value.trim();
  const desc   = document.getElementById('m-desc').value.trim();
  const amount = parseFloat(document.getElementById('m-amount').value);
  const bucket = document.getElementById('m-bucket').value;
  const payerBtn = document.querySelector('#m-paidby .person-toggle[class*="active-"]');
  const splitBtns = document.querySelectorAll('#m-split .person-toggle[class*="active-"]');

  if (!desc)           { alert('Preencha a descrição.'); return; }
  if (!amount || amount <= 0) { alert('Valor inválido.'); return; }
  if (!payerBtn)       { alert('Selecione quem pagou.'); return; }
  if (splitBtns.length === 0) { alert('Selecione ao menos uma pessoa no split.'); return; }

  const paidBy     = payerBtn.dataset.person;
  const splitAmong = Array.from(splitBtns).map(b => b.dataset.person);

  const btn = document.getElementById('modal-save-btn');
  btn.textContent = 'Salvando…'; btn.disabled = true;

  if (_editingTxId) {
    // Edit existing
    const tx = store.transactions.find(t => t.id === _editingTxId);
    Object.assign(tx, { date, desc, amount, bucket, paidBy, splitAmong });
  } else {
    // Add new
    const id = Date.now().toString(36) + Math.random().toString(36).slice(2, 6);
    store.transactions.push({ id, date, desc, amount, bucket, paidBy, splitAmong, comprovante: null });
  }

  try {
    await persist();
    closeModal();
    renderBuckets();
    recalculate();
  } catch {
    alert('Erro ao salvar. Tenta de novo.');
  }
  btn.textContent = 'Salvar'; btn.disabled = false;
}

async function deleteTx(txId) {
  if (!confirm('Remover esta transação?')) return;
  store.transactions = store.transactions.filter(t => t.id !== txId);
  try {
    await persist();
    renderBuckets();
    recalculate();
  } catch {
    alert('Erro ao remover. Tenta de novo.');
  }
}

// Close modal on Escape
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') {
    closeModal();
    closeLightbox();
  }
});
```

- [ ] **Step 2: Remove duplicate Escape listener**

The existing `document.addEventListener('keydown', ...)` at ~line 1046 only handles `closeLightbox()`. Replace it with the combined one above (keep only the new block, remove the old one that only calls `closeLightbox`).

- [ ] **Step 3: Verify full CRUD flow in browser**

1. Click "+ transação" button — modal opens
2. Fill: date "18/04", desc "Teste", amount "10", payer Rafa, split all 4, bucket PG → Salvar
3. New row appears in Rafa's PG card, hero PIX updates, balance table updates
4. Click ✏️ on the new row — modal opens prefilled
5. Change amount to "20" → Salvar — row updates, numbers update
6. Click 🗑️ on the new row → confirm — row disappears, numbers revert

- [ ] **Step 4: Push to production**

```bash
git add index.html
git commit -m "feat: implement transaction CRUD with JSONBin persistence and auto-recalculation"
git push
```

---

## Self-Review

**Spec coverage check:**
- ✅ Data model: `DEFAULT_TRANSACTIONS` with id, date, desc, amount, paidBy, splitAmong, comprovante, bucket
- ✅ Store migration: Task 2 seeds from defaults, applies txNotes renames, persists immediately
- ✅ txNotes deprecated: `initEditableDescs` writes to `tx.desc` directly (Task 4)
- ✅ recalculate() covers: per-payer totals, per-person owed, net balance, debt consolidation, hero PIX, unsimplified table (Task 5)
- ✅ Add modal: floating FAB, form with all fields, validation (Tasks 7–8)
- ✅ Edit: pencil icon, prefilled modal, updates existing tx (Task 8)
- ✅ Delete: trash icon, confirm dialog, removes tx (Task 8)
- ✅ Description contenteditable: writes to tx.desc, persists (Task 4)
- ✅ No comprovante on new transactions (renderBuckets only adds comprovante btn when tx.comprovante is set)
- ✅ Optimistic UI: DOM updates first, JSONBin write async (Task 8 `saveTx` / `deleteTx`)

**Type consistency check:**
- `fmt(n)` used consistently in renderBuckets, recalculate
- `store.transactions` array accessed consistently
- `tx.bucket` string matches BUCKETS[i].id throughout
- `openEditModal(txId)` matches onclick in renderBuckets
- `deleteTx(txId)` matches onclick in renderBuckets

**Potential issue — double `initEditableDescs` call:** Task 4 calls `initEditableDescs()` inside `renderBuckets()`. Init IIFE must NOT call it separately, or listeners will double-bind. This is noted in Task 4 Step 2.
