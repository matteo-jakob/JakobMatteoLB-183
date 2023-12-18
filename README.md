# Sicherheit in der Softwareentwicklung
LB-183 von Matteo Jakob

Dieses ePortfolio gibt einen Einblick in meine Fortschritte und Erfahrungen in Applikationssicherheit. Es beinhaltet eine detaillierte Analyse aktueller Bedrohungen, die Erkennung und Behebung von Sicherheitslücken, die Implementierung von Mechanismen für Authentifizierung und Autorisierung, die Berücksichtigung sicherheitsrelevanter Aspekte im Entwurf und in der Implementierung sowie die Generierung von Informationen für Auditing und Logging.


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
```csharp
public IActionResult Login(string username, string password)
{
    // Logger
    _logger.LogInformation($"Login attempt for user '{username}'");

    // Auditing
    AuditLog(username, "Login attempt");

    // Auswertung
    if (c(username))
    {
        // Alarm
        RaiseSecurityAlarm(username);
        return Unauthorized("Suspicious login attempt detected");
    }

    // Logger
    _logger.LogInformation($"Login successful for user '{username}'");
    return Ok(CreateToken(username));
}

private void AuditLog(string username, string action)
{
    _context.AuditLogs.Add(new AuditLog { Username = username, Action = action, Timestamp = DateTime.UtcNow });
    _context.SaveChanges();
}

```

#### Zielerreichung
Das Ziel wird erreicht, indem ich die Implementierung des Loggers, Auditing, Auswertung und Alarm darstelle.

#### Erklärung
Die Zeile _logger.LogInformation($"Login attempt for user '{username}'"); zeigt, wie man einen Log erstellt und anzeigt, dass ein Anmeldeversuch für einen bestimmten Benutzer erfolgt ist.

Die Methode AuditLog wird aufgerufen, um eine Audit-Protokollierung durchzuführen. In diesem Fall wird ein Eintrag in einer Audit-Log-Tabelle oder einem Audit-Log-Objekt erstellt, um den Benutzernamen, die Aktion ("Login attempt") und den Zeitstempel des Anmeldeversuchs zu erfassen. 

Bei Auswertung und Alarm wird überprüft, ob verdächtige Aktivitäten beim Anmeldeversuch vorliegen. Falls ja, wird ein Sicherheitsalarm ausgelöst. Die Methode Alarm koennte zum Beispiel folgenden Code enthalten: 
```
_logger.LogWarning($"Security alarm triggered for suspicious activity by user '{username}'");
```
Also der Alarm ist ein weiterer Log output.
Bei der Auswertung könnte zum Beispiel geprüft werden, wie viele Male der User sich an einem Tag eingeloggt hat.

#### Rückblick
Die Logging-Funktionalität wurde durch den Einsatz des Loggers realisiert, das Auditing durch die Methode AuditLog, und die Auswertung und Alarm durch die entsprechenden Codeabschnitte im Login-Controller.

Das Handlungsziel wurde erreicht, da ich die Implementierung des Loggers, Auditings, Auswertung und Alarm gezeigt und definiert habe. 

### Selbsteinschätzung
Ich schätze, dass ich die Kompetenz in diesem Modul gut erreicht habe. Die vorgestellten Artefakte und Erklärungen zeigen, dass ich verschiedenste Sicherheitskonzepte verstanden habe. Die Rückblicke geben Einsicht in die Beurteilung der Umsetzung und zeigen auf, wo noch Verbesserungsmöglichkeiten oder weitere Schritte vorhanden sind, wie im Rückblick zu Handlungsziel 2 und 4.

Insgesamt denke ich, dass ich die Sicherheitsmechanismen im Entwurf und in der Implementierung gut umsetzen kann, aber es gibt immer Raum für weiteres Lernen und Vertiefen von Sicherheitskonzepten und Best Practices, da es auch stets ein entwickelndes Gebiet ist und immer neue Sicherheitslücken auftauchen. Es ist wichtig sich stets mit den neusten Sicherheitsmassnahmen und Bedrohungen sich zu informieren.