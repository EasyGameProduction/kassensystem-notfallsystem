<template>
    <div id="kasse" v-if="!this.adminViewActive">
        <div id="Kassensystem2">
            <div id="Bestellung">
                <div class="ElementeContainer">
                    <div class="AuswahlItem" v-for="a in this.ausgewaehlteArtikel" :key="a.kassenProjektArtikelID" @click="artikelEntfernen(a)">
                        <div class="textItem">
                            <div>
                                <span class="Anzahl">{{ a.anzahl }}x</span>
                                <span class="Gericht">{{ a.artikelName }}</span>
                            </div>
                            <span class="Preis">{{ a.preis }} €</span>
                        </div>
                        <div class="textItem" v-if="a.kassenProjektPfandID > 0">
                            <div>
                                <span class="Pfand">Pfand: </span>
                            </div>
                            <span class="Preis">{{ (this.pfand[this.pfand.findIndex(art2 => art2.kassenProjektPfandID == a.kassenProjektPfandID)].pfandPreis * a.anzahl).toFixed(2) }} €</span>
                        </div>
                    </div>
                    <div class="AuswahlItemPfand" v-for="p in this.ausgewaehltePfand" :key="p.kassenProjektPfandID" @click="pfandEntfernen(p)">
                        <div class="textItem">
                            <div>
                                <span class="Anzahl">{{ p.anzahl }}x</span>
                                <span class="Gericht">{{ p.pfandName }}</span>
                            </div>
                            <span class="Preis">{{ p.preis }} €</span>
                        </div>
                    </div>
                </div>
            </div>
            <div id="AuswahlTerminal">
                <div id="auswahlContainer" v-if="!this.bearbeitungsmodusPfand">
                    <div class="gerichtItem" :class="(auswahlmodus)?((getArtikelActive(a))?'artikelAktiv':'artikelNichtAktiv'):''" v-for="a in this.artikel" :key="a.kassenProjektArtikelID" @click="artikelKlick(a)" v-show="(auswahlmodus)? true : getArtikelActive(a)" :style="{ width: this.articleWidth, height: this.articleHeight, background: this.articleColors[a.kassenProjektArtikelID] }">
                        <div class="gerichtItemBezeichnung" :style="{ fontSize: this.articleFontSize }">
                            {{ a.artikelName }}
                        </div>
                        <div class="gerichtItemPreis">
                            <span :style="{ fontSize: this.articlePriceSize }">{{ a.artikelPreis }} €</span>
                        </div>
                        <div v-if="this.bearbeitungsmodus">
                            <button @click="artikelBearbeiten(a)">Bearbeiten</button>
                            <button @click="artikelLoeschen(a)">Löschen</button>
                            <br><br>
                            <button @click="nachLinksVerschieben(a)" style="margin-right: 15px;"><span class="fa">&#xf060;</span></button>
                            <input type="color" v-model="articleColors[a.kassenProjektArtikelID]" @input="updateColor(a)">
                            <button @click="nachRechtsVerschieben(a)" style="margin-left: 15px;"><span class="fa">&#xf061;</span></button>
                        </div>
                    </div>
                    <div class="gerichtItem" v-if="this.bearbeitungsmodus" @click="artikelHinzufuegen()">
                        <button class="hinzufuegenButton">+</button>
                    </div>
                </div>
                <div id="pfandContainer" v-if="!this.bearbeitungsmodus">
                    <div class="pfandItem" :class="(auswahlmodus)?((getPfandActive(p))?'pfandAktiv':'pfandNichtAktiv'):''" v-for="p in this.pfand" :key="p.kassenProjektPfandID" @click="pfandKlick(p)" v-show="(auswahlmodus)? true : getPfandActive(p)">
                        <div class="pfandItemBezeichnung" :style="{ fontSize: this.pfandFontSize }">
                            {{ p.pfandName }}
                        </div>
                        <div class="pfandItemPreis">
                            <span :style="{ fontSize: this.pfandPriceSize }">-{{ p.pfandPreis }} €</span>
                        </div>
                        <div v-if="this.bearbeitungsmodusPfand">
                            <button @click="pfandBearbeiten(p)">Bearbeiten</button>
                            <button @click="pfandLoeschen(p)">Löschen</button>
                        </div>
                    </div>
                    <div class="pfandItem" v-if="this.bearbeitungsmodusPfand" @click="pfandHinzufuegen()">
                        <button class="hinzufuegenButton">+</button>
                    </div>
                </div>
            </div>
        </div>
        <div class="RechnungsContainer">
            <div id="rechnungsTextContainer">
                <div>Rechnungsbetrag</div>
                <div id="rechnungsbetrag">{{ this.rechnungsbetrag }} €</div>
            </div>
            <div id="buttonContainer">
                <button id="clear" @click="this.clear()">
                    clear
                </button>
                <button id="bestaetigen" @click="this.bestaetigen()">
                    bestätigen
                </button>
            </div>
        </div>
        <button id="menuButtonKassensystem" class="fa" @click="this.menuOpen = !this.menuOpen">&#xf0c9;</button>
        <div id="menu" v-if="this.menuOpen">
            <button class="menuButton" @click="this.changeName()"><span>Name ändern</span><span class="fa">&#xf105;</span></button>
            <button class="menuButton" @click="this.bearbeitungsmodus = !this.bearbeitungsmodus; this.menuOpen = !this.menuOpen"><span>Artikel bearbeiten</span><span class="fa">&#xf105;</span></button>
            <button class="menuButton" @click="this.bearbeitungsmodusPfand = !this.bearbeitungsmodusPfand; this.menuOpen = !this.menuOpen"><span>Pfand bearbeiten</span><span class="fa">&#xf105;</span></button>
            <button class="menuButton" @click="this.auswahlmodus = !this.auswahlmodus; this.menuOpen = !this.menuOpen"><span>Auswahlmodus</span><span class="fa">&#xf105;</span></button>
            <button class="menuButton" @click="this.deleteProjekt()"><span>Projekt löschen</span><span class="fa">&#xf105;</span></button>
            <button class="menuButton" @click="this.openSettings()"><span>Einstellungen</span><span class="fa">&#xf105;</span></button>
            <button class="menuButton" @click="this.artikelstaendeUebermitteln(true)"><span>Artikelstände speichern</span><span class="fa">&#xf105;</span></button>
            <button class="menuButton" @click="this.$emit('openKassensystemAuswahl')"><span>Kassensystem beenden</span><span class="fa">&#xf105;</span></button>
            <button class="menuButton" @click="this.openAdminView()"><span>Adminkonsole</span><span class="fa">&#xf105;</span></button>
        </div>
        <div id="menuBackground" @click="this.menuOpen = !this.menuOpen" v-if="this.menuOpen"></div>
        <div id="rechner" v-if="this.rechner">
            <div class="display"><p>Betrag:</p><p id="rechnerBetrag">{{ this.rechnerBetrag }} €</p></div>
            <div class="display"><p>Bekommen:</p><p id="rechnerBekommen">{{ rechnerBekommen ? parseFloat(rechnerBekommen).toFixed(2) : '0.00' }} €</p></div>
            <div class="display"><p>Geben:</p><p id="rechnerGeben" :style="(rechnerGeben < 0)?'color: red':(rechnerGeben == 0)?'color: green':'color: blue'">{{ rechnerGeben }} €</p></div>
            <button class="rechnerButton" @click="rechnerButtonClick('7')">7</button>
            <button class="rechnerButton" @click="rechnerButtonClick('8')">8</button>
            <button class="rechnerButton" @click="rechnerButtonClick('9')">9</button>
            <button class="rechnerButton" @click="rechnerButtonClick('+10')">+10</button>
            <button class="rechnerButton" @click="rechnerButtonClick('4')">4</button>
            <button class="rechnerButton" @click="rechnerButtonClick('5')">5</button>
            <button class="rechnerButton" @click="rechnerButtonClick('6')">6</button>
            <button class="rechnerButton" @click="rechnerButtonClick('+20')">+20</button>
            <button class="rechnerButton" @click="rechnerButtonClick('1')">1</button>
            <button class="rechnerButton" @click="rechnerButtonClick('2')">2</button>
            <button class="rechnerButton" @click="rechnerButtonClick('3')">3</button>
            <button class="rechnerButton" @click="rechnerButtonClick('+50')">+50</button>
            <button class="rechnerButton" @click="rechnerButtonClick('C')">C</button>
            <button class="rechnerButton" @click="rechnerButtonClick('0')">0</button>
            <button class="rechnerButton" @click="rechnerButtonClick('.')">.</button>
            <button class="rechnerButton" @click="rechnerButtonClick('ENDE')">ENDE</button>
        </div>
        <div id="rechnerBackground" @click="this.rechner = !this.rechner" v-if="this.rechner"></div>
    </div>
    
    <!--<button id="homeButton" v-if="!this.adminViewActive" @click="this.$emit('openKassensystemAuswahl')" class="mapButton"><img src="@/assets/Icons/HomeButton.svg"></button>-->
    <button id="homeButton" v-if="this.adminViewActive" @click="this.openAdminView()" class="mapButton"><img src="@/assets/Icons/HomeButton.svg"></button>
