# TransitJSON Schemas

JSON Schema (draft 2020-12) definitions for **TransitJSON 0.4**, a JSON-based data format for describing public transit networks. TransitJSON is designed as a simpler, JSON-native alternative to GTFS, while keeping GTFS-compatible concepts (stops, routes, route patterns, trips, stop times, shapes, fares).

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
| `route_pattern.schema.json` | `route_pattern` | A concrete route variant such as a branch, short-turn, express or alternate alignment. |
| `pattern_stop.schema.json` | `pattern_stop` | Ordered stops belonging to one route pattern. |
| `trip.schema.json` | `trip` | A concrete scheduled trip (one service per weekday/service type). |
| `stop_time.schema.json` | `stop_time` | Departure times per trip/stop/sequence; `departure_time` is required for the first stop. |
| `shape.schema.json` | `shape` | Route geometry as an ordered array of `lat`/`lon` points (no encoded polylines). |
| `fare.schema.json` | `fare` | Fare definition: flat pricing, currency, payment methods, transfer rules. |
| `holiday.schema.json` | `holiday` | Official holidays and which weekday schedule they apply as (default: Sunday). |

## Realtime entities

> **Not: GTFS-Realtime gibi protobuf değildir** — doğrudan düz JSON dosyaları (`vehicles.json`, `announcements.json`) üzerinden tüketilir. Binary feed / `.pb` dosyası / protobuf kütüphanesi gerekmez.

Realtime schemas describe short-lived, point-in-time records that consumers poll regularly. Concepts are aligned with [GTFS-Realtime](https://gtfs.org/realtime/) (VehiclePosition → `vehicle` vb.) but the wire format is **plain JSON snapshots**, not protobuf.

| File | Entity | Description |
| --- | --- | --- |
| `vehicle.schema.json` | `vehicle` | Live vehicle position: coordinates, bearing, speed, license plate, trip/route link, stop approach status, occupancy (JSON only — no protobuf). |
| `announcement.schema.json` | `announcement` | Broadcast text for apps, websites, stop displays and station audio: informational notices as well as service disruptions (cancellations, delays, detours). |

Unlike static entities, realtime records carry absolute RFC 3339 timestamps in `updated_at` and are not archived — each update replaces the previous record with the same id. Tüketim için sadece HTTP üzerinden JSON çekmek yeterlidir.

## Conventions

- **IDs** (`*_id`) are strings, unique project-wide where noted.
- **Slugs** (`city`, `route`) are URL-safe, lowercase identifiers matching `^[a-z0-9]+(-[a-z0-9]+)*$`, used in API routes.
- **Direction codes** are used consistently across `route_pattern`, `trip`, `shape`, legacy `route_stop` and stop platforms:
  - `0` – single/unassigned direction
  - `1` – outbound
  - `2` – inbound
- Loop/ring geometry is represented by `route_pattern.is_loop`; a loop may still carry an outbound/inbound direction when the source system provides one.
- **Times** are local to the city timezone in `HH:MM:SS` format; values may exceed 24 hours (e.g. `25:30:00`) for trips crossing midnight.
- **Timestamps** use RFC 3339 (`format: "date-time"`); dates use `format: "date"`.
- **`updated_at`** is required in every entity.
- **`source`** records the origin of the data (open data portal URL, etc.).
- All schemas set `additionalProperties: false` to keep records strict.
- All property descriptions are in Turkish.

## Route patterns and 0.3 compatibility

- `route_patterns.json` is the canonical owner of a route variant; `pattern_stops.json` contains its stop order and `trips.pattern_id` selects it.
- Multiple patterns and shapes may exist under the same `route_id` + `direction`.
- `routes.route_pattern` is retained as a legacy route-level `round_trip` / `loop` classification. Exact loop state belongs to `route_patterns.is_loop`.
- `route_stops.json` is retained as a compatibility projection of the default pattern for older consumers. New consumers must use `pattern_stops.json`.
- Shapes are plain coordinate arrays; encoded polyline is not used.
- Fares are flat only: the same price applies from origin to destination on all routes.

## Validation

All schemas target [JSON Schema draft 2020-12](https://json-schema.org/draft/2020-12/schema) and can be used with any compliant validator. For example:

```bash
npx ajv -s stop.schema.json -d data/stops.json --strict=false
```

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
