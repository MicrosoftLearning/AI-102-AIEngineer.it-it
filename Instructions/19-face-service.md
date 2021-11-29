---
lab:
  title: Rilevare, analizzare e riconoscere i visi
  module: Module 10 - Detecting, Analyzing, and Recognizing Faces
ms.openlocfilehash: b9565f41eb67b916278508c729860a3471a9e0bd
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625854"
---
# <a name="detect-analyze-and-recognize-faces"></a>Rilevare, analizzare e riconoscere i visi

La possibilità di rilevare, analizzare e riconoscere i visi umani è un'importante funzionalità di intelligenza artificiale. In questo esercizio si esamineranno due servizi cognitivi di Azure che è possibile usare per gestire i visi nelle immagini: **Visione artificiale** e **Viso**.

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
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0
3. Selezionare le caselle di controllo necessarie e creare la risorsa.
4. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.
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
            string annotation = $"Person aged approximately {face.Age}";
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
            annotation = 'Person aged approximately {}'.format(face.age)
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
7. Aprire il file **detected_faces.jpg** generato nella stessa cartella del file di codice per visualizzare i visi annotati. In questo caso, il codice ha usato gli attributi del viso per stimare l'età di ogni persona dell'immagine e le coordinate del rettangolo delimitatore per disegnare un rettangolo intorno a ogni viso.

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

Una delle funzionalità più importanti del servizio Viso consiste nel rilevare i visi in un'immagine e determinarne gli attributi, ad esempio età, espressioni emotive, colore dei capelli, presenza di occhiali e così via.

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione **1**. Questo codice chiama la funzione **DetectFaces,** passando il percorso di un file di immagine.
2. Individuare la funzione **DetectFaces** nel file di codice e quindi, sotto il commento **Specify facial features to be retrieved**, aggiungere i codice seguente:

    **C#**

    ```C#
    // Specify facial features to be retrieved
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Age,
        FaceAttributeType.Emotion,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.age,
                FaceAttributeType.emotion,
                FaceAttributeType.glasses]
    ```

3. Nella funzione **DetectFaces**, sotto il codice appena aggiunto, individuare il commento **Get faces** e aggiungere il codice seguente:

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            // Get face properties
            Console.WriteLine($"\nFace ID: {face.FaceId}");
            Console.WriteLine($" - Age: {face.FaceAttributes.Age}");
            Console.WriteLine($" - Emotions:");
            foreach (var emotion in face.FaceAttributes.Emotion.ToRankedList())
            {
                Console.WriteLine($"   - {emotion}");
            }

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
                                                            return_face_attributes=features)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            print('\nFace ID: {}'.format(face.face_id))
            detected_attributes = face.face_attributes.as_dict()
            age = 'age unknown' if 'age' not in detected_attributes.keys() else int(detected_attributes['age'])
            print(' - Age: {}'.format(age))

            if 'emotion' in detected_attributes:
                print(' - Emotions:')
                for emotion_name in detected_attributes['emotion']:
                    print('   - {}: {}'.format(emotion_name, detected_attributes['emotion'][emotion_name]))
            
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

## <a name="compare-faces"></a>Confrontare i visi

Un'attività comune consiste nel confrontare i visi e trovare quelli simili. In questo scenario non è necessario *identificare* la persona nelle immagini, ma solo determinare se più immagini mostrano la *stessa* persona (o almeno qualcuno somigliante). 

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione **2**. Questo codice chiama la funzione **CompareFaces**, passando il percorso di due file di immagine (**person1.jpg** e **people.jpg**).
2. Individuare la funzione **CompareFaces** nel file di codice e quindi, sotto il codice esistente che stampa un messaggio nella console, aggiungere il codice seguente:

**C#**

