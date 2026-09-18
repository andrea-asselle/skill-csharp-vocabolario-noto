---
name: skill-csharp
description: Convenzioni personali di programmazione C#/.NET dell'utente e regola del vocabolario noto - scrivere solo con i costrutti che l'utente conosce già, e fermarsi a chiedere prima di introdurne di nuovi. Usa SEMPRE questa skill prima di progettare, scrivere, modificare, spiegare o correggere qualsiasi codice C# o file .cs, .csproj, .cshtml, .aspx, .sln; prima di creare un progetto .NET (console, WinForms, ASP.NET Web Forms, ASP.NET Core MVC) o un sito web; prima di lavorare con SQL Server o Entity Framework Core da C#. Vale per qualunque cosa scriva, anche per una singola riga o per una domanda che sembra banale.
---

# Programmare in C# con l'utente

## La regola centrale

L'utente ha imparato il C# da poco e continua a impararlo scrivendo programmi
suoi. Il suo vocabolario di costrutti è preciso e limitato:
è quello che ha effettivamente scritto nei progetti della sua cartella di
lavoro, ed è documentato in questo file.

Questo cambia il significato di "buon codice" qui. Codice corretto, elegante e
moderno ma scritto con costrutti che lui non conosce è, per lui, codice inutile:
non lo sa rileggere, non lo sa spiegare a qualcun altro, non lo sa modificare
fra un mese. La qualità, qui, si misura in **quanto è comprensibile a lui**, non in
quanto è idiomatico.

Quindi: **resta dentro i pattern documentati qui. Se servisse qualcosa che in
questo file non c'è, fermati prima di scrivere qualunque riga e chiedi** — il
protocollo è nella sezione "Quando i pattern non bastano".

Vale allo stesso modo per un vecchio esercizio ripreso in mano e per un programma
che si inventa da zero: il vocabolario è lo stesso.

## Progettare un programma nuovo

Quando non c'è una consegna scritta ma un'idea dell'utente, il rischio più grosso
non è sbagliare una riga: è scegliere un'architettura fuori dal suo vocabolario
*prima* di scrivere codice, e poi presentargliela come fatto compiuto. Le
decisioni di progetto si prendono con lui, non per lui.

Ordine di lavoro:

1. **Chiarisci cosa deve fare il programma**: quali dati entrano, cosa deve
   uscire, cosa va tenuto in memoria. Poche domande concrete, non un'analisi.
2. **Decidi se servono classi tue**:
   - solo calcoli, cicli, array, date, stringhe → top-level statements, nessuna
     classe (vedi "Program.cs");
   - ci sono "cose" con proprietà (un Libro, un Atleta, un Comune) → classi +
     `internal class Program`.
3. **Elenca le classi che pensi di creare, con le proprietà principali, prima di
   scrivere i file.** Una lista di cinque righe, non un progetto UML. È il
   momento in cui l'utente può correggere il tiro a costo zero.
4. **Se ci sono dati da conservare**, scegli tra: niente (tutto in memoria), file
   di testo/CSV, database SQL Server → vedi `references/database-e-web.md`.
5. **Scrivi il codice** seguendo le convenzioni qui sotto.

**Commento in testa al file.** Quando il programma nasce da una consegna scritta
(come nei vecchi esercizi) si incolla il testo come blocco `/* ... */`. Quando
nasce da un'idea dell'utente la consegna non esiste: al suo posto, due o tre
righe di commento che dicono cosa fa il programma e a cosa serve. Stessa funzione — capire di cosa si
tratta riaprendo il file fra sei mesi.

---

# Convenzioni di stile

Queste sono le convenzioni osservate nel codice esistente dell'utente: seguile
quando scrivi codice C#, salvo sue indicazioni diverse.

La cartella di lavoro contiene decine di piccoli progetti
.NET (`net10.0`), ognuno un programma a sé stante, con il proprio `.csproj` ed
eseguibile.

## Struttura progetto

