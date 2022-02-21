---
lab:
  title: Creare una competenza personalizzata per Ricerca cognitiva di Azure
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: e09dbbc4fb72ae51e911f1440fd29d4917e50a6a
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 01/12/2022
ms.locfileid: "135801368"
---
# <a name="create-a-custom-skill-for-azure-cognitive-search"></a>Creare una competenza personalizzata per Ricerca cognitiva di Azure

Ricerca cognitiva di Azure usa una pipeline di arricchimento di competenze cognitive per estrarre dai documenti i campi generati dall'intelligenza artificiale e includerli in un indice di ricerca. È disponibile un set completo di competenze incorporate utilizzabile. In presenza di un requisito specifico che non viene soddisfatto da queste competenze, è possibile creare una competenza personalizzata.

Questo esercizio mostra come creare una competenza personalizzata che rappresenta in forma tabulare la frequenza delle singole parole in un documento, per generare un elenco delle prime cinque parole più usate che verrà aggiunto a una soluzione di ricerca per Margie's Travel, un'agenzia di viaggi fittizia.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se è già stato clonato il repository di codice **AI-102-AIEngineer** nell'ambiente in cui si sta lavorando a questo lab, aprirlo in Visual Studio Code. In caso contrario, seguire questa procedura per clonarlo ora.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale (non importa quale).
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="create-azure-resources"></a>Creare le risorse di Azure

> **Nota**: se in precedenza è stato completato l'esercizio **[Creare una soluzione di Ricerca cognitiva di Azure](22-azure-search.md)** e le risorse di Azure sono ancora disponibili nella sottoscrizione, è possibile ignorare questa sezione e iniziare dalla sezione **Creare una soluzione di ricerca**. In caso contrario, seguire la procedura seguente per effettuare il provisioning delle risorse di Azure necessarie.

1. In un Web browser aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Visualizzare i **Gruppi di risorse** nella sottoscrizione.
3. Se si usa una sottoscrizione limitata in cui è stato fornito un gruppo di risorse, selezionare il gruppo di risorse per visualizzarne le proprietà. In caso contrario, creare un nuovo gruppo di risorse con il nome desiderato e passare a tale gruppo dopo che è stato creato.
4. Nella pagina **Panoramica** per il gruppo di risorse prendere nota dei valori di **ID sottoscrizione** e **Località**. Questi valori saranno necessari, insieme al nome del gruppo di risorse, nei passaggi successivi.
5. In Visual Studio Code espandere la cartella **23-custom-search-skill** e selezionare **setup.cmd**. Questo script batch verrà usato per eseguire i comandi dell'interfaccia della riga di comando di Azure che consentono di creare le risorse di Azure necessarie.
6. Fare clic con il pulsante destro del mouse sulla cartella **23-custom-search-skill** e scegliere **Apri nel terminale integrato**.
7. Nel riquadro del terminale immettere il comando seguente per stabilire una connessione autenticata alla sottoscrizione di Azure.

    ```
    az login --output none
    ```

8. Quando verrà richiesto, accedere alla sottoscrizione di Azure. Tornare quindi a Visual Studio Code e attendere il completamento del processo di accesso.
9. Eseguire il comando seguente per elencare le località di Azure.

    ```
    az account list-locations -o table
    ```

10. Nell'output individuare il valore **Name** corrispondente alla località del gruppo di risorse (ad esempio per *Stati Uniti orientali* il nome corrispondente è *eastus*).
11. Nello script **setup.cmd** modificare le dichiarazioni di variabili **subscription_id**, **resource_group** e **location** con i valori appropriati per ID sottoscrizione, nome del gruppo di risorse e nome della località in uso. Salvare quindi le modifiche.
12. Nel terminale per la cartella **23-custom-search-skill** immettere il comando seguente per eseguire lo script:

    ```
    setup
    ```

    > **Nota**: il modulo dell'interfaccia della riga di comando di ricerca è in versione di anteprima e potrebbe bloccarsi con l'indicazione *- Running...* visualizzata. Se ciò accade per più di 2 minuti, premere CTRL+C per annullare l'operazione a esecuzione prolungata e quindi selezionare **N** quando viene chiesto se si vuole terminare lo script. L'operazione dovrebbe quindi venire completata correttamente.
    >
    > Se si verifica un errore dello script, assicurarsi di avere eseguito il salvataggio con i nomi delle variabili corretti e riprovare.

