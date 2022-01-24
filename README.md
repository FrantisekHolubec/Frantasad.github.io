# 10. Analýza rastrového obrazu (100%)
###### tags: `zpracovaniObrazu`, `PB130`,`PV131`

:::warning

* Segmentace obrazu, algoritmy značení komponent, popis objektů, klasifikace objektů
* Výpočet mapy vzdáleností
* Základy matematické morfologie 
    * dilatace a eroze
    * otevření a uzavření
    * hit-or-miss
    * top-hat
    * watershed

:::

## Analýza
:::success
**Typické fáze analýzy**
* Předzpracovnání obrazu
    * Potlačení šumu, odstranění nerovnoměrného osvitu, apod.
* Segmentace obrazu
    * Rozklad definičního oboru na oblasti odpovídající objektům
* Popis objektů
    * Určení atributů/vlastností objektů nutných pro rozpoznání
* Klasifikace objektů
    * Rozdělení objektů do tříd podle jejich atributů
* Porozumění obrazu
    * Porozumění smyslu objektů v obraze
:::


## Segmentace obrazu a algoritmy značení komponent
* Dělení nebo separace obrazu na segmenty (regiony, spojené množiny) podobných vlastností (atributů)

:::info
**Haralick and Shapiro**
„Oblasti by měly být jednotné a homogenní s ohledem na některé charakteristiky, jako je intenzita nebo textura. Interiéry regionů by měly být jednoduché a bez mnoha malých děr. Sousední oblasti by měly mít výrazně odlišné hodnoty s ohledem na charakteristiku v rámci které jsou jednotné. Hranice každého segmentu by měly být jednoduché, ne členité, a musí být prostorově přesné.“ 
:::

:::success
**Definice:**
* **Sousedé daného pixelu**
    * 2D: Horizontální a vertikální sousedé, 4-sousedé,, 8-sousedé,
    * 3D: 6-sousedé,, 18-sousedé,, 26-sousedé,
* **Sousedství dvou pixelů**
    * 2D: 4-sousedství pixely, 8-sousedství pixely
    * 3D: 6-sousedství pixely, 18-sousedství pixely, 26-sousedství pixely
* **Cesta mezi dvěma pixely**
    * Sekvence rozdílných pixelů $p_0,...,p_n$ takových že $p_0 a p_n$ jsou ony dva pixely a $p_i$ a $p_{i+1}$ jsou sousedící pixely pro každé $0 \leq i \leq n-1$
    * $n$ je délka cesty
    * 2D: 4-cesta, 8-cesta
    * 3D: 6-cesta, 18-cesta, 26-cesta
* **Spojené pixely**
    * Uvažujme podmnožinu $S$ pixelů obrázku
    * Dva pixely jsou spojeny v $S$ pokud mezi nimy existuje cesta složená pouze z pixelů podmnožiny $S$
* **Souvislá komponenta množiny**
    * Uvažujme podmnožinu $S$ pixelů obrázku
    * Pro každý pixel v $S$, množina s ním spojených pixelů v $S$ se nazývá souvislá komponenta $S$
* **Souvislá množina**
    * Uvažujme podmnožinu $S$ pixelů obrázku
    * Pokud $S$ má pouze jednu souvislou komponentu, pak se $S$ nazývá souvislá množina
    * Souvislá množina se také nazývá region, oblast nebo segment obrazu
* **Hranice oblasti**
    * Množina pixelů v regionu které mají alespň jednoho souseda mimo region
:::



### Prahování 
###### *Prahuje sa v Prahe často?* ... Badumtsss
:::success
Pixely jsou rozděleny na regiony podle jejich intenzity: pixely s intenzitou menší než zadaný práh patří do jiného regionu než pixely s intenzitou větší.

* Globální prahování = prahování
    * Práh je stejný pro všechny pixely obrazu (nezávisle na jejich pozici)
* Lokální (adaptivní) prahování
    * Práh závisí na pozici pixelu v obraze
* Určení prahu
    * Manuální - práh je nastaven uživatelem 
    * Automatcké - analýza histogramu intenzity 
:::

#### Dvouúrovňové prahování 
:::success
* Pokud je obrázek jednoduchý (předměty podobné intenzity na uniformním pozadí, např. zadaný text nebo buňky stejného typu), histogram obsahuje dva vrcholy s údolím mezi nimi. Hranice je převzata v údolí (na lokálním minimu mezi dvěma maximy).
* Pro přesnější určení minima může být údolí proloženo parabolou.
* Pokud histogram neobsahuje dvě maxima, může pomoci výpočet histogram pixelů, které mají velký Laplacián
:::

