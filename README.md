# Dokumentacija projekta: Edukativna demonstracija autentikacije korisnika

## Svrha projekta

Ovaj projekt predstavlja interaktivnu edukativnu aplikaciju koja korisnicima objašnjava osnovne koncepte autentikacije i sigurnosti na webu. Aplikacija je dizajnirana kao vođeni tutorial koji korak po korak objašnjava proces prijave korisnika, zaštitu osjetljivih podataka, te načine na koje moderni web sustavi održavaju stanje prijave korisnika.

Glavni cilj aplikacije je demistificirati procese koji se odvijaju "iza kulisa" kada korisnik unese svoje podatke za prijavu na web stranici, s posebnim naglaskom na sigurnosne aspekte i najbolje prakse.

**Projekt je dostupan na: https://auth-example.teomilic.com (Preporuča se projekt otvoriti samo na računalu!)**

## Tehnički pregled

Aplikacija je razvijena korištenjem sljedećih tehnologija:
- **TypeScript** - programski jezik za frontend razvoj
- **Svelte** - frontend framework koji omogućava reaktivno programiranje
- **CryptoJS** - biblioteka za implementaciju kriptografskih funkcija (SHA-256)
- **Jose** - biblioteka za rad s JWT (JSON Web Token) standardom

Aplikacija koristi reaktivno stanje za praćenje napretka korisnika kroz tutorial, pohranu unesenih vrijednosti, i generiranje izlaznih podataka poput hash vrijednosti i JWT tokena.

## Funkcionalne cjeline

### 1. Demonstracija unosa podataka i problema plaintext pohrane
Aplikacija započinje objašnjavanjem problema pohrane lozinki kao običnog teksta. Korisnik unosi email i lozinku, a aplikacija objašnjava zašto bi pohranjivanje tih vrijednosti bez dodatne zaštite predstavljalo sigurnosni rizik.

### 2. Implementacija hash funkcija
U drugom koraku, aplikacija demonstrira korištenje SHA-256 algoritma za stvaranje hash vrijednosti od korisničkih podataka. Objašnjava se kako hash funkcije rade u jednom smjeru i zašto su korisne za sigurno pohranjivanje osjetljivih podataka.

```typescript
function generateHashes() {
  hashBank.hashEmail = sha256(valueBank.email).toString();
  hashBank.hashPswd = sha256(valueBank.password).toString();
}
```

### 3. Simulacija autentikacije korisnika
Aplikacija simulira proces provjere identiteta korisnika na temelju hash vrijednosti, objašnjavajući kako server može provjeriti autentičnost korisnika bez poznavanja originalne lozinke.

### 4. Uvod u kolačiće i stanje prijave
Korisnicima se objašnjava kako web aplikacije moraju održavati stanje prijave korisnika između različitih stranica i posjeta. Objašnjava se važnost kolačića u ovom procesu.

### 5. Implementacija JWT (JSON Web Token)
U završnim koracima, aplikacija objašnjava i demonstrira JWT standard:

```typescript
async function generateJWT() {
  const secret = new TextEncoder().encode("kljuc")
  jwtBank.auth = await new jose.SignJWT({ime_koje_trazis: "branko"})
    .setProtectedHeader({ "alg": "HS256" })
    .setIssuedAt()
    .sign(secret)
  jwtBank.refresh = await new jose.SignJWT({kolacici: "da ne mozda"})
    .setProtectedHeader({ "alg": "HS256" })
    .setIssuedAt()
    .sign(secret)
}
```

Objašnjava se:
- Struktura JWT tokena
- Razlika između access i refresh tokena
- Kako se tokeni verificiraju
- Životni vijek tokena i povezane sigurnosne implikacije

## Korisničko iskustvo

Aplikacija je dizajnirana kao interaktivan vodič koji korisnika vodi kroz sljedeće korake:
1. Unos podataka za prijavu
2. Generiranje hash vrijednosti
3. Simulaciju serverske provjere
4. Prihvaćanje kolačića
5. Generiranje JWT tokena
6. Istraživanje sadržaja JWT tokena na jwt.io
7. Završetak procesa prijave

Na svakom koraku, korisnik dobiva objašnjenja i mora izvršiti određenu radnju kako bi nastavio na sljedeći korak. Ova interaktivnost pomaže u razumijevanju koncepata jer korisnik aktivno sudjeluje u procesu.

## Edukativna vrijednost

Projekt ima značajnu edukativnu vrijednost jer:
- Objašnjava važne koncepte web sigurnosti na pristupačan način
- Demonstrira stvarne tehnike koje se koriste u modernim web aplikacijama
- Daje praktični uvid u funkcioniranje autentikacijskih sustava
- Podiže svijest o važnosti sigurnih praksi razvoja web aplikacija

## Zaključak

Ovaj projekt služi kao vrijedan edukativni alat za razumijevanje osnova autentikacije na webu. Kroz interaktivnu demonstraciju, korisnici dobivaju dublje razumijevanje procesa koji se odvijaju kada se prijavljuju na web stranice, što može povećati njihovu svijest o sigurnosti i potencijalno poboljšati njihove navike kao korisnika interneta.

Aplikacija je pogodna za korištenje u obrazovnim institucijama, na radionicama o web razvoju ili za samostalno učenje o osnovama web sigurnosti.