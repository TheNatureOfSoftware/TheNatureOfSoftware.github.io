+++
date = "2017-06-04T11:58:13+02:00"
title = "Strategiska val för en teknikplattform"
tags = []
categories = []

+++

I ett tidigare inlägg pratade vi om att containers nu finns för alla plattformar.
Men vad får det för påverkan på våra strategiska val när det gäller teknik?

Microsoft har på senare år gjort stora satsningar på Open Source och
[.NET Core](https://www.microsoft.com/net/core#macos) finns idag för de flesta
[plattformar](https://github.com/dotnet/core-setup#daily-builds).

Tillsammans med Docker och containers gör det att vi kan bygga applikationer
och tjänster med nästan vilket språk och teknik som helst, förutsatt att vi
gör ett par strategiska val.

### Alla måste prata samma språk

Om vi nu vill bygga tjänster och applikationer och vill använda den teknik som
passar bäst, så är en förutsättning att alla kan prata samma språk.
[Web Service](https://en.wikipedia.org/wiki/Web_service) är ett sådant
standardiserat sätt, antingen via SOAP eller REST.

Vad man ofta inte tänker på
är att många av dagens system använder huvuddelen av tiden i processorn för att
översätta till och från mellan Web Service kommunikation och vårt valda programspråk.
Det finns idag bättre alternativ så som [Apache Thrift](https://thrift.apache.org/)
eller [Protocol Buffers](https://developers.google.com/protocol-buffers/) men
det bästa alternativet är förmodligen [GRPC](http://www.grpc.io/) som finns
för (nästan) alla [plattformar](http://www.grpc.io/docs/).

### Strategiska val med lager som en lök

Om vi tänker på våra strategiska val som lager på en lök. Längst in är kärnan,
där vi gör våra mest långsiktiga och strategiska val.

Vi kan sedan bygga våra beslut som lager där varje lager (val) anses ha en lägre
risk eller kortare livslängd. Vi låter kärnan i vårt strategiska val vara en
teknikplattform som bygger på containers och en orkestreringslösning så som
[Kubernetes](https://kubernetes.io), och att vi bygger en tjänstearkitektur med
kommunikation via GRPC där allt exponeras som en tjänst. Vi kan därifrån gå
vidare och titta på klientsidan.

#### På skrivbordet eller i mobilen

Webbapplikationer som används från skrivbordet kommer förmodligen vara det
vanligaste scenariot för många företag och organisationer. Men allt fler tittar
på hur man kan använda mobila enheter.

[Xamarin](https://www.xamarin.com/) låter dig bygga applikationer som
körs direkt på din OSX, iOS, Android eller Windows enhet.

[React](https://facebook.github.io/react/) eller [React Native](https://facebook.github.io/react-native/)
kan vara ett annat alternativ.

Vilket alternativ man än väljer så finns det stöd för [GRPC](http://www.grpc.io/)
som också har ett [plugin API](http://www.grpc.io/docs/guides/auth.html)
för autentisering.
