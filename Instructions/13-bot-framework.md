---
lab:
  title: Creare un bot con Bot Framework SDK
  module: Module 7 - Conversational AI and the Azure Bot Service
ms.openlocfilehash: ab51a1f11f4eee99838634d83b5d9261a8c20ecb
ms.sourcegitcommit: 480c5898009ea964025fdecb57900aefeeac81fd
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/07/2022
ms.locfileid: "147019885"
---
# <a name="create-a-bot-with-the-bot-framework-sdk"></a>Creare un bot con Bot Framework SDK

I *bot* sono agenti software che possono partecipare a conversazioni con utenti umani. Microsoft Bot Framework offre una piattaforma completa per la creazione di bot che possono essere distribuiti come servizi cloud tramite il servizio Azure Bot.

In questo esercizio si userà Microsoft Bot Framework SDK per creare e distribuire un bot.

## <a name="before-you-start"></a>Prima di iniziare

Per iniziare, preparare l'ambiente per lo sviluppo di bot.

### <a name="update-the-bot-framework-emulator"></a>Aggiornare Bot Framework Emulator

Si useranno Bot Framework SDK per creare il bot e Bot Framework Emulator per testarlo. L'applicazione Bot Framework Emulator viene aggiornata regolarmente, quindi è necessario assicurarsi che sia installata la versione più recente.

> **Nota**: gli aggiornamenti possono includere modifiche all'interfaccia utente che influiscono sulle istruzioni in questo esercizio.

1. Avviare **Bot Framework Emulator** e, se viene richiesto di installare un aggiornamento, eseguire questa operazione per l'utente attualmente connesso. Se la richiesta non viene visualizzata automaticamente, usare l'opzione **Cerca aggiornamenti** del menu **?** per controllare la disponibilità di aggiornamenti.
2. Dopo avere installato gli aggiornamenti disponibili, chiudere Bot Framework Emulator finché non sarà necessario in un secondo momento.

> **Importante**: in alcuni casi il download dell’aggiornamento non riuscirà e sono in corso indagini per scoprire il motivo. Se non si notano progressi nel tentativo di aggiornamento dopo un paio di minuti, è possibile interrompere il download e usare la versione attualmente installata dell'emulatore.

### <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se il repository di codice **AI-102-AIEngineer** non è già stato clonato nell'ambiente in cui si sta lavorando a questo lab, seguire questa procedura per clonarlo. In caso contrario, aprire la cartella clonata in Visual Studio Code.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale (non importa quale).
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="create-a-bot"></a>Creare un bot

È possibile usare Bot Framework SDK per creare un bot basato su un modello e quindi personalizzare il codice per soddisfare i requisiti specifici.

> **Nota**: in questo esercizio è possibile scegliere se usare **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **13-bot-framework** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella relativa al linguaggio scelto e aprire un terminale integrato.
3. Nel terminale eseguire i comandi seguenti per installare i modelli di bot e i pacchetti necessari:

**C#**

```C#
dotnet new -i Microsoft.Bot.Framework.CSharp.EchoBot
dotnet new -i Microsoft.Bot.Framework.CSharp.CoreBot
dotnet new -i Microsoft.Bot.Framework.CSharp.EmptyBot
```

**Python**

```Python
pip install botbuilder-core
pip install asyncio
pip install aiohttp
pip install cookiecutter==1.7.0
```

4. Dopo avere installato i modelli e i pacchetti, eseguire il comando seguente per creare un bot basato sul modello *EchoBot*:

**C#**

```C#
dotnet new echobot -n TimeBot
```

**Python**

```Python
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

Se si usa Python, quando richiesto da Cookiecutter, immettere i dati seguenti:
- **bot_name**: TimeBot
- **bot_description**: A bot for our times
    
5. Nel riquadro del terminale immettere i comandi seguenti per passare dalla directory corrente alla cartella **TimeBot** in cui sono presenti i file di codice generati per il bot:

    ```Code
    cd TimeBot
    dir
    ```

## <a name="test-the-bot-in-the-bot-framework-emulator"></a>Testare il bot in Bot Framework Emulator

È stato creato un bot basato sul modello *EchoBot*. È ora possibile eseguirlo in locale e testarlo usando Bot Framework Emulator (che deve essere installato nel sistema).

1. Nel riquadro del terminale verificare che la directory corrente sia la cartella **TimeBot** contenente i file di codice del bot e quindi immettere il comando seguente per avviare l'esecuzione del bot in locale.

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```
    
Quando il bot viene avviato, si noti che viene visualizzato l'endpoint in cui è in esecuzione. L'endopint sarà simile a **http://localhost:3978** .