13. Al termine dello script, esaminare l'output visualizzato e prendere nota delle informazioni seguenti sulle risorse di Azure (questi valori saranno necessari in un secondo momento):
    - Nome account di archiviazione
    - Stringa di connessione di archiviazione
    - Account Servizi cognitivi
    - Chiave di Servizi cognitivi
    - Endpoint del servizio di ricerca
    - Chiave amministratore del servizio di ricerca
    - Chiave di query del servizio di ricerca

14. Nel portale di Azure aggiornare il gruppo di risorse e verificare che contenga l'account di archiviazione di Azure, la risorsa di Servizi cognitivi di Azure e la risorsa di Ricerca cognitiva di Azure.

## <a name="create-a-search-solution"></a>Creare una soluzione di ricerca

Ora che le risorse di Azure necessarie sono disponibili, è possibile creare una soluzione di ricerca costituita dai componenti seguenti:

- Un'**origine dati** che fa riferimento ai documenti nel contenitore di archiviazione di Azure.
- Un **set di competenze** che definisce una pipeline di arricchimento delle competenze per estrarre dai documenti i campi generati dall'intelligenza artificiale.
- Un **indice** che definisce un set di record di documenti in cui è possibile eseguire ricerche.
- Un **indicizzatore** che estrae i documenti dall'origine dati, applica il set di competenze e popola l'indice.

In questo esercizio si userà l'interfaccia REST di Ricerca cognitiva di Azure per creare questi componenti inviando richieste JSON.

1. In Visual Studio Code, nella cartella **23-custom-search-skill** espandere la cartella **create-search** e selezionare **data_source.json**. Questo file contiene una definizione JSON per un'origine dati denominata **margies-custom-data**.
2. Sostituire il segnaposto **YOUR_CONNECTION_STRING** con la stringa di connessione per l'account di archiviazione di Azure, che sarà simile alla seguente:

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *È possibile trovare la stringa di connessione nella pagina **Chiavi di accesso** per l'account di archiviazione nel portale di Azure.*

3. Salvare e chiudere il file JSON aggiornato.
4. Nella cartella **create-search** aprire **skillset.json**. Questo file contiene una definizione JSON per un set di competenze denominato **margies-custom-data**.
5. Nella parte superiore della definizione del set di competenze, nell'elemento **cognitiveServices** sostituire il segnaposto **YOUR_COGNITIVE_SERVICES_KEY** con una delle chiavi per le risorse di Servizi cognitivi.

    *È possibile trovare le chiavi nella pagina **Chiavi ed endpoint** della risorsa di Servizi cognitivi nel portale di Azure.*

6. Salvare e chiudere il file JSON aggiornato.
7. Nella cartella **create-search** aprire **index.json**. Questo file contiene una definizione JSON per un indice denominato **margies-custom-index**.
8. Esaminare il codice JSON per l'indice e quindi chiudere il file senza apportare modifiche.
9. Nella cartella **create-search** aprire **indexer.json**. Questo file contiene una definizione JSON per un indicizzatore denominato **margies-custom-indexer**.
10. Esaminare il codice JSON per l'indicizzatore e quindi chiudere il file senza apportare modifiche.
11. Nella cartella **create-search** aprire **create-search.cmd**. Questo script batch usa l'utilità cURL per inviare le definizioni JSON all'interfaccia REST per la risorsa di Ricerca cognitiva di Azure.
12. Sostituire le variabili segnaposto **YOUR_SEARCH_URL** e **YOUR_ADMIN_KEY** con l'**URL** e una delle **chiavi amministratore** della risorsa di Ricerca cognitiva di Azure.

    *È possibile trovare questi valori nelle pagine **Panoramica** e **Chiavi** per la risorsa di Ricerca cognitiva di Azure nel portale di Azure.*