```C
// Determine if the face in image 1 is also in image 2
DetectedFace image_i_face;
using (var image1Data = File.OpenRead(image1))
{    
    // Get the first face in image 1
    var image1_faces = await faceClient.Face.DetectWithStreamAsync(image1Data);
    if (image1_faces.Count > 0)
    {
        image_i_face = image1_faces[0];
        Image img1 = Image.FromFile(image1);
        Graphics graphics = Graphics.FromImage(img1);
        Pen pen = new Pen(Color.LightGreen, 3);
        var r = image_i_face.FaceRectangle;
        Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        String output_file = "face_to_match.jpg";
        img1.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file); 

        //Get all the faces in image 2
        using (var image2Data = File.OpenRead(image2))
        {    
            var image2Faces = await faceClient.Face.DetectWithStreamAsync(image2Data);

            // Get faces
            if (image2Faces.Count > 0)
            {

                var image2FaceIds = image2Faces.Select(f => f.FaceId).ToList<Guid?>();
                var similarFaces = await faceClient.Face.FindSimilarAsync((Guid)image_i_face.FaceId,faceIds:image2FaceIds);
                var similarFaceIds = similarFaces.Select(f => f.FaceId).ToList<Guid?>();

                // Prepare image for drawing
                Image img2 = Image.FromFile(image2);
                Graphics graphics2 = Graphics.FromImage(img2);
                Pen pen2 = new Pen(Color.LightGreen, 3);
                Font font2 = new Font("Arial", 4);
                SolidBrush brush2 = new SolidBrush(Color.Black);

                // Draw and annotate each face
                foreach (var face in image2Faces)
                {
                    if (similarFaceIds.Contains(face.FaceId))
                    {
                        // Draw and annotate face
                        var r2 = face.FaceRectangle;
                        Rectangle rect2 = new Rectangle(r2.Left, r2.Top, r2.Width, r2.Height);
                        graphics2.DrawRectangle(pen2, rect2);
                        string annotation = "Match!";
                        graphics2.DrawString(annotation,font2,brush2,r2.Left, r2.Top);
                    }
                }

                // Save annotated image
                String output_file2 = "matched_faces.jpg";
                img2.Save(output_file2);
                Console.WriteLine(" Results saved in " + output_file2);   
            }
        }

    }
}
```

**Python**

```Python
# Determine if the face in image 1 is also in image 2
with open(image_1, mode="rb") as image_data:
    # Get the first face in image 1
    image_1_faces = face_client.face.detect_with_stream(image=image_data)
    image_1_face = image_1_faces[0]

    # Highlight the face in the image
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_1)
    draw = ImageDraw.Draw(image)
    color = 'lightgreen'
    r = image_1_face.face_rectangle
    bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
    draw = ImageDraw.Draw(image)
    draw.rectangle(bounding_box, outline=color, width=5)
    plt.imshow(image)
    outputfile = 'face_to_match.jpg'
    fig.savefig(outputfile)

# Get all the faces in image 2
with open(image_2, mode="rb") as image_data:
    image_2_faces = face_client.face.detect_with_stream(image=image_data)
    image_2_face_ids = list(map(lambda face: face.face_id, image_2_faces))

    # Find faces in image 2 that are similar to the one in image 1
    similar_faces = face_client.face.find_similar(face_id=image_1_face.face_id, face_ids=image_2_face_ids)
    similar_face_ids = list(map(lambda face: face.face_id, similar_faces))

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_2)
    draw = ImageDraw.Draw(image)

    # Draw and annotate matching faces
    for face in image_2_faces:
        if face.face_id in similar_face_ids:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline='lightgreen', width=10)
            plt.annotate('Match!',(r.left, r.top + r.height + 15), backgroundcolor='white')

    # Save annotated image
    plt.imshow(image)
    outputfile = 'matched_faces.jpg'
    fig.savefig(outputfile)
```

3. Esaminare il codice aggiunto alla funzione **CompareFaces**. Individua il primo viso dell'immagine 1 e lo annota in un nuovo file di immagine denominato **face_to_match.jpg**. Individua quindi tutti i visi dell'immagine 2 e usare i rispettivi ID viso per trovare quelli simili all'immagine 1. Quelli simili vengono annotati e salvati in una nuova immagine denominata **matched_faces.jpg**.
4. Salvare le modifiche e tornare nel terminale integrato per la cartella **face-api**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    *L'output C# può visualizzare avvisi relativi a funzioni asincrone che ora usano l'operatore **await**. È possibile ignorarli.*

    **Python**

    ```
    python analyze-faces.py
    ```
    
