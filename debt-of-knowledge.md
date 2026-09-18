# Debito di conoscenza — C#

Pattern e costrutti che sono serviti durante il lavoro ma che **non** fanno ancora
parte delle convenzioni della skill `skill-csharp`
(`.claude\skills\skill-csharp\SKILL.md`).

**Solo C#.** Qui entrano soltanto i costrutti del linguaggio C# e delle librerie
.NET. CSS, JavaScript, HTML e markup delle view restano fuori: quelli si scrivono
e basta, senza chiedere il permesso e senza lasciare traccia qui.

Ogni voce viene scritta qui **sia se hai detto sì, sia se hai detto no** al momento
in cui è stata proposta: serve a farti capire con calma il pattern, non a
giustificare una scelta già fatta. Sono spiegazioni da leggere a freddo.

Quando ne hai imparata una e vuoi che diventi una regola stabile, dimmi
"promuovi questo nella skill": ogni voce ha in fondo il paragrafo già pronto e la
sezione di destinazione.

---

## 2026-09-09 — Caricare un file (una foto) dal browser: `IFormFile`

**Dove è successo:** progetto `ConcessionarioMVC` (sito del concessionario auto e
moto), file `Controllers\VeicoliController.cs` e le view `CreaAuto.cshtml` /
`CreaMoto.cshtml`. Serviva poter aggiungere la foto di un veicolo direttamente dal
sito, senza copiare il file a mano nella cartella del progetto.

**Cosa avrei usato:**

```csharp
// nel controller: ASP.NET Core ci passa da solo un oggetto che sa dov'è wwwroot
private readonly IWebHostEnvironment ambiente;

public VeicoliController(IWebHostEnvironment ambiente)
{
    this.ambiente = ambiente;
}

// nell'azione POST: "foto" è il file arrivato dal browser
[HttpPost]
public IActionResult CreaAuto(Auto auto, IFormFile? foto)
{
    if (foto != null)
    {
        // wwwroot\immagini\nomefile.jpg
        string percorso = Path.Combine(ambiente.WebRootPath, "immagini", foto.FileName);
        using (FileStream stream = new FileStream(percorso, FileMode.Create))
        {
            foto.CopyTo(stream);   // scrive su disco il contenuto del file caricato
        }
        auto.Immagine = foto.FileName;   // nel CSV salvo solo il nome, non il file
    }
    ...
}
```

e nella view il form deve dichiarare che trasporta un file:

```html
<form method="post" enctype="multipart/form-data">
    <input type="file" name="foto" />
</form>
```

**Perché serviva:** i pattern noti sanno leggere e scrivere file che stanno *già*
sul disco del computer (`StreamReader`/`StreamWriter` in `FileCSV-Lettura` e
`FileCSV-Scrittura`). Un file che arriva da un browser è un'altra cosa: non ha un
percorso su questo computer, viaggia dentro la richiesta HTTP. Serve un tipo che
lo rappresenti mentre è ancora "in viaggio", e serve sapere dove scriverlo.

**Alternativa dentro i pattern noti:** tenere nel modello solo una proprietà
`Immagine` con il nome del file (es. `panda.jpg`) e copiare le foto a mano nella
cartella `wwwroot\immagini` del progetto, mostrandole con
`<img src="~/immagini/@Model.Immagine" />`. Funziona benissimo per un sito
vetrina, ma costa che per aggiungere un veicolo non basta il sito: devi aprire la
cartella del progetto. È l'alternativa che era stata proposta e che hai scartato.

**Come funziona, passo passo:**

1. **`enctype="multipart/form-data"`.** Di default un form HTML manda i campi come
   testo, e di un `<input type="file">` spedirebbe solo il *nome* del file. Con
   questo attributo il browser cambia formato di invio e spedisce anche il
   contenuto vero, in "pezzi" (parti multiple, da cui *multipart*). Se te lo
   dimentichi, il codice C# compila e gira ma `foto` arriva vuoto: è l'errore più
   frequente con gli upload.

2. **`IFormFile`.** È il tipo che ASP.NET Core usa per rappresentare uno di quei
   pezzi. Il nome del parametro nel metodo (`foto`) deve combaciare con
   l'attributo `name` dell'`<input type="file" name="foto" />`: è così che il
   framework capisce quale file mettere in quale parametro. Lo dichiaro
   `IFormFile?` (nullable) perché l'utente può salvare un veicolo senza foto —
   stessa logica dei dati opzionali nelle tue classi.

