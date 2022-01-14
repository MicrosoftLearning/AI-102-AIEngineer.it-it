---
lab:
  title: Leggere il testo nelle immagini
  module: Module 11 - Reading Text in Images and Documents
ms.openlocfilehash: 0dc45d60e307769ebfde165201b97c4a3ff49675
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625835"
---
# <a name="read-text-in-images"></a>Leggere il testo nelle immagini

Il riconoscimento ottico dei caratteri (OCR) è un subset di Visione artificiale che riguarda la lettura di testo in immagini e documenti. Il servizio **Visione artificiale** prevede due API per la lettura del testo, che verranno esaminate in questo esercizio.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se non è già stato fatto, è necessario clonare il repository di codice per questo corso:

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="provision-a-cognitive-services-resource"></a>Effettuare il provisioning di una risorsa di Servizi cognitivi

Se non è già disponibile nella sottoscrizione, sarà necessario effettuare il provisioning di una risorsa di **Servizi cognitivi**.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Selezionare il pulsante **&#65291;Crea una risorsa**, cercare *Servizi cognitivi* e creare una risorsa di **Servizi cognitivi** con le impostazioni seguenti:
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0
3. Selezionare le caselle di controllo necessarie e creare la risorsa.
4. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.
5. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva saranno necessari l'endpoint e una delle chiavi di questa pagina.

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Preparare l'uso di Computer Vision SDK

In questo esercizio si completerà un'applicazione client parzialmente implementata che usa Computer Vision SDK per leggere il testo.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **20-ocr** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **read-text** e aprire un terminale integrato. Installare quindi il pacchetto Computer Vision SDK eseguendo il comando appropriato per il linguaggio scelto:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```

3. Visualizzare il contenuto della cartella **read-text** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e una **chiave** di autenticazione per la risorsa di Servizi cognitivi. Salvare le modifiche.
4. Si noti che la cartella **read-text** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: read-text.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. Quindi, sotto questo commento aggiungere il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare Computer Vision SDK:

**C#**

```C#
// import namespaces
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```

**Python**

```Python
# import namespaces
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes
from msrest.authentication import CognitiveServicesCredentials
```

5. Nel file di codice dell'applicazione client, nella funzione **Main**, si noti che è stato fornito il codice per caricare le impostazioni di configurazione. Individuare quindi il commento **Authenticate Computer Vision client**. Sotto questo commento aggiungere il codice specifico del linguaggio seguente per creare e autenticare un oggetto client Computer Vision:

**C#**

```C#
// Authenticate Computer Vision client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Authenticate Computer Vision client
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```
    
## <a name="use-the-ocr-api"></a>Usare l'API OCR

L'API **OCR** è un'API di riconoscimento ottico dei caratteri ottimizzata le la lettura di piccole o medie quantità di testo stampato nelle immagini in formato *JPG*, *PNG*, *GIF* e *BMP*. Supporta un'ampia gamma di lingue e, oltre a leggere il testo nell'immagine, può determinare l'orientamento di ogni area di testo e restituire informazioni sull'angolo di rotazione del testo in relazione all'immagine

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione di menu **1**. Questo codice chiama la funzione **GetTextOcr**, passando il percorso di un file di immagine.
2. Nella cartella **read-text/images** aprire **Lincoln.jpg** per visualizzare l'immagine che verrà elaborata dal codice.
3. Nel file di codice individuare la funzione **GetTextOcr** e quindi, sotto il codice esistente che stampa un messaggio nella console, aggiungere il codice seguente:

**C#**

```C#
// Use OCR API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var ocrResults = await cvClient.RecognizePrintedTextInStreamAsync(detectOrientation:false, image:imageData);

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Magenta, 3);

    foreach(var region in ocrResults.Regions)
    {
        foreach(var line in region.Lines)
        {
            // Show the position of the line of text
            int[] dims = line.BoundingBox.Split(",").Select(int.Parse).ToArray();
            Rectangle rect = new Rectangle(dims[0], dims[1], dims[2], dims[3]);
            graphics.DrawRectangle(pen, rect);

            // Read the words in the line of text
            string lineText = "";
            foreach(var word in line.Words)
            {
                lineText += word.Text + " ";
            }
            Console.WriteLine(lineText.Trim());
        }
    }

    // Save the image with the text locations highlighted
    String output_file = "ocr_results.jpg";
    image.Save(output_file);
    Console.WriteLine("Results saved in " + output_file);
}
```

**Python**

```Python
# Use OCR API to read text in image
with open(image_file, mode="rb") as image_data:
    ocr_results = cv_client.recognize_printed_text_in_stream(image_data)

# Prepare image for drawing
fig = plt.figure(figsize=(7, 7))
img = Image.open(image_file)
draw = ImageDraw.Draw(img)

# Process the text line by line
for region in ocr_results.regions:
    for line in region.lines:

        # Show the position of the line of text
        l,t,w,h = list(map(int, line.bounding_box.split(',')))
        draw.rectangle(((l,t), (l+w, t+h)), outline='magenta', width=5)

        # Read the words in the line of text
        line_text = ''
        for word in line.words:
            line_text += word.text + ' '
        print(line_text.rstrip())

# Save the image with the text locations highlighted
plt.axis('off')
plt.imshow(img)
outputfile = 'ocr_results.jpg'
fig.savefig(outputfile)
print('Results saved in', outputfile)
```