5. Quando richiesto, immettere **2** e osservare l'output. Aprire quindi i file **face_to_match.jpg** e **matched_faces.jpg** generati nella stessa cartella del file di codice per visualizzare i visi corrispondenti.
6. Modificare il codice nella funzione **Main** per l'opzione di menu **2** per confrontare **person2.jpg** con **people.jpg**, quindi eseguire nuovamente il programma e visualizzare i risultati.

## <a name="train-a-facial-recognition-model"></a>Eseguire il training di un modello di riconoscimento facciale

In alcuni scenari può essere necessario mantenere un modello di persone specifiche i cui visi possono essere riconosciuti da un'applicazione di intelligenza artificiale. Lo scenario può essere ad esempio un sistema di sicurezza biometrica che usa il riconoscimento facciale per verificare l'identità dei dipendenti in un edificio sicuro.

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione **3**. Questo codice chiama la funzione **TrainModel,** passando il nome e l'ID di un nuovo oggetto **PersonGroup**, che verrà registrato nella risorsa di Servizi cognitivi, e un elenco di nomi di dipendenti.
2. Nella cartella **face-api/images** osservare che sono presenti cartelle con gli stessi nomi dei dipendenti. Ogni cartella contiene più immagini di uno specifico dipendente.
3. Individuare la funzione **TrainModel** nel file di codice e quindi, sotto il codice esistente che stampa un messaggio nella console, aggiungere il codice seguente:

**C#**

```C
// Delete group if it already exists
var groups = await faceClient.PersonGroup.ListAsync();
foreach(var group in groups)
{
    if (group.PersonGroupId == groupId)
    {
        await faceClient.PersonGroup.DeleteAsync(groupId);
    }
}

// Create the group
await faceClient.PersonGroup.CreateAsync(groupId, groupName);
Console.WriteLine("Group created!");

// Add each person to the group
Console.Write("Adding people to the group...");
foreach(var personName in imageFolders)
{
    // Add the person
    var person = await faceClient.PersonGroupPerson.CreateAsync(groupId, personName);

    // Add multiple photo's of the person
    string[] images = Directory.GetFiles("images/" + personName);
    foreach(var image in images)
    {
        using (var imageData = File.OpenRead(image))
        { 
            await faceClient.PersonGroupPerson.AddFaceFromStreamAsync(groupId, person.PersonId, imageData);
        }
    }

}

    // Train the model
Console.WriteLine("Training model...");
await faceClient.PersonGroup.TrainAsync(groupId);

// Get the list of people in the group
Console.WriteLine("Facial recognition model trained with the following people:");
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    Console.WriteLine($"-{person.Name}");
}
```

**Python**

```Python
# Delete group if it already exists
groups = face_client.person_group.list()
for group in groups:
    if group.person_group_id == group_id:
        face_client.person_group.delete(group_id)

# Create the group
face_client.person_group.create(group_id, group_name)
print ('Group created!')

# Add each person to the group
print('Adding people to the group...')
for person_name in image_folders:
    # Add the person
    person = face_client.person_group_person.create(group_id, person_name)

    # Add multiple photo's of the person
    folder = os.path.join('images', person_name)
    person_pics = os.listdir(folder)
    for pic in person_pics:
        img_path = os.path.join(folder, pic)
        img_stream = open(img_path, "rb")
        face_client.person_group_person.add_face_from_stream(group_id, person.person_id, img_stream)

# Train the model
print('Training model...')
face_client.person_group.train(group_id)

# Get the list of people in the group
print('Facial recognition model trained with the following people:')
people = face_client.person_group_person.list(group_id)
for person in people:
    print('-', person.name)
```

4. Esaminare il codice aggiunto alla funzione **TrainModel**. Esegue le attività seguenti:
    - Ottiene un elenco di oggetti **PersonGroup** registrati nel servizio ed elimina quello specificato, se esistente.
    - Crea un gruppo con l'ID e il nome specificati.
    - Aggiunge una persona al gruppo per ogni nome specificato, quindi aggiunge più immagini di ogni persona.
    - Esegue il training di un modello di riconoscimento facciale in modo alle persone specificate nel gruppo e alle immagini dei loro visi.
    - Recupera un elenco delle persone specificate nel gruppo e le visualizza.
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