3. **`IWebHostEnvironment` e la dependency injection.** Il file va scritto dentro
   `wwwroot\immagini`, ma il percorso assoluto di quella cartella cambia da
   computer a computer, e non voglio scriverlo fisso nel codice.
   `IWebHostEnvironment` è un oggetto che ASP.NET Core tiene pronto e che, tra le
   altre cose, espone `WebRootPath` = il percorso vero di `wwwroot`.
   *Dependency injection* significa: non lo creo io con `new`, lo dichiaro come
   parametro del costruttore del controller e il framework me lo passa da solo
   quando costruisce il controller per servire una richiesta. È lo stesso
   meccanismo con cui in `GamMVC` il `GamContext` arriva dentro
   `OperasController` — lì l'aveva scritto lo scaffolding, qui lo scrivo a mano.

4. **`Path.Combine`.** Unisce i pezzi di un percorso mettendo i backslash al posto
   giusto. Meglio che concatenare stringhe a mano: non sbaglia mai un separatore.

5. **`new FileStream(percorso, FileMode.Create)` dentro `using`.** Apre (o
   sovrascrive, per `FileMode.Create`) il file di destinazione. È lo stesso
   `using` che usi con `StreamWriter`: garantisce che il file venga chiuso anche
   se qualcosa va storto. `foto.CopyTo(stream)` travasa dentro il contenuto
   arrivato dal browser.

   Nota: quasi tutti gli esempi che trovi in giro usano `await
   foto.CopyToAsync(stream)` dentro un metodo `async Task<IActionResult>`. Esiste
   anche la versione sincrona `CopyTo`, che fa la stessa identica cosa
   aspettando che finisca — e permette di restare con `IActionResult` normale,
   senza tirare dentro anche `async`/`await`. Su un sito con pochi utenti la
   differenza non si vede; `async` serve quando tanti utenti caricano file
   insieme e non vuoi tenere occupato un thread ad aspettare il disco.

6. **Nel file CSV finisce solo il nome** (`panda.jpg`), non l'immagine. Il file
   vero sta in `wwwroot\immagini`, e la pagina lo ripesca con
   `<img src="~/immagini/@Model.Immagine" />`. È la stessa idea del campo
   `Immagine` che c'è già nella tabella `Opera` di `GamMVC`.

