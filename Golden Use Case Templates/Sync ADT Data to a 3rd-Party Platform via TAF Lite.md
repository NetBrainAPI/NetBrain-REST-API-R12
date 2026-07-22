
# Sync ADT Data to a 3rd-Party Platform via TAF Lite
In this use case, we use **TAF Lite** (Triggered Automation Framework Lite) to pull a NetBrain **Automation Data Table (ADT)** — for example, a CVE/vulnerability report built with a Network Intent — and push it into a **3rd-party platform** (e.g. a vulnerability-management, CMDB, or ITSM system) so that platform stays continuously in sync with the data NetBrain maintains for the network. The upload step follows a two-step REST connector pattern (exchange a long-lived API token for a short-lived access token, then upload a file to a named connector) that is common across many 3rd-party platforms — adjust the endpoint paths and payload shape shown in Step 4 to match your specific target system's API.

Unlike the classic TAF flow (which requires the 3rd-party system to raise a ticket and NetBrain to match an incident type), TAF Lite lets the 3rd-party system call one API directly against a published ADT view — no ticket, no incident type, no `specify_a_working_domain` step, because the `endpoint`/`passKey` pair generated when the view is published already scopes the call to the right tenant, domain, and table.

**[Step 1: Use case preparation](Step-1:-Use-case-preparation)**
>> 1a. Import modules and set configuration<br>
>> 1b. Authenticate with OAuth 2.0 client credentials (recommended) or legacy username/password<br>

**[Step 2: One-time setup — publish the ADT view to TAF Lite](Step-2:-One-time-setup)**
>> 2a. Add the ADT view to TAF Lite in the NetBrain GUI<br>
>> 2b. Note the generated Endpoint and Pass-key<br>

**[Step 3: Retrieve ADT data via TAF Lite](Step-3:-Retrieve-ADT-data-via-TAF-Lite)**
>> 3a. Call the Get ADT Data API<br>
>> 3b. Page through the full result set and normalize rows<br>
>> 3c. Write the result to CSV<br>

**[Step 4: Push the data to the 3rd-party platform](Step-4:-Push-the-data-to-the-3rd-party-platform)**
>> 4a. Exchange the 3rd-party API token for a short-lived access token<br>
>> 4b. Upload the CSV to the 3rd-party connector<br>

**[Step 5: Automate the sync (optional)](Step-5:-Automate-the-sync)**

