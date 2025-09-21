# AI Commenter & Dish API

Tämä on Express.js sovellus TypeScriptillä, joka tarjoaa AI-pohjaisia kommenttitoimintoja sekä ruokatietokannan hallintaa automaattisella kuvan generoinnilla.

## 📋 Ominaisuudet

- **AI-kommenttisysteemi**: Käyttää OpenAI:ta koodikommenttien generointiin
- **Ruokatietokanta**: CRUD-operaatiot ruokalistoille
- **Automaattinen kuvan generointi**: DALL-E 2 generoi kuvia ruoille
- **Kuvan käsittely**: Automaattinen thumbnail-kuvien luonti
- **Validointi**: Express-validator kaikkiin pyyntöihin
- **Virheenkäsittely**: Kattava virheenkäsittely CustomError-luokalla

## 🛠️ Teknologiat

- **Backend**: Node.js + Express.js
- **Kieli**: TypeScript
- **AI**: OpenAI API (GPT-5-nano, DALL-E 2)
- **Kuvan käsittely**: Sharp
- **Validointi**: Express-validator
- **Muut**: Morgan, Helmet, CORS

## 📁 Projektin rakenne

```
src/
├── app.ts                 # Express-sovelluksen konfiguraatio
├── index.ts              # Palvelimen käynnistys
├── middlewares.ts        # Kaikki middlewaret
├── api/
│   ├── index.ts
│   ├── controllers/
│   │   └── commentController.ts
│   └── routes/
│       └── commentRoute.ts
├── classes/
│   └── CustomError.ts    # Mukautettu virheluokka
├── controllers/
│   └── dishController.ts # Ruoka-CRUD kontrollerit
├── lib/
│   └── fetchData.ts      # HTTP-pyynnöt
├── routes/
│   └── dishRoute.ts      # Ruoka-reitit
└── types/
    ├── DBTypes.ts        # Tietokantatiivit
    └── MessageTypes.ts   # Viestilaitteet
```

## ⚙️ Asennus

1. **Kloonaa repositorio:**
```bash
git clone <repository-url>
cd AI-commenter-starter-main
```

2. **Asenna riippuvuudet:**
```bash
npm install
```

3. **Luo ympäristömuuttujat:**
Luo `.env` tiedosto projektin juureen:
```env
NODE_ENV=development
PORT=3000
OPENAI_API_URL=https://media2.edu.metropolia.fi/openapi
OPENAI_CHAT_MODEL=gpt-5-nano
OPENAI_API_KEY=your_openai_api_key_here
```

4. **Luo uploads-kansio:**
```bash
mkdir uploads
```

5. **Käynnistä sovellus:**
```bash
# Kehitysmoodi
npm run dev

# Tuotantomoodi
npm run build
npm start
```

Sovellus käynnistyy osoitteessa: `http://localhost:3000`

## 🔌 API Endpoints

### 📝 Kommentit (AI-assistentti)

#### POST `/api/v1/comments`
Generoi AI-kommentteja koodille.

**Request Body:**
```json
{
  "text": "// Create a function that calculates the area of a circle"
}
```

**Response:**
```json
{
  "response": "function calculateCircleArea(radius) {\n  return Math.PI * radius * radius;\n}"
}
```

### 🍽️ Ruoat (Dishes)

#### GET `/api/v1/dishes`
Hae kaikki ruoat.

**Response:**
```json
[
  {
    "dish_id": 1,
    "dish_name": "Grilled Salmon",
    "description": "Fresh Atlantic salmon grilled to perfection",
    "filename": "Grilled_Salmon.png",
    "dish_type": "main course"
  }
]
```

#### GET `/api/v1/dishes/:id`
Hae yksittäinen ruoka ID:n mukaan.

**Response:**
```json
{
  "dish_id": 1,
  "dish_name": "Grilled Salmon",
  "description": "Fresh Atlantic salmon grilled to perfection",
  "filename": "Grilled_Salmon.png",
  "dish_type": "main course"
}
```

#### POST `/api/v1/dishes`
Luo uusi ruoka (generoi automaattisesti AI-kuva).

**Request Body:**
```json
{
  "dish_name": "Grilled Salmon",
  "description": "Fresh Atlantic salmon grilled to perfection with herbs and lemon",
  "dish_type": "main course"
}
```

**Response:**
```json
{
  "message": "Dish created successfully"
}
```

