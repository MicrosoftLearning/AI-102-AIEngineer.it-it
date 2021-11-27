---
lab:
  title: Configurazione dell'ambiente lab
  module: Setup
ms.openlocfilehash: 57363ce9fec94cb3a1a6ef7622a10bca48a6f055
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625958"
---
# <a name="lab-environment-setup"></a>Configurazione dell'ambiente lab

Questi esercizi sono progettati per essere completati in un ambiente lab ospitato. Se si intende completarli nel proprio computer, è possibile eseguire questa operazione installando il software seguente. Quando si usa un ambiente personale, si potrebbero notare dialoghi e comportamenti imprevisti. Considerata l'ampia gamma di possibili configurazioni locali, il team del corso non può fornire assistenza per tutti i problemi che possono verificarsi in un ambiente personale.

> **Nota**: le istruzioni seguenti sono relative a un computer Windows 10. È anche possibile usare Linux o MacOS. Potrebbe essere necessario adattare le istruzioni del lab per il sistema operativo scelto.

### <a name="base-operating-system-windows-10"></a>Sistema operativo di base (Windows 10)

#### <a name="windows-10"></a>Windows 10

Installare Windows 10 e applicare tutti gli aggiornamenti.

#### <a name="edge"></a>Microsoft Edge

Installare [Edge (Chromium)](https://microsoft.com/edge)

### <a name="net-core-sdk"></a>.NET Core SDK

1. Scaricarlo e installarlo da https://dotnet.microsoft.com/download (scaricare .NET Core SDK, non solo il runtime)

### <a name="c-redistributable"></a>C++ Redistributable

1. Scaricare e installare il pacchetto Visual C++ Redistributable (x64) da https://aka.ms/vs/16/release/vc_redist.x64.exe.

### <a name="nodejs"></a>Node.JS

1. Scaricare la versione LTS più recente da https://nodejs.org/en/download/ 
2. Eseguire l'installazione con le opzioni predefinite

### <a name="python-and-required-packages"></a>Python (e pacchetti obbligatori)

1. Scaricare la versione 3.8 da https://docs.conda.io/en/latest/miniconda.html 
2. Eseguire il programma di installazione per installare. **Importante**: selezionare le opzioni per aggiungere Miniconda alla variabile PATH e registrare Miniconda come ambiente Python predefinito.
3. Dopo l'installazione, aprire il prompt di Anaconda e immettere i comandi seguenti per installare i pacchetti: 

```
pip install flask requests python-dotenv pylint matplotlib pillow
pip install --upgrade numpy
```

### <a name="azure-cli"></a>Interfaccia della riga di comando di Azure

1. Scaricarla da https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest 
2. Eseguire l'installazione con le opzioni predefinite

### <a name="git"></a>Git

1. Scaricarlo e installarlo da https://git-scm.com/download.html usando le opzioni predefinite


### <a name="visual-studio-code-and-extensions"></a>Visual Studio Code (ed estensioni)

1. Scaricarlo da https://code.visualstudio.com/Download 
2. Eseguire l'installazione con le opzioni predefinite 
3. Dopo l'installazione, avviare Visual Studio Code e nella scheda **Estensioni** (CTRL+MAIUSC+X), cercare e installare le estensioni seguenti di Microsoft:
    - Python
    - C#
    - Funzioni di Azure
    - PowerShell


### <a name="bot-framework-emulator"></a>Bot Framework Emulator

Seguire le istruzioni disponibili in https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md per scaricare e installare la versione stabile più recente di Bot Framework Emulator per il sistema operativo in uso.

### <a name="bot-framework-composer"></a>Bot Framework Composer

Installare da https://docs.microsoft.com/en-us/composer/install-composer.
