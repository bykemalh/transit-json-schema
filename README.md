# TransitJSON Schemas

JSON Schema (draft 2020-12) definitions for **TransitJSON 0.6**, a JSON-based data format for describing public transit networks. TransitJSON is designed as a simpler, JSON-native alternative to GTFS, while keeping GTFS-compatible concepts (stops, routes, route patterns, trips, stop times, shapes, fares, transfer rules).

## Repository structure

Every file in this directory is a self-contained JSON Schema describing a single entity type. `meta.json` holds machine-readable metadata about the collection as a whole (format version, entity list, source).

| File | Entity | Description |
| --- | --- | --- |
| `country.schema.json` | `country` | Country: ISO 3166-1 alpha-2/alpha-3 codes, default language, supported languages, currency, continent, phone prefix, emoji flag. |
| `city.schema.json` | `city` | City: URL-safe slug, IANA timezone, map center, default zoom, population, region code, website, open data URL. |
| `agency.schema.json` | `agency` | Transit operator in a city (name, phone, website). |
| `stop.schema.json` | `stop` | Stop or station (v2): accessibility flags, physical amenities, optional per-platform details with pattern linkage, parent station for metro entrances/exits. |
| `route.schema.json` | `route` | Route: URL-safe slug, vehicle type, route pattern (round-trip / loop), stop mode (fixed / flexible), sort order, optional fare and color. |
| `route_pattern.schema.json` | `route_pattern` | A concrete route variant such as a branch, short-turn, express or alternate alignment. |
| `pattern_stop.schema.json` | `pattern_stop` | Ordered stops belonging to one route pattern. |
| `trip.schema.json` | `trip` | A concrete scheduled trip linked to a route pattern. Direction comes from the pattern. |
| `stop_time.schema.json` | `stop_time` | Departure times per trip/stop/sequence; `departure_time` is required for the first stop. |
| `shape.schema.json` | `shape` | Route geometry as an ordered array of `lat`/`lon` points (no encoded polylines). |
| `fare.schema.json` | `fare` | Fare definition: flat or stop-count-based pricing, tariff groups, per-route pricing, currency, payment methods, exit-validator refund (BursaRay "Gittiğin Kadar Öde" style). |
| `transfer_rule.schema.json` | `transfer_rule` | Transfer discount rules: discount-based (not price-based), tariff-group-aware, category-specific, directional route/vehicle filters, chain termination. |
| `calendar.schema.json` | `calendar` | Agency-based calendar exceptions: which weekday schedule (`applies_as`) runs on holidays/special days (default: Sunday). |

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
- **Route names** do not include the route code. The code is stored separately in the `code` field. Example: `name: "Emek - Arabayatağı"`, `code: "M1"`.
- **Direction codes** are used consistently across `route_pattern` and `shape`:
  - `0` – single/unassigned direction
  - `1` – outbound
  - `2` – inbound
- Platform direction is expressed through `pattern_ids` references rather than numeric codes.
- Trip direction comes from its linked `route_pattern`, not from the trip itself.
- Loop/ring geometry is represented by `route_pattern.is_loop`; a loop may still carry an outbound/inbound direction when the source system provides one.
- **Times** are local to the city timezone in `HH:MM:SS` format; values may exceed 24 hours (e.g. `25:30:00`) for trips crossing midnight.
- **Timestamps** use RFC 3339 (`format: "date-time"`); dates use `format: "date"`.
- **`updated_at`** is required in every entity.
- **`source`** is defined once in `meta.json`, not repeated in individual entity records.
- All schemas set `additionalProperties: false` to keep records strict.
- All property descriptions are in Turkish.

## Fare model

TransitJSON supports two fare types:

- **`flat`**: Fixed price from origin to destination on all applicable routes.
- **`stop_count`**: Tiered pricing based on the number of stops traveled (e.g. metro distance-based fares).

Each fare record belongs to a **tariff group** (`tariff_group`). Transfer discount amounts are calculated based on the origin fare's tariff group (see `transfer_rule.schema.json`).

**Refund mechanism**: For systems like BursaRay "Gittiğin Kadar Öde" (pay-as-you-go with exit-validator refund), the `refund` object within the fare schema defines stop-count-based refund tiers. This is independent from transfer discounts.

**Transfer rules**: Transfer discounts are defined as separate `transfer_rule` records. The formula is: `fare_paid = destination_fare - discount_amount`. Rules are specific to passenger categories and tariff groups. If no rule exists for a category, no transfer discount applies.

## Route patterns and 0.6 notes

- `route_patterns.json` is the canonical owner of a route variant; `pattern_stops.json` contains its stop order and `trips.pattern_id` selects it.
- Multiple patterns and shapes may exist under the same `route_id` + `direction`.
- `routes.route_pattern` is retained as a legacy route-level `round_trip` / `loop` classification. Exact loop state belongs to `route_patterns.is_loop`.
- `route_stops.json` was removed in 0.5. Use `pattern_stops.json`.
- `holiday.json` was removed in 0.5. Use agency-based `calendar.json` (`date + agency_id -> applies_as`).
- Shapes are plain coordinate arrays; encoded polyline is not used.
- `bounds` was removed from `city.schema.json` in 0.6. Use `center` for map positioning.
- `direction` was removed from `trip.schema.json` in 0.6. Direction comes from the linked `route_pattern`.
- Platform `direction` (0/1/2) was replaced with `pattern_ids` in 0.6.
- `source` was removed from all entity schemas in 0.6. Use `meta.json` for source information.
- `transfer_duration` and `transfer_limit` were removed from `fare.schema.json` in 0.6. Use `transfer_rule.schema.json`.
- `transfer_rule.schema.json` was added in 0.6 for discount-based transfer rules.
- `parent_station` was added to `stop.schema.json` in 0.6 for metro entrance/exit hierarchy.

## Validation

All schemas target [JSON Schema draft 2020-12](https://json-schema.org/draft/2020-12/schema) and can be used with any compliant validator. For example:

```bash
npx ajv -s stop.schema.json -d data/stops.json --strict=false
```

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
