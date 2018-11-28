# Rootless Containers

Presenteras på Tech Days by Init i Stockholm, 29 November 2018.

## Beskrivning

Rootless Containers

En spännande utveckling i container-världen är rootless containers. I
den här presentationen kommer jag ge en kort beskrivning av vad
rootless containers är och hur de fungerar, vilka problem de försöker
lösa och varför man ska bry sig om dem.

## Presentation

### Bakgrund

Jag jobbar på SUSE och har själv inte hållit på direkt med rootless
containers, men jag har pratat en del med de som jobbar med det och
jag tycker det är ett fascinerande ämne eftersom det ger en inblick i
hur containers fungerar både i user space och i
kärnan. Förhoppningsvis kommer jag kunna förmedla min fascination även
om jag kanske kommer begå en del misstag då jag själv inte är expert
på det här. Hoppas ni kan ha överseende med det.

### Introduktion

Låt oss säga att en användare av ett fleranvändarsystem vill köra
Python 3-kod på ett system som bara har Python 2 installerat.

I teorin skulle de kunna köra en container med Python 3 installerat
och köra sin kod i den. Men tyvärr behöver de en container-runtime för
att köra sin container, och systemadministratören är konservativ (med
rätta) och vill inte installera ny och obeprövad mjukvara. Så vad göra?

Tänk om det gick att skapa och köra containers helt som en vanlig
användare, utan tillgång till root?

### Målet

### Vad är en container

Den huvudsakliga pusselbiten som krävs för att implementera containers
i Linux är namespaces, eller namnkontext: Med namespaces kan vi ge
olika processer sina egna miljöer där identifierare i systemet är
unika för den processen. T.ex. så kan olika processer kan ha olika
filsystem monterade, så att för den här processen är /var något annat
än för andra processer.

Namespaces isolerar alltså processer från varandra, lite grann som
virtuella maskiner men inte alls och för enskilda processer.

Det som är mest intressant för oss just nu är user namespaces. Med det
menas isolering av användarlistan i ett system, så att en process ser
andra användare än en annan. Med hjälp av user namespaces skulle man
kunna skapa en miljö för en process där UID 0, det vill säga root, är
en annan användare än UID 0 för andra processer.

Den här möjligheten lades till i Linux 3.8, och har varit användbar
från Linux 3.19 ungefär som släpptes 2015.

### Container runtimes

runC är det som tar hand om att starta
