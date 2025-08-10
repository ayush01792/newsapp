Newsphere — API README
Awesome — here’s a clean, practical README you can drop into your Newsphere (React) app repo to document the backend API. It covers endpoints, auth, examples (fetch/axios), env variables, running & deployment notes, error handling, and tips for testing. Copy-paste and tweak to match your actual implementation.

Newsphere API
Simple REST API that serves news articles & sources to the Newsphere React frontend.

Base URL (example)
http://localhost:5000/api

Replace with your deployed URL for production (e.g. https://api.yourdomain.com/api).

Table of contents
Authentication

Environment variables

Endpoints

Request examples (fetch / axios)

Response format

Error handling & status codes

CORS, rate limiting & security notes

Local setup & run

Testing & debugging tips

Deployment notes

Authentication
The API uses an API key / Bearer token scheme for protected endpoints (e.g. creating/updating articles).

Public read endpoints (listing articles) are open (or optionally limited by rate).

Send the API key in the Authorization header:

makefile
Copy
Edit
Authorization: Bearer <API_KEY>
Environment variables
Example .env file:

ini
Copy
Edit
PORT=5000
NODE_ENV=development
MONGO_URI=mongodb://localhost:27017/newsphere
JWT_SECRET=replace_with_secure_secret
API_KEY=replace_with_api_key_for_admin_actions
NEWS_PROVIDER_API_KEY=your_third_party_news_api_key  # optional
CORS_ORIGIN=http://localhost:3000
Endpoints
Public
GET /api/articles

Query params:

q — search keyword (title/content).

source — filter by source id.

category — category name (e.g., technology).

page — page number (default 1).

limit — items per page (default 20).

Example: /api/articles?q=ai&category=technology&page=1&limit=10

GET /api/articles/:id

Get single article by id.

GET /api/top-headlines

Optional query: country, category, limit.

GET /api/sources

Returns available news sources (id, name, url, category).

GET /api/categories

Returns categories supported.

Protected (admin / editorial)
POST /api/articles

Create article. Body: { title, content, author, sourceId, category, publishedAt, urlToImage }

Requires Authorization: Bearer <API_KEY>.

PUT /api/articles/:id

Update article fields. Protected.

DELETE /api/articles/:id

Delete article. Protected.

Optional / integrations
GET /api/fetch-external?source=...&q=...

Proxy to third-party provider (e.g., NewsAPI) to avoid exposing keys in frontend.

Request examples
Fetch (public articles)
js
Copy
Edit
// simple fetch from React
const res = await fetch('/api/articles?q=technology&page=1');
const data = await res.json();
console.log(data);
Axios (with API key for protected)
js
Copy
Edit
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:5000/api',
  headers: { Authorization: `Bearer ${process.env.REACT_APP_API_KEY}` }
});

// create article (admin)
const newArticle = { title: 'Hello', content: 'World', sourceId: 'bbc' };
await api.post('/articles', newArticle);
Example: fetch top headlines with query params
js
Copy
Edit
const params = new URLSearchParams({ country: 'in', category: 'business', limit: 10 });
fetch(`/api/top-headlines?${params.toString()}`)
  .then(res => res.json())
  .then(data => console.log(data));
Response format
Successful list response (200):

json
Copy
Edit
{
  "status": "ok",
  "page": 1,
  "limit": 10,
  "total": 234,
  "articles": [
    {
      "id": "64f8e4b2a3c8fe1...",
      "title": "AI breakthrough in X",
      "content": "...",
      "author": "Jane Doe",
      "source": { "id": "bbc", "name": "BBC News" },
      "category": "technology",
      "publishedAt": "2025-08-06T12:00:00.000Z",
      "url": "https://...",
      "urlToImage": "https://..."
    }
  ]
}
Single-entity response:

json
Copy
Edit
{ "status": "ok", "article": { ... } }
Error example (400/404/500):

json
Copy
Edit
{
  "status": "error",
  "message": "Validation error: title is required"
}
Error handling & status codes
200 OK — success

201 Created — resource created

400 Bad Request — validation or malformed request

401 Unauthorized — missing/invalid API key

403 Forbidden — valid key, but insufficient permissions

404 Not Found — resource not found

429 Too Many Requests — rate limited

500 Internal Server Error — server error

Always include helpful message in JSON body for frontend to show user-friendly errors.

CORS, rate limiting & security notes
Configure CORS to accept requests from your React app origin (CORS_ORIGIN env var).

Use rate limiting (e.g., express-rate-limit) on public endpoints to prevent abuse.

Never expose third-party API keys to the frontend — use server-side proxy endpoints.

Add input validation (e.g., express-validator) to prevent malformed data.

Sanitize HTML content if storing rich content to avoid XSS.

Local setup & run (Node/Express example)
Clone repo and cd server

Create .env (see env example above)

Install dependencies:

nginx
Copy
Edit
npm install
Run (development):

arduino
Copy
Edit
npm run dev
or

nginx
Copy
Edit
node server.js
API should be available at http://localhost:5000/api.

If you're using MongoDB locally:

Ensure mongod is running

Use MONGO_URI to point to your DB

Sample minimal Express route (reference)
js
Copy
Edit
// routes/articles.js
const express = require('express');
const router = express.Router();

// GET /api/articles
router.get('/', async (req, res) => {
  const { q, page = 1, limit = 20 } = req.query;
  // TODO: implement search + pagination
  const articles = await Article.find(buildQuery(q)).skip((page-1) * limit).limit(Number(limit));
  res.json({ status: 'ok', page: Number(page), limit: Number(limit), total: 123, articles });
});

module.exports = router;
Testing & debugging tips
Use Postman / Insomnia to exercise endpoints (save a collection).

Add logging (morgan/winston) to capture request/response for debugging.

Use unit tests for controllers (Jest + supertest for API tests).

Simulate rate limits in staging and test CORS from the actual React origin.

Frontend integration notes (React)
Keep API base URL in .env:

REACT_APP_API_BASE_URL=http://localhost:5000/api

Create a small API client wrapper (axios instance) for consistent headers & error handling.

Cache lists (React Query / SWR) to reduce hits and improve UX.

Show loading states and friendly error messages.

Deployment notes
Set proper environment variables in your hosting provider (Heroku / Vercel serverless function / AWS Elastic Beanstalk / DigitalOcean).

If deploying serverless, adapt routes to serverless handlers and ensure long-running DB connections are handled correctly.

Use HTTPS (TLS), and restrict CORS to your front-end domain.

Example Postman quick collection (minimal)
GET /api/articles

GET /api/articles/:id

GET /api/top-headlines
