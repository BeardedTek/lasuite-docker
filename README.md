# LaSuite Numerique Docker Setup

This is my [LaSuite Numerique](https://github.com/suitenumerique) Docker Setup with traefik as a reverse proxy.

I intend to get all of their apps working in docker compose eventually.

#### Assumptions

You should have your domains already configured and pointed to the proper server you are running traefik on.
You should be running traefik on the same host with docker labels setup
You should have a fully deployed already working copy of traefik running.

This guide DOES NOT walk you through that.

## What's Here Now

### La Suite Meet

A Zoom / Google Meet / Teams Replacement

#### Setup

I followed their [Instalation with docker compose guide](https://github.com/suitenumerique/meet/blob/main/docs/installation/compose.md) and modified the structure so I can have all the services under one giant compose file eventually.

#### Clone this repository

```bash
git clone https://github.com/beardedtek/lasuite-docker.git
cd lasuite-docker
```

#### Edit `.env`

Fill in `BASE_DOMAIN`

```env
BASE_DOMAIN=example.com
MEET_HOST=meet.${BASE_DOMAIN}
KEYCLOAK_HOST=id.${BASE_DOMAIN}
LIVEKIT_HOST=livekit.${BASE_DOMAIN}
BACKEND_INTERNAL_HOST=backend
FRONTEND_INTERNAL_HOST=frontend
LIVEKIT_INTERNAL_HOST=livekit
REALM_NAME=meet
```

#### Edit `env.d/kc_postgresql`

- Replace `<POSTGRES_PASSWORD>` with your secure password.

#### Edit `env.d/keycloak`

- Replace `<YOUR_PASSWORD>` with your secure password.
- Replace `id.example.com` with `id.yourdomain.com`

#### Bring up Keycloak

```bash
docker compose -f keycloak up -d
```

Now follow this procedure taken from [https://github.com/suitenumerique/meet/blob/main/docs/examples/compose/keycloak/README.md](https://github.com/suitenumerique/meet/blob/main/docs/examples/compose/keycloak/README.md)
This will get you your CLIENT SECRET from Keycloak you will need later.  Note it down.

```
# Creating an OIDC Client for Meet Application

### Step 1: Create a New Realm

1. Log in to the Keycloak administration console.
2. Navigate to the realm tab and click on the "Create realm" button.
3. Enter the name of the realm  - `meet`.
4. Click "Create".

#### Step 2: Create a New Client

1. Navigate to the "Clients" tab.
2. Click on the "Create client" button.
3. Enter the client ID - e.g. `meet`.
4. Enable "Client authentication" option.
6. Set the "Valid redirect URIs" to the URL of your meet application suffixed with `/*` - e.g., "https://meet.example.com/*".
1. Set the "Web Origins" to the URL of your meet application - e.g. `https://meet.example.com`.
1. Click "Save".

#### Step 3: Get Client Credentials

1. Go to the "Credentials" tab.
2. Copy the client ID (`meet` in this example) and the client secret.

#### Step 4: Enable User Registration

If you would like to enable user registration follow these steps, otherwise skip to step 6.

1. Go to the "Realm Settings" tab.
2. Enable "User Registration"
3. Enable "Forgot Password"
4. Enable "Verify Password"

#### Step 5: Enable Email

1. Click on the Admin dropdown and select manage account
2. Set your eamil address
3. Click Save
4. CLick "Back to Security Admin Console"
5. Go to the "Email" tab.
6. Fill out the email connection information
7. Click "Save"
8. Click "Test connection"

#### Step 6: Create Users

If you did not enable registration, you can create users manually.

1. Click Users on the sidebar
2. Click "Add user"
3. Fill out the form with at least email and click create
4. Go to the "Credentials tab.
5. Click "Set password"
6. Fill out password and confirmation
7. Select temporary if you wish the user to change their password on next login.
8. Click Save
```

#### Edit `env.d/meet/common`

- Replace `<YOUR_DJANGO_SECRET>` with a django secret generated from [https://djecrety.ir/](https://djecrety.ir/)
  - NOTE: keep generating a secret until there is no `$` in it.  This will save you loads of time figuring out how to properly escape it.
- Replace `<YOUR_BRAND>` With the name you want displayed in your emails.
- Replace `<CLIENT_SECRET_FROM_KEYCLOAK_MEET_REALM>` with your Client Secret you created above.
- Replace `<LIVEKIT_API_SECRET>` with a jwt secret.  You can generate one at [https://jwtsecretkeygenerator.com/](https://jwtsecretkeygenerator.com/)


#### Edit `env.d/meet/postgresql`

- Replace `<YOUR_POSTGRES_PASSWORD>` with your secure password

#### Edit `config/meet/livekit-server.yaml`

- Replace `<YOUR_SECRET>` with your `<LIVEKIT_API_SECRET>` from `env.d/meet/common`


### Bring up the whole project

```bash
docker compose up -d
```

#### Setup the Django Backend

```bash
docker compose run --rm backend python manage.py migrate
docker compose run --rm backend python manage.py createsuperuser --email <admin email> --password <admin password>
```

### Test It Out!

If everything worked correctly, it should now work
