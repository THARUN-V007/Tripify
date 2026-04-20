# ✈️ Tripify — AI Travel Planner

So I built this during my semester break after I spent like 3 hours trying to plan a trip to Coorg with my friends and we just ended up doing nothing because nobody could agree on anything and the budget math was a mess. I thought — okay what if an app just *does* all of that for you? You say what kind of trip you want, it gives you options, you pick one, done.

That's basically Tripify. It uses Gemini AI to suggest destinations, pulls real photos from Unsplash, and has a built-in expense tracker so you don't overspend. I also connected it to Firebase so trips actually save and you can come back to them later.

Honestly pretty happy with how it turned out. Took about 2.5 weeks of on-and-off work.

---

## What it does

Okay so here's the cool stuff:

- **Login / Signup** with email and password (Firebase Auth). Nothing fancy but it works and routes are protected so you can't just navigate to `/dashboard` without being logged in
- **AI Destination Suggestions** — you fill out a form (trip type, budget level, duration) and Gemini spits out 3 destination suggestions as a JSON array. This part took the longest to get right because the model kept returning markdown code blocks instead of raw JSON
- **Real photos** for each destination pulled from Unsplash. Makes the cards look actually good
- Once you pick a destination, Gemini generates a full **day-by-day itinerary** and it saves to Firestore so it doesn't regenerate every time you open the trip (saved me a lot of API calls)
- **Expense Tracker** per trip — you can add expenses by category, set a budget limit, and see how much you've spent. It's functional, maybe not the prettiest
- **My Trips Dashboard** where all your saved trips show up with the destination photo
- You can delete trips (and their expenses get deleted too, I had to handle that separately)

The whole thing is responsive so it works on mobile too.

---

## Tech Stack

| Layer | Tech |
|---|---|
| Frontend | React 19 + Vite 8 |
| Styling | Tailwind CSS v3 |
| Routing | React Router DOM v7 |
| Auth + DB | Firebase v12 (Auth + Firestore) |
| AI | Google Gemini API (`@google/generative-ai`) |
| Photos | Unsplash REST API |
| State | React Context API |

I went with Vite because CRA is honestly too slow and I'd already used Vite before. Tailwind made the responsive stuff way easier than writing media queries by hand.

---

## Project Structure

```
tripify/
├── public/
│   └── favicon.svg
├── src/
│   ├── components/
│   │   ├── TripCard.jsx          # Reusable destination/trip card UI
│   │   ├── SkeletonCard.jsx      # Loading skeleton for cards
│   │   └── ExpenseItem.jsx       # Reusable expense row
│   ├── context/
│   │   ├── AuthProvider.jsx      # Global auth state provider (Context API)
│   │   └── authContext.js        # Context object
│   ├── hooks/
│   │   ├── useAuth.js            # Read auth context
│   │   └── useTrips.js           # Custom hook for fetching user trips
│   ├── pages/
│   │   ├── LoginPage.jsx         # Sign in / Sign up
│   │   ├── HomePage.jsx          # Trip preference form + background slider
│   │   ├── SuggestionsPage.jsx   # AI-generated destination cards
│   │   ├── TripPage.jsx          # Day-by-day itinerary view
│   │   ├── ExpensePage.jsx       # Expense tracker per trip
│   │   └── DashboardPage.jsx     # All saved trips
│   ├── services/
│   │   ├── firebase.js           # Firebase init (Auth + Firestore)
│   │   ├── ai.js                 # Gemini API calls
│   │   ├── trips.js              # Trip helpers (delete trip + expenses)
│   │   └── unsplash.js           # Unsplash API calls
│   ├── App.jsx                   # Routes + lazy loading + protected routes
│   ├── main.jsx                  # Entry point
│   └── index.css                 # Tailwind directives
├── .env                          # API keys (never commit)
├── .gitignore
├── index.html
├── tailwind.config.js
├── postcss.config.js
├── vite.config.js
└── package.json
```

I tried to keep the services folder clean — Firebase stuff in one file, AI calls in another, etc. Makes it easier to swap things out later.

---

## Environment Variables

Make a `.env` file at the root of the project and add these. Don't skip this step or nothing will work.

```env
VITE_FIREBASE_API_KEY=your_firebase_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id

VITE_GEMINI_API_KEY=your_gemini_api_key
VITE_GEMINI_MODEL=gemini-1.5-flash-001
VITE_UNSPLASH_ACCESS_KEY=your_unsplash_access_key
```

> Important: `.env` is already in `.gitignore` but double check before pushing. I've seen people accidentally push API keys and then have to rotate everything.

---

## Setup

