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
