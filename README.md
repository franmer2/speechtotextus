In this article, I will detail the steps to turn a recorded discussion into a text audio file, enrich that transcript with sentiment analyses, and then end with a representation of that discussion in a Power report BI to make an analysis.

To do this, we will use several Azure services including mainly:

- An Azure storage account
- The cognitive service "Speech to text"
- Azure Logic App
- Azure Cosmos DB

Below is a diagram of the solution that we will put in place in Azure


![sparkle](Pictures/901.png)

You can also directly deploy the solution in your Azure subscription by clicking the button bellow:

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ffranmer2%2Fspeechtotextus%2Fmaster%2FARM%2520Files%2FTemplate.json)

Then you can test the solution by dropping the [audio files]((https://github.com/franmer2/speechtotextus/tree/master/AudioFile)) in the container 'audiofiles' created during the deployment. 


## Prerequisites

- An Azure subscription
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) 
  

# Azure services creation
## Create a resource group
We will start by creating a resource group in order to host the various services of our audio file transcription solution.

From [Azure portal](https://portal.azure.com), click on "**Create a resource**"

![sparkle](Pictures/001.png)

 Look for "**Resource group**"

 ![sparkle](Pictures/002.png)


Click on "**Create**"

![sparkle](Pictures/003.png)

In "**Resource group**" field, give a name to your resource group. Click on the button "**Review + Create**"

![sparkle](Pictures/004.png)

In the validation screen, click on the button"**Create**"

![sparkle](Pictures/005.png)

Return to the Azure portal home page. Click on the burger menu at the top left and then on "**Resource** **groups**"

![sparkle](Pictures/006.png)

Click on the resource group created previously

![sparkle](Pictures/007.png)

## Azure storage creation

Once in the resource group, click on the button "**Add**" 

![sparkle](Pictures/008.png)

Find the storage account

![sparkle](Pictures/009.png)

Click on "**Create**"

![sparkle](Pictures/010.png)

Complete the creation of the storage account and click on "**Review** + **create**"

![sparkle](Pictures/011.png)

After checking the information for creating the storage account, click on the button "**Create**"

![sparkle](Pictures/012.png)

## Azure Logic App creation
In the previously created resource group, click the button "**Add**"

![sparkle](Pictures/013.png)

Find "**Logic App**"

![sparkle](Pictures/014.png)

Click on "**Create**"

![sparkle](Pictures/015.png)

Complete the step to create the "Logic App" service, then click on "**Review + create**"

![sparkle](Pictures/016.png)

In the validation window, click on the button "**Create**"

![sparkle](Pictures/017.png)

## "Speech to text" cognitive service creation
In the previously created resource group, click the button "**Add**"

![sparkle](Pictures/018.png)

Find "**Speech**"

![sparkle](Pictures/019.png)

Click on "**Create**"

![sparkle](Pictures/020.png)

Complete the creation form, then click on the button"**Create**"

In the "**Pricing Tier**" field, select "**S0**", because only this tier supports batch transcription:

https://docs.microsoft.com/fr-fr/azure/cognitive-services/speech-service/batch-transcription#prerequisites


![sparkle](Pictures/021.png)

## Azure Cosmos DB account creation

In the previously created resource group, click the button "**Add**"

![sparkle](Pictures/022.png)

Find "**Azure Cosmos DB**"

![sparkle](Pictures/023.png)

Click on "**Create**"

![sparkle](Pictures/024.png)

Fill in the service delivery form. Choose API "**Core (SQL)**".
Click on "**Review + create**"

![sparkle](Pictures/025.png)

After validating the information, click on the button"**Create**"

![sparkle](Pictures/026.png)

# Services configuration
## Storage account
We will now create a container to receive the audio files

From your resource group, click the storage account service

![sparkle](Pictures/027.png)

Click on "**Containers**"

![sparkle](Pictures/028.png)

Click on "**+ Container**". Give a name to your container then click on the button "**Ok**".

![sparkle](Pictures/029.png)

The new container should appear in the list as shown below

![sparkle](Pictures/030.png)


### Shared accees signature creation (SAS Token)

In your storage account, in the left pane, click on "**Shared access signature**"

![sparkle](Pictures/904.png)

Define your key by choosing at least :

- Services
- Resource type
- Allowed permissions
- Expiration date

In our sample, we will check the boxes:

- "**Blob**"
- "**Object**" (Important)
- "**Read**"
- And choose an expiration date

Below a screenshot

![sparkle](Pictures/905.png)

After clicking on the button "**Generate SAS and connection string**", you can copy your key. We will need it a little later.

![sparkle](Pictures/906.png)



## Azure Cosmos DB collection creation

We will now create a collection to receive the text transciption of the audio file

From your resource group, click on Azure Cosmos DB

![sparkle](Pictures/031.png)

Double check you are in "**Overview**" blade, then Click on "**+ Add Container**"

![sparkle](Pictures/032.png)

In "**Add Container**", select "**Create new**" to create a new database. in this article, I choose "**Manual**" for "Throughput" and set value to **400**.

Finally, give a name to your collection and choose a partition key. Here I choose the date as the key, because for my project, we will have many files per day and we will have to make reports over a time range varying from week to month.

![sparkle](Pictures/033.png)

After creating the container you should get a result similar to the screenshot below:

![sparkle](Pictures/034.png)

## Azure Logic Apps configuration

To transcribe the audio files that will be deposited in the storage account, we will use the Batch transcription REST API:

https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/batch-transcription

The steps below will allow you to configure the Azure Logic App service to call the "Speech" cognitive service REST API, as soon as an audio file arrives in the container.

From your resource group, click on the Azure Logic Apps service

![sparkle](Pictures/035.png)

Click on "**Blank Logic App**"

![sparkle](Pictures/036.png)

You will be redirected to the Azure Logic Apps editor

![sparkle](Pictures/037.png)

### Création des paramètres

We will start by creating the different parameters that we will need. In addition, these parameters can be accessed in the ARM templates.

Click on "**Parameters**"

![sparkle](Pictures/200.png)

Click on "**Add parameters**

![sparkle](Pictures/907.png)

Add following parameters:

- cognitiveSecret (one of the cognitive service keys)
- azureRegion (Azure region name (here "**canadacentral**"))
- storageAccountName (storage account name (here "**audiofiledemo**"))
- SASToken (the shared access signature created previously)

Fill in the default values as shown below:

![sparkle](Pictures/201.png)

### Azure Logic App flow creation


In the search area, enter "**Blob**", then select "**When a blob is added or modified**"

![sparkle](Pictures/038.png)

Give a name to your connection channel then select the storage account created at the beginning of this article.

![sparkle](Pictures/039.png)

In "**Container**" field, click on the icon on the right (1) and select the container previously created in this article. Set 5 seconds of interval between each check for the presence of new files in the container.
 
![sparkle](Pictures/040.png)

Click on "**+ New step**" in order to create an additional action.

![sparkle](Pictures/041.png)

In the search box, enter the word "json" then click on "Data Operations"

![sparkle](Pictures/042.png)

Select "**Compose**"

![sparkle](Pictures/043.png)

In the input field, enter the JSON part below, knowing that we will complete the part framed in red later.


```javascript
{
  "description": "La description est optionnelle",
  "locale": "en-US",
  "name": "transcription-batch",
  "properties": {
    "AddSentiment": "True",
    "AddWordLevelTimestamps": "True",
    "ProfanityFilterMode": "Masked",
    "PunctuationMode": "DictatedAndAutomatic"
  },
  "recordingsUrl": 
}
```


![sparkle](Pictures/044.png)

Click to the right of "**recordingUrl**" to bring up the dynamic content window. Click on "**Expression**" and then on the function "**concat**" 

![sparkle](Pictures/045.png)



In the first part of the concatenation, enter the text: **'https://'**
Which will give the first part of our concatenation:
 
 ![sparkle](Pictures/202.png)

Then add a comma and the storage account parameters "**storageAccountName"**. Warning, parameters are under "**Dynamic content**".

![sparkle](Pictures/203.png)

Add "**,'.blob.core.windows.net',**"

![sparkle](Pictures/204.png)


We will add the specific part to the container and to the file by retrieving the information from the previous step. Click on "**Dynamic Content**" then  "**List of Files Path**"

![sparkle](Pictures/205.png)

Finally, we will add the information concerning the shared access signature (SAS token).

Add a comma then the parameter "**SASToken**"

![sparkle](Pictures/206.png)


Then click on "**OK**" (or *Update*).

![sparkle](Pictures/207.png)

So you should get something similar to the screenshot below:

![sparkle](Pictures/050.png)

Et le script de la fonction de concatenation pour le champ "**recordingUrl**" doit ressembler à celui ci-dessous :

```javascript
concat('https://',parameters('storageAccountName'),'.blob.core.windows.net',triggerBody()?['Path'],parameters('SASToken'))

```

Cliquez sur le bouton "**New step**"

![sparkle](Pictures/051.png)

dans la zone de recherche, entrez "http" puis cliquez sur l'icône "**HTTP**"

![sparkle](Pictures/052.png)

Puis sélectionnez "HTTP"

![sparkle](Pictures/053.png)

Nous allons maintenant envoyer le fichier audio au service cognitif.
Dans la configuration de cette étape, nous allons entrer les valeurs suivantes pour les champs ci-dessous (on fera la partie Body ultérieurement):

- **Method** : POST
- **URI** : concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions')
- **Headers** :
  
|               |                  |
| :------------------------ | -------------------------------: |
| Content-Type              |application/json                  |
| Ocp-Apim-Subscription-Key | *le paramètre de la clef de votre service cognitif*|


========================================================================

Comme vu plus haut, la clef de votre service cognitif se trouve dans la section "**Keys and Endpoint**" de votre service cognitif comme illustré ci-dessous

![sparkle](Pictures/054.png)

========================================================================

Cliquez dans le champ "**URI**" puis rajoutez dans "**Expression**" le script ci-dessous :

```javascript
concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions')
```

Cliquez sur le bouton "**OK**


![sparkle](Pictures/208.png)

Entrez les valeurs suivantes pour la partie "**Header**"

|               |                  |
| :------------------------ | -------------------------------: |
| Content-Type              |application/json                  |
| Ocp-Apim-Subscription-Key | *le paramètre de la clef de votre service cognitif*|

Nous allons donc obtenir au niveau de notre Azure Logic App, quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/209.png)

Nous allons maintenant définir le champ "**Body**".

Cliquez dans le champ "**Body**". Dans le menu contextuel, sélectionnez "**Dynamic content**", puis "**Ouput**" sous la partie "**Compose**"


![sparkle](Pictures/210.png)

Vous pouvez renommer vos étapes en cliquant sur les 3 petits points, puis sur "**Rename**". Par exemple, j'ai nommé cette étape "*Post to cognitive service*"

![sparkle](Pictures/211.png)



Vous devriez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous. 


![sparkle](Pictures/212.png)

Nous allons maintenant récupèrer l'ID de la transcription ainsi que l'URL du fichier audio retournés par le service cognitif
Cliquez sur le bouton "**New Step**"pour rajouter l'étape suivante :

![sparkle](Pictures/213.png)

dans le champ de recherche entrez "**Parse JSON**" puis sélectionnez "**Parse JSON**"

![sparkle](Pictures/214.png)

Cliquez dans le champ "**Content**" puis sur "**Body**" qui se trouve dans "**Dynamic content**" sous "**Post to cognitive service**"

![sparkle](Pictures/215.png)

Dans le champ "**Schema**", copiez le script ci-dessous :

```javascript
{
    "properties": {
        "createdDateTime": {
            "type": "string"
        },
        "id": {
            "type": "string"
        },
        "lastActionDateTime": {
            "type": "string"
        },
        "recordingsUrl": {
            "type": "string"
        },
        "status": {
            "type": "string"
        },
        "statusMessage": {
            "type": "string"
        }
    },
    "type": "object"
}
```
![sparkle](Pictures/216.png)

Cliquez sur le bouton "**New Step**"

![sparkle](Pictures/217.png)




Nous allons maintenant faire une boucle jusqu'à ce que le service cognitif nous retourne la valeur "**Succeeded**"

Dans le champ de recherche entrez le mot "**until**", puis sélectionnez le control "**Until**"

![sparkle](Pictures/058.png)

Cliquez sur "**Add an action**"

![sparkle](Pictures/059.png)

Puis recherchez le mot "**Delay**" et sélectionnez "**Delay**"

![sparkle](Pictures/060.png)

Paramétrez le délai sur quelques secondes. Dans cet exemple, j'ai paramétré sur 10 secondes. Cliquez sur "**Add an action**"

![sparkle](Pictures/061.png)

Recherchez http, puis sélectionnez "**HTTP**"

![sparkle](Pictures/062.png)

Ici, nous allons attendre une réponse du service cognitif.
Dans le champ "**Method**", sélectionnez "**GET**".

Pour le champ "**URI**" copiez la formule ci-dessous :

```javascript
concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions/',body('Parse_transcription_ID_and_result_URL')?['id'])

```

Pour "**Headers**"
|               |                  |
| :------------------------ | -------------------------------: |
| Content-Type              |application/json                  |
| Ocp-Apim-Subscription-Key | *le paramètre de la clef de votre service cognitif*|


![sparkle](Pictures/063.png)

Même si ce n'est pas forcément nécessaire, mais recommandé, il est possible de renommer les étapes. A droite de "HTTP 2", cliquez sur les 3 points puis sur "**Rename**"


![sparkle](Pictures/064.png)

Vous pouvez nommer cette étape "Get from API", par exemple. Cliquez sur "**Add an action**"

![sparkle](Pictures/065.png)

Nous allons maintenant récupérer le résultat de l'étape précédente et parser le message. Dans le champ de recherche, entrez "Parse JSON", puis sélectionnez "**Parse JSON**"

![sparkle](Pictures/066.png)

Cliquez dans le champ "**Content**". Sélectionnez "**Dynamic content**" puis "**Body**" sous "**Get from API**" 

![sparkle](Pictures/067.png)

Dans le champ "**Schema**", entrez le script JSON ci-dessous

```javascript
{
    "properties": {
        "id": {
            "type": "string"
        },
        "recordingsUrl": {
            "type": "string"
        },
        "status": {
            "type": "string"
        }
    },
    "type": "object"
}
```
Vous devez donc obtenir un résultat similaire à celui ci-dessous

![sparkle](Pictures/068.png)

Revenez au niveau de la boucle "**Until**" pour définir la condition de sortie. Cliquez dans le champ "**Choose a value**". Dans le menu contextuel, cliquez sur "**Dynamic content**" puis cliquez sur "**status**" sous "**Parse JSON**":


![sparkle](Pictures/069.png)

Ensuite définissez la condition à "**is equal to**" "**Succeeded**"

Vous devez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous. Cliquez sur "**New step**"

![sparkle](Pictures/070.png)


********************

A titre de précaution, sauvegardez de temps en temps votre travail :

![sparkle](Pictures/900.png)

********************




Rechechez l'opération "**Parse JSON**". Cliquez dans le champ "**Content**", puis "**Dynamic content**" puis sur "**Body**" sous "**Get from API**"

![sparkle](Pictures/071.png)

Dans le champ "**Schema**", entrez le script JSON ci-dessous :

```javascript
{
    "properties": {
        "createdDateTime": {
            "type": "string"
        },
        "description": {
            "type": "string"
        },
        "id": {
            "type": "string"
        },
        "lastActionDateTime": {
            "type": "string"
        },
        "locale": {
            "type": "string"
        },
        "name": {
            "type": "string"
        },
        "recordingsUrl": {
            "type": "string"
        },
        "reportFileUrl": {
            "type": "string"
        },
        "resultsUrls": {
            "properties": {
                "channel_0": {
                    "type": "string"
                },
                "channel_1": {
                    "type": "string"
                }
            },
            "type": "object"
        },
        "status": {
            "type": "string"
        },
        "statusMessage": {
            "type": "string"
        }
    },
    "type": "object"
}

```

(pour obtenir le fichier JSON ci-dessous, j'ai fait un test manuel avec Postman. Je détail cette opération à la fin de cet article)

Vous devriez obtenir quelque chose de similaire à la copie d'écran ci-dessous. Cliquez sur le bouton "**New step**" :

![sparkle](Pictures/072.png)

Dans le champ de recherche entrez "**Condition**" puis cliquez sur "**Condition**"

![sparkle](Pictures/073.png)


Clicquez sur le bouton "**Add**" puis "**Add row**" pour rajouter une seconde condition.

![sparkle](Pictures/074.png)



Cliquez sur "**Choose a value**" puis dans "**Dynamic content**", sélectionnez "**channel_0**" sous "**Parse JSON 2**". Cliquez sur "**OK**".



![sparkle](Pictures/075.png)

Choisissez "**is not equal to**" pour le test, puis l'expression "**null**" comme valeur à tester.

![sparkle](Pictures/076.png)

Répétez l'opération pour la seconde ligne mais pour "**Channel_1**". Vous devriez obtenir quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/077.png)

Vous pouvez renommer cette étape pour plus de clarté. par exemple j'ai nommé cette étape "**Test if channel 0 and 1 is not empty**"


![sparkle](Pictures/079.png)


Dans la partie "**If true**", cliquez sur "**Add an action**"

![sparkle](Pictures/078.png)


Dans le champ de recherche entrez "**http**", puis sélectionnez "**HTTP**

![sparkle](Pictures/080.png)


Dans le champ "**Method**", choisissez "**Get**", puis dans le champ "**URI**" sélectionnez "**Dynamic content**" puis "**channel_0**" sous "**Parse JSON 2**"

Vous pouvez renommer cette étape en quelque chose comme "*Get channel 0 results*". 

Cliquez sur "**Add an action**"

![sparkle](Pictures/081.png)


Dans le champ recherche, entrez "**parse json**" et cliquez sur "**Parse JSON**"

![sparkle](Pictures/082.png)

Dans le champ "**Content**", cliquez sur "**Dynamic content**" puis sur "**Body**" sous "**Get channel 0 results**

![sparkle](Pictures/083.png)

Dans le champ "**Schema**", copiez le script ci-dessous :

```javascript
{
    "properties": {
        "AudioFileResults": {
            "items": {
                "properties": {
                    "AudioFileName": {
                        "type": "string"
                    },
                    "AudioFileUrl": {},
                    "AudioLengthInSeconds": {
                        "type": "number"
                    },
                    "CombinedResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Display": {
                                    "type": "string"
                                },
                                "ITN": {
                                    "type": "string"
                                },
                                "Lexical": {
                                    "type": "string"
                                },
                                "MaskedITN": {
                                    "type": "string"
                                }
                            },
                            "required": [
                                "ChannelNumber",
                                "Lexical",
                                "ITN",
                                "MaskedITN",
                                "Display"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "SegmentResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Duration": {
                                    "type": "integer"
                                },
                                "DurationInSeconds": {
                                    "type": "number"
                                },
                                "NBest": {
                                    "items": {
                                        "properties": {
                                            "Confidence": {
                                                "type": "number"
                                            },
                                            "Display": {
                                                "type": "string"
                                            },
                                            "ITN": {
                                                "type": "string"
                                            },
                                            "Lexical": {
                                                "type": "string"
                                            },
                                            "MaskedITN": {
                                                "type": "string"
                                            },
                                            "Sentiment": {
                                                "properties": {
                                                    "Negative": {
                                                        "type": "number"
                                                    },
                                                    "Neutral": {
                                                        "type": "number"
                                                    },
                                                    "Positive": {
                                                        "type": "number"
                                                    }
                                                },
                                                "type": "object"
                                            },
                                            "Words": {
                                                "items": {
                                                    "properties": {
                                                        "Confidence": {
                                                            "type": "integer"
                                                        },
                                                        "Duration": {
                                                            "type": "integer"
                                                        },
                                                        "DurationInSeconds": {
                                                            "type": "integer"
                                                        },
                                                        "Offset": {
                                                            "type": "integer"
                                                        },
                                                        "OffsetInSeconds": {
                                                            "type": "number"
                                                        },
                                                        "Word": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "required": [
                                                        "Word",
                                                        "Offset",
                                                        "Duration",
                                                        "OffsetInSeconds",
                                                        "DurationInSeconds",
                                                        "Confidence"
                                                    ],
                                                    "type": "object"
                                                },
                                                "type": "array"
                                            }
                                        },
                                        "required": [
                                            "Confidence",
                                            "Lexical",
                                            "ITN",
                                            "MaskedITN",
                                            "Display",
                                            "Sentiment",
                                            "Words"
                                        ],
                                        "type": "object"
                                    },
                                    "type": "array"
                                },
                                "Offset": {
                                    "type": "integer"
                                },
                                "OffsetInSeconds": {
                                    "type": "number"
                                },
                                "RecognitionStatus": {
                                    "type": "string"
                                },
                                "SpeakerId": {}
                            },
                            "required": [
                                "RecognitionStatus",
                                "ChannelNumber",
                                "SpeakerId",
                                "Offset",
                                "Duration",
                                "OffsetInSeconds",
                                "DurationInSeconds",
                                "NBest"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    }
                },
                "required": [
                    "AudioFileName",
                    "AudioFileUrl",
                    "AudioLengthInSeconds",
                    "CombinedResults",
                    "SegmentResults"
                ],
                "type": "object"
            },
            "type": "array"
        }
    },
    "type": "object"
}
```

Vous devez obtenir un résultat similaire à la copie d'écran ci-dessous. 

Renommez l'étape par quelque chose comme "**Parse Channel 0**".

Cliquez sur "**Add an action**".

![sparkle](Pictures/084.png)

Dans le champ de recherche entrez "**http**" puis cliquez sur "**HTTP**".

![sparkle](Pictures/085.png)

Dans le champ "**Method**", choisissez "**Get**", puis dans le champ "**URI**" sélectionnez "**Dynamic content**" puis "**channel_1**" sous "**Parse JSON 2**".

Vous pouvez renommer cette étape en quelque chose comme "*Get channel 1 results*". 

Cliquez sur "**Add an action**".

![sparkle](Pictures/086.png)

Dans le champ de recherche, entrez "**parse json**" puis cliquez sur "**Parse JSON**".

![sparkle](Pictures/087.png)

Cliquez dans le champ "**Content**", puis sur "**Dynamic content**". Cliquez sur "**Body**" qui se trouve sous "**Get channel 1 results**".

![sparkle](Pictures/088.png)

Dans le champ "**Schema**", copiez le script ci-dessous :

```javascript
{
    "properties": {
        "AudioFileResults": {
            "items": {
                "properties": {
                    "AudioFileName": {
                        "type": "string"
                    },
                    "AudioFileUrl": {},
                    "AudioLengthInSeconds": {
                        "type": "number"
                    },
                    "CombinedResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Display": {
                                    "type": "string"
                                },
                                "ITN": {
                                    "type": "string"
                                },
                                "Lexical": {
                                    "type": "string"
                                },
                                "MaskedITN": {
                                    "type": "string"
                                }
                            },
                            "required": [
                                "ChannelNumber",
                                "Lexical",
                                "ITN",
                                "MaskedITN",
                                "Display"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "SegmentResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Duration": {
                                    "type": "integer"
                                },
                                "DurationInSeconds": {
                                    "type": "number"
                                },
                                "NBest": {
                                    "items": {
                                        "properties": {
                                            "Confidence": {
                                                "type": "number"
                                            },
                                            "Display": {
                                                "type": "string"
                                            },
                                            "ITN": {
                                                "type": "string"
                                            },
                                            "Lexical": {
                                                "type": "string"
                                            },
                                            "MaskedITN": {
                                                "type": "string"
                                            },
                                            "Sentiment": {
                                                "properties": {
                                                    "Negative": {
                                                        "type": "number"
                                                    },
                                                    "Neutral": {
                                                        "type": "number"
                                                    },
                                                    "Positive": {
                                                        "type": "number"
                                                    }
                                                },
                                                "type": "object"
                                            },
                                            "Words": {
                                                "items": {
                                                    "properties": {
                                                        "Confidence": {
                                                            "type": "integer"
                                                        },
                                                        "Duration": {
                                                            "type": "integer"
                                                        },
                                                        "DurationInSeconds": {
                                                            "type": "integer"
                                                        },
                                                        "Offset": {
                                                            "type": "integer"
                                                        },
                                                        "OffsetInSeconds": {
                                                            "type": "number"
                                                        },
                                                        "Word": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "required": [
                                                        "Word",
                                                        "Offset",
                                                        "Duration",
                                                        "OffsetInSeconds",
                                                        "DurationInSeconds",
                                                        "Confidence"
                                                    ],
                                                    "type": "object"
                                                },
                                                "type": "array"
                                            }
                                        },
                                        "required": [
                                            "Confidence",
                                            "Lexical",
                                            "ITN",
                                            "MaskedITN",
                                            "Display",
                                            "Sentiment",
                                            "Words"
                                        ],
                                        "type": "object"
                                    },
                                    "type": "array"
                                },
                                "Offset": {
                                    "type": "integer"
                                },
                                "OffsetInSeconds": {
                                    "type": "number"
                                },
                                "RecognitionStatus": {
                                    "type": "string"
                                },
                                "SpeakerId": {}
                            },
                            "required": [
                                "RecognitionStatus",
                                "ChannelNumber",
                                "SpeakerId",
                                "Offset",
                                "Duration",
                                "OffsetInSeconds",
                                "DurationInSeconds",
                                "NBest"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    }
                },
                "required": [
                    "AudioFileName",
                    "AudioFileUrl",
                    "AudioLengthInSeconds",
                    "CombinedResults",
                    "SegmentResults"
                ],
                "type": "object"
            },
            "type": "array"
        }
    },
    "type": "object"
}
```

Renommez cette étape en quelque chose comme "*Parse Channel 1*".

Cliquez sur "**Add an action**"

![sparkle](Pictures/089.png)

Dans la champ de recherche, "**compose**", puis cliquez sur "**Compose**".

![sparkle](Pictures/090.png)

Renommez cette étape par quelque chose comme "*Compose document with channel 0 and 1*".

Dans le champ "**Inputs**", copiez le script ci-dessous :


```javascript
{
  "AudioDetails": [
    {
      "Channel_0": ,
      "Channel_1": 
    }
  ],
  "AudioLink": "",
  "Date": "",
  "Language": "English",
  "MyDate": "",
  "id": ""
}

```
Vous devez obtenir un résultat similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/091.png)

Cliquez à droite de **"Channel_0":** et juste avant la virgule. Rajoutez "**Body**" qui se trouve dans "**Dynamic content**" sous "**Parse Channel 0**"

![sparkle](Pictures/092.png)

Mainenant, cliquez à droite de **"Channel_1":**. Rajoutez "**Body**" qui se trouve dans "**Dynamic content**" sous "**Parse Channel 1**"

![sparkle](Pictures/093.png)

Cliquez à droite de **"AudioLink":**, entre les doubles quotes, puis rajoutez "**recordingUrl**" qui se trouve dans "**Dynamic content**" sous "**Parse transcription ID and result URL**"

![sparkle](Pictures/094.png)

Cliquez à droite de **"Date":**, entre les doubles quotes, puis rajoutez l'expression
```javascript
utcNow()
```

Cliquez sur "**OK**"

![sparkle](Pictures/095.png)

Cliquez à droite de **"MyDate":**, entre les doubles quotes, puis rajoutez l'expression


```javascript
formatDateTime(utcNow(),'yyyy-MM-dd')
```

Cliquez sur "**OK**"

![sparkle](Pictures/096.png)

Cliquez à droite de **"id":**, entre les doubles quotes, puis cliquez sur "**id**" qui se trouve dans "**Dynamic content**" sous "**Parse transcription ID and result URL**"

![sparkle](Pictures/097.png)

Vous devez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous.

Cliquez sur "**Add an action**"

![sparkle](Pictures/098.png)

Dans le champ de recherche, entrez "**parse**" puis cliquez sur "**Parse JSON**"

![sparkle](Pictures/099.png)

Renommez l'étape en quelque chose comme "*Parse document with channel 0 and 1 and additional information*"

Dans le champ "**Content**", cliquez sur "**Outputs**" qui se trouve dans "**Dynamic content**" sous "**Compose document with channel 0 and 1**"


![sparkle](Pictures/100.png)


Dans le chanp "**Schema**", rajoutez le script ci-dessous :

```javascript
{
    "properties": {
        "AudioDetails": {
            "items": {
                "properties": {
                    "Channel_0": {
                        "properties": {
                            "AudioFileResults": {
                                "items": {
                                    "properties": {
                                        "AudioFileName": {
                                            "type": "string"
                                        },
                                        "AudioFileUrl": {},
                                        "SegmentResults": {
                                            "items": {
                                                "properties": {
                                                    "ChannelNumber": {},
                                                    "Duration": {
                                                        "type": "integer"
                                                    },
                                                    "NBest": {
                                                        "items": {
                                                            "properties": {
                                                                "Confidence": {
                                                                    "type": "number"
                                                                },
                                                                "Display": {
                                                                    "type": "string"
                                                                },
                                                                "ITN": {
                                                                    "type": "string"
                                                                },
                                                                "Lexical": {
                                                                    "type": "string"
                                                                },
                                                                "MaskedITN": {
                                                                    "type": "string"
                                                                },
                                                                "Words": {
                                                                    "items": {
                                                                        "properties": {
                                                                            "Duration": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Offset": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Word": {
                                                                                "type": "string"
                                                                            }
                                                                        },
                                                                        "required": [
                                                                            "Word",
                                                                            "Offset",
                                                                            "Duration"
                                                                        ],
                                                                        "type": "object"
                                                                    },
                                                                    "type": "array"
                                                                }
                                                            },
                                                            "required": [
                                                                "Confidence",
                                                                "Lexical",
                                                                "ITN",
                                                                "MaskedITN",
                                                                "Display",
                                                                "Words"
                                                            ],
                                                            "type": "object"
                                                        },
                                                        "type": "array"
                                                    },
                                                    "Offset": {
                                                        "type": "integer"
                                                    },
                                                    "RecognitionStatus": {
                                                        "type": "string"
                                                    },
                                                    "SpeakerId": {}
                                                },
                                                "required": [
                                                    "RecognitionStatus",
                                                    "ChannelNumber",
                                                    "SpeakerId",
                                                    "Offset",
                                                    "Duration",
                                                    "NBest"
                                                ],
                                                "type": "object"
                                            },
                                            "type": "array"
                                        }
                                    },
                                    "required": [
                                        "AudioFileName",
                                        "AudioFileUrl",
                                        "SegmentResults"
                                    ],
                                    "type": "object"
                                },
                                "type": "array"
                            }
                        },
                        "type": "object"
                    },
                    "Channel_1": {
                        "properties": {
                            "AudioFileResults": {
                                "items": {
                                    "properties": {
                                        "AudioFileName": {
                                            "type": "string"
                                        },
                                        "AudioFileUrl": {},
                                        "SegmentResults": {
                                            "items": {
                                                "properties": {
                                                    "ChannelNumber": {},
                                                    "Duration": {
                                                        "type": "integer"
                                                    },
                                                    "NBest": {
                                                        "items": {
                                                            "properties": {
                                                                "Confidence": {
                                                                    "type": "number"
                                                                },
                                                                "Display": {
                                                                    "type": "string"
                                                                },
                                                                "ITN": {
                                                                    "type": "string"
                                                                },
                                                                "Lexical": {
                                                                    "type": "string"
                                                                },
                                                                "MaskedITN": {
                                                                    "type": "string"
                                                                },
                                                                "Words": {
                                                                    "items": {
                                                                        "properties": {
                                                                            "Duration": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Offset": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Word": {
                                                                                "type": "string"
                                                                            }
                                                                        },
                                                                        "required": [
                                                                            "Word",
                                                                            "Offset",
                                                                            "Duration"
                                                                        ],
                                                                        "type": "object"
                                                                    },
                                                                    "type": "array"
                                                                }
                                                            },
                                                            "required": [
                                                                "Confidence",
                                                                "Lexical",
                                                                "ITN",
                                                                "MaskedITN",
                                                                "Display",
                                                                "Words"
                                                            ],
                                                            "type": "object"
                                                        },
                                                        "type": "array"
                                                    },
                                                    "Offset": {
                                                        "type": "integer"
                                                    },
                                                    "RecognitionStatus": {
                                                        "type": "string"
                                                    },
                                                    "SpeakerId": {}
                                                },
                                                "required": [
                                                    "RecognitionStatus",
                                                    "ChannelNumber",
                                                    "SpeakerId",
                                                    "Offset",
                                                    "Duration",
                                                    "NBest"
                                                ],
                                                "type": "object"
                                            },
                                            "type": "array"
                                        }
                                    },
                                    "required": [
                                        "AudioFileName",
                                        "AudioFileUrl",
                                        "SegmentResults"
                                    ],
                                    "type": "object"
                                },
                                "type": "array"
                            }
                        },
                        "type": "object"
                    }
                },
                "required": [
                    "Channel_0",
                    "Channel_1"
                ],
                "type": "object"
            },
            "type": "array"
        },
        "AudioLink": {
            "type": "string"
        },
        "Date": {
            "type": "string"
        },
        "Language": {
            "type": "string"
        },
        "MyDate": {
            "type": "string"
        },
        "id": {
            "type": "string"
        }
    },
    "type": "object"
}
```

Vous devez obtenir quelque chose de similaire à la copie d'écran ci-dessous.

Cliquez sur "**Add an action**".

![sparkle](Pictures/101.png)


Dans le champ de recherche, entrez "**cosmos**", puis cliquez sur "**Create or update document**"


![sparkle](Pictures/102.png)


*********************************
Il est fort probable que Azure Logic App vous demande de créer une connection vers votre compte Azure Cosmos DB.

Choissisez votre compte et cliquez sur le bouton "**Create**"

![sparkle](Pictures/107.png)


*********************************


Complétez les champs "**Database ID**" et "**Collection ID**" avec les informations de votre compte Azure Cosmos DB

![sparkle](Pictures/103.png)

Cliquez dans le champ "**Document**", puis sur "**Body**" qui se trouve dans "**Dynamic content**" sous "**Parse document with channel 0 and 1**"


![sparkle](Pictures/104.png)


Cliquez sur "**Add new parameter**"


![sparkle](Pictures/105.png)


Puis choisissez "**Partition key value**"


![sparkle](Pictures/106.png)

Puis **entre des doubles quotes**, rajoutez le champ "**MyDate**" qui se trouve dans "**Dynamic content**" sous "**Parse document with channel 0 and 1**"


![sparkle](Pictures/108.png)

Nous allons maintenant traiter les fichiers audio avec seulement un seul canal. Comme les étapes sont quasiment identiques à celles que nous venons de voir, je ne vais pas les détailler.

Ci-dessous une vue globale de la branche "**If false**" de notre condition :

![sparkle](Pictures/109.png)



Dans la partie "**If false**" de la première condition "**Test if channel 0 and 1 is not empty**", rajoutez une seconde condition pour ne tester que le canal 0. 

![sparkle](Pictures/218.png)


Dans la partie "**If true**" de notre seconde condition, rajoutez le traitement du canal 0 uniquement, en suivant les étapes précédentes **mais avec les nuances suivantes** :

Pour l'étape HTTP :

![sparkle](Pictures/219.png)

Pour l'étape Parse JSON

![sparkle](Pictures/220.png)

Copiez le script ci-dessous dans le champ schema :

```javascript
{
    "properties": {
        "AudioFileResults": {
            "items": {
                "properties": {
                    "AudioFileName": {
                        "type": "string"
                    },
                    "AudioFileUrl": {},
                    "AudioLengthInSeconds": {
                        "type": "number"
                    },
                    "CombinedResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Display": {
                                    "type": "string"
                                },
                                "ITN": {
                                    "type": "string"
                                },
                                "Lexical": {
                                    "type": "string"
                                },
                                "MaskedITN": {
                                    "type": "string"
                                }
                            },
                            "required": [
                                "ChannelNumber",
                                "Lexical",
                                "ITN",
                                "MaskedITN",
                                "Display"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "SegmentResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Duration": {
                                    "type": "integer"
                                },
                                "DurationInSeconds": {
                                    "type": "number"
                                },
                                "NBest": {
                                    "items": {
                                        "properties": {
                                            "Confidence": {
                                                "type": "number"
                                            },
                                            "Display": {
                                                "type": "string"
                                            },
                                            "ITN": {
                                                "type": "string"
                                            },
                                            "Lexical": {
                                                "type": "string"
                                            },
                                            "MaskedITN": {
                                                "type": "string"
                                            },
                                            "Sentiment": {
                                                "properties": {
                                                    "Negative": {
                                                        "type": "number"
                                                    },
                                                    "Neutral": {
                                                        "type": "number"
                                                    },
                                                    "Positive": {
                                                        "type": "number"
                                                    }
                                                },
                                                "type": "object"
                                            },
                                            "Words": {
                                                "items": {
                                                    "properties": {
                                                        "Confidence": {
                                                            "type": "integer"
                                                        },
                                                        "Duration": {
                                                            "type": "integer"
                                                        },
                                                        "DurationInSeconds": {
                                                            "type": "integer"
                                                        },
                                                        "Offset": {
                                                            "type": "integer"
                                                        },
                                                        "OffsetInSeconds": {
                                                            "type": "number"
                                                        },
                                                        "Word": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "required": [
                                                        "Word",
                                                        "Offset",
                                                        "Duration",
                                                        "OffsetInSeconds",
                                                        "DurationInSeconds",
                                                        "Confidence"
                                                    ],
                                                    "type": "object"
                                                },
                                                "type": "array"
                                            }
                                        },
                                        "required": [
                                            "Confidence",
                                            "Lexical",
                                            "ITN",
                                            "MaskedITN",
                                            "Display",
                                            "Sentiment",
                                            "Words"
                                        ],
                                        "type": "object"
                                    },
                                    "type": "array"
                                },
                                "Offset": {
                                    "type": "integer"
                                },
                                "OffsetInSeconds": {
                                    "type": "number"
                                },
                                "RecognitionStatus": {
                                    "type": "string"
                                },
                                "SpeakerId": {}
                            },
                            "required": [
                                "RecognitionStatus",
                                "ChannelNumber",
                                "SpeakerId",
                                "Offset",
                                "Duration",
                                "OffsetInSeconds",
                                "DurationInSeconds",
                                "NBest"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    }
                },
                "required": [
                    "AudioFileName",
                    "AudioFileUrl",
                    "AudioLengthInSeconds",
                    "CombinedResults",
                    "SegmentResults"
                ],
                "type": "object"
            },
            "type": "array"
        }
    },
    "type": "object"
}
```



Pour l'étape "**Compose for only channel 0**"

![sparkle](Pictures/110.png)
 

Pour l'étape "**Parse document for Channel 0** ", utilisez le script ci-dessous :

```javascript
{
    "properties": {
        "AudioDetails": {
            "items": {
                "properties": {
                    "Channel_0": {
                        "properties": {
                            "AudioFileResults": {
                                "items": {
                                    "properties": {
                                        "AudioFileName": {
                                            "type": "string"
                                        },
                                        "AudioFileUrl": {},
                                        "SegmentResults": {
                                            "items": {
                                                "properties": {
                                                    "ChannelNumber": {},
                                                    "Duration": {
                                                        "type": "integer"
                                                    },
                                                    "NBest": {
                                                        "items": {
                                                            "properties": {
                                                                "Confidence": {
                                                                    "type": "number"
                                                                },
                                                                "Display": {
                                                                    "type": "string"
                                                                },
                                                                "ITN": {
                                                                    "type": "string"
                                                                },
                                                                "Lexical": {
                                                                    "type": "string"
                                                                },
                                                                "MaskedITN": {
                                                                    "type": "string"
                                                                },
                                                                "Words": {
                                                                    "items": {
                                                                        "properties": {
                                                                            "Duration": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Offset": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Word": {
                                                                                "type": "string"
                                                                            }
                                                                        },
                                                                        "required": [
                                                                            "Word",
                                                                            "Offset",
                                                                            "Duration"
                                                                        ],
                                                                        "type": "object"
                                                                    },
                                                                    "type": "array"
                                                                }
                                                            },
                                                            "required": [
                                                                "Confidence",
                                                                "Lexical",
                                                                "ITN",
                                                                "MaskedITN",
                                                                "Display",
                                                                "Words"
                                                            ],
                                                            "type": "object"
                                                        },
                                                        "type": "array"
                                                    },
                                                    "Offset": {
                                                        "type": "integer"
                                                    },
                                                    "RecognitionStatus": {
                                                        "type": "string"
                                                    },
                                                    "SpeakerId": {}
                                                },
                                                "required": [
                                                    "RecognitionStatus",
                                                    "ChannelNumber",
                                                    "SpeakerId",
                                                    "Offset",
                                                    "Duration",
                                                    "NBest"
                                                ],
                                                "type": "object"
                                            },
                                            "type": "array"
                                        }
                                    },
                                    "required": [
                                        "AudioFileName",
                                        "AudioFileUrl",
                                        "SegmentResults"
                                    ],
                                    "type": "object"
                                },
                                "type": "array"
                            }
                        },
                        "type": "object"
                    }
                },
                "required": [
                    "Channel_0"
                ],
                "type": "object"
            },
            "type": "array"
        },
        "AudioLink": {
            "type": "string"
        },
        "Date": {
            "type": "string"
        },
        "Language": {
            "type": "string"
        },
        "MyDate": {
            "type": "string"
        },
        "id": {
            "type": "string"
        }
    },
    "type": "object"
}
```
Vous devez obtenir un résulat similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/111.png)

