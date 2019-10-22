# Audio-widget-I2S
**Interfata audio USB utilizand un AVR32 pe protocolul I2S**

## Introducere
>
>Inca din timpurile dinaintea erei digitale, consumatorii de audio **asa zisii audiofili** au fost in cautare de metode pentru reducerea la minim a distorsiunilor transmise catre difuzoare,si implicit,percepute de urechea umana.In vremea respectiva,data fiind si complexitatea destul de redusa a dispozitivelor,ideea reducerii distorsiunilor in sisteme era destul de limitata de topologiile propuse,de aceea aparitia sistemelor digitale de procesare in audio a dus la reducerea drastica a neajunsurilor semnalelor analogice.Pentru **puristi** , circuitele digitale si DSP-urile sunt considerate _un sacrilegiu_ adus audiofiliei,dar este unul justificat,deoarece in ciuda complexitatii mai ridicate fata de un circuit analog pur , sistemele digitale sunt mult mai eficiente fata de complementele sale,iar posibilitatea customizarii lor este foarte mare (spre deosebire de un circuit analogic unde,pentru a modifica o caracteristica trebuie regandit tot circuitul,in cazul unui circuit digital cu uC este doar o problema de software).

>Aparitia interfetelor audio s-a facut odata cu dezvoltarea PC-urilor si inregistrarea de muzica/sunete pe medii digitale , in pofida benzilor magnetice si a discurilor de vinil , deoarece era necesara o metoda de redare a sunetelor stocate pe calculatoarele vremii. De altfel,primele asa zise **placi de sunet** si-au facut aparitia prin calculatoare in jurul generatiei 80486 , cand deja exista o oarecare structura de operare stabila pe 16 biti , iar pentru o esantionare simpla a sunetelor erau arhisuficiente instructiunile implementate de Intel in procesoarele respective.
>>_Daca facem referinta la vremurile actuale,se poate spune ca practic nici nu se punea problema de calitate sau termenul de **Hi-Fi** in acele vremuri,dar acele sunete au reprezentat ceea ce urma sa fie boom-ul erei digitale_
>
>Ca si istorie a acestei ere in care traim,se pot scrie foarte multe lucruri deja cunoscute,asa ca voi incerca sa ma concentrez pe esential.

>Ideea de "calitate audio" pe digital a aparut in momentul lansarii pe piata mondiala a CD-ului (acel lucru care acum multora dintre noi ni se pare atat de outdated,ori chiar nu il mai folosim),dar el a avut cea mai mare influenta in aparitia de tehnologii de compresie si codare audio.Ca si concept,ideea de CD a aparut prin anul 1980 ,cand Philips si Sony au incheiat un parteneriat de cercetare in domeniul digital,dorindu-se un mediu de stocare unic transmisibil,care sa aibe un spatiu de stocare a informatiilor mult mai mare decat cel al dischetelor "floppy" si al benzilor magnetice,dar totusi sa nu coste cat un hard disk al vremurilor (memo: un hard disk de tip Winchester avea o capacitate de maxim 128MB , cantarea in jur de 15-20kg si era cam cat un desktop SFF actual ca dimensiuni si costa in jur de ~5.000 de dolari unul?).
>>Standardul CD a fost definitivat in jurul anului 1983,iar in urma intelegerii realizate intre cele 2 concerne se va realiza trecerea industriei audio catre acest mediu.

