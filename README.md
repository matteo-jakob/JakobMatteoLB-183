# Applikationssicherheit umsetzen
Verwenden Sie nicht einfach den Modulnamen. Geben Sie einen passenden, aussagekräftigen Titel für Ihr ePortfolio an.
LB-183 von Matteo Jakob

In der Einleitung beschreiben Sie kurz den Inhalt des ePortfolios, damit die Lesenden einen Überblick haben, was sie erwartet.
Abschnitt pro Handlungsziel
Pro Handlungsziel ist ein Abschnitt mit folgendem Inhalt zu erstellen:

## Handlungsziel 1
Aktuelle Bedrohungen erkennen und erläutern können. Aktuelle Informationen zum Thema (Erkennung und Gegenmassnahmen) beschaffen und mögliche Auswirkungen aufzeigen und erklären können.

#### Artefakt
LA_183_04_Grundbegriffe.docx

#### Zielerreichung
Ich habe das Handlungsziel erreicht, da das Artefakt eine Analyse von verschiedenen Sicherheitsszenarien und deren Auswirkungen auf die Schutzziele enthält.

#### Erklärung
Das Artefakt umfasst eine Bewertung von fünf Sicherheitsszenarien hinsichtlich der Schutzziele Vertraulichkeit, Integrität und Verfügbarkeit. Die Bewertung erfolgt auf einer Skala von 0 bis 2, wobei 0 für "gar nicht", 1 für "teilweise" und 2 für "erheblich" steht.

Auch beinhaltet es die wichtigsten Sicherheitsrisiken für eine Webapplikation mittels OWASP, es werden mehrere Fragen dazu beantwortet.

#### Rückblick
Die Umsetzung des Artefakts war erfolgreich, da es eine detaillierte Analyse der Sicherheitsszenarien bietet und die Auswirkungen auf die Schutzziele bewertet. Es ermöglicht eine fundierte Beurteilung der Bedrohungen in Bezug auf Vertraulichkeit, Intedgrität und Verfügbarkeit.

## Handlungsziel 2
Sicherheitslücken und ihre Ursachen in einer Applikation erkennen können. Gegenmassnahmen vorschlagen und implementieren können.

