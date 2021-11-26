---
lab:
  title: Analizzare immagini con Visione artificiale
  module: Module 8 - Getting Started with Computer Vision
ms.openlocfilehash: 5b7f15550844e4bc5efbdb3b8ee00d71760f18c9
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625814"
---
# <a name="analyze-images-with-computer-vision"></a>Analizzare immagini con Visione artificiale

Visione artificiale è una funzionalità di intelligenza artificiale che consente ai sistemi software di interpretare l'input visivo mediante l'analisi di immagini. In Microsoft Azure il servizio cognitivo **Visione artificiale** fornisce modelli predefiniti per attività comuni di visione artificiale, tra cui analisi di immagini per suggerire didascalia e tag, rilevamento di oggetti comuni, luoghi di interesse, celebrità, marchi e presenza di contenuti per adulti. È inoltre possibile usare il servizio Visione artificiale per analizzare i colori e i formati delle immagini e per generare immagini di anteprima "ritagliate in modo intelligente".

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se il repository di codice **AI-102-AIEngineer** non è già stato clonato nell'ambiente in cui si sta lavorando a questo lab, seguire questa procedura per clonarlo. In caso contrario, aprire la cartella clonata in Visual Studio Code.

1. Avviare Visual Studio Code.
2. Aprire il pannello (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="provision-a-cognitive-services-resource"></a>Effettuare il provisioning di una risorsa di Servizi cognitivi

Se non è già disponibile nella sottoscrizione, sarà necessario effettuare il provisioning di una risorsa di **Servizi cognitivi**.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Selezionare il pulsante **&#65291;Crea una risorsa**, cercare *Servizi cognitivi* e creare una risorsa di **Servizi cognitivi** con le impostazioni seguenti:
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, è possibile che non si sia autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0
3. Selezionare le caselle di controllo necessarie e creare la risorsa.
4. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.
5. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva saranno necessari l'endpoint e una delle chiavi di questa pagina.

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Preparare l'uso di Computer Vision SDK

In questo esercizio si completerà un'applicazione client parzialmente implementata che usa Computer Vision SDK per analizzare le immagini.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **15-computer-vision** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **image-analysis** e aprire un terminale integrato. Installare quindi il pacchetto Computer Vision SDK eseguendo il comando appropriato per il linguaggio scelto:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```
    
3. Visualizzare il contenuto della cartella **image-analysis** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e una **chiave** di autenticazione per la risorsa di Servizi cognitivi. Salvare le modifiche.
4. Si noti che la cartella **image-analysis** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: image-analysis.py

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
from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
from msrest.authentication import CognitiveServicesCredentials
```
    
## <a name="view-the-images-you-will-analyze"></a>Visualizzare le immagini che verranno analizzate

In questo esercizio si userà il servizio Visione artificiale per analizzare più immagini.

1. Nella Visual Studio Code espandere la cartella **image-analysis** e la cartella **images** al suo interno.
2. Selezionare un file di immagine alla volta per visualizzarlo quindi in Visual Studio Code.

## <a name="analyze-an-image-to-suggest-a-caption"></a>Analizzare un'immagine per suggerire una didascalia

A questo punto è possibile usare l'SDK per chiamare il servizio Visione artificiale e analizzare un'immagine.

1. Nel file di codice dell'applicazione client (**Program.cs** o **image-analysis.py**), nella funzione **Main**, si noti che è stato fornito il codice per caricare le impostazioni di configurazione. Individuare quindi il commento **Authenticate Computer Vision client**. Sotto questo commento aggiungere il codice specifico del linguaggio seguente per creare e autenticare un oggetto client Computer Vision:

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

2. Nella funzione **Main**, sotto il codice appena aggiunto, si noti che il codice specifica il percorso di un file di immagine e quindi lo passa ad altre due funzioni (**AnalyzeImage** e **GetThumbnail**). Queste funzioni non sono ancora completamente implementate.

3. Nella funzione **AnalyzeImage**, sotto il commento **Specify features to be retrieved**, aggiungere il codice seguente:

**C#**

```C#
// Specify features to be retrieved
List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
{
    VisualFeatureTypes.Description,
    VisualFeatureTypes.Tags,
    VisualFeatureTypes.Categories,
    VisualFeatureTypes.Brands,
    VisualFeatureTypes.Objects,
    VisualFeatureTypes.Adult
};
```

**Python**

```Python
# Specify features to be retrieved
features = [VisualFeatureTypes.description,
            VisualFeatureTypes.tags,
            VisualFeatureTypes.categories,
            VisualFeatureTypes.brands,
            VisualFeatureTypes.objects,
            VisualFeatureTypes.adult]
```
    
4. Nella funzione **AnalyzeImage**, sotto il commento **Get image analysis**, aggiungere il codice seguente, inclusi i commenti che indicano dove verrà aggiunto altro codice in seguito:

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // get image captions
    foreach (var caption in analysis.Description.Captions)
    {
        Console.WriteLine($"Description: {caption.Text} (confidence: {caption.Confidence.ToString("P")})");
    }

    // Get image tags


    // Get image categories


    // Get brands in the image


    // Get objects in the image


    // Get moderation ratings
    

}            
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

# Get image description
for caption in analysis.description.captions:
    print("Description: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get image categories 


# Get brands in the image


# Get objects in the image


# Get moderation ratings

```
    
5. Salvare le modifiche e tornare nel terminale integrato per la cartella **image-analysis**, quindi immettere il comando seguente per eseguire il programma con l'argomento **images/street.jpg**:

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. Osservare l'output, che dovrebbe includere una didascalia suggerita per l'immagine **street.jpg**.
7. Eseguire di nuovo il programma, questa volta con l'argomento **images/building.jpg** per visualizzare la didascalia che viene generata per l'immagine **building.jpg**.
8. Ripetere il passaggio precedente per generare una didascalia per il file **images/person.jpg**.

## <a name="get-suggested-tags-for-an-image"></a>Ottenere tag suggeriti per un'immagine

A volte può risultare utile identificare i *tag* rilevanti che forniscono indicazioni sui contenuti di un'immagine.

1. Nella funzione **AnalyzeImage**, sotto il commento **Get image tags**, aggiungere il codice seguente:

**C#**

```C
// Get image tags
if (analysis.Tags.Count > 0)
{
    Console.WriteLine("Tags:");
    foreach (var tag in analysis.Tags)
    {
        Console.WriteLine($" -{tag.Name} (confidence: {tag.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get image tags
if (len(analysis.tags) > 0):
    print("Tags: ")
    for tag in analysis.tags:
        print(" -'{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images** osservando che oltre alla didascalia dell'immagine viene visualizzato un elenco di tag suggeriti.

## <a name="get-image-categories"></a>Ottenere categorie di immagini

Il servizio Visione artificiale può suggerire *categorie* per le immagini e in ogni categoria può identificare luoghi di interesse noti e celebrità.

1. Nella funzione **AnalyzeImage**, sotto il commento **Get image categories (including celebrities and landmarks)** , aggiungere il codice seguente:

**C#**

```C
// Get image categories (including celebrities and landmarks)
List<LandmarksModel> landmarks = new List<LandmarksModel> {};
List<CelebritiesModel> celebrities = new List<CelebritiesModel> {};
Console.WriteLine("Categories:");
foreach (var category in analysis.Categories)
{
    // Print the category
    Console.WriteLine($" -{category.Name} (confidence: {category.Score.ToString("P")})");

    // Get landmarks in this category
    if (category.Detail?.Landmarks != null)
    {
        foreach (LandmarksModel landmark in category.Detail.Landmarks)
        {
            if (!landmarks.Any(item => item.Name == landmark.Name))
            {
                landmarks.Add(landmark);
            }
        }
    }

    // Get celebrities in this category
    if (category.Detail?.Celebrities != null)
    {
        foreach (CelebritiesModel celebrity in category.Detail.Celebrities)
        {
            if (!celebrities.Any(item => item.Name == celebrity.Name))
            {
                celebrities.Add(celebrity);
            }
        }
    }
}

// If there were landmarks, list them
if (landmarks.Count > 0)
{
    Console.WriteLine("Landmarks:");
    foreach(LandmarksModel landmark in landmarks)
    {
        Console.WriteLine($" -{landmark.Name} (confidence: {landmark.Confidence.ToString("P")})");
    }
}

// If there were celebrities, list them
if (celebrities.Count > 0)
{
    Console.WriteLine("Celebrities:");
    foreach(CelebritiesModel celebrity in celebrities)
    {
        Console.WriteLine($" -{celebrity.Name} (confidence: {celebrity.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get image categories (including celebrities and landmarks)
if (len(analysis.categories) > 0):
    print("Categories:")
    landmarks = []
    celebrities = []
    for category in analysis.categories:
        # Print the category
        print(" -'{}' (confidence: {:.2f}%)".format(category.name, category.score * 100))
        if category.detail:
            # Get landmarks in this category
            if category.detail.landmarks:
                for landmark in category.detail.landmarks:
                    if landmark not in landmarks:
                        landmarks.append(landmark)

            # Get celebrities in this category
            if category.detail.celebrities:
                for celebrity in category.detail.celebrities:
                    if celebrity not in celebrities:
                        celebrities.append(celebrity)

    # If there were landmarks, list them
    if len(landmarks) > 0:
        print("Landmarks:")
        for landmark in landmarks:
            print(" -'{}' (confidence: {:.2f}%)".format(landmark.name, landmark.confidence * 100))

    # If there were celebrities, list them
    if len(celebrities) > 0:
        print("Celebrities:")
        for celebrity in celebrities:
            print(" -'{}' (confidence: {:.2f}%)".format(celebrity.name, celebrity.confidence * 100))

```
    
2. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images**, osservando che oltre alla didascalia e ai tag dell'immagine viene visualizzato un elenco di categorie suggerite, insieme ad eventuali celebrità o luoghi di interesse riconosciuti, in particolare nelle immagini **building.jpg** e **person.jpg**.

## <a name="get-brands-in-an-image"></a>Ottenere i marchi in un'immagine

Alcuni marchi sono visivamente riconoscibili dal logo, anche se il nome del marchio non viene visualizzato. Il servizio Visione artificiale è stato sottoposto a training per identificare migliaia di marchi noti.

1. Nella funzione **AnalyzeImage**, sotto il commento **Get brands in the image**, aggiungere il codice seguente:

**C#**

```C
// Get brands in the image
if (analysis.Brands.Count > 0)
{
    Console.WriteLine("Brands:");
    foreach (var brand in analysis.Brands)
    {
        Console.WriteLine($" -{brand.Name} (confidence: {brand.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get brands in the image
if (len(analysis.brands) > 0):
    print("Brands: ")
    for brand in analysis.brands:
        print(" -'{}' (confidence: {:.2f}%)".format(brand.name, brand.confidence * 100))
```
    
2. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images**, osservando eventuali marchi identificati, in particolare nell'immagine **person.jpg**.

## <a name="detect-and-locate-objects-in-an-image"></a>Rilevare e individuare oggetti in un'immagine

*Rilevamento oggetti* è una forma specifica di visione artificiale, che consente di identificare i singoli oggetti in un'immagine e la rispettiva posizione, indicata da un rettangolo delimitatore.

1. Nella funzione **AnalyzeImage**, sotto il commento **Get objects in the image**, aggiungere il codice seguente:

**C#**

```C
// Get objects in the image
if (analysis.Objects.Count > 0)
{
    Console.WriteLine("Objects in image:");

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.Black);

    foreach (var detectedObject in analysis.Objects)
    {
        // Print object name
        Console.WriteLine($" -{detectedObject.ObjectProperty} (confidence: {detectedObject.Confidence.ToString("P")})");

        // Draw object bounding box
        var r = detectedObject.Rectangle;
        Rectangle rect = new Rectangle(r.X, r.Y, r.W, r.H);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.ObjectProperty,font,brush,r.X, r.Y);

    }
    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file);   
}
```

**Python**

```Python
# Get objects in the image
if len(analysis.objects) > 0:
    print("Objects in image:")

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 8))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    color = 'cyan'
    for detected_object in analysis.objects:
        # Print object name
        print(" -{} (confidence: {:.2f}%)".format(detected_object.object_property, detected_object.confidence * 100))
        
        # Draw object bounding box
        r = detected_object.rectangle
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.object_property,(r.x, r.y), backgroundcolor=color)
    # Save annotated image
    plt.imshow(image)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```
    
2. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images**, osservando eventuali oggetti identificati. Dopo ogni esecuzione, vedere il file **objects.jpg** generato nella stessa cartella del file di codice per visualizzare gli oggetti annotati.

## <a name="get-moderation-ratings-for-an-image"></a>Ottenere le classificazioni di moderazione per un'immagine

È possibile che alcune immagini non siano adatte per tutti i destinatari e che sia necessario applicare un certo livello di moderazione per identificare immagini per adulti o violente.

1. Nella funzione **AnalyzeImage**, sotto il commento **Get moderation ratings**, aggiungere il codice seguente:

**C#**

```C
// Get moderation ratings
string ratings = $"Ratings:\n -Adult: {analysis.Adult.IsAdultContent}\n -Racy: {analysis.Adult.IsRacyContent}\n -Gore: {analysis.Adult.IsGoryContent}";
Console.WriteLine(ratings);
```

**Python**

```Python
# Get moderation ratings
ratings = 'Ratings:\n -Adult: {}\n -Racy: {}\n -Gore: {}'.format(analysis.adult.is_adult_content,
                                                                    analysis.adult.is_racy_content,
                                                                    analysis.adult.is_gory_content)
print(ratings)
```
    
2. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images**, osservando le classificazioni per ogni immagine.

> **Nota**: nelle attività precedenti è stato usato un singolo metodo per analizzare l'immagine e quindi è stato aggiunto in modo incrementale codice per analizzare e visualizzare i risultati. L'SDK fornisce inoltre singoli metodi per suggerire didascalie, identificare tag, rilevare oggetti e così via, consentendo quindi di usare il metodo più appropriato per restituire solo le informazioni necessarie, riducendo le dimensioni del payload di dati da restituire. Per altri dettagli, vedere la [documentazione di .NET SDK](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/computervision?view=azure-dotnet) o la [documentazione di Python SDK](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/computervision?view=azure-python).

## <a name="generate-a-thumbnail-image"></a>Generare un'immagine di anteprima

In alcuni casi potrebbe essere necessario creare una versione più piccola di un'immagine denominata *anteprima*, ritagliandola per includere il soggetto visivo principale all'interno delle nuove dimensioni dell'immagine.

1. Nel file di codice trovare la funzione **GetThumbnail** e, sotto il commento **Generate a thumbnail**, aggiungere il codice seguente:

**C#**

```C
// Generate a thumbnail
using (var imageData = File.OpenRead(imageFile))
{
    // Get thumbnail data
    var thumbnailStream = await cvClient.GenerateThumbnailInStreamAsync(100, 100,imageData, true);

    // Save thumbnail image
    string thumbnailFileName = "thumbnail.png";
    using (Stream thumbnailFile = File.Create(thumbnailFileName))
    {
        thumbnailStream.CopyTo(thumbnailFile);
    }

    Console.WriteLine($"Thumbnail saved in {thumbnailFileName}");
}
```

**Python**

```Python
# Generate a thumbnail
with open(image_file, mode="rb") as image_data:
    # Get thumbnail data
    thumbnail_stream = cv_client.generate_thumbnail_in_stream(100, 100, image_data, True)

# Save thumbnail image
thumbnail_file_name = 'thumbnail.png'
with open(thumbnail_file_name, "wb") as thumbnail_file:
    for chunk in thumbnail_stream:
        thumbnail_file.write(chunk)

print('Thumbnail saved in.', thumbnail_file_name)
```
    
2. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images**, aprendo il file **thumbnail.jpg** generato nella stessa cartella del file di codice per ogni immagine.

## <a name="more-information"></a>Altre informazioni

In questo esercizio sono state esplorate alcune funzionalità di analisi e modifica delle immagini del servizio Visione artificiale. Il servizio include anche funzionalità per la lettura di testo, il rilevamento di visi e altre attività di visione artificiale.

Per altre informazioni sull'uso del servizio **Visione artificiale**, vedere la [documentazione di Visione artificiale](https://docs.microsoft.com/azure/cognitive-services/computer-vision/).