##### Obrázky s unimodální distribucí
* Pokud histogram obsahuje pouze jeden zřejmý vrchol, nazýváme takový obrázek obrazem s unimodální distribucí 
* Nejjednodušší metodou je vzít X% nejvíce/nejméně intenzivních pixelů jako objekt, zbytek jako pozadí (např. X=10 %)
* Sofistikovanější metoda (Rosin)
    * Nakresli úsečku od maxima k druhé straně histogramu
    * Úsečka začíná na nejvetším sloupci a končí na prvním prázdném sloupci za posledním neprázdným sloupcem
    * Práh je index $i$ který maximalizuje kolmou vzálenost mezi úsečkou a bodem $(i, h(i))$ 
    * ![](https://i.imgur.com/hmhcwCD.png =300x)

##### Shlukování histogramu
* Uvažujme dvě skupiny pixelů - popředí a pozadí
* Každá skupina má svůj vlastní rozsah a distribuci
* Rozsahy se obvykle překrývají
* Místo minimálního překryvu nemusí být údolí
* Cílem je minimalizovat chybu klasifikace pixelu pozadí jako pixel popředí a naopak
* ![](https://i.imgur.com/rrYYYXT.png =300x)

> [color=#666666] **Algoritmus:**
> * Zvol práh $T$
> * Rozděl obraz do dvou shluků (před a po prahu $T$)
> * $\mu_B(T)$ = průměr pixelů s hodnotou menší než práh T
> * $\mu_O(T)$ = průměr pixelů s hodnotou větší než práh T
> * Vyhodnoť $\displaystyle T = \frac{\mu_B(T) + \mu_O(T)}{2}$ a opakuj dokud algoritmus nezkonverguje

##### Shlukování histogramu (Otsu 1979)
* Idea - Najdi práh který minimalizuje vážený rozptyl v rámci třídy 
* Stejné jako maximalizovat rozptyl v rámci třídy 
* Operuje přímo na šedotónovém histogramu (např. 256 hodnot) což je rychlé
* Předpokládá uniformní nasvětlení a bimodální nebo multimodální distribuci histogramu
* Lze modifikovat pro lokální použití

> [color=#666666] **Algoritmus:**
> * Pro každý práh $T$:
>   * Rozděl pixely do dvou shluků a dle prahu $T$
>   * Najdi průměry $\mu_B(T)$ a $\mu_O(T)$ pro každý shluk
>   * Najdi rozptyl $\delta_{between}(T)$
> * Najdi maximální rozpyl

##### Gradientní prahování (McAulay)
* Idea - Mezi jakýmkoli objektem a pozadím je zřejmá hrana, intenzity na hranách jsou nejdůležitější.
* Vyhodnocení: Vážený součet všech hodnot na obrázku, klást důraz na ty, které jsou v blízkosti silných hran.
* $\displaystyle T = \sum_m f(m) \cdot \frac{|\triangledown f(m)|}{\displaystyle \sum_n |\triangledown f(n)|}$

#### Víceúrovňové prahování 
::: success
* Idea - Pokud obrázek obsahuje několik typů objektů na uniformním pozadí a každý typ objektu má specifický rozsah intenzity, (např. směs buněk více typů), histogram obsahuje několik vrcholů s údolími mezi nimi. 
* Je přijato několik prahů, jeden práh na každé údolí (na lokálním minimu mezi dvěma sousední maxima).
* Pro přesnější určení minima může být údolí proloženo parabolou. 
:::

#### Prahování barevných obrázků
::: success
* Prahování je provedeno na každý barevný kanál zvlášť (RGB).
* Prahování se neprovádí podle intenzity, ale podle odstínu, sytosti nebo jiných charakteristik. 
* Uživatel (nebo počítač) vybere konkrétní rozsahy barev pro každý typ objektu.
:::

#### Lokální (Adaptivní) prahování
::: success
* Problém
    **A)** Nerovnoměrná intenzita pozadí 
    **B)** Stejnoměrná intenzita pozadí, ale nerovnoměrná intenzita objektu, která netvoří jednotlivé píky v    histogramu (píky se v histogramu překrývají a tvoří jeden širší vrchol)
* Řešení:
    1. Obrázek je rozdělen na pravidelné podoblasti (např. čtverce) a práh se nastavuje individuálně pro každý region pomocí analýzy histogramu vypočítaného z této konkrétní oblasti.
        * Nepříliš dobré výsledky na okrajích mezi oblastmi.
    2. Stejné jako 1., ale jsou určeny prahové hodnoty pro jednotlivé pixely pomocí interpolace prahové hodnoty mezi regiony.
        * Dobré v případě **A)**, ale ne v případě **B)**.
    3. Nejprve se provede globální prahování, aby se oddělilo pozadí objektů. 
       Poté se pro každý objekt vypočítá histogram lokální intenzity a je stanovena prahová hodnota.
       Všechny objekty jsou přesegmentovány pomocí jednotlivých prahů
        * Dobré v případě **B)**, ale problémy nastanou, když se dva typy objektů (různých intenzity) vzájemně dotýkají a je s nimi zacházeno jako s jednou oblastí po iniciálním globálním prahování. 
:::


#### Hysterezní prahování
::: success
* Použijí se dva prahy, nízký a vysoký.
* Pixely s hodnotou vyšší než je vysoký práh jsou označny jako objekt
* Pixely s hodnotou vyšší než je nízký práh jsou označny jako objekt pouze pokud v obrázku vytvořeném prahováním nízkžm prahem existuje cesta mezi tímto pixelem a libovolným pixelem, který je již jako objekt označen.
* Ostatní pixely jsou označeny jako pozadí.
* ![](https://i.imgur.com/B1topa4.png =500x)
:::


### Metody založené na oblastech 
::: success
Iterační metody založené na slučování a/nebo dělení oblastí na základě stupně podobnosti vlastností (atributů) oblasti. 
:::

#### Narůstání oblastí (Region growing)
::: success
* Inicializace: Obraz je rozdělen do velkého počtu segmentů (oblastí). 
* Počáteční oblasti mohou být tvořeny dokonce jednotlivými pixely.
* Iterace: Sousední oblasti jsou seskupeny, pokud jsou jejich vlastnosti (většinou intenzita) podobné.
* Poznámky:
    * Je dobré klást určitá omezení na proces slučování
    * Omezení mohou být poměrně složitá 
:::


#### Štěpení a slučování (Split and Merge)
::: success
* Vstup: 
    1) Původní obrázek (jedna oblast).
    2) Jednotlivé pixely (mnoho oblastí).
    3) Střední počet oblastí.
