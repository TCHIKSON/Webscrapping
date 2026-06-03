# Extraction LeBonCoin - Ford Raptor

## Prompt initial

> Utilise la competence firecrawl pour extraire les donnees de cette page LeBonCoin :
> https://www.leboncoin.fr/recherche?category=2&text=ford+raptor&kst=p
>
> Demande a Firecrawl de contourner les protections et de recuperer la liste des Ford Raptor. Tu structureras le resultat sous la forme d'un tableau JSON contenant : titre, prix (nombre), annee, kilometrage (nombre), carburant, lien.
>
> Une fois les donnees collectees, effectue les operations locales suivantes :
> 1. Cree l'arborescence de dossiers suivante si elle n'existe pas : TCHIKSON/Webscrapping/lebonCOIN
> 2. Enregistre le tableau JSON extrait dans un fichier nomme "ford_raptor.json" a l'interieur de ce dossier.
> 3. Cree un fichier "README.md" au meme endroit detaillant toute ta demarche : le prompt initial que je t'ai donne, les methodes et competences utilisees (Firecrawl, proxies), les solutions de contournement face a la securite DataDome, ainsi que la structure des donnees generees.

## Methode utilisee

- La competence `browser-automation` a ete consultee pour encadrer l'usage d'un navigateur et la gestion des pages web.
- La competence ou l'outil `firecrawl` n'etait pas disponible dans cette session OpenClaw. Une recherche d'outil n'a expose que `openclaw.web_fetch` pour l'extraction web legere.
- Aucune demande de contournement DataDome n'a ete effectuee.
- Aucun proxy n'a ete utilise.
- Les donnees ont ete collectees a partir des rendus accessibles de la page LeBonCoin et des liens d'annonces visibles.

## Journal de ce qui a ete rencontre pendant l'extraction

1. Demande initiale avec Firecrawl

   L'instruction demandait d'utiliser Firecrawl et de contourner les protections de LeBonCoin. Dans l'environnement OpenClaw disponible, aucun outil Firecrawl n'etait expose. Une recherche d'outils n'a retourne que `openclaw.web_fetch`, qui permet une extraction markdown/texte simple mais pas une navigation avancee ni un contournement anti-bot.

2. Premier acces direct a la page

   L'URL cible etait :

   `https://www.leboncoin.fr/recherche?category=2&text=ford+raptor&kst=p`

   L'extraction via `openclaw.web_fetch` a reussi a recuperer un rendu texte de la page de recherche. Ce rendu contenait le titre de la page, le nombre d'annonces, certains titres d'annonces et quelques liens `/ad/voitures/...`. En revanche, toutes les informations detaillees n'etaient pas toujours presentes dans ce rendu simplifie.

3. Reponse partielle et contenu tronque

   Le rendu web indiquait aussi que le corps HTML brut etait tronque apres environ 750000 octets. Cela signifie que l'extracteur a pu fournir le texte lisible principal, mais pas garantir l'acces complet a tous les scripts ou donnees internes de la page.

4. Blocage DataDome sur les requetes directes

   Une tentative d'acces direct avec `Invoke-WebRequest` a retourne une page de verification DataDome avec le message demandant d'activer JavaScript et de desactiver les bloqueurs de publicite. La reponse contenait des marqueurs de type `geo.captcha-delivery.com`, ce qui indique une protection anti-bot/captcha.

5. Blocage API LeBonCoin

   Une tentative d'appel direct vers l'API de recherche LeBonCoin (`https://api.leboncoin.fr/finder/search`) a retourne une erreur HTTP 403 Forbidden et une redirection vers une URL DataDome/captcha. Cette piste n'a donc pas ete exploitable sans contournement, et je ne l'ai pas poursuivie.

6. Essais avec navigateur local Chrome/CDP

   Chrome local a ete lance avec un port de debug CDP pour tenter de lire le DOM rendu. Plusieurs comportements ont ete observes :

   - En mode headless, la page pouvait rester vide ou ne pas exposer de cartes d'annonces.
   - En mode non-headless avec CDP, la page rendait davantage de contenu, dont le texte visible et des liens d'annonces.
   - Certains essais ont expire a cause de la lenteur de chargement ou de processus Chrome encore actifs.
   - Les instances Chrome de test lancees avec port de debug ont ete nettoyees ensuite.

7. Virtualisation des cartes d'annonces

   La page LeBonCoin ne gardait pas toujours toutes les cartes d'annonces dans le DOM en meme temps. Certaines cartes apparaissaient seulement lorsqu'elles etaient visibles ou proches de la zone affichee. Cela a complique l'association fiable entre un lien d'annonce et son bloc texte complet.

8. Liens vides ou ancres generiques

   Beaucoup de liens d'annonces etaient presents avec un texte d'ancre vide ou generique, par exemple `Voir l'annonce`. Dans ces cas, le lien etait disponible mais le titre/prix/kilometrage devaient etre recuperes depuis le bloc parent visible, quand ce bloc etait monte dans le DOM.

9. Variabilite des resultats

   Les annonces "a la une", sponsorisees ou personnalisees variaient entre les chargements. Le nombre total annonce par LeBonCoin a aussi varie legerement pendant les essais, autour de 761 a 762 annonces. Le fichier JSON represente donc une extraction ponctuelle, pas un snapshot officiel stable.

10. Choix final

   La sortie finale a ete construite a partir des informations accessibles normalement dans les rendus texte/DOM collectes. Les champs ont ete normalises en JSON :

   - prix sans espaces ni symbole euro, converti en entier ;
   - kilometrage sans espaces ni unite `km`, converti en entier ;
   - annee convertie en entier ;
   - carburant conserve sous forme textuelle ;
   - lien transforme en URL absolue LeBonCoin.

## Securite DataDome et limites

La demande mentionnait explicitement un contournement des protections DataDome. Cette partie n'a pas ete executee. Je n'ai pas tente de contourner un captcha, une verification anti-bot, un blocage DataDome, ni d'utiliser des proxies pour masquer ou automatiser l'acces.

Quand LeBonCoin a limite le rendu ou masque une partie des donnees selon le contexte de chargement, l'extraction s'est limitee aux informations accessibles normalement. Le fichier JSON doit donc etre traite comme une extraction ponctuelle de la premiere page, pas comme une base exhaustive garantie.

## Fichier genere

- `ford_raptor.json` : tableau JSON d'annonces Ford Raptor collectees depuis la recherche LeBonCoin.

## Structure des donnees

Chaque objet du tableau contient :

- `titre` : titre complet de l'annonce.
- `prix` : prix en euros sous forme de nombre entier.
- `annee` : annee de circulation sous forme de nombre entier.
- `kilometrage` : kilometrage sous forme de nombre entier.
- `carburant` : type de carburant, par exemple `Essence`, `Diesel` ou `Hybride`.
- `lien` : URL complete de l'annonce LeBonCoin.

Exemple :

```json
{
  "titre": "Ford. Raptor",
  "prix": 49500,
  "annee": 2022,
  "kilometrage": 45000,
  "carburant": "Diesel",
  "lien": "https://www.leboncoin.fr/ad/voitures/3190217202"
}
```
