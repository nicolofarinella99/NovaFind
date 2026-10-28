<h1 align="center">NovaFind</h1>

<p align="center">
  <b>Ricerca file velocissima per Windows, in stile Spotlight.</b><br>
  Dischi locali e server di rete SMB · risultati istantanei · un solo .exe portabile
</p>

<p align="center">
  <a href="README.md">🇬🇧 Read in English</a>
</p>

---

La ricerca di Windows è lenta e sui server della rete aziendale spesso non funziona proprio. **NovaFind** resta in background, si apre con una scorciatoia in pochi millisecondi e trova file e cartelle mentre scrivi, sul PC e sui server che aggiungi.

- ⚡ **Istantaneo**: indice in memoria, risultati in 1–3 ms su centinaia di migliaia di file
- 🗂️ **Locale e rete**: dischi, singole cartelle, `\\server\condivisione` o `\\192.168.x.x\condivisione`, con credenziali facoltative
- 🪟 **Stile Spotlight**: finestra compatta, tutto da tastiera, icone vere di Windows, anteprime
- 🎨 **Interfaccia in vetro con tre temi**: Spaziale (stelle, aurora, stelle cadenti), Scuro e Chiaro, più colore principale e trasparenza regolabile
- 📦 **Un solo file portabile**: circa 400 KB, senza installazione, senza DLL, nessun runtime da scaricare
- 🔒 **Privato**: non si collega mai a internet; niente telemetria né controlli aggiornamenti

## Download

Scarica `NovaFind.exe` dalla pagina [Releases](../../releases) e avvialo con un doppio clic.

**Requisiti:** Windows 10 o 11 a 64 bit, con .NET Framework 4.8 (già incluso in Windows 10 1903+ e in Windows 11).

> L'eseguibile non è firmato digitalmente: al primo avvio Windows SmartScreen può mostrare *"Windows ha protetto il PC"*. Clicca **Ulteriori informazioni → Esegui comunque**.

## Uso

1. Avvia `NovaFind.exe`. Resta nell'area di notifica (tasto destro: *Apri / Impostazioni / Esci*).
2. Premi **Ctrl+Alt+Spazio** (modificabile). Se un altro programma la usa già, NovaFind passa ad Alt+Shift+Spazio o Ctrl+Alt+F e te lo segnala.
3. Scrivi: i risultati si aggiornano a ogni tasto.

### Tastiera

| Tasto | Azione |
|---|---|
| ↑ ↓ PagSu PagGiù | scorri i risultati |
| Invio | apri |
| Ctrl+Invio | mostra nella cartella (con il file selezionato) |
| Alt+Invio | proprietà |
| Ctrl+C | copia il percorso |
| Tab | aggiunge/toglie la cartella aperta in Esplora file |
| Ctrl+1 / 2 / 3 | tutto / solo file / solo cartelle |
| Ctrl+P | mostra o nasconde l'anteprima |
| Esc | chiudi |

Tasto destro su un risultato: *Apri, Mostra nella cartella, Copia percorso, Copia nome, Proprietà*. I risultati si possono anche trascinare in altre applicazioni.

### Sintassi di ricerca

| Ricerca | Significato |
|---|---|
| `fattura 2024` | tutte le parole devono comparire nel nome, in qualsiasi ordine |
| `ext:pdf` · `ext:pdf,docx` · `*.pdf` | filtra per estensione |
| `dir:` / `file:` | solo cartelle / solo file |
| `"frase esatta"` | cerca le parole con gli spazi |

Ordinamento: prima il nome esatto, poi i nomi che iniziano con la ricerca, poi i file recenti e quelli aperti spesso.

### Dove cercare

I chip sotto la barra scelgono dove cercare: **Tutto il PC, Desktop, Documenti, Download**, cartelle tue e server di rete. Puoi selezionarne più di uno, e con Ctrl+clic cerchi solo in quello. Se in primo piano c'è una finestra di Esplora file compare il chip **Cartella corrente**, da usare insieme agli altri o da solo.

**+ Aggiungi** apre la finestra per nome, percorso, utente/dominio/password facoltativi, indice sì/no, ogni quanto reindicizzare e quante letture parallele fare sul server, con il pulsante *Test connessione*.

## Aspetto

Dal **pulsante tavolozza** nella finestra di ricerca o da *Impostazioni → Aspetto*. Le modifiche si vedono subito.

- **Tema:** Automatico (segue chiaro/scuro di Windows), Spaziale, Scuro, Chiaro
- **Colore principale:** quello del tema oppure uno tra 10 colori, adattato automaticamente per restare leggibile
- **Trasparenza dello sfondo:** slider 0–100%, con sfocatura effetto vetro facoltativa
- **Animazioni:** complete, leggere (solo transizioni) o nessuna; si fermano quando la finestra è nascosta
- **Dimensione finestra:** compatta, normale o ampia, con pannello di anteprima facoltativo