Puis pour l'étape Azure Cosmos DB

![sparkle](Pictures/221.png)


Nous allons maintenant rajouter la dernière étape de notre Logic App. Cette étape consiste à supprimer les informations de transcription qui restent stockés au niveau du service cognitif afin de laisser la place libre pour la prochaine transcription.

Cliquez sur le bouton "**New step**"



Dans la zone de recherche, entrez "**http**" puis cliquez sur "**HTTP**"

![sparkle](Pictures/121.png)

Dans le champ "**Method**", sélectionnez "**DELETE**".
Cliquez dans le champ URI, puis cliquez sur "**Expression**". Collez le script ci-dessous. Cliquez sur le bouton "**OK**"


```javascript
concat('https://<your-cognitive-service-region>.cris.ai/api/speechtotext/v2.0/Transcriptions/',body('Parse_JSON')[0]['id'])
```

![sparkle](Pictures/122.png)


Remplacez "<your-cognitive-service-region"> par la region de votre service cognitif. Dans mon cas, c'est *canadacentral*

Enfin, dans le champ "**header**", rajoutez les information suivantes :

| Content-Type              |                 application/json |
| :------------------------ | -------------------------------: |
| Ocp-Apim-Subscription-Key | *la clef de votre service cognitif*|

Comme illustré ci-dessous