- Ogni progetto è **autosufficiente**: nessun `.csproj` usa mai
  `ProjectReference` verso un altro progetto della cartella. Se serve una classe
  già scritta altrove (es. `Indirizzo`), riusala copiandola fisicamente nel
  progetto corrente — in Visual Studio: tasto destro sul progetto → **Aggiungi
  elemento esistente** (non "Aggiungi come collegamento", che crea un link
  esterno invece di una copia).
- Un file `.cs` per classe/interfaccia/enum, nome file = nome tipo
  (es. `Autore.cs`, `IAtleta.cs`, `TipoCarburante.cs`).
- Il `namespace` combacia sempre con il nome della cartella di progetto (non con
  eventuali cartelle-contenitore esterne).
- Tutte le classi sono `internal` (eccezione: enum a volte `public`).
- ⚠️ Quando rinomini una classe (es. un controller MVC) o rifai una view, elimina
  il vecchio file `.cs`/`.cshtml` invece di lasciarlo affiancato al nuovo con lo
  stesso nome di classe/route. File duplicati con la stessa classe nello stesso
  namespace rompono la build dell'intero progetto (`CS0101`/`CS0111`), e in
  Visual Studio l'errore si manifesta spesso come falsi errori altrove (es. su
  `Program.cs`) invece che sul file incriminato — controlla sempre l'intero
  output di build, non solo la prima riga segnalata.

## Program.cs — due stili, scelti in base al tipo di programma

- **Programmi algoritmici semplici** (senza classi personalizzate: somme, cicli,
  array, date, dadi, ecc.): **top-level statements**, nessun `namespace`, nessuna
  `class Program`. Nessun metodo helper viene mai estratto — tutto inline nel
  corpo dello statement top-level.
- **Programmi con classi/OOP** (oggetti, ereditarietà, interfacce, file I/O):
  `internal class Program { static void Main(string[] args) { ... } }` dentro
  `namespace NomeProgetto`.
- In entrambi gli stili, in cima al file (o in cima al `Main`) va un blocco di
  commento `/* ... */` in italiano: il testo della consegna se c'è, spesso con
  intestazione `Esercizio - NomeEsercizio`; altrimenti due o tre righe che
  descrivono cosa fa il programma.
- Graffe **Allman** (parentesi graffa aperta su una riga a sé), sempre.

## Proprietà e costruttori

- Auto-properties (`public Tipo Nome { get; set; }`), mai campi pubblici nudi.
- Dati obbligatori: `public required Tipo Nome { get; set; }`.
- Dati opzionali: `public Tipo? Nome { get; set; }` (nullable), mai `required`.
- Costruttori espliciti **solo** per classi "di valore" semplici tipo `Indirizzo`
  o quando c'è ereditarietà con `: base(...)` (es. `Solidi`). Altrimenti si usa
  sempre l'object-initializer con proprietà `required`, senza scrivere un
  costruttore:
  ```csharp
  Autore a1 = new Autore() { Nome = "Piero", Cognome = "Rossi" };
  ```

## Valori derivati/calcolati

Preferisci **metodi** (`public double Prezzo()`, `public int Eta()`) a proprietà
calcolate get-only. Le proprietà computate esistono solo come wrapper semplici
di un campo privato (es. `public double Raggio => _raggio;`), mai per logica di
business.

## Stampa dettaglio

Pattern più recente e da preferire per il codice nuovo: un metodo dedicato che
**ritorna** una stringa formattata, poi stampata da `Program.cs`:
```csharp
public string FormatStampaDettaglio()
{
    string msg = "" +
        $"\nCampo={Valore}";
    return msg;
}
// in Program.cs:
Console.WriteLine(oggetto.FormatStampaDettaglio());
```
In alternativa, nei progetti più vecchi si trova un override di `ToString()`
con interpolazione e `nameof()`. Non stampare mai direttamente con
`Console.WriteLine` dentro la classe modello — la stampa resta sempre
responsabilità di `Program.cs`.