</template>

<script>
import Swal from 'sweetalert2';

export default {
  name: "HomeView",
  components: {
    
  },
  props: {
    
  },
  data() {
    return {
      kassenProjekt: [],
artikel: [
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 1,
    artikelName: 'Bier',
    artikelPreis: '4.00',
    kassenProjektPfandID: 2
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 2,
    artikelName: 'Maß Bier',
    artikelPreis: '8.00',
    kassenProjektPfandID: 2
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 3,
    artikelName: 'Hefe',
    artikelPreis: '4.00',
    kassenProjektPfandID: 2
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 4,
    artikelName: 'Bier 0%',
    artikelPreis: '4.00',
    kassenProjektPfandID: 2
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 5,
    artikelName: 'Hefe 0%',
    artikelPreis: '4.00',
    kassenProjektPfandID: 2
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 6,
    artikelName: 'Limo',
    artikelPreis: '2.00',
    kassenProjektPfandID: 1
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 7,
    artikelName: 'Wasser',
    artikelPreis: '1.50',
    kassenProjektPfandID: 1
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 8,
    artikelName: 'Schoppen',
    artikelPreis: '3.50',
    kassenProjektPfandID: 3
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 9,
    artikelName: 'Wein-Flasche',
    artikelPreis: '15.00',
    kassenProjektPfandID: 3
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 10,
    artikelName: 'Schorle 0,25l',
    artikelPreis: '3.50',
    kassenProjektPfandID: 3
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 11,
    artikelName: 'Schorle 0,5l',
    artikelPreis: '7.00',
    kassenProjektPfandID: 2
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 12,
    artikelName: 'Aperol 0,25l',
    artikelPreis: '4.50',
    kassenProjektPfandID: 3
  },
  {
    kassenProjektID: 2,
    feuerwehrID: 1,
    kassenProjektArtikelID: 13,
    artikelName: 'Aperol 0,5l',
    artikelPreis: '7.50',
    kassenProjektPfandID: 2
  }
],
        aktiveArtikel: [],
        pfand: [{
          kassenProjektID: 2,
          feuerwehrID: 1,
          kassenProjektPfandID: 1,
          pfandName: 'PET',
          pfandPreis: '1.00'
        },
        {
          kassenProjektID: 2,
          feuerwehrID: 1,
          kassenProjektPfandID: 2,
          pfandName: 'Krug',
          pfandPreis: '3.00'
        },
      {
        kassenProjektID: 2,
          feuerwehrID: 1,
          kassenProjektPfandID: 3,
          pfandName: 'Glas/Flasche',
          pfandPreis: '3.00'
      }],
        aktivePfand: [],
        ausgewaehlteArtikel: [],
        ausgewaehltePfand: [],
        rechnungsbetrag: (0.00).toFixed(2),
        bearbeitungsmodus: false,
        bearbeitungsmodusPfand: false,
        auswahlmodus: false,
        menuOpen: false,
        adminViewActive: false,
        rechner: false,
        rechnerBekommen: '',
        rechnerBetrag: '',
        articleSize: parseInt(localStorage.getItem('KassensystemArtikelGroeße')),
        articleFontSize: localStorage.getItem('KassensystemArtikelBezeichnerGroeße'),
        articlePriceSize: localStorage.getItem('KassensystemArtikelPreisGroeße'),
        articleWidth: localStorage.getItem('KassensystemArtikelWeite'),
        articleHeight: localStorage.getItem('KassensystemArtikelHoehe'),
        pfandSize: parseInt(localStorage.getItem('KassensystemPfandGroeße')),
        pfandFontSize: localStorage.getItem('KassensystemPfandBezeichnerGroeße'),
        pfandPriceSize: localStorage.getItem('KassensystemPfandPreisGroeße'),
        articleColors: [],
        artikelReihenfolge: [],
    };
  },
  computed: {
    // Berechnet bei jeder Änderung automatisch „Betrag – Bekommen“
    rechnerGeben() {
      const betrag = parseFloat(this.rechnerBetrag) || 0;
      const bekommen = parseFloat(this.rechnerBekommen) || 0;
      return (bekommen - betrag).toFixed(2);
    }
  },
  methods: {
    arraySortieren(){
        console.log("Test");
        var reihenfolge = JSON.parse(localStorage.getItem("kassenprojektArtikelReihenfolge: kassenProjektID: " + this.kassenProjekt.kassenProjektID));
        if(reihenfolge == null){
            localStorage.setItem("kassenprojektArtikelReihenfolge: kassenProjektID: " + this.kassenProjekt.kassenProjektID, JSON.stringify(this.artikel));
            return;
        }
        var artikelTemp = [];
        var neueArtikel = [];
        this.artikel.forEach(a => {
            var r = reihenfolge.filter(c => c.kassenProjektID == a.kassenProjektID && c.kassenProjektArtikelID == a.kassenProjektArtikelID);
            if(r != null){
                let index = reihenfolge.indexOf(r[0]);
                if(index == -1){
                    neueArtikel.push(a);
                } else{
                    artikelTemp[index] = a;
                }
            } else{
                neueArtikel.push(a);
            }
        });
        neueArtikel.forEach(n => {
            artikelTemp.push(n);
        });
        this.artikel = artikelTemp;
    },
    nachLinksVerschieben(a){
        var index = this.artikel.indexOf(a);
        let artikelTemp = a;
        if(index > 0){
            this.artikel[index] = this.artikel[index - 1];
            this.artikel[index - 1] = artikelTemp;
        }
        localStorage.setItem("kassenprojektArtikelReihenfolge: kassenProjektID: " + this.kassenProjekt.kassenProjektID, JSON.stringify(this.artikel));
    },
    nachRechtsVerschieben(a){
        var index = this.artikel.indexOf(a);
        let artikelTemp = a;
        if(index < this.artikel.length - 1){
            this.artikel[index] = this.artikel[index + 1];
            this.artikel[index + 1] = artikelTemp;
        }
        localStorage.setItem("kassenprojektArtikelReihenfolge: kassenProjektID: " + this.kassenProjekt.kassenProjektID, JSON.stringify(this.artikel));
    },
    updateColor(a){
        localStorage.setItem("ArtikelFarbe: KassenprojektID: " + a.kassenProjektID, JSON.stringify(this.articleColors));
    },
    setColors(){
        this.articleColors = JSON.parse(localStorage.getItem("ArtikelFarbe: KassenprojektID: " + this.kassenProjekt.kassenProjektID));
        if(this.articleColors == null){
            this.articleColors = [];
        }
    },
    clearSettings(){
        this.articleSize = 0;
        localStorage.setItem('KassensystemArtikelGroeße', this.articleSize);
        this.setArticleFontSize('');
        this.setArticlePriceSize('');
        this.setArticleWidthAndHeight('', '')

        this.pfandSize = 0;
        localStorage.setItem('KassensystemPfandGroeße', this.pfandSize);
        this.setPfandFontSize('');
        this.setPfandPriceSize('');
    },
    openSettings() {
      var current = this.articleSize;
      if(current >= 0){

      } else{
        current = 0;
      }
      var currentPfand = this.pfandSize;
      if(currentPfand >= 0){

      } else{
        currentPfand = 0;
      }
      Swal.fire({
        title: 'Einstellungen',
        html:
          `<label for="artikelSizeSlider">Artikelgröße: <span id="sizeValue">${current}</span></label><br>` +
          `<input type="range" id="artikelSizeSlider" min="1" max="6" step="1" value="${current}" ` +
          `oninput="document.getElementById('sizeValue').innerText = this.value"/>` +
        `<br>`+
          `<label for="pfandSizeSlider">Pfandgröße: <span id="pfandSizeValue">${currentPfand}</span></label><br>` +
          `<input type="range" id="pfandSizeSlider" min="1" max="4" step="1" value="${currentPfand}" ` +
          `oninput="document.getElementById('pfandSizeValue').innerText = this.value"/>`,
        showCancelButton: true,
        showDenyButton: true,
        confirmButtonText: 'Speichern',
        denyButtonText: 'Zurücksetzen',
        preConfirm: () => {
          return { artikel: document.getElementById('artikelSizeSlider').value, 
                   pfand: document.getElementById('pfandSizeSlider').value };
        }
      }).then((result) => {
        if (result.isConfirmed) {
          var resultObj = result.value;
          this.articleSize = parseInt(resultObj.artikel);
          this.pfandSize = parseInt(resultObj.pfand);
          localStorage.setItem('KassensystemArtikelGroeße', this.articleSize);
          localStorage.setItem('KassensystemPfandGroeße', this.pfandSize);
          switch(this.articleSize){
            case 1:
                this.setArticleFontSize('20px');
                this.setArticlePriceSize('16px');
                this.setArticleWidthAndHeight('21vw', '3rem')
                break;
            case 2:
                this.setArticleFontSize('23px');
                this.setArticlePriceSize('19px');
                this.setArticleWidthAndHeight('21vw', 'auto')
                break;
            case 3:
                this.setArticleFontSize('26px');
                this.setArticlePriceSize('22px');
                this.setArticleWidthAndHeight('21vw', 'auto')
                break;
            case 4:
                this.setArticleFontSize('29px');
                this.setArticlePriceSize('25px');
                this.setArticleWidthAndHeight('21vw', 'auto')
                break;
            case 5:
                this.setArticleFontSize('32px');
                this.setArticlePriceSize('28px');
                this.setArticleWidthAndHeight('30vw', 'auto')
                break;
            case 6:
                this.setArticleFontSize('35px');
                this.setArticlePriceSize('31px');
                this.setArticleWidthAndHeight('30vw', 'auto')
                break;
          }

          switch(this.pfandSize){
            case 1:
                this.setPfandFontSize('20px');
                this.setPfandPriceSize('16px');
                break;
            case 2:
                this.setPfandFontSize('23px');
                this.setPfandPriceSize('19px');
                break;
            case 3:
                this.setPfandFontSize('26px');
                this.setPfandPriceSize('22px');
                break;
            case 4:
                this.setPfandFontSize('29px');
                this.setPfandPriceSize('25px');
                break;
            case 5:
                this.setPfandFontSize('32px');
                this.setPfandPriceSize('28px');
                break;
            case 6:
                this.setPfandFontSize('35px');
                this.setPfandPriceSize('31px');
                break;
          }
        } else if(result.isDenied){
            this.clearSettings();
        }
        this.menuOpen = false;
      });
    },
    setArticleFontSize(size){
        this.articleFontSize = size;
        localStorage.setItem('KassensystemArtikelBezeichnerGroeße', size)
    },
    setArticlePriceSize(size){
        this.articlePriceSize = size;
        localStorage.setItem('KassensystemArtikelPreisGroeße', size)
    },
    setArticleWidthAndHeight(sizeW,sizeH){
        this.articleWidth = sizeW;
        localStorage.setItem('KassensystemArtikelWeite', sizeW)
        this.articleHeight = sizeH;
        localStorage.setItem('KassensystemArtikelHoehe', sizeH)
    },
    setPfandFontSize(size){
        this.pfandFontSize = size;
        localStorage.setItem('KassensystemPfandBezeichnerGroeße', size)
    },
    setPfandPriceSize(size){
        this.pfandPriceSize = size;
        localStorage.setItem('KassensystemPfandPreisGroeße', size)
    },
    async openAdminView(){
        try{
            this.artikel = await KassenprojektArtikel.get(this.kassenProjekt.kassenProjektID, this.kassenProjekt.feuerwehrID);
            localStorage.setItem("kassenprojektArtikel", JSON.stringify(this.artikel));
            this.pfand = await KassenprojektPfand.get(this.kassenProjekt.kassenProjektID, this.kassenProjekt.feuerwehrID);
            localStorage.setItem("kassenprojektPfand", JSON.stringify(this.pfand));
        } catch(err){
            console.log(err);
            this.artikel = JSON.parse(localStorage.getItem("kassenprojektArtikel"));
            this.pfand = JSON.parse(localStorage.getItem("kassenprojektPfand"));
        }
        this.adminViewActive = !this.adminViewActive;
    },
    artikelKlick(artikel){
        if(this.bearbeitungsmodus){
            return;
        } else if(this.auswahlmodus){
            let artikelObj = {
                kassenProjektID: artikel.kassenProjektID,
                kassenProjektArtikelID: artikel.kassenProjektArtikelID,
                active: !(this.getArtikelActive(artikel))
            }
            localStorage.setItem("AktiveArtikel: KassenprojektID: " + artikelObj.kassenProjektID + " - KassenprojektArtikelID: " + artikelObj.kassenProjektArtikelID, JSON.stringify(artikelObj));
            this.setAktiveArtikel();
            this.auswahlmodus = false;
            this.auswahlmodus = true;
            return;
        }
        if(this.ausgewaehlteArtikel.some(art => art.kassenProjektArtikelID == artikel.kassenProjektArtikelID)){
            var index = this.ausgewaehlteArtikel.findIndex(art2 => art2.kassenProjektArtikelID == artikel.kassenProjektArtikelID);

            this.ausgewaehlteArtikel[index].anzahl += +1;
            this.ausgewaehlteArtikel[index].preis = (+this.ausgewaehlteArtikel[index].preis + +artikel.artikelPreis).toFixed(2);
        }
        else{
            var ausgewaehltesArtikel = {
                kassenProjektArtikelID: artikel.kassenProjektArtikelID,
                artikelName: artikel.artikelName,
                preis: artikel.artikelPreis,
                imgSrc: artikel.imgSrc,
                anzahl: 1,
                kassenProjektPfandID: artikel.kassenProjektPfandID
            };

            this.ausgewaehlteArtikel.push(ausgewaehltesArtikel);
        }

        this.rechnungsbetrag = (+this.rechnungsbetrag + +artikel.artikelPreis).toFixed(2);

        if(artikel.kassenProjektPfandID > 0){
            this.rechnungsbetrag = (+this.rechnungsbetrag + +(this.pfand[this.pfand.findIndex(art2 => art2.kassenProjektPfandID == artikel.kassenProjektPfandID)].pfandPreis)).toFixed(2);
        }
    },
    addPfand(pfandID){
        var pfand = this.pfand[this.pfand.findIndex(art2 => art2.kassenProjektPfandID == pfandID)];
        if(pfand == undefined){
            return;
        }
        if(this.ausgewaehltePfand.some(art => art.kassenProjektPfandID == pfand.kassenProjektPfandID)){
            var index = this.ausgewaehltePfand.findIndex(art2 => art2.kassenProjektPfandID == pfand.kassenProjektPfandID);

            this.ausgewaehltePfand[index].anzahl += +1;
            this.ausgewaehltePfand[index].preis = (+this.ausgewaehltePfand[index].preis + +pfand.pfandPreis).toFixed(2);
        }
        else{
            var ausgewaehltesPfand = {
                kassenProjektPfandID: pfand.kassenProjektPfandID,
                pfandName: pfand.pfandName,
                preis: pfand.pfandPreis,
                anzahl: 1
            };

            this.ausgewaehltePfand.push(ausgewaehltesPfand);
        }

        this.rechnungsbetrag = (+this.rechnungsbetrag + +pfand.pfandPreis).toFixed(2);
    },
    pfandKlick(pfand){
        if(this.bearbeitungsmodusPfand){
            return;
        } else if(this.auswahlmodus){
            let pfandObj = {
                kassenProjektID: pfand.kassenProjektID,
                kassenProjektPfandID: pfand.kassenProjektPfandID,
                active: !(this.getPfandActive(pfand))
            }
            localStorage.setItem("AktivePfand: KassenprojektID: " + pfandObj.kassenProjektID + " - KassenprojektPfandID: " + pfandObj.kassenProjektPfandID, JSON.stringify(pfandObj));
            this.setAktivePfand();
            this.auswahlmodus = false;
            this.auswahlmodus = true;
            return;
        }
        if(this.ausgewaehltePfand.some(art => art.kassenProjektPfandID == pfand.kassenProjektPfandID)){
            var index = this.ausgewaehltePfand.findIndex(art2 => art2.kassenProjektPfandID == pfand.kassenProjektPfandID);

            this.ausgewaehltePfand[index].anzahl -= +1;
            this.ausgewaehltePfand[index].preis = (+this.ausgewaehltePfand[index].preis - +pfand.pfandPreis).toFixed(2);
            if(this.ausgewaehltePfand[index].anzahl == 0){
                this.ausgewaehltePfand.splice(index, 1);
            }
        }
        else{
            var ausgewaehltesPfand = {
                kassenProjektPfandID: pfand.kassenProjektPfandID,
                pfandName: pfand.pfandName,
                preis: -pfand.pfandPreis,
                anzahl: -1
            };

            ausgewaehltesPfand.preis = (ausgewaehltesPfand.preis).toFixed(2);

            this.ausgewaehltePfand.push(ausgewaehltesPfand);
        }


        this.rechnungsbetrag = (+this.rechnungsbetrag - +pfand.pfandPreis).toFixed(2);
    },
    artikelEntfernen(artikel){
        var index = this.ausgewaehlteArtikel.findIndex(art => art.kassenProjektArtikelID == artikel.kassenProjektArtikelID);

        if(this.ausgewaehlteArtikel[index].anzahl == 1){
            this.ausgewaehlteArtikel.splice(index, 1);
        } else{
            this.ausgewaehlteArtikel[index].anzahl -= +1;
            this.ausgewaehlteArtikel[index].preis = (+this.ausgewaehlteArtikel[index].preis - +this.artikel[this.artikel.findIndex(art2 => art2.kassenProjektArtikelID == artikel.kassenProjektArtikelID)].artikelPreis).toFixed(2)
        }
        this.rechnungsbetrag = (this.rechnungsbetrag - +this.artikel[this.artikel.findIndex(a => a.kassenProjektArtikelID == artikel.kassenProjektArtikelID)].artikelPreis).toFixed(2);

        console.log(artikel.kassenProjektPfandID);
        if(artikel.kassenProjektPfandID == undefined){
            return;
        }

        if(artikel.kassenProjektPfandID > 0){
            this.rechnungsbetrag = (+this.rechnungsbetrag - +(this.pfand[this.pfand.findIndex(art2 => art2.kassenProjektPfandID == artikel.kassenProjektPfandID)].pfandPreis)).toFixed(2);
        }

        //var pfand = this.pfand[this.ausgewaehltePfand.findIndex(art => art.kassenProjektPfandID == artikel.kassenProjektPfandID)]
        //this.pfandEntfernen(pfand);
    },
    pfandEntfernen(pfand){
        var index = this.ausgewaehltePfand.findIndex(art => art.kassenProjektPfandID == pfand.kassenProjektPfandID);
        try{
        if(this.ausgewaehltePfand[index].anzahl == 1){
            this.ausgewaehltePfand.splice(index, 1);
            this.rechnungsbetrag = (this.rechnungsbetrag - +this.pfand[this.pfand.findIndex(a => a.kassenProjektArtikelID == pfand.kassenProjektArtikelID)].pfandPreis).toFixed(2);
        } else if(this.ausgewaehltePfand[index].anzahl == -1){
            this.ausgewaehltePfand.splice(index, 1);
            this.rechnungsbetrag = (this.rechnungsbetrag - - +this.pfand[this.pfand.findIndex(a => a.kassenProjektArtikelID == pfand.kassenProjektArtikelID)].pfandPreis).toFixed(2);
        } else{
            if (this.ausgewaehltePfand[index].anzahl < 0){
                this.ausgewaehltePfand[index].anzahl += +1;
                this.ausgewaehltePfand[index].preis = (+this.ausgewaehltePfand[index].preis - - +this.pfand[this.pfand.findIndex(art2 => art2.kassenProjektArtikelID == pfand.kassenProjektArtikelID)].pfandPreis).toFixed(2)
                this.rechnungsbetrag = (this.rechnungsbetrag - - +this.pfand[this.pfand.findIndex(a => a.kassenProjektArtikelID == pfand.kassenProjektArtikelID)].pfandPreis).toFixed(2);
            } else{
                this.ausgewaehltePfand[index].anzahl -= +1;
                this.ausgewaehltePfand[index].preis = (+this.ausgewaehltePfand[index].preis - +this.pfand[this.pfand.findIndex(art2 => art2.kassenProjektArtikelID == pfand.kassenProjektArtikelID)].pfandPreis).toFixed(2)
                this.rechnungsbetrag = (this.rechnungsbetrag - +this.pfand[this.pfand.findIndex(a => a.kassenProjektArtikelID == pfand.kassenProjektArtikelID)].pfandPreis).toFixed(2);
            }
        }
        
        } catch{

        }
    },
    clear(){
        this.ausgewaehlteArtikel = [];
        this.ausgewaehltePfand = [];
        this.rechnungsbetrag = (0.00).toFixed(2);
    },
    async getArtikel(){
        var kassenProjektID = this.kassenProjekt.kassenProjektID;
        var feuerwehrID = this.kassenProjekt.feuerwehrID;
        return await KassenprojektArtikel.get(kassenProjektID, feuerwehrID)
    },
    async refreshArtikel(){
        this.artikel = [];
        this.artikel = await this.getArtikel();
    },
    async openArtikelPopup(artikel,neuerArtikel){
        var title;
        if(neuerArtikel){
            title='Artikel hinzufügen';
            artikel = {
                artikelName: "",
                artikelPreis: "",
                kassenProjektPfandID: null
            }
        } else{
            title='Artikel bearbeiten';
        }

        let pfandOptions = `<option value="">Kein Pfand</option>`;
        for (let pfand of this.pfand) {
            const selected = artikel.kassenProjektPfandID == pfand.kassenProjektPfandID ? "selected" : "";
            pfandOptions += `<option value="${pfand.kassenProjektPfandID}" ${selected}>${pfand.pfandName}</option>`;
        }

        var html = "";
        html += `<input id="bezeichnung" value="` + artikel.artikelName + `" maxLength="20" class="swal2-input" placeholder="Bezeichnung" style="margin-left:0 !important; margin-right: 0 !important; width: 100% !important"><br>`;
        html += `<input id="artikelPreis" value="` + (+artikel.artikelPreis).toFixed(2) + `" type="number" min="0.00" max="999.99" step="0.01" class="swal2-input" placeholder="Preis" style="margin-left:0 !important; margin-right: 0 !important; width: 100% !important"><br>`;
        html += `<select id="kassenProjektPfandID" class="swal2-input" style="margin-left:0 !important; margin-right:0 !important; width:100% !important">${pfandOptions}</select>`;

        const { value: formValues } = await Swal.fire({
            title: title,
            html: html,
            focusConfirm: false,
            preConfirm: () => {
            return [
                document.getElementById("bezeichnung").value,
                document.getElementById("artikelPreis").value,
                document.getElementById("kassenProjektPfandID").value
            ];
            }
        });
        if (formValues) {
            return {bezeichnung, artikelPreis, kassenProjektPfandID};
        }
    },
    async artikelHinzufuegen(){
        var artikelTemp = await this.openArtikelPopup(null, true);
        if(artikelTemp == null){
            return;
        }
        var artikel;
        
        if(this.artikel.length > 0){
            artikel = {
                kassenProjektArtikelID: this.artikel[this.artikel.length - 1].kassenProjektArtikelID + 1,
                kassenProjektID: this.kassenProjekt.kassenProjektID,
                artikelName: artikelTemp.bezeichnung.value,
                artikelPreis: artikelTemp.artikelPreis.value,
                feuerwehrID: this.kassenProjekt.feuerwehrID,
                imgSrc: null,
                kassenProjektPfandID: artikelTemp.kassenProjektPfandID.value,
            }
        } else{
            artikel = {
                kassenProjektArtikelID: 1,
                kassenProjektID: this.kassenProjekt.kassenProjektID,
                artikelName: artikelTemp.bezeichnung.value,
                artikelPreis: artikelTemp.artikelPreis.value,
                feuerwehrID: this.kassenProjekt.feuerwehrID,
                imgSrc: null,
                kassenProjektPfandID: artikelTemp.kassenProjektPfandID.value,
            }
        }

        console.log(artikel);
        await KassenprojektArtikel.add(artikel);
        this.refreshArtikel();
    },
    async artikelBearbeiten(a){
        var artikelTemp = await this.openArtikelPopup(a,false);
        if(artikelTemp == null){
            return;
        }

        var artikel = a;
        artikel.artikelName = artikelTemp.bezeichnung.value;
        artikel.artikelPreis = artikelTemp.artikelPreis.value;
        artikel.kassenProjektPfandID = artikelTemp.kassenProjektPfandID.value;

        await KassenprojektArtikel.update(artikel);
        this.refreshArtikel();
    },
    async artikelLoeschen(a){
        var response = await KassenprojektArtikel.delete(a)
        if(response == true){
            this.refreshArtikel();
        }
    },
    async getPfand(){
        var kassenProjektID = this.kassenProjekt.kassenProjektID;
        var feuerwehrID = this.kassenProjekt.feuerwehrID;
        return await KassenprojektPfand.get(kassenProjektID, feuerwehrID)
    },
    async refreshPfand(){
        this.pfand = [];
        this.pfand = await this.getPfand();
    },
    async openPfandPopup(pfand,neuerPfand){
        var title;
        if(neuerPfand){
            title='Pfand hinzufügen';
            pfand = {
                pfandName: "",
                pfandPreis: ""
            }
        } else{
            title='Pfand bearbeiten';
        }
        var html = "";
        html += `<input id="bezeichnung" value="` + pfand.pfandName + `" maxLength="20" class="swal2-input" placeholder="Bezeichnung" style="margin-left:0 !important; margin-right: 0 !important; width: 100% !important"><br>`;
        html += `<input id="pfandPreis" value="` + (+pfand.pfandPreis).toFixed(2) + `" type="number" min="0.00" max="999.99" step="0.01" class="swal2-input" placeholder="Preis" style="margin-left:0 !important; margin-right: 0 !important; width: 100% !important"><br>`;

        const { value: formValues } = await Swal.fire({
            title: title,
            html: html,
            focusConfirm: false,
            preConfirm: () => {
            return [
                document.getElementById("bezeichnung").value,
                document.getElementById("pfandPreis").value
            ];
            }
        });
        if (formValues) {
            return {bezeichnung, pfandPreis};
        }
    },
    async pfandHinzufuegen(){
        var pfandTemp = await this.openPfandPopup(null, true);
        if(pfandTemp == null){
            return;
        }
        var pfand;
        
        if(this.pfand.length > 0){
            pfand = {
                kassenProjektPfandID: this.pfand[this.pfand.length - 1].kassenProjektPfandID + 1,
                kassenProjektID: this.kassenProjekt.kassenProjektID,
                pfandName: pfandTemp.bezeichnung.value,
                pfandPreis: pfandTemp.pfandPreis.value,
                feuerwehrID: this.kassenProjekt.feuerwehrID
            }
        } else{
            pfand = {
                kassenProjektPfandID: 1,
                kassenProjektID: this.kassenProjekt.kassenProjektID,
                pfandName: pfandTemp.bezeichnung.value,
                pfandPreis: pfandTemp.pfandPreis.value,
                feuerwehrID: this.kassenProjekt.feuerwehrID,
            }
        }

        await KassenprojektPfand.add(pfand);
        this.refreshPfand();
    },
    async pfandBearbeiten(p){
        var pfandTemp = await this.openPfandPopup(p,false);
        if(pfandTemp == null){
            return;
        }

        var pfand = p;
        pfand.pfandName = pfandTemp.bezeichnung.value;
        pfand.pfandPreis = pfandTemp.pfandPreis.value;
        console.log(pfand.pfandPreis);

        await KassenprojektPfand.update(pfand);
        this.refreshPfand();
    },
    async pfandLoeschen(p){
        var response = await KassenprojektPfand.delete(p)
        if(response == true){
            this.refreshPfand();
        }
    },
    async bestaetigen(){
        this.rechnerBetrag = this.rechnungsbetrag;
        this.rechnerBekommen = '';
        this.rechnerGeben = '';
        this.rechner = true;
        var kassenprojekt = this.kassenProjekt;
        kassenprojekt.geldstand = +localStorage.getItem("Kassenprojekt: offenes Geld ID "+kassenprojekt.kassenProjektID);
        kassenprojekt.geldstand = +kassenprojekt.geldstand + +this.rechnungsbetrag;
        //kassenprojekt.geldstand = +localStorage.getItem("Kassenprojekt: offenes Geld ID "+kassenprojekt.kassenProjektID)+ +kassenprojekt.geldstand;

        /*const response = await Kassenprojekt.update(kassenprojekt);

        if(response == true){
            localStorage.setItem("Kassenprojekt: offenes Geld ID " + kassenprojekt.kassenProjektID, 0);
            var konvKey = this.$route.params.id.replace(":", "");
            this.kassenProjekt = await Kassenprojekt.get(konvKey);
        } else{*/
            localStorage.setItem("Kassenprojekt: offenes Geld ID " + kassenprojekt.kassenProjektID, kassenprojekt.geldstand);
        /*}*/

        var counter = 0;
        while(counter < this.ausgewaehlteArtikel.length){
            var artikel = this.artikel;
            let index = this.artikel.findIndex(art => art.kassenProjektArtikelID == this.ausgewaehlteArtikel[counter].kassenProjektArtikelID);

            if(!(artikel[index].anzahlVerkauft > 0)){
                artikel[index].anzahlVerkauft = 0;
            }

            artikel[index].anzahlVerkauft = +(+artikel[index].anzahlVerkauft + +(localStorage.getItem("Kassenprojekt: artikelAnzahl " + this.artikel[index].kassenProjektID + "-" + this.artikel[index].kassenProjektArtikelID)) + +(this.ausgewaehlteArtikel[counter].anzahl));


            //localStorage.setItem("Kassenprojekt: artikelAnzahl " + this.artikel[index].kassenProjektID + "-" + this.artikel[index].kassenProjektArtikelID, artikel[index].anzahlVerkauft);
            localStorage.setItem("Kassenprojekt: artikelAnzahl " + this.artikel[index].kassenProjektID + "-" + this.artikel[index].kassenProjektArtikelID, +(localStorage.getItem("Kassenprojekt: artikelAnzahl " + this.artikel[index].kassenProjektID + "-" + this.artikel[index].kassenProjektArtikelID)) + +(this.ausgewaehlteArtikel[counter].anzahl));
            if(artikel[index].kassenProjektPfandID != null && artikel[index].kassenProjektPfandID > 0){
                let index2 = this.pfand.findIndex(pfa => pfa.kassenProjektPfandID == artikel[index].kassenProjektPfandID);
                if(!(this.pfand[index2].anzahlOffen > 0)){
                    this.pfand[index2].anzahlOffen = 0;
                }
                this.pfand[index2].anzahlOffen = +(+this.pfand[index2].anzahlOffen + +(localStorage.getItem("Kassenprojekt: pfandAnzahlGegeben " + this.artikel[index].kassenProjektID + "-" + this.artikel[index].kassenProjektPfandID)) + +(this.ausgewaehlteArtikel[counter].anzahl));

                localStorage.setItem("Kassenprojekt: pfandAnzahlGegeben " + this.artikel[index].kassenProjektID + "-" + this.artikel[index].kassenProjektPfandID, +(localStorage.getItem("Kassenprojekt: pfandAnzahlGegeben " + this.artikel[index].kassenProjektID + "-" + this.artikel[index].kassenProjektPfandID)) + +(this.ausgewaehlteArtikel[counter].anzahl));
            }
            counter++;
        }

        counter = 0;
        while(counter < this.ausgewaehltePfand.length){
            var pfand = this.pfand;
            let index = this.pfand.findIndex(pfa => pfa.kassenProjektPfandID == this.ausgewaehltePfand[counter].kassenProjektPfandID);

            if(!(pfand[index].anzahlBekommen > 0)){
                pfand[index].anzahlBekommen = 0;
            }

            pfand[index].anzahlBekommen = +(+pfand[index].anzahlBekommen + +(localStorage.getItem("Kassenprojekt: pfandAnzahlBekommen " + this.pfand[index].kassenProjektID + "-" + this.pfand[index].kassenProjektPfandID)) + (+(this.ausgewaehltePfand[counter].anzahl)*(-1)));
            
            //pfand[index].anzahlBekommen = pfand[index].anzahlBekommen+ +(-this.ausgewaehltePfand[counter].anzahl);


            localStorage.setItem("Kassenprojekt: pfandAnzahlBekommen " + this.pfand[index].kassenProjektID + "-" + this.pfand[index].kassenProjektPfandID, +(localStorage.getItem("Kassenprojekt: pfandAnzahlBekommen " + this.pfand[index].kassenProjektID + "-" + this.pfand[index].kassenProjektPfandID)) + (+(this.ausgewaehltePfand[counter].anzahl)*(-1)));
            counter++;
        }
        
        this.rechnungsbetrag = (0.00).toFixed(2);
        this.ausgewaehlteArtikel = [];
        this.ausgewaehltePfand = [];

        //this.artikelstaendeUebermitteln();
    },
    async changeName(){
        var kassenprojekt = this.kassenProjekt;

        Swal.fire({
            title: "Name ändern",
            input: "text",
            showCancelButton: true,
            confirmButtonText: "Weiter",
            inputAttributes: {
                placeholder: kassenprojekt.bezeichnung,
            },
            preConfirm: (eingabe) => {
            return eingabe;
            }
        }).then((result) => {
            if(result.isConfirmed){
                kassenprojekt.bezeichnung = result.value;
                this.changeName2(kassenprojekt);
            }
        });
    },
    async changeName2(kassenprojekt){
        const response = await Kassenprojekt.update(kassenprojekt);
        if(response == true){
            Swal.fire({
                title: "Speichern erfolgreich",
                text: "Der Name wurde erfolgreich geändert",
                icon: "success"
            });
        } else{
            Swal.fire({
                title: "Fehler",
                text: "Name konnte nicht gespeichert werden!",
                icon: "error"
            });
        }
    },
    async deleteProjekt(){
        Swal.fire({
            title: "Das Kassenprojekt wirklich löschen?",
            showDenyButton: true,
            confirmButtonText: "Löschen",
            denyButtonText: `Nicht löschen`
            }).then((result) => {
            if (result.isConfirmed) {
                this.deleteProjekt2(this.kassenProjekt)
            } 
        });
    },
    async deleteProjekt2(projekt){
        const response = await Kassenprojekt.delete(projekt);

        var prefix = "Kassenprojekt: artikelAnzahl 1-";

        for (let i = 0; i < localStorage.length; i++) {
            const key = localStorage.key(i);
            if (key.startsWith(prefix)) {
                localStorage.removeItem(key);
            } else if(key == "Kassenprojekt: offenes Geld ID " + this.kassenProjekt.kassenProjektID){
                localStorage.removeItem(key);
            }
        }

        if(response){
            await Swal.fire({
                title: "Löschung erfolgreich",
                text: "Kassenprojekt erfolgreich gelöscht",
                icon: "success"
            })
            this.$emit('openKassensystemAuswahl');
        } else{
            Swal.fire({
                title: "Löschung abgebrochen",
                text: "Kassenprojekt konnte nicht gelöscht werden",
                icon: "error"
            })
        }
    },
    async artikelstaendeUebermitteln(meldung){
        var counter = 0;
        var error = false;
        var updated = false;
        /*try{
            this.artikel = await KassenprojektArtikel.get(this.kassenProjekt.kassenProjektID, this.kassenProjekt.feuerwehrID);
        }catch(err){
            this.artikel = JSON.parse(localStorage.getItem("kassenprojektArtikel"));
        }
        while(counter < this.artikel.length){
            var artikel = this.artikel;

            if(!(artikel[counter].anzahlVerkauft > 0)){
                artikel[counter].anzahlVerkauft = 0;
            }

            artikel[counter].anzahlVerkauft = +(+artikel[counter].anzahlVerkauft + +(localStorage.getItem("Kassenprojekt: artikelAnzahl " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektArtikelID)));
            if(artikel[counter].kassenProjektPfandID != null){
                try{
                    var pfand2 = this.pfand.filter(p => 
                        p.kassenProjektPfandID === artikel[counter].kassenProjektPfandID &&
                        p.kassenProjektID        === artikel[counter].kassenProjektID
                    );
                    pfand2 = pfand2[0];
                    pfand2.anzahlOffen = +pfand2.anzahlOffen +(+(localStorage.getItem("Kassenprojekt: artikelAnzahl " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektArtikelID)));
                    const response2 = await KassenprojektPfand.update(pfand2);
                    //localStorage.setItem("Kassenprojekt: pfandAnzahlGegeben " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektPfandID, 0);
                    updated = true;
                } catch(err){
                    //localStorage.setItem("Kassenprojekt: pfandAnzahlGegeben " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektPfandID, +localStorage.getItem("Kassenprojekt: pfandAnzahlGegeben " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektPfandID));
                }
            }
            
            try{
                const response = await KassenprojektArtikel.update(artikel[counter]);
                if(response){
                    localStorage.setItem("Kassenprojekt: artikelAnzahl " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektArtikelID, 0);
                } else{
                    error = true;
                }
            } catch(err){
                localStorage.setItem("Kassenprojekt: artikelAnzahl " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektArtikelID, artikel[counter].anzahlVerkauft);
            }
            counter++;
        }*/
        counter = 0;
        while(counter < this.artikel.length){
            var artikel = this.artikel;

            if(!(artikel[counter].anzahlVerkauft > 0)){
                artikel[counter].anzahlVerkauft = 0;
            }

            //artikel[counter].anzahlVerkauft = +(+artikel[counter].anzahlVerkauft + +(localStorage.getItem("Kassenprojekt: artikelAnzahlBekommen " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektPfandID)));
            
            try{
                this.artikel = await KassenprojektArtikel.get(this.kassenProjekt.kassenProjektID, this.kassenProjekt.feuerwehrID);
                let index = this.artikel.findIndex(art => art.kassenProjektArtikelID == artikel[counter].kassenProjektArtikelID);
                this.artikel[index].anzahlVerkauft += +(localStorage.getItem("Kassenprojekt: artikelAnzahl " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektArtikelID));
                const response = await KassenprojektArtikel.update(this.artikel[index]);
                if(response){
                    localStorage.setItem("Kassenprojekt: artikelAnzahl " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektArtikelID, 0);
                } else{
                    error = true;
                }
            } catch(err){
                //localStorage.setItem("Kassenprojekt: artikelAnzahl " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektArtikelID, artikel[counter].anzahlVerkauft);
            }

            if(artikel[counter].kassenProjektPfandID != null && artikel[counter].kassenProjektPfandID > 0){
                try{
                    this.pfand = await KassenprojektPfand.get(this.kassenProjekt.kassenProjektID, this.kassenProjekt.feuerwehrID);
                    let index = this.pfand.findIndex(pfa => pfa.kassenProjektPfandID == artikel[counter].kassenProjektPfandID);
                    this.pfand[index].anzahlOffen += +(localStorage.getItem("Kassenprojekt: pfandAnzahlGegeben " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektPfandID));
                    const response = await KassenprojektPfand.update(this.pfand[index]);
                    if(response){
                        localStorage.setItem("Kassenprojekt: pfandAnzahlGegeben " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektPfandID, 0);
                    } else{
                        error = true;
                    }
                } catch(err){
                    //localStorage.setItem("Kassenprojekt: pfandAnzahlGegeben " + this.artikel[counter].kassenProjektID + "-" + this.artikel[counter].kassenProjektPfandID, this.pfand[index].anzahlOffen);
                }
            }
            counter++;
        }

        counter = 0;
        var errorGet = false;
        try{
            this.pfand = await KassenprojektPfand.get(this.kassenProjekt.kassenProjektID, this.kassenProjekt.feuerwehrID);
        } catch(err){
            errorGet = true;
        }
        while(counter < this.pfand.length){
            var pfand = this.pfand;

            if(!(pfand[counter].anzahlBekommen > 0)){
                pfand[counter].anzahlBekommen = 0;
            }

            //pfand[counter].anzahlBekommen = +(+pfand[counter].anzahlBekommen + +(localStorage.getItem("Kassenprojekt: pfandAnzahlBekommen " + this.pfand[counter].kassenProjektID + "-" + this.pfand[counter].kassenProjektPfandID)));
            
            try{
                if(errorGet){
                    throw 'Error Get KassenprojektPfand';
                }
                this.pfand[counter].anzahlBekommen += +(localStorage.getItem("Kassenprojekt: pfandAnzahlBekommen " + this.pfand[counter].kassenProjektID + "-" + this.pfand[counter].kassenProjektPfandID))
                const response = await KassenprojektPfand.update(this.pfand[counter]);
                if(response){
                    localStorage.setItem("Kassenprojekt: pfandAnzahlBekommen " + this.pfand[counter].kassenProjektID + "-" + this.pfand[counter].kassenProjektPfandID, 0);
                } else{
                    error = true;
                }
            } catch(err){
                //localStorage.setItem("Kassenprojekt: pfandAnzahlBekommen " + this.pfand[counter].kassenProjektID + "-" + this.pfand[counter].kassenProjektPfandID, pfand[counter].anzahlBekommen);
            }
            counter++;
        }

        if(error==true){
            Swal.fire({
                title: "Speicherung abgebrochen",
                text: "Keine Speicherung möglich",
                icon: "error"
            })
        }

        var konvKey = this.$route.params.id.replace(":", "");
        var kassenprojekt;
        try{
            kassenprojekt = await Kassenprojekt.get(konvKey);
        }catch(err){
            kassenprojekt = JSON.parse(localStorage.getItem("kassenprojekt"));
        }
        kassenprojekt[0].geldstand = +(+(kassenprojekt[0].geldstand) + +(localStorage.getItem("Kassenprojekt: offenes Geld ID "+kassenprojekt[0].kassenProjektID)));
        try{
            const response = await Kassenprojekt.update(kassenprojekt[0]);
            console.log(kassenprojekt[0].geldstand);

            if(response == true){
                console.log("Test offenes Geld");
                localStorage.setItem("Kassenprojekt: offenes Geld ID " + kassenprojekt[0].kassenProjektID, 0);
                if(meldung){
                    Swal.fire({
                        title: "Speicherung erfolgreich",
                        text: "Daten wurden erfolgreich gespeichert",
                        icon: "success"
                    })
                }
            } else{
                if(meldung){
                    Swal.fire({
                        title: "Speicherung abgebrochen",
                        text: "Keine Speicherung möglich",
                        icon: "error"
                    })
                }
            }
        } catch(err){
            console.log(err);
        }
    },
    rechnerButtonClick(val) {
      if (val === 'C') {
        // alles löschen
        this.rechnerBekommen = '';
      } else if (val.startsWith('+')) {
        // Münz-/Scheineingabe, z.B. "+10"
        const add = parseFloat(val.slice(1)) || 0;
        const aktuell = parseFloat(this.rechnerBekommen) || 0;
        this.rechnerBekommen = (aktuell + add).toFixed(2);
      } else if (val === 'DEL') {
      // letztes Zeichen löschen
      this.rechnerBekommen = this.rechnerBekommen.slice(0, -1);
      } else if(val === 'ENDE'){
        this.rechner = false;
      } else if (val === '.') {
        // nur einen Punkt erlauben
        if (!this.rechnerBekommen.includes('.')) {
          this.rechnerBekommen += '.';
        }
      } else if (val === '=') {
        // hier könntest du noch extra Logik einbauen – momentan tut sich nichts
      } else {
        // Ziffer anhängen
        if(!this.rechnerBekommen.includes('.00')){
            this.rechnerBekommen += val;
        }
      }
    },
    setAktiveArtikel(){
        this.artikel.forEach(art => {
            let artikelObj = JSON.parse(localStorage.getItem("AktiveArtikel: KassenprojektID: " + art.kassenProjektID + " - KassenprojektArtikelID: " + art.kassenProjektArtikelID));
            if(artikelObj == null){
                artikelObj = {
                    kassenProjektID: art.kassenProjektID,
                    kassenProjektArtikelID: art.kassenProjektArtikelID,
                    active: true
                }
                this.aktiveArtikel.push(artikelObj);
                localStorage.setItem("AktiveArtikel: KassenprojektID: " + artikelObj.kassenProjektID + " - KassenprojektArtikelID: " + artikelObj.kassenProjektArtikelID, JSON.stringify(artikelObj));
            }
        });
    },
    getArtikelActive(art){
        try{
            return (JSON.parse(localStorage.getItem("AktiveArtikel: KassenprojektID: " + art.kassenProjektID + " - KassenprojektArtikelID: " + art.kassenProjektArtikelID)).active);
        } catch(err){
            return true;
        }
    },
    setAktivePfand(){
        this.pfand.forEach(art => {
            let pfandObj = JSON.parse(localStorage.getItem("AktivePfand: KassenprojektID: " + art.kassenProjektID + " - KassenprojektPfandID: " + art.kassenProjektPfandID));
            if(pfandObj == null){
                pfandObj = {
                    kassenProjektID: art.kassenProjektID,
                    kassenProjektPfandID: art.kassenProjektPfandID,
                    active: true
                }
                this.aktivePfand.push(pfandObj);
                localStorage.setItem("AktivePfand: KassenprojektID: " + pfandObj.kassenProjektID + " - KassenprojektPfandID: " + pfandObj.kassenProjektPfandID, JSON.stringify(pfandObj));
            }
        });
    },
    getPfandActive(pfand){
        try{
            return (JSON.parse(localStorage.getItem("AktivePfand: KassenprojektID: " + pfand.kassenProjektID + " - KassenprojektPfandID: " + pfand.kassenProjektPfandID)).active);
        } catch(err){
            return true;
        }
    },
  },
  async created(){
    try{
      this.kassenProjekt = {
      kassenProjektID: 2,
      feuerwehrID: 1,
      bezeichnung: 'Bierfest Notfallsystem'
    };

        //this.artikel = await KassenprojektArtikel.get(this.kassenProjekt.kassenProjektID, this.kassenProjekt.feuerwehrID);
        localStorage.setItem("kassenprojektArtikel", JSON.stringify(this.artikel));
        this.setAktiveArtikel();

        //this.pfand = await KassenprojektPfand.get(this.kassenProjekt.kassenProjektID, this.kassenProjekt.feuerwehrID);
        localStorage.setItem("kassenprojektPfand", JSON.stringify(this.pfand));
        this.setAktivePfand();

        this.setColors();
        this.arraySortieren();
    } catch(err){
        console.log(err);
        this.artikel = JSON.parse(localStorage.getItem("kassenprojektArtikel"));
        this.pfand = JSON.parse(localStorage.getItem("kassenprojektPfand"));
    }
  }
};
</script>

