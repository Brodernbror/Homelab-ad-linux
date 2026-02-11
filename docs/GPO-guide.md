# GPO-guide – Active Directory Homelab

Denne guiden tar deg gjennom tre praktiske Group Policy Objects (GPO-er) i din `lab.local`-domene.  
Du trenger: tilgang til **DC01** (Windows Server 2022) og **Group Policy Management Console (GPMC)**.

---

## Forberedelse – Åpne Group Policy Management

1. Logg inn på DC01 som `Administrator`
2. Åpne **Server Manager** → **Tools** → **Group Policy Management**
3. Naviger til: `Forest: lab.local` → `Domains` → `lab.local`

Du vil se en **Default Domain Policy** her. Vi lager egne GPO-er i stedet for å redigere denne – det er best practice.

---

## GPO 1 – Passordpolicy og kontolåsing

Dette styrer krav til passord og hva som skjer om noen taster feil passord for mange ganger.

### Opprett GPO-en
1. Høyreklikk på `lab.local` → **Create a GPO in this domain, and Link it here...**
2. Gi den navnet: `Security - Password and Lockout Policy`
3. Høyreklikk den nye GPO-en → **Edit**

### Passordinnstillinger
Naviger til:
```
Computer Configuration
  └── Policies
        └── Windows Settings
              └── Security Settings
                    └── Account Policies
                          └── Password Policy
```

| Innstilling | Anbefalt verdi | Beskrivelse |
|---|---|---|
| Minimum password length | 12 | Minimum antall tegn |
| Password must meet complexity requirements | Enabled | Krever store/små bokstaver, tall og tegn |
| Maximum password age | 90 days | Tvinger passordbytte |
| Minimum password age | 1 day | Hindrer umiddelbar gjenbruk |
| Enforce password history | 10 passwords | Kan ikke gjenbruke de 10 siste |

### Kontolåsing
Naviger til:
```
Account Policies
  └── Account Lockout Policy
```

| Innstilling | Anbefalt verdi | Beskrivelse |
|---|---|---|
| Account lockout threshold | 5 invalid attempts | Låses etter 5 feil forsøk |
| Account lockout duration | 15 minutes | Låst i 15 min før auto-opplåsing |
| Reset account lockout counter after | 15 minutes | Tilbakestill telleretter 15 min |

> **Tips:** I et skikkelig produksjonsmiljø setter man gjerne terskelen lavere (3 forsøk), men 5 er greit for et lab-miljø der du tester mye.

### Verifisér at GPO-en gjelder
Kjør dette på Ubuntu-klienten eller DC01 for å tvinge oppdatering:
```bash
# På DC01 (PowerShell)
gpupdate /force

# Sjekk at policyen er aktiv
gpresult /r
```

---

## GPO 2 – Login-banner (sikkerhetsmeldinger)

En login-banner er en melding brukere ser før de logger inn. Den brukes i alle seriøse bedrifter og er også et juridisk krav i mange sektorer.

### Opprett GPO-en
1. Høyreklikk på `lab.local` → **Create a GPO in this domain, and Link it here...**
2. Gi den navnet: `Security - Login Banner`
3. Høyreklikk → **Edit**

### Konfigurer banneret
Naviger til:
```
Computer Configuration
  └── Policies
        └── Windows Settings
              └── Security Settings
                    └── Local Policies
                          └── Security Options
```

Finn disse to innstillingene og aktiver dem:

**Interactive logon: Message title for users attempting to log on**
```
LAB.LOCAL – Authorized Access Only
```

**Interactive logon: Message text for users attempting to log on**
```
This system is part of the lab.local domain and is for authorized 
users only. All activity may be monitored and logged. Unauthorized 
access is strictly prohibited.
```

> **Merk:** Banneret vises på Windows-klienter. Ubuntu-klienter håndterer dette via `/etc/issue` og `/etc/motd` – se seksjonen under.

### Login-banner på Ubuntu (via SSH/terminal)
Selv om GPO ikke direkte styrer Ubuntu-terminalen, kan du sette opp en tilsvarende melding manuelt:

```bash
# Rediger /etc/issue (vises FØR innlogging)
sudo nano /etc/issue

# Legg inn for eksempel:
# ############################################
# #   LAB.LOCAL – Authorized Access Only     #
# #   All activity is monitored and logged.  #
# ############################################

# Rediger /etc/motd (vises ETTER innlogging – "Message of the Day")
sudo nano /etc/motd
```

---

## GPO 3 – Mappe nettverksstasjoner til brukere

Dette gir brukere automatisk tilgang til en nettverksdisk (f.eks. `H:`) når de logger inn. Krever at du har en SMB-share på DC01 eller en annen server.

### Steg 1: Opprett en filshare på DC01

Åpne **File Explorer** på DC01:
1. Opprett mappen `C:\Shares\UserData`
2. Høyreklikk mappen → **Properties** → **Sharing** → **Advanced Sharing**
3. Huk av **Share this folder**
4. Gi sharenavn: `UserData`
5. Klikk **Permissions** → legg til **Domain Users** med **Read/Change**-tilgang

UNC-stien til sharingen blir: `\\DC01\UserData`

### Steg 2: Opprett GPO-en
1. Høyreklikk på `lab.local` → **Create a GPO in this domain, and Link it here...**
2. Gi den navnet: `User - Map Network Drives`
3. Høyreklikk → **Edit**

### Steg 3: Konfigurer drive mapping
Naviger til:
```
User Configuration
  └── Preferences
        └── Windows Settings
              └── Drive Maps
```

1. Høyreklikk i høyre panel → **New** → **Mapped Drive**
2. Fyll inn:

| Felt | Verdi |
|---|---|
| Action | Create |
| Location | `\\DC01\UserData` |
| Drive Letter | H: |
| Label as | UserData |
| Reconnect | ✅ Huk av |

3. Klikk **OK** og lukk

> **Tips:** Under fanen **Common** kan du huke av **Item-level targeting** for å bare gi spesifikke grupper/brukere denne stasjonen – nyttig for å øve på mer avansert GPO-styring.

### Steg 4: Test det fra Windows-klient
Logg inn med en domenebruker på en Windows-klient. `H:`-stasjonen skal dukke opp automatisk i File Explorer.

---

## Verifisere at GPO-er virker

### Se hvilke GPO-er som gjelder en maskin/bruker
```powershell
# Kjør på DC01 eller en Windows-klient
gpresult /h C:\gpresult.html
# Åpne gpresult.html i nettleseren for full oversikt

# Rask tekstversjon
gpresult /r
```

### Tvinge oppdatering
```powershell
# Lokalt
gpupdate /force

# Mot en annen maskin (fra DC01)
Invoke-GPUpdate -Computer "UBUNTU-CLIENT" -Force
```

### Feilsøke med Event Viewer
Åpne **Event Viewer** → `Applications and Services Logs` → `Microsoft` → `Windows` → `GroupPolicy` → `Operational`  
Her ser du om GPO-er feiler og hvorfor.

---

## Hva nå?

Når du har fått disse tre GPO-ene på plass, er naturlige neste steg:

- **Organizational Units (OU-er)** – organisere maskiner og brukere i grupper, og knytte GPO-er til spesifikke OU-er i stedet for hele domenet
- **Windows-klient** – join en Windows 10/11 VM til domenet og test at GPO-ene treffer
- **Sentralisert logging** – sett opp Wazuh eller Syslog for å fange opp hendelser fra domenet
