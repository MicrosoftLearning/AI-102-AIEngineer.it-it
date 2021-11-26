---
lab:
  title: Creare una soluzione per domande e risposte
  module: Module 6 - Building a QnA Solution
ms.openlocfilehash: 15b1b41d16f389392315f31c3daba81bc18101c9
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625979"
---
# <a name="create-a-qna-solution"></a>Creare una soluzione per domande e risposte

Uno degli scenari di conversazione più comuni consiste nel fornire supporto tramite una knowledge base di domande frequenti. Molte organizzazioni pubblicano le domande frequenti come documenti o pagine Web. Questo approccio risulta ottimale per un set ridotto di coppie di domanda e risposta, ma la ricerca nei documenti di grandi dimensioni può risultare difficile e può richiedere molto tempo.

QnA Maker è un servizio cognitivo che consente di creare una knowledge base di coppie di domanda e risposta che possono essere sottoposte a query mediante input in linguaggio naturale e viene usata più comunemente come risorsa che può essere usata da un bot per cercare risposte alle domande inviate dagli utenti.

In questo lab verrà usato Managed QnA Maker, una funzionalità disponibile in Analisi del testo. 

## <a name="create-a-text-analytics-resource"></a>Creare una risorsa Analisi del testo 

Per creare e ospitare una knowledge base usando Managed QnA Maker, è necessario che nella sottoscrizione di Azure sia disponibile una risorsa di Analisi del testo.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Selezionare il pulsante **&#65291;Crea una risorsa**, cercare *Analisi del testo* e creare una risorsa di **Analisi del testo**. 
3. Fare clic su **Seleziona** nel blocco **Custom question answering (anteprima)** . Fare quindi clic su **Continue to create your resource** (Continuare per creare la risorsa). Sarà necessario immettere le impostazioni seguenti:
    
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, è possibile che non si sia autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere qualsiasi località disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S
    - **Posizione di Ricerca di Azure**\*: *scegliere una località nella stessa area globale della risorsa di QnA Maker locale*.
    - **Piano tariffario di Ricerca di Azure**: Gratuito (F) *Se questo piano non è disponibile, selezionare Basic (B)*
    - **Note legali**: _accettare le note_ 
    - **Avviso per intelligenza artificiale responsabile**: _accettare_
    
    \*Custom Question Answering usa Ricerca di Azure peer indicizzare ed eseguire query nella knowledge base di domande e risposte.

4. Attendere il completamento della distribuzione e quindi visualizzare i dettagli della distribuzione.

## <a name="create-a-knowledge-base"></a>Creare una knowledge base

