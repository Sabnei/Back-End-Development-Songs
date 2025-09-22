# Get Songs Microservice (Flask)

This repository contains a Flask microservice that exposes song lyrics from a MongoDB database.

It corresponds to a part of the "Back-end Application Development Capstone" course project:
- View popular song lyrics
- Health endpoint
- Song CRUD

## Requirements
- Python 3.10+
- MongoDB reachable on the network (local or container)

## Installation

```bash
# Create and activate a virtual environment (optional)
python -m venv .venv
# Windows PowerShell
. .venv\\Scripts\\Activate.ps1

# Install dependencies
pip install -r requirements.txt
```

## Environment variables
The service connects to MongoDB using environment variables. Set them before starting the app:

- `MONGODB_SERVICE`: host or host:port of MongoDB service (required). E.g., `localhost:27017` or `mongodb`.
- `MONGODB_USERNAME`: username (optional if Mongo has no auth)
- `MONGODB_PASSWORD`: password (optional if Mongo has no auth)

Example (PowerShell):

```powershell
$env:MONGODB_SERVICE="localhost:27017"
# If authentication applies
# $env:MONGODB_USERNAME="user"
# $env:MONGODB_PASSWORD="secret"
```

Note: If `MONGODB_SERVICE` is not set, the application will exit on startup.

## Seed data
On startup, the service reads `backend/data/songs.json` and performs:
- `db.songs.drop()`
- `db.songs.insert_many(songs_list)`

This repopulates the `songs` collection every time the app starts.

## Run locally

```bash
python app.py
```

By default it listens on `http://0.0.0.0:8080`.

## Endpoints

Base URL: `http://<host>:8080`

- `GET /health` → service health
  - Response: `{ "status": "OK" }`

- `GET /count` → number of documents in `songs`
  - Response: `{ "count": <number> }`

- `GET /song` → list all songs
  - Response: `{ "songs": [ { ... }, ... ] }`

- `GET /song/<id>` → get a song by numeric `id`
  - 200: song document
  - 404: `{ "message": "song with id not found" }`

- `POST /song` → create a song
  - Example JSON body:
    ```json
    {
      "id": 101,
      "title": "My Song",
      "lyrics": "Lorem ipsum..."
    }
    ```
  - 201: `{ "inserted id": "<ObjectId>" }`
  - 302 if id already exists: `{ "Message": "song with id <id> already present" }`

- `PUT /song/<id>` → update fields of the song with that `id`
  - 201: updated document
  - 200 if no changes: `{ "message": "song found, but nothing updated" }`
  - 404 if not found: `{ "message": "song not found" }`

- `DELETE /song/<id>` → delete the song
  - 204 with no body
  - 404 if not found: `{ "message": "song not found" }`

## Quick examples (curl)

```bash
# Health
curl -s http://localhost:8080/health

# Count
curl -s http://localhost:8080/count

# List songs
curl -s http://localhost:8080/song | jq '.songs | length'

# Get by id
curl -s http://localhost:8080/song/1 | jq

# Create
curl -s -X POST http://localhost:8080/song \
  -H "Content-Type: application/json" \
  -d '{"id":201,"title":"New","lyrics":"hi"}' | jq

# Update
curl -s -X PUT http://localhost:8080/song/201 \
  -H "Content-Type: application/json" \
  -d '{"title":"Updated"}' | jq

# Delete
curl -i -X DELETE http://localhost:8080/song/201
```

## Tests

This repo includes minimal tests with `pytest`.

```bash
pytest -q
```

The current test validates `GET /health`.

## Deployment

This microservice is part of the capstone, deployed to platforms such as IBM Code Engine / OpenShift / IKS as per the course instructions. Configure environment variables and MongoDB connection secrets per environment.