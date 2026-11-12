# L'Almanacco del giorno dopo — schema dei contenuti (v1)

## File

- `giorni/MM-GG.json` — una scheda per giorno (366 file, compreso `02-29.json`), indipendente dall'anno.
- `feste_mobili.json` — regole per le ricorrenze che cambiano data ogni anno (Pasqua e derivate, "terza domenica di…").
- `smorfia.json` — i 90 numeri della Smorfia napoletana (versione completa, senza censure): napoletano, italiano, spiegazione.

Tutto ciò che dipende dall'anno NON sta nei file: lo calcola l'app.

| Dato | Dove |
|---|---|
| santo, eventi, nati, curiosità, citazioni, prima pagina | scheda del giorno |
| giorno della settimana, alba/tramonto, fase lunare, "X anni fa" | calcolato dall'app |
| feste mobili (Pasqua, Ceneri, Cristo Re…) | `feste_mobili.json` + calcolo |
| notizia del giorno, meteo | scaricati all'apertura (RSS, Open-Meteo); se offline la sezione si nasconde |

## Scheda del giorno

```json
{
  "schema": 1,
  "giorno": "11-13",
  "santo": {
    "nome": "Sant'Omobono",
    "nota": "una o due righe",
    "altri": ["San Brizio di Tours", "..."]
  },
  "ricorrenze": [
    { "nome": "Giornata mondiale della gentilezza", "ambito": "mondo" }
  ],
  "eventi": [
    {
      "id": "11-13-1917",
      "anno": 1917,
      "titolo": "Titolo breve, da giornale",
      "testo": "Due-tre righe nel tono dell'almanacco.",
      "ambito": "italia | mondo",
      "categoria": "storia | scienza | tecnologia | spazio | cultura | sport | curiosita",
      "peso": 5,
      "wiki": "Titolo esatto della voce su it.wikipedia",
      "tono": "leggero | storico | tragico",
      "indizio": "Domani: … (per gli eventi con peso >= 4 non tragici)"
    }
  ],
  "nati": [
    { "nome": "Robert Louis Stevenson", "anno": 1850, "chi": "breve descrizione", "wiki": "Robert Louis Stevenson" }
  ],
  "curiosita": [
    { "categoria": "lingua | scienza | tecnologia | cultura | storia", "titolo": "…", "testo": "…" }
  ],
  "citazioni": [
    { "testo": "…", "autore": "…", "fonte": "opera o 'proverbio'" }
  ],
  "proverbi": [
    { "testo": "Sasso trato e parola dita no torna più indrìo.", "regione": "Veneto",
      "traduzione": "…", "spiegazione": "…", "fonte": "Wikiquote, Proverbi veneti" }
  ],
  "prima_pagina": null
}
```

- Scorte per giorno: 12-15 eventi, 5 curiosità, 5 citazioni, 3 proverbi di regioni diverse.
- `tono`: `tragico` = disastri, stragi, battaglie sanguinose. Mai protagonista, mai indizio, al massimo uno fra gli eventi minori.
- `anno` negativo = avanti Cristo (es. `-42`).
- `peso` da 1 a 5: quanto il fatto è memorabile per un lettore italiano.
- `wiki`: l'app ricava la foto e i crediti da Wikipedia/Commons (`/api/rest_v1/page/summary/{titolo}` + `imageinfo` con `extmetadata` per autore e licenza) e li mette in cache. Nessuna immagine è dentro l'APK.
- `prima_pagina` (al massimo 60 giorni l'anno):

```json
{
  "testata": "The Sun",
  "luogo": "Baltimora",
  "data": "1863-11-21",
  "evento_id": "11-19-1863",
  "fonte": "Library of Congress, …",
  "url": "https://www.loc.gov/item/…",
  "loc_item": "scsm001130",
  "diritti": "Pubblico dominio (1863)",
  "didascalia": "Cosa guardare nella pagina."
}
```

## Cosa mostra l'app ogni anno (rotazione)

Ogni scheda ha più materiale di quanto se ne mostri (8-10 eventi, 3 curiosità, 3 citazioni). L'app sceglie in modo **deterministico**: stesso giorno e stesso anno danno sempre la stessa pagina; un anno diverso dà una combinazione diversa.

1. **Punteggio** di ogni evento per l'anno `A`:
   `punti = peso × 6 + 2 × bonus_anniversario + h(A, id) mod 12`
   con età = `A − anno` (senza anno zero per gli eventi avanti Cristo) e bonus: multiplo di 100 → +30, di 50 → +20, di 25 → +12, di 10 → +5.
   `h` = FNV-1a a 32 bit sulla stringa `"A|id"`. La parte casuale (0-11) fa ruotare i protagonisti fra un anno e l'altro; il bonus raddoppiato fa vincere quasi sempre l'anniversario tondo.
2. **Protagonista** = evento con più punti fra quelli con `peso ≥ 4`, oppure `peso 3` con un anniversario multiplo di 50 (se non ce ne sono, fra tutti). L'etichetta "X anni fa" è evidenziata dai 25 anni tondi in su.
3. **Altri tre eventi**: i migliori fra i restanti, preferendo anni diversi fra loro e dal protagonista, con almeno un evento italiano e uno dal mondo se disponibili.
   Al punteggio si aggiungono +6 se `tono` è `leggero`; i `tragici` sono esclusi dal protagonista. Fra i quattro eventi visibili ce n'è sempre almeno uno di `storia`.
4. **Curiosità, citazione e proverbio**: indice `h(A, giorno…) mod n`.
5. **Numero della Smorfia**: `h(A, giorno) mod 90 + 1`, fisso per tutto il giorno; il pulsante «Scuoti» ne estrae uno a caso.
6. **Nati**: stellina sull'anniversario tondo della nascita (25, 50, 100… anni).
7. **Indizio di domani**: il campo `indizio` del protagonista del giorno dopo (dalle 20:00).

## Feste mobili

`feste_mobili.json`: tipi di regola `pasqua` (scarto in giorni dalla domenica di Pasqua), `ennesima_domenica` (mese, n), `ultima_domenica` (mese), `domenica_prima` (di una data fissa). Con `"sostituisce_santo": true` la festa prende il posto del santo in cima al foglietto.

## Longevità

- **Memoria del già visto** (nell'APK): l'app registra gli id mostrati e, a parità di punti, preferisce ciò che non hai ancora letto, finché la scorta non è esaurita.
- **Pacchetti annuali**: un file `contenuti-AAAA.json` su GitHub. L'app lo cerca una volta al mese; se non c'è o è rotto usa quello incluso nell'APK. Il campo `schema` permette pacchetti futuri compatibili.
- **Offline per sempre**: schede, Smorfia, proverbi e foto dei protagonisti (compresse) sono dentro l'APK. Da internet arrivano solo notizia, meteo e foto secondarie, che spariscono senza errori se il servizio non risponde.
- **Indirizzi modificabili**: gli indirizzi dei feed RSS si cambiano dalle impostazioni, senza ricompilare.
- **Ricostruibile**: gli script di generazione e verifica restano nel progetto, così rifare o allargare un pacchetto segue lo stesso metodo.
