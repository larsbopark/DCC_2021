# the_hunt

> Da jeg i skrivende stund ikke har opgave beskrivelsen, har jeg udeladt den for nu forhåbentlig kan jeg få et screenshot.

## Første blik

Vi ved fra opgave beskrivelsen af det er en 'Reverse Engineering' opgave og en binary(**game**) er tilgængelig plus en commando til at forbinde til applicationen på den server vi skal udnytte. 

Det først jeg laver er et connect script, så jeg er fri for at copy-paste hvergang jeg vil tilgå serveren.
(scriptet ses herunder og ligger i samme mappe som dettte writeup som **connect.sh**)

    #!/bin/sh
    nc the-game.hkn 4242

Vi kan prøve at eksekvere filen:
![First prompt](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game1.png)
Vi kan se at vi skal finde en bestemt værdi. Vi kan prøve at skrive noget abritræt, i dette tilfælde er det: **heretolearn**

![Second promt](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game2.png)

Hvilket ser ud til ikke at føre nogle steder, da programmet returnere og lukker.

## Neddykning
Det første jeg som regel prøver er at kigge på hvilket strenge som kan udskrives, hvis flaget ligger i programmet som en streng er den hurtig fundet, men i dette tilfælde ved vi at vi skal have en 'shell'. Så vi åbner filen med[Cutter](https://cutter.re/). Cutter tilbyder en feature hvor at diverse functioner bliver oversat til C kode, hvilket gør assembly nemmere at forstå. Det første som altid er interessant at kigge på er selvfølgelig at kigge på `main()`.

![main()](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game3.png)

Som vi kan se har vi noget der ligner det første prompt. Vi har en funtion navngivet `clrscr()`, som sandsyndligvis er sammentrækning af `clear screen` hvilket rydder skærmen. Så har vi nogle `printf()` og `puts()`, ikke noget af særlige interesse. Dog virker `find_my_values()` interessant, lad os tage et blik på den. 

![find_my_values()](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game4.png)

Aha, her har vi resten af teksten som bliver prompted og `fgets()` som læser vores input. Det som jeg finder interessant er den betinget kontrolstruktur, som tjekker `var_4h` mod `uVar1` hvor vores input ligger (formoder jeg). Vi kan se at `var_4h` bliver tildelt hex værdien `0x2a`. Så hvis vores input matcher `0x2a` så køres funktion `more_values_here()`. Vi kan finde decimal værdien af `0x2a` ved at finde et tool online, men jeg vælger at starte **Python** og lader den konvertere tallet til decimal:

![Python](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game5.png)

Hvilket giver os `42`. Vi kan nu prøve at indtaste det når vi eksekvere `game`. 


Det ser ud til at vi gik forbi tjekket og nu har vi fået et nyt prompt. Min umiddelbare konklusion er at vi bare skal tilpasse vores input til de givne tjeks. 

![more_values_here()](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game7.png)

Så vi følger funktions kaldene, som vi ser her har vi nu `more_values_here()`, og det ser ud til at vi bare skal matche `0x38f`. Det er heldigvis hurtigt regnet ud med hjælp fra **Python**. Har valgt at udelade skærmbillede ;) Vi har at `0x38f = 911`. Det skriver vi ned også fortsætter vi bare med at følge kaldene.

![the_split_oh_no()](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game8.png)

Det ser ud til at der sker lidt mere here, vi har blandt andet to værdier `var_4h` og `var_8h`. Dog ser det kun ud til at vi skal matche `var_4h` for at kommer videre og kalde `split_one()`. Da `var_4h = 0x45` kan vi hurtig smide det i **Python**, og vi har at `0x45 = 69` så det skriver vi ned, og fortsætter.

### First Split, oopsieee

Det ser ud til at der sker en helt masse her. En masse initialiseringer og så ligner det ellers samme struktur som i de andre funktioner. Dog for at undgå `you_failed_bad()` skal vi undgå at betinglelsen bliver sand. Vi kan se at for at undgå første kontrolstruktur skal vi bare match `var_4h = 0x21` hvilket er `33` i decimal. 

![split_one1()](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game9.png)

Herefter har vi at der er noget `strcmp()` hvilket med en hurtig google-søgning, giver os et link til [tutorialspoint](https://www.tutorialspoint.com/c_standard_library/c_function_strcmp.htm). Vi kan se at to strenge bliver sammenlignet. Så hvis vi bare matcher strengen så kan vi bevæge os videre. Hvilket giver os `deep`

![split_one2()](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game10.png)

Nu er vi der snart, ser det ud som om. Det ser ud til at vi bare skal matche `misc` og `gang`, og så er vi i mål. 
12
> Og det finder vi så hurtigt ud af ikke er sandt. Da vi indtaster de forskellige værdier, ser det rigtig ud. Men efter det sidste prompt `One more level and we get the reward!` og vi indtaster `gang` får vi **Wrong value!** YIKES

Vi tager et ekstra kig på `atoi_was_here()` . Som bliver kaldt i den sidste kontrolstruktur lige før `return`.

![split_one2()](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game11.png)

Det ser ud som om der et tjek mere vi skal overholde, men det bliver givet som argument, og hvis vi kigger tilbage på hvor `atoi_was_here()` bliver kaldt, kan vi se at den bliver kaldt med `0`. Øv bøv. Så backtracker vi. Der må være en måde at kalde `atoi_was_here()` med `1` og få en shell!

### Second Split, on the right track

Vi opdager at man kan kalde en funktion `split_two()` hvis vi istedet for at matche `var_4h` at vi så matcher `var_8h = 0x60`, hvilket er `96` decimal. 

![split_two1()](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game12.png)

Vi bliver mødt af flere betinget kontrolstrukturer, men det ser ud til at vi nærmere os mål. Det første tjek skal vi matche `var_ch` hvilket ser ud til at blive tildelt `var_8h + (int32_t)var_4`, Og vi kan se at `var_4h = 0x62` og `var_8h = 0x16` som vi så ligger sammen med lidt hjælp fra **Python**. Hvilket giver os `120` i decimal. Den anden var lidt mere tricky, men prøvede mig lidt frem og vi kan se at `*(int32_t *)0x0 = var_ch + 0x4c1`, hvilket giver os `120+0x4c1 = 1337` i decimal. 

![split_two2()](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game13.png)

Som vi kan se har vi et `strcmp()` og en masse andet, men lad os bare smide `GTFO` ind og se hvad der sker.


## The solution, finally

Så nu kan vi indsætte det forskellige værdier. 
> 42 -> 911 -> 96 -> 120 -> 1337 -> GTFO

![the shell](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game14.png)

Så fik vi sørme adgang. Sådan nu kan vi finde flaget i hjemmebiblioteket. Dog har jeg ikke mulighed for at dele flaget da jeg ikke har gemt det og tjenesten er nede. 