![sparkle](Pictures/123.png)

Cliquez sur le bouton "**Save**"

![sparkle](Pictures/124.png)























# Test the solution

Now that we have completed our logic app, we will make sure that our service is well in status "**Enabled**" 


![sparkle](Pictures/129.png)

If not, click the button "**Enable**"

![sparkle](Pictures/130.png)

With Azure Storage Explorer, place the audio file, which is in the [AudioFile](https://github.com/franmer2/speechtotextus/tree/master/AudioFile) folder, into your storage account. Your Azure Logic App must then trigger and start the audio file transcription process

![sparkle](Pictures/131.png)

If all goes well, the operation must proceed successfully

![sparkle](Pictures/132.png)

Click on "**Succeeded**" (or "Failed" depending on the case :)) for more details.

![sparkle](Pictures/133.png)

Check out the result in your Azure Cosmos DB collection.

![sparkle](Pictures/134.png)


# Power BI
Now that the file data is stored in an Azure Cosmos DB database, it's natural to visualize that data via Power BI. I'm not going to detail how to make the report but just illustrate how to get started. And I also [share](https://github.com/franmer2/speechtotextus/tree/master/Power%20BI) with you a report I made, for example.

From Power BI Desktop, click on "**Get Data**" then on "**More**"

![sparkle](Pictures/137.png)

Click on "**Azure**" then on "**Azure Cosmos DB**". Click on "**Connect**"

![sparkle](Pictures/138.png)

Enter the URL of your Azure Cosmos DB account (which you can find on the Azure portal), then click the "**OK**" button

![sparkle](Pictures/139.png)

Enter the key to your Azure Cosmos DB account (which you can find on the Azure portal). Click the "**Connect**" button

![sparkle](Pictures/140.png)

You should be able to navigate your databases and collections. Choose your collection and click the "**Transform Data**" button

![sparkle](Pictures/141.png)

In Power Query Editor, click on the arrows to access the fields in your collection

![sparkle](Pictures/142.png)

Choose the fields you need and click the "**OK**" button

![sparkle](Pictures/143.png)

For columns that still have double arrows, click on them to choose which fields you want to integrate into the report. It is possible to repeat this operation several times, it all depends on the depth of the structure of the document.

![sparkle](Pictures/144.png)

Below is an example of a Power BI report (available under the folder [*Power BI*](https://github.com/franmer2/speechtotextus/tree/master/Power%20BI)). I added the possibility of doing a "**Drillthrough**" to get the details of the conversations.

![sparkle](Pictures/902.png)


To represent the discussion in the report, I used a Gantt Diagram visualization

![sparkle](Pictures/145.png)

# Use parameters

In Azure Logic App, it is possible to use parameters, which can be useful to use them in * ARM templates *. For the sake of simplicity, I have not illustrated the parameters in this article, but consider using them as much as possible.

![sparkle](Pictures/152.png)

Then, these parameters will be usable in your Azure Logic App but above all, accessible at the ARM Template Level.

![sparkle](Pictures/153.png)


# What if I have files in another language?

I had to process files in English and French.
At the time i did this project (summer 2019 :)), I created another container to receive the files in French, I cloned my Logic App, in which I changed the trigger to point it to the new container . Then I changed the configuration of the call to cognitive service to pass the French language as a parameter. With a big restriction! Sentiment analysis is not supported, so you must set this parameter to "** False **" :( !
![sparkle](Pictures/146.png)

The list of supported languages is available [here](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/language-support)

Today, it may be possible to detect the language of the audio file and pass it as a parameter, but I have not looked at this point.

# Regarding security

In order to secure access to audio files by the cognitive service, it is possible, for example, to use a "Shared Access Signature". With Azure Storage Explorer, it's easy to create one, as shown below:

![sparkle](Pictures/147.png)

Define the level of security required as well as the expiration date. Click on the "**Create**" button

![sparkle](Pictures/148.png)

Copy the value found in the "Query String" field

![sparkle](Pictures/149.png)

In Azure Logip App, add a new parameter

![sparkle](Pictures/150.png)

Then in the "**Compose**" step, modify the "**recordingsUrl**" field to add the "**Shared Access Signature**"


![sparkle](Pictures/151.png)

# Under the hood

You may ask yourself about the long scripts that need to be added during the "Parse JSON" steps and how to generate them. To do this, I used the tool [Postman](https://www.postman.com/) (but I'm not saying it's the best way to do). With Postman, I manually sent an audio file to the cognitive service. With the Postman tool again, I get the results in JSON format.

Below, the Postman's setup to send an audio file to cognitive service.

"**Headers**" configuration

![sparkle](Pictures/125.png)

A "**Body**" configuration example

![sparkle](Pictures/126.png)

We get the result with the "**Get**" command

![sparkle](Pictures/127.png)

I copy the result in the "**Use sample payload to generate schema**" optin, in the "**Parse JSON**" Azure Logic App step.

![sparkle](Pictures/128.png)

# Troubleshooting

One of the errors I had during the tests was that the cognitive service kept the information from the previous audio files. That's why the last stage of the Logic App is a "Delete" action in cognitive service.

In case you have an error with Azure Logic app, you can check with Postman, with a 'GET' command, if information is still present at the cognitive service level. If this is the case, copy the id of the returned document.

![sparkle](Pictures/135.png)


Then use this id to send a "DELETE" command with Postman, in the way shown below:


![sparkle](Pictures/136.png)