6. Quando richiesto, immettere **3** e osservare l'output, notando che l'oggetto **PersonGroup** creato include due persone.

## <a name="recognize-faces"></a>Riconoscere visi

Una volta definito un oggetto **PeopleGroup** ed eseguito il training di un modello di riconoscimento facciale, è possibile usarlo per riconoscere gli individui specificati in un'immagine.

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione **4**. Questo codice chiama la funzione **RecognizeFaces**, passando il percorso di un file di immagine (**people.jpg**) e l'ID dell'oggetto **PeopleGroup** da usare per l'identificazione dei visi.
2. Individuare la funzione **RecognizeFaces** nel file di codice e quindi, sotto il codice esistente che stampa un messaggio nella console, aggiungere il codice seguente:

**C#**

```C
// Detect faces in the image
using (var imageData = File.OpenRead(imageFile))
{    
    var detectedFaces = await faceClient.Face.DetectWithStreamAsync(imageData);

    // Get faces
    if (detectedFaces.Count > 0)
    {
        
        // Get a list of face IDs
        var faceIds = detectedFaces.Select(f => f.FaceId).ToList<Guid?>();

        // Identify the faces in the people group
        var recognizedFaces = await faceClient.Face.IdentifyAsync(faceIds, groupId);

        // Get names for recognized faces
        var faceNames = new Dictionary<Guid?, string>();
        if (recognizedFaces.Count> 0)
        {
            foreach(var face in recognizedFaces)
            {
                var person = await faceClient.PersonGroupPerson.GetAsync(groupId, face.Candidates[0].PersonId);
                Console.WriteLine($"-{person.Name}");
                faceNames.Add(face.FaceId, person.Name);

            }
        }

        
        // Annotate faces in image
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen penYes = new Pen(Color.LightGreen, 3);
        Pen penNo = new Pen(Color.Magenta, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Cyan);
        foreach (var face in detectedFaces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            if (faceNames.ContainsKey(face.FaceId))
            {
                // If the face is recognized, annotate in green with the name
                graphics.DrawRectangle(penYes, rect);
                string personName = faceNames[face.FaceId];
                graphics.DrawString(personName,font,brush,r.Left, r.Top);
            }
            else
            {
                // Otherwise, just annotate the unrecognized face in magenta
                graphics.DrawRectangle(penNo, rect);
            }
        }

        // Save annotated image
        String output_file = "recognized_faces.jpg";
        image.Save(output_file);
        Console.WriteLine("Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Detect faces in the image
with open(image_file, mode="rb") as image_data:

    # Get faces
    detected_faces = face_client.face.detect_with_stream(image=image_data)

    # Get a list of face IDs
    face_ids = list(map(lambda face: face.face_id, detected_faces))

    # Identify the faces in the people group
    recognized_faces = face_client.face.identify(face_ids, group_id)

    # Get names for recognized faces
    face_names = {}
    if len(recognized_faces) > 0:
        print(len(recognized_faces), 'faces recognized.')
        for face in recognized_faces:
            person_name = face_client.person_group_person.get(group_id, face.candidates[0].person_id).name
            print('-', person_name)
            face_names[face.face_id] = person_name

    # Annotate faces in image
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    for face in detected_faces:
        r = face.face_rectangle
        bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
        draw = ImageDraw.Draw(image)
        if face.face_id in face_names:
            # If the face is recognized, annotate in green with the name
            draw.rectangle(bounding_box, outline='lightgreen', width=3)
            plt.annotate(face_names[face.face_id],
                        (r.left, r.top + r.height + 15), backgroundcolor='white')
        else:
            # Otherwise, just annotate the unrecognized face in magenta
            draw.rectangle(bounding_box, outline='magenta', width=3)

    # Save annotated image
    plt.imshow(image)
    outputfile = 'recognized_faces.jpg'
    fig.savefig(outputfile)

    print('\nResults saved in', outputfile)
```
    
