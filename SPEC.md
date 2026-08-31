# MAJESTIC POS — SPECIFICA COMPLETA (v2 FINAL)

## COS'È
Sistema di gestione per FCE Majestic, cocktail bar + cinema privato, Roma. Un solo locale.
1 PC Windows touch (server + app manager) + app camerieri Android + 2 display KDS (bar/cucina, tablet con Chrome).

## BRAND
- Logo: assets/logo.png (scritta corsiva oro + icona colonna). Icona colonna = app icon.
- Tema scuro: sfondo #0D0D0F, superfici #1A1A1E e blu navy #16233B, oro #D4A853,
  testo #FFFFFF / secondario #C9C4BC, verde ok #2e7d4f, arancio alert #e07b54, blu conto #3b82c4.
- Font Poppins. TUTTO IN ITALIANO. Bottoni grandi (min 48px), touch-first.

## ARCHITETTURA
- server/ : Node.js + Express + Socket.IO + better-sqlite3. Gira sul PC Windows.
- public/kds/ : display bar/cucina, pagine web semplici servite dal server, aperte a schermo intero.
- apps/waiter/ : Flutter (Android APK + Windows exe).
- apps/manager/ : Flutter (Windows, touch).
- Formato valuta: "€ 12,50".

## REGOLE DI BUSINESS (ESATTE)
1. 18 tavoli: 9 area SALA + 9 area BAR + vendita al banco (ordine type='COUNTER', senza tavolo).
2. Stati tavolo: LIBERO → APERTO → (CONTO opzionale) → LIBERO. Chiudere libera il tavolo.
3. APERTURA TAVOLO (v2): obbligatorio inserire { clientsNormal, clientsCinemaPlus, waiterName }.
   Salvare clients_normal e clients_cinema_plus sul tavolo. UI: dialog con +/- grandi per
   NORMALE e CINEMA PLUS, totale clienti visibile, bottone APRI TAVOLO.
4. CLIENTI CINEMA PLUS: hanno già pagato il biglietto ONLINE. Non esiste nessun prodotto
   Cinema Plus nel menu, nessun €20. Sul conto valgono €0. NON calcolare due volte.
5. SNACK AUTOMATICO: prodotto nascosto 'CINEMA PLUS SNACK' (category CINEMA_PLUS, price 0,
   station CUCINA, visible=0). All'apertura con clientsCinemaPlus > 0 il server crea
   AUTOMATICAMENTE una riga d'ordine qty=clientsCinemaPlus, €0, la stampa in CUCINA e la
   mostra sul KDS CUCINA col nome del tavolo. Nessuna azione del cameriere. Sempre lo stesso snack.
6. DRINK INCLUSI (v2): prodotti is_included_drink=1: PROSECCO, S.PELLEGRINO 0.75, CRODINO,
   COCA COLA, COCA COLA ZERO, COCA COLA ZERO ZERO, FANTA, SPRITE, SUCCO ACE, SUCCO ANANAS,
   GALVANINA TE LIMONE, GALVANINA TE PESCA. Il cameriere sceglie il prodotto specifico.
   LIMITE: 1 drink incluso per ogni cliente Cinema Plus (allowance = clients_cinema_plus,
   contato su TUTTI gli ordini del tavolo, anche inviati dopo). Se allowance finito → il
   drink viene preso al PREZZO NORMALE del menu (included=0). MAI bloccare, mai errore.
   drink incluso = unit_price 0, included=1, scarica l'inventario normalmente.
7. CHIUSURA: un tavolo si può SEMPRE chiudere, anche con totale €0,00 (bug del vecchio app — testarlo).
8. BANCO: scegli prodotti → CONTANTI/CARTA → stampa ricevunta → fine. Nessun tavolo.
9. INVENTARIO: product_ingredients collega prodotti→materie prime con qty; scarico all'invio ordine.
   Vino: 1 calice = 0.2 bottiglia (5 calici per bottiglia). Spiriti ≈ 0.04, amari 0.06,
   birre/soft/acqua = 1 unità. PROSECCO incluso → Antoniazzi 0.2.
10. FORNITORI: ogni voce inventario ha fornitore opzionale + prezzo acquisto.
    Bottone "CREA ORDINE": per stock < minimo suggerisce qty = minimo − stock,
    raggruppa per fornitore, salva in purchase_orders (items JSON), marca ordered=1,
    e offre "Copia testo WhatsApp" (lista in italiano formattata).
