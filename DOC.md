# Documentazione GDXP v1

Il formato GDXP è formalizzato nel file GDXP-1.json, che contiene il [JSON Schema](https://json-schema.org/) su cui è possibile validare i propri files.

Di seguito qualche esempio con le relative indicazioni sul contenuto ed il formato di ogni attributo. Nota bene: i documenti qui sotto non sono documenti JSON validi, in quanto il formato JSON non contempla i commenti.

## Caso 1: Listino Fornitore

Da utilizzare per scambiare listini tra diverse applicazioni.

```
{
	/*
		Numero di versione GDXP del file.
	*/
	"protocolVersion": "1.0",

	/*
		Data di creazione del file, in formato YYYY-MM-DD
	*/
	"creationDate": "2020-02-23",

	/*
		Il nome dell'applicazione che ha generato il file.
	*/
	"applicationSignature": "GASware Turbo Pro",

	/*
		Informazioni relative al soggetto che ha generato il file. Vengono successivamente
		utilizzate nello scenario di scambio dati bidirezionali, ad esempio tra un RES che
		genera un ordine ed un GAS che comunica le proprie prenotazioni su quell'ordine.
	*/
	"subject": {
		"name": "GAS di Prova",
		"taxCode": "",
		"vatNumber": "",
		"address": {
			"street": "Via Garibaldi 13",
			"locality": "Roma",
			"zipCode": "00153"
		},

		/*
			I contatti (del soggetto che ha generato il file, ma anche del fornitore -
			vedi sotto) sono organizzati in un array che può contenere informazioni in
			numero arbitrario.
			Ciascun contatto è identificato da una tipologia, tra cui
			- phoneNumber: numero di telefono
			- faxNumber: numero di fax
			- mobileNumber: numero di cellulare
			- emailAddress: indirizzo e-mail
			- webSite: sito web
		*/
		"contacts": [
			{
				"type": "emailAddress",
				"value": "gas@prova.it"
			}
		]
	},

	/*
		Il file GDXP è strutturato per contenere potenzialmente diverse informazioni, ad
		esempio più listini o più ordini. Ciascun blocco all'interno dell'array BLOCKS è
		isolato rispetto agli altri.
	*/
	"blocks": [
		{
			"supplier": {
				"name": "Fornitore di Prova",
				"taxCode": "0123456789",

				/*
					Nota bene: la partita IVA del fornitore è di fatto usata
					come chiave globale per identificarlo rispetto agli altri.
					Questo attributo deve obbligatoriamente essere valorizzato.
				*/
				"vatNumber": "9876543210",

				"address": {
					"street": "Via Roma 42",
					"locality": "Milano",
					"zipCode": "20090"
				},

				/*
					Per dettagli sull'array CONTACTS, vedi sopra.
				*/
				"contacts": [
					{
						"type": "emailAddress",
						"value": "info@fornitore.com"
					},
					{
						"type": "phoneNumber",
						"value": "0123456789"
					},
					{
						"type": "webSite",
						"value": "http:\/\/www.example.com"
					}
				],

				/*
					L'array PRODUCTS contiene i prodotti contemplati nel
					listino in oggetto, ogni oggetto al suo interno
					rappresenta un singolo prodotto con tutti i suoi attributi.
				*/
				"products": [
					{
						"name": "Mele",

						/*
							Gli attributi UM (unità di misura) e
							CATEGORY (categoria) sono arbitrari: non
							esiste un catalogo centrale dei possibili
							valori, né tantomeno può esistere data la
							classificazione usata da ogni soggetto
							operante.
						*/
						"um": "Chili",
						"category": "Non Specificato",

						"sku": "M0123",
						"description": "Una mela al giorno leva il medico di torno",
						"orderInfo": {
							/*
								Se diverso da 1, questo valore
								indica il numero di pezzi presenti
								in una singola confezione
								acquistabile dal fornitore.
								Non può essere 0.
							*/
							"packageQty": 1,

							/*
								Se diverso da 0, questo valore
								indica la quantità massima
								ordinabile (per unità di misura)
								da ogni utente.
							*/
							"maxQty": 0,

							/*
								Se diverso da 0, questo valore
								indica la quantità minima
								ordinabile (per unità di misura)
								da ogni utente.
							*/
							"minQty": 0,

							/*
								Se diverso da 0, questo valore
								indica che la quantità ordinabile
								da ogni utente deve essere un
								multiplo di quella indicata.
							*/
							"mulQty": 1,

							/*
								Se diverso da 0, questo valore
								indica la quantità massima
								disponibile del prodotto.
								Significativo soprattutto nel caso
								della descrizione di un ordine.
							*/
							"availableQty": 0,

							/*
								Questo è il prezzo per unità di
								misura.
								Si intende sempre ivato, a
								prescindere dalla presenza
								dell'aliquota IVA (vedi
								l'attributo VATRATE).
							*/
							"umPrice": 0.84,

							/*
								Se diverso da 0, questo valore
								indica il prezzo di trasporto del
								singolo prodotto per unità di
								misura.
							*/
							"shippingCost": 0,

							/*
								Se diverso da 0, questo valore
								indica l'aliquota IVA applicata al
								prezzo del prodotto.
							*/
							"vatRate": 22
						},

						/*
							Booleano che indica se il prodotto
							presente nel listino è effettivamente
							ordinabile (true) o meno (false).
							Usato per trasmettere anche prodotti che,
							per qualsiasi motivo, sono temporaneamente
							non disponibili.
						*/
						"active": true
					},
					{
						"name": "Pere",
						"um": "Cassette",
						"sku": "",
						"category": "Frutta e Verdura",
						"description": "",
						"orderInfo": {
							"packageQty": 1,
							"maxQty": 0,
							"minQty": 0,
							"mulQty": 1,
							"availableQty": 0,
							"umPrice": 7.65,
							"shippingCost": 0,
							"vatRate": 22
						},
						"active": false
					}
				]
			}
		}
	]
}
```

## Raccomandazioni per l'implementazione

In fase di generazione di un nuovo file GDXP si consiglia l'uso delle funzioni di serializzazione JSON già presenti nel proprio ambiente di sviluppo (ad esempio: [json_encode()](https://www.php.net/manual/en/function.json-encode.php) per PHP) per garantire il corretto escaping delle stringhe e la validità formale del documento prodotto.

In fase di importazione di un file GDXP è consigliato mostrare all'utente le informazioni presenti nell'intestazione del file stesso.

È consigliato prevedere strumenti interattivi di importazione, per permettere all'utente di intervenire manualmente e decidere come intervenire nei confronti di fornitori, prodotti, unità di misura e categorie già esistenti nella base dati locale.

