# Una skill per scrivere C# che sai rileggere

Questa è la skill Claude Code che uso quando scrivo C#/.NET, insieme al registro
che le sta accanto. La pubblico perché il meccanismo mi sembra riutilizzabile —
le regole di stile, quelle no: sono mie.

## Il problema

Quando chiedi a un assistente di scriverti del codice C#, ti scrive del **buon**
codice C#: LINQ, pattern matching, `record`, `async/await`. Se stai imparando,
quel codice è inutile. Non lo sai rileggere, non lo sai spiegare a nessuno, non
lo sai modificare fra un mese. È corretto e non è tuo.

## L'idea: il vocabolario noto

La skill parte da un ribaltamento: **la qualità del codice si misura in quanto è
comprensibile a chi lo riceve**, non in quanto è idiomatico.

Da lì discendono tre pezzi, che sono la parte interessante:

1. **Il vocabolario.** Un elenco esplicito dei costrutti che conosco davvero,
   ricavato dal codice che ho già scritto. L'assistente può usare quelli e basta.

2. **Il protocollo di scostamento.** Se per fare il lavoro servirebbe qualcosa
   che nel vocabolario non c'è, l'assistente si ferma **prima di scrivere** e
   chiede, in quattro punti: cosa servirebbe (con codice vero, non astratto),
   perché i pattern noti non bastano, quale alternativa esiste dentro il
   vocabolario e cosa costa, e la domanda diretta — procedo o no?

3. **Il registro del debito di conoscenza.** Ogni volta che succede, la cosa
   viene scritta in [`debt-of-knowledge.md`](debt-of-knowledge.md) con una
   spiegazione didattica vera — **sia quando dico sì, sia quando dico no**. È il
   punto di tutto: un pattern rifiutato oggi è un pattern da studiare, non un
   pattern da dimenticare.

Ogni voce finisce con il paragrafo già pronto da incollare nelle convenzioni, il
giorno in cui dico "questo l'ho imparato, promuovilo". È quello che fa crescere
la skill da sola, una voce alla volta.

Il risultato è che il codice cresce alla velocità con cui cresco io, e la lista
delle cose che non so ancora è scritta da qualche parte invece di essere un vago
disagio.

**Solo C#.** CSS, JavaScript, HTML e markup delle view sono fuori dal meccanismo
per scelta: quelli li scrivo senza fermarmi a chiedere e senza registrarli.
Altrimenti il registro si riempie di roba che non c'entra con quello su cui sto
lavorando, e a quel punto smetti di rileggerlo.

## Prima di copiare le regole

**`SKILL.md` contiene le mie convenzioni, non buone pratiche C# universali.**
Graffe Allman, `int.Parse` invece di `Convert.ToInt32`, importi formattati con
`{valore:F2}` e la parola "euro", commenti e nomi di variabili in italiano: sono
scelte mie, tarate sul tipo di programmi che scrivo — piccoli, uno scopo ciascuno.
A te serviranno altre regole.

**Copia il meccanismo, riscrivi il vocabolario.** La parte riutilizzabile è come
la skill è costruita, non quello che ci ho scritto dentro.

## Come usarla

1. Copia `SKILL.md` in `.claude/skills/skill-csharp/SKILL.md`, dentro il progetto
   (vale solo lì) oppure in `~/.claude/skills/` (vale ovunque).
2. Crea un `debt-of-knowledge.md` vuoto e correggi il percorso che la skill
   cita nella sezione "Il registro del debito di conoscenza".
3. **Butta via il mio vocabolario e scrivi il tuo.** Il modo più rapido: fatti
   rileggere il codice che hai già scritto e fatti elencare i costrutti che
   compaiono davvero. Quella lista è il tuo punto di partenza.
4. Cerca i segnaposto `<cartella-progetti>` e l'istanza SQL Server nella sezione
   "Istanza SQL Server da usare": sono i due punti dove ho tolto i riferimenti
   alla mia macchina e vanno riempiti con i tuoi.

Il meccanismo non ha niente di specifico del C#: serve solo che il vocabolario sia
scritto e che il registro esista.

## Cosa c'è qui dentro

| File | Cosa contiene |
|---|---|
| [`SKILL.md`](SKILL.md) | La skill: regola del vocabolario, convenzioni di stile, protocollo di scostamento, formato del registro |
| [`debt-of-knowledge.md`](debt-of-knowledge.md) | Il registro vero, con le voci accumulate finora — è la parte che fa capire il meccanismo meglio di qualunque spiegazione |

Nel registro i nomi dei miei progetti sono rimasti: lì servono, perché "dove è
successo" è metà del valore di una voce. In `SKILL.md` invece li ho sostituiti
con segnaposto generici, perché lì sarebbero solo rumore per chi legge.

## Nota

Questa è una fotografia, non una copia viva: la mia versione locale continua a
cambiare ogni volta che promuovo una voce dal registro alle convenzioni.
Riallineo ogni tanto.
