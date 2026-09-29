# Museboard

Museboard is a web-based fashion mood-boarding and outfit-planning application. It lets you collect style inspiration into visual boards, catalogue your own wardrobe, and get coordinated outfit suggestions generated from a colour harmony engine and a trained ML garment classifier.

---

## 1. What you need before you start

- **Docker Desktop** installed and running (Windows/Mac/Linux)
- That's it — MySQL, phpMyAdmin, the PHP app, and the ML service all run inside containers, so you don't need PHP, MySQL, or Python installed locally.
- Optional: a Kaggle account + API token (`kaggle.json`) only if you plan to retrain the ML model yourself. Not needed just to run the app.

---

## 2. Setting up the environment

1. **Clone the project** (`git clone https://github.com/YOUR-USERNAME/museboard.git`) or unzip it, so you have this folder structure:
   ```
   Museboard/
   ├── docker-compose.yml
   ├── dockerfile
   ├── museboard_src/        (PHP app)
   └── ml/                   (Python ML service)
   ```

2. **Open a terminal in the project's root folder** (the one with `docker-compose.yml` in it).

3. **Build and start everything** with one command:
   ```bash
   docker compose up -d --build
   ```
   This starts four containers:
   - `web` (container: museboard_web) — PHP + Apache, the main app
   - `db` (container: museboard_db) — MySQL 8.0 database
   - `phpmyadmin` (container: museboard_phpmyadmin) — database admin UI
   - `ml` (container: museboard_ml) — Flask ML classification service

4. **Wait for MySQL to finish initialising** the first time (check with `docker compose logs -f db` — it's ready when you see `ready for connections`). The schema is created automatically from `museboard_src/config/init.sql`.

5. **(Optional) Add your Unsplash API key.** Unsplash search needs a free key from [unsplash.com/developers](https://unsplash.com/developers). Copy `.env.example` to `.env`, paste your key after `UNSPLASH_ACCESS_KEY=`, then run `docker compose up -d --build`. Without a key the rest of the app still works; only Unsplash search is disabled.

---

## 3. Opening the platform

Once the containers are running, open these in your browser:

- **Museboard app** — http://localhost:8080
- **phpMyAdmin** (view/edit the database directly) — http://localhost:8081 — login with username `root`, password `root`
- **ML service health check** — http://localhost:5001/health

To stop everything: `docker compose down`
To stop and wipe the database too: `docker compose down -v`

---

## 4. Logging in

Museboard ships with two test accounts:

**Admin account**
- Email: `jt5530936@gmail.com`
- Password: `jeeson`
- Can do everything a normal user can, plus manage users from `/admin.php` (promote/demote admins, suspend accounts).

**Regular user account**
- Email: `erin127@gmail.com`
- Password: `erin127`
- Standard access: boards, wardrobe, outfit builder, Week Ahead, favourites.

Go to http://localhost:8080/login.php and sign in with either set of credentials. To create a brand new account instead, use http://localhost:8080/register.php.

---

## 5. Using the platform

### Adding a board
1. From the Dashboard, click **+ New Board**.
2. Give it a title and pick a category (Top, Bottom, Jewellery, Shoe, Bag, or General).
3. The board appears under its category section on the Dashboard — click into it to start adding images.

### Uploading images
- Inside a board, click **Upload Image** and choose a file from your device, **or**
- Use the Search Unsplash panel (see below) to add images without leaving your browser.
- Every image is automatically colour-analysed and sent to the ML service for tagging (type/subtype/formality) — no manual tagging needed.

### Searching Unsplash
1. Click **Search Unsplash** from a board or the Outfit Builder.
2. Type a query (e.g. "women trousers skirt fashion") or tap one of the quick category/style chips (Tops, Bottoms, Dresses, Jewellery, Shoes, Bags, Minimalist, Dark, Cottagecore).
3. For each result you get two options:
   - **Use in outfit** — guesses the right category from your search, saves the photo, and drops it straight into the matching slot in the Outfit Builder.
   - **Save to board…** — pick a specific board from the dropdown to just save it there, without touching the outfit you're building.

### Using the Outfit Builder
1. Go to **Outfit Builder** from the sidebar.
2. You'll see six category columns: Top, Bottom, Dress, Jewellery, Shoe, Bag.
3. Click any item in a column to select it into that slot (click again to deselect).
4. Once you've picked a few pieces, click **Complete My Outfit** to auto-fill the remaining empty slots — Museboard scores every candidate item in your wardrobe against what you've already picked (colour harmony + formality match) and fills each slot with the best match.
5. Click **Save Outfit** to keep it, or check the Vibe Check panel for a colour-compatibility read and style tips.

### Using The Week Ahead (weekly planner)
1. Go to **Week Ahead** from the sidebar.
2. It pulls the live weather forecast for your location and, for each day, lets you pick an occasion (e.g. Office, Casual, Interview, Dinner, Party).
3. Museboard scores your wardrobe against the day's weather and the occasion's target formality level, and suggests a coordinated outfit per day — no manual outfit-by-outfit planning needed.
4. You can regenerate a day's suggestion or manually swap pieces if you don't like the pick.

### Favouriting boards and outfits
- Click the heart icon on any board or outfit card to favourite it — it updates instantly without a page reload.
- View everything you've favourited under **Favourites** in the sidebar.

### Sharing a board (collaboration)
1. Open a board you own and go to its sharing panel.
2. Enter the username of another registered user and choose their role: **Editor** (can add/edit images) or **Viewer** (read-only).
3. They'll see the board show up as a shared board next time they log in. Remove access any time from the same panel.

### Admin panel (admin account only)
1. Log in with the admin account above and go to **http://localhost:8080/admin.php**.
2. View all registered users, promote/demote admins, or suspend an account.

---

## 6. Common issues

- **App won't load at localhost:8080** — check `docker compose ps` and make sure the `web` container is running
- **Uploads succeed but items never get tagged** — check `docker compose logs ml`; the ML container may still be starting, or the model hasn't been trained yet (uploads still work, just untagged)
- **Database connection errors** — give MySQL a minute to finish initialising on first boot, then restart the web container: `docker compose restart web`
- **Port already in use** — something else on your machine is using 8080/8081/3307/5001; stop that service or change the port mapping in `docker-compose.yml`