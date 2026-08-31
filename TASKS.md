# ISTRUZIONI PER CODEX — leggi SPEC.md prima di tutto. Lavora UNA FASE ALLA VOLTA.
# Non iniziare la fase successiva finché la fase corrente non è completata e verificata.

## FASE 1 — SERVER
Crea in server/: schema.sql, db.js (better-sqlite3, WAL), business.js (includedDrinksAvailable,
openTable con clientsNormal/clientsCinemaPlus, auto CINEMA PLUS SNACK a CUCINA, addItems con
logica drink inclusi della SPEC §6 — mai bloccare, prezzo normale se allowance finito,
getBill, closeTable che funziona anche a €0, counterSale, CRUD camerieri/fornitori/prodotti/
inventario + restock + createPurchaseOrder), server.js (Express + REST /api completo + Socket.IO
con tutti gli eventi della SPEC §15 + report: incasso per metodo, top prodotti, vendite per
cameriere, cinema plus), printer.js (ESC/POS TCP 9100, non bloccante), seed.js (SPEC SEED),
settings.js (IP stampanti), inventory import/export (.xlsx via npm 'xlsx', SPEC §11).
Poi: `npm install`, lancia `node test-workflow.js` e CORREGGI finché TUTTI i test ✅.
Poi `node seed.js && node server.js` deve servire GET /api/tables con 18 tavoli.
FERMATI. Non fare altro.

## FASE 2 — KDS
Crea public/kds/ (index.html + kds.js + kds.css, vanilla, no framework) secondo SPEC §13:
?station=BAR|CUCINA, landscape, 4 ticket per riga, tema Majestic, logo.png nell'header.
Nuovo ticket: oro pulsante + suono ripetuto ogni 4s (Web Audio API). APRI: suono fermo,
card attenuata, PRONTO ✓ verde → evento orderReady. Riconnessione automatica + snapshot.
FERMATI.

## FASE 3 — ASSETS
Da assets/logo.png genera logo_icon.png (solo colonna, quadrato 1024) e logo_full.png.
Configura flutter_launcher_icons (Android + Windows) con l'icona su sfondo #0D0D0F.
FERMATI.

## FASE 4 — APP CAMERIERE (apps/waiter, Flutter, Android + Windows)
Indirizzo server configurabile (prima schermata, default http://192.168.1.10:3000, shared_preferences).
REST /api + socket_io_client. Riverpod. Tema e lingua della SPEC. Schermate:
1. Splash nero con logo che appare.
2. Scelta cameriere: grandi card oro con i nomi da GET /api/waiters + campo per nome nuovo.
3. Mappa tavoli: tab SALA / BAR, 9 card ciascuno + bottone fisso "★ VENDITA DIRETTA — BANCO".
   Card: bordo verde=LIBERO, oro=APERTO con totale + minuti + cameriere, blu=CONTO.
   Live con tablesUpdate.
4. APERTURA TAVOLO: dialog con stepper +/- grandi per NORMALE e CINEMA PLUS, totale,
   bottone APRI TAVOLO (SPEC §3 §5).
5. Schermata ordine: chips categorie orizzontali, griglia prodotti 2 colonne (icona, nome,
   tag stazione BAR 🖨 blu / CUCINA 🖨 arancio, prezzo). Tab CINEMA+: drink inclusi con badge
   "INCLUSO (N rimasti)" e prezzo normale in piccolo; se allowance finito il badge sparisce
   (prezzo normale). Carrello: righe con +/- (rimuovibili prima dell'invio), EXTRA € (dialog
   nome+prezzo), totale, INVIA ORDINE → resta sulla schermata (tavolo aperto). Header mostra
   tavolo, minuti, totale conto live (billUpdate). Bottone CONTO → conto completo →
   CONTANTI / CARTA → tableClose → conferma → torna alla mappa.
6. BANCO: griglia full-width, carrello, CONTANTI/CARTA → counterSale → stampa → schermata ✓ grande.
7. Toast verde su orderReady: "Tavolo X: ordine PRONTO ✓".
Assicurati che compili. FERMATI.

## FASE 5 — APP MANAGER (apps/manager, Flutter, Windows 1280x800 touch)
Sidebar con: Riepilogo, Tavoli, Prodotti, Inventario, Fornitori, Camerieri, Report, Stampanti, Impostazioni.
1. Riepilogo: 4 stat card (incasso oggi, tavoli serviti, cinema plus clienti serviti oggi,
   ticket bar/cucina) da /api/reports/today; striscia live 18 tavoli + banco; lista SCORTE BASSE
   (stock < minimo, bordo rosso); azioni rapide.
2. Tavoli: griglia completa, tap → dettaglio conto + chiusura forzata con conferma.
3. Prodotti: lista per categoria, aggiungi/modifica (nome, categoria, prezzo, stazione, icona,
   is_included_drink checkbox, visible), disattiva; editor collegamenti ingredienti
   (materia prima + qty, aggiungi/rimuovi righe).
4. Inventario: lista con stock/unità/minimo/fornitore/prezzo; modifica; rifornimento (+qty);
   bottoni grandi ESPORTA 📤 e IMPORTA 📥 (.xlsx, SPEC §11, mostra riepilogo aggiornati/aggiunti);
   bottone oro "🛒 CREA ORDINE" (SPEC §10, copia testo WhatsApp).
5. Fornitori: aggiungi/modifica (nome, telefono/email).
6. Camerieri: aggiungi/disattiva nomi.
7. Report: data; totale, split CONTANTI/CARTA, top 10 prodotti, vendite per cameriere,
   clienti cinema plus; esporta CSV.
8. Stampanti: test stampa per BAR/CUCINA, edit IP/porta via settings API.
Assicurati che compili. FERMATI.