4. Esaminare il codice aggiunto alla funzione **GetTextOcr**. Rileva le aree di testo stampato da un file di immagine e per ogni area estrae le righe di testo ed evidenzia la relativa posizione nell'immagine. Estrae quindi le parole in ogni riga e le stampa.
5. Salvare le modifiche e tornare nel terminale integrato per la cartella **read-text**, quindi immettere il comando seguente per eseguire il programma:

**C#**

```
dotnet run
```

*L'output C# può visualizzare avvisi relativi a funzioni asincrone che ora usano l'operatore **await**. È possibile ignorarli.*

**Python**

```
python read-text.py
```

6. Quando richiesto, immettere **1** e osservare l'output, ovvero il testo estratto dall'immagine.
7. Aprire il file **objects.jpg** generato nella stessa cartella del file di codice per visualizzare le righe di testo annotate nell'immagine.

## <a name="use-the-read-api"></a>Usare l'API Lettura

L'API **Lettura** usa un modello di riconoscimento del testo più recente rispetto all'API OCR e offre prestazioni migliori per le immagini più grandi che contengono molto testo. Supporta anche l'estrazione di testo da file *PDF* e può riconoscere sia testo stampato (in più lingue) che testo scritto a mano (in inglese).

L'API **Lettura** usa un modello di operazione asincrona, in cui viene inviata una richiesta per avviare il riconoscimento del testo e l'ID operazione restituito dalla richiesta può essere usato successivamente per controllare lo stato di avanzamento e recuperare i risultati.

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione di menu **2**. Questo codice chiama la funzione **GetTextRead**, passando il percorso di un file di documento PDF.
2. Nella cartella **read-text/images** fare clic con il pulsante destro del mouse su **Rome.pdf** e scegliere **Visualizza in Esplora file**. Quindi in Esplora file aprire il file PDF per visualizzarlo.
3. Nel file di codice in Visual Studio Code individuare la funzione **GetTextRead** e quindi, sotto il codice esistente che stampa un messaggio nella console, aggiungere il codice seguente:

**C#**

```C#
// Use Read API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var readOp = await cvClient.ReadInStreamAsync(imageData);

    // Get the async operation ID so we can check for the results
    string operationLocation = readOp.OperationLocation;
    string operationId = operationLocation.Substring(operationLocation.Length - 36);

    // Wait for the asynchronous operation to complete
    ReadOperationResult results;
    do
    {
        Thread.Sleep(1000);
        results = await cvClient.GetReadResultAsync(Guid.Parse(operationId));
    }
    while ((results.Status == OperationStatusCodes.Running ||
            results.Status == OperationStatusCodes.NotStarted));

    // If the operation was successfuly, process the text line by line
    if (results.Status == OperationStatusCodes.Succeeded)
    {
        var textUrlFileResults = results.AnalyzeResult.ReadResults;
        foreach (ReadResult page in textUrlFileResults)
        {
            foreach (Line line in page.Lines)
            {
                Console.WriteLine(line.Text);
            }
        }
    }
}  
```

**Python**

```Python
# Use Read API to read text in image
with open(image_file, mode="rb") as image_data:
    read_op = cv_client.read_in_stream(image_data, raw=True)

    # Get the async operation ID so we can check for the results
    operation_location = read_op.headers["Operation-Location"]
    operation_id = operation_location.split("/")[-1]

    # Wait for the asynchronous operation to complete
    while True:
        read_results = cv_client.get_read_result(operation_id)
        if read_results.status not in [OperationStatusCodes.running, OperationStatusCodes.not_started]:
            break
        time.sleep(1)

    # If the operation was successfuly, process the text line by line
    if read_results.status == OperationStatusCodes.succeeded:
        for page in read_results.analyze_result.read_results:
            for line in page.lines:
                print(line.text)
```
    
4. Esaminare il codice aggiunto alla funzione **GetTextRead**. Invia una richiesta per un'operazione di lettura e quindi controlla ripetutamente lo stato fino al completamento dell'operazione. Se l'operazione riesce, il codice elabora i risultati scorrendo ogni pagina e quindi ogni riga.
5. Salvare le modifiche e tornare nel terminale integrato per la cartella **read-text**, quindi immettere il comando seguente per eseguire il programma:

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

6. Quando richiesto, immettere **2** e osservare l'output, ovvero il testo estratto dal documento.

## <a name="read-handwritten-text"></a>Leggere il testo scritto a mano

Oltre al testo stampato, l'API **lettura** può estrarre testo scritto a mano in inglese.

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione di menu **3**. Questo codice chiama la funzione **GetTextRead**, passando il percorso di un file di immagine.
2. Nella cartella **read-text/images** aprire **Note.jpg** per visualizzare l'immagine che verrà elaborata dal codice.
3. Nel terminale integrato per la cartella **read-text** immettere il comando seguente per eseguire il programma:

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

4. Quando richiesto, immettere **3** e osservare l'output, ovvero il testo estratto dal documento.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sull'uso del servizio **Visione artificiale** per la lettura di testo, vedere la [documentazione di Visione artificiale](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text).
