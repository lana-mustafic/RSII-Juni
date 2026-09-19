# RSII – Detaljni vodič za ispit 24.06.2026.

> **Cilj ovog dokumenta:** Da razumiješ *logiku* rješavanja RSII ispita, a ne da kopiraš gotov kod.
> **Pravilo:** Prvo čitaj, razumij, zatim sama piši. Koristi postojeće primjere u templateu kao „udžbenik", ne kao rješenje.
> **Ovaj vodič NAMJERNO ne sadrži gotovo rješenje ispita.** Daje ti način razmišljanja, mapu projekta i redoslijed rada.

---

## Sadržaj

1. [Kako koristiti ovaj vodič](#1-kako-koristiti-ovaj-vodič)
2. [Analiza ispitnog zadatka](#2-analiza-ispitnog-zadatka)
3. [Kako čitati RSII zadatak](#3-kako-čitati-rsii-zadatak)
4. [Arhitektura projekta](#4-arhitektura-projekta)
5. [Entity – od teksta zadatka do klase](#5-entity--od-teksta-zadatka-do-klase)
6. [Enum](#6-enum)
7. [EF Core konfiguracija](#7-ef-core-konfiguracija)
8. [Relacije između Entityja](#8-relacije-između-entityja)
9. [DbContext](#9-dbcontext)
10. [Database migration](#10-database-migration)
11. [Servisi umjesto CQRS](#11-servisi-umjesto-cqrs)
12. [Insert / Create](#12-insert--create)
13. [Update](#13-update)
14. [Delete](#14-delete)
15. [GetAll / List / Search / Filter](#15-getall--list--search--filter)
16. [GetById](#16-getbyid)
17. [DTO – Request i Response](#17-dto--request-i-response)
18. [Validation](#18-validation)
19. [API Endpointi](#19-api-endpointi)
20. [Dependency Injection](#20-dependency-injection)
21. [Frontend – Flutter mobile](#21-frontend--flutter-mobile)
22. [Kako povezati sve dijelove](#22-kako-povezati-sve-dijelove)
23. [Redoslijed rada na ispitu](#23-redoslijed-rada-na-ispitu)
24. [Kako samostalno riješiti ovaj zadatak](#24-kako-samostalno-riješiti-ovaj-zadatak)
25. [Checklist prije predaje](#25-checklist-prije-predaje)
26. [Najčešće greške](#26-najčešće-greške)
27. [Šta naučiti napamet, a šta razumjeti](#27-šta-naučiti-napamet-a-šta-razumjeti)
28. [Mini primjeri za vježbu](#28-mini-primjeri-za-vježbu)
29. [Cheat Sheet](#29-cheat-sheet)
30. [Rječnik pojmova](#30-rječnik-pojmova)
31. [KODOVI koje pišeš na ispitu](#31-kodovi-koje-pišeš-na-ispitu)

---

# 1. Kako koristiti ovaj vodič

## 1.1. Kome je vodič namijenjen

Ovaj vodič je namijenjen tebi kao studentici predmeta **Razvoj softvera II**.

Pretpostavke vodiča:

* znaš osnovni C# (klase, propertyji, metode);
* znaš šta je REST API na nivou „frontend šalje HTTP zahtjev, backend odgovara";
* **ne moraš** znati CQRS, Clean Architecture, Domain/Application/Infrastructure slojeve iz RS1;
* **ne moraš** znati EF Core napamet;
* **ne moraš** znati Flutter napamet – template već ima obrasce koje kopiraš po principu.

Ako si radila RS1, ovo je **drugačiji template**. Nemoj tražiti `Market.Domain`, `Commands`, `Queries` i `Handlere`. Njih ovdje **nema**.

## 1.2. Šta treba naučiti

Na ovom ispitu ne učiš novi programski jezik. Učiš **obrazac rada na gotovom projektu**:

1. pročitati tekst zadatka;
2. prepoznati šta je nova tabela / entitet;
3. prepoznati šta je relacija;
4. prepoznati šta je validacija;
5. prepoznati šta je API;
6. prepoznati šta je Flutter ekran;
7. uraditi to **istim stilom kojim je template već urađen**.

Najvažnija vještina nije pisanje koda iz glave. Najvažnija vještina je:

> „Ovo već postoji za Category / ProductReview / Order. Ja trebam uraditi istu stvar za novi pojam iz zadatka."

## 1.3. Kako koristiti vodič prije ispita

Radni plan učenja (nije jedan sjedim i čitam 80 strana):

1. **Dan 1 – mapa projekta.** Prođi sekciju 4. Otvori Visual Studio i stvarno klikni kroz foldere. Cilj: da znaš gdje se šta nalazi bez vodiča.
2. **Dan 2 – jedan postojeći CRUD.** Uzmi `Category` i prati ga kroz sve slojeve: Entity → Request/Response → Search → Validator → Service → Controller → Flutter (desktop kategorije). Cilj: da vidiš cijeli lanac.
3. **Dan 3 – posebni obrasci.** Pogledaj `OrderService.CheckoutAsync` i `ProductReviewService`. To su obrasci za „ulogovani korisnik" i „poslovno pravilo".
4. **Dan 4 – ovaj ispit.** Čitaj sekciju 2 i 24. Na papiru izvuci entitete, relacije, validacije, Flutter ekrane. **Ne piši još kod.**
5. **Dan 5 – vježba.** Uradi jedan mini zadatak iz sekcije 28 na kopiji projekta. Zatim pokušaj ovaj ispit sama, koristeći vodič samo kad zapneš.

## 1.4. Kako koristiti vodič tokom vježbanja

Kad zapneš, ne skrolaj nasumično. Pitaj se:

1. Koji **sloj** mi fali? (baza, servis, API, Flutter)
2. Koji **postojeći primjer** u templateu radi sličnu stvar?
3. Koja **ključna riječ** iz zadatka mi govori šta treba?

Tek onda otvori odgovarajuću sekciju vodiča.

## 1.5. Šta ne treba učiti napamet

Nemoj učiti napamet:

* cijele klase;
* cijeli `Program.cs`;
* Flutter widget stablo;
* SQL koji EF Core generiše;
* tačan broj linija u `BaseCRUDService`.

To se na ispitu **gleda iz templatea**. Template je tvoj udžbenik.

## 1.6. Šta treba razumjeti

Moraš razumjeti:

* zašto Entity nije isto što i Request;
* zašto Controller ne smije praviti `new ECommerceDbContext()`;
* zašto se stanje kartice **računa**, a ne čuva;
* zašto Insert ide kroz servis, a ne kroz kontroler direktno u bazu;
* kako iz rečenice „jedan korisnik može imati više kartica" nastane relacija 1:N;
* kako iz rečenice „broj kartice mora sadržavati 12 cifara" nastane FluentValidation pravilo.

## 1.7. Važno upozorenje o templateu

Zadatak kaže da preuzmeš template sa FTP-a:

* folder: `Download/RSII`
* user/pass: `student_250250`

Zvanični GitHub template za 2025/26 je:

`https://github.com/Adil-Eminagic/rsII_exam_template_2025_26`

**Ovaj vodič je pisan prema tom 2025/26 templateu**, jer je to ispitni template.

Ako lokalno imaš stariji folder tipa `rsII_exam_template_2024_25`, **ne uči na njemu**. Stariji template ima sličnu ideju, ali nema sve što 2025/26 ima (JWT, `ClinetException`, `OrderService.CheckoutAsync`, `IAuthenticatedUserAccessor`, validatore, Flutter profil/korpa).

Na ispitu radiš na templateu koji dobiješ sa FTP-a. Prije pisanja koda **provjeri** da vidiš:

* `eCommerce.Services/Database/`
* `eCommerce.Model/Requests/`
* `eCommerce.WebAPI/Controllers/BaseCRUDController.cs`
* `eCommerce/UI/ecommerce_mobile/lib/screens/profile_screen.dart`

Ako to postoji, u pravom si projektu.

## 1.8. Važno upozorenje o RS1 navikama

Ako u glavi imaš RS1, ovo zapamti odmah:

| RS1 pojam | Postoji li u RSII templateu? | Šta koristiš umjesto toga |
|-----------|------------------------------|---------------------------|
| `Market.Domain` | **Ne** | `eCommerce.Services/Database/` |
| `Market.Application` Commands/Queries | **Ne** | `eCommerce.Services/*Service.cs` |
| Handler | **Ne** | metoda u servisu (`InsertAsync`, `GetAllAsync`, `CheckoutAsync`) |
| `Market.Infrastructure` | **Ne** | isti `eCommerce.Services` projekat (tu su i Entity i DbContext) |
| FluentValidation Validator uz Command | Da, ali uz **Request** | `eCommerce.Services/Validators/` |
| Angular | **Ne** | Flutter (`ecommerce_mobile` i `ecommerce_desktop`) |
| CQRS | **Ne** | klasični servis + kontroler |

Ako kreneš praviti `Commands` folder, radiš pogrešan ispit.

---

# 2. Analiza ispitnog zadatka

Ispit: **INTEGRALNI ispit iz predmeta Razvoj softvera II – 24.06.2026.**

Ispit ima 2 stranice. Logički se dijeli na:

1. pripremu okruženja;
2. backend za platne kartice;
3. Flutter prikaz/dodavanje kartica u profilu;
4. plaćanje narudžbe karticom i računanje stanja;
5. proširenje `Order` entiteta;
6. edit kartice + lista transakcija;
7. pakovanje i predaja.

Ispod je **svaki** zahtjev rastavljen. Ne rješavam ga. Učim te kako da ga sama rastaviš.

---

## 2.0. Priprema okruženja (prije koda)

### Šta zadatak kaže

Sa FTP-a preuzmi template. Promijeni konekcijski string u `appsettings.Development.json`.

* Server: `192.168.0.1\Exams`
* Port: `1999`
* Baza: tvoj broj indeksa, npr. `IB150051`
* SQL login: `john` / `doe2025`

Zatim u Package Manager Console, odaberi **Service** projekat, pa:

```
Update-Database
```

Nakon migracije u aplikaciji postoje:

* username = `customer1`
* password = `Test123`

### Šta to zapravo znači

SQL server na ispitu je **zajednički**. Svaki student mora imati **svoju bazu**. Ako ostaviš `Database=eCommerce`, možeš pokvariti tuđu bazu ili raditi po tuđim podacima.

`john` / `doe2025` je login za **SQL Server**, ne za Flutter aplikaciju.

`customer1` / `Test123` je login za **eCommerce aplikaciju** (seed podaci).

### Šta ja trebam napraviti

1. Otvori `eCommerce/eCommerce.WebAPI/appsettings.Development.json`.
2. Nađi `ConnectionStrings:DefaultConnection`.
3. Promijeni `Server`, `Database`, `User Id`, `Password`.
4. Pokreni `Update-Database` na `eCommerce.Services`.
5. U SSMS-u provjeri da baza tvog indeksa postoji i da ima tabele.
6. Pokreni API i Flutter, uloguj se kao `customer1`.

### U kojem dijelu projekta se to radi

* konekcija: `eCommerce.WebAPI/appsettings.Development.json` (i po potrebi `appsettings.json`)
* migracije: `eCommerce.Services/Migrations/`
* seed korisnici: `eCommerce.Services/Database/eCommerceSeed.cs`

### Koji su koraci

1. Visual Studio → otvori `eCommerce.sln`.
2. Desni klik na solution → Set Startup Project → `eCommerce.WebAPI`.
3. Tools → NuGet Package Manager → Package Manager Console.
4. U PMC dropdownu **Default project** stavi `eCommerce.Services`.
5. Upisi `Update-Database`.
6. Ako ne uspije, prvo provjeri connection string, pa da li SQL server prima tvoj login.

Primjer formata connection stringa (stavi **svoj** indeks, ne ovaj):

```
Server=192.168.0.1\\Exams,1999;Database=TVOJ_INDEKS;User Id=john;Password=doe2025;TrustServerCertificate=True;
```

U C# stringu ili JSON-u backslash često treba duplirati: `\\Exams`.

### Šta trebam provjeriti na kraju

* [ ] Baza se zove moj indeks, ne `eCommerce`
* [ ] `Update-Database` je prošao bez errora
* [ ] U bazi vidim tabele `Users`, `Orders`, `Products`, ...
* [ ] API se pokreće na `http://localhost:5126` (vidi `launchSettings.json`)
* [ ] Flutter login sa `customer1` / `Test123` radi

### Tipična zamka

Zadatak kaže da svi rade na istoj instanci servera. Ako zaboraviš promijeniti ime baze, to je ozbiljna greška. Uradi ovo **prije** bilo kakvog koda.

---

## 2.1. Pravilo arhitekture (crveni okvir zadatka)

### Šta zadatak kaže

Obavezno poštovati postojeću strukturu i principe projekta. To podrazumijeva dodavanje potrebnih (dto)entiteta, servisa i kontrolera u odgovarajuće slojeve. **Nikako** instanciranje `DbContext` i drugih objekata npr. na nivou kontrolera.

### Šta to zapravo znači

Profesor ne želi da u `PaymentCardsController` napišeš:

```csharp
var db = new ECommerceDbContext(...);
db.PaymentCards.Add(...);
```

To kvari cijeli projekat jer:

* preskačeš validaciju;
* preskačeš mapiranje;
* preskačeš filtere;
* kontroler prestaje biti tanak sloj.

### Šta ja trebam napraviti

Isti lanac kao za `Category`:

Entity → DbSet → (konfiguracija) → migracija → Request/Response/Search → Validator → Interface + Service → registracija u `Program.cs` → Controller koji nasljeđuje `BaseCRUDController`.

### U kojem dijelu projekta se to radi

Svaki sloj ima svoje mjesto. Pogledaj sekciju 4.

### Koji su koraci

Nemoj početi od Fluttera. Nemoj početi od kontrolera. Počni od podataka, jer sve ostalo visi o tabeli.

### Šta trebam provjeriti na kraju

* [ ] U kontroleru nema `new ECommerceDbContext`
* [ ] Servis dobija `ECommerceDbContext` kroz konstruktor
* [ ] Novi servis je registrovan u `Program.cs`

---

## 2.2. Zahtjev 2 – Entitet platne kartice

### Šta zadatak kaže

Na Web API dijelu dodati entitet `PaymentCardBrojIndeksa` koji služi za čuvanje podataka o platnim/kreditnim karticama korisnika.

Jedan korisnik može imati više kartica.

Kartica treba sadržavati:

* vezu na korisnika;
* broj kartice;
* CVC;
* datum isteka;
* početno stanje kartice;
* druge atribute koje smatraš potrebnim.

Pravila:

* broj kartice mora imati **12 cifara**;
* CVC mora imati **3 cifre**;
* dozvoljen je unos **samo cifara**, bez razmaka i posebnih znakova;
* datum isteka mora biti validan i **ne smije biti već istekao**;
* početno stanje mora biti **decimalna vrijednost >= 0**.

Potrebno je dodati svu infrastrukturu: model, baznu tabelu, migraciju, DTO/request/response modele, servis, kontroler i validacije.

### Šta to zapravo znači

Ovo je **puni CRUD stub** za novi entitet, plus stroga validacija.

Ime entiteta nije `PaymentCard`. Ime je `PaymentCard` + **tvoj broj indeksa**. Ako ti je indeks `IB210001`, entitet se zove npr. `PaymentCardIB210001`.

„Druge atribute koje smatraš potrebnim" nije dozvola da izmisliš 15 polja. To znači: pogledaj kako izgledaju postojeći entiteti (`Category`, `Product`, `ProductReview`) i dodaj ono što ovaj projekat **obično** ima, npr.:

* `Id`
* `CreatedAt`
* `UpdatedAt`
* možda `IsActive`

Ne dodaj polje „trenutno stanje", jer zahtjev 4 izričito kaže da se to **ne čuva**, nego se **računa**.

### Kako da prepoznam šta se od mene traži

| Fraza u zadatku | Šta to znači u templateu |
|-----------------|--------------------------|
| „dodati entitet" | nova klasa u `eCommerce.Services/Database/` |
| „veza na korisnika" | `UserId` + navigation `User` |
| „jedan korisnik više kartica" | relacija 1:N |
| „12 cifara / 3 cifre / samo cifre" | FluentValidation + možda MaxLength na entitetu |
| „datum isteka ne smije biti istekao" | validacija na Insert i na plaćanje |
| „decimalna vrijednost >= 0" | `decimal` + `GreaterThanOrEqualTo(0)` |
| „model, tabela, migracija, DTO, servis, kontroler, validacije" | kompletan lanac kao `Category` |

### Šta ja trebam napraviti

Ne pišem ti gotovu klasu. Pišem ti **spisak odluka** koje moraš sama donijeti:

1. Kako ću nazvati klasu? Mora sadržavati broj indeksa.
2. Koja polja su obavezna iz teksta?
3. Koja polja su „potrebna jer tako radi template"?
4. Koje polje **ne smijem** dodati kao kolonu? (trenutno stanje)
5. Kako ću spremiti datum isteka? `DateTime`? `int` mjesec+godina? `string`? Odaberi jedan način i budi dosljedna u Request, Entity i Flutter formi.
6. Da li CVC treba ići u Response? Zadatak ne kaže. Razmisli: za prikaz liste možda ne treba puni CVC. Za edit formu možda treba. Na ispitu je važnije da CRUD radi nego da bude bankarski sigurno, ali nemoj slati CVC ako nije potreban.

### U kojem dijelu projekta se to radi

* Entity: `eCommerce.Services/Database/PaymentCardTVOJINDEKS.cs`
* Navigation na User: `eCommerce.Services/Database/User.cs`
* Relacija/delete behavior: `eCommerce.Services/Database/eCommerceConfiguration.cs`
* DbSet: `eCommerce.Services/Database/eCommerceDbContext.cs`
* Migracija: PMC na `eCommerce.Services`
* Request/Response/Search: `eCommerce.Model/...`
* Validator: `eCommerce.Services/Validators/`
* Service + interface: `eCommerce.Services/`
* Controller: `eCommerce.WebAPI/Controllers/`
* DI: `eCommerce.WebAPI/Program.cs`

### Koji su koraci

1. Napravi Entity.
2. Dodaj `ICollection<...>` na `User` ako treba navigation sa druge strane.
3. Dodaj `DbSet`.
4. Konfiguriši relaciju ako default nije dovoljan.
5. `Add-Migration` pa `Update-Database`.
6. Request, UpdateRequest, Response, SearchObject.
7. InsertValidator i UpdateValidator.
8. `I...Service` + `...Service` koji nasljeđuje `BaseCRUDService`.
9. Override `ApplyFilters` da možeš filtrirati po korisniku.
10. Registruj servis i validatore u `Program.cs`.
11. Napravi Controller koji nasljeđuje `BaseCRUDController`.
12. Testiraj u Swaggeru / Scalar-u **prije** Fluttera.

### Šta trebam provjeriti na kraju

* [ ] Tabela postoji u mojoj bazi
* [ ] Ime entiteta sadrži broj indeksa
* [ ] `UserId` je foreign key
* [ ] Insert bez 12 cifara vraća 400
* [ ] Insert sa negativnim početnim stanjem vraća 400
* [ ] Insert sa isteklom karticom vraća 400
* [ ] GetAll vraća `PageResult` (items + totalCount), ne golim niz

---

## 2.3. Zahtjev 3 – Flutter profil: lista i dodavanje kartica

### Šta zadatak kaže

U Flutter dijelu, u okviru **profila korisnika**, omogućiti prikaz, dodavanje i pregled detalja platnih kartica.

Na formi profila dodati sekciju **„Moje karice"** (u PDF-u je tipfeler; misli se na „Moje kartice") u kojoj su sve kartice **trenutno prijavljenog** korisnika. Za svaku karticu prikazati osnovne podatke.

Dodati dugme **„Dodaj karticu"** koje otvara novu formu (`payment_card_details.dart`). Forma omogućava unos:

* broja kartice;
* CVC-a;
* datuma isteka;
* početnog stanja.

Prilikom unosa uraditi validaciju. Nakon uspješnog dodavanja vratiti korisnika na profil i **ponovo učitati** listu kartica.

### Šta to zapravo znači

Ovo NIJE desktop admin dio.

Desktop (`ecommerce_desktop`) je admin: lista proizvoda, korisnika, kategorija.

Mobile (`ecommerce_mobile`) ima tab **Profile** (`ProfileScreen`). Zadatak kaže „profil korisnika" i „prijavljeni korisnik". To je **mobile**.

Fajl `payment_card_details.dart` je **imenovan u zadatku**. Moraš ga napraviti. Nemoj ga nazvati `card_form.dart` ako zadatak traži tačno to ime – profesor može tražiti fajl po imenu.

### Kako da prepoznam šta se od mene traži

| Fraza | Šta napraviti |
|-------|----------------|
| „u okviru profila" | mijenjaj `profile_screen.dart` |
| „sekcija Moje kartice" | novi widget/dio na profilu, ne novi tab u bottom bar-u osim ako baš želiš; zadatak kaže na formi profila |
| „sve kartice trenutno prijavljenog korisnika" | GET sa filterom po userId **ili** backend sam uzima userId iz JWT-a |
| „osnovni podaci" | npr. maskirani broj, datum isteka, početno stanje; ne moraš prikazati CVC na listi |
| „Dodaj karticu" | `Navigator.push` na `payment_card_details.dart` |
| „validacija prilikom unosa" | Flutter validacija **i** backend validacija |
| „vratiti na profil i ponovo učitati" | `Navigator.pop` + ponovo pozvati load metodu, isti obrazac kao Edit profile (`refresh == 'reload'`) |

### Šta ja trebam napraviti

Flutter lanac, po uzoru na postojeće:

1. model u `lib/models/` (json_serializable);
2. `build_runner` da se napravi `.g.dart`;
3. provider u `lib/providers/` koji nasljeđuje `BaseProvider`;
4. registracija providera u `main.dart` unutar `MultiProvider`;
5. sekcija na `ProfileScreen`;
6. ekran `payment_card_details.dart`.

Pogledaj kako `ProfileScreen` već radi reload nakon `ProfileSettingsScreen`:

* `var refresh = await Navigator.push(...)`
* `if (refresh == 'reload') { initData(); }`

To je obrazac koji trebaš ponoviti za kartice.

### U kojem dijelu projekta se to radi

`eCommerce/UI/ecommerce_mobile/`

Konkretno:

* `lib/screens/profile_screen.dart`
* `lib/screens/payment_card_details.dart` (novi fajl, ime iz zadatka)
* `lib/models/`
* `lib/providers/`
* `lib/main.dart`

### Koji su koraci

1. Backend mora raditi u Swaggeru. Ako backend ne radi, Flutter neće raditi.
2. Napravi Dart model.
3. Generiši `.g.dart`.
4. Napravi provider. `endpoint` mora odgovarati imenu kontrolera. Ako je kontroler `PaymentCardsIB210001Controller`, ruta je `/PaymentCardsIB210001`.
5. Registruj provider.
6. Na profilu učitaj listu.
7. Napravi formu za insert.
8. Nakon inserta pop + reload.

### Šta trebam provjeriti na kraju

* [ ] Na profilu vidim sekciju kartica
* [ ] Kad nemam kartica, ekran ne puca (prazna lista)
* [ ] Dodaj karticu otvara `payment_card_details.dart`
* [ ] Loš unos (11 cifara, slova, negativan iznos) se ne šalje ili backend vrati grešku koju vidim
* [ ] Nakon uspjeha sam opet na profilu i nova kartica se vidi bez restarta aplikacije

---

## 2.4. Zahtjev 4 – Plaćanje narudžbe karticom i računanje stanja

Ovo je **najvažniji i najlakše pogrešno shvaćen** dio ispita.

### Šta zadatak kaže

Kartice iz profila trebaju se pojaviti kao **opcija prilikom plaćanja narudžbe/računa**.

Korisnik bira **jednu** karticu.

Plaćanje se može izvršiti samo ako:

1. odabrana kartica **nije istekla**;
2. ima **dovoljno dostupnog novca** za ukupan iznos računa.

Trenutno dostupno stanje **ne čuva se kao vrijednost**, nego se **računa**:

> početno stanje minus suma svih uspješno realizovanih transakcija tom karticom.

Kartica **ne smije otići u minus**. Ako nema dovoljno sredstava, prikaži jasnu poruku i spriječi plaćanje.

Primjer iz zadatka:

* početno stanje = 200 KM
* ranija plaćanja = 50 + 30
* dostupno = 120
* novi račun = 130 → **ne dozvoliti**

### Šta to zapravo znači

Tri odvojene stvari:

1. **UI za odabir kartice** na checkout-u.
2. **Backend pravilo** koje može odbiti plaćanje.
3. **Formula stanja** koja koristi narudžbe, ne novo polje u tabeli kartica.

U templateu checkout već postoji:

* backend: `OrderService.CheckoutAsync`
* API: `POST /Orders/Checkout`
* Flutter: `CartListScreen._checkout()` → `OrderProvider.checkout(...)`

Trenutno checkout **ne pita** za karticu. Samo šalje `items`.

Znači: ne praviš novi „Payment" modul iz nule. **Proširuješ postojeći checkout.**

### Kako da prepoznam šta se od mene traži

| Fraza | Šta NE raditi | Šta raditi |
|-------|----------------|------------|
| „stanje se ne čuva" | dodati kolonu `CurrentBalance` i oduzimati je | računati svaki put |
| „transakcije tom karticom" | nužno praviti tabelu `Transactions` | koristiti narudžbe vezane za tu karticu (zahtjev 5) |
| „spriječiti izvršenje" | samo Flutter if | **obavezno backend** (`ClinetException`) |
| „jasna poruka korisniku" | 500 Internal Server Error | 400 + poruka koju Flutter već zna prikazati (`ApiClientException`) |

Zašto backend mora? Jer Flutter validaciju mogu zaobići. Profesor testira API.

Zašto Flutter i dalje treba poruku? Jer zadatak kaže da korisnik vidi zašto ne može platiti.

### Šta ja trebam napraviti

Odluke koje moraš sama donijeti:

1. Gdje korisnik bira karticu? Najlogičnije: korpa, prije `Place order`, jer tamo već postoji checkout.
2. Kako checkout request dobija id kartice? Proširi `CheckoutRequest`.
3. Kako znam da je transakcija „uspješno realizovana"? Moraš definirati: npr. narudžba koja je stvarno kreirana i nije cancelled. Template trenutno stavlja `Status = Processing` i ne puni `PaymentDate`. Ako želiš koristiti `PaymentDate`, postavi ga kad plaćanje prođe.
4. Formula: `available = initial - sum(successful orders for this card)`.
5. Ako `available < totalAmount` → baci `ClinetException` sa jasnom porukom.
6. Ako je kartica istekla → isto, `ClinetException`.
7. Ako kartica nije od tog korisnika → nemoj dozvoliti plaćanje tuđom karticom.

### U kojem dijelu projekta se to radi

Backend:

* `eCommerce.Model/Requests/CheckoutRequest.cs`
* `eCommerce.Services/OrderService.cs` metoda `CheckoutAsync`
* možda kartica Response da Flutter može prikazati dostupno stanje (ali formula mora živjeti na backendu)

Flutter:

* `lib/screens/cart_list_screen.dart`
* `lib/providers/order_provider.dart` (body checkout-a)
* provider kartica da učitaš listu za dropdown

### Koji su koraci

1. Prvo zahtjev 5 (veza Order → kartica), jer bez toga nemaš „transakcije".
2. Proširi checkout request.
3. U `CheckoutAsync`, **prije** `SaveChanges`, izračunaj dostupno stanje i odluči.
4. Tek onda snimi narudžbu sa id-em kartice.
5. Na Flutter korpi dodaj odabir kartice.
6. Testiraj primjer iz zadatka brojkama.

### Šta trebam provjeriti na kraju

* [ ] Ne mogu platiti bez odabrane kartice
* [ ] Ne mogu platiti isteklom karticom
* [ ] Ne mogu platiti ako 200 - 50 - 30 = 120, a račun je 130
* [ ] Mogu platiti ako račun je 120 ili manje
* [ ] Kartica u bazi i dalje ima **isto početno stanje** nakon plaćanja (nije se smanjilo kao kolona)
* [ ] Poruka je čitljiva, nije stack trace

### Mini checklist razmišljanja za formulu

Na papiru, prije koda, napiši:

```
dostupno = početnoStanjeKartice
           - zbir(iznosNarudžbe)
             gdje je narudžba plaćena TOM karticom
             i smatra se uspješnom
```

Ako ne znaš šta je „uspješna", definiraj to konzistentno. Najjednostavnije na ovom ispitu: svaka narudžba koja je prošla `CheckoutAsync` i vezana je za tu karticu, osim ako je status `Cancelled`.

---

## 2.5. Zahtjev 5 – Proširiti Orders

### Šta zadatak kaže

Na adekvatan način proširiti entitet `Orders` kako bi osigurao čuvanje informacije o izvršenom plaćanju u kontekstu korištene kartice.

### Šta to zapravo znači

`Order` već postoji. Ne praviš novi entitet za narudžbu.

`Order` već ima:

* `PaymentTransactionId` (string, nullable)
* `PaymentDate` (DateTime?, nullable)

To **nije dovoljno**, jer zadatak traži informaciju **koja kartica** je korištena. String transaction id ne vezuje red na tvoju novu tabelu kartica.

„Adekvatan način" u ovom templateu znači:

* dodaj foreign key na karticu;
* dodaj navigation property;
* konfiguriši relaciju;
* nova migracija;
* u `CheckoutAsync` popuni taj FK;
* po potrebi proširi `OrderResponse` ako Flutter treba prikazati karticu.

### Kako da prepoznam šta se od mene traži

Ključna riječ: **proširiti postojeći entitet**.

To nije novi CRUD. To je:

1. novo polje / FK na starom entityju;
2. migracija;
3. mjesto u kodu koje to polje puni (checkout).

### Šta ja trebam napraviti

Pitaj se:

* Da li je kartica obavezna na svakoj staroj narudžbi? Stare narudžbe u seedu nemaju karticu. FK treba biti **nullable**, inače migracija može pasti ili stari podaci postanu neispravni.
* Za nove checkout-e kartica jeste obavezna po zadatku.
* Delete behavior: ako obrišeš karticu, šta sa narudžbama? `Restrict` je često sigurniji nego `Cascade` (ne želiš da brisanje kartice obriše historiju narudžbi). Pogledaj kako je `ProductReview.Order` urađen: `OnDelete(DeleteBehavior.Restrict)`.

### U kojem dijelu projekta se to radi

* `eCommerce.Services/Database/Order.cs`
* `eCommerce.Services/Database/eCommerceConfiguration.cs`
* `eCommerce.Services/OrderService.cs`
* `eCommerce.Model/Responses/OrderResponse.cs` (ako treba)
* nova migracija

### Koji su koraci

1. Dodaj FK na `Order`.
2. Konfiguriši relaciju.
3. Migracija.
4. U checkoutu postavi FK kad je plaćanje prihvaćeno.
5. Razmisli da li da popuniš i postojeće `PaymentDate`.

### Šta trebam provjeriti na kraju

* [ ] Stare narudžbe i dalje postoje
* [ ] Nova narudžba u bazi ima id kartice
* [ ] U SSMS-u vidim FK kolonu
* [ ] Brisanje kartice ne briše cijelu historiju narudžbi (osim ako si svjesno stavila Cascade – obično ne želiš)

---

## 2.6. Zahtjev 6 – Edit kartice i lista transakcija

### Šta zadatak kaže

Na formi profila, odabirom neke od kartica omogućiti **editovanje** njenih podataka (`payment_card_details.dart`).

Ispod podataka o kartici prikazati **osnovne informacije o transakcijama** obavljenim odabranom karticom (**datum i iznos**).

### Šta to zapravo znači

Isti ekran `payment_card_details.dart` radi **dva moda**:

* create (nema postojeće kartice);
* edit (stigla je kartica / id kartice).

To je isti obrazac kao desktop `CategoryDetailsScreen`:

* `final Category? category;`
* ako je `null` → create;
* ako nije → edit.

Transakcije nisu treći CRUD. Transakcije su **narudžbe plaćene tom karticom**. Prikazuješ datum i iznos. Ne moraš praviti novu tabelu `Transaction` ako već imaš Order + FK kartice.

### Kako da prepoznam šta se od mene traži

| Fraza | Šta napraviti |
|-------|----------------|
| „odabirom kartice" | `onTap` na stavku liste |
| „editovanje" | `PUT /PaymentCards.../{id}` preko `provider.update` |
| „isti fajl payment_card_details.dart" | jedan screen, dva moda |
| „ispod podataka... transakcije" | lista samo u edit modu (nova kartica još nema transakcija) |
| „datum i iznos" | npr. `orderDate` + `totalAmount` |

### Šta ja trebam napraviti

Odluke:

1. Da li edit smije mijenjati početno stanje? Zadatak kaže editovanje podataka. Ako dozvoliš izmjenu početnog stanja, formula dostupnog stanja se mijenja. To može biti OK, ali testiraj. Broj kartice i CVC – zadatak ne zabranjuje edit.
2. Kako dobiti transakcije? Opcije:
   * u `GetById` kartice vratiš nested listu transakcija u Response;
   * ili na frontendu filtriraš `Orders` po id-u kartice.
   
   Čistije je da backend vrati to što UI treba, jer Orders trenutno vraća samo narudžbe korisnika, a ne nužno filter po kartici.
3. Datum isteka u edit formi treba biti popunjen.

### U kojem dijelu projekta se to radi

* Flutter: `payment_card_details.dart`, `profile_screen.dart`
* Backend: Response kartice i/ili Order search filter po `PaymentCardId`
* `OrderSearchObject` trenutno filtrira status, ne karticu – možda treba proširiti

### Koji su koraci

1. Lista na profilu: tap → details u edit modu.
2. Forma ima Save koji zove `update`.
3. Ispod forme ListView transakcija.
4. Ako nema transakcija, pokaži prazno stanje, ne error.

### Šta trebam provjeriti na kraju

* [ ] Tap na karticu otvara formu sa postojećim podacima
* [ ] Save mijenja podatke
* [ ] Nova kartica nema transakcija
* [ ] Nakon uspješnog checkouta, na toj kartici se vidi nova transakcija (datum + iznos)
* [ ] Transakcija druge kartice se ne vidi ovdje

---

## 2.7. Zahtjev 7 – Predaja

### Šta zadatak kaže

Na backendu: **Clean Solution** (desni klik na naziv solution-a).

U Flutter projektu, u terminalu VS Code-a:

```
flutter clean
```

Zapakuj projekat (.zip ili .rar) u folder imenovan brojem indeksa.

Postavi na FTP: `Upload/RSII`, user/pass `student_250250`.

### Šta to zapravo znači

Ne predaji `bin/`, `obj/`, `build/` nered. Clean smanjuje veličinu i izbjegava probleme.

Ime zip-a = tvoj indeks. Ne `mojIspit.zip`.

### Šta trebam provjeriti

* [ ] Clean Solution urađen
* [ ] `flutter clean` urađen u **mobile** projektu (to je dio koji si mijenjala; ako si dirala i desktop, i tamo)
* [ ] Zip se zove kao indeks
* [ ] Connection string u predaji: razmisli da li ostaviti ispitni server. Obično ostaviš onako kako radi na ispitu.
* [ ] Nisi zaboravila migracijske fajlove (oni MORAJU biti u zipu)

---

## 2.8. Sažeta mapa cijelog ispita

```
PRIPREMA
  connection string → Update-Database → login customer1

BACKEND NOVI ENTITET
  PaymentCard + tvoj indeks
  1 korisnik : N kartica
  validacije 12 cifara / 3 cifre / datum / decimal >= 0
  CRUD lanac kao Category

BACKEND PLAĆANJE
  Order dobija FK na karticu
  Checkout bira karticu
  stanje = početno - suma uspješnih narudžbi
  ne smije u minus
  istekla kartica ne prolazi

FLUTTER MOBILE
  Profile: lista kartica + dodaj
  payment_card_details.dart: create + edit + transakcije
  Cart: odabir kartice prije Place order
```

Ako na papiru ne možeš nacrtati ovu mapu, još nisi spremna pisati kod.

---

# 3. Kako čitati RSII zadatak

Ovo je vještina koju prenosiš na **svaki** budući RSII ispit.

## 3.1. Prva tri pitanja

Čim dobiješ PDF, na papir napiši:

1. **Koji novi pojam se pojavljuje?** (ovdje: platna kartica)
2. **Koji stari pojam se mijenja?** (ovdje: Order, checkout, profil, korpa)
3. **Koji UI?** (ovdje: Flutter mobile profil + checkout, ne desktop)

Ako preskočiš pitanje 2, uradićeš CRUD kartice i dobiti pola bodova, jer plaćanje i Order neće raditi.

## 3.2. Tabela ključnih riječi → šta napraviti u OVOM templateu

Ovo je RSII rječnik. RS1 rječnik (Command/Query) ovdje ne važi.

### Ako u zadatku piše da treba Entity

**Kako prepoznati:** imenica koju sistem treba pamtiti: kartica, recenzija, kategorija, proizvod.

**Šta napraviti:** klasa u `eCommerce.Services/Database/`.

**U ovom ispitu:** platna kartica.

### Ako piše enum

**Kako prepoznati:** zatvorena lista stanja/tipova: „tip može biti A, B ili C", „status: aktivna/blokirana".

**Šta napraviti:** `enum` pored entiteta ili u istom fajlu. U ovom templateu `OrderStatus` živi u `Order.cs`. Folder `eCommerce.Model/Enums` je predložen u `scaffold.bash`, ali u stvarnom projektu **nije korišten**.

**U ovom ispitu:** nije eksplicitno tražen novi enum. Nemoj ga izmišljati bez potrebe. Ako želiš `CardType`, to nije iz teksta – gubiš vrijeme.

### Ako piše da treba dodati property

**Kako prepoznati:** „kartica treba sadržavati X", „proširiti entitet Y kako bi čuvao Z".

**Šta napraviti:** polje na Entity + vjerovatno Request/Response + migracija.

**U ovom ispitu:** polja kartice; FK na `Order`.

### Ako piše relacija

**Kako prepoznati:**

* „jedan korisnik može imati više kartica" → 1:N
* „proizvod pripada više kategorija, kategorija ima više proizvoda" → N:M (u templateu preko `ProductCategory`)
* „narudžba koristi jednu karticu" → N:1 sa strane narudžbe

**Šta napraviti:** `UserId` + `User` navigation + `ICollection` na drugoj strani + konfiguracija ako treba Restrict/Cascade.

### Ako piše EF konfiguracija

**Kako prepoznati:** relacija, delete ponašanje, unique, preciznost decimala.

**Šta napraviti u ovom templateu:**

* jednostavna polja → Data Annotations na entitetu (`[Required]`, `[MaxLength]`, `[Column(TypeName = "decimal(18,2)")]`);
* relacije koje nisu očite → `CreateConfiguration` u `eCommerceConfiguration.cs`.

Nema odvojenih `IEntityTypeConfiguration<T>` klasa. Sve je u partial `ECommerceDbContext`.

### Ako piše migracija / baza / tabela

**Kako prepoznati:** „dodati baznu tabelu", „kreirajte bazu migracijama", bilo kakva izmjena Entityja.

**Šta napraviti:** `Add-Migration Ime` pa `Update-Database`. Bez migracije tvoja C# klasa **ne postoji** u SQL-u.

### Ako piše Create / dodati / unijeti / omogućiti dodavanje

U RS1 bi to bio Create Command.

**U RSII:** `InsertRequest` + `InsertValidator` + `InsertAsync` (već postoji u `BaseCRUDService`) + `POST` u `BaseCRUDController`.

### Ako piše Update / izmijeniti / editovanje

**U RSII:** `UpdateRequest` + `UpdateValidator` + `UpdateAsync` + `PUT {id}`.

### Ako piše Delete / obrisati

**U RSII:** `DeleteAsync` već postoji u bazi. Ovaj ispit **ne traži** brisanje kartice. Nemoj gubiti vrijeme na delete dugme osim ako stigneš i želiš.

### Ako piše List / prikazati sve / pregled

**U RSII:** `GetAllAsync` + `SearchObject` + `GET` bez id-a. Response je `PageResult<T>`.

### Ako piše GetById / detalji / pregled detalja

**U RSII:** `GetByIdAsync` + `GET {id}`.

### Ako piše DTO

**U RSII:** ne postoji folder `DTOs`. Postoje:

* `eCommerce.Model/Requests/` – šta klijent šalje;
* `eCommerce.Model/Responses/` – šta API vraća;
* `eCommerce.Model/SearchObjects/` – filteri na GET.

To SU DTO-i. Samo se drugačije zovu.

### Ako piše validator / mora sadržavati / ne smije / samo cifre

**U RSII:** FluentValidation u `eCommerce.Services/Validators/`. Data Annotations na Request klasama su većinom zakomentarisane kod `Category` – prava validacija je FluentValidation.

### Ako piše endpoint / API / Web API

**U RSII:** Controller u `eCommerce.WebAPI/Controllers/` koji **nasljeđuje** `BaseCRUDController` ili `BaseReadController`. Često ne pišeš GET/POST ručno – nasljeđuješ ih.

### Ako piše frontend / Flutter / forma / profil / ekran

**U ovom ispitu:** `ecommerce_mobile`. Desktop diraj samo ako zadatak kaže admin.

### Ako piše filter / search

**U RSII:** novo polje na `*SearchObject` + logika u `ApplyFilters`.

### Ako piše pagination

Već postoji u `BaseSearchObject`: `Page`, `PageSize`, `IncludeTotalCount`. Flutter `BaseProvider.get` već šalje query string.

### Ako piše „prijavljeni korisnik" / „trenutno prijavljen"

**U RSII:** `IAuthenticatedUserAccessor.GetUserId()`. Pogledaj `OrderService` i `ProductReviewService`. Nemoj vjerovati `userId` iz body-ja bez provjere.

### Ako piše poslovno pravilo / ne smije / spriječiti / ako nema dovoljno

**U RSII:** u servisu baci `ClinetException("jasna poruka")`. `ExceptionFilter` to pretvara u HTTP 400. Flutter `ApiClientException` prikaže poruku.

Nemoj bacati običan `Exception` – to postane 500 i poruka „Server side error".

### Ako piše „računa se" / „ne čuva se kao vrijednost"

**To je computed vrijednost.** Nije kolona. Računaš je u servisu ili kao `[NotMapped]` property. U ovom ispitu: dostupno stanje kartice.

Pogledaj postojeći primjer `[NotMapped]` na `OrderItem.Total`.

### Ako piše state machine / activate / draft

To postoji za **Product**, nije dio ovog ispita. Nemoj dirati `ProductStateMachine` osim ako novi zadatak to traži.

---

## 3.3. Mini vježba prepoznavanja (bez koda)

Pročitaj rečenicu i reci naglas šta fali:

**„Administrator može dodati novu kategoriju."**

* kategorija → Entity (već postoji)
* dodati → Insert
* administrator → možda `[Authorize]` / uloga, ne nužno novi entitet

**„Jedan korisnik može imati više kartica."**

* dva entiteta: User (postoji) i Card (novi)
* 1:N
* FK na kartici, ne na useru

**„Stanje se računa na osnovu početnog stanja umanjenog za sumu transakcija."**

* NE nova kolona current balance
* DA formula
* DA historija (ovdje: Orders)

**„Na profilu prikazati kartice."**

* Flutter mobile ProfileScreen
* GET lista
* filter po useru

Ako možeš ovako čitati, kod je samo prepisivanje obrasca.

---

# 4. Arhitektura projekta

## 4.1. Šta vidiš kad otvoriš solution

Otvori `eCommerce/eCommerce.sln`.

Projekti:

```
eCommerce.sln
├── eCommerce.WebAPI              ← HTTP, kontroleri, Program.cs, JWT, Swagger
├── eCommerce.Services            ← Entity, DbContext, servisi, validatori, migracije
├── eCommerce.Model               ← Request, Response, SearchObject, Access, Exception
└── eCommerce.Common.Services     ← CryptoService (hash lozinke) – rijetko diraš na ovom ispitu
```

Pored toga, van C# solutiona:

```
eCommerce/UI/ecommerce_desktop    ← Flutter admin (Windows/desktop)
eCommerce/UI/ecommerce_mobile     ← Flutter kupac (profil, korpa, proizvodi)
eCommerce/docker-compose.yml      ← lokalni SQL Server na portu 1435
```

## 4.2. Kako su projekti povezani

Iz `scaffold.bash` i stvarnih `.csproj` fajlova:

```
WebAPI  →  Services  →  Model
WebAPI  →  Common.Services
Services → Common.Services
```

Pravila:

* **WebAPI smije** zvati Services.
* **Services smije** zvati Model.
* **Model ne smije** zvati Services (nema EF, nema DbContext).
* **WebAPI ne treba** direktno koristiti Entity klase u kontrolerima. Kontoleri rade sa Request/Response.

Ako u kontroler napišeš `using eCommerce.Services.Database` i vraćaš `PaymentCard` entitet, prekršila si princip. Vraćaš Response.

## 4.3. eCommerce.Model – „šta ulazi i šta izlazi"

### Čemu služi

Ovo je ugovor između API-ja i klijenta (Flutter, Swagger).

### Šta se tamo nalazi

| Folder | Primjer | Uloga |
|--------|---------|--------|
| `Requests/` | `CategoriesInsertRequest` | tijelo POST/PUT |
| `Responses/` | `CategoryResponse` | šta API vraća |
| `SearchObjects/` | `CategorySearchObject` | query string na GET |
| `Access/` | `UserLoginRequest` | login/refresh |
| `Exceptions/` | `ClinetException` | poslovna greška (naziv ima typo: Clinet) |

### Šta NE treba biti tamo

* `ECommerceDbContext`
* `[Key]` / `[ForeignKey]` osim ako baš želiš DataAnnotations na requestu – u ovom projektu validacija je FluentValidation
* navigation collections ka drugim tabelama kao na entitetu
* password hash

### Kako komunicira

Flutter šalje JSON koji odgovara Request klasi. API vraća JSON koji odgovara Response klasi. Mapster u servisu pretvara Entity ↔ Response.

## 4.4. eCommerce.Services – srce aplikacije

Ovdje je **i domen i infrastruktura**. Za RS1 mozak: Domain + Application + Infrastructure su spojeni u jedan projekat.

### Čemu služi

* opisuje tabele (Entity);
* priča sa SQL-om (DbContext, migracije);
* radi poslovnu logiku (servisi);
* validira Request (FluentValidation).

### Šta se tamo nalazi

```
eCommerce.Services/
├── Database/
│   ├── Category.cs, User.cs, Order.cs, Product.cs, ...
│   ├── eCommerceDbContext.cs          ← DbSet-ovi
│   ├── eCommerceConfiguration.cs      ← relacije
│   └── eCommerceSeed.cs               ← početni podaci
├── Migrations/                        ← historija baze
├── Validators/                        ← FluentValidation
├── ProductStateMachine/               ← samo za Product
├── BaseReadService.cs
├── BaseCRUDService.cs
├── CategoryService.cs, OrderService.cs, ...
└── ICategoryService.cs, IOrderService.cs, ...
```

Napomena: `scaffold.bash` predlaže foldere `Interfaces/` i `Implementations/`, ali stvarni kod **nije** tako organizovan. Interfejsi i klase stoje u rootu `eCommerce.Services`. Ti radi **kao postojeći kod**, ne kao scaffold skripta.

### Šta NE treba biti tamo

* `[HttpGet]`, `[ApiController]` – to je WebAPI
* Flutter
* `builder.Services.AddScoped` – to je `Program.cs`

### Kako komunicira

Kontroler pozove `ICategoryService`. Servis koristi `_dbContext` i `_mapper`. Servis vraća `CategoryResponse`.

## 4.5. eCommerce.WebAPI – vrata prema vani

### Čemu služi

Prima HTTP, autentifikuje JWT, hvata greške, vraća JSON.

### Šta se tamo nalazi

```
eCommerce.WebAPI/
├── Controllers/
│   ├── BaseReadController.cs
│   ├── BaseCRUDController.cs
│   ├── CategoriesController.cs
│   ├── OrdersController.cs
│   ├── ProductsController.cs
│   └── ...
├── Filters/
│   ├── ExceptionFilter.cs
│   └── AuthorizationAttribute.cs
├── Services/          ← AccessManager, HttpAuthenticatedUserAccessor
├── Program.cs         ← DI, JWT, Swagger, Mapster config
├── appsettings.json
└── appsettings.Development.json
```

### Šta NE treba biti tamo

* LINQ prema tabelama
* `SaveChanges`
* poslovna pravila („ako nema para, nemoj")

Ako pravilo staviš u kontroler, Flutter to ne može zaobići samo ako svi klijenti budu pošteni. Swagger nije pošten. Zato pravilo ide u servis.

## 4.6. Flutter

Dva odvojena projekta.

### ecommerce_desktop

Admin aplikacija:

* lista/detalji proizvoda, kategorija, korisnika, recenzija;
* `MasterScreen` layout;
* `flutter_form_builder`.

**Ovaj ispit je ne traži.** Nemoj tamo dodavati „Moje kartice".

### ecommerce_mobile

Kupac:

* Home, Categories, Cart, Profile;
* login;
* checkout iz korpe;
* recenzije.

**Ovaj ispit se radi ovdje.**

Struktura koju trebaš znati:

```
lib/
├── main.dart                 ← MultiProvider + LoginPage
├── layouts/container_screen.dart  ← bottom navigation, tab Profile
├── models/                   ← json_serializable klase
├── providers/                ← HTTP klijenti
│   └── base_provider.dart    ← get/insert/update/delete
├── screens/
│   ├── profile_screen.dart   ← OVDJE sekcija kartica
│   └── cart_list_screen.dart ← OVDJE odabir kartice za plaćanje
└── utils/
    ├── api_client_exception.dart
    └── utils_widgets.dart
```

## 4.7. Tok zahtjeva kroz slojeve

```
Flutter provider
    HTTP JSON
        Controller (WebAPI)
            IXxxService (interface)
                XxxService (Services)
                    FluentValidation
                    Mapster
                    ECommerceDbContext
                        SQL Server
```

Obrnuto za odgovor:

```
SQL red
    Entity
        Mapster → Response
            JSON
                Dart model
                    Widget
```

## 4.8. Gdje se u templateu inače dodaje šta

Ovo je mapa koju trebaš znati napamet kao **lokacije**, ne kao kod.

| Šta dodaješ | Tačna lokacija |
|-------------|----------------|
| Entity klasa | `eCommerce.Services/Database/Ime.cs` |
| Enum | obično u istom fajlu kao Entity (`OrderStatus` u `Order.cs`) |
| EF relacija | `Database/eCommerceConfiguration.cs` metoda `CreateConfiguration` |
| DbSet | `Database/eCommerceDbContext.cs` |
| Seed | `Database/eCommerceSeed.cs` (ovaj ispit ne traži seed kartica) |
| Migracija | PMC, default project = Services |
| Insert/Update Request | `eCommerce.Model/Requests/` |
| Response | `eCommerce.Model/Responses/` |
| Search | `eCommerce.Model/SearchObjects/` |
| Validator | `eCommerce.Services/Validators/` |
| Interface servisa | `eCommerce.Services/IImeService.cs` |
| Servis | `eCommerce.Services/ImeService.cs` |
| Registracija | `eCommerce.WebAPI/Program.cs` |
| Controller | `eCommerce.WebAPI/Controllers/` |
| Mapster specijalno mapiranje | `Program.cs` `TypeAdapterConfig<...>NewConfig()` |
| Flutter model | `UI/ecommerce_mobile/lib/models/` |
| Flutter provider | `UI/ecommerce_mobile/lib/providers/` |
| Flutter screen | `UI/ecommerce_mobile/lib/screens/` |
| Flutter DI | `UI/ecommerce_mobile/lib/main.dart` |

## 4.9. Referentni primjeri u templateu – kopiraj OBRASAC

Prije pisanja bilo čega za kartice, otvori i prouči:

**Jednostavan CRUD:** `Category`

* Entity: `Database/Category.cs`
* Request: `Requests/CategoriesInsertRequest.cs`, `CategoriesUpdateRequest.cs`
* Response: `Responses/CategoryResponse.cs`
* Search: `SearchObjects/CategorySearch.cs`
* Validator: `Validators/CategoryInsertValidator.cs`
* Service: `CategoryService.cs`
* Controller: `CategoriesController.cs`

**CRUD + prijavljeni korisnik:** `ProductReview`

* servis koristi `_userAccessor.GetUserId()`
* kontroler ima `[Authorize]`

**Poslovno pravilo + transakcija:** `OrderService.CheckoutAsync`

* `ClinetException` za praznu korpu, nestali proizvod, nema na stanju
* `BeginTransactionAsync`
* ovo je najbliži obrazac za plaćanje karticom

**Flutter lista + forma:**

* mobile: `AddReviewScreen` (prosta forma, insert, pop)
* mobile: `ProfileScreen` (reload nakon povratka)
* desktop: `CategoryDetailsScreen` (create/edit isti ekran) – koristan obrazac, iako je desktop

**Pravilo:** Novi entitet = isti folder pattern, ista imena fajlova, samo drugi naziv.

---

# 5. Entity – od teksta zadatka do klase

### Općenito

**Entity** je C# klasa koja predstavlja **jedan red u tabeli**.

Ako baza ima tabelu `Categories`, postoji klasa `Category`. Jedan objekat `Category` = jedan red.

U ovom templateu entity NIJE u Domain projektu. Entity je u:

`eCommerce.Services/Database/`

Nema `BaseEntity` klase. Svaki entity sam ima `Id`, a često i `CreatedAt` / `UpdatedAt`.

### Kako se to prepoznaje na ispitu

Pitaj se: „Da li sistem treba ovo **pamtiti** nakon što se aplikacija ugasi?"

Ako da, to je entity (ili polje na entityju).

* kartica → da, pamti se
* dostupno stanje → ne kao kolona, računa se
* odabrana kartica u dropdownu dok korisnik gleda korpu → privremeno, nije tabela

### Kako se koristi u ovom ispitu

Novi entity za karticu + izmjena `User` (kolekcija) + izmjena `Order` (FK).

### Kako odrediti properties

Iz teksta izvuci imenice koje su podaci:

* broj kartice → string (cifre, ali u C# i dalje `string`, jer može imati vodeće nule)
* CVC → string (iste razloga: `012` nije isti kao int `12` ako ikad zatreba, a zadatak kaže 3 cifre)
* datum isteka → datum
* početno stanje → decimal
* korisnik → ne cijeli User objekat kao kolona, nego `UserId` (int)

### Kada koji tip

| Situacija | Tip | Zašto |
|-----------|-----|--------|
| ime, broj kartice, CVC, adresa | `string` | tekst, vodeće nule, fiksna dužina cifara |
| id, količina, FK | `int` | cijeli broj |
| cijena, stanje novca | `decimal` | novac nikad `float`/`double` u bazi |
| datum isteka, created at | `DateTime` | vrijeme |
| je li aktivno | `bool` | da/ne |
| zatvoren skup vrijednosti | `enum` | npr. `OrderStatus` |
| može nedostajati | `int?`, `DateTime?`, `string?` | npr. stari Order bez kartice |

**Zašto string za broj kartice, ne `long`?**

Zadatak kaže 12 cifara. `long` može držati 12 cifara, ali:

* validacija „samo cifre, tačno 12" je prirodnija na stringu;
* ne gubiš vodeće nule ako ikad dođu;
* template već čuva „brojeve koji nisu za računanje" kao string (`OrderNumber`, `SKU`).

**Zašto `decimal` za novac, ne `double`?**

`double` je binarni i za novac može dati 0.1 + 0.2 = 0.30000000004. U templateu cijena je:

```csharp
[Column(TypeName = "decimal(18,2)")]
public decimal Price { get; set; }
```

To kopiraš za početno stanje kartice i za iznose.

### Nullable property

Pitaj: „Da li red u bazi može postojati bez te vrijednosti?"

* `UserId` na kartici: ne, kartica mora imati vlasnika → `int` ne `int?`
* FK kartice na `Order`: stare narudžbe nemaju karticu → `int?`
* `UpdatedAt` na Category: da, nullable

### Kako se koristi „BaseEntity" ovdje

Nikako. Nema ga. Kad vidiš u RS1 vodiču `BaseEntity`, zanemari.

Umjesto toga, gledaj šta **većina** entiteta ima i budi dosljedna:

* `[Key] public int Id`
* `CreatedAt = DateTime.UtcNow`
* često `UpdatedAt`
* često `IsActive`

`BaseCRUDService.InsertAsync` refleksijom postavlja `CreatedAt` ako property postoji. `UpdateAsync` postavlja `UpdatedAt` ako postoji. Zato se isplati imati ta polja – baza servisa već zna za njih.

### Kako razmišljati prije pisanja klase

Na papiru tabela:

| Polje | Tip | Obavezno? | Zašto |
|-------|-----|-----------|-------|
| Id | int | da | PK |
| UserId | int | da | veza na korisnika |
| ... | ... | ... | iz zadatka |

Tek onda otvori `Category.cs` i `ProductReview.cs` i piši **sličnim stilom**.

Generički primjer (NIJE rješenje ispita):

```csharp
public class Ticket
{
    [Key]
    public int Id { get; set; }

    [Required]
    [MaxLength(100)]
    public string Title { get; set; } = string.Empty;

    public int UserId { get; set; }

    [ForeignKey("UserId")]
    public User User { get; set; } = null!;

    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
}
```

Ovo ti pokazuje oblik. Kartica nije Ticket. Ti pišeš polja iz **svog** zadatka.

### Šta trebam provjeriti

* [ ] Klasa je u namespace `eCommerce.Services.Database`
* [ ] Ima `Id`
* [ ] Novac je `decimal` sa `decimal(18,2)`
* [ ] FK je `int` + navigation
* [ ] Nema kolone za izračunato stanje

---

# 6. Enum

### Općenito

**Enum** je imenovani skup cijelih brojeva.

```csharp
public enum OrderStatus
{
    Pending,      // 0
    Processing,   // 1
    Shipped,      // 2
    Delivered,    // 3
    Cancelled,    // 4
    Returned      // 5
}
```

Jednostavno: umjesto da u bazi pišeš string „Processing", pišeš broj, a u kodu koristiš riječ.

### Kada ga koristiti

Kad je lista vrijednosti:

* mala;
* zatvorena;
* neće je korisnik slobodno kucati;
* ne treba joj posebna tabela (nema opisa, nema CRUD-a za te vrijednosti).

Ako bi korisnik dodavao nove tipove kroz UI, to NIJE enum, to je Entity (`Category`, `ProductType`).

### Kako se prepoznaje na ispitu

„Tip može biti: Interni, Eksterni, Freelancer – ne pravi novu tabelu."

To je klasičan enum signal.

Ako piše „kategorija proizvoda", to je tabela, jer kategorija ima ime, opis, parent, CRUD.

### Kako ga povezati sa Entityjem

Property tipa enum:

```csharp
public OrderStatus Status { get; set; } = OrderStatus.Pending;
```

### Kako ga EF Core sprema

Po defaultu kao **int**. Zato `OrderResponse.Status` je `int`, a Mapster u `Program.cs` ima:

```csharp
TypeAdapterConfig<Order, OrderResponse>.NewConfig()
    .Map(dest => dest.Status, src => (int)src.Status);
```

### Razlika enum vs string

| | Enum | String |
|--|------|--------|
| greška u kucanju | kompajler javi | „Pendng" se spremi |
| nova vrijednost | mijenjaš kod | možeš upisati bilo šta |
| čitljivost | odlična u C# | odlična u bazi |

### Kako se koristi u ovom ispitu

Novi enum **nije tražen**. Postojeći `OrderStatus` ti je koristan kad definiraš šta je „uspješna transakcija" (npr. isključi `Cancelled`).

Nemoj praviti `enum CardBrand { Visa, Mastercard }` ako zadatak to ne traži.

### Šta trebam provjeriti

* [ ] Nisam napravila enum bez teksta u zadatku
* [ ] Ako koristim `OrderStatus` u filteru, znam da se kroz API često šalje kao int

---

# 7. EF Core konfiguracija

### Općenito

**EF Core** (Entity Framework Core) je biblioteka koja pretvara tvoje C# klase u SQL tabele i obrnuto.

Ti pišeš:

```csharp
_dbContext.Set<Category>().Add(entity);
await _dbContext.SaveChangesAsync();
```

EF napiše `INSERT INTO ...`.

### Zašto postoji konfiguracija Entityja

C# klasa ne kaže sve:

* da li brisanje usera briše kartice;
* koliko decimala ima novac;
* da li je relacija obavezna.

Dio toga pišeš **atributima na klasi** (Data Annotations). Dio pišeš u `CreateConfiguration`.

### Šta se tamo definiše u OVOM templateu

Otvori `eCommerceConfiguration.cs`. Vidiš samo **relacije i delete behavior**, npr.:

```csharp
modelBuilder.Entity<ProductReview>()
    .HasOne(pr => pr.Order)
    .WithMany(o => o.ProductReviews)
    .HasForeignKey(pr => pr.OrderId)
    .OnDelete(DeleteBehavior.Restrict);
```

Ostalo je na entitetu:

* `[Key]` – primary key
* `[Required]` – NOT NULL
* `[MaxLength(100)]` – nvarchar dužina
* `[Column(TypeName = "decimal(18,2)")]` – preciznost
* `[ForeignKey("UserId")]` – FK
* `[NotMapped]` – nije kolona (`OrderItem.Total`)
* `[Range(1,5)]` – na `ProductReview.Rating`

### Primary key

U templateu uvijek `int Id` sa `[Key]`. EF će napraviti identity (auto increment).

Nemoj sama zadavati Id pri insertu. `BaseCRUDService` ima čak i zakomentarisan `GenerateNewId` – ne treba ti.

### Required / optional

* `string Name` + `[Required]` → obavezno
* `string? PhoneNumber` → može null
* `int? ParentCategoryId` → kategorija ne mora imati roditelja

### Max length

Uvijek stavi MaxLength na stringove koje korisnik unosi. Bez toga SQL često napravi `nvarchar(max)`.

Za 12 cifara, MaxLength 12 ima smisla. Za CVC, MaxLength 3.

### Precision (decimal)

Bez `decimal(18,2)` SQL Server može uzeti `decimal(18,2)` default ili nešto neočekivano. Template eksplicitno stavlja na `Price`, `TotalAmount`, `UnitPrice`. Kopiraj taj atribut za novac.

### Relationship, foreign key, delete behavior

Pogledaj postojeće:

| Relacija | DeleteBehavior | Zašto otprilike |
|----------|----------------|-----------------|
| Category parent/child | Restrict | da ne obrišeš roditelja koji ima djecu slučajno |
| ProductCategory | Cascade | spojna tabela, nema smisla bez proizvoda |
| UserRole | Cascade | spoja |
| Asset → Product | Cascade | slike proizvoda |
| ProductReview → Order | Restrict | recenzija ne smije obrisati/nestati kaosom narudžbe |

**Kako izabrati za kartice:**

* User ima kartice: ako obrišeš usera, cascade na kartice je čest.
* Order pokazuje na karticu: Restrict je pametniji, da brisanje kartice ne obriše narudžbe.

### Enum konfiguracija

Nema posebne. Int po defaultu.

### Kako iz zahtjeva zaključiti šta staviti u konfiguraciju

1. Ima li relacija? Ako da, da li je EF može pogoditi sam? 1:N sa `UserId` + `User` + `ICollection` često može. Ipak, ako trebaš Restrict, piši u `CreateConfiguration`.
2. Ima li novac? `decimal(18,2)` na entitetu.
3. Ima li fiksna dužina? MaxLength.
4. Ima li izračunato polje? NotMapped ili uopšte ne stavljaj na entity.

Ovaj template **nema** odvojene `*Configuration.cs` fajlove po entitetu. Nemoj ih uvoditi. Dopuni `CreateConfiguration`.

### Šta trebam provjeriti

* [ ] Relacija User–kartica postoji
* [ ] Relacija Order–kartica postoji
* [ ] Decimal ima precision
* [ ] Nisam stavila `[NotMapped]` na polje koje treba biti u bazi
* [ ] Nisam stavila u bazu polje koje treba biti izračunato

---

# 8. Relacije između Entityja

### Općenito

Relacija kaže kako se dva entiteta povezuju.

## 8.1. 1:1

Jedan red A odgovara tačno jednom redu B.

Primjer (generički): korisnik ima tačno jedan profil detalj.

Rijetko na ovom templateu. `User` i `Cart` bi mogli biti 1:1, ali nisu fokus ovog ispita.

**Kako prepoznati:** „svaki X ima tačno jedan Y".

## 8.2. 1:N (jedan prema više)

Jedan red A ima više redova B. Svaki B ima tačno jednog A.

Ovo je **najčešća** relacija na ispitu.

Zadatak: „Jedan korisnik može imati više kartica."

To je:

```
User 1 ──────< N PaymentCard
```

Na strani „više" stoji FK:

```csharp
public int UserId { get; set; }
public User User { get; set; } = null!;
```

Na strani „jedan" stoji kolekcija:

```csharp
public ICollection<PaymentCard...> Cards { get; set; } = new List<...>();
```

**Navigation property** – objekat ili lista u C# kojim „šetam" relaciju, npr. `card.User.Email`.

**Foreign key** – int kolona u tabeli, npr. `UserId`.

**Collection** – lista na roditelju.

Bez FK-a EF ne zna koji user. Bez navigation i dalje može raditi, ali Include i Mapster relacije postaju teži. Template uvijek stavlja oboje.

## 8.3. N:M (više prema više)

Proizvod može biti u više kategorija, kategorija ima više proizvoda.

U ovom templateu to NIJE `List<Category>` direktno na Product. Ima **spojna tabela** `ProductCategory` sa `ProductId` + `CategoryId`.

**Kako prepoznati:** „više X može imati više Y".

**U ovom ispitu N:M nije potreban.** Kartica ne pripada više korisnika. Narudžba koristi jednu karticu.

## 8.4. Kako prepoznati relaciju iz teksta

| Tekst | Relacija | FK gdje |
|-------|----------|---------|
| jedan korisnik, više kartica | 1:N | na kartici `UserId` |
| narudžba se plaća jednom karticom; kartica ima više plaćanja | 1:N | na narudžbi `PaymentCardId` |
| proizvod i kategorija obostrano više | N:M | spojna tabela |

Trik: FK stoji na strani **N**.

## 8.5. EF konfiguracija relacije

Generički oblik iz templatea:

```csharp
modelBuilder.Entity<Dijete>()
    .HasOne(d => d.Roditelj)
    .WithMany(r => r.Djeca)
    .HasForeignKey(d => d.RoditeljId)
    .OnDelete(DeleteBehavior.Restrict);
```

Čitaj naglas:

* Dijete **ima jednog** roditelja;
* roditelj **ima mnogo** djece;
* FK je `RoditeljId`;
* brisanje je Restrict.

### Kako se koristi u ovom ispitu

Dvije 1:N relacije:

1. User → kartice
2. Kartica → narudžbe (plaćanja)

To je dovoljno da izračunaš stanje i da prikažeš transakcije.

### Šta trebam provjeriti

* [ ] FK je na pravoj strani
* [ ] Navigation postoji
* [ ] ICollection inicijalizirana sa `new List<>()` da ne bude null
* [ ] Nema N:M „jer možda zatreba"

---

# 9. DbContext

### Općenito

**DbContext** je „sesija sa bazom". U ovom projektu klasa se zove `ECommerceDbContext`.

On zna:

* koje tabele postoje (`DbSet<T>`);
* kako su povezane (`OnModelCreating` → `CreateConfiguration`);
* početne podatke (`CreateSeed`).

### DbSet

```csharp
public DbSet<Category> Categories { get; set; }
```

**DbSet** je ulaz u tabelu. `Categories` je ime tabele/skupa. Query:

```csharp
_dbContext.Categories.Where(...)
```

ili generički, kako baza servisa radi:

```csharp
this._dbContext.Set<TEntity>()
```

Zato `BaseCRUDService` radi za bilo koji entity čim postoji `DbSet` (tačnije, čim je entity u modelu). `Set<T>()` radi i preko konvencije, ali u ovom projektu **svi** entiteti imaju eksplicitan DbSet. Dodaj i ti.

### Gdje se dodaje novi Entity

U `eCommerceDbContext.cs`, uz ostale DbSet-ove.

Ako zaboraviš DbSet, migracija ponekad i dalje uvrsti entity ako je otkriven preko navigacije (`User.Cards`). **Ne oslanjaj se na to.** Dodaj DbSet kao svi ostali.

### Zašto je to potrebno

Da bi:

* `_dbContext.PaymentCards.Add(...)` radilo;
* migracija napravila tabelu;
* Swagger/servis imao izvor podataka.

### Kako EF zna za Entity

`OnModelCreating`:

```csharp
CreateConfiguration(modelBuilder);
CreateSeed(modelBuilder);
```

Klase su `partial`. Zato su DbContext, Configuration i Seed u **tri fajla**, a ista klasa `ECommerceDbContext`.

To je važno: kad tražiš „gdje su relacije", nisu u glavnom fajlu sa DbSetovima. U `eCommerceConfiguration.cs`.

### Kako se koristi u ovom ispitu

Dodaješ DbSet za kartice. `Orders` i `Users` već postoje – njih samo proširuješ poljima.

### Šta trebam provjeriti

* [ ] DbSet dodan
* [ ] namespace i using ne fale
* [ ] projekat se builda prije migracije

---

# 10. Database migration

### Općenito

**Migracija** je C# fajl koji opisuje **promjenu šeme baze**: nova tabela, nova kolona, novi FK.

EF gleda tvoje entityje, usporedi ih sa zadnjim snapshotom (`ECommerceDbContextModelSnapshot.cs`), i napiše razliku.

### Zašto se pravi

C# nije baza. Ako samo dodaš klasu, SQL Server ne zna za nju. API će pasti čim uradiš query na nepostojeću tabelu.

### Kada se pravi

Svaki put kad:

* dodaš entity;
* dodaš/promijeniš/obrišeš property koji je mapiran;
* promijeniš relaciju;
* promijeniš required/length/precision.

### Šta se dešava nakon promjene Entityja

1. Promijeniš C#.
2. `Add-Migration NekoIme`.
3. Pregledaš generisani fajl: ima li `CreateTable` / `AddColumn` koje očekuješ.
4. `Update-Database`.
5. U SSMS-u osvježiš tabele.

### Šta znači Update-Database

Izvršava sve migracije koje u toj bazi još nisu primijenjene. Piše u tabelu `__EFMigrationsHistory`.

Zadatak na ispitu prvo traži `Update-Database` da se **postojeće** template migracije primijene na tvoju novu bazu. Kasnije, nakon tvog koda, radiš **novu** migraciju pa opet `Update-Database`.

### Kako prepoznati da treba nova migracija

Ako si dirnula `Database/*.cs` entitete ili konfiguraciju, treba migracija.

Ako si dirnula samo Validator ili Flutter, **ne treba**.

### Konkretne komande za ovaj template

Ispit kaže Package Manager Console, **Service projekat**.

U Visual Studio:

1. Default project: `eCommerce.Services`
2. Startup project: `eCommerce.WebAPI` (jer tamo je connection string)

```
Add-Migration AddPaymentCardsIBXXXX
Update-Database
```

Zamijeni ime migracije nečim što opisuje tvoju izmjenu. Ime migracije nije ime entiteta – može biti `AddPaymentCards`.

Ako PMC ne vidi `Add-Migration`, Tools paket je u `eCommerce.Services.csproj`:

```
Microsoft.EntityFrameworkCore.Tools
```

**dotnet CLI** (ako radiš iz terminala, iz foldera `eCommerce`):

```
dotnet ef migrations add AddPaymentCards --project eCommerce.Services --startup-project eCommerce.WebAPI
dotnet ef database update --project eCommerce.Services --startup-project eCommerce.WebAPI
```

Ako `dotnet ef` nije prepoznat:

```
dotnet tool install --global dotnet-ef
```

Na ispitu je PMC dovoljan. Uči PMC.

### Tipične greške

| Greška | Zašto | Kako popraviti |
|--------|-------|----------------|
| `No database provider has been configured` | krivi startup project | startup = WebAPI |
| `Cannot find DbContext` | default project nije Services | default = Services |
| migracija prazna | nisi sačuvala entity fajl / nema DbSet / krivi projekat | build, pa ponovo |
| `There is already an object named ...` | baza već ima tabelu, historija nije usklađena | nemoj paničiti; na ispitu koristi **novu** bazu svog indeksa |
| connection fails | krivi server/port/login | provjeri `appsettings.Development.json` |
| Update-Database uradi tuđu bazu | ostao `Database=eCommerce` | odmah ispravi ime baze |

### Kako se koristi u ovom ispitu

Najmanje dvije situacije:

1. početni `Update-Database` (postojeće migracije);
2. tvoja migracija za kartice + Order FK.

Možeš oba entiteta u **jednoj** migraciji ako uradiš sav Database sloj pa tek onda `Add-Migration`. To je čak bolje: manje nereda.

### Šta trebam provjeriti

* [ ] Migracijski fajlovi postoje u `eCommerce.Services/Migrations/`
* [ ] Snapshot je ažuriran
* [ ] U SSMS-u vidim novu tabelu i novu kolonu na Orders
* [ ] Predajem i migracije, ne samo klase

---

# 11. Servisi umjesto CQRS

Ovdje namjerno **ne** učim CQRS kao u RS1. Učim šta ovaj template stvarno radi, pa mapiram riječi iz RS1 da se ne zbuniš.

## 11.1. Šta je servis u ovom projektu

**Servis** je klasa koja radi posao za jedan entitet ili jedan slučaj korištenja.

* `CategoryService` – CRUD kategorija
* `UserService` – CRUD korisnika + login pomoćne stvari
* `OrderService` – lista narudžbi + `CheckoutAsync`
* `ProductService` – CRUD + state machine

Kontroler je tanak. Servis je debeo.

## 11.2. Command vs Query – kako to izgleda ovdje

### Općenito (pojam)

**Command** = naredba koja **mijenja** podatke: create, update, delete, checkout, activate.

**Query** = pitanje koje **čita** podatke: list, get by id, search.

### Kako se to prepoznaje na ispitu

* „dodati karticu" → command / Insert
* „prikazati kartice" → query / GetAll
* „platiti" → command / Checkout (posebna metoda, nije običan Insert na Order)

### Kako se koristi u ovom ispitu

Nema `CreatePaymentCardCommand.cs`. Ima:

* `InsertAsync` na kartica servisu;
* `GetAllAsync` / `GetByIdAsync`;
* `CheckoutAsync` na order servisu (već postoji, ti ga proširuješ).

## 11.3. Hijerarhija klasa – ovo MORAS razumjeti

```
IBaseReadService<TResponse, TSearch>
    GetByIdAsync
    GetAllAsync

IBaseCRUDService<...> : IBaseReadService
    InsertAsync
    UpdateAsync
    DeleteAsync

BaseReadService<TEntity, TResponse, TSearch>
    koristi DbContext + Mapster
    ApplyFilters (abstract – MORAŠ override)
    IncludeRelatedEntitiesAsync (virtual)

BaseCRUDService<...> : BaseReadService
    InsertAsync + FluentValidation
    UpdateAsync + FluentValidation
    DeleteAsync
```

Kontroleri:

```
BaseReadController  → GET, GET/{id}
BaseCRUDController  → + POST, PUT/{id}, DELETE/{id}
```

### Šta to znači za tebe na ispitu

Ako praviš običan CRUD (kartice: lista, detalj, insert, update):

1. `IPaymentCard...Service : IBaseCRUDService<Response, Search, Insert, Update>`
2. klasa `: BaseCRUDService<Entity, Response, Search, Insert, Update>, I...Service`
3. kontroler `: BaseCRUDController<Response, Search, Insert, Update, I...Service>`

Ako je nešto samo čitanje + specijalna akcija (Order):

* `IOrderService : IBaseReadService<...>` + `CheckoutAsync`
* `OrdersController : BaseReadController` + `[HttpPost("Checkout")]`

Kartice su CRUD. Checkout je specijalna akcija na **Order** servisu.

## 11.4. Zašto ne instancirati DbContext u kontroleru

Zadatak to izričito zabranjuje. Razlog:

* `BaseCRUDService` već radi Add/SaveChanges;
* validator se zove u InsertAsync;
* `CreatedAt` se postavlja u InsertAsync;
* ExceptionFilter očekuje određene izuzetke.

Ako zaobiđeš servis, zaobiđeš pola projekta.

## 11.5. Mapster

**Mapster** – biblioteka koja kopira propertyje istog imena iz jednog objekta u drugi.

`InsertRequest.Name` → `Entity.Name` → `Response.Name` ako se zovu isto.

Zato **isti nazivi polja** štede kod. Ako se u Requestu zove `CardNumber`, na Entityju nemoj staviti `BrojKartice` bez Mapster configa.

Posebna mapiranja idu u `Program.cs` (`TypeAdapterConfig`). Treba ti samo ako imena nisu ista ili ako mapiraš nested stvari (kao `OrderItem.ProductName`).

## 11.6. Mini checklist

* [ ] Znam razliku: kartica CRUD vs Order checkout
* [ ] Znam koju baznu klasu naslijediti
* [ ] Ne pravim Commands folder

---

# 12. Insert / Create

### Općenito

Insert = novi red u tabeli.

U ovom templateu tok je već napisan u `BaseCRUDService.InsertAsync`:

1. `_insertValidator.ValidateAsync(request)`
2. ako nije valid → `ValidationException` → ExceptionFilter → HTTP 400
3. `MapInsertRequestToEntity(request)` (Mapster)
4. ako postoji `CreatedAt`, stavi `DateTime.UtcNow`
5. `_dbContext.Set<TEntity>().Add(entity)`
6. `SaveChangesAsync()`
7. mapiraj u `TResponse` i vrati

Kontroler:

```csharp
[HttpPost]
public async Task<ActionResult<TResponse>> Create([FromBody] TInsertRequest request)
{
    var result = await _service.InsertAsync(request);
    return result;
}
```

Ruta: `POST /Categories` jer je `[Route("[controller]")]` i klasa `CategoriesController`.

### Kako prepoznati Create zahtjev u tekstu

„dodavanje", „unos", „administrator može dodati", „dugme Dodaj karticu".

### Šta treba napraviti

Za običan slučaj **ne prepisuješ** InsertAsync. Napraviš:

* InsertRequest
* InsertValidator
* Service koji samo override-a `ApplyFilters`
* Controller prazan (samo konstruktor)

Insert „besplatno" dođe iz bazne klase.

### Kada override-ati Insert

Kad treba **više od mapiranja**, npr.:

* `UserService.InsertAsync` – hash lozinke, provjera da email nije zauzet
* `ProductReviewService` – postavi `UserId` iz JWT-a, ne vjeruj klijentu
* kartice: vjerovatno postavi `UserId` iz `_userAccessor.GetUserId()` da korisnik ne može dodati karticu tuđem id-u

To je tačka na kojoj **moraš razmisliti**, ne kopirati slijepo Category.

Category ne treba current user. Kartica treba: „kartice trenutno prijavljenog korisnika".

Generički smjer razmišljanja (nije gotov kod ispita):

```csharp
protected override TEntity MapInsertRequestToEntity(TInsertRequest request)
{
    var entity = base.MapInsertRequestToEntity(request);
    // ovdje smiješ dopuniti polja koja ne dolaze iz forme
    return entity;
}
```

ili override cijelog `InsertAsync` kao UserService.

### Kako se vrši validacija

Pogledaj `CategoryInsertValidator`: `RuleFor(x => x.Name).NotEmpty()...`

Za „samo cifre, tačno N" koristiš FluentValidation `Matches` i `Length`. Vidi sekciju 18.

### Kako se kreira Entity i snima

Ne radiš `new` u kontroleru. Mapster napravi entity. `Add` + `SaveChangesAsync` su u bazi servisa.

### Šta se vraća kao response

Response DTO, ne Entity. Flutter `insert` očekuje JSON koji `fromJson` razumije.

### Kako se koristi u ovom ispitu

Create kartice sa profila. UserId iz tokena. Validacije iz zadatka.

### Šta trebam provjeriti

* [ ] POST u Swaggeru radi
* [ ] Nevalidan body → 400, ne 500
* [ ] U bazi se vidi red
* [ ] UserId je moj, ne 0 i ne tuđi

---

# 13. Update

### Općenito

Update = nađi postojeći red, promijeni polja, snimi.

`BaseCRUDService.UpdateAsync(int id, TUpdateRequest request)`:

1. validiraj request
2. `Find(id)`
3. ako nema → `KeyNotFoundException`
4. `MapUpdateRequestToEntity` (Mapster kopira request preko entityja)
5. postavi `UpdatedAt` ako postoji
6. `SaveChangesAsync`
7. vrati Response

HTTP: `PUT /Categories/{id}`

### Kako prepoznati

„editovanje", „izmjena", „odabirom kartice omogućiti editovanje".

### Posebno: pronalazak i provjera

Ako `Find` vrati null, baza servisa baci `KeyNotFoundException`. `GetById` u `BaseReadController` to pretvara u 404. `Update` u `BaseCRUDController` **nema try/catch** za 404 – ExceptionFilter može od toga napraviti 500.

To je postojeća slabost templatea. Za ispit: ili ostavi kako jeste (kao Category), ili u svom override-u baci `ClinetException`. Nemoj gubiti sat vremena na savršen status code ako ostalo ne radi. Ali znaj da 404 za GetById već postoji.

### Izmjena podataka

Mapster prepisuje ista imena. Ako u UpdateRequest staviš `UserId`, a ne želiš da korisnik premjesti karticu na tuđi nalog, **nemoj** staviti `UserId` u UpdateRequest, ili ga ignoriši u override-u.

### Kako se koristi u ovom ispitu

Isti ekran `payment_card_details.dart` u edit modu zove `provider.update(id, request)`.

Pitaj se šta smije da se mijenja. Početno stanje: ako ga izmijeniš, formula dostupnog novca se mijenja. Zadatak dozvoljava edit podataka. Testiraj da transakcije i dalje imaju smisla.

### Šta trebam provjeriti

* [ ] PUT sa ispravnim id radi
* [ ] PUT sa lažnim id ne pravi novi red
* [ ] Ne mogu update-ati tuđu karticu (override + userAccessor)

---

# 14. Delete

### Općenito

`DeleteAsync(id)`: Find, Remove, SaveChanges.

HTTP: `DELETE /Categories/{id}` → `204 NoContent`

### Kako prepoznati

„obrisati", „ukloniti".

### Kako se koristi u ovom ispitu

**Nije traženo.** `BaseCRUDController` će delete i dalje izložiti ako naslijediš CRUD kontroler. To je OK. Flutter delete dugme ne moraš praviti.

Ako obrišeš karticu koja ima narudžbe, a FK je Restrict, baza će odbiti. To je razlog da dobro izabereš delete behavior.

### Šta trebam provjeriti

* [ ] Nisam potrošila vrijeme na delete UI
* [ ] Ako delete postoji na API-ju, znam da može pasti zbog FK

---

# 15. GetAll / List / Search / Filter

### Općenito

`BaseReadService.GetAllAsync`:

1. `_dbContext.Set<TEntity>()` – svi redovi
2. `IncludeRelatedEntitiesAsync` – Join/Include
3. `ApplyFilters` – **ti pišeš**
4. opcionalno `Count` ako `IncludeTotalCount`
5. `OrderBy(search.SortBy)` preko Dynamic LINQ
6. `Skip/Take` za page
7. Mapster u listu Response
8. vrati `PageResult<T>` sa `Items` i `TotalCount`

**PageResult** – omotač: nije gol `List<T>`, nego objekat `{ items, totalCount }`. Flutter `SearchResult` čita baš ta dva polja.

**IQueryable** – u komentarima i u `IncludeRelatedEntitiesAsync` vidiš `IQueryable`. To je „upit koji se još nije izvršio". Možeš dodavati `.Include`, `.Where`, pa tek na kraju ide u bazu.

U `GetAllAsync` ima smjene `IEnumerable` / `IQueryable`. `ApplyFilters` u Category radi na `IEnumerable` (u memoriji, `StringComparison.OrdinalIgnoreCase`). To nije savršeno za velike tabele, ali **tako radi template**. Na ispitu radi **kao template**, ne „kako piše u knjizi o IQueryable".

### Kako prepoznati

„prikazati sve kartice trenutno prijavljenog korisnika" = GetAll + filter po useru.

### Sortiranje

`BaseSearchObject.SortBy` je string, npr. `"OrderDate desc"`. OrderService defaultuje to ako klijent ne pošalje.

### Pagination

`Page` default 1, `PageSize` default 10.

Ako Flutter zovne `get()` bez filtera, dobićeš **samo 10** kartica. Za profil možda želiš `pageSize: 50` ili `100`, kao `OrderProvider.fetchMyOrders`.

Ovo je česta zamka: „dodala sam 12 kartica, vidim samo 10".

### Kako se koristi u ovom ispitu

Override `ApplyFilters`:

* uvijek ograniči na `GetUserId()` (kao OrderService), **ili**
* filtriraj `search.UserId` ali ga na servisu namesti iz tokena.

Ne dozvoli da neko u Swaggeru upiše `userId=5` i vidi tuđe kartice.

### Šta trebam provjeriti

* [ ] Get vraća `{ items: [...], totalCount: ... }`
* [ ] Vidim samo svoje kartice
* [ ] Nisam zgubila redove zbog pageSize=10

---

# 16. GetById

### Općenito

`Find(id)`, mapiraj, vrati. Ako nema, `KeyNotFoundException` → u `BaseReadController` 404.

### Kako prepoznati

„pregled detalja", „odabirom kartice".

### Šta ako ne postoji

404 na GET. Flutter treba try/catch.

### Kako se koristi u ovom ispitu

Edit ekran može:

* dobiti objekat već iz liste (brže, manje podataka);
* ili zvati `getById(id)` da dobije i transakcije.

Ako transakcije nisu na list Response-u (ne trebaju na listi), GetById je pravo mjesto da ih dodaš u Response.

Override `GetByIdAsync` kao OrderService: Include, provjeri userId, mapiraj.

### Šta trebam provjeriti

* [ ] Tuđu karticu po id-u ne mogu otvoriti
* [ ] Nepostojeći id ne ruši API sa 500 ako možeš to srediti
* [ ] Detalj ima dovoljno podataka za formu + transakcije

---

# 17. DTO – Request i Response

### Općenito

**DTO** (Data Transfer Object) – objekat za prijenos podataka preko API-ja. Nije tabela.

U ovom projektu se zove Request / Response / SearchObject.

### Zašto ne vraćamo uvijek Entity

Entity ima:

* `PasswordHash`, `PasswordSalt`
* navigation `User.RefreshTokens`
* EF proxy stvari
* kružne reference (User → Cards → User → ...) koje JSON ne umije lijepo serijalizirati

Response ima samo što UI treba.

### Request DTO

Šta klijent **šalje**.

InsertRequest: polja forme.

UpdateRequest: polja koja se smiju mijenjati.

CheckoutRequest: `Items` + (u tvom zadatku) identifikator kartice.

Request **nema** `Id` na insertu (id dodijeli baza). Request **nema** `CreatedAt`.

### Response DTO

Šta klijent **prima**.

Ima `Id`. Ima ono za prikaz. Može imati izračunato `AvailableBalance` koje **nije** kolona – puniš ga u servisu/mapiranju.

To je dozvoljeno i pametno: zadatak kaže da se stanje računa. Response smije pokazati rezultat računice. Tabela ne smije čuvati to kao izvor istine.

### SearchObject

Nije ni insert ni response. To su filteri.

Nasljeđuje `BaseSearchObject` da dobije Page, PageSize, SortBy, IncludeTotalCount.

### Kako iz teksta prepoznati šta ide u koji DTO

| Polje | Entity | InsertRequest | UpdateRequest | Response | Search |
|-------|--------|---------------|---------------|----------|--------|
| Id | da | ne | ne (ide u URL) | da | možda |
| UserId | da | ne (iz tokena) | ne | da | interno |
| broj kartice | da | da | možda | da (možda maskiran) | ne nužno |
| CVC | da | da | možda | pažljivo | ne |
| datum isteka | da | da | da | da | ne |
| početno stanje | da | da | možda | da | ne |
| dostupno stanje | ne | ne | ne | **možda da** | ne |
| lista transakcija | ne (to su Orders) | ne | ne | **možda da na GetById** | ne |

Ova tabela nije naredba rješenja. To je način da **ti** odlučiš. Popuni je na papiru prije koda.

### Kako se koristi u ovom ispitu

Najmanje:

* InsertRequest kartice
* UpdateRequest kartice
* Response kartice
* SearchObject kartice
* proširenje CheckoutRequest
* možda proširenje OrderResponse / OrderSearchObject

### Šta trebam provjeriti

* [ ] Entity nije return type kontrolera
* [ ] Request nema hash, nema navigacije
* [ ] Imena polja ista kao na Entity da Mapster radi
* [ ] Flutter JSON ključevi su camelCase; ASP.NET po defaultu serijalizira `CardNumber` kao `cardNumber`. Dart model koristi `cardNumber` uz json_serializable.

---

# 18. Validation

### Općenito

**Validacija** = provjera da je ulaz prihvatljiv **prije** snimanja.

Zašto: da u bazu ne uđe kartica sa 2 cifre i negativnim novcem.

Gdje: u ovom projektu u **FluentValidation** klasama, koje zove `BaseCRUDService`.

### FluentValidation

Biblioteka. Pišeš klasu:

```csharp
public class CategoryInsertValidator : AbstractValidator<CategoriesInsertRequest>
{
    public CategoryInsertValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Name is required.")
            .MaximumLength(50)
            .MinimumLength(4);
    }
}
```

**RuleFor** – za koje polje.
**NotEmpty** – obavezno.
**WithMessage** – tekst koji ide klijentu.

Registracija u `Program.cs`:

```csharp
builder.Services.AddScoped<IValidator<CategoriesInsertRequest>, CategoryInsertValidator>();
```

Ako zaboraviš registraciju, aplikacija neće moći napraviti `CategoryService` jer konstruktor traži `IValidator<...>`.

### Tipična pravila

| Zadatak kaže | FluentValidation |
|--------------|------------------|
| obavezno | `NotEmpty()` |
| max dužina | `MaximumLength(n)` |
| min dužina | `MinimumLength(n)` |
| tačno N karaktera | `Length(n)` ili `Must(x => x.Length == n)` |
| samo cifre | `Matches(@"^[0-9]+$")` |
| broj >= 0 | `GreaterThanOrEqualTo(0)` |
| email | `EmailAddress()` |
| uvjetno | `.When(x => ...)` |

Generički primjer (nije copy-paste rješenje kartice, ali je tačna sintaksa):

```csharp
RuleFor(x => x.Code)
    .NotEmpty()
    .Length(12)
    .Matches("^[0-9]{12}$")
    .WithMessage("Code must be 12 digits.");
```

Za datum „ne smije biti istekao":

```csharp
RuleFor(x => x.ExpiryDate)
    .Must(d => d > DateTime.UtcNow)
    .WithMessage("Card is expired.");
```

To je **princip**. Tvoj tip datuma može biti drugačiji (samo mjesec/godina). Ako čuvaš `DateTime`, odluči da li je istek kraja mjeseca. Na ispitu je važno da istekla kartica ne prođe, ne da bude bankarski standard ISO.

### Validacija na backendu vs Flutteru

Zadatak kaže validaciju na formi **i** pravila na entitetu.

Radi oba:

* Flutter: da korisnik odmah vidi grešku (prazno polje, nije 12 cifara);
* Backend: pravi zaštita.

Ako uradiš samo Flutter, profesor u Swaggeru unese smeće i dobiješ 0 na tom dijelu.

### Custom / poslovna validacija

FluentValidation je za oblik podataka (12 cifara, >= 0).

„Nema dovoljno novca za ovaj račun" NIJE InsertValidator kartice. To je pravilo u `CheckoutAsync`, jer ovisi o **drugim tabelama** (narudžbe, total).

Tu koristiš `ClinetException`.

### Kako se koristi u ovom ispitu

Insert/Update kartice: cifre, dužine, datum, decimal >= 0.

Checkout: istekla, nedovoljno sredstava, tuđa kartica, nije odabrana.

### Šta trebam provjeriti

* [ ] Oba validatora registrovana
* [ ] Poruke na bosanskom ili engleskom – template je na engleskom; nije bitno, bitno da su jasne
* [ ] `ClinetException` za novac, ne FluentValidation na CheckoutRequest osim ako i to dodaš

---

# 19. API Endpointi

### Općenito

**Endpoint** je URL + HTTP metoda.

Primjer: `POST http://localhost:5126/Categories`

### HTTP metode u ovom templateu

| Metoda | Značenje | Bazni kontroler |
|--------|----------|-----------------|
| GET `/Controller` | lista | BaseRead |
| GET `/Controller/{id}` | jedan | BaseRead |
| POST `/Controller` | insert | BaseCRUD |
| PUT `/Controller/{id}` | update | BaseCRUD |
| DELETE `/Controller/{id}` | delete | BaseCRUD |
| POST `/Orders/Checkout` | specijalna akcija | ručno na OrdersController |

**GET** – čitaj. **POST** – stvori / pokreni akciju. **PUT** – zamijeni/izmijeni. **DELETE** – obriši.

### Routing

`[Route("[controller]")]` uzima ime klase bez sufiksa `Controller`.

`CategoriesController` → `/Categories`

Ako napraviš `PaymentCardsIB210001Controller`, Flutter endpoint mora biti tačno `PaymentCardsIB210001`.

Veliko/malo slovo: ASP.NET je obično case-insensitive, ali piši konzistentno.

### Request i response

* POST/PUT: `[FromBody]` JSON
* GET lista: `[FromQuery]` SearchObject (`?name=abc&page=1`)
* GET id: iz rute `{id}`

### Status codes

| Kod | Značenje | Kad u ovom projektu |
|-----|----------|---------------------|
| 200 | OK | Get, Update, Checkout |
| 201 | Created | deklarisano na Create, ali metoda samo `return result` (često i dalje 200) |
| 204 | No content | Delete |
| 400 | Bad request | validacija, ClinetException |
| 401 | Unauthorized | nema/istekao JWT |
| 404 | Not found | GetById KeyNotFound |
| 500 | Server error | neuhvaćen Exception |

Ne paniči ako Create nije 201. Template tako radi.

### Autorizacija

Neki kontroleri imaju `[Authorize]` (`Orders`, `ProductReviews`).

`BaseReadController` ima **zakomentarisan** `//[Authorize]`. Znači mnogi GET-ovi rade i bez tokena u templateu.

Za kartice: logično je `[Authorize]`, jer su to lični podaci. Pogledaj ProductReviewsController.

Flutter `BaseProvider.createHeaders()` već šalje `Authorization: Bearer ...`.

### Kako Command/Query dolazi do endpointa

Nema MediatR. Kontroler ima `_service` iz konstruktora. DI ubaci konkretnu klasu.

### Swagger / Scalar

Dok API radi, otvori:

* Swagger: obično `/swagger`
* Scalar: template zove `MapScalarApiReference()`

Tu testiraš backend **prije** Fluttera. To je pola ispita vremenski.

### Kako se koristi u ovom ispitu

Novi CRUD kontroler za kartice.

Postojeći `POST /Orders/Checkout` proširuješ bodyjem.

Ne pravi novi `PaymentsController` osim ako baš želiš – zadatak kaže proširiti Orders i dodati infrastrukturu za kartice.

### Šta trebam provjeriti

* [ ] Swagger vidi novi kontroler
* [ ] Login pa Authorize u Swaggeru (lock ikona, JWT)
* [ ] Flutter URL se poklapa

---

# 20. Dependency Injection

### Općenito

**Dependency Injection (DI)** – ne radiš `new CategoryService()` u kontroleru. Kažeš: „trebam `ICategoryService`". Sistem ti da instancu.

Zašto: testiranje, jedan DbContext po requestu, manje spregnut kod. Na ispitu: **zato što template tako radi i zadatak zabranjuje ručno instanciranje**.

### Gdje se registruju servisi

`eCommerce.WebAPI/Program.cs`:

```csharp
builder.Services.AddScoped<ICategoryService, CategoryService>();
builder.Services.AddScoped<IValidator<CategoriesInsertRequest>, CategoryInsertValidator>();
builder.Services.AddDbContext<ECommerceDbContext>(...);
builder.Services.AddMapster();
```

**AddScoped** – jedna instanca po HTTP requestu. Tako registruj i ti.

### Kako Handler (ovdje: Service) dobija dependency

Konstruktor:

```csharp
public CategoryService(
    ECommerceDbContext dbContext,
    IMapper mapper,
    IValidator<CategoriesInsertRequest> insertValidator,
    IValidator<CategoriesUpdateRequest> updateValidator)
    : base(dbContext, mapper, insertValidator, updateValidator)
{ }
```

Ako dodaš `IAuthenticatedUserAccessor`, dodaj parametar. DI će ga dati jer je već registrovan:

```csharp
builder.Services.AddScoped<IAuthenticatedUserAccessor, HttpAuthenticatedUserAccessor>();
```

### Kako API dobija dependency

```csharp
public CategoriesController(ICategoryService categoryService) : base(categoryService)
{ }
```

Prazan kontroler je **dobar** kontroler u ovom projektu.

### Flutter „DI"

`MultiProvider` u `main.dart`:

```dart
ChangeNotifierProvider(create: (_) => OrderProvider()),
```

Ako napraviš `PaymentCardProvider` a ne registruješ ga, `context.read<PaymentCardProvider>()` puca.

### Kako se koristi u ovom ispitu

Svaki novi servis + oba validatora + Flutter provider.

Zaboravljena registracija je jedna od top 3 grešaka.

### Šta trebam provjeriti

* [ ] `Program.cs` ima AddScoped za servis i validatore
* [ ] `main.dart` ima ChangeNotifierProvider
* [ ] Aplikacija se pokreće (DI error se vidi čim udariš endpoint)

---

# 21. Frontend – Flutter mobile

Ovaj ispit **traži frontend**. To je **ecommerce_mobile**, ne desktop.

## 21.1. Koje stranice treba napraviti / mijenjati

| Ekran | Postoji? | Šta radiš |
|-------|----------|-----------|
| `profile_screen.dart` | da | sekcija „Moje kartice", lista, dugme dodaj, tap za edit |
| `payment_card_details.dart` | **ne, ti praviš** | create + edit + transakcije |
| `cart_list_screen.dart` | da | odabir kartice prije Place order |
| `main.dart` | da | registruj provider |
| novi tab u bottom bar | ne treba | zadatak kaže na profilu |

## 21.2. Modeli

Folder `lib/models/`.

Obrazac `user.dart`:

* `import 'package:json_annotation/json_annotation.dart';`
* `part 'user.g.dart';`
* `@JsonSerializable()`
* `factory fromJson` + `toJson`

Nakon pisanja modela, u terminalu **u folderu `ecommerce_mobile`**:

```
dart run build_runner build --delete-conflicting-outputs
```

To napravi `ime.g.dart`. Bez toga projekat ne kompajlira `part` fajl.

Polja u Dartu su camelCase i nullable (`int?`, `String?`) kao u postojećim modelima.

Decimal sa API-ja dolazi kao broj. U Dartu često `double?` (pogledaj `product.dart` → `price`). Možeš i `num`. Budi konzistentna.

## 21.3. Provider (servis na frontendu)

`BaseProvider<T>` već ima:

* `get(filter:)` → GET lista
* `getById(id)`
* `insert(request)`
* `update(id, request)`
* `remove(id)`

Tvoj provider:

```dart
class NestoProvider extends BaseProvider<Nesto> {
  NestoProvider() : super("ImeKontrolera");

  @override
  Nesto fromJson(data) => Nesto.fromJson(data);
}
```

`endpoint` mora odgovarati ruti kontrolera.

Za checkout, `OrderProvider` već ima ručni `http.post` na `Orders/Checkout`. Kad proširiš body, **ovdje** dodaješ id kartice u JSON, ne u BaseProvider.

## 21.4. Forme i validacija

Dva stila u templateu:

1. **Prosti TextEditingController** – `AddReviewScreen`, login. Dovoljno za ispit.
2. **flutter_form_builder** – desktop kategorije. Možeš koristiti i na mobile (paket već jeste u `pubspec.yaml`).

Za ispit je važno:

* polja: broj, CVC, datum isteka, početno stanje;
* lokalna provjera prije `insert`;
* `alertBox` kad API vrati grešku.

Datum isteka na mobilnom: možeš `TextField` (MM/YYYY) ili date picker. Šta god da izabereš, JSON mora odgovarati backend Request tipu.

## 21.5. HTTP pozivi i greške

`validateResponse` baca `ApiClientException` sa porukom iz API-ja (`message` polje koje ExceptionFilter šalje).

`CartListScreen` već hvata:

```dart
on ApiClientException catch (e) {
  alertBox(context, 'Order could not be placed', e.message);
}
```

Isti obrazac koristi za karticu i checkout. Zato backend poruka „Insufficient funds" mora biti ljudska.

## 21.6. Prikaz liste

Na profilu, nakon `initData` za usera, učitaj kartice.

Prazna lista: `Text('Nemate kartica')`, ne crveni error.

Za svaku karticu „osnovni podaci": npr. posljednje 4 cifre, istek, početno stanje. CVC na listi nije potreban.

## 21.7. Create / Edit / nijanse

Jedan widget, kao CategoryDetails:

* `final PaymentCard? card;`
* null → naslov Dodaj, dugme insert, nema transakcija
* not null → naslov Uredi, insert vs update, lista transakcija ispod

Nakon uspjeha:

```dart
Navigator.pop(context, 'reload');
```

Na profilu, kao edit profile.

## 21.8. Search / filter / pagination

Za profil: filter po useru je backend problem. Frontend šalje Bearer token. Ako backend uzima userId iz tokena, Flutter može `get(filter: {'page': 1, 'pageSize': 50, 'includeTotalCount': true})`.

## 21.9. Checkout UI

U `_checkout()` trenutno:

```dart
final body = jsonEncode({'items': items});
```

Trebaš:

1. prije toga imati odabranu karticu;
2. ako nije odabrana, `alertBox` i return;
3. u body staviti i id kartice (ime propertyja = ime na `CheckoutRequest`).

Gdje UI: iznad `Place order` dropdown ili lista radio dugmadi. Učitaj kartice istim providerom.

Ako korisnik nema kartica, poruka: dodaj karticu u profilu. Nemoj crashati.

## 21.10. Šta NIJE dio ovog ispita

* desktop admin za kartice
* novi login
* dizajn, animacije, teme
* brisanje kartice
* 3D secure

Ne troši vrijeme na lijepu karticu sa gradientom. Tabela/lista + forma prolaze.

## 21.11. Pokretanje Fluttera

API mora biti upaljen.

Mobile emulator Android vidi host mašinu kao `10.0.2.2`, zato `BaseProvider` default:

```
http://10.0.2.2:5126/
```

To odgovara `launchSettings.json` portu **5126**.

Ako API nije na 5126, Flutter neće raditi. Provjeri.

Windows desktop `flutter run -d windows` koristi `localhost`, ali ovaj ispit je mobile profil – radi u emulatoru ili Chrome ako imate web, ali default URL je Android emulator.

Na ispitu: pitaj asistenta koji device koristite. URL je najčešći uzrok „frontend ne radi".

## 21.12. Šta trebam provjeriti

* [ ] `build_runner` urađen
* [ ] provider u MultiProvider
* [ ] profil pokazuje kartice
* [ ] add + edit rade
* [ ] transakcije se vide nakon plaćanja
* [ ] checkout traži karticu
* [ ] poruka kad nema novca

---

# 22. Kako povezati sve dijelove

Ovo je slika koju trebaš umjeti ispričati naglas.

## 22.1. Cijeli tok Create kartice

```
Korisnik na ProfileScreen
    tap "Dodaj karticu"
        payment_card_details.dart (prazna forma)
            korisnik unese podatke
            Flutter validacija
            PaymentCardProvider.insert({...})
                HTTP POST /PaymentCardsIBXXXX
                    Authorization: Bearer jwt
                        PaymentCardsController.Create
                            IPaymentCardService.InsertAsync
                                InsertValidator
                                MapInsertRequestToEntity
                                UserId iz IAuthenticatedUserAccessor
                                DbContext.Add
                                SQL INSERT
                            Response JSON
                Dart fromJson
            Navigator.pop('reload')
        ProfileScreen ponovo GET listu
            korisnik vidi novu karticu
```

## 22.2. Cijeli tok Place order

```
Korisnik u korpi
    bira karticu
    tap Place order
        OrderProvider.checkout(items, cardId)
            POST /Orders/Checkout
                OrderService.CheckoutAsync
                    uzmi userId iz tokena
                    nađi karticu, provjeri vlasnika
                    provjeri istek
                    izračunaj: početno - suma narudžbi te kartice
                    usporedi sa total korpe
                    ako ne valja → ClinetException
                    ako valja → kreiraj Order sa FK kartice
                    SaveChanges
                OrderResponse
        korpa se očisti
        OrderDetailScreen
```

Ako kasnije otvori karticu:

```
GET kartica by id
    response uključuje transakcije
        svaka transakcija = narudžba (datum, iznos)
```

## 22.3. Šta se dešava kad validacija padne

```
FluentValidation fail
    ValidationException
        ExceptionFilter
            400 { message, errors: { CardNumber: ["..."] } }
                Flutter ApiClientException
                    alertBox
```

```
Nema dovoljno novca
    ClinetException("...")
        ExceptionFilter
            400 { message, errors: { clientError: ["..."] } }
                ista Flutter grana
```

Zato **ne** bacaj `InvalidOperationException` za poslovno pravilo – UserService to radi za email, i to može postati 500. Za korisničke greške u novijem kodu koristi `ClinetException` (OrderService, ProductReviewService).

## 22.4. Gdje se stanje „mijenja" a gdje ne

Korisnik misli: „platila sam 50 KM, na kartici je sad manje".

U bazi:

* `InitialBalance` ostaje 200
* nova `Order` ima `TotalAmount = 50` i FK na karticu
* sljedeći put dostupno = 200 - 50 - ...

Ako u UI prikažeš `InitialBalance` kao da je trenutno, zbunićeš i sebe i profesora. Ako prikazuješ dostupno, moraš ga izračunati.

---

# 23. Redoslijed rada na ispitu

Ovo nije RS1 redoslijed (Entity → Command → Query → Angular). Ovo je redoslijed **ovog** templatea i **ovog** zadatka.

## 23.1. Prije koda (15–20 min)

1. Pročitaj cijeli PDF dva puta. Druga čitanja su za „šta sam preskočila".
2. Na papiru izvuci:
   * novi entitet + polja + validacije;
   * relacije;
   * šta se računa a šta se čuva;
   * koji Flutter ekrani;
   * koji postojeći fajlovi se SAMO proširuju (`Order`, `CheckoutRequest`, `OrderService`, `profile_screen`, `cart_list_screen`).
3. Otvori template i nađi referentne fajlove: Category, OrderService, ProfileScreen, CartListScreen, ProductReviewService.
4. Connection string + `Update-Database` + login `customer1`.
5. Swagger radi? Flutter login radi? Ako ne, **ne piši feature**. Prvo okruženje.

## 23.2. Backend Database sloj

6. Entity kartice (ime sa indeksom).
7. Navigation na `User`.
8. FK na `Order` (nullable).
9. `CreateConfiguration` za relacije.
10. `DbSet`.
11. Build projekta.
12. **Jedna** migracija za sve Database izmjene.
13. `Update-Database`.
14. SSMS: vidiš li tabelu i kolonu na Orders?

Ne idi na Flutter dok tabela ne postoji.

## 23.3. Backend Application sloj (Model + Service + Validator)

15. InsertRequest, UpdateRequest, Response, SearchObject.
16. InsertValidator, UpdateValidator.
17. Interface + Service (`ApplyFilters`, userId, možda GetById sa transakcijama).
18. `Program.cs` registracija + Mapster ako treba.
19. Controller `: BaseCRUDController` + `[Authorize]`.
20. Proširi `CheckoutRequest`.
21. Proširi `CheckoutAsync`: pravila + upis FK + formula.
22. Test u Swaggeru:
    * login (`POST /Access/Login`) → token;
    * insert kartice;
    * get lista;
    * get by id;
    * update;
    * checkout bez kartice / sa isteklom / sa premalo novca / sa dovoljno.

**Tek sada frontend.**

## 23.4. Flutter

23. Model + build_runner.
24. Provider + MultiProvider.
25. Sekcija na profilu + lista.
26. `payment_card_details.dart` insert.
27. Reload liste.
28. Edit + transakcije.
29. Cart odabir kartice + checkout body.
30. Ručni test scenarija iz PDF-a (200 − 50 − 30, račun 130).

## 23.5. Predaja

31. Prođi checklist (sekcija 25).
32. Clean Solution.
33. `flutter clean` u mobile.
34. Zip = indeks.
35. FTP Upload/RSII.

## 23.6. Šta NE raditi ovim redom

* Ne počinji od Flutter forme. Backend te neće imati.
* Ne pravi migraciju prije nego što znaš FK na Order – ili ćeš imati dvije migracije (nije greška, ali gubi vrijeme).
* Ne diraj ProductStateMachine, Docker, CryptoService, desktop, seed, JWT konfiguraciju.
* Ne prepisuj cijeli `BaseCRUDService`.

## 23.7. Ako zapneš i imaš 40 minuta

Prioritet bodova (gruba procjena, nije zvanična):

1. Entitet + migracija + CRUD API + validacije
2. Order FK + checkout pravila (formula)
3. Flutter lista + dodaj
4. Flutter checkout odabir
5. Flutter edit + transakcije

Bolje radni backend i polovičan UI nego lijepa forma koja zove nepostojeći API.

---

# 24. Kako samostalno riješiti ovaj zadatak

Za svaku vrstu rečenice iz **ovog** PDF-a: algoritam razmišljanja. Bez gotovog koda.

---

### Ako piše: „dodati entitet PaymentCardBrojIndeksa"

1. „Entitet" → klasa u `Database/`.
2. Ime mora sadržavati **moj** indeks. Otvori indeks i prepiši tačno.
3. Otvaram `Category.cs` i `ProductReview.cs` kao kalup.
4. Pitam: koja polja su u sljedećoj rečenici?
5. Pišem papirnatu tabelu polja.
6. Tek onda klasu.

---

### Ako piše: „jedan korisnik može imati više kartica"

1. „Jedan … više" → 1:N.
2. FK na kartici, ne na useru. (Da li je UserId lista? Ne.)
3. Na `User` dodajem `ICollection<...>`.
4. U konfiguraciji `HasOne.WithMany.HasForeignKey`.
5. DeleteBehavior: razmisli, pogledaj slične (UserRole Cascade vs Review Restrict).

---

### Ako piše: „broj kartice mora sadržavati 12 cifara … CVC 3 … samo cifre"

1. To NIJE nova tabela.
2. To NIJE enum.
3. To JE validacija.
4. Gdje? `Validators/*InsertValidator` i UpdateValidator.
5. Sintaksa: `Matches`, `Length`.
6. Na entitetu MaxLength da baza ne primi duži string.
7. Na Flutteru `if (value.length != 12)` da korisnik vidi odmah.
8. Test: `"123"` mora pasti, `"12ab"` mora pasti, razmak mora pasti.

---

### Ako piše: „datum isteka mora biti validan i ne smije predstavljati već isteklu karticu"

1. „Validan" → može se parsirati kao datum.
2. „Nije istekao" → usporedba sa `DateTime.UtcNow` (ili krajem mjeseca).
3. Ista provjera na **insert** i na **checkout**. Zašto? Kartica je mogla biti unešena dok je bila validna, pa istekla do plaćanja. Zadatak kod plaćanja ponovo kaže „samo ako nije istekla".

---

### Ako piše: „početno stanje mora biti decimalna vrijednost veća ili jednaka 0"

1. Tip: `decimal`, ne `int` (može 100.50 KM).
2. EF: `decimal(18,2)` kao Price.
3. Validator: `GreaterThanOrEqualTo(0)`.
4. Flutter: `double.tryParse`, provjera `>= 0`.

---

### Ako piše: „potrebno je dodati model, baznu tabelu, migraciju, DTO, servis, kontroler i validacije"

To je **checklist kompletnog CRUD lanca**. Odmah napiši na papir 11 stavki:

1. Entity
2. DbSet
3. Configuration
4. Migration
5. Request insert
6. Request update
7. Response
8. Search
9. 2 validatora
10. Interface + Service
11. Controller + Program.cs

Ako ijedna fali, lanac je prekinut. Najčešće fali Program.cs ili SearchObject (GetAll ne kompajlira jer BaseSearchObject constraint).

---

### Ako piše: „u okviru profila korisnika"

1. Koji projekat? Mobile. `ProfileScreen`.
2. Nije `user_details_screen` na desktopu (to je admin).
3. Sekcija unutar postojećeg Column-a, ispod profil info ili unutar menija – zadatak kaže na formi profila.
4. „Trenutno prijavljen" → Auth token već postoji. Backend uzima Id iz JWT-a. Flutter šalje Bearer.

Kako ProfileScreen već zna user id:

```dart
int.tryParse(AuthProvider.accessTokenDecoded?['Id'] ?? '0')
```

To je za `Users/id`. Za kartice bolje da backend ignoriše klijentski userId i uzme token. Sigurnije i manje koda na frontu.

---

### Ako piše: „dugmić Dodaj karticu … payment_card_details.dart"

1. Novi fajl, **to ime**.
2. `Navigator.push` + `MaterialPageRoute`.
3. Kalup: `AddReviewScreen` (jednostavniji) ili desktop CategoryDetails (create/edit).
4. Nakon inserta `pop` sa signalom za reload.

---

### Ako piše: „kartice … kao opcija prilikom plaćanja narudžbe"

1. Gdje se plaća? Traži `checkout` u projektu. Naći ćeš `CartListScreen` i `OrderService.CheckoutAsync`.
2. Ne pravi novi PaymentScreen osim ako baš želiš. Zadatak kaže prilikom plaćanja – to je postojeći Place order.
3. UI: dropdown kartica.
4. API: novo polje na CheckoutRequest.
5. Ako zaboraviš backend, UI izgleda gotovo a plaćanje ne veže karticu.

---

### Ako piše: „trenutno dostupno stanje se ne čuva, nego se računa"

Algoritam:

1. Ima li u tekstu riječ „ne čuva se"? → **zabrana kolone**.
2. Formula je data. Prepiši je na papir s primjerom 200, 50, 30, 130.
3. „Transakcije" u sljedećem zahtjevu su vezane za Order. Znači suma ide po Orders gdje je FK kartice = ova kartica.
4. Gdje računati? Metoda u servisu, npr. prije checkouta i pri GetById.
5. Test case iz PDF-a je **obavezan**. Ako tvoj kod na 130 KM prođe, pala si taj zahtjev.

Pseudo (namjerno nije C# rješenje):

```
dostupno = kartica.PočetnoStanje
za svaku narudžbu te kartice koja se računa kao uspješna:
    dostupno = dostupno - narudžba.TotalAmount
ako dostupno < iznosNovogRačuna: odbij
```

---

### Ako piše: „kartica ne smije otići u minus" + „jasna poruka"

1. `if (available < total) throw new ClinetException("...");`
2. Poruka neka spomene iznos. Korisnik treba razumjeti.
3. Flutter već ima `ApiClientException`.
4. Ne raditi samo `if` na frontu. Swagger test je profesorov test.

---

### Ako piše: „proširiti entitet Orders … korištene kartice"

1. Otvori `Order.cs`.
2. Već ima PaymentTransactionId – to je string, nije FK na tvoju tabelu.
3. Dodaj FK + navigation.
4. Nullable.
5. Checkout mora to polje popuniti. Inače je kolona uvijek null i formula je uvijek jednak početnom stanju. To je tiha padajuća greška: sve kartice „imaju para" zauvijek.

---

### Ako piše: „editovanje … ispod … transakcije (datum i iznos)"

1. Isti details ekran, drugi mod.
2. Transakcija = narudžba te kartice.
3. Prikaz: datum (`orderDate`) i iznos (`totalAmount`).
4. Ne trebaš novi Transaction entity.
5. Ako GetAll kartica ne vraća narudžbe (ne treba na listi), GetById ili poseban filter Orders.

---

### Ako piše: „poštovati strukturu … nikako instanciranje DbContext na kontroleru"

Svaki put kad pišeš kod u Controlleru, pitaj: „Da li CategoryController ovako radi?" Ako CategoryController ima 10 linija, ni tvoj ne treba 80 linija LINQ-a.

---

### Ako piše: Clean Solution, flutter clean, zip indeks, FTP

Ovo nije tehnički feature, ali je **zahtjev**. Ostavi 10 minuta. Zip bez migracija = profesor ne može podići bazu.

---

# 25. Checklist prije predaje ispita

Prilagođeno **ovom** ispitu i **ovom** templateu. Idi redom.

## 25.1. Priprema

* [ ] Baza = moj broj indeksa, ne `eCommerce`
* [ ] `appsettings.Development.json` pokazuje ispitni SQL server
* [ ] `Update-Database` prošao
* [ ] Login `customer1` / `Test123` radi u Flutteru

## 25.2. Database / Domain (u Services/Database)

* [ ] Entitet kartice postoji
* [ ] Ime entiteta sadrži broj indeksa
* [ ] Polja: veza na usera, broj, CVC, istek, početno stanje
* [ ] Početno stanje je `decimal(18,2)`
* [ ] Nema kolone „trenutno stanje"
* [ ] `User` ima kolekciju kartica (ako koristiš navigation)
* [ ] `Order` ima FK na karticu (nullable)
* [ ] Relacije u `CreateConfiguration` ako treba Restrict/Cascade
* [ ] `DbSet` dodan

## 25.3. Migracija

* [ ] `Add-Migration` urađen
* [ ] Fajl postoji u `Migrations/`
* [ ] `Update-Database` nakon moje migracije
* [ ] U SSMS-u vidim tabelu kartica
* [ ] U SSMS-u vidim novu kolonu na Orders
* [ ] Migracije su u zipu

## 25.4. Model (DTO)

* [ ] InsertRequest
* [ ] UpdateRequest
* [ ] Response
* [ ] SearchObject nasljeđuje `BaseSearchObject`
* [ ] CheckoutRequest zna za karticu
* [ ] Response po potrebi ima dostupno stanje i/ili transakcije – ali to je izračunato

## 25.5. Validacija

* [ ] InsertValidator: 12 cifara, 3 cifre, samo cifre, istek, >= 0
* [ ] UpdateValidator: ista pravila po potrebi
* [ ] Oba registrovana u `Program.cs`
* [ ] Loš POST vraća 400

## 25.6. Servis

* [ ] Interface nasljeđuje `IBaseCRUDService<...>`
* [ ] Klasa nasljeđuje `BaseCRUDService<...>`
* [ ] `ApplyFilters` implementiran (mora, metoda je abstract)
* [ ] Lista je samo za prijavljenog usera
* [ ] Insert veže userId iz tokena
* [ ] Nema `new ECommerceDbContext`

## 25.7. Order / plaćanje

* [ ] `CheckoutAsync` prima karticu
* [ ] Provjera vlasništva kartice
* [ ] Provjera isteka
* [ ] Formula dostupnog stanja
* [ ] 130 KM na 120 KM dostupnih → 400 + poruka
* [ ] 120 KM ili manje → narudžba se kreira
* [ ] Nova narudžba ima FK kartice
* [ ] Početno stanje u tabeli kartica se **nije** smanjilo
* [ ] Koristi se `ClinetException`, ne 500

## 25.8. API

* [ ] Controller nasljeđuje `BaseCRUDController`
* [ ] Routing ime = Flutter endpoint
* [ ] Servis registrovan `AddScoped`
* [ ] `[Authorize]` razumno postavljen
* [ ] Swagger: CRUD + checkout scenariji prošli

## 25.9. Flutter mobile

* [ ] Model + `.g.dart` (build_runner)
* [ ] Provider registrovan u `main.dart`
* [ ] Profil ima sekciju kartica
* [ ] Lista kartica prijavljenog usera
* [ ] Dugme Dodaj
* [ ] Fajl `payment_card_details.dart` postoji
* [ ] Validacija na formi
* [ ] Nakon dodavanja reload liste
* [ ] Tap → edit
* [ ] Ispod edit forme transakcije (datum, iznos)
* [ ] Korpa: odabir kartice
* [ ] Place order šalje id kartice
* [ ] Poruka kad nema sredstava
* [ ] Desktop nisam nepotrebno dirala

## 25.10. Predaja

* [ ] Nisam ostavila `TODO` koji lomi build
* [ ] Solution se builda
* [ ] Clean Solution
* [ ] `flutter clean`
* [ ] Zip ime = indeks
* [ ] FTP Upload/RSII

---

# 26. Najčešće greške

Za svaku: šta je, zašto nastaje, kako prepoznati, kako ispraviti, kako spriječiti.

---

### Greška 1: Tražiš CQRS foldere

**Šta:** Praviš `Commands/CreatePaymentCard`.

**Zašto:** RS1 navika.

**Kako prepoznati:** U solutionu nema `Market.Application`.

**Kako ispraviti:** Obriši to. Idi na Service + Controller.

**Spriječiti:** Prva stvar na ispitu: otvori `CategoryService` i kopiraj **taj** obrazac.

---

### Greška 2: Entitet bez broja indeksa

**Šta:** Klasa se zove `PaymentCard`.

**Zašto:** Čitaš zadatak dijagonalno.

**Kako prepoznati:** PDF kaže `PaymentCardBrojIndeksa`.

**Kako ispraviti:** Rename klase, DbSet, tabele (nova migracija ako je stara već primijenjena).

**Spriječiti:** Ime klase napiši na papir čim pročitaš zahtjev 2.

---

### Greška 3: Kolona CurrentBalance

**Šta:** Nakon plaćanja radiš `card.Balance -= total`.

**Zašto:** Tako je „lakše".

**Kako prepoznati:** Zadatak kaže da se ne čuva, plus primjer 200-50-30.

**Kako ispraviti:** Vrati početno stanje, računaj iz Orders.

**Spriječiti:** U checklisti stavka „nema kolone trenutno stanje".

---

### Greška 4: Checkout ne upisuje FK kartice

**Šta:** Formula uvijek vraća početno stanje. Minus nikad ne radi. Sve narudžbe prolaze.

**Zašto:** Proširila si Order, ali zaboravila `CheckoutAsync`.

**Kako prepoznati:** U SSMS-u Order.PaymentCardId je NULL. Drugo plaćanje i dalje misli da ima 200 KM.

**Kako ispraviti:** Pri `new Order { ... }` postavi FK.

**Spriječiti:** Nakon prvog checkouta odmah SELECT u SSMS.

---

### Greška 5: Validacija samo na Flutteru

**Šta:** Forma ne da 11 cifara, ali Swagger da.

**Zašto:** „Frontend je ionako tražen".

**Kako prepoznati:** POST iz Swaggera sa `"cardNumber":"1"` vrati 200.

**Kako ispraviti:** FluentValidation.

**Spriječiti:** Svako pravilo iz PDF-a testiraj u Swaggeru.

---

### Greška 6: Zaboravljen Program.cs

**Šta:** Runtime: Unable to resolve service for type `IPaymentCardService`.

**Zašto:** Kopirala si klase, ne registraciju.

**Kako prepoznati:** API padne čim udariš endpoint. `CategoryService` je registrovan oko linije 67 u `Program.cs` – tamo dodaješ i svoje.

**Kako ispraviti:** `AddScoped` za servis i oba validatora.

**Spriječiti:** Checklist „DI".

---

### Greška 7: ApplyFilters nije override-an

**Šta:** Projekat se ne kompajlira. `BaseReadService` ima `abstract ApplyFilters`.

**Zašto:** Napravila si prazan servis.

**Kako prepoznati:** Compile error „does not implement inherited abstract member".

**Kako ispraviti:** Kopiraj tijelo iz CategoryService i prilagodi filtere.

---

### Greška 8: SearchObject ne nasljeđuje BaseSearchObject

**Šta:** Constraint `where TSearch : BaseSearchObject` puca.

**Zašto:** Napravila si praznu klasu.

**Kako ispraviti:** `: BaseSearchObject`

---

### Greška 9: DbContext u kontroleru

**Šta:** `new ECommerceDbContext` ili ubacivanje DbContext direktno u kontroler i LINQ tamo.

**Zašto:** „Brže je".

**Kako prepoznati:** PDF to zabranjuje. Profesor to gleda.

**Kako ispraviti:** Premjesti u servis.

---

### Greška 10: pageSize 10 sakrije kartice

**Šta:** Imaš 11 kartica, vidiš 10.

**Zašto:** Default u `BaseSearchObject`.

**Kako ispraviti:** U GET-u pošalji veći pageSize.

---

### Greška 11: Flutter endpoint ne odgovara kontroleru

**Šta:** 404 na `PaymentCards`, a kontroler je `PaymentCardsIB210001`.

**Kako ispraviti:** `super("PaymentCardsIB210001")`

**Spriječiti:** Ime kontrolera i string u provideru gledaj jedno pored drugog.

---

### Greška 12: Nisi pokrenula build_runner

**Šta:** `user.g.dart` analogon ne postoji, `part` error.

**Kako ispraviti:** `dart run build_runner build --delete-conflicting-outputs`

---

### Greška 13: Provider nije u MultiProvider

**Šta:** `Could not find the correct Provider`.

**Kako ispraviti:** `main.dart` lista providera.

---

### Greška 14: Radiš desktop umjesto mobile

**Šta:** Lijepa admin tabela kartica, profil kupca prazan.

**Zašto:** Desktop ima `user_list` / details, izgleda „više kao CRUD".

**Kako prepoznati:** Zadatak: profil korisnika, korpa, plaćanje računa.

---

### Greška 15: ClinetException vs Exception

**Šta:** Korisnik vidi „Server side error, please check logs." umjesto „nemate dovoljno sredstava".

**Zašto:** ExceptionFilter za običan Exception šalje 500 i skriva poruku.

**Kako ispraviti:** `throw new ClinetException("...")`.

Napomena: klasa se zove `ClinetException` (typo Client). Koristi to ime.

---

### Greška 16: Tuđe kartice

**Šta:** U SearchObject ima UserId, a klijent ga može promijeniti.

**Kako ispraviti:** U servisu uvijek `_userAccessor.GetUserId()` kao OrderService.

---

### Greška 17: double za novac u C# entitetu

**Šta:** `public double InitialBalance`.

**Kako ispraviti:** `decimal` + Column TypeName. U Dartu je double OK.

---

### Greška 18: Migracija na krivu bazu

**Šta:** Tabele nastanu u `eCommerce` na localhost:1435 umjesto na ispitnom serveru.

**Kako prepoznati:** SSMS na ispitnom serveru prazan, lokalni Docker pun.

**Kako ispraviti:** Connection string, ponovo Update-Database. Na ispitu **mora** ispitni server.

---

### Greška 19: Jedna migracija po slovu

**Šta:** 7 migracija jer si 7 puta mijenjala ime polja.

**Zašto:** Add-Migration nakon svakog Ctrl+S.

**Kako ispraviti:** Ako migracija još NIJE na bazi, možeš je ukloniti pažljivo i napraviti jednu čistu. Ako JE na bazi, napravi novu. Na ispitu: prvo dovrši entity, onda JEDNOM migriraj.

Nemoj `Remove-Migration` ako nisi sigurna. Sigurnije: nova migracija.

---

### Greška 20: Zaboraviš zipati migracije / flutter clean predaja u zadnjoj minuti

**Šta:** Upload kasni, zip ogroman ili nekompletan.

**Spriječiti:** alarm 15 min prije kraja samo za predaju.

---

### Greška 21: Include zaboravljen, transakcije prazne

**Šta:** GetById ne radi `.Include` narudžbi, lista u UI uvijek prazna iako checkout radi.

**Kako ispraviti:** Override GetById/IncludeRelatedEntitiesAsync kao OrderService.

---

### Greška 22: Datum isteka u krivom formatu

**Šta:** Backend očekuje `2027-01-31T00:00:00Z`, Flutter šalje `"01/27"`.

**Kako prepoznati:** 400 parse error ili 0 na checkoutu.

**Kako ispraviti:** Dogovori format. Ako je `DateTime` na Requestu, šalji ISO 8601. Ako želiš MM/YYYY, na backendu to budu dva int-a ili string koji ti parsiraš.

---

# 27. Šta trebam naučiti napamet, a šta razumjeti

## Naučiti napamet

* Mapa foldera (sekcija 4.8)
* Category lanac: Entity, Request, Response, Search, Validator, Service, Controller, Program.cs
* PMC: default project Services, `Add-Migration`, `Update-Database`
* `throw new ClinetException("poruka");`
* `[Column(TypeName = "decimal(18,2)")]`
* Flutter: `BaseProvider`, `MultiProvider`, `build_runner` komanda
* Default API port `5126`, Android `10.0.2.2`
* Login aplikacije `customer1` / `Test123`
* Ime fajla `payment_card_details.dart`
* ExceptionFilter: validacija i ClinetException → 400
* `IAuthenticatedUserAccessor.GetUserId()`
* Clean Solution + `flutter clean` + zip = indeks

## Razumjeti

* Zašto Entity nije Response
* Zašto Controller ne dira DbContext
* Zašto Insert ide kroz servis
* Zašto 1:N stavlja FK na stranu N
* Zašto se stanje kartice računa
* Zašto checkout mora upisati FK inače formula laže
* Zašto backend validacija mora postojati i kad Flutter validira
* Kako JWT user id štiti tuđe kartice
* Kako `PageResult.items` odgovara Flutter `SearchResult`
* Kako pročitati bilo koji budući RSII PDF ovim rječnikom (sekcija 3)

Ako razumiješ donju listu, gornju možeš pogledati u templateu. Ako naučiš samo gornju, sljedeći ispit sa „ magacinom" umjesto „karticom" te zbuni.

---

# 28. Mini primjeri za vježbu

Nisu ovaj ispit. Cilj: da vježbaš **prepoznavanje**, ne da dobiješ rješenje.

Uradi na papiru. Ako želiš kod, radi na **kopiji** projekta, ne na ispitnom zipu.

---

## Vježba A: „Korisnik može kreirati projekat."

Na papiru odgovori:

1. Koji Entity?
2. Koja polja nisu navedena, pa ih ne izmišljaš?
3. Koji Request?
4. Koji Service bazni tip – CRUD ili samo Read?
5. Treba li Flutter desktop ili mobile? (nije rečeno – na pravom ispitu BIće rečeno; ovdje zapiši da mora biti rečeno)
6. Koji postojeći fajl je kalup?

Ne piši klasu dok ne odgovoriš.

---

## Vježba B: „Jedan projekat ima više zadataka. Zadatak ima naslov i rok."

1. Relacija?
2. Gdje je FK?
3. Treba li spojna tabela?
4. Treba li nova migracija ako Project već postoji?

---

## Vježba C: „Cijena mora biti veća od 0. Šifra tačno 5 cifara."

1. Entity polje tip?
2. Validator pravila?
3. Da li je to enum?

---

## Vježba D: „Popust se ne čuva, računa se 10% od cijene."

1. Kolona Popust? Da ili ne?
2. Gdje smije stajati izračunata vrijednost?
3. Koji postojeći primjer u templateu je sličan? (`OrderItem.Total` je `[NotMapped]`; dostupno stanje kartice na ovom ispitu je ista ideja)

---

## Vježba E: „Na korpi korisnik bira adresu dostave iz svojih adresa."

Ovo je slično ovom ispitu. Na papiru mapiraj:

* novi Entity Address?
* 1:N User-Address
* checkout proširiti AddressId
* Flutter cart dropdown
* ne dirati desktop

Usporedi sa karticama. Ako vidiš isti oblik, naučila si princip.

---

## Vježba F: Čitanje lažnog PDF pasusa

> Omogućiti brisanje recenzije samo autoru. Admin vidi sve recenzije.

1. Novi entity? (ne – ProductReview postoji)
2. Gdje logika? (ProductReviewService već ima Admin vs user – otvori i pročitaj)
3. Šta bi override-ala?

Ova vježba uči: **prvo potraži da li već postoji**.

---

## Vježba G: Brojevi iz ovog ispita bez koda

Početno 200. Transakcije 50 i 30. Račun 120. Prolazi?

Račun 120.01? 

Ako nisi sigurna za `decimal` usporedbu, na papiru: dostupno 120, račun mora biti `<= 120`. Jednako je dozvoljeno („ne smije u minus", 120-120=0 nije minus).

---

## Vježba H: Imenovanje

Indeks IB239999. Kako se zove:

* entity?
* controller?
* Flutter endpoint string?
* tabela (konvencija EF)?

Ne moraš pogoditi tačno ime tabele, ali controller i endpoint MORAJU biti isti.

---

# 29. Cheat Sheet

Printaj ili imaj otvoreno zadnjih 10 minuta prije ispita.

## 29.1. Kad šta

| Treba mi | Kad u tekstu | Gdje u templateu |
|----------|--------------|------------------|
| Entity | imenica koju pamtimo | `Services/Database/` |
| Enum | zatvorena lista A/B/C, ne tabela | pored entiteta, kao `OrderStatus` |
| Property | „sadrži X", „proširiti Y" | polje na klasi + često Request/Response |
| Relacija 1:N | „jedan ima više" | FK na N + ICollection na 1 |
| Relacija N:M | „više prema više" | spojna tabela kao `ProductCategory` |
| EF konfiguracija | relacija, delete, specijalni slučaj | `eCommerceConfiguration.cs` |
| Data Annotations | required, length, decimal | na Entityju |
| DbSet | novi entity | `eCommerceDbContext.cs` |
| Migracija | bilo šta na tabeli | PMC, project Services |
| Insert | dodati, unijeti, create | Request + Validator + naslijeđeni POST |
| Update | edit, izmijeniti | UpdateRequest + PUT |
| Delete | obrisati | već u BaseCRUD; ovaj ispit ne traži UI |
| GetAll | prikazati sve, lista | SearchObject + ApplyFilters |
| GetById | detalji | GET `{id}` |
| DTO Request | šta forma šalje | `Model/Requests/` |
| DTO Response | šta UI prikazuje | `Model/Responses/` |
| SearchObject | filter, page | `Model/SearchObjects/` |
| Validator | mora, ne smije, tačno N cifara | `Services/Validators/` |
| ClinetException | poslovno pravilo, poruka korisniku | u servisu |
| Service | logika | `*Service.cs` : BaseCRUDService |
| Controller | HTTP | `: BaseCRUDController` |
| DI | da bi se uopšte pokrenulo | `Program.cs` AddScoped |
| Flutter model | JSON | `lib/models/` + build_runner |
| Flutter provider | HTTP | `lib/providers/` + MultiProvider |
| Flutter screen | ekran, forma, profil | `lib/screens/` |
| Computed vrijednost | „ne čuva se, računa se" | servis / `[NotMapped]` / Response polje |

## 29.2. HTTP

| Metoda | Kad |
|--------|-----|
| GET | čitanje liste ili detalja |
| POST | insert ili akcija (`/Checkout`) |
| PUT | update po id |
| DELETE | brisanje po id |

## 29.3. Komande

```
Update-Database
Add-Migration ImeMigracije
```

Default project: `eCommerce.Services`. Startup: `eCommerce.WebAPI`.

```
dart run build_runner build --delete-conflicting-outputs
flutter clean
```

## 29.4. Connection string (ispit)

Server `192.168.0.1\Exams`, port `1999`, baza = indeks, user `john`, pass `doe2025`.

Aplikacijski login nakon seeda: `customer1` / `Test123`.

## 29.5. Ovaj ispit u 8 redova

1. Kartica (ime + indeks), 1:N na User.
2. CRUD lanac kao Category + user iz JWT.
3. Validacije: 12 / 3 cifre, datum, decimal >= 0.
4. Order.FK na karticu.
5. Checkout bira karticu, računa stanje, ne smije minus.
6. Mobile profil: lista, dodaj, edit, transakcije.
7. Fajl `payment_card_details.dart`.
8. Clean, zip indeks, FTP.

## 29.6. Kalupi koje otvaraš odmah

* CRUD: `Category*`
* JWT user + pravila: `ProductReviewService`, `OrderService`
* Checkout: `OrderService.CheckoutAsync`, `CartListScreen`, `OrderProvider`
* Profil reload: `ProfileScreen`
* Forma insert: `AddReviewScreen`
* Create/edit isti ekran: desktop `CategoryDetailsScreen` (obrazac, ne lokacija)
* Greška korisniku: `ClinetException` + `ApiClientException`

## 29.7. Formula ovog ispita

```
dostupno = početnoStanje - suma(uspješnih narudžbi te kartice)
plaćanje OK samo ako kartica nije istekla I dostupno >= iznosRačuna
početnoStanje se NE smanjuje u tabeli
```

---

# 30. Rječnik pojmova

Svaki pojam: jednostavno objašnjenje → gdje je u templateu → kako ga prepoznati na ispitu.

**Entity** – C# klasa = red u tabeli. Folder `Database/`. Na ispitu: „dodati entitet".

**DbContext** – most prema bazi. `ECommerceDbContext`. Ne praviš `new` u kontroleru.

**DbSet** – ulaz u jednu tabelu. Property na DbContextu.

**Migracija** – skripta koja mijenja SQL šemu. Folder `Migrations/`. Kad god dirneš entity.

**Request** – JSON koji Flutter šalje. `Model/Requests/`.

**Response** – JSON koji API vraća. `Model/Responses/`.

**SearchObject** – filteri na GET. Nasljeđuje `BaseSearchObject` (page, pageSize).

**DTO** – Request/Response/Search. Nije entity.

**Mapster** – kopira ista imena polja. `IMapper`.

**FluentValidation** – pravila na Requestu. Folder `Validators/`.

**Service** – poslovna logika. `CategoryService`. Umjesto RS1 handlera.

**BaseCRUDService** – gotov Insert/Update/Delete. Ti dodaš filtere i specijalne slučajeve.

**Controller** – HTTP vrata. Tanki. `CategoriesController`.

**Endpoint** – URL + metoda. `/Orders/Checkout`.

**DI / AddScoped** – registracija u `Program.cs`.

**JWT** – token nakon logina. Flutter ga šalje kao Bearer.

**IAuthenticatedUserAccessor** – „ko je ulogovan". `GetUserId()`.

**ClinetException** – namjerna poslovna greška → HTTP 400 + poruka. Typo u imenu ostavi.

**ExceptionFilter** – hvata greške i pretvara ih u JSON.

**PageResult** – `{ items, totalCount }`.

**IQueryable** – SQL upit koji se još nije izvršio. `Include` se radi na njemu.

**Include** – učitaj povezane tabele (OrderItems, User). Inače su navigation null.

**Navigation property** – `card.User`, `user.Cards`.

**Foreign key (FK)** – int kolona koja pokazuje na drugi red, npr. `UserId`.

**1:N** – jedan roditelj, više djece. Najčešće na ispitu.

**Cascade / Restrict** – šta se desi djeci kad obrišeš roditelja.

**Seed** – početni podaci. `eCommerceSeed.cs`. `customer1` je odatle.

**Provider (Flutter)** – klasa koja zove API. `UserProvider`.

**json_serializable** – od JSON-a pravi Dart objekat. Treba `.g.dart`.

**build_runner** – generiše `.g.dart`.

**StatefulWidget** – ekran koji ima `setState` (liste, forme).

**Navigator.push / pop** – otvori/zatvori ekran.

**Computed / izračunato** – nije kolona. Stanje kartice na ovom ispitu.

**Checkout** – pretvaranje korpe u narudžbu. Već postoji, ti dodaješ karticu.

**Scalar / Swagger** – UI za testiranje API-ja u browseru.

---

# Dodatak A: Kako izgleda „jedan puni CRUD" kroz fajlove

Kad na sljedećem ispitu dobiješ novi entitet, hodaj ovim putem. Imena su iz **postojećeg** Category, da imaš mapu. Za ispit mijenjaš imena.

1. `eCommerce.Services/Database/Category.cs`
2. `eCommerce.Services/Database/eCommerceDbContext.cs` → `DbSet<Category>`
3. `eCommerce.Services/Database/eCommerceConfiguration.cs` ako treba relacija
4. Migracija
5. `eCommerce.Model/Requests/CategoriesInsertRequest.cs`
6. `eCommerce.Model/Requests/CategoriesUpdateRequest.cs`
7. `eCommerce.Model/Responses/CategoryResponse.cs`
8. `eCommerce.Model/SearchObjects/CategorySearch.cs`
9. `eCommerce.Services/Validators/CategoryInsertValidator.cs`
10. `eCommerce.Services/Validators/CategoryUpdateValidator.cs`
11. `eCommerce.Services/ICategoryService.cs`
12. `eCommerce.Services/CategoryService.cs`
13. `eCommerce.WebAPI/Program.cs` – AddScoped service + validators
14. `eCommerce.WebAPI/Controllers/CategoriesController.cs`
15. Swagger test
16. Flutter model, provider, screen, main.dart

Ako umiješ ovo uraditi za Category **očima** (ne napamet kod), umiješ uraditi kartice.

---

# Dodatak B: Šta otvoriti u Visual Studio-u prvih 5 minuta

1. Solution Explorer: vidiš 4 C# projekta.
2. `eCommerce.WebAPI` = Set as Startup Project.
3. `appsettings.Development.json` → connection string.
4. PMC → Default project `eCommerce.Services` → `Update-Database`.
5. F5. Browser/Swagger.
6. `POST /Access/Login` body:

```json
{ "username": "customer1", "password": "Test123" }
```

7. Copy token. Authorize u Swaggeru.
8. `GET /Categories` – ako ovo radi, template je živ.
9. Tek onda piši svoj kod.

---

# Dodatak C: Šta otvoriti u VS Code / Flutter prvih 5 minuta

1. File → Open folder `eCommerce/UI/ecommerce_mobile`.
2. Terminal: `flutter pub get`.
3. Provjeri da API sluša na 5126.
4. Pokreni emulator / device.
5. Login customer1.
6. Tab Profile mora se otvoriti. Tu ćeš dodati sekciju.
7. Tab Cart mora imati Place order. Tu ćeš dodati odabir kartice.

Ako login ne radi: ili API nije upaljen, ili URL nije `10.0.2.2:5126`, ili baza nije migrirana pa user ne postoji.

---

# Dodatak D: Kako misliti o „drugim atributima koje smatrate potrebnim"

Zadatak to kaže namjerno. Profesor ne želi 20 polja. Želi da vidiš obrazac postojećih entiteta.

Razumno:

* `Id`
* `CreatedAt` / `UpdatedAt` (baza servisa to već poštuje)
* možda `IsActive`

Nije razumno:

* `CurrentBalance` (zabranjeno tekstom)
* `BankName`, `CardBrand`, `Color`, `Nickname` – osim ako stigneš i ne kvariš validaciju
* `Transactions` kao **kolona string**. Transakcije su redovi narudžbi.

Ako nisi sigurna, dodaj samo što tekst traži + Id + CreatedAt. To je dovoljno.

---

# Dodatak E: Kako testirati primjer iz PDF-a ručno

1. Insert kartice, početno stanje 200, istek u budućnosti.
2. Checkout proizvoda u vrijednosti 50, ta kartica. (U korpu stavljaj proizvode čija suma cijena * količina = 50. Cijene vidi u bazi/Swagger Products.)
3. Još jedan checkout 30, ista kartica.
4. GetById kartice: dostupno treba biti 120 (ako vraćaš to polje) ili transakcije 50 i 30.
5. Checkout 130 → mora pasti.
6. Checkout 120 → mora proći.
7. U tabeli kartica početno i dalje 200.

Ako nemaš proizvode od tačno 50: uzmi bilo koje iznose, ali **sama izračunaj** očekivano. Profesorov primjer je za razumijevanje; tvoj test mora poštovati istu formulu.

---

# 31. KODOVI koje pišeš na ispitu

Ovo je sekcija zbog koje možeš sjesti za Visual Studio i **znati šta kucati**.

Pravila čitanja:

* `IBXXXXXX` **uvijek** zamijeni svojim indeksom, npr. `IB210001`.
* Kod je kalup u **istom stilu** kao template. Nije čarolija — to je Category/Order/Review, samo druga imena.
* Gdje piše `// DODAJ`, to je jedna linija u **postojećem** fajlu.
* Gdje piše cijeli fajl, praviš **novi** fajl desnim klikom → Add → Class / New File.
* Nemoj mijenjati `BaseCRUDService.cs`. On već radi Insert/Update/Delete.

Prije pisanja otvori u Visual Studiju **pored** svog novog fajla:

* Entity: `Database/Category.cs` i `Database/ProductReview.cs`
* Servis: `CategoryService.cs` i `ProductReviewService.cs`
* Checkout: `OrderService.cs`

---

## 31.1. Imena koja moraš uskladiti

Ako ti je indeks `IB210001`, onda:

| Šta | Ime |
|-----|-----|
| Entity klasa | `PaymentCardIB210001` |
| Tabela / DbSet | `PaymentCardsIB210001` |
| Insert request | `PaymentCardIB210001InsertRequest` |
| Update request | `PaymentCardIB210001UpdateRequest` |
| Response | `PaymentCardIB210001Response` |
| Search | `PaymentCardIB210001Search` |
| Validator insert | `PaymentCardIB210001InsertValidator` |
| Validator update | `PaymentCardIB210001UpdateValidator` |
| Interface | `IPaymentCardIB210001Service` |
| Servis | `PaymentCardIB210001Service` |
| Controller | `PaymentCardIB210001Controller` |
| Ruta | `/PaymentCardIB210001` |
| Flutter endpoint | `PaymentCardIB210001` |
| FK na Order | `PaymentCardIB210001Id` |

U kalupima ispod ostavljam `IBXXXXXX`. **Find & Replace** prije ispita u glavi: XXXXXX → tvoj broj.

---

## 31.2. Connection string (postojeći fajl)

Fajl: `eCommerce.WebAPI/appsettings.Development.json`

**Šta već piše** (lokalni Docker):

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=localhost,1435;Database=eCommerce;User Id=sa;Password=qweasd123!;TrustServerCertificate=True;"
}
```

**Šta ti napišeš na ispitu** (svoj indeks, ne IB150051):

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=192.168.0.1\\Exams,1999;Database=IBXXXXXX;User Id=john;Password=doe2025;TrustServerCertificate=True;"
}
```

PMC (Default project = `eCommerce.Services`):

```
Update-Database
```

Poslije tvoje nove tabele:

```
Add-Migration AddPaymentCardsIBXXXXXX
Update-Database
```

---

## 31.3. NOVI fajl: Entity kartice

Desni klik na folder `eCommerce.Services/Database` → Add → Class.

Ime fajla: `PaymentCardIBXXXXXX.cs`

**Zašto ovako:** isti atributi kao `Product` (novac) i `ProductReview` (FK na User).

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace eCommerce.Services.Database
{
    public class PaymentCardIBXXXXXX
    {
        [Key]
        public int Id { get; set; }

        // veza na korisnika — „jedan korisnik više kartica"
        public int UserId { get; set; }

        [ForeignKey(nameof(UserId))]
        public User User { get; set; } = null!;

        // 12 cifara — string zbog vodećih nula i validacije
        [Required]
        [MaxLength(12)]
        public string CardNumber { get; set; } = string.Empty;

        [Required]
        [MaxLength(3)]
        public string Cvc { get; set; } = string.Empty;

        // istek — DateTime je najlakše validirati sa UtcNow
        [Required]
        public DateTime ExpiryDate { get; set; }

        [Required]
        [Column(TypeName = "decimal(18,2)")]
        public decimal InitialBalance { get; set; }

        // NEMA CurrentBalance — zadatak kaže da se računa

        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
        public DateTime? UpdatedAt { get; set; }

        // narudžbe plaćene ovom karticom = transakcije
        public ICollection<Order> Orders { get; set; } = new List<Order>();
    }
}
```

Ako želiš kraj mjeseca umjesto tačnog dana: i dalje koristi `DateTime`, npr. unesi `2027-06-30`. Na ispitu je važno da istekla kartica padne, ne ISO standard banaka.

---

## 31.4. DODAJ u postojeći `User.cs`

Fajl: `eCommerce.Services/Database/User.cs`

Na dno klase, pored ostalih `ICollection`:

```csharp
public ICollection<PaymentCardIBXXXXXX> PaymentCardsIBXXXXXX { get; set; }
    = new List<PaymentCardIBXXXXXX>();
```

To je strana **1** u relaciji 1:N.

---

## 31.5. DODAJ u postojeći `Order.cs`

Fajl: `eCommerce.Services/Database/Order.cs`

Unutar klase `Order`, npr. ispod `PaymentDate`:

```csharp
// nullable jer stare narudžbe u seedu nemaju karticu
public int? PaymentCardIBXXXXXXId { get; set; }

[ForeignKey(nameof(PaymentCardIBXXXXXXId))]
public PaymentCardIBXXXXXX? PaymentCardIBXXXXXX { get; set; }
```

Zašto `int?` a ne `int`: migracija neće puknuti na starim redovima.

---

## 31.6. DODAJ u postojeći `eCommerceConfiguration.cs`

Fajl: `eCommerce.Services/Database/eCommerceConfiguration.cs`

Unutar `CreateConfiguration`, kopiraj **isti oblik** kao ProductReview → Order:

```csharp
modelBuilder.Entity<PaymentCardIBXXXXXX>()
    .HasOne(c => c.User)
    .WithMany(u => u.PaymentCardsIBXXXXXX)
    .HasForeignKey(c => c.UserId)
    .OnDelete(DeleteBehavior.Cascade);

modelBuilder.Entity<Order>()
    .HasOne(o => o.PaymentCardIBXXXXXX)
    .WithMany(c => c.Orders)
    .HasForeignKey(o => o.PaymentCardIBXXXXXXId)
    .OnDelete(DeleteBehavior.Restrict);
```

**Cascade** na kartice: ako se obriše user, idu i njegove kartice.

**Restrict** na narudžbe: brisanje kartice ne smije obrisati historiju plaćanja.

---

## 31.7. DODAJ u postojeći `eCommerceDbContext.cs`

Fajl: `eCommerce.Services/Database/eCommerceDbContext.cs`

Pored ostalih `DbSet`:

```csharp
public DbSet<PaymentCardIBXXXXXX> PaymentCardsIBXXXXXX { get; set; }
```

Sada Build (Ctrl+Shift+B). Ako ima crvenih grešaka, ne radi migraciju.

Zatim PMC:

```
Add-Migration AddPaymentCardsIBXXXXXX
Update-Database
```

Otvori generisani fajl u `Migrations/` i provjeri da vidiš `CreateTable` i `AddColumn` na Orders.

---

## 31.8. NOVI DTO fajlovi u `eCommerce.Model`

Kalup je `CategoriesInsertRequest` — **bez** `[Key]`, **bez** navigation objekata.

### `Requests/PaymentCardIBXXXXXXInsertRequest.cs`

```csharp
namespace eCommerce.Model.Requests
{
    public class PaymentCardIBXXXXXXInsertRequest
    {
        public string CardNumber { get; set; } = string.Empty;
        public string Cvc { get; set; } = string.Empty;
        public DateTime ExpiryDate { get; set; }
        public decimal InitialBalance { get; set; }
        // UserId NE šaljemo iz forme — uzima se iz JWT-a u servisu
    }
}
```

### `Requests/PaymentCardIBXXXXXXUpdateRequest.cs`

```csharp
namespace eCommerce.Model.Requests
{
    public class PaymentCardIBXXXXXXUpdateRequest
    {
        public string CardNumber { get; set; } = string.Empty;
        public string Cvc { get; set; } = string.Empty;
        public DateTime ExpiryDate { get; set; }
        public decimal InitialBalance { get; set; }
    }
}
```

### `Responses/PaymentCardIBXXXXXXResponse.cs`

```csharp
namespace eCommerce.Model.Responses
{
    public class PaymentCardIBXXXXXXResponse
    {
        public int Id { get; set; }
        public int UserId { get; set; }
        public string CardNumber { get; set; } = string.Empty;
        public string Cvc { get; set; } = string.Empty;
        public DateTime ExpiryDate { get; set; }
        public decimal InitialBalance { get; set; }

        // izračunato, NIJE kolona
        public decimal AvailableBalance { get; set; }

        public DateTime CreatedAt { get; set; }
        public DateTime? UpdatedAt { get; set; }

        // za edit ekran: datum + iznos transakcija
        public List<PaymentCardTransactionResponse> Transactions { get; set; } = new();
    }

    public class PaymentCardTransactionResponse
    {
        public DateTime Date { get; set; }
        public decimal Amount { get; set; }
    }
}
```

`Transactions` i `AvailableBalance` Mapster **ne može** sam popuniti iz entiteta. Njih puniš u servisu (vidi 31.11). Lista na GetAll može ostati prazna — puni ih u GetById da lista bude brža.

### `SearchObjects/PaymentCardIBXXXXXXSearch.cs`

```csharp
namespace eCommerce.Model.SearchObjects
{
    public class PaymentCardIBXXXXXXSearch : BaseSearchObject
    {
        public int? UserId { get; set; }
    }
}
```

**Mora** naslijediti `BaseSearchObject`. Bez toga se projekat ne kompajlira.

---

## 31.9. NOVI validatori

Kalup: `CategoryInsertValidator` / `ProductReviewInsertValidator`.

Fajl: `eCommerce.Services/Validators/PaymentCardIBXXXXXXInsertValidator.cs`

```csharp
using eCommerce.Model.Requests;
using FluentValidation;

namespace eCommerce.Services.Validators
{
    public class PaymentCardIBXXXXXXInsertValidator
        : AbstractValidator<PaymentCardIBXXXXXXInsertRequest>
    {
        public PaymentCardIBXXXXXXInsertValidator()
        {
            RuleFor(x => x.CardNumber)
                .NotEmpty().WithMessage("Card number is required.")
                .Matches("^[0-9]{12}$")
                .WithMessage("Card number must be exactly 12 digits.");

            RuleFor(x => x.Cvc)
                .NotEmpty().WithMessage("CVC is required.")
                .Matches("^[0-9]{3}$")
                .WithMessage("CVC must be exactly 3 digits.");

            RuleFor(x => x.ExpiryDate)
                .Must(d => d > DateTime.UtcNow)
                .WithMessage("Card is expired.");

            RuleFor(x => x.InitialBalance)
                .GreaterThanOrEqualTo(0)
                .WithMessage("Initial balance must be 0 or greater.");
        }
    }
}
```

Šta znači regex:

* `^` početak stringa
* `[0-9]` samo cifre
* `{12}` tačno 12
* `$` kraj stringa

Zato `"1234 5678 9012"` (razmak) **pada**, `"123"` **pada**, `"12ab"` **pada**.

Update validator je **isti** `RuleFor` blok, samo:

```csharp
public class PaymentCardIBXXXXXXUpdateValidator
    : AbstractValidator<PaymentCardIBXXXXXXUpdateRequest>
```

Možeš copy-paste insert validatora i zamijeniti ime klase i `T` u `AbstractValidator<...>`.

---

## 31.10. NOVI interface

Fajl: `eCommerce.Services/IPaymentCardIBXXXXXXService.cs`

Ovo je doslovno `ICategoryService` sa drugim tipovima:

```csharp
using eCommerce.Model.Requests;
using eCommerce.Model.Responses;
using eCommerce.Model.SearchObjects;

namespace eCommerce.Services
{
    public interface IPaymentCardIBXXXXXXService
        : IBaseCRUDService<
            PaymentCardIBXXXXXXResponse,
            PaymentCardIBXXXXXXSearch,
            PaymentCardIBXXXXXXInsertRequest,
            PaymentCardIBXXXXXXUpdateRequest>
    {
    }
}
```

Prazno tijelo je OK. Insert/Update/Delete/Get dolaze iz `IBaseCRUDService`.

---

## 31.11. NOVI servis — ovo je fajl koji najviše pišeš rukom

Fajl: `eCommerce.Services/PaymentCardIBXXXXXXService.cs`

Dva kalupa spojena:

* `CategoryService` = nasljeđivanje + `ApplyFilters`
* `ProductReviewService` = `IAuthenticatedUserAccessor` da kartica pripada ulogovanom useru

```csharp
using eCommerce.Model.Exceptions;
using eCommerce.Model.Requests;
using eCommerce.Model.Responses;
using eCommerce.Model.SearchObjects;
using eCommerce.Services.Database;
using FluentValidation;
using MapsterMapper;
using Microsoft.EntityFrameworkCore;

namespace eCommerce.Services
{
    public class PaymentCardIBXXXXXXService
        : BaseCRUDService<
            PaymentCardIBXXXXXX,
            PaymentCardIBXXXXXXResponse,
            PaymentCardIBXXXXXXSearch,
            PaymentCardIBXXXXXXInsertRequest,
            PaymentCardIBXXXXXXUpdateRequest>,
          IPaymentCardIBXXXXXXService
    {
        private readonly IAuthenticatedUserAccessor _userAccessor;

        public PaymentCardIBXXXXXXService(
            ECommerceDbContext dbContext,
            IMapper mapper,
            IValidator<PaymentCardIBXXXXXXInsertRequest> insertValidator,
            IValidator<PaymentCardIBXXXXXXUpdateRequest> updateValidator,
            IAuthenticatedUserAccessor userAccessor)
            : base(dbContext, mapper, insertValidator, updateValidator)
        {
            _userAccessor = userAccessor;
        }

        private int RequireUserId()
        {
            return _userAccessor.GetUserId()
                   ?? throw new InvalidOperationException("User id claim is missing.");
        }

        protected override IEnumerable<PaymentCardIBXXXXXX> ApplyFilters(
            IEnumerable<PaymentCardIBXXXXXX> query,
            PaymentCardIBXXXXXXSearch? search)
        {
            var userId = _userAccessor.GetUserId();
            if (!userId.HasValue)
            {
                return Enumerable.Empty<PaymentCardIBXXXXXX>();
            }

            // samo moje kartice — NE vjeruj search.UserId sa klijenta
            query = query.Where(c => c.UserId == userId.Value);
            return query;
        }

        protected override PaymentCardIBXXXXXX MapInsertRequestToEntity(
            PaymentCardIBXXXXXXInsertRequest request)
        {
            var entity = base.MapInsertRequestToEntity(request);
            entity.UserId = RequireUserId();
            return entity;
        }

        public override async Task<PaymentCardIBXXXXXXResponse> GetByIdAsync(int id)
        {
            var userId = RequireUserId();

            var entity = await _dbContext.Set<PaymentCardIBXXXXXX>()
                .AsNoTracking()
                .Include(c => c.Orders)
                .FirstOrDefaultAsync(c => c.Id == id && c.UserId == userId);

            if (entity == null)
            {
                throw new KeyNotFoundException($"PaymentCard with id {id} not found.");
            }

            return ToResponse(entity);
        }

        public override async Task<PageResult<PaymentCardIBXXXXXXResponse>> GetAllAsync(
            PaymentCardIBXXXXXXSearch? search = null)
        {
            search ??= new PaymentCardIBXXXXXXSearch();
            if (search.PageSize == null || search.PageSize < 50)
            {
                search.PageSize = 50; // da na profilu ne nestanu kartice zbog default 10
            }

            var page = await base.GetAllAsync(search);

            // base.GetAll ne računa AvailableBalance — dopuni
            foreach (var item in page.Items)
            {
                item.AvailableBalance = await CalculateAvailableAsync(item.Id, item.InitialBalance);
            }

            return page;
        }

        public override async Task<PaymentCardIBXXXXXXResponse> UpdateAsync(
            int id,
            PaymentCardIBXXXXXXUpdateRequest request)
        {
            var userId = RequireUserId();
            var entity = await _dbContext.Set<PaymentCardIBXXXXXX>().FindAsync(id);

            if (entity == null || entity.UserId != userId)
            {
                throw new KeyNotFoundException($"PaymentCard with id {id} not found.");
            }

            // baza UpdateAsync bi našla i tuđu karticu po id — zato override
            return await base.UpdateAsync(id, request);
        }

        private PaymentCardIBXXXXXXResponse ToResponse(PaymentCardIBXXXXXX entity)
        {
            var response = _mapper.Map<PaymentCardIBXXXXXXResponse>(entity);

            var successful = entity.Orders
                .Where(o => o.Status != OrderStatus.Cancelled)
                .ToList();

            response.AvailableBalance =
                entity.InitialBalance - successful.Sum(o => o.TotalAmount);

            response.Transactions = successful
                .OrderByDescending(o => o.OrderDate)
                .Select(o => new PaymentCardTransactionResponse
                {
                    Date = o.OrderDate,
                    Amount = o.TotalAmount
                })
                .ToList();

            return response;
        }

        private async Task<decimal> CalculateAvailableAsync(int cardId, decimal initial)
        {
            var spent = await _dbContext.Orders
                .Where(o => o.PaymentCardIBXXXXXXId == cardId
                            && o.Status != OrderStatus.Cancelled)
                .SumAsync(o => (decimal?)o.TotalAmount) ?? 0;

            return initial - spent;
        }
    }
}
```

Zašto `SumAsync(o => (decimal?)o.TotalAmount) ?? 0`:

Ako nema nijedne narudžbe, `Sum` na praznom skupu za `decimal` može baciti grešku. Nullable decimal + `?? 0` je siguran.

`OrderStatus` je u istom namespaceu `eCommerce.Services.Database` kao `Order.cs`.

Ako `GetAllAsync` override bude spor ili kompliciran na ispitu, radi **minimum**: `ApplyFilters` + `MapInsertRequestToEntity` + `GetByIdAsync` sa transakcijama. Listu na profilu možeš računati dostupno ili samo prikazati početno stanje + broj kartice. Formula **mora** živjeti u checkoutu.

---

## 31.12. DODAJ u postojeći `Program.cs`

Fajl: `eCommerce.WebAPI/Program.cs`

Nađi linije kao:

```csharp
builder.Services.AddScoped<ICategoryService, CategoryService>();
builder.Services.AddScoped<IValidator<CategoriesInsertRequest>, CategoryInsertValidator>();
```

**Ispod njih dopiši:**

```csharp
builder.Services.AddScoped<IPaymentCardIBXXXXXXService, PaymentCardIBXXXXXXService>();
builder.Services.AddScoped<IValidator<PaymentCardIBXXXXXXInsertRequest>, PaymentCardIBXXXXXXInsertValidator>();
builder.Services.AddScoped<IValidator<PaymentCardIBXXXXXXUpdateRequest>, PaymentCardIBXXXXXXUpdateValidator>();
```

Using-i na vrhu fajla već imaju `eCommerce.Services`, `eCommerce.Model.Requests`, `FluentValidation`. Ako kompajler ne vidi validator, dodaj:

```csharp
using eCommerce.Services.Validators;
```

(taj using već postoji u 2025/26 `Program.cs`.)

Mapster: `CardNumber` na requestu = `CardNumber` na entitetu = `CardNumber` na responseu. **Ne treba** `TypeAdapterConfig` ako su imena ista. `AvailableBalance` i `Transactions` ionako puniš ručno.

---

## 31.13. NOVI kontroler

Fajl: `eCommerce.WebAPI/Controllers/PaymentCardIBXXXXXXController.cs`

Kalup: `ProductReviewsController` (ima `[Authorize]`), ne `CategoriesController` (GetAll je AllowAnonymous — kartice nisu javne).

```csharp
using eCommerce.Model.Requests;
using eCommerce.Model.Responses;
using eCommerce.Model.SearchObjects;
using eCommerce.Services;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace eCommerce.WebAPI.Controllers;

[Authorize]
public class PaymentCardIBXXXXXXController
    : BaseCRUDController<
        PaymentCardIBXXXXXXResponse,
        PaymentCardIBXXXXXXSearch,
        PaymentCardIBXXXXXXInsertRequest,
        PaymentCardIBXXXXXXUpdateRequest,
        IPaymentCardIBXXXXXXService>
{
    public PaymentCardIBXXXXXXController(IPaymentCardIBXXXXXXService service)
        : base(service)
    {
    }
}
```

**Nema** `new ECommerceDbContext`. Nema LINQ. Prazan kontroler je tačan odgovor.

Rute koje dobiješ besplatno:

| HTTP | URL | Metoda u bazi |
|------|-----|----------------|
| GET | `/PaymentCardIBXXXXXX` | GetAll |
| GET | `/PaymentCardIBXXXXXX/5` | GetById |
| POST | `/PaymentCardIBXXXXXX` | Insert |
| PUT | `/PaymentCardIBXXXXXX/5` | Update |
| DELETE | `/PaymentCardIBXXXXXX/5` | Delete |

Swagger body za POST (primjer):

```json
{
  "cardNumber": "123456789012",
  "cvc": "123",
  "expiryDate": "2027-12-31T00:00:00Z",
  "initialBalance": 200
}
```

JSON koristi **camelCase**. Zato Flutter šalje `cardNumber`, ne `CardNumber`.

---

## 31.14. Checkout — proširi postojeće, ne pravi novi kontroler

### DODAJ u `CheckoutRequest.cs`

Fajl: `eCommerce.Model/Requests/CheckoutRequest.cs`

```csharp
public int PaymentCardIBXXXXXXId { get; set; }
```

cijeli fajl poslije izmjene izgleda ovako:

```csharp
namespace eCommerce.Model.Requests;

public class CheckoutRequest
{
    public List<CheckoutLineRequest> Items { get; set; } = new();

    public string? ShippingAddress { get; set; }
    public string? ShippingCity { get; set; }
    public string? ShippingState { get; set; }
    public string? ShippingZipCode { get; set; }
    public string? ShippingCountry { get; set; }

    public int PaymentCardIBXXXXXXId { get; set; }
}
```

### DODAJ u `OrderService.CheckoutAsync`

Fajl: `eCommerce.Services/OrderService.cs`

Metoda već postoji. **Ne briši** provjeru korpe i stocka. Ubaci karticu **prije** `Orders.Add`, a FK na `new Order { ... }`.

Na vrh metode, odmah nakon `userId`:

```csharp
if (request.PaymentCardIBXXXXXXId <= 0)
{
    throw new ClinetException("Please select a payment card.");
}

var card = await _dbContext.Set<PaymentCardIBXXXXXX>()
    .FirstOrDefaultAsync(c => c.Id == request.PaymentCardIBXXXXXXId && c.UserId == userId);

if (card == null)
{
    throw new ClinetException("Payment card was not found.");
}

if (card.ExpiryDate <= DateTime.UtcNow)
{
    throw new ClinetException("The selected card is expired.");
}
```

Unutar `new Order { ... }` dodaj property (pored `UserId`, `Status`, ...):

```csharp
PaymentCardIBXXXXXXId = card.Id,
PaymentDate = DateTime.UtcNow,
```

**Nakon** što izračunaš `total` (u templateu je to `order.TotalAmount = total;` prije `Add`), a **prije** `Add`/`SaveChanges`:

```csharp
var spent = await _dbContext.Orders
    .Where(o => o.PaymentCardIBXXXXXXId == card.Id
                && o.Status != OrderStatus.Cancelled)
    .SumAsync(o => (decimal?)o.TotalAmount) ?? 0;

var available = card.InitialBalance - spent;

if (available < total)
{
    throw new ClinetException(
        $"Insufficient funds. Available: {available:0.00}, order: {total:0.00}.");
}
```

Važno:

* `card.InitialBalance` se **ne smanjuje**.
* `ClinetException` (typo ime klase) → Flutter vidi poruku.
* Ako staviš ovaj blok **prije** petlje proizvoda, `total` još nije izračunat. Zato ide **poslije** `order.TotalAmount = total`.

Using na vrhu `OrderService.cs` već ima `eCommerce.Model.Exceptions` i `Microsoft.EntityFrameworkCore`.

---

## 31.15. Flutter model

Folder: `eCommerce/UI/ecommerce_mobile/lib/models/`

Novi fajl: `payment_card.dart`

Kalup: `product_review.dart`

```dart
import 'package:json_annotation/json_annotation.dart';

part 'payment_card.g.dart';

@JsonSerializable()
class PaymentCardTransaction {
  final DateTime? date;
  final double? amount;

  PaymentCardTransaction({this.date, this.amount});

  factory PaymentCardTransaction.fromJson(Map<String, dynamic> json) =>
      _$PaymentCardTransactionFromJson(json);

  Map<String, dynamic> toJson() => _$PaymentCardTransactionToJson(this);
}

@JsonSerializable()
class PaymentCard {
  final int? id;
  final int? userId;
  final String? cardNumber;
  final String? cvc;
  final DateTime? expiryDate;
  final double? initialBalance;
  final double? availableBalance;
  final List<PaymentCardTransaction> transactions;

  PaymentCard({
    this.id,
    this.userId,
    this.cardNumber,
    this.cvc,
    this.expiryDate,
    this.initialBalance,
    this.availableBalance,
    this.transactions = const [],
  });

  factory PaymentCard.fromJson(Map<String, dynamic> json) =>
      _$PaymentCardFromJson(json);

  Map<String, dynamic> toJson() => _$PaymentCardToJson(this);
}
```

U terminalu, folder `ecommerce_mobile`:

```
dart run build_runner build --delete-conflicting-outputs
```

To kreira `payment_card.g.dart`. **Bez ovoga** `part 'payment_card.g.dart';` ne postoji i Flutter je crven.

---

## 31.16. Flutter provider

Novi fajl: `lib/providers/payment_card_provider.dart`

Kalup: `product_review_provider.dart`

```dart
import 'package:ecommerce_mobile/models/payment_card.dart';
import 'package:ecommerce_mobile/providers/base_provider.dart';

class PaymentCardProvider extends BaseProvider<PaymentCard> {
  PaymentCardProvider() : super('PaymentCardIBXXXXXX');

  @override
  PaymentCard fromJson(data) =>
      PaymentCard.fromJson(data as Map<String, dynamic>);
}
```

String u `super(...)` mora biti **isto** kao ime kontrolera bez `Controller`.

Get/insert/update već postoje u `BaseProvider`. Ne pišeš HTTP ručno za CRUD.

---

## 31.17. DODAJ provider u `main.dart`

Fajl: `lib/main.dart`

1. Import:

```dart
import 'package:ecommerce_mobile/providers/payment_card_provider.dart';
```

2. Unutar `MultiProvider(providers: [ ... ]` dodaj pored `OrderProvider`:

```dart
ChangeNotifierProvider(create: (_) => PaymentCardProvider()),
```

Ako ovo zaboraviš, ekran pukne: `Could not find the correct Provider<PaymentCardProvider>`.

---

## 31.18. DODAJ sekciju na `profile_screen.dart`

Fajl: `lib/screens/profile_screen.dart`

### Importi na vrh

```dart
import 'package:ecommerce_mobile/models/payment_card.dart';
import 'package:ecommerce_mobile/providers/payment_card_provider.dart';
import 'package:ecommerce_mobile/screens/payment_card_details.dart';
```

### State polja (pored `late User user`)

```dart
List<PaymentCard> _cards = [];
```

### U `initData()` nakon što učitaš usera

```dart
final cards = await context.read<PaymentCardProvider>().get(
  filter: {'page': 1, 'pageSize': 50},
);
```

pa u `setState`:

```dart
_cards = cards.items ?? [];
```

`get` vraća `SearchResult` sa `items` — to je `PageResult` sa backenda.

### U `build`, unutar `Column` djece (npr. između `_buildProfileInfo()` i `_buildProfileMenu()`)

```dart
_buildMyCards(),
```

### Nove metode u `_ProfileScreenState`

```dart
Widget _buildMyCards() {
  return Padding(
    padding: const EdgeInsets.fromLTRB(26, 0, 26, 24),
    child: Column(
      crossAxisAlignment: CrossAxisAlignment.stretch,
      children: [
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            const Text(
              'Moje kartice',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            TextButton(
              onPressed: () async {
                final refresh = await Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (_) => const PaymentCardDetailsScreen(),
                  ),
                );
                if (refresh == 'reload') {
                  initData();
                }
              },
              child: const Text('Dodaj karticu'),
            ),
          ],
        ),
        if (_cards.isEmpty)
          const Text('Nemate sačuvanih kartica.')
        else
          ..._cards.map((card) {
            return Card(
              child: ListTile(
                title: Text(card.cardNumber ?? ''),
                subtitle: Text(
                  'Istek: ${card.expiryDate ?? ''}   '
                  'Početno: ${card.initialBalance ?? 0}',
                ),
                onTap: () async {
                  final refresh = await Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (_) => PaymentCardDetailsScreen(card: card),
                    ),
                  );
                  if (refresh == 'reload') {
                    initData();
                  }
                },
              ),
            );
          }),
      ],
    ),
  );
}
```

Ovo je isti `refresh == 'reload'` obrazac koji profil već koristi za `ProfileSettingsScreen`.

---

## 31.19. NOVI ekran `payment_card_details.dart`

Zadatak **imenuje** ovaj fajl. Mora se zvati tačno tako.

Kalup: `add_review_screen.dart` (TextEditingController + Save + pop).

Fajl: `lib/screens/payment_card_details.dart`

```dart
import 'package:ecommerce_mobile/models/payment_card.dart';
import 'package:ecommerce_mobile/providers/payment_card_provider.dart';
import 'package:ecommerce_mobile/utils/api_client_exception.dart';
import 'package:ecommerce_mobile/utils/utils_widgets.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class PaymentCardDetailsScreen extends StatefulWidget {
  final PaymentCard? card;

  const PaymentCardDetailsScreen({super.key, this.card});

  @override
  State<PaymentCardDetailsScreen> createState() =>
      _PaymentCardDetailsScreenState();
}

class _PaymentCardDetailsScreenState extends State<PaymentCardDetailsScreen> {
  final _number = TextEditingController();
  final _cvc = TextEditingController();
  final _expiry = TextEditingController(); // npr. 2027-12-31
  final _balance = TextEditingController();
  bool _saving = false;
  PaymentCard? _loaded;

  bool get _isEdit => widget.card?.id != null;

  @override
  void initState() {
    super.initState();
    final c = widget.card;
    if (c != null) {
      _number.text = c.cardNumber ?? '';
      _cvc.text = c.cvc ?? '';
      _expiry.text = c.expiryDate?.toIso8601String().split('T').first ?? '';
      _balance.text = c.initialBalance?.toString() ?? '';
    }
    if (_isEdit) {
      _loadDetails();
    }
  }

  Future<void> _loadDetails() async {
    try {
      final full = await context.read<PaymentCardProvider>().getById(
        widget.card!.id!,
      );
      setState(() => _loaded = full);
    } on Exception catch (e) {
      if (mounted) alertBox(context, 'Error', e.toString());
    }
  }

  @override
  void dispose() {
    _number.dispose();
    _cvc.dispose();
    _expiry.dispose();
    _balance.dispose();
    super.dispose();
  }

  String? _localError() {
    if (!RegExp(r'^[0-9]{12}$').hasMatch(_number.text)) {
      return 'Broj kartice mora imati tačno 12 cifara.';
    }
    if (!RegExp(r'^[0-9]{3}$').hasMatch(_cvc.text)) {
      return 'CVC mora imati tačno 3 cifre.';
    }
    final expiry = DateTime.tryParse(_expiry.text);
    if (expiry == null) {
      return 'Unesi datum isteka (npr. 2027-12-31).';
    }
    if (!expiry.isAfter(DateTime.now())) {
      return 'Kartica je istekla.';
    }
    final balance = double.tryParse(_balance.text.replaceAll(',', '.'));
    if (balance == null || balance < 0) {
      return 'Početno stanje mora biti broj ≥ 0.';
    }
    return null;
  }

  Map<String, dynamic> _body() {
    final expiry = DateTime.parse(_expiry.text);
    return {
      'cardNumber': _number.text,
      'cvc': _cvc.text,
      'expiryDate': expiry.toUtc().toIso8601String(),
      'initialBalance': double.parse(_balance.text.replaceAll(',', '.')),
    };
  }

  Future<void> _save() async {
    final err = _localError();
    if (err != null) {
      alertBox(context, 'Validacija', err);
      return;
    }

    setState(() => _saving = true);
    try {
      final provider = context.read<PaymentCardProvider>();
      if (_isEdit) {
        await provider.update(widget.card!.id!, _body());
      } else {
        await provider.insert(_body());
      }
      if (mounted) Navigator.pop(context, 'reload');
    } on ApiClientException catch (e) {
      if (mounted) alertBox(context, 'Error', e.message);
    } on Exception catch (e) {
      if (mounted) alertBox(context, 'Error', e.toString());
    } finally {
      if (mounted) setState(() => _saving = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    final txs = _loaded?.transactions ?? widget.card?.transactions ?? [];

    return Scaffold(
      appBar: AppBar(
        title: Text(_isEdit ? 'Uredi karticu' : 'Dodaj karticu'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            TextField(
              controller: _number,
              decoration: const InputDecoration(labelText: 'Broj kartice'),
              keyboardType: TextInputType.number,
              maxLength: 12,
            ),
            TextField(
              controller: _cvc,
              decoration: const InputDecoration(labelText: 'CVC'),
              keyboardType: TextInputType.number,
              maxLength: 3,
            ),
            TextField(
              controller: _expiry,
              decoration: const InputDecoration(
                labelText: 'Datum isteka (YYYY-MM-DD)',
              ),
            ),
            TextField(
              controller: _balance,
              decoration: const InputDecoration(labelText: 'Početno stanje'),
              keyboardType: TextInputType.number,
            ),
            const SizedBox(height: 16),
            SizedBox(
              width: double.infinity,
              height: 48,
              child: FilledButton(
                onPressed: _saving ? null : _save,
                child: Text(_isEdit ? 'Spremi' : 'Dodaj'),
              ),
            ),
            if (_isEdit) ...[
              const SizedBox(height: 24),
              const Text(
                'Transakcije',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
              Expanded(
                child: txs.isEmpty
                    ? const Text('Nema transakcija.')
                    : ListView.builder(
                        itemCount: txs.length,
                        itemBuilder: (_, i) {
                          final t = txs[i];
                          return ListTile(
                            title: Text('${t.amount ?? 0}'),
                            subtitle: Text('${t.date ?? ''}'),
                          );
                        },
                      ),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

Ako `Column` + `Expanded` pravi overflow, umotaj tijelo u `SingleChildScrollView` i transakcije stavi kao običan `Column` djece bez `Expanded`. Na ispitu važnije da se vidi lista nego da layout bude savršen.

---

## 31.20. DODAJ odabir kartice na korpi

### `order_provider.dart` — proširi `checkout`

Sada šalje samo `items`. Treba i id kartice.

```dart
Future<Order> checkout(
  List<Map<String, dynamic>> items, {
  required int paymentCardId,
}) async {
  final uri = Uri.parse('${BaseProvider.baseUrl}Orders/Checkout');
  final headers = createHeaders();
  final body = jsonEncode({
    'items': items,
    'paymentCardIBXXXXXXId': paymentCardId,
  });
  final response = await http.post(uri, headers: headers, body: body);
  validateResponse(response);
  return Order.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
}
```

JSON key mora biti **camelCase** imena C# propertyja `PaymentCardIBXXXXXXId` → `paymentCardIBXXXXXXId`.

Ako zaboraviš zamijeniti IBXXXXXX, backend neće bindati polje (ostaje 0) i checkout će reći „select a payment card".

### `cart_list_screen.dart`

Import:

```dart
import 'package:ecommerce_mobile/models/payment_card.dart';
import 'package:ecommerce_mobile/providers/payment_card_provider.dart';
```

State:

```dart
List<PaymentCard> _cards = [];
int? _selectedCardId;
```

U `initState` nakon `_cartProvider = ...` pozovi load:

```dart
_loadCards();
```

```dart
Future<void> _loadCards() async {
  try {
    final result = await context.read<PaymentCardProvider>().get(
      filter: {'page': 1, 'pageSize': 50},
    );
    setState(() {
      _cards = result.items ?? [];
      if (_cards.isNotEmpty) {
        _selectedCardId = _cards.first.id;
      }
    });
  } on Exception catch (e) {
    if (mounted) alertBox(context, 'Cards', e.toString());
  }
}
```

U `_checkout()`, prije `setState(() => _checkoutBusy = true)`:

```dart
if (_selectedCardId == null) {
  alertBox(context, 'Payment', 'Odaberite karticu ili je dodajte u profilu.');
  return;
}
```

Poziv checkouta zamijeni sa:

```dart
final order = await context.read<OrderProvider>().checkout(
  payload,
  paymentCardId: _selectedCardId!,
);
```

U `_buildFooter`, iznad `FilledButton` Place order:

```dart
if (_cards.isEmpty)
  const Padding(
    padding: EdgeInsets.only(bottom: 8),
    child: Text('Dodajte karticu u profilu prije plaćanja.'),
  )
else
  Padding(
    padding: const EdgeInsets.only(bottom: 8),
    child: DropdownButtonFormField<int>(
      value: _selectedCardId,
      decoration: const InputDecoration(labelText: 'Platna kartica'),
      items: _cards
          .map(
            (c) => DropdownMenuItem(
              value: c.id,
              child: Text(
                '${c.cardNumber}  (dostupno: ${c.availableBalance ?? c.initialBalance})',
              ),
            ),
          )
          .toList(),
      onChanged: (v) => setState(() => _selectedCardId = v),
    ),
  ),
```

Ako `DropdownButtonFormField.value` bude deprecated u tvojoj Flutter verziji, koristi `initialValue` kako IDE predloži. Bitno je `onChanged` i `items`.

---

## 31.21. Redoslijed kucanja (skraćeno)

1. Connection string + `Update-Database`
2. Entity + User collection + Order FK + Configuration + DbSet
3. Build → `Add-Migration` → `Update-Database`
4. 4 DTO fajla + 2 validatora
5. Interface + Service
6. `Program.cs` 3 linije
7. Controller
8. `CheckoutRequest` + blok u `CheckoutAsync`
9. Swagger: login, POST kartice, GET, checkout
10. Flutter: model → build_runner → provider → main → profil → details → korpa

Ako kompajler javi grešku, čitaj **prvi** error. Najčešće: nisi zamijenila `IBXXXXXX`, fali `using`, fali `ApplyFilters`, fali `AddScoped`.

---

## 31.22. Mini „šta kucam" za postojeći Category — da vidiš analogiju

Kad zapneš, otvori ovo i usporedi sa svojim imenima.

**Interface:**

```csharp
public interface ICategoryService
    : IBaseCRUDService<CategoryResponse, CategorySearchObject, CategoriesInsertRequest, CategoriesUpdateRequest>
{ }
```

**Servis nasljeđivanje (jedna linija):**

```csharp
public class CategoryService
    : BaseCRUDService<Category, CategoryResponse, CategorySearchObject, CategoriesInsertRequest, CategoriesUpdateRequest>,
      ICategoryService
```

**Konstruktor servisa:**

```csharp
public CategoryService(
    ECommerceDbContext dbContext,
    MapsterMapper.IMapper mapper,
    IValidator<CategoriesInsertRequest> insertValidator,
    IValidator<CategoriesUpdateRequest> updateValidator)
    : base(dbContext, mapper, insertValidator, updateValidator)
{ }
```

**Kontroler nasljeđivanje:**

```csharp
public class CategoriesController
    : BaseCRUDController<CategoryResponse, CategorySearchObject, CategoriesInsertRequest, CategoriesUpdateRequest, ICategoryService>
```

Tvoja kartica je ista slika, 5 generičkih parametara, druga imena.

---

## 31.23. Šta NE kucaš

```csharp
var db = new ECommerceDbContext(...); // ZABRANJENO u kontroleru
```

```csharp
public decimal CurrentBalance { get; set; } // ZABRANJENO zadatkom
```

```csharp
card.InitialBalance -= total; // POGREŠNO — stanje se računa
```

```
Commands/CreatePaymentCard/  // TO JE RS1, NE RSII
```

---

# Završna poruka

RSII ispit nije „napiši aplikaciju iz nule". RSII ispit je:

> Pročitaj tekst. Prepoznaj obrazac. Pronađi isti obrazac u templateu. Ponovi ga za novi pojam. Proširi postojeći checkout. Poveži Flutter ekran koji već postoji.

Kartica je samo novi `Category` sa strožom validacijom, vlasnikom iz JWT-a, i vezom na `Order`.

Ako zapneš, ne pitaj „kako se piše C# klasa". Pitaj:

1. Koji sloj?
2. Koji postojeći fajl je sličan?
3. Koja rečenica u PDF-u ovo traži?

Taj trik pitanja je cijeli predmet.

Sretno na ispitu. Piši sama. Template je udžbenik. Ovaj vodič je kompas, ne rješenje.

---

*Vodič je pisan prema stvarnom templateu `Adil-Eminagic/rsII_exam_template_2025_26` i PDF-u `RSII_Zadatak_24062026.pdf`. Nije gotovo rješenje ispita.*


