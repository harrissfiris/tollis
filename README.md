# TOLLIS: Toll Interoperability System

TOLLIS lets highway toll operators settle what they owe each other. When a vehicle with one operator's tag passes through another operator's toll station, the pass is recorded and charged to the tag's operator. TOLLIS stores these passes and provides the analysis and settlement on top of them: passes per station, charges between operators, who owes whom for a given period, and payments that clear the balance.

Built by **Team 27** for the *Software Engineering* course (2024-25), School of Electrical and Computer Engineering, NTUA.

## Components

| Component | Path | Description |
|---|---|---|
| REST API | `back-end/rest_api` | Flask API over MySQL, served over HTTPS on port `9115` under `/api` |
| Web portal | `back-end/portal-back-end`, `front-end` | Flask web app for operators: dashboard, statistics, station map (Leaflet) and payments |
| CLI client | `cli-client/se2427.py` | Command-line access to every API function, with JSON or CSV output |
| Database | `back-end/rest_api/database` | MySQL schema and setup script (`db_create.sql`) |
| Documentation | `documentation` | UML model (Visual Paradigm), SRS, Postman collection |

**Stack:** Python, Flask, MySQL, HTML/JavaScript, Leaflet.

## API

Authentication is token-based: `POST /api/login` returns a token, which is sent in the `X-OBSERVATORY-AUTH` header.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/tollStationPasses/{tollStationID}/{date_from}/{date_to}` | Passes through a toll station |
| GET | `/api/passAnalysis/{stationOpID}/{tagOpID}/{date_from}/{date_to}` | Passes of one operator's tags at another operator's stations |
| GET | `/api/passesCost/{tollOpID}/{tagOpID}/{date_from}/{date_to}` | Number and total cost of those passes |
| GET | `/api/chargesBy/{tollOpID}/{date_from}/{date_to}` | What every other operator owes a given operator |
| GET | `/api/owedBy/{OpID}/{date_from}/{date_to}` | What a given operator owes the others |
| POST | `/api/makePayment/{FromOpID}/{ToOpID}/{date_from}/{date_to}` | Settle a balance between two operators |
| GET | `/api/getOpNames`, `/api/getTollStations` | Reference data |
| GET/POST | `/api/admin/healthcheck`, `/resetstations`, `/resetpasses`, `/addpasses` | Administration |

Dates use the `YYYYMMDD` format. The Postman collection in `documentation/` contains example requests.

## Running locally

Requirements: Python 3.10+, MySQL 8, OpenSSL.

```bash
git clone https://github.com/harrissfiris/tollis.git
cd tollis

# 1. Create the database
mysql -u root -p < back-end/rest_api/database/db_create.sql

# 2. Create the virtual environment, install dependencies, generate SSL certificates and the .env file
make                         # Windows
make -f MakefileForMac       # macOS / Linux

# 3. Start the API and the web portal (separate terminals)
make run_api                 # or: make -f MakefileForMac run_api
make run_portal              # or: make -f MakefileForMac run_portal
```

The API and portal print their HTTPS address when they start. The certificates are self-signed, so the browser will ask you to accept them.

## CLI

```bash
cd cli-client
python se2427.py login --username <user> --passw <password>
python se2427.py healthcheck
python se2427.py tollstationpasses --station <id> --from 20220101 --to 20220131 --format csv
python se2427.py passanalysis --stationop <op> --tagop <op> --from <date> --to <date>
python se2427.py passescost  --stationop <op> --tagop <op> --from <date> --to <date>
python se2427.py chargesby   --opid <op> --from <date> --to <date>
python se2427.py admin --addpasses --source <file.csv>
```

Run `python se2427.py -h` for the full list of commands and options.
