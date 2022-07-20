---
lab:
  title: Rilevare e analizzare i volti
  module: Module 10 - Detecting, Analyzing, and Recognizing Faces
ms.openlocfilehash: 4f7f366d7f2fb221d4fcb89fdb3a4b21c88b915f
ms.sourcegitcommit: 3de3c160e5b2eae51308cacbd0bcd3dbdbe337a5
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/19/2022
ms.locfileid: "147375792"
---
# <a name="detect-and-analyze-faces"></a>Rilevare e analizzare i volti

La possibilità di rilevare, analizzare e riconoscere i volti umani è una delle principali funzionalità di intelligenza artificiale. In questo esercizio si esamineranno due servizi cognitivi di Azure che è possibile usare per gestire i visi nelle immagini: **Visione artificiale** e **Viso**.

> **Nota**: Dal 21 giugno 2022 le funzionalità dei servizi cognitivi che restituiscono informazioni personali dell'utente finale sono limitate ai clienti a cui è stato concesso l'[accesso limitato](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Inoltre, le funzionalità che deducono lo stato emotivo non sono più disponibili. Queste restrizioni possono influire sull'esecuzione di questo esercizio del lab. Stiamo lavorando per risolvere questo problema, ma nel frattempo è possibile che si riscontrino alcuni errori durante l'esecuzione della procedura seguente. Ci scusiamo per l'inconveniente. Per altre informazioni sulle modifiche apportate da Microsoft e sul relativo motivo, vedere [Misure di sicurezza e investimenti responsabili nell'intelligenza artificiale per il riconoscimento facciale](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se non è già stato fatto, è necessario clonare il repository di codice per questo corso:

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale (non importa quale).
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
4. Attendere il completamento della distribuzione e quindi visualizzare i dettagli della distribuzione.
5. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva saranno necessari l'endpoint e una delle chiavi di questa pagina.

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Preparare l'uso di Computer Vision SDK

In questo esercizio si completerà un'applicazione client parzialmente implementata che usa Computer Vision SDK per analizzare i visi in un'immagine.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **19-face** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **computer-vision** e aprire un terminale integrato. Installare quindi il pacchetto Computer Vision SDK eseguendo il comando appropriato per il linguaggio scelto:

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-computervision==0.7.0
    ```
    
3. Visualizzare il contenuto della cartella **computer-vision** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

4. Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e una **chiave** di autenticazione per la risorsa di Servizi cognitivi. Salvare le modifiche.

5. Si noti che la cartella **computer-vision** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: detect-faces.py

6. Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. Quindi, sotto questo commento aggiungere il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare Computer Vision SDK:

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

## <a name="view-the-image-you-will-analyze"></a>Visualizzare l'immagine che verrà analizzata

In questo esercizio si userà il servizio Visione artificiale per analizzare un'immagine di persone.

1. Nella Visual Studio Code espandere la cartella **computer-vision** e la cartella **images** al suo interno.
2. Selezionare l'immagine **people.jpg** per visualizzarla.

## <a name="detect-faces-in-an-image"></a>Rilevare i visi in un'immagine

A questo punto è possibile usare l'SDK per chiamare il servizio Visione artificiale e rilevare i visi in un'immagine.

1. Nel file di codice dell'applicazione client (**Program.cs** o **detect-faces.py**), nella funzione **Main**, si noti che è stato fornito il codice per caricare le impostazioni di configurazione. Individuare quindi il commento **Authenticate Computer Vision client**. Sotto questo commento aggiungere il codice specifico del linguaggio seguente per creare e autenticare un oggetto client Computer Vision:

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

2. Nella funzione **Main**, sotto il codice appena aggiunto, si noti che il codice specifica il percorso di un file di immagine e quindi lo passa a una funzione denominata **AnalyzeFaces.** Questa funzione non è ancora completamente implementata.

3. Nella funzione **AnalyzeFaces**, sotto il commento **Specify features to be retrieved (faces)** , aggiungere il codice seguente:

    **C#**

    ```C#
    // Specify features to be retrieved (faces)
    List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
    {
        VisualFeatureTypes.Faces
    };
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (faces)
    features = [VisualFeatureTypes.faces]
    ```

4. Nella funzione **AnalyzeFaces**, sotto il commento **Get image analysis**, aggiungere il codice seguente:

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // Get faces
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // Draw and annotate each face
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person at approximately {r.Left}, {r.Top}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

    # Get faces
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person at approximately {}, {}'.format(r.left, r.top)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```

5. Salvare le modifiche e tornare nel terminale integrato per la cartella **computer-vision**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-faces.py
    ```

6. Osservare l'output, che dovrebbe indicare il numero di visi rilevati.
7. Aprire il file **detected_faces.jpg** generato nella stessa cartella del file di codice per visualizzare i visi annotati. In questo caso, il codice ha usato gli attributi del viso per etichettare la località nella parte in alto a sinistra del riquadro e le coordinate del rettangolo delimitatore per disegnare un rettangolo intorno a ogni viso.

## <a name="prepare-to-use-the-face-sdk"></a>Preparare l'uso di Face SDK

Mentre il servizio **Visione artificiale** offre funzionalità di rilevamento del viso di base (oltre a molte altre funzionalità di analisi delle immagini), il servizio **Viso** offre funzionalità più complete per l'analisi e il riconoscimento facciale.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **19-face** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **face-api** e aprire un terminale integrato. Installare quindi il pacchetto Face SDK eseguendo il comando appropriato per il linguaggio scelto:

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.4.1
    ```
    
3. Visualizzare il contenuto della cartella **face-api** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

4. Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e una **chiave** di autenticazione per la risorsa di Servizi cognitivi. Salvare le modifiche.

5. Si noti che la cartella **face-api** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: analyze-faces.py

6. Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. Quindi, sotto questo commento aggiungere il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare Computer Vision SDK:

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. Nella funzione **Main** si noti che è stato fornito il codice per caricare le impostazioni di configurazione. Individuare quindi il commento **Authenticate Face client**. Sotto questo commento aggiungere il codice specifico del linguaggio seguente per creare e autenticare un oggetto **FaceClient**:

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. Nella funzione **Main**, sotto il codice appena aggiunto, si noti che il codice visualizza un menu che consente di chiamare funzioni per esplorare le funzionalità del servizio Viso. Queste funzioni verranno implementate nella parte restante di questo esercizio.

## <a name="detect-and-analyze-faces"></a>Rilevare e analizzare i volti

Una delle funzionalità più importanti del servizio Viso consente di rilevare i visi in un'immagine e determinarne gli attributi, ad esempio posa della testa, sfocatura, presenza di occhiali e così via.

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione **1**. Questo codice chiama la funzione **DetectFaces,** passando il percorso di un file di immagine.
2. Individuare la funzione **DetectFaces** nel file di codice e quindi, sotto il commento **Specify facial features to be retrieved**, aggiungere i codice seguente:

    **C#**

    ```C#
    // Specify facial features to be retrieved
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Occlusion,
        FaceAttributeType.Blur,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.occlusion,
                FaceAttributeType.blur,
                FaceAttributeType.glasses]
    ```

3. Nella funzione **DetectFaces**, sotto il codice appena aggiunto, individuare il commento **Get faces** e aggiungere il codice seguente:

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features, returnFaceId: false);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);
        int faceCount=0;

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            faceCount++;
            Console.WriteLine($"\nFace number {faceCount}");
            
            // Get face properties
            Console.WriteLine($" - Mouth Occluded: {face.FaceAttributes.Occlusion.MouthOccluded}");
            Console.WriteLine($" - Eye Occluded: {face.FaceAttributes.Occlusion.EyeOccluded}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face ID: {face.FaceId}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features,                     return_face_id=False)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'
        face_count = 0

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))

            detected_attributes = face.face_attributes.as_dict()
            if 'blur' in detected_attributes:
                print(' - Blur:')
                for blur_name in detected_attributes['blur']:
                    print('   - {}: {}'.format(blur_name, detected_attributes['blur'][blur_name]))
                    
            if 'occlusion' in detected_attributes:
                print(' - Occlusion:')
                for occlusion_name in detected_attributes['occlusion']:
                    print('   - {}: {}'.format(occlusion_name, detected_attributes['occlusion'][occlusion_name]))

            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face ID: {}'.format(face.face_id)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. Esaminare il codice aggiunto alla funzione **DetectFaces**. Il codice analizza un file di immagine e rileva tutti i visi che contiene, inclusi gli attributi relativi a età, emozioni e presenza di occhiali. Vengono visualizzati i dettagli di ogni viso, con un identificatore univoco assegnato a ogni viso. Inoltre, sull'immagine è indicata la posizione dei visi con un rettangolo delimitatore.
5. Salvare le modifiche e tornare nel terminale integrato per la cartella **face-api**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    *L'output C# può visualizzare avvisi relativi a funzioni asincrone che ora usano l'operatore **await**. È possibile ignorarli.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. Quando richiesto, immettere **1** e osservare l'output, che dovrà includere l'ID e gli attributi di ogni viso rilevato.
7. Aprire il file **detected_faces.jpg** generato nella stessa cartella del file di codice per visualizzare i visi annotati.

## <a name="more-information"></a>Altre informazioni

Sono disponibili diverse funzionalità aggiuntive all'interno del servizio **Viso**, ma secondo lo [standard di IA responsabile](https://aka.ms/aah91ff) queste funzionalità sono soggette ai criteri di Accesso limitato. Queste funzionalità includono l'identificazione, la verifica e la creazione di modelli di riconoscimento facciale. Per altre informazioni e per richiedere l'accesso, vedere [Accesso limitato per i servizi cognitivi](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access).

Per altre formazioni sull'uso del servizio **Visione artificiale** per il rilevamento dei visi, vedere la [documentazione di Visione artificiale](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Per altre informazioni sul servizio **Viso**, vedere la [documentazione di Viso](https://docs.microsoft.com/azure/cognitive-services/face/).