11. IMPORT/EXPORT INVENTARIO (v2): ESPORTA → file .xlsx (npm 'xlsx') con colonne
    categoria | prodotto | stock | unit | minimo | fornitore | prezzo_acquisto.
    IMPORTA → legge .xlsx o CSV, matcha su "prodotto" (trim, case-insensitive), aggiorna
    stock/minimo/fornitore/prezzo; righe nuove vengono AGGIUNTE; risponde con
    {aggiornati, aggiunti, nonTrovati}. Stesso formato per fare andare e tornare.
12. CAMERIERI: nessun PIN. Lista nomi, tap per scegliere, si può aggiungere un nome nuovo.
    Ogni ordine salva waiter_name.
13. KDS: nuovo ticket → card dorata pulsante + suono di avviso ripetuto (Web Audio, nessun file audio);
    tap su APRI → suono fermo, card attenuata, badge IN CORSO, bottone diventa PRONTO ✓ (verde);
    PRONTO → evento "orderReady" all'app cameriere. 4 ticket per riga, layout landscape,
    filtra per stazione con ?station=BAR o ?station=CUCINA.
14. STAMPA: stampanti termiche di rete ESC/POS, TCP raw porta 9100. Una BAR una CUCINA.
    All'invio ordine: dividi le righe per stazione, stampa un biglietto per stazione.
    Testa: "MAJESTIC", nome tavolo, ora, righe con qty. Se la stampa fallisce l'ordine NON si blocca
    (log + evento printerError). Config IP/porta nelle impostazioni.
15. REAL-TIME Socket.IO: client→server: tableOpen, orderSend, requestBill, tableClose, itemState, counterSale.
    server→client: tablesUpdate, orderNew, orderReady, billUpdate, printerError, tablesSnapshot.

## DATI (schema SQLite)
waiters(id, name, active) · suppliers(id, name, contact) ·
products(id, name, category, price, station BAR|CUCINA, icon emoji, active, is_included_drink, visible) ·
inventory(id, name, stock, min_alert, unit, supplier_id, purchase_price, ordered) ·
product_ingredients(product_id, inventory_id, qty) ·
venue_tables(id, number, area SALA|BAR, state, waiter_name, opened_at, clients_normal, clients_cinema_plus) ·
orders(id, table_id NULL=banco, type TABLE|COUNTER, waiter_name, state APERTO|INVIATO|PAGATO, created_at) ·
order_items(id, order_id, product_id, name, qty, unit_price, included, note, state NUOVO|IN_PREPARAZIONE|PRONTO) ·
payments(id, order_id, amount, method CONTANTI|CARTA, created_at) ·
purchase_orders(id, supplier_id, items JSON, status, created_at)

## SEED
- Menu: importa fce_club_menu.csv (colonne category, item, price_eur) verbatim.
  Stazioni: tutto BAR tranne FOOD MENU → CUCINA. Icone emoji per categoria
  (SPRITZ 🍹, CLASSIC 🍸, GIN 🥃, VINO 🍷, BEER 🍺, BEVANDE 🥤, CAFFETTERIA ☕, FOOD 🍽, AMARI 🥃, CHAMPAGNE 🥂, EXTRA ➕).
  Aggiungi PROSECCO (category BEVANDE, price 6, BAR, is_included_drink=1) se manca.
- Inventario: importa inventory-seed.csv (colonne categoria, prodotto, giacenza, unita, minimo)
  + aggiungi: Crodino (Bevande analcoliche, stock 24, unità, min 12), Caffè (kg), Latte,
  Ginseng, Orzo, Tagliere ingredienti, Hummus, Tiramisù.
- Collegamenti ingredienti: collega quelli ovvi (spritz → aperol+prosecco ecc., gin tonic →
  gin 0.04 + tonica 1, birre 1 unità, calici vino 0.2 della bottiglia corrispondente,
  soft 1 unità, caffetteria dalle nuove voci). I restanti restano senza link, modificabili dal Manager.
- 18 tavoli, cameriere "Marco", prodotto nascosto CINEMA PLUS SNACK.

## TEST OBBLIGATORI (server/test-workflow.js — tutti ✅ richiesti)
1. Apri tavolo: 2 normali + 3 CP → verificati clients salvati + ordine automatico
   3× CINEMA PLUS SNACK su CUCINA a €0.
2. 3 drink inclusi (PROSECCO, CRODINO, COCA COLA) → tutti €0, included=1, stock scaricato.
3. 4° drink incluso (SPRITE) → prezzo NORMALE (5), included=0, NESSUN blocco.
4. Conto = solo 5 → chiudi → tavolo LIBERO.
5. Tavolo con SOLO snack + drink inclusi → totale €0 → chiusura RIUSCISCE.
6. Round-trip inventario: export → modifica stock → import → verificato.