3. Esaminare il codice aggiunto alla funzione **RecognizeFaces**. Il codice individua i visti nell'immagine e crea un elenco dei relativi ID. Usa quindi il gruppo di persone per provare a identificare i visti nell'elenco di ID viso. I visi riconosciuti vengono annotati con il nome della persona identificata, quindi i risultati vengono salvati in **recognized_faces.jpg**.
4. Salvare le modifiche e tornare nel terminale integrato per la cartella **face-api**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    *L'output C# può visualizzare avvisi relativi a funzioni asincrone che ora usano l'operatore **await**. È possibile ignorarli.*

    **Python**

    ```
    python analyze-faces.py
    ```

5. Quando richiesto, immettere **4** e osservare l'output. Aprire quindi il file **recognized_faces.jpg** generato nella stessa cartella del file di codice per visualizzare le persone identificate.
6. Modificare il codice nella funzione **Main** per l'opzione di menu **4** per riconoscere visi in **people2.jpg**, quindi eseguire nuovamente il programma e visualizzare i risultati. Dovrebbero essere riconosciute le stesse persone.

## <a name="verify-a-face"></a>Verificare un viso

Il riconoscimento facciale viene spesso usato per la verifica dell'identità. Con il servizio Viso è possibile verificare un viso in un'immagine confrontandolo con un altro viso o con quello di una persona registrata in un oggetto **PersonGroup**.

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione **5**. Questo codice chiama la funzione **VerifyFace**, passando il percorso di un file di immagine (**person1.jpg**) e l'ID dell'oggetto **PeopleGroup** da usare per l'identificazione dei visi.
2. Individuare la funzione **VerifyFace** nel file di codice e quindi, sotto il commento **Get the ID of the person from the people group** (sopra il codice che stampa il risultato) aggiungere il codice seguente:

**C#**

```C
// Get the ID of the person from the people group
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    if (person.Name == personName)
    {
        Guid personId = person.PersonId;

        // Get the first face in the image
        using (var imageData = File.OpenRead(personImage))
        {    
            var faces = await faceClient.Face.DetectWithStreamAsync(imageData);
            if (faces.Count > 0)
            {
                Guid faceId = (Guid)faces[0].FaceId;

                //We have a face and an ID. Do they match?
                var verification = await faceClient.Face.VerifyFaceToPersonAsync(faceId, personId, groupId);
                if (verification.IsIdentical)
                {
                    result = "Verified";
                }
            }
        }
    }
}
```

**Python**

```Python
# Get the ID of the person from the people group
people = face_client.person_group_person.list(group_id)
for person in people:
    if person.name == person_name:
        person_id = person.person_id

        # Get the first face in the image
        with open(person_image, mode="rb") as image_data:
            faces = face_client.face.detect_with_stream(image=image_data)
            if len(faces) > 0:
                face_id = faces[0].face_id

                # We have a face and an ID. Do they match?
                verification = face_client.face.verify_face_to_person(face_id, person_id, group_id)
                if verification.is_identical:
                    result = 'Verified'
```

3. Esaminare il codice aggiunto alla funzione **VerifyFace**. Il codice cerca nel gruppo l'ID della persona con il nome specificato. Se la persona esiste, il codice ottiene l'ID viso del primo viso dell'immagine. Infine, se nell'immagine è presente un viso, il codice lo verifica rispetto all'ID della persona specificata.
4. Salvare le modifiche e tornare nel terminale integrato per la cartella **face-api**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python analyze-faces.py
    ```

5. Quando richiesto, immettere **5** e osservare il risultato.
6. Modificare il codice nella funzione **Main** per l'opzione di menu **5** per sperimentare con diverse combinazioni di nomi e con le immagini **person1.jpg** e **person2.jpg**.

## <a name="more-information"></a>Altre informazioni

Per altre formazioni sull'uso del servizio **Visione artificiale** per il rilevamento dei visi, vedere la [documentazione di Visione artificiale](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Per altre informazioni sul servizio **Viso**, vedere la [documentazione di Viso](https://docs.microsoft.com/azure/cognitive-services/face/).