**[Full Reusable Module](https://github.com/NetBrainAPI/NetBrain-REST-API-R12/blob/main/Golden%20Use%20Case%20Templates/Sync%20ADT%20Data%20to%20a%203rd-Party%20Platform%20via%20TAF%20Lite.md#full-reusable-module)** — a production-ready, 3-file version of Steps 1–4 for scheduled/unattended runs.

## Step 1: Use case preparation
***1a. Import modules and set configuration.***
> Note: Replace `nb_url` with your own NetBrain server URL before running this.

***1b. Authenticate.***
> NetBrain R12+ supports OAuth 2.0 client credentials via Open API, which is recommended over legacy username/password login because tokens auto-refresh and no user password is stored in the script. To set it up: sign in as a System Admin, go to **System Management > Open API**, confirm the OAuth server is enabled, then **Add OAuth Client** and copy the generated **Client ID** / **Client Secret**. See [Login API — OAuth2.0](https://github.com/NetBrainAPI/NetBrain-REST-API-R12/blob/main/REST%20APIs%20Documentation/Authentication%20and%20Authorization/Login%20API.md#oauth20---recommended) for the full walkthrough. Legacy username/password login ([Login API — Session API](https://github.com/NetBrainAPI/NetBrain-REST-API-R12/blob/main/REST%20APIs%20Documentation/Authentication%20and%20Authorization/Login%20API.md#session-api)) still works and is shown as a fallback.

```python
import json
import time
import requests
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

nb_url = "https://<your-netbrain-host>"     # e.g. "https://nb.company.com"
verify_ssl = False                          # NetBrain often uses a self-signed cert

headers = {"Content-Type": "application/json", "Accept": "application/json"}
```

```python
# 1b (recommended). OAuth 2.0 client credentials
# Note: the token endpoint requires form-urlencoded, not JSON — a JSON body returns
# {"error": "invalid grant type"}. Send client_id/client_secret as form fields
# (not HTTP Basic auth) -- this is the shape confirmed to work against NetBrain.

client_id = "<your-oauth-client-id>"
client_secret = "<your-oauth-client-secret>"

def login_oauth(nb_url, client_id, client_secret):
    url = nb_url + "/ServicesAPI/auth/oauth2/token"
    body = {
        "grant_type": "client_credentials",
        "client_id": client_id,
        "client_secret": client_secret,
    }
    oauth_headers = {"Content-Type": "application/x-www-form-urlencoded",
                      "Accept": "application/json"}
    resp = requests.post(url, data=body, headers=oauth_headers, verify=verify_ssl)
    resp.raise_for_status()
    data = resp.json()
    return data["access_token"], data.get("expires_in", 3600)

token, expires_in = login_oauth(nb_url, client_id, client_secret)
print(f"token acquired, expires in {expires_in}s")
```

API Response:

    token acquired, expires in 3600s

> **Production tip:** for a script that runs once and exits, a single token is enough. For a script that runs continuously or on a long-lived schedule, wrap the client in a small class that tracks the token's `expires_in` and re-calls `login_oauth()` a safety margin (e.g. 60s) before it expires, so the token never goes stale mid-run. See the reusable `NetBrainADTClient` in the **Full Reusable Module** section below for a working example of this pattern.

```python
# 1b (fallback). Legacy username/password session login

def login_session(nb_url, username, password):
    url = nb_url + "/ServicesAPI/API/V1/Session"
    body = {"username": username, "password": password}
    resp = requests.post(url, data=json.dumps(body), headers=headers, verify=verify_ssl)
    resp.raise_for_status()
    return resp.json()["token"]

# token = login_session(nb_url, "<username>", "<password>")
```

## Step 2: One-time setup
***2a. Add the ADT view to TAF Lite.***
> This is a one-time step done in the NetBrain GUI, not the API — TAF Lite authorizes each ADT view individually so an external system can only reach the views you've explicitly published.

> Note: publishing an ADT view to TAF Lite (Triggered Automation Manager) requires the **Triggered Diagnosis Management** privilege, which by default is granted only to the **Domain Admin** role.

1. In NetBrain, build (or open) the ADT view you want to sync — for this use case, a CVE/vulnerability report generated by a Network Intent, with columns such as `Device`, `CVE Number`, `Severity`, `CVSS Score`, `Description`.
2. Go to **Triggered Automation Manager > TAF Lite tab > +Add ADT**.
3. Select the **ADT** and the **ADT View** to expose (all columns in the selected view become callable by API).
4. Under **Set API Trigger Parameters**, note the generated **Endpoint** (e.g. `T10001U`) and **Pass-key** (e.g. `TAFLite!1234567890`). These authenticate the API caller to this one view — they are not the `t`/`d`/`id` values from an `adtShare.html` link, which are for the read-only browser share instead.

```python
adt_endpoint = "T10001U"
adt_passkey  = "TAFLite!1234567890"
```

## Step 3: Retrieve ADT data via TAF Lite
***3a. Call the Get ADT Data API.***
> `POST /ServicesAPI/API/V3/TAF/Lite/adt/data` returns one page of rows for the published view, each cell typed per column (`string`, `device`, `map`, `time`, etc. — see [4 - Get ADT data.md](https://github.com/NetBrainAPI/NetBrain-REST-API-R12/blob/main/REST%20APIs%20Documentation/TAF%20Lite%20%28Triggered%20Automation%20Framework%20Lite%29/4%20-%20Get%20ADT%20data.md) for the full cell-type reference). If you get errors, verify the endpoint/passKey are the values from "Add to TAF Lite" (not placeholders) and that the view is still published.

***3b. Page through the full result set.***
> The API pages results; keep requesting `pageNumber + 1` until a page comes back shorter than `pageSize`.

```python
def get_adt_page(nb_url, token, endpoint, pass_key, page_number, page_size=200,
                  filter_devices=None, columns=None, auto_refresh=False):
    url = nb_url + "/ServicesAPI/API/V3/TAF/Lite/adt/data"
    body = {
        "endpoint": endpoint,
        "passKey": pass_key,
        "filterDevices": filter_devices or [],
        "columns": columns or [],          # [] = return all columns
        "option": {
            "autoRefresh": auto_refresh,
            "pageSize": page_size,
            "pageNumber": page_number,
        },
    }
    call_headers = dict(headers, token=token)
    resp = requests.post(url, json=body, headers=call_headers, verify=verify_ssl)
    resp.raise_for_status()
    data = resp.json()
    if data.get("statusCode") not in (None, 790200):
        raise RuntimeError(f"ADT API error {data.get('statusCode')}: {data.get('statusDescription')}")
    return data


def extract_cell(cell):
    """Pull the human-meaningful value out of a typed ADT cell."""
    if isinstance(cell, dict):
        return cell.get("value", cell.get("name", cell))
    return cell


def get_all_adt_records(nb_url, token, endpoint, pass_key, page_size=200):
    records = []
    columns = []
    page = 1
    while True:
        data = get_adt_page(nb_url, token, endpoint, pass_key, page, page_size)
        columns = data.get("columns") or columns
        col_names = [c["name"] for c in columns]
        rows = data.get("rows", []) or []
        for row in rows:
            records.append({name: extract_cell(cell) for name, cell in zip(col_names, row)})
        if len(rows) < page_size:
            break
        page += 1
    return data.get("adtName", ""), columns, records


adt_name, columns, records = get_all_adt_records(nb_url, token, adt_endpoint, adt_passkey)
print(f"ADT '{adt_name}': {len(records)} rows, {len(columns)} columns")
records[:2]
```

API Response:

    ADT 'CVE Report': 1234 rows, 5 columns

    [{'Device': 'core-sw01', 'CVE Number': 'CVE-2024-XXXXX', 'Severity': 'Critical',
      'CVSS Score': '9.8', 'Description': 'Example vulnerability description'},
     {'Device': 'edge-rtr02', 'CVE Number': 'CVE-2024-YYYYY', 'Severity': 'High',
      'CVSS Score': '7.5', 'Description': 'Example vulnerability description'}]

***3c. Write the result to CSV.***
> Most 3rd-party connectors consume a CSV whose columns match a mapping configured on the connector's side — align your ADT view's columns with that mapping before uploading (see Step 4).

```python
import csv

def write_csv(path, columns, records):
    names = [c["name"] for c in columns]
    with open(path, "w", newline="", encoding="utf-8-sig") as fh:
        writer = csv.DictWriter(fh, fieldnames=names, extrasaction="ignore")
        writer.writeheader()
        for rec in records:
            writer.writerow(rec)

csv_path = "adt_export.csv"
write_csv(csv_path, columns, records)
print(f"wrote {csv_path}")
```

API Response:

    wrote adt_export.csv

## Step 4: Push the data to the 3rd-party platform
Many 3rd-party connector APIs use a two-step flow: exchange a long-lived API token for a short-lived access token, then upload the file to a specific named connector. The example below follows that shape — **replace the endpoint paths, field names, and payload shape with the ones documented by your actual target platform.**

***4a. Exchange the API token for an access token.***
> Generate the long-lived API token from your 3rd-party platform's admin/settings UI (commonly under a section like "API Tokens" or "Integrations").

```python
thirdparty_base_url = "https://<3rd-party-host>/api/v1"       # your target platform's API base URL
thirdparty_api_token = "<your-3rd-party-api-token>"

def get_thirdparty_access_token(base_url, api_token):
    # Example shape — adjust the path and payload to match your platform's docs.
    url = f"{base_url}/authentication/token/"
    resp = requests.post(url, json={"name": "netbrain-adt-sync"},
                          headers={"Authorization": api_token,
                                   "Content-Type": "application/json",
                                   "Accept": "application/json"})
    resp.raise_for_status()
    return resp.json()["access_token"]

access_token = get_thirdparty_access_token(thirdparty_base_url, thirdparty_api_token)
```

***4b. Upload the CSV to the 3rd-party connector.***
> Create/configure the connector first in the 3rd-party platform's UI — its ID typically appears in the connector's URL or connector list, and its column mapping must match the CSV columns you exported in Step 3c. Many platforms also expose an "end of cycle" flag: set it `True` when this is the last file in a sync run so the platform closes out the batch; use `False` if more files will follow before the batch should be processed.

```python
connector_id = "<your-connector-id>"

def upload_to_thirdparty(base_url, access_token, connector_id, csv_path, is_end_of_cycle=True):
    # Example shape — adjust the path, multipart field name, and form fields to match
    # your platform's docs.
    url = f"{base_url}/report/upload/{connector_id}/"
    with open(csv_path, "rb") as fh:
        files = {"file": (csv_path, fh, "text/csv")}
        data = {"is_end_of_cycle": str(is_end_of_cycle).lower()}
        resp = requests.post(url, headers={"Authorization": f"Bearer {access_token}"},
                              files=files, data=data)
    resp.raise_for_status()
    return resp.json() if resp.content else {}

result = upload_to_thirdparty(thirdparty_base_url, access_token, connector_id, csv_path)
print(f"succeeded: {len(result.get('success_uploading', []))}, "
      f"failed: {len(result.get('failed_uploading', []))}")
```

API Response:

    succeeded: 1234, failed: 0

## Step 5: Automate the sync
Wrap Steps 1–4 into a single script and run it on a schedule so the 3rd-party platform stays continuously in sync:

- **Windows Task Scheduler** — Program: `python`, Arguments: `sync_adt_data.py`, Start in: the script's folder.
- **Linux cron** (hourly):
  ```
  0 * * * * cd /opt/netbrain-sync && /usr/bin/python3 sync_adt_data.py >> sync.log 2>&1
  ```
- Skip the upload if the ADT came back empty, so an empty file doesn't close out a sync cycle with no data:
  ```python
  if len(records) == 0:
      print("ADT is empty -- skipping upload.")
  else:
      upload_to_thirdparty(thirdparty_base_url, access_token, connector_id, csv_path)
  ```

## Full Reusable Module
The step-by-step cells above are meant to be copy-pasted and run interactively. For a real deployment (e.g. a scheduled sync job), it's worth splitting the same logic into three small files so NetBrain-side and 3rd-party-side code stay independent and either side can be swapped out without touching the other:

| File | Role |
|---|---|
| `sync_backbone.py` | The only file you edit and run. Holds configuration and the `main()` that drives the end-to-end flow. |
| `netbrain_adt_client.py` | All NetBrain operations — OAuth/session auth with automatic token refresh, TAF Lite paging, CSV/JSON export. |
| `thirdparty_client.py` | All 3rd-party operations — token exchange and file upload. Replace the endpoint paths inside with your actual target platform's API. |

***`netbrain_adt_client.py`***
```python
"""
netbrain_adt_client.py
=======================
NetBrain operations module for syncing ADT data to a 3rd-party platform.

Encapsulates everything needed to fetch the contents of a NetBrain Automation
Data Table (ADT) through the TAF Lite (Triggered Automation Framework Lite)
REST API and return it as plain Python objects, plus JSON / CSV export.

This module holds NO configuration and NO entry point -- it is driven by the
backbone script (e.g. sync_backbone.py), which calls `fetch_adt(...)`.
It can also be imported directly:

    from netbrain_adt_client import fetch_adt, NetBrainADTClient, ADTResult

API reference:
  NetBrain REST API R12.3a -> TAF Lite -> "4 - Get ADT data"
  POST /ServicesAPI/API/V3/TAF/Lite/adt/data

Authentication model (two layers):
  1. API token   -> obtained from OAuth 2.0 (CLIENT_ID / CLIENT_SECRET) or a
                    legacy session login (USERNAME / PASSWORD). Sent in the
                    `token` request header.
  2. ADT pass key -> the `endpoint` + `passKey` pair that NetBrain generates
                    when an ADT *view* is added to TAF Lite. This authorizes
                    access to that one specific ADT view.

NOTE on the adtShare.html link
------------------------------
A share URL such as
    https://<server>/adtShare.html?t=<tenantId>&d=<domainId>&id=<adtId>
is the browser "read-only share" of a table. Its t / d / id values are NOT the
TAF Lite `endpoint` / `passKey`. To pull data programmatically you must, inside
NetBrain, open the ADT -> Table Builder / "..." menu -> "Add to TAF Lite"
(a.k.a. publish to TAF Lite). That action produces the `endpoint` (e.g.
"T100007") and `passKey` (e.g. "TAFLite!1234567890") that this module needs.
See parse_share_url() for a helper that extracts t/d/id for reference.

Dependencies:  pip install requests
"""

from __future__ import annotations

import csv
import json
import time
from dataclasses import dataclass, field
from typing import Any, Dict, Iterable, List, Optional
from urllib.parse import urlparse, parse_qs

import requests


@dataclass
class NetBrainADTClient:
    """Thin client around the NetBrain TAF Lite 'Get ADT data' endpoint."""

    server_url: str                       # e.g. "https://nb.company.com"
    token: Optional[str] = None           # current API token
    verify_ssl: bool = True               # set False for self-signed certs
    timeout: int = 60                     # seconds
    # OAuth client-credentials (kept so the token can be auto-refreshed on expiry)
    client_id: Optional[str] = None
    client_secret: Optional[str] = None
    token_lifetime_fallback: int = 3600   # used if the server omits expires_in
    refresh_margin: int = 60              # refresh this many sec before expiry
    session: requests.Session = field(default_factory=requests.Session, repr=False)
    _token_expiry: Optional[float] = field(default=None, repr=False)  # epoch secs

    def __post_init__(self) -> None:
        self.server_url = self.server_url.rstrip("/")
        self.session.verify = self.verify_ssl
        if not self.verify_ssl:
            requests.packages.urllib3.disable_warnings()  # type: ignore[attr-defined]

    # ---- authentication --------------------------------------------------- #
    def login(self, username: str, password: str) -> str:
        """Legacy session login (USERNAME / PASSWORD). Returns the token."""
        url = f"{self.server_url}/ServicesAPI/API/V1/Session"
        resp = self.session.post(
            url,
            json={"username": username, "password": password},
            headers={"Content-Type": "application/json", "Accept": "application/json"},
            timeout=self.timeout,
        )
        resp.raise_for_status()
        data = resp.json()
        token = data.get("token")
        if not token:
            raise RuntimeError(f"Login failed: {data}")
        self.token = token
        self._token_expiry = None          # session tokens are not auto-refreshed here
        return token

    def login_oauth(self, client_id: str, client_secret: str) -> str:
        """
        Obtain an OAuth 2.0 access token via client_credentials and store it
        together with its expiry. The credentials are retained so the token
        can be transparently re-fetched once it expires.
        POST /ServicesAPI/auth/oauth2/token
        """
        if not client_id or not client_secret:
            raise RuntimeError("CLIENT_ID and CLIENT_SECRET are required for OAuth.")
        url = f"{self.server_url}/ServicesAPI/auth/oauth2/token"
        # NetBrain's token endpoint requires form-urlencoded (OAuth 2.0 standard);
        # a JSON body is rejected with {"error":"invalid grant type"}.
        resp = self.session.post(
            url,
            data={
                "grant_type": "client_credentials",
                "client_id": client_id,
                "client_secret": client_secret,
            },
            headers={
                "Content-Type": "application/x-www-form-urlencoded",
                "Accept": "application/json",
            },
            timeout=self.timeout,
        )
        resp.raise_for_status()
        data = resp.json()
        token = data.get("access_token")
        if not token:
            raise RuntimeError(f"OAuth token request failed: {data}")
        expires_in = data.get("expires_in") or self.token_lifetime_fallback
        self.client_id = client_id
        self.client_secret = client_secret
        self.token = token
        self._token_expiry = time.time() + float(expires_in)
        return token

    def _token_expired(self) -> bool:
        """True if the token is missing or within refresh_margin of expiry."""
        if self._token_expiry is None:
            return False                   # non-expiring (session) token
        return time.time() >= (self._token_expiry - self.refresh_margin)

    def _ensure_token(self) -> None:
        """Refresh the OAuth token if it has expired (or is about to)."""
        if self.client_id and self.client_secret and (
            self.token is None or self._token_expired()
        ):
            self.login_oauth(self.client_id, self.client_secret)

    def logout(self) -> None:
        """Best-effort session teardown (only meaningful for userpass login)."""
        if not self.token:
            return
        url = f"{self.server_url}/ServicesAPI/API/V1/Session"
        try:
            self.session.delete(
                url,
                headers={
                    "Content-Type": "application/json",
                    "Accept": "application/json",
                    "token": self.token,
                },
                timeout=self.timeout,
            )
        except requests.RequestException:
            pass
        finally:
            self.token = None
            self._token_expiry = None

    def _headers(self) -> Dict[str, str]:
        self._ensure_token()               # transparently refresh expired OAuth tokens
        if not self.token:
            raise RuntimeError("No token. Call login_oauth() or login() first.")
        return {
            "Content-Type": "application/json",
            "Accept": "application/json",
            "token": self.token,
        }

    # ---- raw API call ----------------------------------------------------- #
    def get_adt_page(
        self,
        endpoint: str,
        pass_key: str,
        *,
        filter_devices: Optional[List[str]] = None,
        columns: Optional[List[str]] = None,
        row_filter: Optional[Dict[str, Any]] = None,
        auto_refresh: bool = False,
        page_size: int = 100,
        page_number: int = 1,
    ) -> Dict[str, Any]:
        """Call POST /TAF/Lite/adt/data once and return the raw JSON response."""
        url = f"{self.server_url}/ServicesAPI/API/V3/TAF/Lite/adt/data"
        option: Dict[str, Any] = {
            "autoRefresh": auto_refresh,
            "pageSize": page_size,
            "pageNumber": page_number,
        }
        if row_filter:
            option["rowFilter"] = row_filter

        body = {
            "endpoint": endpoint,
            "passKey": pass_key,
            "filterDevices": filter_devices or [],
            "columns": columns or [],
            "option": option,
        }
        resp = self.session.post(
            url, json=body, headers=self._headers(), timeout=self.timeout
        )
        resp.raise_for_status()
        data = resp.json()
        status = data.get("statusCode")
        if status not in (None, 790200):
            raise RuntimeError(
                f"ADT API error {status}: {data.get('statusDescription')}"
            )
        return data

    # ---- high level: fetch everything, normalized to dicts ---------------- #
    def get_adt_records(
        self,
        endpoint: str,
        pass_key: str,
        *,
        filter_devices: Optional[List[str]] = None,
        columns: Optional[List[str]] = None,
        row_filter: Optional[Dict[str, Any]] = None,
        auto_refresh: bool = False,
        page_size: int = 200,
        max_rows: Optional[int] = None,
    ) -> "ADTResult":
        """
        Page through the entire ADT and return an ADTResult containing the
        column metadata plus rows normalized to a list of {column: value} dicts.
        """
        col_meta: List[Dict[str, str]] = []
        records: List[Dict[str, Any]] = []
        adt_name = ""
        page = 1

        while True:
            data = self.get_adt_page(
                endpoint,
                pass_key,
                filter_devices=filter_devices,
                columns=columns,
                row_filter=row_filter,
                auto_refresh=auto_refresh and page == 1,  # refresh once, on page 1
                page_size=page_size,
                page_number=page,
            )
            adt_name = data.get("adtName", adt_name)
            page_cols = data.get("columns", [])
            if page_cols:
                col_meta = page_cols
            col_names = [c["name"] for c in col_meta]

            rows = data.get("rows", []) or []
            for raw_row in rows:
                records.append(_row_to_dict(col_names, raw_row))
                if max_rows is not None and len(records) >= max_rows:
                    return ADTResult(adt_name, col_meta, records[:max_rows])

            if len(rows) < page_size:        # last page reached
                break
            page += 1

        return ADTResult(adt_name, col_meta, records)


@dataclass
class ADTResult:
    """Normalized ADT data ready for external consumption."""

    name: str
    columns: List[Dict[str, str]]          # [{"name": ..., "type": ...}, ...]
    records: List[Dict[str, Any]]          # [{column_name: value, ...}, ...]

    @property
    def column_names(self) -> List[str]:
        return [c["name"] for c in self.columns]

    def to_json(self, path: str, indent: int = 2) -> None:
        with open(path, "w", encoding="utf-8") as fh:
            json.dump(
                {"name": self.name, "columns": self.columns, "records": self.records},
                fh, ensure_ascii=False, indent=indent,
            )

    def to_csv(self, path: str) -> None:
        names = self.column_names or sorted({k for r in self.records for k in r.keys()})
        with open(path, "w", newline="", encoding="utf-8-sig") as fh:
            writer = csv.DictWriter(fh, fieldnames=names, extrasaction="ignore")
            writer.writeheader()
            for rec in self.records:
                writer.writerow({k: _stringify(v) for k, v in rec.items() if k in names})

    def __len__(self) -> int:
        return len(self.records)


def _row_to_dict(col_names: List[str], raw_row: Iterable[Any]) -> Dict[str, Any]:
    """Convert one ADT row (a list of typed cell objects) into {column: value}."""
    return {name: _extract_cell(cell) for name, cell in zip(col_names, raw_row)}


def _extract_cell(cell: Any) -> Any:
    """Pull the human-meaningful value out of a typed ADT cell."""
    if isinstance(cell, dict):
        if "value" in cell:
            return cell["value"]
        if "name" in cell:          # Map / Site / Path objects
            return cell["name"]
        return cell                 # unknown shape -> keep raw
    return cell


def _stringify(value: Any) -> str:
    if value is None:
        return ""
    if isinstance(value, (list, dict)):
        return json.dumps(value, ensure_ascii=False)
    return str(value)


def parse_share_url(url: str) -> Dict[str, str]:
    """Extract t / d / id from an adtShare.html link, for reference only."""
    q = parse_qs(urlparse(url).query)
    return {
        "tenantId": (q.get("t") or [""])[0],
        "domainId": (q.get("d") or [""])[0],
        "adtId": (q.get("id") or [""])[0],
    }


# ---- Public interface -- called by the backbone script -------------------- #
def fetch_adt(
    *,
    server_url: str,
    adt_endpoint: str,
    adt_passkey: str,
    auth_method: str = "oauth",
    client_id: Optional[str] = None,
    client_secret: Optional[str] = None,
    username: Optional[str] = None,
    password: Optional[str] = None,
    verify_ssl: bool = True,
    filter_devices: Optional[List[str]] = None,
    columns: Optional[List[str]] = None,
    row_filter: Optional[Dict[str, Any]] = None,
    auto_refresh: bool = False,
    page_size: int = 200,
    max_rows: Optional[int] = None,
    token_lifetime_fallback: int = 3600,
    token_refresh_margin: int = 60,
    timeout: int = 60,
) -> ADTResult:
    """
    Authenticate to NetBrain, page through the configured ADT view and return
    an ADTResult. This is the single entry point used by the backbone script.

    auth_method:
        "oauth"    -> use client_id / client_secret (recommended).
        "userpass" -> legacy session login with username / password.
    """
    client = NetBrainADTClient(
        server_url=server_url,
        verify_ssl=verify_ssl,
        timeout=timeout,
        token_lifetime_fallback=token_lifetime_fallback,
        refresh_margin=token_refresh_margin,
    )

    method = (auth_method or "oauth").lower()
    used_session_login = False
    try:
        if method == "oauth":
            client.login_oauth(client_id or "", client_secret or "")
        elif method == "userpass":
            client.login(username or "", password or "")
            used_session_login = True
        else:
            raise RuntimeError(f"Unknown auth_method: {auth_method!r}")

        return client.get_adt_records(
            endpoint=adt_endpoint,
            pass_key=adt_passkey,
            filter_devices=filter_devices or None,
            columns=columns or None,
            row_filter=row_filter or None,
            auto_refresh=auto_refresh,
            page_size=page_size,
            max_rows=max_rows,
        )
    finally:
        if used_session_login:
            client.logout()
```

***`thirdparty_client.py`***
> This module is illustrative — every 3rd-party platform's connector API differs. It's built around the two-step pattern (exchange a long-lived API token for a short-lived access token, then upload a file to a named connector) that's common to many CMDB, vulnerability-management, and ITSM connector APIs, but the endpoint paths, field names, and response shape below are placeholders. Replace them with what your target platform's API documentation specifies.

```python
"""
thirdparty_client.py
=====================
3rd-party platform operations module for the NetBrain ADT sync.

Encapsulates everything needed to upload a CSV file to a 3rd-party platform
through its Connector / Report upload API. Designed to consume the CSV
produced by the NetBrain side (netbrain_adt_client.py).

This module holds NO configuration and NO entry point -- it is driven by the
backbone script (e.g. sync_backbone.py), which calls `upload_csv(...)`.
It can also be imported directly:

    from thirdparty_client import upload_csv, ThirdPartyClient

Two-step flow (illustrative -- confirm against your platform's own API docs)
-----------------------------------------------------------------------------
  1. Exchange a long-lived API token for a short-lived access token:
        POST {base}/authentication/token/
        Header: Authorization: <API_TOKEN>           (the key from the UI)
        Body:   {"name": ...}
        -> returns {"access_token": ..., "expires_in": ..., ...}

  2. Upload the CSV to a File/Report connector:
        POST {base}/report/upload/{connector_id}/
        Header: Authorization: Bearer <access_token>
        Body:   multipart/form-data with the CSV as the `file` part
        Optional form field: is_end_of_cycle (true=last file in sync cycle)

Where to get the values
-----------------------
  API_TOKEN     -> the 3rd-party platform's UI, typically under
                   Settings/Integrations > API Tokens (generate one).
  CONNECTOR_ID  -> create a file/report connector in the platform's UI; its id
                   appears in the connector URL / connector list. That connector
                   defines how CSV columns map to the platform's own records.
  BASE_URL      -> your tenant's API base URL, e.g. https://<tenant>.example.com/api/v1

Dependencies:  pip install requests
"""

from __future__ import annotations

import os
from typing import Optional

import requests


class ThirdPartyClient:
    """Thin client around a generic 3rd-party connector upload API."""

    def __init__(self, base_url: str, verify_ssl: bool = True, timeout: int = 120):
        self.base_url = base_url.rstrip("/")
        self.verify_ssl = verify_ssl
        self.timeout = timeout
        self.session = requests.Session()
        self.session.verify = verify_ssl
        self.access_token: Optional[str] = None
        if not verify_ssl:
            requests.packages.urllib3.disable_warnings()  # type: ignore[attr-defined]

    # ---- step 1: API token -> access token -------------------------------- #
    def authenticate(self, api_token: str, name: str = "netbrain-adt-upload") -> str:
        url = f"{self.base_url}/authentication/token/"
        resp = self.session.post(
            url,
            json={"name": name},
            headers={
                "Authorization": api_token,
                "Content-Type": "application/json",
                "Accept": "application/json",
            },
            timeout=self.timeout,
        )
        resp.raise_for_status()
        data = resp.json()
        token = data.get("access_token")
        if not token:
            raise RuntimeError(f"Token exchange failed: {data}")
        self.access_token = token
        return token

    # ---- step 2: upload the CSV ------------------------------------------- #
    def upload_report(
        self,
        connector_id: str,
        csv_path: str,
        *,
        is_end_of_cycle: bool = True,
        file_field: str = "file",
    ) -> dict:
        if not self.access_token:
            raise RuntimeError("Not authenticated. Call authenticate() first.")
        if not os.path.isfile(csv_path):
            raise FileNotFoundError(csv_path)

        url = f"{self.base_url}/report/upload/{connector_id}/"
        # NOTE: do NOT set Content-Type manually for multipart -- requests adds
        # the correct boundary when `files=` is used.
        headers = {
            "Authorization": f"Bearer {self.access_token}",
            "Accept": "application/json",
        }
        with open(csv_path, "rb") as fh:
            files = {file_field: (os.path.basename(csv_path), fh, "text/csv")}
            data = {"is_end_of_cycle": str(is_end_of_cycle).lower()}
            resp = self.session.post(
                url, headers=headers, files=files, data=data, timeout=self.timeout,
            )
        resp.raise_for_status()
        return resp.json() if resp.content else {}


# ---- Public interface -- called by the backbone script -------------------- #
def upload_csv(
    *,
    base_url: str,
    api_token: str,
    connector_id: str,
    csv_path: str,
    is_end_of_cycle: bool = True,
    file_field: str = "file",
    verify_ssl: bool = True,
    timeout: int = 120,
) -> dict:
    """
    Authenticate to the 3rd-party platform and upload `csv_path` to the
    configured connector. Returns the parsed JSON response. This is the single
    entry point used by the backbone script.
    """
    client = ThirdPartyClient(base_url, verify_ssl=verify_ssl, timeout=timeout)
    client.authenticate(api_token)
    return client.upload_report(
        connector_id, csv_path, is_end_of_cycle=is_end_of_cycle, file_field=file_field,
    )
```

***`sync_backbone.py`***
```python
"""
sync_backbone.py
=================
Backbone of the NetBrain -> 3rd-party platform ADT sync.

This is the only file you normally edit and run. It holds all configuration
(NetBrain + 3rd-party platform) and a single `main()` that drives the
end-to-end flow by calling the two operation modules:

    1. netbrain_adt_client.fetch_adt(...)  -> pull the ADT view via TAF Lite
    2. ADTResult.to_csv(...)               -> write the CSV (and optional JSON)
    3. thirdparty_client.upload_csv(...)   -> push the CSV to the 3rd-party connector

Run:
    pip install requests
    python sync_backbone.py

The heavy lifting lives in:
    netbrain_adt_client.py    -- all NetBrain operations
    thirdparty_client.py      -- all 3rd-party platform operations
This script only wires their interfaces together.
"""

from __future__ import annotations

import json
import sys

import netbrain_adt_client as netbrain
import thirdparty_client as thirdparty


# ===========================================================================
# CONFIGURATION -- edit these values, then run:  python sync_backbone.py
# ===========================================================================
class NetBrainConfig:
    # ---- NetBrain server -------------------------------------------------- #
    SERVER_URL = "https://<your-netbrain-host>"
    VERIFY_SSL = False           # NetBrain often uses a self-signed cert -> False

    # ---- Authentication --------------------------------------------------- #
    # AUTH_METHOD selects how the API token is obtained:
    #   "oauth"    -> OAuth 2.0 with CLIENT_ID + CLIENT_SECRET (recommended).
    #                 The token carries an expiry; it is auto-refreshed by
    #                 re-requesting with the client credentials when needed.
    #   "userpass" -> legacy session login with USERNAME + PASSWORD.
    AUTH_METHOD = "oauth"

    # -- oauth method (Open API > Add OAuth Client, then copy the values) --
    CLIENT_ID = ""
    CLIENT_SECRET = ""
    TOKEN_LIFETIME_FALLBACK = 3600      # used if the server response omits expires_in
    TOKEN_REFRESH_MARGIN = 60           # refresh this many seconds before expiry

    # -- userpass method --
    USERNAME = ""
    PASSWORD = ""

    # ---- TAF Lite ADT view credentials ------------------------------------ #
    # Generated by NetBrain when the ADT view is published to TAF Lite.
    # (NOT the t/d/id from the adtShare.html link -- see netbrain_adt_client.py docstring.)
    ADT_ENDPOINT = "T10001U"
    ADT_PASSKEY = "TAFLite!1234567890"

    # ---- Query options ---------------------------------------------------- #
    FILTER_DEVICES: list = []          # e.g. ["core-sw01"]; [] = all
    COLUMNS: list = []                 # e.g. ["Device", "CVE Number"]; [] = all columns
    ROW_FILTER: dict = {}              # e.g. {"Severity": "Critical"} (AND logic)
    AUTO_REFRESH = False               # rebuild the ADT before reading
    PAGE_SIZE = 200
    MAX_ROWS = None                    # cap total rows; None = no cap


class ThirdPartyConfig:
    # ---- 3rd-party platform tenant ----------------------------------------- #
    BASE_URL = "https://<3rd-party-host>/api/v1"
    VERIFY_SSL = True

    # ---- Authentication --------------------------------------------------- #
    # Long-lived API token generated in the 3rd-party platform's UI.
    API_TOKEN = ""

    # ---- Target connector ------------------------------------------------- #
    CONNECTOR_ID = ""            # the File/Report connector id from the UI
    # True  -> this is the last file in the sync cycle (platform closes the cycle).
    # False -> more files will follow before the cycle is processed.
    IS_END_OF_CYCLE = True
    # Multipart field name for the file part; override only if your connector
    # expects a different name.
    FILE_FIELD = "file"


class SyncConfig:
    # ---- Shared between the two stages ------------------------------------ #
    CSV_PATH = "adt_export.csv"        # CSV written by NetBrain, uploaded to the 3rd party
    OUT_JSON = None                    # set a path to also write JSON, for debugging
    PRINT_RECORDS = False              # also print fetched records to stdout
    # If the ADT comes back empty, skip the upload instead of pushing an empty
    # file (which would otherwise close a sync cycle with no data).
    SKIP_UPLOAD_IF_EMPTY = True
# ===========================================================================


def fetch_from_netbrain() -> "netbrain.ADTResult":
    """Stage 1: pull the ADT view from NetBrain via TAF Lite."""
    print("[1/3] Fetching ADT from NetBrain ...")
    result = netbrain.fetch_adt(
        server_url=NetBrainConfig.SERVER_URL,
        verify_ssl=NetBrainConfig.VERIFY_SSL,
        auth_method=NetBrainConfig.AUTH_METHOD,
        client_id=NetBrainConfig.CLIENT_ID,
        client_secret=NetBrainConfig.CLIENT_SECRET,
        username=NetBrainConfig.USERNAME,
        password=NetBrainConfig.PASSWORD,
        adt_endpoint=NetBrainConfig.ADT_ENDPOINT,
        adt_passkey=NetBrainConfig.ADT_PASSKEY,
        filter_devices=NetBrainConfig.FILTER_DEVICES,
        columns=NetBrainConfig.COLUMNS,
        row_filter=NetBrainConfig.ROW_FILTER,
        auto_refresh=NetBrainConfig.AUTO_REFRESH,
        page_size=NetBrainConfig.PAGE_SIZE,
        max_rows=NetBrainConfig.MAX_ROWS,
        token_lifetime_fallback=NetBrainConfig.TOKEN_LIFETIME_FALLBACK,
        token_refresh_margin=NetBrainConfig.TOKEN_REFRESH_MARGIN,
    )
    print(f"      ADT '{result.name}': {len(result)} rows, {len(result.columns)} columns")
    return result


def write_csv(result: "netbrain.ADTResult") -> None:
    """Stage 2: write the CSV (and optional JSON) that the 3rd party will consume."""
    print("[2/3] Writing CSV ...")
    result.to_csv(SyncConfig.CSV_PATH)
    print(f"      wrote {SyncConfig.CSV_PATH}")
    if SyncConfig.OUT_JSON:
        result.to_json(SyncConfig.OUT_JSON)
        print(f"      wrote {SyncConfig.OUT_JSON}")
    if SyncConfig.PRINT_RECORDS:
        print(json.dumps(result.records, ensure_ascii=False, indent=2))


def push_to_thirdparty() -> dict:
    """Stage 3: upload the CSV to the 3rd-party connector."""
    print("[3/3] Uploading CSV to 3rd-party platform ...")
    result = thirdparty.upload_csv(
        base_url=ThirdPartyConfig.BASE_URL,
        api_token=ThirdPartyConfig.API_TOKEN,
        connector_id=ThirdPartyConfig.CONNECTOR_ID,
        csv_path=SyncConfig.CSV_PATH,
        is_end_of_cycle=ThirdPartyConfig.IS_END_OF_CYCLE,
        file_field=ThirdPartyConfig.FILE_FIELD,
        verify_ssl=ThirdPartyConfig.VERIFY_SSL,
    )
    success = result.get("success_uploading", [])
    failed = result.get("failed_uploading", [])
    print(f"      Upload complete: {len(success)} succeeded, {len(failed)} failed")
    if failed:
        print("      failed:", failed)
    return result


def main() -> int:
    """End-to-end NetBrain -> 3rd-party platform sync."""
    result = fetch_from_netbrain()
    write_csv(result)

    if len(result) == 0 and SyncConfig.SKIP_UPLOAD_IF_EMPTY:
        print("ADT is empty -- skipping upload (SKIP_UPLOAD_IF_EMPTY=True).")
        return 0

    push_to_thirdparty()
    print("Done.")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

## Troubleshooting
| Symptom | Likely cause / fix |
|---|---|
| OAuth token request failed | Wrong `client_id`/`client_secret`, or OAuth is not enabled under System Management > Open API. |
| ADT API error 79xxxx | Bad `endpoint`/`passKey`, or the ADT view is no longer published to TAF Lite. |
| SSL certificate errors against NetBrain | Set `verify_ssl = False` for self-signed certificates. |
| 3rd-party token exchange failed | Invalid or expired `thirdparty_api_token` — regenerate it in the 3rd-party platform's settings. |
| `0 succeeded, N failed` from the 3rd-party platform | CSV columns don't match the connector's mapping — align the ADT view's columns with the connector configuration. |

## Reference:
> 1) Login API (OAuth 2.0 and legacy Session):<br>
> https://github.com/NetBrainAPI/NetBrain-REST-API-R12/blob/main/REST%20APIs%20Documentation/Authentication%20and%20Authorization/Login%20API.md

> 2) Get ADT Data API (TAF Lite):<br>
> https://github.com/NetBrainAPI/NetBrain-REST-API-R12/blob/main/REST%20APIs%20Documentation/TAF%20Lite%20%28Triggered%20Automation%20Framework%20Lite%29/4%20-%20Get%20ADT%20data.md

> 3) TAF Lite overview and adding an ADT view:<br>
> https://www.netbrain.com/docs/12ne3wy0aw/help/HTML/taf-lite.html<br>
> https://www.netbrain.com/docs/12ne3wy0aw/help/HTML/add-adt-view-to-taf-lite.html

> 4) 3rd-Party Platform Connector API — see your specific target platform's API documentation for the authoritative reference (API token generation, and the report/file connector docs).