>Se pune problema "compresiei audio".In a nutshell,algoritmul de compresie audio se refera la un standard de scriere-redare a sunetelor pe calculatoare. Calculatoarele,la momentul respectiv avand sisteme de operare stabile precum UNIX si DOS (vorbind de anii '80,dar regula se aplica si in prezent pe sistemele actuale),pentru a reda sunetele stocate pe medii precum hard-diskul se folosesc de niste software numite **codec** .
>>Codecurile,in esenta,sunt niste pseudo-drivere pe care sistemul de operare le interpreteaza atunci cand se executa in sistem un fisier cu extensia ce corespunde codecului respectiv.
>>Adica,spre exemplu, avem un fisier cu extensia ".mp3" , SO va sti ca la instantierea acestui fisier va trebui sa apeleze un Codec LAME(Lame Ain't an MP3 Encoder - care este o derivatie imbunatatita de-a lungul anilor de diversi programatori din conceptul original gandit de institutul Fraunhofer din Germania).

>De altfel , toate procedeele de redare/compresie si extensiile de sistem au fost posibile datorita codarii PCM (Pulse Code Modulation).
PCM reprezinta transformarea semnalelor analogice in semnale digitale printr-un algoritm de esantionare constanta a amplitudinii semnalului analogic ,iar fiecare esantion este cuantizat la cea mai apropiata valoare dintr-o gama de cuante apropiate.
>>**Codarea PCM prezinta 2 proprietati ce determina calitatea conversiei semnalului original raportat la acesta : rata de esantionare a semnalului si _bit depth_,care reprezinta cantitatea de biti al fiecarui esantion realizat (un CD-Audio prezinta un bit depth de 16 biti per esantion , pe cand un Blu-ray sau DVD-Audio 24 de biti per esantion).**
>>> Extensiile posibile unei compresii PCM sunt ".wav" ,".L16",".PCM",".AIFF",".AU"

>Pe baza codarii/compresiei PCM a fisierelor audio, cei de la Philips au realizat in anul 1986 (si ulterior revizuit in 1996) un protocol de transmisie seriala al datelor audio intre dispozitive / circuite integrate , protocol care inca se foloseste cu succes si in prezent in dispozitivele audio. Este cunoscut sub acronimul I2S sau IIS **Inter-IC Sound** si se bazeaza pe o magistrala sincrona de transmisie de date (clockul de transmisie intre circuite este separat de magistrala de date,deci implicit receptorul nu mai are nevoie de un circuit suplimentar de refacere a frontului de ceas trimis de emitator).
>Protocolul este relativ usor de interpretat, in principal avem de-a face cu 3 semnale : bit clock line(**BCLK**) , word clock line (cunoscut sub denumirea oficiala de 'word select' **WS** sau 'left-right clock' **LRCLK** ) si cel putin o linie de date multiplexata **SD**

>>Adeseori se mai poate intalni peste cele 3 semnale cunoscute inca un semnal de tact (care nu face parte din protocol) cu rolul de a sincroniza operatiile interne ale DAC-urilor cu cele ale controllerului . Este gasit sub acronimul de "Master clock" **MCLK**
>>>Frecventa tipica de lucru se calculeaza dupa formula : **MCLK = 256 * LRCLK**

![500px-I2S_Timing svg](https://user-images.githubusercontent.com/54248886/66950287-bf636680-f060-11e9-9627-63df32e9b9f9.png)

###### Cum functioneaza protocolul:

>Semnalul de "Word Select" permite dispozitivului receptor sa 'vada' ce canal urmeaza sa fie trimis , deoarece I2S permite ca pe aceeasi linie de date sa fie trimise 2 canale,fiind un semnal cu factor de umplere 50% si frecventa egala cu cea de esantionare a semnalului trimis.
>>**Pentru semnale stereo , in brosura de specificatii a I2S se spune ca semnalul audio de pe canalul stang se transmite pe palierul de 0 al Word Select , iar canalul drept pe palierul de 1 . De obicei , semnalul "Word Select" este sincronizat dupa fronturile cazatoare ale semnalului BCLK , deoarece datele sunt zavorate pe fronturile crescatoare**.
>>
>Se stabileste semnul liniei de date si se codeaza in complement fata de 2 cu primul bit MSB-ul . **Acest lucru permite ca numarul de biti per cadru sa fie arbitrar , fara a fi nevoie de "negociere" intre emitator si receptor **.

## Specificatii montaj:

1.Microcontroller utilizat:  Atmel AT32UC3A3256 
 >>>  A complete system-on-chip 32-bit AVR microcontroller. It is designed for cost-sensitive embedded applications that require low power consumption, high code density and high performance.

 >>> The microcontroller's Memory Protection Unit (MPU) and fast, flexible interrupt controller support the latest real-time operating systems. Higher computation capabilities are achievable using a rich set of DSP instructions. The device incorporates on-chip flash and SRAM memories for secure and fast access. 64 KBytes of SRAM are directly coupled to the 32-bit AVR UC3 for performance optimization. Two blocks of 32 Kbytes SRAM are independently attached to the high speed bus matrix for real ping-pong management.

 >>> The microcontroller achieves exceptionally high data throughput by combining the multi-layered 32-bit AVR databus,128 KB on-chip SRAM with triple high speed interfaces, multi-channel peripheral, memory-to-memory DMA controller, high-speed USB embedded host, SD/SDIO card, MLC NAND flash with ECC, and SDRAM interfaces.

 >>> This device features 256KB internal high-speed flash and full-duplex multi-channel I2S audio interface.

2.Utilizeaza USB Audio Class 1 / 2.

3.Capabilitati de redare audio pana la 24bit/196kHz (bit depth / sampling rate).
