---
name: Search Hydrosat STAC imagery
description: Authenticate and search the Hydrosat Discovery STAC API for thermal/VNIR satellite imagery over an area and time range, then retrieve item assets.
api: openapi/hydrosat-stac-openapi.json
operations:
  - Search_search_post
  - Search_search_get
  - Get_Collections_collections_get
  - Get_Item_collections__collection_id__items__item_id__get
---

# Search Hydrosat STAC imagery

Operating instructions for an agent using the Hydrosat Discovery STAC API
(`https://stac.hydrosat.com/`). The API is a standard SpatioTemporal Asset
Catalog (STAC) and is OAuth2-protected.

## 1. Authenticate
Every request needs a bearer token. Obtain one with the client-credentials grant
(see `authentication/hydrosat-authentication.yml`):

```
POST https://auth.hydrosat.com/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=<id>&client_secret=<secret>
```

The response gives `access_token` (valid `expires_in` = 3600s). Send
`Authorization: Bearer <access_token>` on every STAC call and refresh before
expiry. Create the `client_id`/`client_secret` in the Discovery Portal
(Account > API Clients, Org Admin role; max 4 clients per account).

## 2. Discover collections
Call `Get_Collections_collections_get` (`GET /collections`) to list available
data collections (e.g. `vz-l2`).

## 3. Search
Use `Search_search_post` (`POST /search`) with a JSON body, or
`Search_search_get` (`GET /search`) with query params. Filter with:
- `collections`: e.g. `["vz-l2"]`
- `bbox`: `[west, south, east, north]` or `intersects`: a GeoJSON geometry
- `datetime`: an ISO 8601 range
- `query`: property filters, e.g. `{"eo:cloud_cover": {"lte": 20}}`
- `sortby`: e.g. `+datetime`

Pagination follows STAC `next`/`prev` links; cap results with `limit`.

## 4. Retrieve an item and its assets
Call `Get_Item_collections__collection_id__items__item_id__get`
(`GET /collections/{collection_id}/items/{item_id}`) for a single scene. Read
its `assets` for downloadable imagery (L1A/L1B/L2 bands and QA/cloud masks).

## Conventions & errors
- Read-only catalog — no writes, no idempotency key needed.
- `401` means a missing/expired token — re-authenticate.
- `422` is a validation error (check bbox/datetime/query shape).
- See `conventions/hydrosat-conventions.yml` and `errors/hydrosat-problem-types.yml`.

> Tip: the official `pystac_client` library (`Client.open(...)`) handles paging
> and search ergonomics; see the Hydrosat `vz-tutorials` example repo.