## Ereditarietà, interfacce, enum

- Classe base astratta con metodo `public abstract` o `public virtual`;
  le sottoclassi fanno `override` e spesso richiamano `base.Metodo(...)`.
- Interfacce con prefisso `I` (es. `IAtleta`, `ITennista`); più interfacce
  possono essere composte in una interfaccia "universale" che le eredita tutte.
- Enum con prefisso `Tipo` (es. `TipoCarburante`, `TipoMateriale`).
- Per il controllo di tipo si usa `is`/cast esplicito, non ancora pattern
  matching `is X x`.

## Collezioni

Array (`T[]`) per insiemi piccoli e fissi (es. `Autore[] autori`); `List<T>`
solo quando la collezione deve crescere dinamicamente.

## Input/Output

- Lettura da console: sempre `Console.ReadLine()` + `int.Parse(...)` /
  `double.Parse(...)`. Mai `Convert.ToInt32`/`Convert.ToDouble`.
- Output: string interpolation `$"..."` come stile dominante.
- File di testo/CSV: I/O manuale con `StreamReader`/`StreamWriter` (o
  `File`), parsing CSV fatto a mano con `Split(';')`/interpolazione — mai
  librerie esterne come CsvHelper. Preferisci blocchi `using (...)` per gli
  stream (osservato ma non sempre applicato in modo coerente nel codice
  esistente).
- Percorsi file: preferisci percorsi relativi (es. `@"..\..\..\File\dati.csv"`)
  a percorsi assoluti hardcoded.

## Valori monetari

Formattazione sempre con `{valore:F2}` seguito dalla stringa letterale
`" euro"` (es. `$"Prezzo: {Prezzo():F2} euro"`). Mai `ToString("C")`, mai il
simbolo `€`.

## Siti web senza database

Capita di voler fare un sito che non salva niente: la vetrina di una piccola
attività, una pagina di presentazione, un listino. Le convenzioni valgono lo
stesso — sia quelle generali qui sopra, sia quelle sui progetti web qui sotto.

Il modello già documentato per questo caso è il **progetto MVC senza database**:
ASP.NET Core MVC
con struttura Controllers/Models/Views e i dati scritti a mano nel controller come
`List<T>`, senza DAL e senza Entity Framework. Se le informazioni del sito sono
poche e cambiano di rado (servizi offerti, orari, listino), quella lista nel
controller è la risposta giusta: non serve un database per tenere otto righe di
testo.

Resta valido anche il resto delle regole MVC: il link nel menu di
`Views/Shared/_Layout.cshtml` va aggiunto a mano, i titoli delle view vanno
tradotti in italiano, `Details(int codice)` con `NotFound()` per la pagina di
dettaglio.

Quello che invece un sito "vero" chiede e che qui non è documentato — un form di
contatto che invia email, l'aspetto grafico oltre al Bootstrap del template, la
pubblicazione online — non è coperto da nessun pattern noto: sono esattamente i
casi in cui vale il protocollo della sezione "Quando i pattern non bastano".

## Accesso a database (SQL Server)

Alcuni progetti (WinForms, Web Forms, MVC) si appoggiano a SQL Server. Sono stati osservati tre stili distinti, mai mischiati all'interno
dello stesso progetto — quando crei un nuovo progetto con persistenza dati, scegli lo stile
in base al tipo di progetto (WinForms vs Web Forms vs MVC) seguendo questi pattern.

### Istanza SQL Server da usare

Sulla macchina possono essere installate più istanze SQL Server. Scrivi qui **quale
istanza usare di default** (es. `.\SQLEXPRESS`) e usala in ogni nuova connection string,
sia nei `Web.config` sia nel codice ADO.NET (`SqlConnectionStringBuilder.DataSource`).
Usa un'istanza diversa **solo** se l'utente lo specifica esplicitamente.

