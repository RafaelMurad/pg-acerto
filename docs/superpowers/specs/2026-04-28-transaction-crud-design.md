# Transaction CRUD — Design Spec
**Date:** 2026-04-28  
**Project:** pg-acerto (static GitHub Pages trip expense settlement)

---

## Overview

Add the ability to add, edit, and delete transactions directly in the page, with all totals, balances, and PIX amounts recalculating automatically. Persistence via the existing JSONBin integration. No new external services.

---

## Data Model

Each transaction is a JS object:

```js
{
  id: string,              // unique — snake_case for seeded rows, Date.now().toString(36) for new
  date: string,            // "dd/mm"
  desc: string,            // display description
  amount: number,          // BRL as float (e.g. 231.00)
  paidBy: string,          // "rafa" | "ju" | "higor" | "joao"
  splitAmong: string[],    // subset of the 4 people
  comprovante: string|null // relative path, null for user-added rows
}
```

JSONBin store schema becomes:
```js
{ comments: [], txNotes: {}, transactions: [] }
```

### Seeding / Migration

- A `DEFAULT_TRANSACTIONS` constant is baked into the HTML containing the current 22 rows as JS objects.
- On `loadAll()`: if `data.record.transactions` is empty or absent, fall back to `DEFAULT_TRANSACTIONS` (applying any saved `txNotes` renames to `desc`).
- After seeding from defaults, `store.transactions` is immediately persisted so future loads skip the seed.
- `txNotes` becomes unused going forward; description edits write directly to `tx.desc`.

---

## Recalculation

A single `recalculate()` function runs after every CRUD operation and derives all displayed values from `store.transactions`:

1. **Per-payer totals** — sum of `amount` for each payer → updates each payer table footer.
2. **Per-person owed** — for each transaction, each person in `splitAmong` owes `amount / splitAmong.length` → sum per person.
3. **Bucket total / per-person line** — sum all transaction amounts / 4 (for the single bucket).
4. **Net balance** — `paid - owed` per person → updates balance table (devia, pagou, saldo).
5. **Debt consolidation** — greedy algorithm: compute net balances, match largest debtor to largest creditor, settle, repeat → produces minimal PIX list (currently 3).
6. **Hero PIX section** — updated from consolidation output.
7. **Unsimplified table** — updated from raw per-payer owed amounts.

---

## Rendering

All payer table `<tbody>` elements are emptied and rebuilt from JS on every render. Each row includes:
- Date, description (`contenteditable`), amount, comprovante button (if `comprovante` is set)
- Pencil icon → opens edit modal
- Trash icon → delete with `confirm()` dialog

The description `contenteditable` blur handler saves to `tx.desc` in the store and persists.

---

## CRUD UI

### Add (floating button)
- A **"+ transação"** button fixed at bottom-right of the page.
- Opens a modal overlay with:
  - Date field (`dd/mm`, prefilled with today's date)
  - Description text input
  - Amount number input (BRL)
  - "Quem pagou" — 4-button toggle (Rafa / Ju / Higor / João)
  - "Dividir entre" — 4 checkboxes (all checked by default)
  - **Salvar** button — validates (amount > 0, at least 1 person in splitAmong, desc non-empty), appends to `store.transactions`, persists, recalculates, closes modal
  - **Cancelar** button — closes modal, no changes

### Edit
- Pencil icon on each row (visible on hover on desktop, always visible on mobile).
- Opens the same modal prefilled with the transaction's current data.
- Salvar overwrites the existing transaction in `store.transactions` by `id`, persists, recalculates.

### Delete
- Trash icon on each row.
- `confirm()` dialog: "Remover esta transação?"
- On confirm: removes from `store.transactions`, persists, recalculates.

### No comprovante upload
- New/edited transactions have no comprovante (no file hosting available).
- Comprovante button only renders for seeded rows where `comprovante` is set.

---

## Persistence

- Every write (add, edit, delete, description blur) calls the existing `persist()` function (JSONBin PUT).
- Optimistic UI: DOM updates immediately, JSONBin write happens in background; errors are silently swallowed (same as current behavior).

---

## Constraints

- Single HTML file — all JS and CSS inline.
- No build toolchain, no npm packages.
- No auth — anyone with the URL can modify transactions. Acceptable for a small private trip group.
- JSONBin free tier — bin size should stay well under limits given the small number of transactions.
