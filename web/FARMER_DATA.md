# Farmer data provider

`farmer-data.js` exposes `FarmerData.createProvider(options)` to browser scripts. It implements the frontend mock for the contract in [`../FARMER_INPUT_CONTRACT.md`](../FARMER_INPUT_CONTRACT.md). No backend server or extra package is required for mock mode. The existing map viewer loads the provider, but its farmer-facing UI is separate work in issue #15.

```js
const data = FarmerData.createProvider({ mode: 'mock' });
const { parcels } = await data.findParcels({ number: 'DEMO-001' });
const matches = await data.findParcels({ lon: 20.76, lat: 52.175 });
const { crops } = await data.getCrops();
const { fields } = await data.listFields();
const field = await data.getField('field-demo-1');
field.crop_areas[0].sowing_date = '2024-09-16';
await data.saveField(field.field_id, field);
```

All methods return promises. `findParcels` accepts either `{number}` or `{lon, lat}`. `getField` and `saveField` accept a field ID; save accepts the complete contract field object as its second argument. To create a field on one of the two empty example plots, construct a valid field with a new ID, the selected parcel, and at least one complete crop area, then call `saveField(id, field)`. `past_sprays: []` means no sprays yet. Crop options and geometries come from the provider. Responses contain `data_status: 'mock'`.

There are five fictional adjacent parcels, `DEMO-001` through `DEMO-005`. The first three have saved fields `field-demo-1` through `field-demo-3`; the last two have no saved fields. The first field has two crop areas and a farmer crop/map crop disagreement. A point on the boundary between adjacent plots can return multiple parcel matches. These are synthetic examples, not GUGiK records or proof of ownership.

Changes are stored **in memory per provider instance** and disappear on page reload or when a new instance is created. They are not sent to a server, shared with QGIS, or durable. Returned data is copied so clients must call `saveField` to save changes. `data.setAvailable(false)` simulates a service failure; call `data.setAvailable(true)` to restore it. Errors are rejected promises with `status` and `error: {code, message, fields?}`. Empty search results return `{parcels: [], data_status: 'mock'}`. A missing field rejects with status 404; invalid input rejects with 400 or 422; simulated unavailability rejects with 503.

The mock validates required values, date format, positive spray durations, crop choices and basic closed GeoJSON polygon structure. It does not validate self-intersections, overlapping areas, containment in parcels, or offer nearby parcels; those are separate UI/spatial integration work. No land is silently claimed.

To switch providers when the real API is ready, keep the same method calls and configure:

```js
const data = FarmerData.createProvider({
  mode: 'backend',
  baseUrl: 'http://127.0.0.1:8000/api/v1'
});
```

Backend mode uses `fetch` and the contract's `/parcels`, `/crops`, and `/fields` paths. It does not fall back to mock data if the backend fails. The real endpoints do not yet exist or have not been verified; integration, CORS, authentication and persistence remain backend work. Callers must catch rejected promises and present a useful message.
