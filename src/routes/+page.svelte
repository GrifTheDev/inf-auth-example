<script lang="ts">
  import sha256 from "crypto-js/sha256";
  import * as jose from 'jose'

  let tutorialState = $state(0);
  let valueBank = $state({
    email: "",
    password: "",
  });
  let cookieConsent = $state("")
  let nameEnding = $state("")
  let hashBank = $state({ hashEmail: "", hashPswd: "" });
  let jwtBank = $state({auth: "", refresh: ""})

  let messageBank: Record<number, string> = $derived({
    0: `Svakodnevno se u životu suočavamo s ovim prizorom - strašna stranica za prijavljivanje. No, jeste li se ikada zapitali kako takav sustav radi?<br/><br/> Za početak, pokušaj upisati neku kombinaciju maila i lozinke. Iako ova stranica ne sprema podatke, molim te nemoj upisati svoje osobne podatke. Kada upišete neke podatke, pritisnite gumb 'Nastavi'.`,
    1: `Upisao si <b>${valueBank.email}</b> za mail i <b>${valueBank.password}</b> za lozinku. Naravno, nije pametno čuvati lozinku i mail kao sami tekst. Ako netko hakira našu bazu podataka, imat će pristup svim lozinkama i mailovima koje upišemo! <br/><br/> Zato ćemo iskoristi tzv. 'hashing alogritam'. Ovaj algoritam za unos nekog teksta uvijek vraća isti izlaz. Jednom kada generiramo tu 'hash vrijednost', ona ne može biti vraćena u prvotni oblik. To znači da ćemo imati vrijednost koju ćemo uvijek moći pridružiti svakom posebnom mailu i lozinci, ali koju nećemo moći razumjeti.<br/><br/> Kako biste generirali svoje 'hash vrijednosti' pomoću SHA-256 algoritma, pritisnite gumb.`,
    2: `Tvoj hash par je: <br/><b>Mail: </b>${hashBank.hashEmail}<br/><b>Lozinka: </b>${hashBank.hashPswd}<br/><br/>Sada nitko ne može shvatiti što smo upisali <i>(iako se za lozinku koristi malo drugačiji algoritam, ali za ovu demonstraciju nema veze)</i>! Sljedeći je korak da server provjeri postoji li takav hash par u bazi podataka te kojem korisniku pripada. Pritisnite gumb kako biste simulirali tu provjeru.`,
    3: `Uspjeh, server je shvatio da unesene vrijednosti pripadaju korisniku <b>Vedranu Kušiću!</b> Bitno je zapamtiti da se svo procesiranje podataka treba odvijati na serveru, jer se web klijenti mogu vrlo lagano manipulirati.<br/><br/>Sada kada smo sigurni da se osoba istinito predstavila, trebamo način na koji možemo pratiti je li korisnik ulogiran, neovisno na kojem dijelu naše stranice se ona nalazi. Kada bismo provjeru prijave pratili samo na ovoj stranici, korisnik bi se trebao ponovno prijaviti na svakoj novoj ruti (zamislite da za svaki YouTube video koji želite pogledati, trebate ponovno proći proces prijave)!<br/><br/> No, rješenje tom problemu nalazi se u jednom pitanju - prihvaćate li kolačiće?`,
    4: `Drago mi je da želiš moje kolačiće :).<br/> Sada kada imamo dopuštenje za korištenje kolačića, moramo ponovno spremiti neki <b>anonimni</b> podatak (kolačići se svakodnevno kradu) koji naš server može pročitati kada korisnik želi preći na rutu koja zahtjeva prijavu. Kada server pročita tu vrijednost, mora ponovno zaviriti u bazu podataka i vidjeti komu ta vrijednost pripada. Nazovimo tu vrijednost <b>token</b>.<br/><br/>Ovaj sustav možemo zamisliti kao pomoćni sustav za prijavu, jer zapravo pretražuje kodirane podatke kao i u prvom koraku. Jedina je razlika što ovi kodirani podatci nisu derivirani od korisnikove lozinke i maila te nisu šifrirani.<br/><br/>Za formu tokena koristit ćemo <b>JSON Web Token (JWT)</b> standard. Ova vrsta kodiranja (ne šifriranja!) podataka, dopušta nam da u jednostavan komad teksta sakrijemo podatke i mehanizam za provjeru njihove validnosti.`,
    5: `Tvoj par JWT tokena je spreman: <br/><b>Access Token: </b>${jwtBank.auth}<br/><b>Refresh Token: </b>${jwtBank.refresh}<br/><br/>Kako to da imamo par tokena? Zbog istog straha od početka, ako netko ukrade token i zamijeni svoj token s tuđim, može se predstaviti kao netko drugi (jer sustav pomoćne prijave samo provjerava je li token validan). Zato koristimo <b>par tokena</b>. Jedan token, access token, koristi se za provjeru identiteta korisnika. To je token za koji server po bazi podataka traži validnog korisnika. Taj token ima životni vijek od 2 min (što kraće to bolje). Kada to vrijeme istekne, taj token više nije validan te ga sustav odbija primiti. Tu dolazi drugi token, refresh token. On vrijedi 1 tjedan te se, kada server pokuša iskoristiti access token koji nije validan, on konzumira kako bi se stvorio novi access token, odnosno novi par access i refresh tokena. Pritisni gumb kako bi se JWT token par spremio u kolačiće.`,
    6: `Prije toga, moramo proći još jednu bitnu stvar. Kako programeri vole biti efikasni, nakon nekoliko desetljeća boravljenja samo u svojim špiljama, izumili su tehnologiju koja dopušta da se u JWT spreme bilo koji podatci te da se njihova validnost može provjeriti s ključem. Kako biste završili proces prijave, otiđite na web stranicu <a href="https://jwt.io" style="color:cyan" target="_blank">jwt.io</a> te u kućicu za JWT token (Encoded Value) unesite svoj <b>access token</b>, zatim u prozoru desno (Decoded Payload) pročitajte koji su podatci skriveni u tokenu.<br/><br/> Možda primjetite da stranica pokazuje kako je verifikacija validnosti podataka neuspješna (crveno: Invalid Signature). U kućicu za provjeru validnosti podataka (JWT Signature Verification) unesite riječ 'kljuc'.<br/><br/> Kako bi završili prijavu, unesite ime koje se pokazuje u podacima skrivenima u tokenu. <br/><b>Access Token: </b>${jwtBank.auth}`    
});

  function generateHashes() {
    hashBank.hashEmail = sha256(valueBank.email).toString();
    hashBank.hashPswd = sha256(valueBank.password).toString();
  }

  async function generateJWT() {
    const secret = new TextEncoder().encode("kljuc")

    jwtBank.auth = await new jose.SignJWT({ime_koje_trazis: "branko"}).setProtectedHeader({ "alg": "HS256" }).setIssuedAt().sign(secret)
    jwtBank.refresh = await new jose.SignJWT({kolacici: "da ne mozda"}).setProtectedHeader({ "alg": "HS256" }).setIssuedAt().sign(secret)
  }
