# SuriRent deployen op Coolify

Deze repository bevat een Docker Compose-deploy voor de complete applicatie:

- `frontend`: React-site en Nginx reverse proxy voor `/api/*`
- `app`: FastAPI-backend
- `mongo`: MongoDB 7 met persistente Docker-volume

## 1. Resource aanmaken

1. Maak in Coolify een **Docker Compose**-resource aan en koppel deze repository.
2. Gebruik de repository-root als Base Directory; Coolify vindt daar `docker-compose.yml`.
3. Voeg de configuratievariabelen uit stap 2 toe in **Environment Variables**.
4. Configureer bij de service `frontend` een domein op containerpoort `80`. De backend en MongoDB hoeven niet publiek bereikbaar te zijn.
5. Deploy de resource. Coolify bouwt de images en start MongoDB, backend en frontend in de juiste volgorde.

De frontend stuurt `/api/*` intern door naar de backend. Eén domein werkt zonder extra frontendinstellingen. Voor een aparte marketing- en app-host kan hetzelfde frontend-domein aan beide hosts worden gekoppeld; stel dan de optionele frontendvariabelen hieronder in.

## 2. Vereiste Coolify-variabelen

Maak sterke, unieke waarden en bewaar ze als secrets in Coolify. Genereer bijvoorbeeld hexwaarden met `openssl rand -hex 32`; hex is ook geschikt voor de MongoDB-wachtwoorden in de connection string.

| Variabele | Voorbeeld / gebruik |
|---|---|
| `MONGO_ROOT_USERNAME` | MongoDB-beheeraccount, bijvoorbeeld `surirent` |
| `MONGO_ROOT_PASSWORD` | Uniek wachtwoord; gebruik een hexwaarde |
| `DB_NAME` | `vastgoed_kiosk` |
| `JWT_SECRET` | Unieke willekeurige hexwaarde |
| `SETTINGS_ENCRYPTION_KEY` | Eigen willekeurige hexwaarde |
| `PDF_AES_PASSWORD` | Eigen willekeurig wachtwoord |
| `ADMIN_EMAIL` | E-mailadres voor het standaardbeheeraccount |
| `ADMIN_PASSWORD` | Sterk wachtwoord voor dat beheeraccount |
| `SUPERADMIN_EMAIL` | E-mailadres voor het superadmin-account |
| `SUPERADMIN_PASSWORD` | Sterk wachtwoord voor dat superadmin-account |
| `APP_PUBLIC_URL` | Publieke URL van de app, bijvoorbeeld `https://app.example.com` |
| `CORS_ORIGINS` | Publieke origins, komma-gescheiden, inclusief schema; bijvoorbeeld `https://app.example.com,https://example.com` |

De backend initialiseert deze beheeraccounts. Bij iedere start wordt het wachtwoord van `ADMIN_EMAIL` bijgewerkt naar `ADMIN_PASSWORD`; houd deze waarden stabiel of verander het wachtwoord via de app en pas daarna de Coolify-secret aan. `SUPERADMIN_PASSWORD` wordt alleen gebruikt wanneer het superadmin-account voor het eerst wordt aangemaakt. Stel de database- en encryptiesecrets vóór de eerste deploy in en wijzig ze niet achteraf zonder een geplande migratie.

De Compose-configuratie zet het opnieuw instellen van de demo-PIN en de periodieke demo-reset uit. Stel optionele integraties (zoals e-mail, Stripe, push en AI) alleen in wanneer die nodig zijn; hun bestaande omgevingsvariabelen worden door de backend gelezen.

## 3. Aparte marketing- en app-host (optioneel)

Dezelfde frontendservice kan beide domeinen bedienen. Stel hiervoor vóór het bouwen deze variabelen in; wijzigingen hieraan vereisen een nieuwe image-build:

| Variabele | Waarde |
|---|---|
| `REACT_APP_BACKEND_URL` | App-URL, bijvoorbeeld `https://app.example.com` |
| `REACT_APP_APP_URL` | Dezelfde app-URL |
| `REACT_APP_MARKETING_HOST` | Marketing-host zonder schema, bijvoorbeeld `example.com` |

Koppel zowel het marketingdomein als het appdomein aan `frontend` in Coolify, configureer DNS naar de Coolify-server en gebruik beide origins in `CORS_ORIGINS`. Zonder deze variabelen werkt de site in hybrid mode op één domein.

## 4. Data en controle

MongoDB-data blijft staan in het Compose-volume `mongo_data` wanneer containers opnieuw worden gebouwd. Configureer daarnaast periodieke databaseback-ups naar opslag buiten deze Coolify-resource; een Docker-volume is geen back-up.

Kom je van een bestaande CloudPanel-installatie, maak dan eerst een `mongodump` en herstel die in de nieuwe MongoDB voordat je gebruikers naar Coolify omschakelt. De nieuwe `mongo_data`-volume bevat aanvankelijk een lege database.

Controleer na de deploy `https://<app-domein>/api/health`; verwacht `{"ok":true,"service":"vastgoed-kiosk-api"}`. Coolify gebruikt dezelfde endpoint voor de backend-healthcheck.
