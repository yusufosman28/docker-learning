# docker-learning

![Docker](https://img.shields.io/badge/Containers-Docker-2496ED?logo=docker&logoColor=white)
![Compose](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![Flask](https://img.shields.io/badge/Python-Flask-000000?logo=flask&logoColor=white)
![MySQL](https://img.shields.io/badge/Database-MySQL%205.7-4479A1?logo=mysql&logoColor=white)

A Flask app that connects to a MySQL database, containerised with Docker and run with Docker Compose. The page shows the version of the MySQL server it reached.

The image is published on Docker Hub as [`yusufosman1280/flask-mysql:v1`](https://hub.docker.com/r/yusufosman1280/flask-mysql).

---

## What I Built

The project grew in stages, one commit per stage:

1. **A plain Flask app** on port 5003, run locally first with `python3 app.py`
2. **My own Dockerfile** on `python:3.12-slim`, built and run with `docker build` and `docker run -p`
3. **Flask connected to MySQL** - the app container and a `mysql:5.7` container on a custom Docker network, the app finding the database by name
4. **One `docker-compose.yml`** replacing four separate commands (network, database, build, run)
5. **Published to Docker Hub** - tagged, pushed, deleted locally and pulled back to prove the round trip

---

## Architecture

```
Browser ──► web (Flask, port 5003, published) ──► mydb (MySQL, port 3306, not published)
```

- Compose creates one network and puts both services on it.
- `web` finds the database by its **service name**, `mydb`.
- Only `web` publishes a port, because only the browser calls it from outside. `mydb` is reached from inside the network, so its port stays closed to the laptop.

---

## Project Structure

```
docker-learning/
├── README.md
└── hello_flask/
    ├── app.py               # Flask app: connects to mydb and returns the MySQL version
    ├── Dockerfile           # python:3.12-slim + build tools + flask and mysqlclient
    └── docker-compose.yml   # web (built from the Dockerfile) + mydb (mysql:5.7)
```

---

## How to Run It

Needs Docker with Docker Compose.

```bash
git clone https://github.com/yusufosman28/docker-learning.git
cd docker-learning/hello_flask
docker compose up -d
```

Open `http://localhost:5003`:

```
Hello, World! MySQL version: 5.7.44
```

Stop and remove the containers and network:

```bash
docker compose down
```

> [!NOTE]
> The MySQL root password in `app.py` and `docker-compose.yml` is a local demo value for this learning project, not a real credential.

---

## Results

| Measurement | Result |
|---|---|
| Plain Flask image, `python:3.12` (full) vs `python:3.12-slim` | **418 MB → 49.1 MB**, about 88% smaller |
| Flask + MySQL image on Docker Hub (`v1`) | **138.8 MB** |

The slim vs full comparison used the same plain Flask app with only the base image changed, built from a separate test Dockerfile that is not in this repository.

---

## What Broke and How I Fixed It

- **`mysqlclient` would not build on the slim image.** Filtering the build output with `grep` found `pkg-config: not found` (exit status 127). Added `gcc`, `pkg-config` and `default-libmysqlclient-dev` instead of switching to the 418 MB full image. [Full write-up](https://github.com/yusufosman28/DevOps_labs/blob/main/05-docker/troubleshooting/01-mysqlclient-build-failed.md)
- **Compose could not find the Dockerfile.** The compose file was in the folder above `hello_flask`, so `build: .` pointed at the wrong place. [Full write-up](https://github.com/yusufosman28/DevOps_labs/blob/main/05-docker/troubleshooting/03-compose-file-in-the-wrong-folder.md)
- **`Unknown server host 'mydb'`.** The app asked for `mydb` but the Compose service was called `db`. Renamed the service. [Full write-up](https://github.com/yusufosman28/DevOps_labs/blob/main/04-networking/troubleshooting/02-unknown-server-host.md)
- **The browser showed a working page while the app was returning a 500.** An old tab was showing a saved copy; a hard refresh showed the real error. [Full write-up](https://github.com/yusufosman28/DevOps_labs/blob/main/04-networking/troubleshooting/01-browser-showed-a-saved-page.md)

---

## Key Takeaways

- A failed build means no image, so every command after it fails too. Fix the first error.
- Containers on the same network find each other by name; only traffic from outside needs a published port.
- Changing `app.py` needs a rebuild (`docker compose up --build`), because `COPY . .` copies the code in at build time.

---

My notes for the whole Docker module are in [DevOps_labs/05-docker](https://github.com/yusufosman28/DevOps_labs/tree/main/05-docker).
