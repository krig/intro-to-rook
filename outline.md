# Rootless containers

Först och främst. Vad är en container? Kan någon förklara i stora drag
vad en container är?

Uppifrån sett så är en container ett sätt att isolera en process från
resten av systemet, så att den bara får en viss vy av resten av
systemet. Det vill säga, genom att starta en process i en container så
kan man ge processen sitt eget filsystem, sin egen bild av hur
nätverket ser ut, en begränsad insyn i vilka andra processer som kör
och så vidare.

Allt det här sköts av en container runtime, som ser till att sätta upp
precis det som processen behöver baserat på en beskrivning.