Perché la regola esiste: istanze diverse possono contenere database con lo **stesso nome**
ma vuoti. Se ti colleghi a quella sbagliata ottieni errori "oggetto non trovato" che
sembrano bug di codice e invece sono solo la connessione sbagliata — è una mezza giornata
persa a cercare un bug che non c'è.

### WinForms — DAL manuale con ADO.NET (`Microsoft.Data.SqlClient`)

- Package NuGet `Microsoft.Data.SqlClient` (non il vecchio `System.Data.SqlClient`).
- Una classe `<Entità>DAL` (`internal`) dedicata per entità, separata da model e form.
  Contiene una `SqlConnectionStringBuilder` privata costruita nel costruttore (`DataSource`,
  `UserID`, `Password`, `InitialCatalog`, `TrustServerCertificate = true`) — hardcoded nel
  codice, non letta da file di configurazione.
- Ogni metodo apre e chiude la propria `using (SqlConnection ...)` / `using (SqlCommand ...)`
  (nessuna connessione condivisa o riutilizzata tra metodi).
- Metodi CRUD con naming fisso: `Elenco()` → `List<T>`, `Dettaglio(id)` → `T?` (nullable,
  singolo record), `Nuovo(T)` / `Modifica(T)` / `Elimina(T)` → `bool` (`true` se
  `rows == 1`).
- Insert/Update/Delete **sempre parametrizzati**:
  `command.Parameters.Add("@Nome", SqlDbType.VarChar, 50).Value = ...`.
- Le Form (code-behind) istanziano il DAL direttamente (`new StudentiDAL()`); nessuna
  dependency injection, nessun layer di servizio intermedio.
- Pattern MDI: una Form contenitore (`MDI : Form`) con voci di menu che aprono le form figlie
  impostando `MdiParent = this`.

### ASP.NET Web Forms — `SqlDataSource` dichiarativo (nessun ADO.NET manuale)

- Questi progetti targettano **.NET Framework 4.7.2** (non net10.0) — eccezione alla
  convenzione generale del resto della cartella.
- Tutta la logica CRUD vive nel markup `.aspx` tramite `<asp:SqlDataSource>` con
  `SelectCommand`/`InsertCommand`/`UpdateCommand`/`DeleteCommand` parametrizzati (`@Nome` +
  `<asp:Parameter>`/`<asp:QueryStringParameter>`), legata a `<asp:GridView>`/
  `<asp:DetailsView>` per la visualizzazione. Il code-behind (`.aspx.cs`) resta quasi vuoto
  (solo `Page_Load` vuoto).
- Connection string sempre in `Web.config` sotto `<connectionStrings>`, referenziata dal
  markup con `<%$ ConnectionStrings:NomeConnectionString %>`. Naming osservato:
  `DefaultConnection` oppure `<NomeProgetto>ConnectionString`.
- Layout condiviso tramite `Site.Master` + `ContentPlaceHolder` (`MainContent`).
- Pattern drill-down gerarchico (es. Regione → Provincia → Comune) con
  `INNER JOIN` nella `SelectCommand` e navigazione tramite query string
  (`~/Dettaglio?id={0}`).
- I progetti Web Forms creati da template Visual Studio portano con sé lo scaffolding
  Identity/Membership completo (cartella `Account/`, `Startup.Auth.cs`,
  `IdentityConfig.cs`, Entity Framework) anche quando il progetto reale è solo 1-2 pagine —
  è boilerplate del template, non codice scritto per il progetto; non va replicato per i
  nuovi progetti salvo richiesta esplicita.

### ASP.NET Core MVC — EF Core Database-First (scaffolding)

- Questi progetti sono generati con `Scaffold-DbContext` (pacchetto
  `Microsoft.VisualStudio.Web.CodeGeneration.Design`) a partire da un database SQL Server
  già esistente: nessun DAL manuale — il controller inietta il `DbContext` via costruttore
  (DI) e lo interroga direttamente con LINQ/EF Core.