Per creare una knowledge base nella risorsa di Analisi del testo, è possibile usare il portale di QnA Maker. In questo caso, si creerà un knowledge base contenente domande e risposte su [Microsoft Learn](https://docs.microsoft.com/learn).

1. In una nuova scheda del browser passare al portale di QnA Maker in `https://qnamaker.ai` e accedere usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Nella parte superiore del portale selezionare **Crea una knowledge base**.
3. È già stata creata una risorsa di QnA Maker, quindi è possibile saltare il passaggio 1. Nella sezione **Passaggio 2** selezionare le impostazioni seguenti:
    - **ID Microsoft Azure Directory**: directory di Azure contenente la sottoscrizione.
    - **Nome della sottoscrizione di Azure**: sottoscrizione di Azure usata.
    - **Servizio di domande e risposte di Azure**: risorsa di Analisi del testo creata in precedenza.
    - **Lingua**: inglese. *Per impostazione predefinita, questa opzione è disponibile solo per la prima knowledge base creata*.
4. Nella sezione **Passaggio 3** immettere **Learn FAQ** come nome della knowledge base.

    È possibile creare una knowledge base completamente nuova, ma in genere si inizia importando domande e risposte da una pagina o da un documento di domande frequenti esistente.

5. Nella sezione **Passaggio 4**:
    - Nella casella **URL** digitare `https://docs.microsoft.com/en-us/learn/support/faq` e fare clic su **&#65291; Aggiungi URL**.
    - Nella sezione **Chiacchiere** selezionare **Amichevole**.
6. Nella sezione **Passaggio 5** selezionare **Create your KB** (Creare una KB personalizzata) e attendere il completamento della creazione della knowledge base.

## <a name="modify-the-knowledge-base"></a>Modificare la knowledge base

La knowledge base è stata popolata con coppie di domanda e risposta dalle domande frequenti di Microsoft Learn, a cui è stato aggiunto un set di coppie di domanda e risposta di *Chiacchiere* basate su conversazione. È possibile estendere la knowledge base aggiungendo altre coppie di domanda e risposta.

1. Nella knowledge base selezionare **&#65291; Add QnA pair** (Aggiungi coppia di domanda e risposta).
2. Nella casella **Domanda** digitare `What is Microsoft certification?`
3. Selezionare **&#65291; Aggiungi frasi alternative** e digitare `How can I demonstrate my Microsoft technology skills?`.
4. Nella casella **Risposta** digitare `The Microsoft Certified Professional program enables you to validate and prove your skills with Microsoft technologies.`

    In alcuni casi, è opportuno consentire all'utente di porre una domanda di completamento alla risposta per creare una conversazione *a più turni* che consenta all'utente di perfezionare in modo iterativo la domanda per ottenere la risposta di cui ha bisogno.

5. Nella risposta immessa per la domanda di certificazione selezionare **&#65291; Add follow-up prompt** (Aggiungi richiesta di completamento).
6. Nella finestra di dialogo **Richiesta di completamento** immettere le impostazioni seguenti:
    - **Testo visualizzato**: `Learn more about certification.`
    - **Link to QnA**\* (Collegamento a domande e risposte): `You can learn more about certification on the [Microsoft certification page](https://docs.microsoft.com/learn/certifications/).`
    - **Context-only** (Solo contesto): opzione selezionata. *Questa opzione assicura che la risposta venga restituita solo nel contesto di una domanda di completamento dalla domanda di certificazione originale.*

    \*Se si digita nella casella **Link to QnA** (Collegamento a domande e risposte), vengono cercate le risposte esistenti nella knowledge base. Quando non viene trovata alcuna corrispondenza, viene creata per impostazione predefinita una nuova coppia di domanda e risposta. Si noti che il testo digitato qui è in formato Markdown.

## <a name="train-and-test-the-knowledge-base"></a>Eseguire il training e il test della knowledge base

Ora che hai una knowledge base, puoi testarla nel portale di QnA Maker.

1. In alto a destra nella pagina, fai clic su **Salva ed esegui il training** per eseguire il training della tua knowledge base.
2. Al termine del training, fare clic su **&larr; Test** per aprire il riquadro di test.
3. Nella parte superiore del riquadro di test *deselezionare* la casella con testo *Display Short Answer* (Visualizza risposta breve). Nella parte inferiore immettere il messaggio *Hello*. Dovrebbe essere restituita una risposta appropriata.
4. Nella parte inferiore del riquadro di test immettere il messaggio *What is Microsoft Learn?* . Dovrebbe comparire una risposta adeguata dalle domande frequenti.
5. Immettere il messaggio *That makes me happy!* Dovrebbe essere restituita una risposta appropriata di Chiacchiere.
6. Immettere il messaggio *Tell me about certification*. La risposta creata dovrebbe essere restituita insieme a un pulsante per la richiesta di completamento.
7. Selezionare il pulsante di completamento **Learn more about certification** (Altre informazioni sulla certificazione). Dovrebbe essere restituita una risposta di completamento con un collegamento alla pagina di certificazione.
8. Al termine del test della knowledge base, fare clic su **&rarr; Test** per chiudere il riquadro di test.

## <a name="publish-the-knowledge-base"></a>Pubblicare la knowledge base

La knowledge base offre un servizio back-end che le applicazioni client possono usare per rispondere alle domande. È ora possibile pubblicare la knowledge base e accedere alla rispettiva interfaccia REST da un client.

1. Nella parte superiore della pagina di QnA Maker fare clic su **Pubblica**. Nella pagina **Domande frequenti di Learn** fare clic su **Pubblica**.
2. Al termine della pubblicazione, visualizzare il codice di esempio fornito per usare l'endpoint REST della knowledge base. Sono disponibili un esempio per *Postman* e un esempio per *Curl*.
3. Visualizzare la scheda **Curl** e copiare il codice di esempio.
4. Avviare Visual Studio Code e aprire un riquadro del terminale.
5. Incollare il codice copiato nel terminale, quindi modificarlo per sostituire **&lt;your question&gt;** con **What is a learning path?** .
6. Immettere il comando e visualizzare la risposta JSON restituita dalla knowledge base.

## <a name="create-a-bot-for-the-knowledge-base"></a>Creare un bot per la knowledge base

Le applicazioni client usate più spesso per recuperare risposte da una knowledge base sono bot.

1. Nella pagina contenente la conferma di pubblicazione e il codice Curl di esempio selezionare **Create Bot** (Crea bot). Il portale di Azure si apre in una nuova scheda del browser, così puoi creare un bot per app Web nella tua sottoscrizione di Azure.
2. Nel portale di Azure, crea un bot per app Web con le impostazioni seguenti (la maggior parte sono già popolate):

    *Se alcuni valori non sono presenti, aggiornare il browser.*  

  - **Handle di bot**: *nome univoco per il bot*
  - **Sottoscrizione**: *la propria sottoscrizione di Azure*
  - **Gruppo di risorse**: *gruppo di risorse contenente la risorsa di QnA Maker*
  - **Posizione**: *la stessa posizione del servizio Analisi del testo*.
  - **Piano tariffario**: F0
  - **Nome dell'app**: *uguale a **Handle di bot** con *.azurewebsites.net* aggiunto automaticamente
  - **Lingua dell'SDK**: *scegliere C# o Node.js*
  - **Chiave di autenticazione QnA**: *questo valore dovrebbe essere impostato automaticamente sulla chiave di autenticazione dalla knowledge base di domande e risposte*
  - **Piano di servizio app/Località**: *è possibile che questo valore sia impostato automaticamente su una posizione e un piano idonei, se disponibili. In caso contrario, creare un nuovo piano*
  - **Application Insights**: disattivato
  - **Password e ID app di Microsoft**: l'ID e la password dell'app vengono creati automaticamente.
3. Attendi che il tuo bot venga creato (l'icona di notifica a forma di campana in alto a destra si muove durante l'attesa). Quindi, nella notifica di completamento della distribuzione, fai clic su **Vai alla risorsa** (oppure in alternativa, sulla home page, fai clic su **Gruppi di risorse**, apri il gruppo di risorse in cui hai creato il bot per app Web e selezionalo.)
4. Nel pannello per il bot visualizzare la pagina **Test in chat Web** e attendere fino alla visualizzazione del messaggio **Hello and welcome!** da parte del bot (l'inizializzazione potrebbe richiedere alcuni minuti).
5. Usa l'interfaccia della chat di test per assicurarti che il bot risponda alle domande dalla tua knowledge base come previsto. Provare ad esempio a inviare *What is Microsoft certification?* .

## <a name="more-information"></a>Altre informazioni

Per altre informazioni su QnA Maker, vedere la [documentazione di QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/).