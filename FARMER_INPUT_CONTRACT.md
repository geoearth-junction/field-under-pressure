# Farmer input API contract — issue #11

This document records the agreed input direction and an example contract. It is not an implemented API. No example represents a real parcel or person.

## Intended flow

1. The farmer can centre the map on device location. This is a client action; it does not select a parcel or prove ownership.
2. The client obtains parcel lookup results from the backend by parcel identifier or map coordinates and displays their geometry. The farmer selects a result as their working field.
3. The backend supplies crop choices. The farmer divides or selects one or more crop-area polygons and enters the crop, sowing dateand previous sprays for each. A field may have multiple areas.
4. MapLibre saves the data via the backend. QGIS reads the same backend representation for spatial inspection. The crop reported by a farmer is distinguishable from any crop-map value.
5. A mock supplies example responses and accepts and returns submitted data. It must clearly identify mock data and document its storage lifetime.

## Proposed field representation

The example uses GeoJSON geometry in longitude/latitude and ISO calendar dates. Empty lists mean no records were supplied, **not** that sprays are known never to have occurred. `null` means unknown or not supplied. Dates describe calendar days, not timestamps.

```json
{
  "field_id": "field-demo-1",
  "parcel": {
    "parcel_id": "parcel-demo-1",
    "parcel_number": "DEMO-123",
    "geometry": {
      "type": "Polygon",
      "coordinates": [[[20.750, 52.180], [20.760, 52.180], [20.760, 52.170], [20.750, 52.170], [20.750, 52.180]]]
    }
  },
  "crop_areas": [
    {
      "crop_area_id": "area-demo-1",
      "geometry": {
        "type": "Polygon",
        "coordinates": [[[20.750, 52.180], [20.755, 52.180], [20.755, 52.170], [20.750, 52.170], [20.750, 52.180]]]
      },
      "crop": {"farmer_crop_id": "wheat", "map_crop_id": "maize"},
      "sowing_date": "2024-09-15",
      "past_sprays": [{"date": "2024-10-10"}]
    },
    {
      "crop_area_id": "area-demo-2",
      "geometry": {
        "type": "Polygon",
        "coordinates": [[[20.755, 52.180], [20.760, 52.180], [20.760, 52.170], [20.755, 52.170], [20.755, 52.180]]]
      },
      "crop": {"farmer_crop_id": "soy", "map_crop_id": null},
      "sowing_date": null,
      "past_sprays": []
    }
  ],
  "data_status": "mock"
}
```

The identifiers and crop codes above are illustrative; the backend must provide actual available choices. The farmer value and map value are separate so an override does not erase provenance. A parcel selection denotes a working association, not legal ownership.

## Contract operations to agree

| Need | Proposed behaviour | Example result |
| --- | --- | --- |
| Parcel by number | Client sends a parcel identifier; backend returns matching parcels and geometries. | A list, which may be empty or contain multiple matches. |
| Parcel by click | Client sends map coordinates; backend returns matching parcels and geometries. | A list, allowing no match or an ambiguous match. |
| Crop choices | Client requests current backend-provided choices. | IDs and display labels for selection. |
| Save field | Client sends selected parcel, crop areas and entered data. | Saved field and stable identifiers, or field-level validation errors. |
| Read field | QGIS or MapLibre requests the saved field. | The same field and crop-area geometries and attributes. |

Exact endpoints, request and response envelopes, identifiers, update semantics and mock persistence are intentionally open for the team to decide. Both clients need to see the same saved data; a process-local mock may reset on restart, which must be disclosed.

## Implementation questions

- What parcel identifier and lookup inputs does the real cadastral service accept, and how should multiple matches and service failures be returned?
- Should the user draw crop-area boundaries, select existing geometry, or use both? How should geometry validity, overlap, containment and coordinate reference systems be checked?
- Which fields are required to save a field, and how do we distinguish unknown, not applicable and an explicitly empty spray history?
- Does a spray need product, amount or other information now, or only its date? Can multiple sprays occur on the same day?
- How should farmer corrections and source-map values carry provenance, timestamps and history?
- What access and isolation rules associate saved fields with users? What mock behaviour is appropriate before those rules are agreed?
- Which backend-provided options and errors must the farmer-facing flow show, and what does QGIS need to retrieve?

## Acceptance walkthrough after agreement

Using the same backend mock, select a sample parcel by number and by map click, create two crop-area polygons, save wheat for one and soy for the other, enter a full sowing date and past spray date. Retrieve the saved result from both client workflows and compare identifiers, polygons and attributes. Check missing inputs, no parcel match, ambiguous results, invalid dates, and backend unavailability. Record what is mocked and not yet implemented. Locate-me should only centre the map.