* Iterace: 
    * Oblasti, které nejsou homogenní, jsou rozděleny do několika menších oblastí 
    * Sousední oblasti, které mají podobné vlastnosti, jsou sloučeny dohromady.
:::

::: info
* Podmínky pro rozdělení a sloučení mohou být poměrně složité.
* Často se používá metoda **Quad-Tree**
    * Nehomogenní oblasti se rozdělují na 4 stejné podoblasti (kvadranty) 
    * 4 sousední oblasti podobných vlastností se stejným rodičem se v rámci možností spojují dohromady. 
    * Následuje sloučení regionů na různých úrovních pyramidy nebo mající jiného rodiče.
    * ![](https://i.imgur.com/FzJue71.png =300x)
    * ![](https://i.imgur.com/NthXT88.png =200x) ![](https://i.imgur.com/QBMErLO.png =200x)
:::

### Segmentace největší kontury (Largest Contour Segmentation - LCS)
> [color=#666666] 
> 1. Odfiltrování šumu (průměr nebo medián)
>    * Velikost jádra závisí na typu a velikosti šumu
>2. Určení lokálních maxim a minimální intenzity v obraze ($GlobalMinI$)
>3. Výpočet středů lokálních maxim, tj. redukce lokálních maxim na 1 pixel.
>4. Pro každé lokální maximum:
>    4.1) Současný práh $(CurT) = ($ Maximální intenzita $(LocalMaxI) + GlobalMinI) / 2$
>    4.2) Počet iterací $(I) = 0$
>    4.3) Provede se růst oblasti od středu lokálního maxima tak, aby byly zahrnuty všechny sousední pixely, jejichž intenzita $\geq CurT$ .
>    4.4) $N$ = počet středů uvnitř tohoto regionu.
>    4.5) $I = I + 1$
>    4.6) Pokud $I = BitDepth$, dokončíme iteraci pro daný střed a přejdeme ke kroku 5.
>    4.7) Pokud $N>1$, $CurT = (CurT +$ předchozí vyšší prahová hodnota$) / 2$
>    4.8) Pokud $N=1$, $CurT = (CurT +$ předchozí spodní prahová hodnota$) / 2$
>    4.9) Přejdeme na krok 4.3
>5. Pokud $N>1$, Konečný práh $(FinT) = CurT+1$
>   Pokud $N=1$, konečný práh $(FinT) = CurT$
> 
> **Ukázka:**
> ![](https://i.imgur.com/mQbjx4G.png =500x)
> ![](https://i.imgur.com/N78PeQf.png =500x)

::: info
* Systematická chyba
    * Objekt je často protáhlý směrem k sousednímu objektu (objektům)
    * Hranice objektu je nepřesná
    * Střed objektu je posunut směrem k sousednímu objektu (objektům)
:::

### Segmentace pomocí detekce hran
::: success
* Určení hranic objektů ze snímku okrajové mapy
* Konstrukce šedotónové mapy hran
    * První derivace původního obrázku (např. Sobelův filtr)
    * Druhá derivace původního obrázku (např. Laplaceův filtr)
* Konstrukce binární mapy hran 
    * Vytvoření binárního obrázku (např. prahování)
        * 1 představuje hranové pixely 
        * 0 představuje ostatní pixely 
* Problémy s šumem
    * Před zvýrazněním hran (high-pass filtr) by měl být šum redukován vyhlazením (low-pas filtr)
* Kombinace vícero směrů 
    * Často jsou detekovány hrany v několika směrech a výsledky jsou kombinovány (např. Sobelův filtr)
    * Používají se různé techniky: maximum, součet (průměr), odmocnina ze součtu čtverců atd.
* Post-processing šedotónové mapy hran
    * Ztenčení hran na šířku 1 pixelu (např. Canny)
* Konstrukce binární mapy hran
    * Prahování nebo hysterezní prahování (dvojité prahování) šedotónové mapy hran pro získání pouze relevantních hran. 
:::

**Problém - Hranice v prahované mapě hran nejsou souvislé!** 

* Řešení 1 - Proložení křivkou (Curve fitting)
    * Předpokladáme že objekty na obrázku jsou jednoduché, okrajová mapa neobsahuje větve.
    * Iterativní přizpůsobení koncového bodu
        * ![](https://i.imgur.com/hdjUOV4.png =200x)
    * Proložení polygonem
    * Proložení polynomem
    * Bezierův polynom nebo spline
        * ![](https://i.imgur.com/Uled7om.png =100x)
* Řešení 2 - Heuristické spojování hrany
    * Předpokladáme že objekty na obrázku nejsou příliš složité.
    * Hraniční body jsou propojeny čarami pomocí určitého heuristického algoritmu
    * Následně jsou vymazány vícenásobné spoje a větve a hraniční fragmenty spojeny tzv. „mosty“ 
    * ![](https://i.imgur.com/QQlfDWv.png =250x) ![](https://i.imgur.com/NIGkj7q.png =250x)
* Řešení 3 - Houghova transformace 
    * Předpokladáme že objekty na obrázku nejsou příliš složité.
    * Hledáme známé tvary (např. čáry, kruhy, čtverce atd.)
    * ![](https://i.imgur.com/Syq2YMj.png =400x)


### Segmentace textur
::: success
* Rozdělení obrazu do oblastí na základě charakteristik textury. 
* Metoda 1 (Rosenfeld et al.)
    * Míra textury je definována a vypočítána pro všechny pixely (v určitém okolí daného pixelu). 
    * Textura je tedy převedena na amplitudu a poté je použita jedna z metod segmentace amplitudy
    * Nevýhoda - hranice textur jsou rozmazané.
* Metoda 2 (Thompson)
    * Jsou detekovány přechody mezi oblastmi s různou texturou čímž se vytvoří okrajová mapa. 
    * Okrajová mapa je zpracována podobně jako segmentace pomocí detekce hran
    * Nevýhoda - okraje nejsou souvislé. 
    
![](https://i.imgur.com/5HuaV13.png =500x)
:::

### Metody shlukování 
:::success
* Metody založené na analýze shluků.
* U každého pixelu obrázku je vypočítán vektor $x = [x_1 ,...,x_N]$ $N$ různých měření (vlastností) (obvykle $N$ je asi 10). 
* Měří se různé uživatelsky definované charakteristiky. 
* Poté se provede shluková analýza v $N$-rozměrném prostoru, tj. ty pixely, které tvoří shluk v N-rozměrném prostoru, jsou segmentovány do jedné oblasti.
* Výhoda - Snadné
* Nevýhoda - Pomalé 
:::

### Srovnávání se vzorem
:::success
* Hledání šablony (vzoru) v obrázku výpočtem korelace mezi vzorem a hledanými daty obrázku.
* Vyhodnoťí se kritérium shody pro každé umístění a otočení vzoru na obrázku.
* Lokální maxima tohoto kritéria překračující přednastavený práh představují umístění vzoru v obraze.
* Nevýhody:
    * Velmi časově náročné, zvláště u velkých vzorů.
    * Citlivé na geometrické zkreslení obrazu.
:::


## Popis objektů

### Proč popisovat obrázky/objekty?

:::success
**Popisovače – deskriptory**
* Popisovač je funkce, která pro daný obraz, oblast obrazu (často odpovídající nalezenému objektu), nebo hranici objektu vrátí popis vlastnosti nebo atributu
* Popisem může být
	* Číslo
	* Vektor hodnot
	* Řetězec
	* Graf
	* A další
* Úrovně
    * Globální deskriptory
        * Popis celého obrazu
    * Lokální deskriptory
        * Extrakce zajímavých rysů z obrazu
        * Rohy, lokální struktury, apod.
        * Není nutná segmentace
    * Deskriptory objektů
        * Nutná segmentace
        * Citlivé na okluzi, šum a vzorkování
        * Popis tvaru, textury, barvy, apod.
:::

:::info
* Obrázky nebo předměty je třeba popsat, aby bylo možné provést následnou klasifikační fázi. 
* Obvykle se počítají ty atributy, které jsou potřebné pro klasifikaci.
* Alternativně lze popis použít pro vyhledávání podobných obrázků nebo předmětů ve velké sbírce.
* Vyhledávání se provádí na základě podobnosti popisů (podpisů), které jsou stručné a indexované pro zvýšení rychlosti.
* K jejich popisu není nutná žádná a předchozí znalost obrázků nebo předmětů.
* Preferujeme deskriptory, které jsou invariantní vůči translaci, měřítku, rotaci a ideálně i zkreslení.
* Existují také standardizované sady deskriptorů, např. MPEG-7 
:::



### Analýza atributů
:::success
* Analýza intenzity (střední a maximální intenzita, rozptyl intenzity atd.)
* Analýza histogramu (centrální momenty, uniformita, entropie atd.)
* Analýza frekvencí (Fourierova spektrální analýza)
* Analýza barev
* Analýza textury 
:::

### Atributy objektů
:::info
**Ohraničující obdélník (Bounding box)**
* Nejmenší ohraničující obdélník ve 2D (nebo ohraničující kvádr ve 3D) (
    * ve 2D: obdélník o minimální ploše, který obsahuje daný objekt 
    * ve 3D: rovnoběžnostěn o minimálním objemu, který obsahuje daný objekt
* Rozlišujeme 
    * Obrazově orientovaný ohraničující rámeček (orientovaný podél os obrázku)
    * Objektově orientovaný ohraničovací rámeček (orientovaný podél hlavní osy objektu)
    * ![](https://i.imgur.com/YLvF3fR.png =300x)
* Lze spočítat 
    * Rozměry a velikost ohraničujícího rámečku (plochu nebo objem) 
    * Poměr velikosti rámečku k velikosti objektu 
:::
    
:::info
**Průměr**
* Orientace ohraničovacího rámečku určuje také měření průměru
* Maximální a minimální průměr lze intuitivně definovat jako nejdelší a nejkratší šířku ohraničovacího rámečku respektině.
:::

:::info
**Kruhovost**
* Parametr udávající (od 0 do 1) jak moc je daný objekt blízký ideálnímu kruhu
    * $\displaystyle Circularity(R) = 4\pi \frac{Area(R)}{Perimeter^2(R)}$
:::

:::info
**Konvexní obal**
* Nejmenší polygon, který obsahuje všechny pixely regionu $R$
* Využití k výpočtu hustoty (poměr obsahu objektu k obsahu jeho konvexního obalu)
:::

:::info
**Moment setrvačnosti**
* Míra setrvačnosti tělesa při otáčivém pohybu
* Na nalezený objekt, tj. množinu jeho pixelů R (ne nutně souvislého) se díváme jako na rozložení hmotnosti v objektu
* Můžeme spočítat jeho těžiště (centroid) a tzv. centrální momenty z nichž lze spočítat natočení objektu
* ![](https://i.imgur.com/xPpRS3B.png =200x)

:::

:::info
**Feretův průměr**
* Umožňuje měření průměru v libovolném směru (pro libovolný úhel)
* Je definován jako délka projekce objektu v daném směru
* Rovná vzdálenosti mezi dvěma rovnoběžnými čarami (s daným sklonem), které jsou kolmé k hranicím objektu z opačných stran
* ![](https://i.imgur.com/OBmsLNY.png =200x)
:::

:::info
**Hranice**
* Obvykle reprezentována jako zakódovaný řetězec bodů
* Je uložena pouze absolutní poloha prvního bodu a poté jsou uloženy pouze směry z jednoho bodu do druhého - Tzv. Freemanův kód (Freeman 1961) 
    * ![](https://i.imgur.com/mRFcIOR.png =100x) ![](https://i.imgur.com/QAvOtq9.png =200x)
* Má své vlastní vlastnosti, které lze také vypočítat.
    * Křivost - definována jako zlomek mezi počtem hraničních pixelů, kde hranice výrazně mění svůj směr, a celkovým počtem hraničních pixelů.
    * Členité okraje s vysokým zakřivením (úzké výstupky, úzké zálivy atd.) jsou často následkem prahování.
        * Aby se tomuto efektu zabránilo, je užitečné použít morfologické transformace po prahování (např. otevření/zavření). 
        * Pozor, po morfologickém zpracování se u objektů změní i další parametry (nejen hranice, ale i např. obvod)!
::: 
        
:::info
**Souřadnice**
* Souřadnice rohu obrazově orientovaného ohraničujícího obdélníku (např. levý dolní roh ve 2D)
* Souřadnice jednoho z hraničních bodů (např. nejspodnější levý hraniční bod ve 2D)
* Geometrický střed objektu
    * Průměr všech souřadnic pixelů/voxelů objektu, tj. součet všech souřadnic pixelů/voxelů objektu dělený počtem pixelů/voxelů objektu
* Těžiště objektu (centroid) 
    * Vážený průměr všech souřadnic pixelů/voxelů objektu s váhami danými intenzitami, tj. součet všech souřadnic pixelů/voxelů objektu vynásobený jejich intenzitami dělený součet intenzit všech pixelů/voxelů objektu) 
:::

:::info
**Prostorová orientace**
* Směr podlouhlého předmětu 
    - Směr delší strany minimálního ohraničujícího obdélníku
* Lze rozlišit obrazově objektově orientovaný ohraničující obdelník
* Lze vypočítat minimální poloměr a maximální poloměr (vzhledem ke středu objektu a hranici objektu), jejich úhly a také poměr těchto dvou délek a/nebo úhlů. 
* Orientaci lze spočítat z centrálních momentů druhého řádu
    * Měření orientace pomocí momentů je obecně velmi přesné
:::

:::info
**Průměrná a maximální intenzita**
* Průměrná a maximální hodnota přes všechny intenzity pixelů/voxelů objektu
* Lze vypočítat rozptyl intenzit jako směrodatnou odchylku těchto hodnot 
:::

:::info
**Velikost (plocha nebo objem)**
* Počet pixelů/voxelů objektu vynásobený velikostí pixelů/voxelů
* Vznikají artefakty způsobené hraniční pixelací
    * ![](https://i.imgur.com/l83odGw.png =400x)
:::

:::info
**Podlouhlost (elongatedness/eccentricity)**
* Podlouhlost objektu lze spočítat přímo z centrálních momentů druhého řádu
:::

:::info
**Obvod nebo povrch**
* Počet hraničních pixelů/voxelů vynásobený konstantou (např. šířkou čtvercového pixelu ve 2D)
* Generování hraničního řetězce (Freemanův kód) ve 2D a výpočet součtu
    * Součet horizontálních/vertikálních kroků násobených konstantou (např. 1) plus součet diagonálních kroků násobených jinou konstantou (např. $\sqrt{2}$)
* Vznikají artefakty způsobené hraniční pixelací
:::

:::info
**Profily neboli projekce**
* Projekce počtu jedničkových pixelů podél dané osy
* Je možné je použít pro analýzu struktury dokumentů
* Často se provádí podél hlavní osy
* ![](https://i.imgur.com/PlIhT75.png =300x)
:::

:::info
**Topologické vlastnosti**
* Mezi důležité vlastnosti objektů patří i jejich topologické vlastnosti, například počet souvislých komponent $N_C$ nebo počet souvislých děr $N_H$
* Eulerovo číslo $N_E$ udává rozdíl mezi počtem souvislých komponent a počtem jejich děr
    * ![](https://i.imgur.com/pK1pp4Q.png =400x)

:::


### Atributy textur
* Nejsou vázány na výsledek segmentace (region $R$), ale lze je použít i například před segmentací textur
* Typicky kombinují informaci z lokálního okolí
    * Například pomocí matice současného výskytu (co-occurrence matrix – Haralick 1973)
    * Lokální binární vzory (local binary patterns – Ojala 1996)

:::info
**Matice současného výskytu**
* Počítá jak časté jsou výskyty stejných hodnot v zadaném směru $\Delta x$, $\Delta y$
* Před výpočtem se často provádí kvantizace kvůli snížení počtu úrovní
:::

:::info
**Haralickovy rysy**
* Nad maticí současného výskytu po normalizaci se počítají různé statistiky/charakteristiky
* Např. různé momenty, míra korelace hodnot, apod.
* ![](https://i.imgur.com/oxSVQ2Y.png =500x)
:::

:::info
**Lokální binární vzory**
* Základní myšlenka spočívá v nahrazení pixelu na dané pozici 8-bitovým číslem, které udává výsledek porovnání dané hodnoty s hodnotami v osmi okolí
* Deskriptor zobecněn i pro větší okolí
* V praxi dosahuje často velmi dobrých výsledků (je invariantní k nerovnoměrnému osvětlení)
* Byl rozšířen i na rotační nezávislost
* ![](https://i.imgur.com/dRNUlZ6.png =500x)

:::


### Lokální detekce vlastností + popisy
:::success
* Kromě globálních prvků (jako jsou Haralickovy) lze počítat i lokální prvky = zajímavé/dominantní body v obrázku.
* To lze provést např. detekcí lokálních maxim a minim v pyramidě HPF napříč stupnicemi.
* Každému zajímavému bodu jsou přiřazeny jeho souřadnice a další charakteristiky (např. síla, orientace atd.)
* Nejedná ani o segmentaci (obrázek není rozdělen na oblasti), ani o detekci objektů (objekty nejsou extrahovány, pouze jejich rohy a další dominantní body, ale bez znalosti toho, které body patří ke kterému objektu)
* Lokální extrakce rysů je však více než jen globální analýza obrazu a je dobrým kompromisem, pokud není třeba extrahovat objekty nebo je těžké je extrahovat. 
:::

## Klasifikace objektů
:::success
* Klasifikační krok se pokouší zařadit obrázek nebo objekty detekované během kroku segmentace do několika tříd.
* Vlastnosti jednotlivých tříd musí být známy předem.
* Počet tříd je také obvykle znám předem – je odvozen od specifikace problému.
* Obrázky/objekty jsou obvykle klasifikovány podle jejich popisů, které jsou porovnávány s popisy jednotlivých tříd.
:::

### Přístupy ke klasifikačním krokům
1. Formální popis je konstruován (známý algoritmus).
    * Pokud lze napsat formální popis, lze klasifikátor celkem snadno realizovat pomocí vhodného programovacího jazyka. 
    * Formální popis složitějších tříd je často psán přesně pomocí formálních gramatik (formálních jazyků), predikátové logiky nebo jiných matematických nástrojů.
2. Klasifikátor je trénován na sadě příkladů (učení pod dohledem).
    * Počítač se krok za krokem učí, který vstup (který atributový vektor) odpovídá které třídě.
    * Nejčastějším přístupem ke klasifikaci založeném na učení na sadě příkladů jsou neuronové sítě nebo přístup pomocí vektorových strojů.
3. Klasifikátor není vyškolen (učení bez dozoru).
    * Klastrová analýza je typicky aplikována na body ve vícerozměrném prostoru, kde osy prostoru odpovídají měřeným atributům obrázku/objektu a každý bod představuje jeden obrázek/objekt. 
:::

:::info
**Učení pod dohledem (Supervised Learning)**
* **Lineární klasifikátory**
    * Body (atributové vektory) ve vícerozměrném prostoru atributů jsou rozděleny do shluků pouze pomocí nadrovin (hyperplane), žádné jiné hranice tříd nejsou povoleny.
    * ![](https://i.imgur.com/zsrmvOo.png =200x)
* **Support Vector Machine (SVM)**
    * Cílem je najít takovou nadrovinu, která má největší okraje 
    * ![](https://i.imgur.com/gY4FvPD.png =420x)
    * je povolen n-rozměrný prostor transformovat nelineárně, tak že konečná hranice není nadrovina 
* **Klasifikátory k-nejbližších sousedů (k-NN)**
    * Neznámému vstupu je přiřazena stejná třída jako většině jeho nejbližších k sousedů (které patří do trénovací datové sady se známými třídami) 
:::

:::info
**Učení bez dozoru (Unsupervised Learning)**
* **Hierarchické shlukování** (shlukování založené na konektivitě)
    * Podobně rostoucím regionům
    * Začne každým objektem/obrázekem patřím do jiné třídy
    * Sloučí ty nejbližší 
    * Následně se iterativně uvolňuje kritérium blízkosti (nebo naopak, jako rozdělení regionů)
    * Vytváří celou hierarchii klastrů: 
        * ![](https://i.imgur.com/f8nEkne.png =400x)
* **K-Means** (shlukování založené na centroidech)
    * Shluky jsou reprezentovány centrálním vektorem, který nemusí být nutně součástí souboru dat
    * Počet shluků pevně stanoven na $k$, K-Means shlukování je optimalizační problém: 
        * Nalezni $k$ středů shluků a přiřaď objekty nejbližšímu středu shluku tak, že součet čtverečních vzdáleností od středu shluku je minimalizován.
    * Samotný optimalizační problém je NP-těžký, takže běžný přístup je hledat pouze přibližná řešení, typicky se používá následující iterativní algoritmus: 
        * ![](https://i.imgur.com/UOW0fhW.png)
* **Shlukování založené na distribuci** 
    * Předpokládá, že body ve shluku jsou rozmístěny podle určitého známého rozdělení, typicky Gaussova rozdělení
    * Pomocí optimalizační procedury přizpůsobí známou distribuci datům tak, aby vytvořily shluky:  
        * ![](https://i.imgur.com/Pf7dueU.png =200x)
:::

### Měření kvality klasifikace
* Je třeba Ground Truth (GT)
    * ![](https://i.imgur.com/F4cSdae.png)

#### Získání Ground Truth (GT)
:::success
* GT není známá, je určeno odborníky
    * Obvykle pouze pro určitou část obrazových dat
    * Velmi subjektivní, odhady se u komplikovaných nebo rozmazaných/zašuměných snímků mohou mezi odborníky výrazně lišit
    * Za účelem snížení chyb v GT může být zapojeno několik odborníků a jejich GT sloučeny (většinové hlasování)
* GT je známá, je určena z předchozích znalostí
    * Získávají se objekty známých vlastností (standardizované testovací objekty, fantomy lidských orgánů v lékařském zobrazování atd.)
    * GT lze snadno určit, protože máme k dispozici obraz i skutečný předmět, který jsme získali (můžeme se ho dotýkat, měřit ho, počítat jeho vlastnosti atd.)
* GT je přesně a úplně známá pro velký soubor dat
    * Vstupní objekty i jejich obrázky jsou simulovány, tj. používáme syntetické testovací objekty v paměti počítače (digitální fantomy)
     * Můžeme generovat velké datové sady společně s GT, bez lidské práce! 
:::

#### Kontrola kvality Ground Truth (GT)
:::success
* Výsledky segmentace
    * GT = správná binární maska
    * Kvalita (přesnost) algoritmu lze měřit na základě korelačních koeficientů (překrytí vypočítané a GT masky) např. Jaccardův index podobnosti nebo Diceův koeficient
* Výsledky měření (pozice, délka, velikost/plocha/objem)
    * GT = správná hodnota měření
    * Chyba = rozdíl mezi naměřenou hodnotou a GT
    * Přesnost lze měřit na základě rozložení chyb (histogram)
* Výsledky klasifikace
    * GT = správná třída pro každý pixel/voxel/objekt/obrázek
    * Kvalita algoritmu lze měřit na základě poměrů TP/FP/TN/FN
* Statistické výsledky
    * GT = správná míra výskytu (v procentech) pro každou třídu
    * Přesnost lze měřit na základě srovnání hodnot v procentech 
:::

## Mapa vzdáleností 
- Každému bodu popředí přiřadíme jeho vzdálenost (v dané metrice) k nejbližšímu bodu pozadí
- Význam popředí a pozadí lze zaměnit (vzdálenosti počítáme pro body pozadí) a případně kombinovat do jednoho obrazu pomocí různého znaménka
- ![](https://i.imgur.com/Lp36nIQ.png =350x)
- ![](https://i.imgur.com/4KfTW3L.png =250x)

> [color=#666666] **Algoritmus pro $D_4$ a $D_8$**
> * Nutné pouze 2 průchody
>   * Zleva doprava a shora dolů (dopředný)
>   * Zprava doleva a zdola nahoru (zpětný)
> * Aktualizace vzdálenosti na základě již navštívených pixelů
> ![](https://i.imgur.com/Y2k2LRP.png =400x)
> ![](https://i.imgur.com/1N2rMyM.png =400x)

* Popsaný dvouprůchodový algoritmus lze použít pro každou regulární metriku
    * Regulární  vzdálenost je možné propagovat inkrementálně
* Euklidovská metrika není regulární


:::info
**Využití**
* Oddělení dotýkajících se objektů
* Výpočet morfologických operátorů
* Výpočet geometrických reprezentací a měr (např. kostry, Voroného diagramy, osy souměrností, apod.)
* Navigace robotů
* Porovnávání vzorů
:::

### Časté metriky vzdálenosti
* Euklidovská metrika $D_E$ ![](https://i.imgur.com/bEbBEgZ.png =70x)
* Šachovnicová metrika $D_8$ ![](https://i.imgur.com/a5Zytcc.png =70x)
* Taxikářská metrika $D_4$ ![](https://i.imgur.com/tvq9GiC.png =70x)

![](https://i.imgur.com/8pWJgbR.png =200x)


### Voroného diagramy
- Každému objektu je přiřazena taková oblast definičního oboru, že všechny body v této oblasti mají menší vzdálenost k přiřazenému objektu než k jakémukoli jinému objektu
- Lze spočítat aplikací algoritmu záplava (MM) na mapu vzdáleností
    - ![](https://i.imgur.com/NxHcMR1.png =500x)
- Dilatace s různými aproximacemi disku pomocí prahování mapy vzdáleností
    - ![](https://i.imgur.com/Zp70CPR.png =450x)

### Porovnávání vzorů
- Vzory v obraze lze vyhledávat pomocí křížové-korelace (odpovídá konvoluci s překlopením vzoru $P$)
    - ![](https://i.imgur.com/sX1Urz4.png =500x)
- Normalizovaný součet vzdáleností (matching score) pixelů vzoru $P$ od objektů v obraze
    - ![](https://i.imgur.com/YZDFv2c.png =500x)



## Matematická morfologie
* Transformace 
    * Binární
    * Šedotónové
* Transformace fungují v lokálním sousedství každého pixelů (podobně jako konvoluce) definovaném tzv. strukturním prvkem. 
* Strukturní prvek může být čtvercový, křížový nebo jakýkoli jiný tvar s definovaným počátkem (referenční bod).
* Referenční bod nemusí patřit do masky definované elementem
* ![](https://i.imgur.com/BpBfLYq.png)


### Dilatace
::: success
* Nahradí centrální pixel maximem jeho sousedů. 
:::

### Eroze
::: success
* Nahradí centrální pixel minimem jeho sousedů. 
:::

### Uzavření
::: success
* Dilatace následovaná erozí.
* Spojuje husté aglomerace lokálních maxim dohromady, vyplňuje malé dírky a vyhlazuje hranice.
:::

### Otevření
::: success
* Eroze následovaná dilatací.
* Odstraňuje jednotlivá lokální maxima, tenké čáry a rozděluje objekty spojené úzkou cestou. 
:::

> [color=#666666] **Příklad:**
> a. Originál
> b. Po 2 cyklech eroze
> c. Po 4 cyklech eroze
> d. Po 7 cylech eroze - separováno
> e. po 4 cyklech dilatace z obrázku d.
> f. po 7 cyklech dilatace (logika prevence spojení objektů)
> g. po 9 cyklech dilatace s omezením na vlastnosti původních okraju
>![](https://i.imgur.com/mRE5xa9.png =500x)

### Bílý top-hat (WTH)
::: success
* Definován jako rozdíl mezi původním obrázkem a jeho otevřením
* Extrahuje světlé skvrny (lokální maxima) nezávisle na jejich intenzitě, bere v úvahu pouze tvar
* Lze použít pro korekci nerovnoměrného osvětlení (tmavé pozadí) 
:::

### Černý top-hat (BTH)
::: success
* Definován jako rozdíl mezi uzavřením a původním obrázkem
* Komplementární operátor k bílému top-hatu
* Extrahuje tmavé skvrny (lokální minima) nezávisle na jejich intenzitě, bere v úvahu pouze tvar
* Lze použít pro korekci nerovnoměrného osvětlení (světlé pozadí) 
:::

> [color=#666666] **Příklad:**
> ![](https://i.imgur.com/14W7Oot.png =500x)

### Watershed
::: success
* Simulace zvyšování hladiny vody krok za krokem. 
* Šedotónový obraz je považován za topografický povrch s údolími a vodních předělů
* spočítáme rozdělení pomocí stavby přehrad mezi údolími během procesu zaplavení. 
:::

![](https://i.imgur.com/yneIZMf.png)

::: success
* Vyhlazení obrazu (obvykle Gaussův filtr), velikost jádra závisí na velikosti objektu a také na šumu.
* Určení maximální intenzity (MaxI) a minimální intenzity (MinI) celého snímku.
* Určení tzv. markerů, tedy zárodků budoucích objektů. Lze provést např. pomocí minimálního filtru.
* Binární obrázek (BinImg) = prázdný obrázek (nuly)
* Pro CurI = MinI až MaxI provedeme růst oblasti ze markerů (nebo již existujících oblastí BinImg), aby byly zahrnuty všechny pixely s intenzitou nižší nebo rovnou CurI. Regiony nesmí splývat!
* BinImg nyní definuje území objektů. Počet objektů se rovná počtu markerů. 
:::

:::info
* Obvykle se používá na gradient (Sobelův filtr) za účelem nalezení hranic objektů
* Může fungovat bez markerů, v tomto případě jsou markery všechny lokální minima na obrázku (v sousedství 3x3) 
* Zaplavení lze provést v opačném směru pro invertované obrázky (anti-watershed)
* Markery lze určit ručně nebo automaticky pomocí lepšího přístupu než minimální filtr, např. pomocí transformace vzdálenosti 
    * Obraz je prahován (dvouúrovňové prahování) a jsou určeny hranice 
    * Nalezne se nejkratší vzdálenost každého pixelu k nejbližšímu hraničnímu bodu. 
    * Ty pixely, které mají velkou vzdálenost k hranici (větší než předem definovaný práh) jsou brány jako markery. 
:::