- Nuovo progetto in Visual Studio: template **ASP.NET Core Web App (Model-View-Controller)**
  con tipo di autenticazione **"Individual Accounts"** (non "None", non "Microsoft identity
  platform") — è quello che porta con sé lo scaffolding Identity descritto più sotto.
- Le classi generate dallo scaffolding EF (model in `Models/`, il `DbContext`) sono
  **eccezioni esplicite** alle convenzioni generali di questo file — non vanno "corrette"
  per allinearle, sono output rigenerabile del tool:
  - `public partial class` invece di `internal`.
  - namespace file-scoped (`namespace NomeProgetto.NomeCartella;`) invece del namespace a
    blocco `{ }`.
- Quando lo scaffolding genera nomi in automatico con la "s" finale (pluralizzazione
  inglese di default), rinominali sempre con il plurale italiano corretto.
- Nei progetti ASP.NET Core MVC, `Program.cs` usa sempre lo stile
  `public class Program { static void Main(string[] args) { ... } }` dentro
  `namespace NomeProgetto` (template ASP.NET Core Web) — diverso da `internal class Program`
  previsto dalla convenzione generale per i "programmi con classi/OOP".
- `DbContext` (naming `<NomeDb>Context`, es. `GamContext`): due costruttori (uno vuoto, uno
  con `DbContextOptions<T>`) e un `OnConfiguring` con connection string hardcoded generata
  dal tool — fallback usato quando `Program.cs` chiama
  `AddDbContext<T>(options => options.UseSqlServer())` senza passare la connection string
  esplicitamente. Il `DbContext` di Identity (`ApplicationDbContext`) riceve invece la
  connection string esplicitamente da `appsettings.json` — è normale che nello stesso
  progetto coesistano due `DbContext` con questa differenza.
- Relazioni configurate in `OnModelCreating` via Fluent API: `ToTable(...)`,
  `HasColumnName(...)` quando la colonna SQL non combacia col nome della proprietà C#,
  `HasOne(...).WithMany(...)` per le FK 1-N, `UsingEntity<Dictionary<string, object>>(...)`
  per le relazioni N-N esplicite, `HasNoKey().ToView(...)` per mappare una vista SQL.
  Navigation properties sempre `virtual`.
- Controller CRUD scaffolded: azioni `async Task<IActionResult>` per
  Index/Details/Create/Edit/Delete; sui `POST` di Create/Edit, `[Bind("Prop1,Prop2,...")]`
  esplicito (anti-overposting) e `[ValidateAntiForgeryToken]`; per le FK, una `SelectList`
  in `ViewData["NomeProprietaFk"]`; in `Edit`, un metodo privato `<Entità>Exists(id)` per
  gestire `DbUpdateConcurrencyException`.
- Ricerca/filtro in `Index`: parametro opzionale `string? searchTesto`, query con
  `.AsQueryable()` + `.Where(...)` condizionale solo se il filtro è valorizzato, risultato
  limitato con `.Take(20)`, valore del filtro passato alla view via `ViewBag`.
- Viste CRUD scaffolded: `asp-for`/`asp-validation-for`, `asp-validation-summary="ModelOnly"`,
  classi Bootstrap `form-group`/`text-danger`, dropdown FK con `asp-items="ViewBag.NomeFk"`,
  validazione client-side con `RenderPartialAsync("_ValidationScriptsPartial")`.
- Come per i Web Forms, portano con sé lo scaffolding Identity completo
  (`ApplicationDbContext : IdentityDbContext`, `AddDefaultIdentity<IdentityUser>`) anche
  quando il lavoro riguarda solo il secondo `DbContext` applicativo — boilerplate del
  template, non va replicato/rimosso salvo richiesta esplicita.
- Istanza SQL Server: vale la regola generale di questo file — `.\SQLEXPRESS`.

#### Procedura consigliata (nuovo progetto MVC + EF Core Database-First)

1. Crea il progetto: template "ASP.NET Core Web App (Model-View-Controller)",
   autenticazione "Individual Accounts".
2. In `appsettings.json` correggi `DefaultConnection`, sostituendo `Server`/`Database` con
   `.\SQLEXPRESS`/nome db, aggiungendo in fondo `TrustServerCertificate=True`.
3. Ricompila (tasto destro sul progetto → Compila) per controllare errori nascosti.
4. Scaffolding dei modelli: Strumenti → Gestione pacchetti NuGet → Console di Gestione
   pacchetti NuGet →
   `Scaffold-DbContext "<connection string>" Microsoft.EntityFrameworkCore.SqlServer
   -OutputDir Models`.
   ⚠️ La connection string di `appsettings.json` è in formato JSON (`Server=.\\SQLEXPRESS`,
   doppio backslash per l'escaping) — nel comando di Package Manager Console va incollata
   con un solo backslash (`Server=.\SQLEXPRESS`), altrimenti resta letterale nella stringa
   e lo scaffolding punta all'istanza sbagliata.
5. Rinomina subito i nomi generati con la "s" finale (pluralizzazione inglese di default)
   col plurale italiano corretto — sul `DbContext` (`DbSet`) e ovunque compaiano.
6. Aggiungi il controller: tasto destro su `Controllers` → Aggiungi → Controller →
   "Controller MVC con visualizzazioni, che usa Entity Framework" (l'opzione più lunga
   delle 3) → scegli la classe modello e il `DbContext` applicativo (mai
   `ApplicationDbContext`, riservato a Identity).
7. ⚠️ Ripeti il controllo del punto 5 anche dopo il punto 6: la wizard del controller
   pluralizza di nuovo in inglese il nome della classe entità, indipendentemente dal
   `DbSet` già rinominato — controlla e correggi anche nome del controller, cartella
   `Views/` e route — è un passaggio che si dimentica facilmente, e il risultato è un
   controller e una cartella `Views/` rimasti con la "s" inglese.
8. In `Program.cs`, verifica/aggiungi `builder.Services.AddDbContext<NomeContext>(options
   => options.UseSqlServer());` — funziona grazie alla connection string hardcoded che lo
   scaffolding genera nell'`OnConfiguring` del `DbContext`.
9. Nelle view CRUD generate, traduci in italiano sia `ViewData["Title"]` sia l'`<h1>`
   corrispondente (nello scaffold di default sono entrambi testo statico, non collegati
   dinamicamente).
10. Nei model, aggiungi `[DisplayName("...")]` dove serve un'etichetta diversa dal nome
    della proprietà (es. `Opera.cs`).
11. ⚠️ Aggiungi a mano un link al nuovo controller nel menu di `Views/Shared/_Layout.cshtml`
    (`<li class="nav-item"><a asp-controller="..." asp-action="Index">...</a></li>`): né la
    wizard di Visual Studio né il tool CLI toccano `_Layout.cshtml` — senza questo passaggio
    il CRUD generato è raggiungibile solo digitando l'URL a mano, non appare da nessuna parte
    nel sito.

Trappole aggiuntive su pluralizzazione e rinomine:

- ⚠️ La pluralizzazione automatica dello scaffolding non produce solo nomi con la "s"
  finale in stile inglese — a volte genera nomi pseudo-latini sbagliati che non finiscono
  affatto per "s" (es. `Provincia` → classe `Provincium`). Controlla ogni nome generato
  per correttezza in italiano, non solo quelli con la "s".
- ⚠️ Se un'entità è referenziata da più entità diverse (es. `Comune` ha FK verso
  `Provincia`, `ZonaAltimetrica` e `ZonaMontana`), ognuna di queste ha una **propria**
  collection navigation separata verso di essa, generata con lo stesso nome sbagliato
  (es. `Comunes` in tutte e tre) — rinominarla in una sola non basta: va corretta in
  ognuna, altrimenti resta un'incoerenza silenziosa (compila comunque, non dà errore).
- ⚠️ Prima di rinominare un `DbSet`/una proprietà generata dallo scaffolding, controlla se
  l'entità ha già un `entity.ToTable("NomeTabella")` esplicito in `OnModelCreating`. Se non
  ce l'ha, il nome coincide per convenzione col nome reale della tabella — rinominarlo
  senza aggiungere `ToTable(...)` rompe il mapping e produce un `SqlException` **solo a
  runtime** ("nome di oggetto non valido"), non in fase di build.

#### Se scaffoldi da terminale (CLI) invece che da Visual Studio

- `dotnet new mvc -au Individual` da solo genera **SQLite**, non SQL Server — serve
  `-uld`/`--use-local-db` per allinearsi al comportamento della wizard di VS.
- `dotnet new` non crea né il file soluzione né la sottocartella di progetto nidificata
  che Visual Studio genera sempre — vanno creati a mano:
  `dotnet new sln -n "NomeProgetto" --format slnx` seguito da
  `dotnet sln add "NomeProgetto/NomeProgetto.csproj"`.

### Filtri e ricerche: sempre parametrizzati

Vale anche per le `SELECT`, non solo per insert/update/delete. Nel DAL ADO.NET si usa
`command.Parameters.Add("@Nome", SqlDbType.VarChar, 50).Value = ...`; nei Web Forms si mette
`@Nome` nella `SelectCommand` e si passa il valore con `SelectParameters` (preceduto da
`SelectParameters.Clear()`, altrimenti i parametri si accumulano a ogni postback). I
caratteri jolly `%` della `like` vanno nel valore del parametro, mai nel testo della query.

Non si concatena mai testo digitato dall'utente dentro una query: sarebbe SQL injection
(chi scrive nella casella di ricerca può far eseguire al database comandi non previsti).
Non è una validazione dei dati: è il modo normale di scrivere la query.

## Naming e commenti

- Nomi di variabili brevi, in italiano, camelCase (`n1`, `qi`, `contaPositivi`).
- Commenti in italiano, spesso esplicativi/didattici sul *perché* di una scelta
  (es. perché `do-while` invece di `while`).
- Nessun test automatico, nessun package NuGet oltre l'SDK di base.

---

# Quando i pattern non bastano

Se per fare il lavoro ti servirebbe qualcosa che in questo file non c'è,
**fermati prima di scrivere o modificare qualunque riga** e chiedi.

Non è una formalità burocratica: è il modo in cui l'utente controlla cosa entra
nel suo codice e cosa deve ancora imparare. Applicare un pattern nuovo di
nascosto gli toglie esattamente la cosa che gli serve.

**Tutto questo vale solo per il C#.** Il vocabolario noto riguarda il linguaggio
C# e le librerie .NET. CSS, JavaScript, HTML e markup delle view (`.cshtml`,
`.aspx`) sono fuori per scelta dell'utente: lì scrivi quello che serve, senza
fermarti a chiedere e senza scrivere nessuna voce nel registro.

