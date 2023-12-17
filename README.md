# Applikationssicherheit umsetzen
Verwenden Sie nicht einfach den Modulnamen. Geben Sie einen passenden, aussagekräftigen Titel für Ihr ePortfolio an.
LB-183 von Matteo Jakob

In der Einleitung beschreiben Sie kurz den Inhalt des ePortfolios, damit die Lesenden einen Überblick haben, was sie erwartet.
Abschnitt pro Handlungsziel
Pro Handlungsziel ist ein Abschnitt mit folgendem Inhalt zu erstellen:

## Handlungsziel 1
Aktuelle Bedrohungen erkennen und erläutern können. Aktuelle Informationen zum Thema (Erkennung und Gegenmassnahmen) beschaffen und mögliche Auswirkungen aufzeigen und erklären können.

##### Artefakt
LA_183_04_Grundbegriffe.docx

##### Zielerreichung
Ich habe das Handlungsziel erreicht, da das Artefakt eine Analyse von verschiedenen Sicherheitsszenarien und deren Auswirkungen auf die Schutzziele enthält.

##### Erklärung
Das Artefakt umfasst eine Bewertung von fünf Sicherheitsszenarien hinsichtlich der Schutzziele Vertraulichkeit, Integrität und Verfügbarkeit. Die Bewertung erfolgt auf einer Skala von 0 bis 2, wobei 0 für "gar nicht", 1 für "teilweise" und 2 für "erheblich" steht.

Auch beinhaltet es die wichtigsten Sicherheitsrisiken für eine Webapplikation mittels OWASP, es werden mehrere Fragen dazu beantwortet.

##### Rückblick
Die Umsetzung des Artefakts war erfolgreich, da es eine detaillierte Analyse der Sicherheitsszenarien bietet und die Auswirkungen auf die Schutzziele bewertet. Es ermöglicht eine fundierte Beurteilung der Bedrohungen in Bezug auf Vertraulichkeit, Intedgrität und Verfügbarkeit.

## Handlungsziel 2
Sicherheitslücken und ihre Ursachen in einer Applikation erkennen können. Gegenmassnahmen vorschlagen und implementieren können.

![code before](https://github.com/[username]/[reponame]/blob/[branch]/image.jpg?raw=true)
![code after](https://github.com/[username]/[reponame]/blob/[branch]/image.jpg?raw=true)

Ich habe dieses Handlungsziel erreicht, indem ich eine Sicherheitslücke in einem Codebeispiel erkennt habe (Bild 1) und diese behoben habe (Bild 2).

Das Artefakt hat eine SQL-Injection Sicherheitslücke, das heisst dass es nicht eine genügende Eingabevalidierung gibt und somit potenzielle Angreiffer mit einem einfachen prompt, wie ``"'; DROP TABLE *; --"``,  die Datenbank kompromieren können. 
Um dies zu verhindern, habe ich direkt den User gefiltert, anstatt einen String zu machen.

Die Umsetzung des Artefakts war teilweis erfolgreich, da ich eine SQL-Injection verhindern konnte, jedoch gibt es noch weitere Sicherheitslücken, wie Cross Site Scripting (XSS). XSS ist eine Angriffstechnik, bei der bösartiger Code in Webseiten eingefügt wird, der dann von anderen Nutzern dieser Seite ausgeführt wird. Dadurch können sensible Daten gestohlen werden.

Hier sind Codebeispiele für XSS:
```html
<script>alert('XSS')</script>. 
<a href="javascript:alert('XSS')">Klick</a> 
<img src="kitten.png" onload="alert('XSS')"/>
```
Um dies zu beheben muss die Eingabe escaped werden, siehe pics/3.png

## Handlungsziel 3
Mechanismen für die Authentifizierung und Autorisierung umsetzen können.

1. Wählen Sie ein Artefakt, welches Sie selbst erstellt haben und anhand dem Sie zeigen können, dass Sie das Handlungsziel erreicht haben.

2. Weisen Sie nach, wie Sie das Handlungsziel erreicht haben. Verweisen Sie dabei auf das von Ihnen erstellte Artefakt. Das Artefakt muss im ePortfolio sichtbar oder verlinkt sein.

3. Erklären Sie das Artefakt in wenigen Sätzen. Sollte das Artefakt mehrere Handlungsziele beinhalten dürfen Sie die Erklärung auch zusammenfassen.

4. Beurteilen Sie die Umsetzung Ihres Artefakts im Hinblick auf das Handlungsziel kritisch. Sollten gewisse Aspekte des Handlungsziels fehlen, haben Sie die Möglichkeit, in diesem Teil darauf einzugehen.

## Handlungsziel 4
Sicherheitsrelevante Aspekte bei Entwurf, Implementierung und Inbetriebnahme berücksichtigen.

1. Wählen Sie ein Artefakt, welches Sie selbst erstellt haben und anhand dem Sie zeigen können, dass Sie das Handlungsziel erreicht haben.

2. Weisen Sie nach, wie Sie das Handlungsziel erreicht haben. Verweisen Sie dabei auf das von Ihnen erstellte Artefakt. Das Artefakt muss im ePortfolio sichtbar oder verlinkt sein.

3. Erklären Sie das Artefakt in wenigen Sätzen. Sollte das Artefakt mehrere Handlungsziele beinhalten dürfen Sie die Erklärung auch zusammenfassen.

4. Beurteilen Sie die Umsetzung Ihres Artefakts im Hinblick auf das Handlungsziel kritisch. Sollten gewisse Aspekte des Handlungsziels fehlen, haben Sie die Möglichkeit, in diesem Teil darauf einzugehen.

## Handlungsziel 5
Informationen für Auditing und Logging generieren. Auswertungen und Alarme definieren und implementieren.

1. Wählen Sie ein Artefakt, welches Sie selbst erstellt haben und anhand dem Sie zeigen können, dass Sie das Handlungsziel erreicht haben.

2. Weisen Sie nach, wie Sie das Handlungsziel erreicht haben. Verweisen Sie dabei auf das von Ihnen erstellte Artefakt. Das Artefakt muss im ePortfolio sichtbar oder verlinkt sein.

3. Erklären Sie das Artefakt in wenigen Sätzen. Sollte das Artefakt mehrere Handlungsziele beinhalten dürfen Sie die Erklärung auch zusammenfassen.

4. Beurteilen Sie die Umsetzung Ihres Artefakts im Hinblick auf das Handlungsziel kritisch. Sollten gewisse Aspekte des Handlungsziels fehlen, haben Sie die Möglichkeit, in diesem Teil darauf einzugehen.






Selbsteinschätzung des Erreichungsgrades der Kompetenz des Moduls
Geben Sie eine Selbsteinschätzung zu der Kompetenz in diesem Modul ab. Schätzen Sie selbst ein, inwiefern Sie die Kompetenz dieses Moduls erreicht haben und inwiefern nicht. Es geht in diesem Abschnitt nicht darum, auf die einzelnen Handlungsziele einzugehen. Das haben Sie bereits gemacht. Begründen Sie ihre Aussagen. 