#### Artefakt
![code before](https://i.imgur.com/aqPaOVO.png)
![code after](https://i.imgur.com/pvPS0Qx.png)

#### Zielerreichung
Ich habe dieses Handlungsziel erreicht, indem ich eine Sicherheitslücke in einem Codebeispiel erkennt habe (Bild 1) und diese behoben habe (Bild 2).

#### Erklärung
Das Artefakt hat eine SQL-Injection Sicherheitslücke, das heisst dass es nicht eine genügende Eingabevalidierung gibt und somit potenzielle Angreiffer mit einem einfachen prompt, wie ``"'; DROP TABLE *; --"``,  die Datenbank kompromieren können. 
Um dies zu verhindern, habe ich direkt den User gefiltert, anstatt einen String zu machen.

#### Rückblick
Die Umsetzung des Artefakts war teilweis erfolgreich, da ich eine SQL-Injection verhindern konnte, jedoch gibt es noch weitere Sicherheitslücken, wie Cross Site Scripting (XSS). XSS ist eine Angriffstechnik, bei der bösartiger Code in Webseiten eingefügt wird, der dann von anderen Nutzern dieser Seite ausgeführt wird. Dadurch können sensible Daten gestohlen werden.

Hier sind Codebeispiele für XSS:
```html
<script>alert('XSS')</script>. 
<a href="javascript:alert('XSS')">Klick</a> 
<img src="kitten.png" onload="alert('XSS')"/>
```
Um dies zu beheben muss die Eingabe escaped werden, siehe pics/3.png

Somit ist das Handlungsziel erreicht.

## Handlungsziel 3
Mechanismen für die Authentifizierung und Autorisierung umsetzen können.

#### Artefakt
```Csharp
if (user.SecretKey2FA != null)
    {
        string secretKey = user.SecretKey2FA;
        string userUniqueKey = user.Username + secretKey;
        TwoFactorAuthenticator authenticator = new TwoFactorAuthenticator();
        bool isAuthenticated = authenticator.ValidateTwoFactorPIN(userUniqueKey, request.UserKey);
        if (!isAuthenticated)
        {
            return Unauthorized("login failed");
        }
    }
```
```Csharp
[HttpPost]
[ProducesResponseType(200)]
[ProducesResponseType(404)]
public ActionResult<Auth2FADto> Enable2FA()
{
    var user = _context.Users.Find(_userService.GetUserId());
    if (user == null)
    {
        return NotFound(string.Format("User {0} not found", _userService.GetUsername()));
    }
    {
        var secretKey = Guid.NewGuid().ToString().Replace("-", "").Substring(0, 10);
        string userUniqueKey = user.Username + secretKey;
        string issuer = _configuration.GetSection("Jwt:Issuer").Value!;
        TwoFactorAuthenticator authenticator = new TwoFactorAuthenticator();
        SetupCode setupInfo = authenticator.GenerateSetupCode(issuer, user.Username, userUniqueKey, false, 3);

        user.SecretKey2FA = secretKey;
        _context.Update(user);
        _context.SaveChanges();

        Auth2FADto auth2FADto = new Auth2FADto();
        auth2FADto.QrCodeSetupImageUrl = setupInfo.QrCodeSetupImageUrl;

        return Ok(auth2FADto);
    }
}

```

#### Zielerreichung
Das Handlungsziel wird erreicht, indem 2-Faktor-Authentifizierung an einem Beispielprogramm angewandt wurde. Das Artefakt zeigt den Code für die Authentifizierung.

#### Erklärung
Im ersten Teil wird die 2FA während des Login-Prozesses durchgeführt. Falls ein Benutzer eine 2FA-Schlüssel (SecretKey2FA) besitzt, wird überprüft, ob der eingegebene Benutzer-Schlüssel (request.UserKey) korrekt ist. Bei Fehlschlag wird eine "Unauthorized" (401) Antwort zurückgegeben.

Im zweiten Teil wird die 2FA für einen Benutzer aktiviert. Hier wird ein neuer 2FA-Schlüssel generiert, ein Setup-Code erstellt. Der neu generierte Schlüssel wird dem Benutzer zugeordnet und in der Datenbank aktualisiert. Die erfolgreiche Aktivierung wird durch die Bereitstellung eines DTO-Objekts (Auth2FADto) mit dem QR-Code-Setup-Image-URL bestätigt.

#### Rückblick
Die Umsetzung des Handlungszieles ist teilweise erfolgreich, wir haben 2-Faktor-Authentifizierung angewandt, jedoch noch keine Autoriserung. Autorisierung gibt dem System die Möglichkeit nicht nur normale Benutzer sondern auch Admins hinzuzufügen.

Um autorisierung zu implementieren, werden wir es in JWT token einschieben:

```Csharp
private string CreateToken(User user)
        {
            string issuer = _configuration.GetSection("Jwt:Issuer").Value!;
            string audience = _configuration.GetSection("Jwt:Audience").Value!;

            List<Claim> claims = new List<Claim> {
                    new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
                    new Claim(JwtRegisteredClaimNames.NameId, user.Id.ToString()),
                    new Claim(JwtRegisteredClaimNames.UniqueName, user.Username),
                    new Claim(ClaimTypes.Role,  (user.IsAdmin ? "admin" : "user"))
            };

            string base64Key = _configuration.GetSection("Jwt:Key").Value!;
            SymmetricSecurityKey securityKey = new SymmetricSecurityKey(Convert.FromBase64String(base64Key));

            SigningCredentials credentials = new SigningCredentials(
                    securityKey,
                    SecurityAlgorithms.HmacSha512Signature);

            JwtSecurityToken token = new JwtSecurityToken(
                issuer: issuer,
                audience: audience,
                claims: claims,
                notBefore: DateTime.Now,
                expires: DateTime.Now.AddDays(1),
                signingCredentials: credentials
             );

            return new JwtSecurityTokenHandler().WriteToken(token);
        }
```
Hier wird die Methode CreateToken verwendet, um ein JWT-Token zu generieren, das verschiedene claims enthält, darunter auch die Rolle des Benutzers. Danach wird der Token erstellt und signiert.

Somit haben wir das Handlungsziel erreicht.

## Handlungsziel 4
Sicherheitsrelevante Aspekte bei Entwurf, Implementierung und Inbetriebnahme berücksichtigen.
    
#### Artefakt
```Csharp
private string validateNewPasswort(string newPassword)
{
    // Check small letter.
    string patternSmall = "[a-zäöü]";
    Regex regexSmall = new Regex(patternSmall);
    bool hasSmallLetter = regexSmall.Match(newPassword).Success;

    string patternCapital = "[A-ZÄÖÜ]";
    Regex regexCapital = new Regex(patternCapital);
    bool hasCapitalLetter = regexCapital.Match(newPassword).Success;

    string patternNumber = "[0-9]";
    Regex regexNumber = new Regex(patternNumber);
    bool hasNumber = regexNumber.Match(newPassword).Success;

    List<string> result = new List<string>();
    if (!hasSmallLetter)
    {
        result.Add("keinen Kleinbuchstaben");
    }
    if (!hasCapitalLetter)
    {
        result.Add("keinen Grossbuchstaben");
    }
    if (!hasNumber)
    {
        result.Add("keine Zahl");
    }

    if (result.Count > 0)
    {
        return "Das Passwort beinhaltet " + string.Join(", ", result);
    }
    return "";
}

```
#### Zielerreichung
Das Ziel wird erreicht, indem der Human Factor einbezogen wird ins Programm. Das Artefakt zeigt die Implementierung.

#### Erklärung
Das gezeigte Artefakt ist eine Methode validateNewPassword, die sicherheitsrelevante Überprüfungen für ein neues Passwort durchführt. Es prüft, ob das Passwort bestimmte Kriterien erfüllt, die für die Sicherheit von Passwörtern relevant sind. Konkret werden folgende Aspekte überprüft:

Kleinschreibung: Es wird geprüft, ob das Passwort mindestens einen Kleinbuchstaben enthält.
Grosschreibung: Es wird geprüft, ob das Passwort mindestens einen Grossbuchstaben enthält.
Zahl: Es wird geprüft, ob das Passwort mindestens eine Zahl enthält.

Dies trägt dazu bei, dass die Widerstandsfähigkeit von Passwörtern gegenüber Brute-Force- oder Dictionary-Angriffen erhöht und die allgemeine Sicherheit des Systems gestärkt wird.

#### Rückblick
Das Handlungsziel wird teilweise erreicht indem die Inbetriebnahme gezeigt wird und erklärt wurde für Human Factor, jedoch nicht Entwurf und Implementierung.

**Entwurf**
Beim Entwurf des Programmes muss berücksichtigt werde, dass man die Tests, Fachwissen, Reviews, Zeit, etc. nach Risiko priorisiert 
(Risiko = Schadenspotential * Eintrittswahrscheinlichkeit)
Es sollte auch Ein- und Ausgabevalidierung haben, sowie Authentifizierung und Autorisierung. 
Es ist wichtig zu berücksichtigen, dass bei der Verwendung von externen Bibliotheken mehr Sicherheitslücken geöffnet werden.


**Implementierung**
Security sollte konsistent implementiert werden:
1. Sicherheit sollte konsistent umgesetzt werden durch klare Abläufe, einheitliche Benennung und Vermeidung von Komplexität.

2. Security sollte zentral implementiert werden, an einer Stelle, von der aus sie angewendet und geprüft werden kann.

3. Es ist wichtig, Annahmen zu überprüfen: Der Code sollte alle Annahmen bezüglich Werte und Daten prüfen.

4. Um Sicherheit zu fördern, sollte die Arbeit für Entwickler minimiert werden, da dadurch die Anwendung von Sicherheitspraktiken erleichtert wird.

Mit Entwurf und Implementierung nun erklärt, ist das Handlungsziel erfüllt.

## Handlungsziel 5
Informationen für Auditing und Logging generieren. Auswertungen und Alarme definieren und implementieren.

#### Artefakt
1. Wählen Sie ein Artefakt, welches Sie selbst erstellt haben und anhand dem Sie zeigen können, dass Sie das Handlungsziel erreicht haben.

#### Zielerreichung
2. Weisen Sie nach, wie Sie das Handlungsziel erreicht haben. Verweisen Sie dabei auf das von Ihnen erstellte Artefakt. Das Artefakt muss im ePortfolio sichtbar oder verlinkt sein.

#### Erklärung
3. Erklären Sie das Artefakt in wenigen Sätzen. Sollte das Artefakt mehrere Handlungsziele beinhalten dürfen Sie die Erklärung auch zusammenfassen.

#### Rückblick
4. Beurteilen Sie die Umsetzung Ihres Artefakts im Hinblick auf das Handlungsziel kritisch. Sollten gewisse Aspekte des Handlungsziels fehlen, haben Sie die Möglichkeit, in diesem Teil darauf einzugehen.




Selbsteinschätzung des Erreichungsgrades der Kompetenz des Moduls
Geben Sie eine Selbsteinschätzung zu der Kompetenz in diesem Modul ab. Schätzen Sie selbst ein, inwiefern Sie die Kompetenz dieses Moduls erreicht haben und inwiefern nicht. Es geht in diesem Abschnitt nicht darum, auf die einzelnen Handlungsziele einzugehen. Das haben Sie bereits gemacht. Begründen Sie ihre Aussagen. 