**Middleware-ketju:**
1. `validate` - Validoi syötteen
2. `getAiImage` - Generoi kuva DALL-E 2:lla
3. `saveAiImage` - Tallentaa kuva uploads-kansioon
4. `makeThumbnail` - Luo 160x160 thumbnail
5. `createDish` - Tallentaa ruoan tietokantaan

#### PUT `/api/v1/dishes/:id`
Päivitä ruokaa.

**Request Body:**
```json
{
  "dish_name": "Updated Grilled Salmon",
  "description": "Updated description"
}
```

#### DELETE `/api/v1/dishes/:id`
Poista ruoka.

**Response:**
```json
{
  "message": "Dish deleted successfully"
}
```

## 🧪 Testaus Postmanilla

### 1. AI-kommenttien testaus
- **Method**: POST
- **URL**: `http://localhost:3000/api/v1/comments`
- **Headers**: `Content-Type: application/json`
- **Body**:
```json
{
  "text": "// Create a function that validates email addresses"
}
```

### 2. Ruoan lisäys (AI-kuvan kanssa)
- **Method**: POST
- **URL**: `http://localhost:3000/api/v1/dishes`
- **Headers**: `Content-Type: application/json`
- **Body**:
```json
{
  "dish_name": "Margherita Pizza",
  "description": "Classic Italian pizza with fresh mozzarella, tomatoes and basil",
  "dish_type": "main course"
}
```

### 3. Kaikkien ruokien haku
- **Method**: GET
- **URL**: `http://localhost:3000/api/v1/dishes`

### 4. Yksittäisen ruoan haku
- **Method**: GET
- **URL**: `http://localhost:3000/api/v1/dishes/1`

### 5. Ruoan päivittäminen
- **Method**: PUT
- **URL**: `http://localhost:3000/api/v1/dishes/1`
- **Headers**: `Content-Type: application/json`
- **Body**:
```json
{
  "dish_name": "Updated Pizza Name"
}
```

### 6. Ruoan poistaminen
- **Method**: DELETE
- **URL**: `http://localhost:3000/api/v1/dishes/1`

## 📂 Kuvien käsittely

Sovellus tallentaa kuvat `./uploads/` kansioon:
- **Alkuperäinen kuva**: `dish_name.png`
- **Thumbnail**: `thumb_dish_name.png` (160x160px)
- **Oletuskuva**: `default.png` (jos AI-kuvan generointi epäonnistuu)

## 🔧 Middlewaret

### `validate`
Validoi express-validator säännöt ja palauttaa virheet.

### `getAiImage`
Generoi AI-kuva DALL-E 2:lla ruoan tietojen perusteella.

### `saveAiImage`
Lataa generoidun kuvan palvelimelle ja tallentaa sen uploads-kansioon.

### `makeThumbnail`
Luo 160x160 pikselin thumbnail-kuva Sharp-kirjastolla.

### `notFound`
Käsittelee 404-virheet.

### `errorHandler`
Yleinen virheenkäsittelijä CustomError-objekteille.

## 📊 Virheenkäsittely

Sovellus käyttää mukautettua `CustomError`-luokkaa:

```typescript
class CustomError extends Error {
  status = 400;
  constructor(message: string, status: number) {
    super(message);
    this.status = status;
  }
}
```

**Esimerkkivirheet:**
- `400`: Validointivirhe
- `404`: Ei löydy
- `500`: Palvelinvirhe

## 🚀 Kehittäminen

### Käytettävät scriptit:
```bash
npm run dev          # Kehitysmoodi (nodemon)
npm run build        # Käännä TypeScript
npm run start        # Käynnistä tuotannossa
npm run lint         # ESLint tarkistus
npm run test         # Jest testit
```

### Tulevat ominaisuudet:
- [ ] Tietokanta integraatio (MySQL/PostgreSQL)
- [ ] Käyttäjähallinta ja autentikointi
- [ ] Kuvan upload file-systemin sijaan
- [ ] Rate limiting AI-kutsuille
- [ ] Kuvien optimointi eri kokoisille näytöille

## 📄 Lisenssi

MIT License

## 👥 Kehittäjät

- **Azzni96** - Pääkehittäjä

---

**Huom:** Muista lisätä `OPENAI_API_KEY` ympäristömuuttujiin ennen sovelluksen käyttöä!
