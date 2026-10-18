# Phoebe

## Overview

A Python micro-service that fills open caregiver shifts by fanning out SMS and voice notifications, accepting the first claim, and escalating if unfilled.

## Functional requirements

- The service interface should expose two endpoints:
  - An API route to trigger fanout for a shift: `POST /shifts/{shift_id}/fanout`
  - An API route to receive an incoming SMS/phone message: `POST /messages/inbound`
- The escalation logic should consist of two rounds of contact attempts:
  1. An immediate round of contact attempts via SMS.
  2. If no caregiver accepts the shift within 10 minutes, a second round of contact attempts via phone call.
- Only caregivers with the role required by the shift (LPN, RN, CNA, etc.) should be contacted for a shift.
- The first caregiver to accept should be assigned to the shift.
- A shift should only ever be given to one caregiver, even if multiple caregivers accept it near-simultaneously.

## Constraints

- Endpoints should be idempotent; re-posting the same shift fan-out must not send duplicate notifications.
- The service may have multiple in-process worker tasks running concurrently. Guard against race conditions when marking a shift as claimed.
- For storage, use the provided in-memory key/value database (modify as desired); do not use a persistent database or migrations.

## Assumptions & simplifications

- We do not need to handle the case where no caregiver ever accepts the shift.
- Assume that sending and receiving phone calls has the same API as a text message: we send an outgoing message and receive an incoming message. Don't worry about multi-turn phone conversations or streaming call transcripts.
- Assume only one instance of this micro-service is running (no cross-process locking required).

## What’s included

- FastAPI bootstrap (`app/api.py`)
- Stub notifier (`app/notifier.py`)
- Stub intent classifier (`app/intent.py`)
- Generic in-memory key/value database (`app/database.py`)
- Example tests (`tests/example_test.py`)
- Sample JSON data to use as reference (`sample_data.json`).

Everything else (how you model data, structure modules, design the service) is up to you to decide.

## Environment setup

Install [uv](https://docs.astral.sh/uv/) (for Windows see [installation guide](https://docs.astral.sh/uv/getting-started/installation/)):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

If using VSCode/Cursor, we recommend you install the [Ruff](https://marketplace.cursorapi.com/items?itemName=charliermarsh.ruff) and [Pylance](https://marketplace.cursorapi.com/items?itemName=ms-python.vscode-pylance) extensions.

## Commands

#### Run tests

```bash
uv run pytest
```

#### Lint code

```bash
uv run ruff check
```

#### Run API server

_(Not strictly necessary, only if you want to test manually making requests to your api server via `curl`)_

```bash
uv run uvicorn app.api:create_app --factory --reload
```

```bash
curl http://127.0.0.1:8000/health
```
