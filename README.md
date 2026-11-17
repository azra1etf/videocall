#  Video poziv – vlastiti signalizacijski protokol i video codec

Softverski proizvod za video pozive jedan na jedan, razvijen u Pythonu. Dva korisnika, svaki na svom računaru, prijavljuju se na registrar, uspostavljaju poziv i vide jedan drugog. Signalizacijski protokol (po uzoru na SIP) i video codec (po uzoru na JPEG / Motion JPEG) implementiramo sami.

---

##  Tim

| Ime i prezime | Uloga |
|---|---|
| Darijo Raca | Product Owner / developer |
| Azra Kovač | Scrum Master / developer |
| Semir Jamaković | Developer |
| Ilhana Balić | Developer |
| Nejra Kozica | Developer |
| Bakir Goran| Developer |
| Kerim Đendušić | Developer |

---

##  Vizija proizvoda

**Za** studente i korisnike u istoj lokalnoj mreži
**koji** žele uspostaviti video poziv jedan na jedan i razumjeti kako takav sistem radi „ispod haube“,
**naš proizvod** je aplikacija za video pozive s vlastitim signalizacijskim protokolom i vlastitim video codecom,
**koja** omogućava registraciju korisnika, pozivanje po imenu, prihvatanje ili odbijanje poziva i dvosmjerni prenos slike s kamere u realnom vremenu.
**Za razliku od** gotovih rješenja (WebRTC, SIP klijenti) koja sakrivaju detalje,
**naš proizvod** je potpuno dokumentovan i transparentan: svaka poruka protokola i svaki korak kompresije slike su naša implementacija, testirani su i mjerljivi (PSNR, protok).

### Ciljevi proizvoda
- Pouzdana uspostava i prekid poziva (prihvaćen, odbijen, zauzet, korisnik ne postoji)
- Video 320×240 pri 10–15 frejmova u sekundi u oba smjera
- Video ide direktno između klijenata, nikad kroz registrar
- Sistem nastavlja raditi i kada se paketi izgube (prikazuje se posljednji ispravan frejm)

---

##  Arhitektura (pregled)

Sistem čine tri komponente:

- **Registrar**: pamti na kojoj je adresi koji korisnik i povezuje pozivaoca s pozvanim
- **Klijent A** i **Klijent B**: registruju se, upućuju i primaju pozive, kodiraju i šalju video

Signalizacija: tekstualne poruke tipa zahtjev–odgovor (REGISTER, INVITE, ACK, BYE + odgovori „zvoni“, „prihvaćeno“, greške).
Video: kodirani frejm se dijeli u pakete (≤ 1400 B, po uzoru na RTP) i šalje UDP-om direktno drugom klijentu.

### Video codec (po uzoru na JPEG)
1. RGB → YCbCr (ITU-R BT.601)
2. Poduzorkovanje hrominancije 4:2:0
3. Blokovi 8×8 + DCT
4. Kvantizacija (tabele + faktor kvaliteta)
5. Cik-cak redoslijed
6. Kompresija pomoću `zlib`

---

##  Tehnologije

- Python 3.11+ (`socket` / `asyncio`)
- NumPy, SciPy (`scipy.fft.dctn`, `idctn`)
- `zlib`
- OpenCV (samo kamera, čitanje videa i prikaz)
- pytest, GitHub Actions, Docker
- Wireshark, `tc netem`

---

##  Način rada (Scrum)

- Rad u **7 iteracija (sprintova)** od po **2 sedmice**
- Svaka iteracija završava **demonstracijom softvera koji radi**
- Backlog i priče vodimo na **GitHub Projects** ploči (kolone: *Backlog → To Do → In Progress → Review → Done*)
- Svaka priča ima **kriterije prihvatanja**
- Na početku iteracije: planiranje; na kraju: demonstracija i retrospektiva

| Iteracija | Tema | Cilj |
|---|---|---|
| 1 | Proizvod, Scrum i Git | vizija, ploča s pričama, repozitorij s README-om |
| 2 | Zahtjevi i arhitektura | persona, priče s kriterijima, specifikacija protokola, ADR o transportu |
| 3 | Linux i API ugovori | registracija i poziv bez slike; codec za jednu sliku + PSNR |
| 4 | Testiranje | testovi mašine stanja i sastavljanja frejma; prenos video fajla |
| 5 | Docker i CI/CD | pravi poziv kamerom, registrar u kontejneru, CI |
| 6 | Pouzdanost i sigurnost | ispadi registrara/klijenta, gubitak paketa, bez tajni u repozitoriju |
| 7 | Cloud i nadzor | statistika poziva, kompletna specifikacija i teorijski dio |

---

##  Git workflow

- `main` je **zaštićena grana**, nema direktnog pusha
- Za svaku priču pravimo **novu granu**:
  - `feature/<kratak-opis>` (npr. `feature/registracija`)
  - `fix/<kratak-opis>` za ispravke
- Promjene ulaze u `main` samo kroz **pull request**
- Svaki pull request mora **pregledati i odobriti** barem jedan član tima
- Poruke commita su kratke i jasne, npr. `Dodaj parsiranje INVITE poruke`

---

##  Prve korisničke priče

1. **Registracija:** Kao korisnik, želim se prijaviti na registrar svojim imenom, kako bi me drugi mogli pozvati.
2. **Pozivanje po imenu:** Kao korisnik, želim pozvati drugog korisnika po imenu, kako bih započeo video poziv.
3. **Prihvatanje / odbijanje:** Kao pozvani korisnik, želim prihvatiti ili odbiti dolazni poziv.
4. **Greške pri pozivu:** Kao pozivalac, želim dobiti poruku ako je korisnik zauzet ili ne postoji.
5. **Video poziv:** Kao korisnik, želim vidjeti sliku s kamere druge strane u prozoru.
6. **Prekid poziva:** Kao korisnik, želim prekinuti poziv, nakon čega se oba klijenta vraćaju u početno stanje.
7. **Kompresija slike:** Kao developer, želim kodirati i dekodirati frejm vlastitim codecom, kako bi video stao u mrežu.
8. **Gubitak paketa:** Kao korisnik, želim da se slika ne „raspadne“ kad se paketi izgube, nego da ostane posljednji ispravan frejm.

Detaljni kriteriji prihvatanja nalaze se na ploči projekta.

---

##  Pokretanje

*U izradi, naredbe za pokretanje registrara i dva klijenta bit će dodane u narednim iteracijama.*

```bash
# registrar (planirano)
docker run ... registrar

# klijenti (planirano)
python client.py --user alice
python client.py --user bob
```

---

##  Literatura

- RFC 3261: SIP: Session Initiation Protocol
- RFC 3264: An Offer/Answer Model with SDP
- RFC 3550: RTP: A Transport Protocol for Real-Time Applications
- RFC 2435: RTP Payload Format for JPEG-compressed Video
- G. K. Wallace, *The JPEG Still Picture Compression Standard*, 1991
- ITU-T T.81: JPEG