2. Avviare Bot Framework Emulator e aprire il bot specificando l'endpoint e aggiungendo il percorso **/api/messages**, come illustrato di seguito:

    `http://localhost:3978/api/messages`

3. Dopo aver aperto la conversazione in un riquadro **Live chat**, attendere il messaggio *Hello and welcome!* .
4. Immettere un messaggio, ad esempio *Hello*, e visualizzare la risposta del bot, che ripeterà il messaggio immesso.
5. Chiudere Bot Framework Emulator e tornare a Visual Studio Code, quindi nella finestra del terminale immettere **CTRL+C** per arrestare il bot.

## <a name="modify-the-bot-code"></a>Modificare il codice del bot

È stato creato un bot che ripete l'input dell'utente. Non è particolarmente utile, ma serve a illustrare il flusso di base di una conversazione. Una conversazione con un bot è costituita da una sequenza di *attività*, in cui testo, grafica o *schede* dell'interfaccia utente vengono usati per scambiare informazioni. Il bot inizia la conversazione con un saluto, che è il risultato di un'attività di *aggiornamento della conversazione* che viene attivata quando un utente inizializza una sessione di chat con il bot. La conversazione è quindi costituita da una sequenza di altre attività in cui l'utente e il bot inviano *messaggi* a turno.

1. In Visual Studio Code aprire il file di codice seguente per il bot:
    - **C#** : TimeBot/Bots/EchoBot.cs
    - **Python**: TimeBot/bot.py

    Si noti che il codice in questo file è costituito da funzioni del *gestore attività*, una per l'attività di aggiornamento della conversazione *Member Added* (quando un utente si unisce alla sessione di chat) e un'altra per l'attività *Message* (quando viene ricevuto un messaggio). La conversazione si basa sul concetto di *turni*, in cui ogni turno rappresenta un'interazione in cui il bot riceve un'attività, la elabora e risponde. Il *contesto del turno* viene usato per tenere traccia delle informazioni sull'attività elaborata nel turno corrente.

2. Nella parte superiore del file di codice aggiungere l'istruzione di importazione dello spazio dei nomi seguente:

**C#**

```C#
using System;
```

**Python**

```Python
from datetime import datetime
```

3. Modificare la funzione del gestore attività per l'attività *Message* in modo che corrisponda al codice seguente:

**C#**

```C#
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    string inputMessage = turnContext.Activity.Text;
    string responseMessage = "Ask me what the time is.";
    if (inputMessage.ToLower().StartsWith("what") && inputMessage.ToLower().Contains("time"))
    {
        var now = DateTime.Now;
        responseMessage = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");
    }
    await turnContext.SendActivityAsync(MessageFactory.Text(responseMessage, responseMessage), cancellationToken);
}
```

**Python**

```Python
async def on_message_activity(self, turn_context: TurnContext):
    input_message = turn_context.activity.text
    response_message = 'Ask me what the time is.'
    if (input_message.lower().startswith('what') and 'time' in input_message.lower()):
        now = datetime.now()
        response_message = 'The time is {}:{:02d}.'.format(now.hour,now.minute)
    await turn_context.send_activity(response_message)
```
    
4. Salvare le modifiche e nel riquadro del terminale verificare che la directory corrente sia la cartella **TimeBot** contenente i file di codice del bot, quindi immettere il comando seguente per avviare l'esecuzione del bot in locale.

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```

Come in precedenza, quando il bot viene avviato, si noti che viene visualizzato l'endpoint in cui è in esecuzione.

5. Avviare Bot Framework Emulator e aprire il bot specificando l'endpoint e aggiungendo il percorso **/api/messages**, come illustrato di seguito:

    `http://localhost:3978/api/messages`

6. Dopo aver aperto la conversazione in un riquadro **Live chat**, attendere il messaggio *Hello and welcome!* .
7. Immettere un messaggio, ad esempio *Hello*, e visualizzare la risposta del bot, che sarà *Ask me what the time is*.
8. Immettere *What is the time?* e visualizzare la risposta.

    Il bot risponde ora alla query "What is the time?" visualizzando l'ora locale dove il bot è in esecuzione. Per qualsiasi altra query, viene richiesto all'utente di chiedere l'ora. Si tratta di un bot molto limitato, che può essere migliorato tramite l'integrazione con il servizio Language Understanding e codice personalizzato aggiuntivo, ma serve da esempio pratico di come creare una soluzione con Bot Framework SDK estendendo un bot creato da un modello.

9. Chiudere Bot Framework Emulator e tornare a Visual Studio Code, quindi nella finestra del terminale immettere **CTRL+C** per arrestare il bot.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni su Bot Framework, vedere la [documentazione di Bot Framework](https://docs.microsoft.com/azure/bot-service/index-bf-sdk).