<style lang="scss" scoped>
#kasse{
  //  overflow-y: hidden;
  background-color: white;
  width: 100vw;
  height: 100vh;

    #Kassensystem2{
        display: flex;
        overflow-y: hidden;

        .artikelAktiv{
            //background-color: green !important;
            border: solid 1px green !important;
        }

        .artikelNichtAktiv{
            //background-color: red !important;
            background: repeating-linear-gradient(
                45deg,
                red 0 3px,   /* Roter Streifen: 6px breit */
                white 3px 20px  /* Weißer Abstand: 14px breit (6px bis 20px) */
            ) !important;
            //border: solid 2px red !important;
        }

        .pfandAktiv{
            //background-color: green !important;
            border: solid 1px green !important;
        }

        .pfandNichtAktiv{
            //background-color: red !important;
            background: repeating-linear-gradient(
                45deg,
                red 0 3px,   /* Roter Streifen: 6px breit */
                white 3px 20px  /* Weißer Abstand: 14px breit (6px bis 20px) */
            ) !important;
            //border: solid 2px red !important;
        }

        #Bestellung{
            background-color: rgb(238, 239, 243);
            height: 84vh;
            width: 33vw;
            border-right: 3px solid rgb(221, 222, 226);

            .ElementeContainer{
                height: 84vh;
                padding-top: 1rem;
                overflow-y: scroll;
            }

            .AuswahlItem{
                box-shadow: 0 0 0 2px rgba(221, 222, 226, 0.486);
                background-color: white;
                margin-left: 1rem;
                margin-right: 1rem;
                margin-top: 0.3rem;
                padding: 1rem;
                border-radius: 5px;
                border-left: solid 0.3rem rgb(0, 89, 255);

                .textItem{
                    display: flex;
                    justify-content: space-between;

                    .Anzahl{
                        font-weight: bold;
                        margin-right: 0.7rem;
                    }

                    .Gericht{
                        font-weight: bold;
                    }
                }

            }

            .AuswahlItemPfand{
                box-shadow: 0 0 0 2px rgba(221, 222, 226, 0.486);
                background-color: white;
                margin-left: 1rem;
                margin-right: 1rem;
                margin-top: 0.3rem;
                padding: 1rem;
                border-radius: 5px;
                border-left: solid 0.3rem green;

                .textItem{
                    display: flex;
                    justify-content: space-between;

                    .Anzahl{
                        font-weight: bold;
                        margin-right: 0.7rem;
                    }

                    .Gericht{
                        font-weight: bold;
                    }
                }

            }
        }

        #AuswahlTerminal{
            background-color: rgb(238, 239, 243);
            height: 84vh;
            width: 67vw;
            padding-left: 0.7rem;
            position: relative;
            overflow-y: scroll;

            #auswahlContainer{
                padding-top: 1rem;
                display: flex;
                flex-wrap:wrap;
                gap: 0.3rem;
                align-content: start;
            }

            #pfandContainer{
                position:absolute;
                bottom: 0;
                padding-top: 1rem;
                display: flex;
                flex-wrap:wrap;
                gap: 0.3rem;
                align-content: start;
                border-top: 3px solid rgb(221, 222, 226);
                width: 97%;
            }

            .gerichtItem{
                background-color: white;
                box-shadow: 0 0 0 2px rgba(221, 222, 226, 0.486);
                margin-left: 0.3rem;
                margin-top: 0.3rem;
                width: 21vw;
                min-width: 10rem;
                //height: 3rem;
                height: auto;
                padding-top: 1rem;
                padding-bottom: 1rem;
                border-radius: 5px;

                .gerichtItemBezeichnung{
                    font-size:larger;
                    font-weight: bold;
                }

                .hinzufuegenButton{
                    margin-top: -0.4rem;
                    font-size: xxx-large;
                    border: none;
                    background: none;
                }
            }

            .pfandItem{
                background-color: white;
                box-shadow: 0 0 0 2px rgba(221, 222, 226, 0.486);
                margin-left: 0.3rem;
                margin-bottom: 0.3rem;
                width: 21vw;
                min-width: 10rem;
                //height: 3rem;
                height: auto;
                padding-top: 1rem;
                padding-bottom: 1rem;
                border-radius: 5px;

                .pfandItemBezeichnung{
                    font-size:larger;
                    font-weight: bold;
                }

                .hinzufuegenButton{
                    margin-top: -0.4rem;
                    font-size: xxx-large;
                    border: none;
                    background: none;
                }
            }
        }

        #homeButton{
            position: absolute;
            bottom: -2.7rem;
            left: 50%;
            transform: translateX(-50%);
        }
    }

    .RechnungsContainer{
        overflow-y: hidden;
        height: 14vh;
        padding-top: 0.5rem;
        background-color: white;
        border-top: 3px solid rgb(221, 222, 226);
        display: flex;
        justify-content: space-between;
        padding-bottom: 0.3rem;

        #rechnungsbetrag{
            height: 100%;
            font-size: xxx-large;
            line-height: 13vh;
        }

        #rechnungsTextContainer{
            width: 33vw;
        }

        #buttonContainer{
            width: 40%;
            height: 95%;
            display: flex;
            margin-right: 2rem;

            #bestaetigen{
                background-color: rgb(0, 95, 0);
                border: none;
                padding: 0;
                //height: 62%;
                width: 100%;
                color: white;
                font-size: x-large;
                //margin-top: 0.2rem;
            }

            #clear{
                background-color: rgb(143, 0, 0);
                border: none;
                padding: 0;
                //height: 38%;
                width: 100%;
                color: white;
                font-size: x-large;
            }
        }
    }

    #menuButtonKassensystem{
        display: block;
        position: fixed;
        right: 2rem;
        top: 2rem;
        z-index: 30;
        border: none;
        border-radius: 90px;
        box-shadow: 0 0 2px 2px rgba(0, 0, 0, 0.11);
        margin: 0px !important;
        height: 3rem !important;
        width: 3rem !important;
        background-color: white;
        color: black;
    }

    #menu{
        width: 75vw;
        max-width: 250px;
        background-color: white !important;
        position: absolute !important;
        top: 4.3rem !important;
        right: 3.8rem !important;
        z-index: 4 !important;
        padding: 1rem !important;
        border-radius: 7px !important;
        border: solid grey 1px;
        button{
            display: flex;
            justify-content: space-between;
            span{
                margin-left: 1rem;
                margin-right: 1rem;
                font-size: large;
            }
            padding-top: 0.5rem !important;
            padding-bottom: 0.5rem !important;
            background-color: white;
            border:none;
            //border-top: solid black 1px;
            //font-weight: 300;
        }
        button:hover{
            background-color: rgb(223, 223, 223);
        }
    }

    #menuBackground{
        width: 100vw;
        height: 100vh;
        z-index: 3;
        position: fixed;
        left: 0;
        top: 0;
    }

    #rechner {
        position: fixed;
        top: 50%;
        left: 50%;
        width: 70vh;
        height: 50vh;
        z-index: 5;
        display: grid;
        transform: translate(-50%, -65%);
        grid-template-columns: repeat(4, 1fr);
        grid-template-rows: auto repeat(4, 1fr);
        gap: 1px;

        .display{
            grid-column: 1 / -1;
            background-color: white;
            color: black;
            font-size: 2em;
            display: flex;
            align-items: center;
            //justify-content: flex-end;
            justify-content: space-between;
            padding: 0.5em;
            p{
                margin: 0;
                padding: 0;
            }

            #rechnerBetrag{
                color: black;
            }

            #rechnerBekommen{
                color: green;
            }

            #rechnerGeben{
                color: red;
            }
        }

        .rechnerButton{
            display: table-cell;
            width: 17.5vh;
            height: 10.5vh;
            text-align: center;
            vertical-align: middle;
            font-size: 1.5em;
            background-color: #eee;
            border: none;
            cursor: pointer;
        }
    }

    #rechnerBackground{
        width: 100vw;
        height: 100vh;
        z-index: 3;
        position: fixed;
        left: 0;
        top: 0;
        background-color: rgba(0, 0, 0, 0.555);
    }
}
</style>