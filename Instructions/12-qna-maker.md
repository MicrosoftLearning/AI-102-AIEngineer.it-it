---
lab:
  title: Creare una soluzione di risposta alle domande
  module: Module 6 - Building a QnA Solution
ms.openlocfilehash: 42b3b6e6955188d3d442a4f587ead6afe0f389c6
ms.sourcegitcommit: a879eae298aaf2ab6415fc4fdb67f52f592d907a
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/02/2022
ms.locfileid: "146902482"
---
# <a name="create-a-question-answering-solution"></a>Creare una soluzione di risposta alle domande

Uno degli scenari di conversazione più comuni consiste nel fornire supporto tramite una knowledge base di domande frequenti. Molte organizzazioni pubblicano le domande frequenti come documenti o pagine Web. Questo approccio risulta ottimale per un set ridotto di coppie di domanda e risposta, ma la ricerca nei documenti di grandi dimensioni può risultare difficile e può richiedere molto tempo.

Il servizio **Lingua** include una funzionalità di *risposta alle domande* che consente di creare una knowledge base di coppie di domanda e risposta su cui è possibile eseguire query mediante input in linguaggio naturale e che viene spesso usata come risorsa a cui un bot può fare riferimento per cercare le risposte alle domande inviate dagli utenti.

> **Nota**: la funzionalità di risposta alle domande nel servizio Lingua è una versione più recente del servizio QnA Maker, che è ancora disponibile come servizio separato.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se il repository di codice **AI-102-AIEngineer** non è già stato clonato nell'ambiente in cui si sta lavorando a questo lab, seguire questa procedura per clonarlo. In caso contrario, aprire la cartella clonata in Visual Studio Code.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale (non importa quale).
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="create-a-language-resource"></a>Creare una risorsa del servizio Lingua

Per creare e ospitare un knowledge base per la risposta alle domande, è necessaria una risorsa del **servizio Lingua** nella sottoscrizione di Azure.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Fare clic sul pulsante **&#65291;Crea una risorsa**, cercare *Lingua* e creare una risorsa del **servizio Lingua**.
3. Fare clic su **Seleziona** nel blocco **Risposta personalizzata alle domande**. Fare quindi clic su **Continue to create your resource** (Continuare per creare la risorsa). Sarà necessario immettere le impostazioni seguenti:
    
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, è possibile che non si sia autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere qualsiasi località disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S
    - **Posizione di Ricerca di Azure**\*: *scegliere una località nella stessa area globale della risorsa del servizio Lingua*.
    - **Piano tariffario di Ricerca di Azure**: Gratuito (F) *Se questo piano non è disponibile, selezionare Basic (B)*
    - **Note legali**: _accettare le note_ 
    - **Avviso per intelligenza artificiale responsabile**: _accettare_
    
    \*Custom Question Answering usa Ricerca di Azure peer indicizzare ed eseguire query nella knowledge base di domande e risposte.

4. Attendere il completamento della distribuzione e quindi visualizzare i dettagli della distribuzione.

## <a name="create-a-question-answering-project"></a>Creare un progetto di risposta alle domande

