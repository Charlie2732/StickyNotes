SQL Injection – GET /Notes (filter)

Vad var problemet?
Get metoden byggde SQL frågan genom att klistra in användarens sökterm (containing) direkt i en textsträng med FromSqlRaw.
Jag kunde skriva SQL-syntax i sökfältet (`%' OR Author LIKE '%`)
och på så sätt kringgå ägarkontrollen och läsa ut alla 1000 notes från alla användare,
inte bara mina egna.

Vad kan hända om det utnyttjas?
En angripare kan läsa,ändra och radera all data i databasen, oavsett ägare,
totalt brott mot dataintegritet och sekretess.

Hur det förebyggs?
Jag bytte ut FromSqlRaw mot LINQ (.Where(...)). Då bygger EF Core frågan på ett säkert sätt istället,
så att det jag skriver i sökfältet alltid tolkas som vanlig text och aldrig som SQL kod som kan köras.

Verifiering
Samma payload (`%' OR Author LIKE '%`) gav innan fixen 1000+ notes från alla användare.
Efter fixen gav samma payload endast 3 notes, alla mina egna.


Cryptographic Failures / Sensitive Data Exposure (A04)

Vad var problemet?
Lösenordet för testanvändaren "Test" låg sparat i klartext i konfigurationsfilen appsettings.json, som committas till Git-repot.
Vem som helst som fick tillgång till repot kunde läsa lösenordet direkt utan att behöva knäcka någon kryptering.

Vad kan hända om det utnyttjas?
En angripare som hittar filen får direkt tillgång till ett giltigt konto,
utan att behöva gissa eller knäcka lösenordet.
Om samma lösenord återanvänds på andra ställen kan skadan spridas till fler system.

Hur det åtgärdades?
Jag löste det genom att flytta lösenordet till appsettings.Development.json (gitignored). Nu innehåller appsettings.json inte längre TestUserPassword i klartext.
Applikationen startades om och inloggning med Test/123 fungerade fortfarande som det ska.

Hur det förebyggs i framtida projekt?
Känsliga uppgifter som lösenord, API-nycklar och anslutningssträngar ska aldrig committas i klartext till versionshantering. De bör hållas i separata, gitignore eller Key Vault.



Software Supply Chain Failures – Impostor-paket (Swashbukle.AspNetCore)

Vad var problemet?
Jag hittade ett paket i projektet som hette Swashbukle.AspNetCore (felstavat, saknar bokstaven "c".
Det riktiga paketet heter Swashbuckle.AspNetCore.
Det falska paketet hade lagt in egen kod i en funktion som heter `.ToArray()`. Koden skrev ut "VULNERABILITY ENABLED" när jag slog på en viss inställning.
Det bevisade att paketet gjorde saker jag inte visste om.

Vad kan hända om det utnyttjas?
Om jag råkar installera fel paket kan koden i det göra vad som helst i min app, till exempel stjäla data eller skapa en bakdörr, utan att jag märker det.

Hur det förebyggs
Jag bytte ut det felstavade paketet mot rätt paket i Notes.Api.csproj, från Swashbukle.AspNetCore version 2.0.0 till Swashbuckle.AspNetCore version 6.2.3.
Sen körde jag dotnet restore och dotnet build för att hämta det riktiga paketet.

Verifiering
Innan jag fixade det skrev appen ut "VULNERABILITY ENABLED" tre gånger när den startade.
Efter fixen, med samma inställning fortfarande på, skrevs inget meddelande ut.
Det visar att det falska paketet är helt borta.


Cross-Site Scripting (XSS) – Visning av note-innehåll

Vad var problemet?
I script.js, i funktionen noteListItem, användes innerHTML för att visa innehållet i en note content.innerHTML = note.content.
Det gjorde att webbläsaren tolkade note innehållet som riktig HTML/kod istället för text.
Jag testade flera payloads (`<script>`, `<svg onload>`) som blockerades av webbläsaren, men <img src=x onerror=alert('XSS')> fungerade och gav en riktig alert ruta med texten XSS.

Vad kan hända om det utnyttjas?
En angripare kan skapa en note med skadlig kod. När en annan användare öppnar och ser den noten körs koden i deras webbläsare,
utan att de vet om det. Koden kan till exempel stjäla inloggningsuppgifter, göra saker i appen som om det vore den andra användaren.

Hur det förebyggs
Jag bytte innerHTML mot innerText i noteListItem funktionen.
innerText visar allt som ren text, aldrig som körbar kod, oavsett vad innehållet är.

Verifiering
Innan fixen körde <img src=x onerror=alert('XSS')> en alert-ruta i webbläsaren.
Efter fixen visas samma text bara som bokstäver på lappen, ingen kod körs längre.