## Come funziona

NovaFind è scritto in **C# con WPF su .NET Framework 4.8** e compilato con il `csc.exe` incluso in Windows. Niente pacchetti NuGet, niente librerie di terze parti.

- **Indice compatto:** nessun oggetto per file, solo array paralleli e un unico buffer di byte per i nomi, circa 40 byte per elemento. È salvato su disco e all'avvio si carica in circa 10 ms.
- **Scansione parallela** con `FindFirstFileEx` (large fetch), a bassa priorità di I/O per le riscansioni. Le cartelle locali restano allineate con `FileSystemWatcher`. Lettura facoltativa della MFT NTFS se avviato come amministratore.
- **Ricerca parallela** sull'indice con confronto `unsafe` senza allocazioni. Dove non c'è un indice si usa la **ricerca live in streaming**: i risultati compaiono man mano.
- **Gentile con i server:** concorrenza limitata, controlli di raggiungibilità con timeout, messaggi d'errore chiari, reindicizzazione solo su richiesta o all'intervallo scelto.
- **Integrazione nativa con Windows** via P/Invoke: icone e miniature della shell, `SHOpenFolderAndSelectItems`, `WNetAddConnection2`, scorciatoia globale, sfocatura di composizione.

### Prestazioni

Misurate su un PC di prova (Windows 11, 8 thread, SSD, utente standard):

| | |
|---|---|
| Apertura finestra (scorciatoia → primo frame) | 10–30 ms |
| Indicizzazione di ~330.000 elementi | ~1,4 s (cache del file system già calda) |
| Caricamento dell'indice all'avvio | ~10 ms |
| Ricerca sull'indice | 1–3 ms tipica, ~13 ms per una sola lettera |
| RAM a riposo (working set) / CPU | ~25 MB / 0% |

## Compilare dai sorgenti

```bash
build.bat
```

Funziona offline usando solo il compilatore C# incluso in Windows (`%WINDIR%\Microsoft.NET\Framework64\v4.0.30319\csc.exe`). Lo script prima disegna il logo in codice e scrive l'`.ico` (`tools\IconGen.cs`), poi compila `dist\NovaFind.exe`.

Tutti i parametri grafici (tavolozze, colori, font, raggi, misure, durate delle animazioni, campo di stelle) sono in [`src/Theme.cs`](src/Theme.cs).

```
src/
  App.cs              avvio, istanza singola, tray, scorciatoia, modalità benchmark
  Theme.cs            temi e tutti i parametri grafici
  Logo.cs             logo vettoriale e scrittura del file .ico
  FileIndex.cs        indice compatto in memoria + cache su disco
  Scanner.cs          scansione parallela, aggiornamenti incrementali, lettura MFT
  IndexManager.cs     indicizzazione in background, monitoraggio file, riscansioni
  Search.cs           query, ricerca parallela, punteggio, ricerca live
  Network.cs          accesso SMB, credenziali, timeout, elenco condivisioni
  Settings.cs         impostazioni, destinazioni, cronologia, avvio con Windows
  Shell.cs            icone, miniature, apri / mostra / proprietà, cartella di Esplora file
  UI/                 finestra di ricerca, impostazioni, dialoghi, stili, controlli
tools/IconGen.cs      generatore dell'icona usato da build.bat
```

## Dati e privacy

Tutto resta nel profilo dell'utente, in `%LOCALAPPDATA%\NovaFind\`:

| File | Contenuto |
|---|---|
| `settings.json` | impostazioni e aspetto |
| `destinations.json` | destinazioni; le password solo **cifrate con DPAPI** di Windows |
| `history.json` | file aperti e ricerche recenti |
| `index\*.nfx` | cache degli indici |
| `novafind.log` | diagnostica (mai password) |

*Impostazioni → Privacy e dati → Cancella tutti i dati locali* elimina tutto e rimuove l'avvio automatico.

Riga di comando: `--background`, `--toggle`, `--settings`, `--quit`, `--stats`, `--bench <cartella> <file-output> [query…] [--live]`.

## Limiti noti

- Cerca nei **nomi** di file e cartelle, non nel contenuto.
- I server di rete non sono monitorati in tempo reale: si aggiornano su richiesta o all'intervallo scelto.
- L'indicizzazione veloce tramite MFT richiede i diritti di amministratore; altrimenti si usa la scansione normale.
- La sfocatura effetto vetro richiede Windows 10 1803 o successivo.
- La memoria privata impegnata da .NET/WPF è più alta del working set (~150–200 MB).