```bash
# 1. Clone the repository
git clone https://github.com/yourusername/tripify.git
cd tripify

# 2. Install dependencies
npm install

# 3. Add your .env file (see section above)

# 4. Start the development server
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) in your browser.

**Tip:** If you get a blank screen after login, it's almost always a missing or wrong env variable. Check the browser console first.

---

## Getting the API Keys

### Firebase
1. Go to [firebase.google.com](https://firebase.google.com) and create a project
2. Add a Web App and copy the `firebaseConfig` values into your `.env`
3. Go to Authentication > Sign-in method > enable Email/Password
4. Go to Firestore Database > Create database > Start in **test mode** (important — production mode will block all reads/writes until you set rules)

> If you forget to enable Email/Password auth, login will silently fail. Spent 20 minutes debugging that once.

### Google Gemini
1. Go to [aistudio.google.com](https://aistudio.google.com)
2. Click Get API Key > Create new key
3. Paste it into `.env` as `VITE_GEMINI_API_KEY`

> The free tier has rate limits. If you're testing a lot, slow down between requests or you'll get 429 errors.

### Unsplash
1. Go to [unsplash.com/developers](https://unsplash.com/developers) and register an app
2. Copy the Access Key (not the Secret Key) into `.env`

> New Unsplash apps are in demo mode which gives 50 requests/hour. More than enough for testing.

---

## Pages

| Page | Route | What it does |
|---|---|---|
| Login | `/login` | Sign up or sign in |
| Home | `/home` | Pick trip type, budget, duration |
| Suggestions | `/suggestions` | 3 AI destination cards with photos |
| Trip Detail | `/trip/:id` | Full itinerary day by day |
| Expense Tracker | `/expenses/:id` | Track spending per trip |
| Dashboard | `/dashboard` | All your saved trips |

---

## How the AI part works

The Home page has a form where you pick a trip type (Beach / Mountains / City), budget level, and how many days. That gets formatted into a prompt and sent to Gemini, which returns exactly 3 destinations as a JSON array.

Then for each destination, Unsplash fetches a landscape photo using the destination name as the search query. When you click on one and save the trip, Gemini also generates a day-by-day itinerary which gets stored in Firestore. That way if you open the trip again it just loads from the database instead of calling Gemini again.

Getting the AI to return clean JSON was the trickiest part honestly. Had to be very explicit in the prompt that I didn't want markdown formatting, just raw JSON.

---

## Firestore Data Model

```
trips/ (collection)
  └── {tripId}/ (document)
        ├── userId
        ├── destinationName
        ├── destinationCountry
        ├── destinationImage
        ├── budget ("Budget" | "Moderate" | "Luxury")
        ├── numericBudget (number)
        ├── tripType
        ├── duration
        ├── itinerary (array, generated on first view)
        └── createdAt

      expenses/ (sub-collection)
        └── {expenseId}/
              ├── amount
              ├── category
              ├── description
              ├── day
              └── createdAt
```

Expenses are stored as a sub-collection under each trip. This made delete-trip logic a bit annoying because Firestore doesn't auto-delete sub-collections, so I had to query and delete the expenses separately before deleting the trip document.

---

## React Concepts (for the rubric)

My professor wanted to see specific React concepts used — here's where each one shows up. I'm not just listing them, I actually used all of these intentionally.

| Concept | Where |
|---|---|
| `useState` | All pages — form state, loading flags, data |
| `useEffect` | Fetching data in SuggestionsPage, TripPage, etc. |
| `useContext` | `useAuth()` used across all protected pages |
| `useMemo` | Derived values (active trip in HomePage, sorting) |
| `useCallback` | Stable event handlers in HomePage, SuggestionsPage, DashboardPage |
| `useRef` | Auto-focus on email input in LoginPage |
| Context API | `AuthProvider.jsx` wraps the whole app for global auth state |
| Controlled Components | Every form input (budget, duration, expenses) |
| Conditional Rendering | Loading states, error states, empty states everywhere |
| Protected Routes | `ProtectedRoute` component in `App.jsx` |
| React Router v7 | 6 routes total |
| `React.lazy` + `Suspense` | All page components are lazily loaded |
| Lifting State Up | Trip preferences passed between pages via `location.state` |

---

## Known Issues / Future Plans

Being honest here:

- The expense tracker UI is a bit rough. The layout on smaller phones looks a bit cramped and I want to redo it with a cleaner card layout
- No edit functionality yet — you can't update a trip's name or duration after saving it
- The AI sometimes generates a generic itinerary if the destination is not very well known. Need to improve the prompt
- I want to add a map view eventually (Google Maps or Leaflet) to show the destinations
- Dark mode would be nice
- Right now if the Unsplash API fails, the card just shows no image. Should add a fallback

---

## Build & Deploy

```bash
# Build for production
npm run build

# Preview the production build locally
npm run preview

# Deploy to Vercel (recommended)
npx vercel

# Or deploy to Firebase Hosting
npm install -g firebase-tools
firebase login
firebase init hosting   # set dist/ as public directory
firebase deploy
```

Vercel is honestly the easiest option here. Just connect your GitHub repo and it auto-deploys on every push. Make sure you add all the env variables in the Vercel dashboard under Project Settings > Environment Variables — this is easy to forget.

---

## Author

**Tharun V** — 1st year CS student. Built this to actually learn React properly and it worked out better than expected. If something's broken or you have suggestions, feel free to open an issue.

---

MIT License