13. Salvare il file batch aggiornato.
14. Fare clic con il pulsante destro del mouse sulla cartella **create-search** e scegliere **Apri nel terminale integrato**.
15. Nel riquadro del terminale per la cartella **create-search** immettere il comando seguente per eseguire lo script batch.

    ```
    create-search
    ```

16. Al termine dello script, nella pagina della risorsa di Ricerca cognitiva di Azure nel portale di Azure selezionare la pagina **Indicizzatori** e attendere il completamento del processo di indicizzazione.

    *È possibile selezionare **Aggiorna** per tenere traccia dello stato di avanzamento dell'operazione di indicizzazione. Il completamento può richiedere circa un minuto.*

## <a name="search-the-index"></a>Eseguire ricerche nell'indice

Ora è possibile eseguire ricerche nell'indice disponibile.

1. Nella parte superiore del pannello della risorsa di Ricerca cognitiva di Azure selezionare **Esplora ricerche**.
2. Nella casella **Stringa di query** di Esplora ricerche immettere la stringa di query seguente e quindi selezionare **Cerca**.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    La query recupera gli elementi **url**, **sentiment** e **keyphrases** per tutti i documenti in cui è presente la parola *London*, il cui autore è *Reviewer* e che hanno un'etichetta di **sentiment** equivalente a "positive" (in altre parole, recensioni positive che parlano di Londra)

## <a name="create-an-azure-function-for-a-custom-skill"></a>Creare una funzione di Azure per una competenza personalizzata

La soluzione di ricerca include una serie di competenze cognitive incorporate che arricchiscono l'indice con le informazioni dei documenti, ad esempio i punteggi del sentiment e gli elenchi di frasi chiave visualizzate nell'attività precedente.

È possibile migliorare ulteriormente l'indice creando competenze personalizzate. Ad esempio, potrebbe essere utile identificare le parole usate più di frequente in ogni documento, ma non c’è una competenza predefinita che offre questa funzionalità.

Per implementare la funzionalità di conteggio delle parole come competenza personalizzata, si creerà una funzione di Azure nel linguaggio preferito.

