# TransitJSON Schemas

JSON Schema (draft 2020-12) definitions for **TransitJSON**, a JSON-based data format for describing public transit networks. TransitJSON is designed as a simpler, JSON-native alternative to GTFS, while keeping GTFS-compatible concepts (stops, routes, trips, stop times, shapes, fares).

## Repository structure

Every file in this directory is a self-contained JSON Schema describing a single entity type. `meta.json` holds machine-readable metadata about the collection as a whole (format version, entity list).

| File | Entity | Description |
| --- | --- | --- |
| `country.schema.json` | `country` | Country (ISO 3166-1 alpha-2 code recommended, e.g. `TR`). |
| `city.schema.json` | `city` | City: URL-safe slug, IANA timezone, map center, bounds, default zoom. |
| `agency.schema.json` | `agency` | Transit operator in a city (name, phone, website). |
| `stop.schema.json` | `stop` | Stop or station (v2): accessibility flags, physical amenities, optional per-platform details. |
| `route.schema.json` | `route` | Route: URL-safe slug, vehicle type, route pattern (round-trip / loop), stop mode (fixed / flexible), optional fare and color. |
| `route_stop.schema.json` | `route_stop` | Many-to-many link between routes and stops, ordered per direction and sequence. |
| `trip.schema.json` | `trip` | A concrete scheduled trip (one service per weekday/service type). |
| `stop_time.schema.json` | `stop_time` | Departure times per trip/stop/sequence; `departure_time` is required for the first stop. |
| `shape.schema.json` | `shape` | Route geometry as an ordered array of `lat`/`lon` points (no encoded polylines). |
| `fare.schema.json` | `fare` | Fare definition: flat pricing, currency, payment methods, transfer rules. |
| `holiday.schema.json` | `holiday` | Official holidays and which weekday schedule they apply as (default: Sunday). |

## Realtime entities

Realtime schemas describe short-lived, point-in-time records that consumers poll regularly. They are conceptually aligned with [GTFS-Realtime](https://gtfs.org/realtime/) but use plain JSON snapshots instead of protobuf feeds.

| File | Entity | Description |
| --- | --- | --- |
| `vehicle.schema.json` | `vehicle` | Live vehicle position: coordinates, bearing, speed, license plate, trip/route link, stop approach status, occupancy (GTFS-RT VehiclePosition analog). |
| `announcement.schema.json` | `announcement` | Broadcast text for apps, websites, stop displays and station audio: informational notices as well as service disruptions (cancellations, delays, detours). |

Unlike static entities, realtime records carry absolute RFC 3339 timestamps in `updated_at` and are not archived — each update replaces the previous record with the same id.

## Conventions

- **IDs** (`*_id`) are strings, unique project-wide where noted.
- **Slugs** (`city`, `route`) are URL-safe, lowercase identifiers matching `^[a-z0-9]+(-[a-z0-9]+)*$`, used in API routes.
- **Direction codes** are used consistently across `trip`, `shape`, `route_stop` and stop platforms:
  - `0` – loop (one-way ring line)
  - `1` – outbound
  - `2` – inbound
- **Times** are local to the city timezone in `HH:MM:SS` format; values may exceed 24 hours (e.g. `25:30:00`) for trips crossing midnight.
- **Timestamps** use RFC 3339 (`format: "date-time"`); dates use `format: "date"`.
- **`updated_at`** is required in every entity.
- **`source`** records the origin of the data (open data portal URL, etc.).
- All schemas set `additionalProperties: false` to keep records strict.
- All property descriptions are in Turkish.

## v1 scope limits

- Each `route_id` + `direction` has exactly one shape and one ordered stop sequence. Trips do not link to per-trip shape/route_stop variants; short-turns, branches and variant routes are **not** supported.
- Shapes are plain coordinate arrays; encoded polyline is not used.
- Fares are flat only: the same price applies from origin to destination on all routes.

## Validation

All schemas target [JSON Schema draft 2020-12](https://json-schema.org/draft/2020-12/schema) and can be used with any compliant validator. For example:

```bash
npx ajv -s stop.schema.json -d data/stops.json --strict=false
```