## Quando serve la conferma

Fermati e chiedi se stai per:

- usare un costrutto del linguaggio che qui non compare (es. LINQ scritto a mano,
  `is X x`, `switch` expression, tuple, `record`, generics tuoi, `async/await`
  scritto da te, delegate o eventi);
- aggiungere un package NuGet non già in uso;
- introdurre un livello di architettura non descritto (repository, classi di
  servizio, dependency injection fatta a mano, AutoMapper...);
- cambiare lo stile di `Program.cs` rispetto a quello previsto per quel tipo di
  programma;
- aggiungere validazioni, `try/catch` o controlli sull'input non richiesti;
- cambiare l'organizzazione di file e progetti rispetto a quella documentata.

## Quando NON serve

Non fermarti per queste, altrimenti diventi ingestibile:

- codice prodotto dagli strumenti (scaffolding EF Core, wizard controller/viste,
  template di Visual Studio): sono già trattati come eccezione esplicita;
- riusare un pattern già documentato in un contesto nuovo;
- scrivere CSS, JavaScript, HTML o markup di una view: sono fuori dal
  vocabolario noto per scelta (vedi sopra);
- rinominare, tradurre in italiano, riformattare, sistemare l'indentazione;
- correggere un errore di compilazione usando costrutti già noti.

Nel dubbio, chiedi: costa un messaggio, mentre un pattern sconosciuto infilato
nel codice costa all'utente un pezzo di programma che non sa spiegare.

