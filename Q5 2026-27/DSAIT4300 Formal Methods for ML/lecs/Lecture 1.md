SDM lecture doorlezen

- SDM1 lezen

- Vertelt over hoe sommige systemen zich in continuous time bevinden en die moeilijker zijn om te verifyen, hiervoor bestaan tactieken zoals het infinite time -> bounded time, non-linear -> linear approx, continuous time -> discrete (die apart toe te passen zijn). Verifyen houd in om bewijs te leveren dat een bepaalde property holds voor elk soort keuze het systeem M zou kunnen maken, of ipv dat je een proof geeft je een counterexample geeft om te laten zien dat het niet hold. Certification is waar je een scalar hebt die je naar nul wil brengen ( of een onder een drempel wil houden hangt af van architectuur), test met meerdere wiskundige regels om er zeker van te zijn dat de property hold voor elke soort state in elk soort moment in tijd.

- SDM2 lezen

- Er zijn manieren om te checken of een bepaalde policy echt safe is, 1 van deze manieren zijn certificaten. Er zijn hier verschillende soorten van, zoals een lyapunov cert: het doel is om richting nul te gaan, barrier function: het doel is om tussen bepaalde grenzen te blijven. Naast het gebruiken van ready made certs kun je ze ook zelf vinden -> geautomatiseerde synthesis methoden. Search beperken tot een template (polynomial func/nn) en dan searchen over dit template (convex optim/gradient optim). Eenmaal een functie geleerd, maakt het niet gelijk een cert, maar een candidate -> checken of er coutnerexamples zijn, zo ja functie relearnen en dan weer testen, geld de functie voor alle soorten states -> officieel een cert
- Template kiezen: polynomial → lineaire combinatie van monomials met onbekende coëfficiënten (bv. graad 2 in 2D), trade-off: hogere graad = expressiever maar zwaarder te optimaliseren
- Convex optim werkt voor polynomials dankzij Sum-of-Squares (SOS): p(x)p(x) p(x) is SOS → p(x)≥0p(x)\ge0 p(x)≥0 gegarandeerd, en zoeken naar zo'n decompositie is zelf een convex probleem (SDP) → geen check per punt nodig, één keer oplossen bewijst het voor alle xx x tegelijk
- Lyapunov-condities worden dan: positiviteit en decrease-conditie elk omgezet naar "dit is SOS" (met kleine marge ϵ\epsilon ϵ), equilibrium-conditie (V(0)=0V(0)=0 V(0)=0) wordt gewoon een lineaire constraint op de coëfficiënten → alles samen als 1 SDP oplossen
- NN-template i.p.v. polynomial: veel expressiever, schaalt naar hoge dimensies/complexe systemen, maar verliest de convexiteit → geen exacte oplossing meer mogelijk
- Bij NN: elke conditie wordt een loss-term (hinge loss: max⁡(0,marge−conditie)\max(0, \text{marge} - \text{conditie}) max(0,marge−conditie)) i.p.v. harde constraint, train met gradient descent, totale loss = som van alle losses
- Takeaway/trade-off: SOS/convex = sterke garanties maar beperkt schaalbaar; NN/learning = schaalbaar en expressief maar heeft die extra verificatiestap (CEGIS) nodig → reliable vs scalable

**Volgende papers doorlezen en keuze maken:**
(**VeRecycle Paper**)[![[0052.pdf]]]

(**Supermartingales paper**)[![[00299-NeustroevG.pdf]]]