# Farmer input API contract — issue #11

This document records the agreed input direction and an example contract. It is not an implemented API. No example represents a real parcel or person.

## Intended flow

1. The farmer can centre the map on device location. This is a client action; it does not select a parcel or prove ownership.
2. The client obtains parcel lookup results from the backend by parcel identifier or map coordinates and displays their geometry. The farmer can select multiple results as their working plots.
3. The backend supplies crop choices. The farmer draws one or more crop-area polygons on top of each selected GUGiK parcel and enters the crop, sowing date, and past spray date and protection duration in days for each crop area. Each plot may have multiple crop areas. All required input must be supplied before saving; the spray list can be empty when no sprays have been carried out. Saved data remains until updated.
4. MapLibre saves the data via the backend. QGIS reads the same backend representation for spatial inspection. The crop reported by a farmer is distinguishable from any crop-map value.
5. The switchable frontend mock supplies five example plots, some with saved data and some empty, and accepts and returns submitted data. The frontend will switch to the real backend once endpoints are ready. QGIS can use the same mock examples; frontend-local changes are not shared with QGIS without an agreed exchange. Clearly identify mock data and document its storage lifetime.

## Proposed field representation

The example uses GeoJSON geometry in longitude/latitude and ISO calendar dates. The examples show values in a saved record and an unfilled example. All required inputs must be supplied before a record is saved. An empty spray list is allowed when no sprays have been carried out. Dates describe calendar days, not timestamps.

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
      "past_sprays": [{"date": "2024-10-10", "protection_duration_days": 14}]
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

The second crop area illustrates an unfilled form, not a valid saved record. The example protection duration is illustrative and expressed in days. The identifiers and crop codes above are illustrative; the backend must provide actual available choices. The farmer-selected crop is used for the crop area; the farmer value and map value are kept separate so the map value is not erased. A parcel selection denotes a working association, not legal ownership.

## Contract operations to agree

| Need | Proposed behaviour | Example result |
| --- | --- | --- |
| Parcel by number | Client sends a parcel identifier; backend returns matching parcels and geometries. | A list, which may be empty or contain multiple matches. |
| Parcel by click | Client sends map coordinates; backend returns matching parcels and geometries. | A list, allowing no match or an ambiguous match. |
| Crop choices | Client requests current backend-provided choices. | IDs and display labels for selection. |
| Save field | Client sends selected parcel, crop areas and entered data. | Saved field and stable identifiers, or field-level validation errors. |
| Read field | QGIS or MapLibre requests the saved field. | The same field and crop-area geometries and attributes. |

Exact endpoints, request and response envelopes, identifiers, update semantics and mock persistence are intentionally open for the team to decide. The frontend mock is local to the frontend and is not a shared backend; its storage lifetime must be disclosed. QGIS and MapLibre can use the same example contract, but sharing saved data between them requires a backend or a separately agreed exchange.

## Implementation questions

- What parcel identifier and lookup inputs does the real cadastral service accept, and how should multiple matches and service failures be returned?
- How should drawn crop-area geometry validity, overlap, containment within the selected parcel and coordinate reference systems be checked?
- Can multiple sprays occur on the same day? Each recorded spray requires its date and protection duration in days.
- What metadata should accompany farmer corrections and source-map values beyond retaining both values?
- What access and isolation rules associate saved fields with users? What mock behaviour is appropriate before those rules are agreed?
- Which backend-provided options and errors must the farmer-facing flow show, and what does QGIS need to retrieve?

## Acceptance walkthrough after agreement

Using five frontend mock plots (some saved, some empty), select more than one sample parcel by number and by map click, draw crop-area polygons inside them, enter crops and full sowing dates, and record spray dates and protection durations. Retrieve the result in the frontend and compare it with the shared example contract used for QGIS. Verify shared saved data across both clients once the backend is available. Check missing inputs, no parcel match, ambiguous results, invalid dates, and backend unavailability. Record what is mocked and not yet implemented. Locate-me should only centre the map.
