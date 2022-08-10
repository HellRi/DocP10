# Outil de suivi et analyse

## Le contexte

Nous sommes **Fly Me**, une agence proposant des voyages "clef en main" pour ses clients.
Nous sommes désireux de créer un ChatBot permettant d'apporter un support à nous utilisateurs, afin de choisir une offre optimale, de voyage.
Nous retrouverons ici, une présentation de la ressource Insight permettant de suivre très précisément les problème et "Traces" remontés par notre système.

## Le fonctionnement global:

Une fois la partie NLP démontrant des performances acceptables, il faut alors créer un **ChatBot**.
Celui-ci nous servira d'interface avec l'utilisateur.
En quelque sorte il orchestrera le dialogue afin d'en tirer les 5 informations (les **Entities**) nécessaires pour répondre à la demande de l'utilisateur.
Après un message d'introduction, l'utilisateur pourra alors échanger des messages instantanés avec ce ChatBot.
Ce dernier, par l'intermédiaire du modèle entrainé sur la plateforme **LUIS**, interprétera les messages en langage naturel, et donc ce que désire l'utilisateur.
De plus il lui posera, si nécessaire, des questions pertinentes pour compléter la requête de l'utilisateur en cours.
Enfin, le ChatBot reformulera la requête de l'utilisateur et celui-ci devra, répondre par "oui" ou "non" (**étape de confirmation**) en fonction de la compréhension correcte ou non de sa demande.

## Utilisation de "BotTelemetryClient" pour tracer les "No"

La stratégie implémentée pour déterminer un mauvais fonctionnement du ChatBot, sera d'utiliser les réponses "No" des utilisateurs.
A chaque fois qu'un utilisateur cliquera sur le bouton "No" le ChatBot, dans un premier temps, récupérera les **Entities** collectées et les enverra à la ressource **Insights** de la plateforme Azure, via la :

```python
class BotTelemetryClient(ABC):
```

et une de ses méthodes :

```python
def track_trace(name, properties=None, severity: Severity=None):
```

## Ci-dessous, un exemple de dialogue:

<img src="./pictures/chatBot01 intro.png?msec=1660033374309" title="" alt="" data-align="center">

<img src="./pictures/chatBot02%20bad%20answer%20type.png" title="" alt="" data-align="center">

Ici nous pouvons observer un défaut de compréhension.
la date de retour est ambigüe donc l'utilisateur cliquera sur le bouton "No".
Le Bot Framework, nous donne accès aux "traces" de réponse de LUIS, voyons ce qu'il c'est passé:

<img src="./pictures/luis01%20acces%20Trace.png" title="" alt="" data-align="center">

<img src="./pictures/luis02%20trace%20bot%20framework.png" title="" alt="" data-align="center">

Nous pouvons donc observer que LUIS a correctement interprété un "**timex**", issu de l'entité Prebuilt "datetimeV2", mais pas d'entités de date de départ ou d'arrivée, entrainées par notre DataSet.

## Du point de vue du portail Azure:

Chaque fois que le bouton "No" est cliqué, une "trace" est envoyée à la ressource Insight.
Dans la page de la ressource, nous pouvons facilement retrouver, par divers moyens, la "trace" concernant notre problème:

<img src="./pictures/insight01%20recherche.png" title="" alt="" data-align="center">

La méthode "**track_trace(...)**", cités auparavant, permet d'ajouter à notre "trace" un tag, ici "*bad answer*". 

Ceci nous permet de la retrouver aisément et de consulter ce que le ChatBot a interprété du dialogue.

<img src="./pictures/insight02%20exemple%20bad%20answer.png" title="" alt="" data-align="center">La Trace en particulier est accessible via un lien unique que nous pouvons consulter, pour notre exemple, [ici](https://portal.azure.com/#blade/AppInsightsExtension/DetailsV2Blade/DataModel/%7B%22eventId%22:%2275f9869f-14cb-11ed-90de-002248a7607e%22,%22timestamp%22:%222022-08-05T14:30:24.602Z%22%7D/ComponentId/%7B%22Name%22:%22P10AppInsightRessource%22,%22ResourceGroup%22:%22P10RessourceGroup%22,%22SubscriptionId%22:%22d9d359df-4416-4937-86ee-51dbc2d56ca7%22%7D).

# Seuil d'alerte sur les critères d'évaluation de performance:

Afin de détecter au plus vite un problème important dans la conception du ChatBot, il est possible de lever une **Alert** qui enverra un mail à/aux destinataires paramétrés.
Nous utiliserons comme critère, un nombre de Trace supérieur à 2, dans un intervalle de 5 minutes.
Cette alerte est paramétrée comme suit: 

> Chaque fois que le nombre de traces est supérieur ou égal à 3, dans un intervalle de 5 minutes, un mail est envoyé

En voici un exemple:

<img src="./pictures/alert01%20vue%20global%20sur%20trois%20jours.png" title="" alt="" data-align="center">

Ci-dessus, une vue globale, paramétrée dans le **DashBoard** du portail Azure.

<img src="./pictures/alert02%20superieur%20a%20deux.png" title="" alt="" data-align="center">

Le 2 aout, plus de 2 Traces ont été interceptées, donc un mail fut envoyé.

<img src="./pictures/alert03%20mail%20type%20ressource%20insight%20alert.png" title="" alt="" data-align="center">

<img src="./pictures/alert04%20mail%20deux%20heures%20decallage.png" title="" alt="" data-align="center">

En voici un extrait, avec la confirmation de la date correspondante à cette alerte.

<img src="./pictures/alert05%20transformationIssuesDuneBadAnswer.png" title="" alt="" data-align="center">

## Une option intéressante?

A noter qu'en option, il est possible de créer automatiquement une "Issue" dans notre outil de gestion de version, GitHub!

<img src="./pictures/alert06%20issue%20Github.png" title="" alt="" data-align="center">

# Pour aller plus loin...

## Le ChatBot

Par la suite nous pourrions aussi étoffer le spectre des questions de notre ChatBot, afin de récupérer de l'utilisateur, des informations plus précises et mieux interprétables.
De plus ces retours permettront d'améliorer encore notre modèle LUIS.

## Le suivi de problèmes

Pour en revenir aux Issues de Github, générés via l'option "**Work Item**" de Insight:
Il me parait intéressant de l'exploiter afin de se garder la possibilité de rappeler le client ayant répondu "**No**".
Ainsi, un nouveau dialogue, étiqueté, pourra être généré et utilisé pour une mise à jour du modèle!?