1. In Visual Studio Code visualizzare la scheda Estensioni di Azure ( **&boxplus;** ) e verificare che l'estensione **Funzioni di Azure** installata. Questa estensione consente di creare e distribuire Funzioni di Azure da Visual Studio Code.
2. Nel riquadro **Funzioni di Azure** della scheda Azure ( **&Delta;** ) creare un nuovo progetto (&#128194;) con le impostazioni seguenti, a seconda del linguaggio preferito:

    ### <a name="c"></a>**C#**

    - **Cartella**: passare a **23-custom-search-skill/C-Sharp/wordcount**
    - **Linguaggio**: C#
    - **Modello**: trigger HTTP
    - **Nome funzione**: wordcount
    - **Spazio dei nomi**: margies.search
    - **Livello di autorizzazione**: Funzione

    ### <a name="python"></a>**Python**

    - **Cartella**: passare a **23-custom-search-skill/Python/wordcount**
    - **Linguaggio**: Python
    - **Ambiente virtuale**: ignorare l'ambiente virtuale
    - **Modello**: trigger HTTP
    - **Nome funzione**: wordcount
    - **Livello di autorizzazione**: Funzione

    *Sovrascrivere **launch.json** se viene richiesto.*

3. Tornare alla scheda **Explorer** ( **&#128461;** ) e verificare che la cartella **wordcount** contenga ora i file di codice per la funzione di Azure.

    *Se si sceglie Python, i file di codice potrebbero essere in una sottocartella, denominata **wordcount***

4. Il file di codice principale per la funzione dovrebbe essere stato aperto automaticamente. In caso contrario, aprire il file appropriato per il linguaggio scelto:
    - **C#** : wordcount.cs
    - **Python**: \_\_init\_\_&#46;py

5. Sostituire l'intero contenuto del file con il codice seguente per il linguaggio scelto:

### <a name="c"></a>**C#**

```C#
using System.IO;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Newtonsoft.Json;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;
using System.Text.RegularExpressions;
using System.Linq;

namespace margies.search
{
    public static class wordcount
    {

        //define classes for responses
        private class WebApiResponseError
        {
            public string message { get; set; }
        }

        private class WebApiResponseWarning
        {
            public string message { get; set; }
        }

        private class WebApiResponseRecord
        {
            public string recordId { get; set; }
            public Dictionary<string, object> data { get; set; }
            public List<WebApiResponseError> errors { get; set; }
            public List<WebApiResponseWarning> warnings { get; set; }
        }

        private class WebApiEnricherResponse
        {
            public List<WebApiResponseRecord> values { get; set; }
        }

        //function for custom skill
        [FunctionName("wordcount")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)]HttpRequest req, ILogger log)
        {
            log.LogInformation("Function initiated.");

            string recordId = null;
            string originalText = null;

            string requestBody = new StreamReader(req.Body).ReadToEnd();
            dynamic data = JsonConvert.DeserializeObject(requestBody);

            // Validation
            if (data?.values == null)
            {
                return new BadRequestObjectResult(" Could not find values array");
            }
            if (data?.values.HasValues == false || data?.values.First.HasValues == false)
            {
                return new BadRequestObjectResult("Could not find valid records in values array");
            }

            WebApiEnricherResponse response = new WebApiEnricherResponse();
            response.values = new List<WebApiResponseRecord>();
            foreach (var record in data?.values)
            {
                recordId = record.recordId?.Value as string;
                originalText = record.data?.text?.Value as string;

                if (recordId == null)
                {
                    return new BadRequestObjectResult("recordId cannot be null");
                }

                // Put together response.
                WebApiResponseRecord responseRecord = new WebApiResponseRecord();
                responseRecord.data = new Dictionary<string, object>();
                responseRecord.recordId = recordId;
                responseRecord.data.Add("text", Count(originalText));

                response.values.Add(responseRecord);
            }

            return (ActionResult)new OkObjectResult(response); 
        }


            public static string RemoveHtmlTags(string html)
        {
            string htmlRemoved = Regex.Replace(html, @"<script[^>]*>[\s\S]*?</script>|<[^>]+>| ", " ").Trim();
            string normalised = Regex.Replace(htmlRemoved, @"\s{2,}", " ");
            return normalised;
        }

        public static List<string> Count(string text)
        {
            
            //remove html elements
            text=text.ToLowerInvariant();
            string html = RemoveHtmlTags(text);
            
            //split into list of words
            List<string> list = html.Split(" ").ToList();
            
            //remove any non alphabet characters
            var onlyAlphabetRegEx = new Regex(@"^[A-z]+$");
            list = list.Where(f => onlyAlphabetRegEx.IsMatch(f)).ToList();

            //remove stop words
            string[] stopwords = { "", "i", "me", "my", "myself", "we", "our", "ours", "ourselves", "you", 
                    "you're", "you've", "you'll", "you'd", "your", "yours", "yourself", 
                    "yourselves", "he", "him", "his", "himself", "she", "she's", "her", 
                    "hers", "herself", "it", "it's", "its", "itself", "they", "them", 
                    "their", "theirs", "themselves", "what", "which", "who", "whom", 
                    "this", "that", "that'll", "these", "those", "am", "is", "are", "was",
                    "were", "be", "been", "being", "have", "has", "had", "having", "do", 
                    "does", "did", "doing", "a", "an", "the", "and", "but", "if", "or", 
                    "because", "as", "until", "while", "of", "at", "by", "for", "with", 
                    "about", "against", "between", "into", "through", "during", "before", 
                    "after", "above", "below", "to", "from", "up", "down", "in", "out", 
                    "on", "off", "over", "under", "again", "further", "then", "once", "here", 
                    "there", "when", "where", "why", "how", "all", "any", "both", "each", 
                    "few", "more", "most", "other", "some", "such", "no", "nor", "not", 
                    "only", "own", "same", "so", "than", "too", "very", "s", "t", "can", 
                    "will", "just", "don", "don't", "should", "should've", "now", "d", "ll",
                    "m", "o", "re", "ve", "y", "ain", "aren", "aren't", "couldn", "couldn't", 
                    "didn", "didn't", "doesn", "doesn't", "hadn", "hadn't", "hasn", "hasn't", 
                    "haven", "haven't", "isn", "isn't", "ma", "mightn", "mightn't", "mustn", 
                    "mustn't", "needn", "needn't", "shan", "shan't", "shouldn", "shouldn't", "wasn", 
                    "wasn't", "weren", "weren't", "won", "won't", "wouldn", "wouldn't"}; 
            list = list.Where(x => x.Length > 2).Where(x => !stopwords.Contains(x)).ToList();
            
            //get distict words by key and count, and then order by count.
            var keywords = list.GroupBy(x => x).OrderByDescending(x => x.Count());
            var klist = keywords.ToList();

            // return the top 10 words
            var numofWords = 10;
            if(klist.Count<10)
                numofWords=klist.Count;
            List<string> resList = new List<string>();
            for (int i = 0; i < numofWords; i++)
            {
                resList.Add(klist[i].Key);
            }
            return resList;
        }
    }
}
```

## <a name="python"></a>**Python**

```Python
import logging
import os
import sys
import json
from string import punctuation
from collections import Counter
import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Wordcount function initiated.')

    # The result will be a "values" bag
    result = {
        "values": []
    }
    statuscode = 200

    # We're going to exclude words from this list in the word counts
    stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
                "you're", "you've", "you'll", "you'd", 'your', 'yours', 'yourself', 
                'yourselves', 'he', 'him', 'his', 'himself', 'she', "she's", 'her', 
                'hers', 'herself', 'it', "it's", 'its', 'itself', 'they', 'them', 
                'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
                'this', 'that', "that'll", 'these', 'those', 'am', 'is', 'are', 'was',
                'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
                'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
                'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
                'about', 'against', 'between', 'into', 'through', 'during', 'before', 
                'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
                'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
                'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
                'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
                'only', 'own', 'same', 'so', 'than', 'too', 'very', 's', 't', 'can', 
                'will', 'just', 'don', "don't", 'should', "should've", 'now', 'd', 'll',
                'm', 'o', 're', 've', 'y', 'ain', 'aren', "aren't", 'couldn', "couldn't", 
                'didn', "didn't", 'doesn', "doesn't", 'hadn', "hadn't", 'hasn', "hasn't", 
                'haven', "haven't", 'isn', "isn't", 'ma', 'mightn', "mightn't", 'mustn', 
                "mustn't", 'needn', "needn't", 'shan', "shan't", 'shouldn', "shouldn't", 'wasn', 
                "wasn't", 'weren', "weren't", 'won', "won't", 'wouldn', "wouldn't"]

    try:
        values = req.get_json().get('values')
        logging.info(values)

        for rec in values:
            # Construct the basic JSON response for this record
            val = {
                    "recordId": rec['recordId'],
                    "data": {
                        "text":None
                    },
                    "errors": None,
                    "warnings": None
                }
            try:
                # get the text to be processed from the input record
                txt = rec['data']['text']
                # remove numeric digits
                txt = ''.join(c for c in txt if not c.isdigit())
                # remove punctuation and make lower case
                txt = ''.join(c for c in txt if c not in punctuation).lower()
                # remove stopwords
                txt = ' '.join(w for w in txt.split() if w not in stopwords)
                # Count the words and get the most common 10
                wordcount = Counter(txt.split()).most_common(10)
                words = [w[0] for w in wordcount]
                # Add the top 10 words to the output for this text record
                val["data"]["text"] = words
            except:
                # An error occured for this text record, so add lists of errors and warning
                val["errors"] =[{"message": "An error occurred processing the text."}]
                val["warnings"] = [{"message": "One or more inputs failed to process."}]
            finally:
                # Add the value for this record to the response
                result["values"].append(val)
    except Exception as ex:
        statuscode = 500
        # A global error occurred, so return an error response
        val = {
                "recordId": None,
                "data": {
                    "text":None
                },
                "errors": [{"message": ex.args}],
                "warnings": [{"message": "The request failed to process."}]
            }
        result["values"].append(val)
    finally:
        # Return the response
        return func.HttpResponse(body=json.dumps(result), mimetype="application/json", status_code=statuscode)
```
    
6. Salvare il file aggiornato.
7. Fare clic con il pulsante destro del mouse sulla cartella **wordcount** contenente i file di codice e scegliere **Distribuisci in app per le funzioni**. Distribuire quindi la funzione con le seguenti impostazioni specifiche del linguaggio, accedendo ad Azure se richiesto:

    ### <a name="c"></a>**C#**

    - **Sottoscrizione** (se richiesto): selezionare la sottoscrizione di Azure.
    - **Funzione**: creare una nuova app per le funzioni in Azure (Avanzato)
    - **Nome dell'app per le funzioni**: immettere un nome univoco globale.
    - **Runtime**: .NET Core 3.1
    - **Sistema operativo**: Linux
    - **Piano di hosting**: a consumo
    - **Gruppo di risorse**: gruppo di risorse contenente la risorsa di Ricerca cognitiva di Azure.
        - Nota: se questo gruppo di risorse contiene già un'app Web basata su Windows, non sarà possibile distribuirvi una funzione basata su Linux. Eliminare l'app Web esistente o distribuire la funzione in un gruppo di risorse diverso.
    - **Account di archiviazione**: account di archiviazione in cui sono archiviati i documenti di Margie's Travel.
    - **Application Insights**: ignorare per ora

    *Visual Studio Code distribuirà la versione compilata della funzione, nella sottocartella **bin**, in base alle impostazioni di configurazione nella cartella **.vscode** salvate durante la creazione del progetto di funzione.*

    ### <a name="python"></a>**Python**

    - **Sottoscrizione** (se richiesto): selezionare la sottoscrizione di Azure.
    - **Funzione**: creare una nuova app per le funzioni in Azure (Avanzato)
    - **Nome dell'app per le funzioni**: immettere un nome univoco globale.
    - **Runtime**: Python 3.8
    - **Piano di hosting**: a consumo
    - **Gruppo di risorse**: gruppo di risorse contenente la risorsa di Ricerca cognitiva di Azure.
        - Nota: se questo gruppo di risorse contiene già un'app Web basata su Windows, non sarà possibile distribuirvi una funzione basata su Linux. Eliminare l'app Web esistente o distribuire la funzione in un gruppo di risorse diverso.
    - **Account di archiviazione**: account di archiviazione in cui sono archiviati i documenti di Margie's Travel.
    - **Application Insights**: ignorare per ora

8. Attendere la distribuzione della funzione da parte di Visual Studio Code. Al termine della distribuzione verrà visualizzata una notifica.

## <a name="test-the-function"></a>Testare la funzione

Ora che la funzione è stata distribuita in Azure, è possibile testarla nel portale di Azure.

1. Aprire il [portale di Azure](https://portal.azure.com) e passare al gruppo di risorse in cui è stata creata l'app per le funzioni. Aprire quindi il servizio app per l'app per le funzioni.
2. Nel pannello del servizio app aprire la funzione **wordcount** nella pagina **Funzioni**.
3. Nel pannello della funzione **wordcount** visualizzare la pagina **Codice e test** e aprire il riquadro **Test/Esegui**.
4. Nel riquadro **Test/Esegui** sostituire il **Corpo** esistente con il codice JSON riportato di seguito, che riflette lo schema previsto da una competenza Ricerca cognitiva di Azure in cui i record contenenti dati per uno o più documenti vengono inviati per l'elaborazione:

    ```
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```
    
5. Fare clic su **Esegui** e visualizzare il contenuto della risposta HTTP restituito dalla funzione. Questo riflette lo schema previsto da Ricerca cognitiva di Azure quando si utilizza una competenza, in cui viene restituita una risposta per ogni documento. In tal caso, la risposta è costituita da un massimo di 10 termini in ogni documento, in ordine decrescente in base alla frequenza:

    ```
    {
    "values": [
        {
        "recordId": "a1",
        "data": {
            "text": [
            "tiger",
            "burning",
            "bright",
            "darkness",
            "night"
            ]
        },
        "errors": null,
        "warnings": null
        },
        {
        "recordId": "a2",
        "data": {
            "text": [
            "rain",
            "spain",
            "stays",
            "mainly",
            "plains",
            "thats",
            "youll",
            "find"
            ]
        },
        "errors": null,
        "warnings": null
        }
    ]
    }
    ```

6. Chiudere il riquadro **Test/Esegui** e fare clic su **Recupera URL della funzione** nel pannello della funzione **wordcount**. Copiare quindi l'URL della chiave predefinita negli Appunti. Sarà necessario nella procedura successiva.

## <a name="add-the-custom-skill-to-the-search-solution"></a>Aggiungere la competenza personalizzata alla soluzione di ricerca

A questo punto è necessario includere la funzione come competenza personalizzata nel set di competenze della soluzione di ricerca ed eseguire il mapping dei risultati prodotti a un campo nell'indice. 

1. Nella cartella **23-custom-search-skill/update-search** di Visual Studio Code aprire il file **update-skillset.json**. Contiene la definizione JSON di un set di competenze.
2. Esaminare la definizione del set di competenze. Include le stesse competenze di prima e una nuova competenza **WebApiSkill** denominata **get-top-words**.
3. Modificare la definizione della competenza **get-top-words** per impostare il valore **uri** sull'URL della funzione di Azure copiato negli Appunti durante la procedura precedente, sostituendo **YOUR-FUNCTION-APP-URL**.
4. Nella parte superiore della definizione del set di competenze, nell'elemento **cognitiveServices** sostituire il segnaposto **YOUR_COGNITIVE_SERVICES_KEY** con una delle chiavi per le risorse di Servizi cognitivi.

    *È possibile trovare le chiavi nella pagina **Chiavi ed endpoint** della risorsa di Servizi cognitivi nel portale di Azure.*

5. Salvare e chiudere il file JSON aggiornato.
6. Nella cartella **update-search** aprire **update-index.json**. Questo file contiene la definizione JSON dell'indice **margies-custom-index**, con un campo aggiuntivo denominato **top_words** nella parte inferiore della definizione dell'indice.
7. Esaminare il codice JSON per l'indice e quindi chiudere il file senza apportare modifiche.
8. Nella cartella **update-search** aprire **update-indexer.json**. Questo file contiene una definizione JSON per **margies-custom-indexer**, con un mapping aggiuntivo per il campo personalizzato **top_words**.
9. Esaminare il codice JSON per l'indicizzatore e quindi chiudere il file senza apportare modifiche.
10. Nella cartella **update-search** aprire **update-search.cmd**. Questo script batch usa l'utilità cURL per inviare le definizioni JSON aggiornate all'interfaccia REST per la risorsa di Ricerca cognitiva di Azure.
11. Sostituire le variabili segnaposto **YOUR_SEARCH_URL** e **YOUR_ADMIN_KEY** con l'**URL** e una delle **chiavi amministratore** della risorsa di Ricerca cognitiva di Azure.

    *È possibile trovare questi valori nelle pagine **Panoramica** e **Chiavi** per la risorsa di Ricerca cognitiva di Azure nel portale di Azure.*

12. Salvare il file batch aggiornato.
13. Fare clic con il pulsante destro del mouse sulla cartella **update-search** e scegliere **Apri nel terminale integrato**.
14. Nel riquadro del terminale per la cartella **update-search** immettere il comando seguente per eseguire lo script batch.

    ```
    update-search
    ```

15. Al termine dello script, nella pagina della risorsa di Ricerca cognitiva di Azure nel portale di Azure selezionare la pagina **Indicizzatori** e attendere il completamento del processo di indicizzazione.

    *È possibile selezionare **Aggiorna** per tenere traccia dello stato di avanzamento dell'operazione di indicizzazione. Il completamento può richiedere circa un minuto.*

## <a name="search-the-index"></a>Eseguire ricerche nell'indice

Ora è possibile eseguire ricerche nell'indice disponibile.

1. Nella parte superiore del pannello della risorsa di Ricerca cognitiva di Azure selezionare **Esplora ricerche**.
2. Nella casella **Stringa di query** di Esplora ricerche immettere la stringa di query seguente e quindi selezionare **Cerca**.

    ```
    search=Las Vegas&$select=url,top_words
    ```

    Questa query recupera i campi **url** e **top_words** per tutti i documenti che menzionano *Las Vegas*.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sulla creazione di competenze personalizzate per Ricerca cognitiva di Azure, vedere la [documentazione di Ricerca cognitiva di Azure](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface).