</script>

<div class="h-screen flex flex-col items-center justify-center bg-slate-800">
  {#if tutorialState < 7}

  
  <form
    class="rounded-lg border-white border-2 w-[75%] h-4/5 flex flex-col items-center justify-center bg-dashboard-bg"
  >
    <h1 class="text-5xl font-bold m-3 text-center text-white">TeoKorporacija d.o.o. - Prijava</h1>
    <label class="w-full flex flex-col items-center justify-center">
      <p class="text-xl font-thin text-white">E-pošta</p>
      <input
        class="static rounded-[3px] w-2/3 h-10 pl-2 pr-2 text-center text-lg text-white font-light border-2 border-gray-400 bg-transparent focus:outline-none [appearance:textfield] [&::-webkit-outer-spin-button]:appearance-none [&::-webkit-inner-spin-button]:appearance-none"
        name="email"
        type="email"
        bind:value={valueBank.email}
      />
    </label>
    <label class="w-full flex flex-col items-center justify-center">
      <p class="text-xl font-thin text-white">Lozinka</p>
      <input
        class="static rounded-[3px] w-2/3 h-10 pl-2 pr-2 mb-4 text-center text-lg text-white font-light border-2 border-gray-400 bg-transparent focus:outline-none [appearance:textfield] [&::-webkit-outer-spin-button]:appearance-none [&::-webkit-inner-spin-button]:appearance-none"
        name="password"
        type="password"
        bind:value={valueBank.password}
      />
    </label>

    <div
      class="rounded-lg border-white border-2 border-dashed w-[80%] flex flex-col items-center justify-center bg-dashboard-bg p-3 my-2"
    >
      <h1 class="text-white text-center font-bold text-2xl">👋Vodič👋</h1>
      <p class="text-white text-center font-medium break-normal">{@html messageBank[tutorialState]}</p>

      {#if tutorialState == 0}
        <button
          class="text-white text-center text-lg border-[3px] bg-transparent border-white rounded-md py-2 px-6 mt-2 transition ease-in-out duration-200 disabled:opacity-50 disabled:border-gray-400 enabled:hover:scale-105 enabled:hover:shadow-whiteGlowPlus enabled:hover:-rotate-1 rotate enabled:active:scale-95 enabled:active:rotate-1 enabled:active:shadow-none"
          disabled={Object.values(valueBank).includes("")}
          onclick={() => {
            tutorialState += 1;
          }}
        >
          Nastavi
        </button>
      {:else if tutorialState == 1}
        <button
          class="text-white text-center text-lg border-[3px] bg-transparent border-white rounded-md py-2 px-6 mt-2 transition ease-in-out duration-200 disabled:opacity-50 disabled:border-gray-400 enabled:hover:scale-105 enabled:hover:shadow-whiteGlowPlus enabled:hover:-rotate-1 rotate enabled:active:scale-95 enabled:active:rotate-1 enabled:active:shadow-none"
          onclick={() => {
            generateHashes();
            tutorialState += 1;
          }}
        >
          Nastavi
        </button>
      {:else if tutorialState == 2}
        <button
          class="text-white text-center text-lg border-[3px] bg-transparent border-white rounded-md py-2 px-6 mt-2 transition ease-in-out duration-200 disabled:opacity-50 disabled:border-gray-400 enabled:hover:scale-105 enabled:hover:shadow-whiteGlowPlus enabled:hover:-rotate-1 rotate enabled:active:scale-95 enabled:active:rotate-1 enabled:active:shadow-none"
          onclick={() => {
            tutorialState += 1;
          }}
        >
          Nastavi
        </button>
        {:else if tutorialState == 3}

        <p class="text-white text-center font-medium break-words" style="user-select:none">Molim te upiši "Ja obožavam i prihvaćam tvoje kolačiće" za nastavak.</p>
        <input
        class="static rounded-[3px] w-2/3 h-10 pl-2 pr-2 text-center text-lg text-white font-light border-2 border-gray-400 bg-transparent focus:outline-none [appearance:textfield] [&::-webkit-outer-spin-button]:appearance-none [&::-webkit-inner-spin-button]:appearance-none"
        type="text"
        bind:value={cookieConsent}
      />
        <button
          class="text-white text-center text-lg border-[3px] bg-transparent border-white rounded-md py-2 px-6 mt-2 transition ease-in-out duration-200 disabled:opacity-50 disabled:border-gray-400 enabled:hover:scale-105 enabled:hover:shadow-whiteGlowPlus enabled:hover:-rotate-1 rotate enabled:active:scale-95 enabled:active:rotate-1 enabled:active:shadow-none"
          disabled={cookieConsent.toLowerCase() != "ja obožavam i prihvaćam tvoje kolačiće"}
          onclick={() => {
            tutorialState += 1;
          }}
        >
          {cookieConsent.toLowerCase() != "ja obožavam i prihvaćam tvoje kolačiće" ? "Nastavi" : "Tvoji kolačići su najbolji"} 
        </button>
        {:else if tutorialState == 3}
        <button
          class="text-white text-center text-lg border-[3px] bg-transparent border-white rounded-md py-2 px-6 mt-2 transition ease-in-out duration-200 disabled:opacity-50 disabled:border-gray-400 enabled:hover:scale-105 enabled:hover:shadow-whiteGlowPlus enabled:hover:-rotate-1 rotate enabled:active:scale-95 enabled:active:rotate-1 enabled:active:shadow-none"
          onclick={() => {
            tutorialState += 1;
          }}
        >
          Nastavi
        </button>    
        {:else if tutorialState == 4}
        <button
          class="text-white text-center text-lg border-[3px] bg-transparent border-white rounded-md py-2 px-6 mt-2 transition ease-in-out duration-200 disabled:opacity-50 disabled:border-gray-400 enabled:hover:scale-105 enabled:hover:shadow-whiteGlowPlus enabled:hover:-rotate-1 rotate enabled:active:scale-95 enabled:active:rotate-1 enabled:active:shadow-none"
          onclick={async () => {
            await generateJWT()
            tutorialState += 1;
          }}
        >
          Nastavi
          
        </button>
        {:else if tutorialState == 5}
        <button
          class="text-white text-center text-lg border-[3px] bg-transparent border-white rounded-md py-2 px-6 mt-2 transition ease-in-out duration-200 disabled:opacity-50 disabled:border-gray-400 enabled:hover:scale-105 enabled:hover:shadow-whiteGlowPlus enabled:hover:-rotate-1 rotate enabled:active:scale-95 enabled:active:rotate-1 enabled:active:shadow-none"
          onclick={async () => {
            await generateJWT()
            tutorialState += 1;
          }}
        >
          Spremi te ukusne kolačiće!
          
        </button>
        {:else if tutorialState == 6}

        <input
        class="static rounded-[3px] w-2/3 h-10 pl-2 pr-2 text-center text-lg text-white font-light border-2 border-gray-400 bg-transparent focus:outline-none [appearance:textfield] [&::-webkit-outer-spin-button]:appearance-none [&::-webkit-inner-spin-button]:appearance-none"
        type="text"
        bind:value={nameEnding}
      />
        <button
          class="text-white text-center text-lg border-[3px] bg-transparent border-white rounded-md py-2 px-6 mt-2 transition ease-in-out duration-200 disabled:opacity-50 disabled:border-gray-400 enabled:hover:scale-105 enabled:hover:shadow-whiteGlowPlus enabled:hover:-rotate-1 rotate enabled:active:scale-95 enabled:active:rotate-1 enabled:active:shadow-none"
          disabled={nameEnding.toLowerCase() != "branko"}
          onclick={() => {
            tutorialState += 1;
          }}
        >
          {nameEnding.toLowerCase() != "branko" ? "Nekima bi ovo moglo biti poznato..." : "Završi (hvala Branko😘)"} 
        </button>
      {/if}
    </div>
  </form>
  {:else}
  <h1 class="text-5xl font-bold m-3 text-center text-white">Uspješno ste se prijavili!</h1>
  {/if}
</div>