Per creare una knowledge base per la risposta alle domande nella risorsa del servizio Lingua, è possibile usare il portale di Language Studio per creare un progetto di risposta alle domande. In questo caso, si creerà un knowledge base contenente domande e risposte su [Microsoft Learn](https://docs.microsoft.com/learn).

1. In una nuova scheda del browser passare al portale *Language Studio* all'indirizzo `https://language.azure.com` e accedere usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Se viene richiesto di scegliere una risorsa del servizio Lingua, selezionare le impostazioni seguenti:
    - **Directory di Azure**: directory di Azure contenente la sottoscrizione.
    - **Sottoscrizione di Azure**: sottoscrizione di Azure in uso.
    - **Language resource** (Risorsa Lingua): risorsa del servizio Lingua creata in precedenza.
3. Se <u>non</u> viene chiesto di scegliere una risorsa del servizio Lingua, è possibile che nella sottoscrizione siano presenti più risorse di questo tipo e in tal caso:
    1. Sulla barra nella parte superiore della pagina fare clic sul pulsante **Impostazioni (&#9881;)** .
    2. Nella pagina **Impostazioni** visualizzare la scheda **Risorse**.
    3. Selezionare la risorsa del servizio Lingua appena creata e fare clic su **Cambiare risorsa**.
    4. Nella parte superiore della pagina fare clic su **Language Studio** per tornare alla home page di Language Studio.
3. Nella parte superiore del portale scegliere **Risposta personalizzata alle domande** dal menu **Crea nuovo**.
4. Nella procedura guidata ***Crea un progetto**, nella pagina **Choose language setting** (Scegli impostazione lingua) selezionare l'opzione per impostare la lingua per tutti i progetti in questa risorsa e scegliere **inglese** come lingua. Quindi fare clic su **Avanti**.
5. Nella pagina **Enter basic information** (Immetti informazioni di base) immettere i dati seguenti e quindi fare clic su **Avanti**:
    - **Nome** LearnFAQ
    - **Descrizione**: FAQ for Microsoft Learn
    - **Default answer when no answer is returned** (Risposta predefinita quando non viene restituita una risposta): Sorry, I don't understand the question
6. Nella pagina **Rivedere e completare** fare clic su **Crea progetto**.

## <a name="add-a-sources-to-the-knowledge-base"></a>Aggiungere un'origine alla knowledge base

È possibile creare una knowledge base completamente nuova, ma in genere si inizia importando domande e risposte da una pagina o da un documento di domande frequenti esistente. In questo caso, si importeranno i dati da una pagina Web di domande frequenti esistente per Microsoft Learn e si importeranno anche alcune domande e risposte predefinite per supportare gli scambi colloquiali.

1. Nella pagina **Manage sources** (Gestisci origini) per il progetto di risposta alle domande selezionare **URL** nell'elenco **&#9547; Aggiungi origine**. Nella finestra di dialogo **Aggiungi URL** fare clic su **&#9547; Aggiungi URL**, aggiungere l'URL seguente e quindi fare clic su **Aggiungi tutto** per aggiungerlo alla knowledge base:
    - **Nome**: `Learn FAQ Page`
    - **URL**: `https://docs.microsoft.com/en-us/learn/support/faq`
2. Nella pagina **Manage sources** (Gestisci origini) per il progetto di risposta alle domande selezionare **Chitchat** (Chiacchiere) nell'elenco **&#9547; Aggiungi origine**. Nella finestra di dialogo **Add chit chat** (Aggiungi chiacchiere) selezionare **Friendly** (Colloquiale) e fare clic su **Add chit chat** (Aggiungi chiacchiere).

## <a name="edit-the-knowledge-base"></a>Modificare la knowledge base

La knowledge base è stata popolata con coppie di domanda e risposta dalle domande frequenti di Microsoft Learn, a cui è stato aggiunto un set di coppie di domanda e risposta di *Chiacchiere* basate su conversazione. È possibile estendere la knowledge base aggiungendo altre coppie di domanda e risposta.

1. Nel progetto **LearnFAQ** in Language Studio selezionare la pagina **Modifica knowledge base** per visualizzare le coppie di domande e risposte esistenti (se vengono visualizzati suggerimenti, leggerli e fare clic su **OK** per eliminarli oppure fare clic su **Ignora tutto**)
2. Nella knowledge base selezionare **&#65291; Add question pair** (Aggiungi coppia domanda/risposta).
3. Nella casella **Domanda** digitare `What is Microsoft certification?` e premere **INVIO**.
4. Selezionare **&#65291; Aggiungi frase alternativa**, digitare `How can I demonstrate my Microsoft technology skills?` e premere **INVIO**.
5. Nella casella **Risposta** digitare `The Microsoft Certified Professional program enables you to validate and prove your skills with Microsoft technologies.` e quindi fare clic su **Invia** per aggiungere la domanda (inclusa la formulazione alternativa) e la risposta alla knowledge base.

    In alcuni casi, è opportuno consentire all'utente di porre un'ulteriore domanda in seguito alla risposta, in modo da creare una conversazione *a più turni* che consenta all'utente di perfezionare in modo iterativo la domanda per ottenere la risposta necessaria.

6. Nella risposta immessa per la domanda sulla certificazione selezionare **&#65291; Add follow-up prompts** (Aggiungi richieste di completamento).
7. Nella finestra di dialogo **Richiesta di completamento** immettere le impostazioni seguenti e quindi fare clic su **Add prompt** (Aggiungi richiesta):
    - **Text displayed in the prompt to the user**: `Learn more about certification`.
    - Selezionare **Create link to new pair** (Crea collegamento a nuova coppia) e immettere il testo seguente: `You can learn more about certification on the [Microsoft certification page](https://docs.microsoft.com/learn/certifications/).`
    - **Show in contextual flow only** (Mostra solo nel flusso contestuale): opzione selezionata. *Questa opzione assicura che la risposta venga restituita solo nel contesto di una domanda di completamento dalla domanda di certificazione originale.*

## <a name="train-and-test-the-knowledge-base"></a>Eseguire il training e il test della knowledge base

Ora che si ha una knowledge base, è possibile testarla in Language Studio.

1. In alto a destra nella pagina fare clic su **Salva modifiche**.
2. Dopo aver salvato le modifiche, fare clic su **Prova** per aprire il riquadro di test.
3. Nella parte superiore del riquadro di test *deselezionare* l'opzione per visualizzare le risposte brevi. Nella parte inferiore immettere quindi il messaggio `Hello`. Dovrebbe essere restituita una risposta appropriata.
4. Nella parte inferiore del riquadro di test immettere il messaggio `What is Microsoft Learn?`. Dovrebbe comparire una risposta adeguata dalle domande frequenti.
5. Immettere il messaggio `Thanks!`. Verrà restituita una risposta colloquiale appropriata.
6. Immettere il messaggio `Tell me about certification`. Verrà restituita la risposta creata insieme a un collegamento per la richiesta di completamento.
7. Selezionare il collegamento di completamento **Learn more about certification**. Dovrebbe essere restituita una risposta di completamento con un collegamento alla pagina di certificazione.
8. Al termine del test della knowledge base, chiudere il riquadro di test.

## <a name="deploy-and-test-the-knowledge-base"></a>Distribuire e testare la knowledge base

La knowledge base offre un servizio back-end che le applicazioni client possono usare per rispondere alle domande. È ora possibile pubblicare la knowledge base e accedere alla rispettiva interfaccia REST da un client.

1. Nel progetto **LearnFAQ** in Language Studio selezionare la pagina **Deploy knowledge base** (Distribuisci knowledge base).
2. Nella parte superiore della pagina fare clic su **Distribuisci**. Fare quindi clic su **Distribuisci** per confermare la distribuzione della knowledge base.
3. Al termine della distribuzione, fare clic su **Get prediction URL** (Ottieni URL di stima) per visualizzare l'endpoint REST per la knowledge base e copiarlo negli Appunti (senza chiudere per il momento la finestra di dialogo).
4. In Visual Studio Code, nella cartella **12-qna** aprire **ask-question.cmd**. Questo script usa *Curl* per chiamare l'interfaccia REST di un endpoint di risposta alle domande.
5. Nello script sostituire *YOUR_PREDICTION_ENDPOINT* con l'endpoint di stima copiato, assicurandosi che sia racchiuso tra virgolette.
6. Tornare al browser e nella finestra di dialogo **Get prediction URL** (Ottieni URL di stima) osservare che la richiesta di esempio include un valore per il parametro **Ocp-Apim-Subscription-Key**, simile a *ab12c345de678fg9hijk01lmno2pqrs34*. Si tratta della chiave di autorizzazione per la risorsa. Copiare il valore negli Appunti e quindi fare clic su **OK** per chiudere la finestra di dialogo.
7. Tornare a Visual Studio Code e nello script **ask-question.cmd** sostituire *YOUR_KEY* con la chiave copiata, assicurandosi che sia racchiusa tra virgolette.
8. Si noti che il comando Curl nello script invia un parametro **question** con il valore **What is a Learning Path?** .
9. Verificare che l'intero script sia simile al codice seguente e quindi salvare il file.

    ```
    @echo off
    SETLOCAL ENABLEDELAYEDEXPANSION

    rem Set variables
    set prediction_url="https://some-name.cognitiveservices.azure.com/language/......."
    set key="ab12c345de678fg9hijk01lmno2pqrs34"

    curl -X POST !prediction_url! -H "Ocp-Apim-Subscription-Key: !key!" -H "Content-Type: application/json" -d "{'question': 'What is a learning Path?' }"
    ```

10. Nel riquadro di esplorazione di Visual Studio Code fare clic con il pulsante destro del mouse sulla cartella **ask-question.cmd** e scegliere **Apri nel terminale integrato**.
11. Nel riquadro del terminale immettere il comando `ask-question.cmd` per eseguire lo script e visualizzare la risposta JSON restituita dal servizio, che deve contenere una risposta appropriata alla domanda *What is a learning path?* .

## <a name="create-a-bot-for-the-knowledge-base"></a>Creare un bot per la knowledge base

Le applicazioni client usate più spesso per recuperare risposte da una knowledge base sono bot.

1. Tornare a Language Studio nel browser e nella pagina **Deploy knowledge base** (Distribuisci knowledge base) selezionare **Create Bot** (Crea bot). Verrà aperto il portale di Azure in una nuova scheda del browser, dove è possibile creare un bot per app Web nella sottoscrizione di Azure in uso.
2. Nel portale di Azure, crea un bot per app Web con le impostazioni seguenti (la maggior parte sono già popolate):

    *Se alcuni valori non sono presenti, aggiornare il browser.*  

  - **Handle di bot**: *nome univoco per il bot*
  - **Sottoscrizione**: *la propria sottoscrizione di Azure*
  - **Gruppo di risorse**: *gruppo di risorse contenente la risorsa Lingua*
  - **Posizione**: *la stessa posizione del servizio Analisi del testo*.
  - **Piano tariffario**: F0
  - **Nome app**: *uguale a **Handle del bot** con un ID univoco e *.azurewebsites.net* aggiunto automaticamente
  - **Lingua dell'SDK**: *scegliere C# o Node.js*
  - **Chiave di autenticazione QnA**: *questo valore dovrebbe essere impostato automaticamente sulla chiave di autenticazione dalla knowledge base di domande e risposte*
  - **Piano di servizio app/Località**: *è possibile che questo valore sia impostato automaticamente su una posizione e un piano idonei, se disponibili. In caso contrario, creare un nuovo piano*
  - **Application Insights**: disattivato
  - **Password e ID app di Microsoft**: l'ID e la password dell'app vengono creati automaticamente.
3. Attendere la creazione del bot. Fare quindi clic su **Vai alla risorsa** o, in alternativa, nella home page, fare clic su **Gruppi di risorse**, aprire il gruppo di risorse in cui è stato creato il bot per app Web e fare clic su di esso.
4. Nel pannello per il bot visualizzare la pagina **Test in chat Web** e attendere fino alla visualizzazione del messaggio **Hello and welcome!** da parte del bot (l'inizializzazione potrebbe richiedere alcuni minuti).
5. Usa l'interfaccia della chat di test per assicurarti che il bot risponda alle domande dalla tua knowledge base come previsto. Provare ad esempio a inviare `What is Microsoft certification?`.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sulla risposta alle domande nel servizio Lingua, vedere la [documentazione del servizio Lingua](https://docs.microsoft.com/en-us/azure/cognitive-services/language-service/question-answering/overview).
