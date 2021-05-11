# the_hunt
> Da jeg i skrivende stund ikke har opgave beskrivelsen, har jeg udeladt den for nu forhåbentlig kan jeg få et screenshot.

Vi ved fra opgave beskrivelsen af det er en 'Reverse Engineering' opgave og en binary(**game**) er tilgængelig plus en commando til at forbinde til applicationen på den server vi skal udnytte. 

Det først jeg laver er et connect script, så jeg er fri for at copy-paste hvergang jeg vil tilgå serveren.
(scriptet ses herunder og ligger i samme mappe som dettte writeup som **connect.sh**)

    #!/bin/sh
    nc the-game.hkn 4242

Vi kan prøve at eksekvere filen:
![First prompt](https://raw.githubusercontent.com/larsbopark/DCC_2021/main/images/the_game1.png)