## Come chiedere

In italiano, breve, con codice concreto. Copri questi quattro punti e poi
**aspetta la risposta**, senza toccare il codice nel frattempo:

1. **Cosa servirebbe** — il nome del costrutto e tre-sei righe di codice che lo
   mostrano nel contesto reale, non un esempio astratto.
2. **Perché i pattern noti non bastano** — cosa non riesci a fare con quelli.
3. **L'alternativa dentro il vocabolario noto**, se esiste, e cosa costa (più
   righe? codice ripetuto? meno leggibile?). Se non esiste, dillo chiaramente.
4. **La domanda diretta**: procedo col pattern nuovo o con l'alternativa?

Se l'utente dice di no, implementa con l'alternativa nota. Se un'alternativa non
esiste, non inventare: spiegalo e decidete insieme come ridurre la richiesta.

## Il registro del debito di conoscenza

**Qualunque sia la risposta — sì o no — scrivi la voce.** È il punto di tutto il
meccanismo: il file serve a imparare, quindi anche un pattern rifiutato va
spiegato, perché l'utente possa capirlo con calma e magari accettarlo la volta
dopo.

File: `<cartella-progetti>\debt-of-knowledge.md`.
Le voci nuove vanno **in fondo**, in ordine cronologico, con questo formato:

~~~markdown
## AAAA-MM-GG — <Nome del pattern, in italiano semplice>

