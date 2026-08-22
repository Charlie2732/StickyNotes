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

