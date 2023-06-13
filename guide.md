# **Guide til GitPushers auktionsplatform på Azure!**
> Anders, Frederik, Jacob & Jacob

Her er en step-by-step guide til at benytte vores auktionsplatform og de forskellige endpoints som skal benyttes for at udføre en auktion.

Du kan følge med på vores **Grafana** dashbaord for at se, om de forskellige API'er bliver kaldt, ved at gå ind på: ``http://40.117.188.130:3000/d/a19cb934-3706-4116-985e-6064e3d830f9/auktionshus-dashboard?orgId=1&refresh=5s``

Her skal du logge ind med admin brugeren:  
``Username: admin Password: admin``

Du kan selv vælge om du vil tilgå de forskellige endpoints via Postman eller cURL scripts. I guiden benytter vi cURL scripts for hurtigere adgang og eksekvering.

> Du kan også importere vores workspace collections som indeholder alle de forskellige API'er på vores services. Filerne ligger i den tilsendte mappe.
## **1. Opret Bruger**
---
For at oprette en bruger, tilgår vi ``Users-service``. Du kan skrive følgende cURL-kommando ind i en shell terminal, for at oprette brugeren:
``` bash
curl --request POST \
  --url http://gitpushersauktionshuset.eastus.cloudapp.azure.com:4000/Users/addUser/ \
  --header 'Content-Type: application/json' \
  --data '{
	"FirstName": "Henrik",
	"LastName": "Jensen",
	"Address": "Soenderhoej 30",
	"Phone": "12341234",
	"Email": "henrikjensen@gmail.com",
	"Password": "password",
	"Verified": true,
	"Rating": 9.0,
	"Username": "henrik"
}'
```
> Husk at gemme dit **userID** eller find det i databasen.

Vi gemmer **userID** i en bash-variabel:
``` console
$ userId="<indsæt userID her>"
```

Test om variablen er gemt ved at køre kommandoen:
``` console
$ echo $userId
```

## **2. Login**
---
For at login, skal du bruge dit ``Username`` og ``Password`` til at tilgå login endpointen, som ligger i ``Auth-service``.
Kør følgende kommando i terminalen:
``` bash
curl --request POST \
  --url http://localhost:4000/AuthService/login \
  --header 'Content-Type: application/json' \
  --data ' { 
    "Username": "henrik",
    "Password": "password"
}'
```
Efter login, får du en JWT-token returneret. Din token ser nogenlunde ud som denne:
``` json
{ 
"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6ImhlbnJpayIsImV4cCI6MTY4NTQ0MTcyMSwiaXNzIjoiSkFEQURBRERBQURBIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdCJ9.PKotjwX3xb6NOilLOWtqObw6hd7WJsmYFuuIY9NyrZo"
}
```
Denne token skal gemmes(kun strengen, uden " "), da den bruges til at tilgå endpoints som er beskyttet og som kræver **Authorization** for at tilgå dem.

Vi gemmer også **token** i en bash-variabel:
``` console
$ token="<indsæt token her>"
```

Du kan teste om din token virker, ved at køre følgende kommando i terminalen:
``` bash
curl --request GET \
  --url http://localhost:4000/Users/getAllUsers \
  --header "Authorization: Bearer $token"
```

## **3. Opret Article**
--- 
Eftersom vi allerede har en masse seed data oprettet i databasen, skal du benytte vores auktionshus som vi har oprettet på forhånd. Vi gemmer derfor ``auctionhouseId`` i en bash variabel:

``` bash
$ auctionhouseId="64884f829f0181f169457039"
```

Næste trin er at oprette en effekt, som skal lægges op til auktion. Vi kalder ``addArticle`` endpointet i ``Article-service`` med følgende cURL kommando:
``` bash
curl --request POST \
  --url http://localhost:4000/ArticleService/addArticle \
  --header "Authorization: Bearer $token" \
  --header 'Content-Type: application/json' \
  --data '{
  "Name": "Spisestol",
  "NoReserve": true,
  "EstimatedPrice": 4556,
  "Description": "Laekker kvalitet, aegte gedeskind",
  "Category": "CH",
  "Sold": false,
  "AuctionhouseID": "'"$auctionhouseId"'",
  "SellerID": "'"$userId"'",
  "MinPrice": 2400,
  "BuyerID": ""
}'
```
> Husk at gemme den returnerede **articleID** som bliver returneret i terminalen eller find det i databasen.

Gemmer **articleID** i en bash variabel:
``` console
$ articleId="<indsæt articleId her>"
```

Vi kan også vedhæfte et billede til vores nye effekt ved at køre denne kommando:
``` bash
curl --location --request PUT "http://localhost:4000/ArticleService/addArticleImage/$articleId" \
--header "Authorization: Bearer $token" \
--form "=@$(pwd)/spisestol.png"
```
> For at ovenstående kommando virker med ``$pwd``, skal man stå i mappen hvor billedet er gemt.

## **4. Opret Auction**
---
Næste skridt er, at oprette en auktion med den nyoprettede effekt. Vi kalder ``addAuction`` endpointet i ``AuctionPlanning-service`` med følgende cURL kommando:
``` bash
curl --location 'http://localhost:4000/AuctionPlanning/addAuction' \
--header 'Content-Type: application/json' \
--header "Authorization: Bearer $token" \
--data '{
  "StartDate": "2023-06-13T12:00:00",
  "EndDate": "2023-06-24T14:00:00",
  "ArticleID": "'"$articleId"'"
}'
```
> Husk at gemme den returnerede **auctionID** som bliver returneret i terminalen eller find det i databasen.

Vi gemmer også **auctionID** i en bash variabel:
``` console
$ auctionId="<indsæt auctionID her>"
```

## **5. Add Bid**
---
Til sidst skal vi tilføje et bud til vores nye auktion. Det gør vi ved at kalde addBid endpointet i Auction-service med følgende cUrl kommando:
``` bash
curl --location --request PUT 'http://localhost:4000/AuctionService/addBid' \
--header 'Content-Type: application/json' \
--header "Authorization: Bearer $token" \
--data '{
    "Price": 2400,
    "BidderID": "'"$userId"'",
    "AuctionID": "'"$auctionId"'"
}'
```

Vi har nu været igennem hele flowet for at afgive et bud på en auktion.