**Cosa manca, e che qui non è stato fatto:** in un sito vero l'upload va
controllato (che sia davvero un'immagine, che non sia enorme, e soprattutto che
il nome del file non contenga `..\` per scrivere fuori da `wwwroot`). Qui non c'è
niente di tutto questo, coerentemente con la tua convenzione "nessun controllo
sui dati inseriti": è un sito che gira solo sul tuo computer. Se un giorno lo
pubblichi davvero, questo è il primo punto da sistemare.

**Esito:** approvato. Nel codice sono finiti `IFormFile`, `IWebHostEnvironment`
iniettato nel costruttore e `enctype="multipart/form-data"`; `async`/`await` no,
grazie alla versione sincrona `CopyTo`.

**Se lo promuovi nella skill:** sezione di destinazione `## Siti web senza
database` di `SKILL.md`, come nuovo sotto-paragrafo.

> **Caricamento di immagini dal sito.** Quando una pagina deve accettare un file
> (tipicamente la foto di un articolo), l'azione POST riceve un parametro
> `IFormFile? nomeCampo` che combacia con il `name` dell'`<input type="file">`, e
> il form dichiara `enctype="multipart/form-data"`. Il controller si fa passare
> `IWebHostEnvironment` nel costruttore per ricavare `WebRootPath` e scrive il
> file con `new FileStream(percorso, FileMode.Create)` dentro un `using`, usando
> `foto.CopyTo(stream)` (versione sincrona: niente `async`/`await`). Nel modello
> e nel file dati si salva solo il nome del file; la view lo mostra con
> `<img src="~/immagini/@Model.Immagine" />`.

---

## 2026-09-09 — Modelli `public` invece di `internal` in un controller MVC

**Dove è successo:** progetto `ConcessionarioMVC`, file `Models\Veicolo.cs`,
`Auto.cs`, `Moto.cs` e i tre enum. Non è un costrutto nuovo — è uno **scostamento
da una convenzione** della skill ("tutte le classi sono `internal`") che il
compilatore ha imposto.

**Cosa è successo:** con le classi `internal` la compilazione si è fermata con
quattro errori tutti uguali:

```
error CS0051: Accessibilità incoerente: il tipo parametro 'Auto' è meno
accessibile del metodo 'VeicoliController.CreaAuto(Auto, IFormFile?)'
```

**Perché:** un controller MVC **deve** essere `public` (è così che ASP.NET Core lo
trova e lo usa). Le sue azioni sono quindi metodi pubblici. C# non permette che un
metodo pubblico abbia nel proprio elenco di parametri un tipo che il mondo esterno
non può vedere: sarebbe un metodo che nessuno può chiamare, e il compilatore lo
segnala come incoerenza. Stesso motivo per gli enum: sono il tipo di proprietà
pubbliche di classi pubbliche.

Nota: in `EsempioMVC` la classe `Prodotto` è `internal` e compila lo stesso, perché
lì i modelli sono solo **restituiti** alle view (`return View(prodotto)`), mai
ricevuti come parametro di un'azione. È l'arrivo delle form di inserimento e
modifica a cambiare le carte in tavola.

**Alternativa dentro i pattern noti:** tenere i modelli `internal` e scrivere le
azioni POST con i parametri sciolti, come `HomeController.RecuperoContatto(string
Nominativo, string Email, ...)` in `EsempioMVC`, ricostruendo l'oggetto a mano
dentro il metodo. Funziona, ma qui voleva dire tredici parametri per azione su
quattro azioni, e gli enum sarebbero comunque diventati `int` da convertire a
mano. Molto più codice e molto meno leggibile.

**Esito:** nel codice sono finite `public class Veicolo`, `public class Auto`,
`public class Moto` e i tre enum `public`. `VeicoloDAL` è rimasta `internal`,
perché la usano solo i controller al proprio interno: lì nessuna incoerenza.

**Se lo promuovi nella skill:** sezione `## Struttura progetto` di `SKILL.md`,
come precisazione alla riga sulle classi `internal`.

> Nei progetti ASP.NET Core MVC le classi modello che compaiono come **parametro**
> di un'azione di un controller (le form di inserimento e modifica) devono essere
> `public`, insieme agli enum usati come tipo delle loro proprietà: un controller è
> pubblico per forza, e C# rifiuta un metodo pubblico con parametri di tipo
> `internal` (`CS0051`). Restano `internal` i modelli solo restituiti alle view e
> le classi di servizio usate dentro il controller (es. un DAL).

---

## 2026-09-18 — Parametrizzare una ricerca: `@Cerca` al posto del testo incollato

**Dove è successo:** due punti rimasti indietro rispetto al resto del codice, che
invece parametrizza già tutto:

- `AnagraficaGUI\AnagraficaGUI\AnagraficaGUI\StudentiDAL.cs`, metodo `Dettaglio`;
- `WebFormsIstat\WebFormsIstat\Ricerca.aspx.cs`, click del pulsante "Cerca".

**Cosa avrei usato:**

```csharp
// PRIMA — il testo digitato finisce dentro la query, mescolato ai comandi SQL
sdsComuni.SelectCommand = "... WHERE c.[Denominazione] like '%" + txtCerca.Text + "%' ...";

// DOPO — nella query c'e' solo un segnaposto; il valore viaggia a parte
sdsComuni.SelectCommand = "... WHERE c.[Denominazione] like @Cerca ...";
sdsComuni.SelectParameters.Clear();                                  // senno' si accumula a ogni ricerca
sdsComuni.SelectParameters.Add("Cerca", "%" + txtCerca.Text + "%");  // i % stanno nel valore
```

**Perché serviva:** finché il testo viene incollato dentro la query, quello che
scrivi nella casella di ricerca *è* SQL. Se uno scrive `' OR 1=1 --` la query che
arriva al database non è più quella che avevi in mente: è un'altra, scritta da
lui. Si chiama **SQL injection** (letteralmente: iniettare comandi dentro una
query). Con il parametro il database riceve due cose separate — il testo della
query da una parte, il valore dall'altra — e il valore non viene mai letto come
comando, qualunque cosa contenga.

Nel caso di `StudentiDAL.Dettaglio` il parametro era un `int`: dentro un intero
un comando SQL non ci sta, quindi lì il buco non c'era davvero. Era però l'unico
metodo del DAL scritto in modo diverso dagli altri quattro, e un'incoerenza del
genere è proprio quella che poi ti fa copiare lo schema sbagliato nel prossimo
progetto.

**Alternativa dentro i pattern noti:** per il DAL non serviva nulla di nuovo —
`command.Parameters.Add("@Matricola", SqlDbType.Int).Value = matricola;` è già la
riga che usano `Nuovo`, `Modifica` ed `Elimina`. Per la pagina Web Forms invece
c'era una scelta:

- **quella che ho usato:** lasciare la query nel code-behind com'era e aggiungere
  `SelectParameters` in C# — due righe in più, il file resta dov'è;
- **quella più in linea con le convenzioni:** spostare la query nel markup
  `Ricerca.aspx` dentro `<asp:SqlDataSource>` e legare la casella di testo con
  `<asp:ControlParameter ControlID="txtCerca" Name="Cerca" PropertyName="Text" />`,
  lasciando nel code-behind il solo `gvComuni.DataBind()`. È più coerente con la
  regola "nei Web Forms la logica sta nel markup e il code-behind resta quasi
  vuoto", ma introduce un tag che non avevi ancora usato.

**Come funziona, passo passo:**

1. `SelectCommand` contiene `@Cerca`. Per il database quello non è testo da
   cercare: è un **segnaposto**, cioè "qui arriverà un valore, te lo passo dopo".
2. `SelectParameters.Add("Cerca", valore)` è la promessa mantenuta: dice quale
   valore mettere in quel segnaposto. Il nome si scrive senza `@`, il segnaposto
   nella query con `@` — è una stranezza di ASP.NET, non un errore.
3. I due `%` della `like` (i caratteri jolly: "qualsiasi cosa prima, qualsiasi
   cosa dopo") vanno messi **nel valore**, non nella query. Se li lasci nella
   query attorno al segnaposto non funziona più niente, perché il segnaposto è
   già un valore intero, non un pezzo di testo da concatenare.
4. `SelectParameters.Clear()` prima di `Add` serve perché il `SqlDataSource` vive
   quanto la pagina: senza `Clear`, al secondo clic su "Cerca" ti ritrovi due
   parametri `Cerca` e la query non parte.
5. Nel DAL il meccanismo è identico, cambia solo il nome del metodo:
   `command.Parameters.Add("@Matricola", SqlDbType.Int).Value = matricola`, dove
   in più dichiari il tipo SQL della colonna.

**Esito:** approvato. Nel codice sono finite le due correzioni, entrambe con i
commenti che spiegano il perché. Per la pagina Web Forms è stata scelta la prima
alternativa (parametri in C#); se vuoi passare alla seconda (`ControlParameter`
nel markup) è una modifica di dieci minuti.

**Promosso nella skill il 2026-09-18:** la vecchia sezione
`### ⚠️ Nota sicurezza — da non replicare` è stata sostituita da
`### Filtri e ricerche: sempre parametrizzati`, dentro
`## Accesso a database (SQL Server)`. Il paragrafo qui sotto è quello che ci è
finito.

> **Filtri e ricerche: sempre parametrizzati.** Anche nelle `SELECT`, non solo in
> insert/update/delete. Nel DAL ADO.NET si usa
> `command.Parameters.Add("@Nome", SqlDbType.VarChar, 50).Value = ...`; nei Web
> Forms si mette `@Nome` nella `SelectCommand` e si passa il valore con
> `SelectParameters` (preceduto da `SelectParameters.Clear()`, altrimenti i
> parametri si accumulano a ogni postback). I caratteri jolly `%` della `like`
> vanno nel valore del parametro, mai nel testo della query. Non si concatena mai
> testo digitato dall'utente dentro una query: sarebbe SQL injection (chi scrive
> nella casella di ricerca può far eseguire al database comandi non previsti).