**Dove è successo:** progetto/file e cosa stavamo facendo.

**Cosa avrei usato:**
```csharp
// codice minimo, commentato in italiano
```

**Perché serviva:** cosa non si riusciva a fare con i pattern già noti.

**Alternativa dentro i pattern noti:** cosa si può fare restando nel vocabolario
attuale, e cosa costa. (Oppure: "nessuna", spiegando perché.)

**Come funziona, passo passo:** la spiegazione didattica vera e propria — cosa fa
ogni pezzo, in che ordine, e perché quel costrutto esiste nel linguaggio. È la
parte che l'utente legge per imparare: prenditi lo spazio che serve.

**Esito:** approvato / rifiutato — e cosa è finito davvero nel codice.

**Se lo promuovi nella skill:** sezione di destinazione in `SKILL.md` e paragrafo
già pronto da incollare lì.
~~~

L'ultimo campo non è un vezzo: quando l'utente dirà "questo l'ho imparato,
aggiungilo alle convenzioni", il testo è già scritto e già coerente con lo stile
del resto del file.

Prima di chiedere, dai un'occhiata al registro: se il pattern c'è già, chiedi lo
stesso — una voce lì significa "già spiegato una volta", non "approvato per
sempre" — ma puoi essere breve e rimandare alla voce esistente invece di
rispiegare tutto da capo.

---

# Come spiegare le cose all'utente

- **Sempre in italiano.**
- La prima volta che usi un termine tecnico (dependency injection, overposting,
  scaffolding, migration, SQL injection...), spiegalo in una riga tra parentesi.
  Non dare per scontato il gergo.
- Spiega il **perché**, non solo il cosa: sta ancora imparando, non è un compito
  da chiudere in fretta.
- Frasi corte, esempi concreti presi dal suo codice, non astratti.
- Per le azioni in Visual Studio usa i nomi dei menu **in italiano** (es. "tasto
  destro sul progetto → Aggiungi elemento esistente"): la sua installazione è in
  italiano.
- Quando nomini un file, dai il percorso completo: non dare per scontato che
  sappia dove si trova.
