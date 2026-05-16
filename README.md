# OSM Field Tagger AI

> Proof of Concept idea for a mobile, human-verified OpenStreetMap field survey assistant using on-device AI.

## Status

**Idea / Proof of Concept**

This repository is intended to document and prototype a mobile application concept for updating OpenStreetMap from field observations.  
The first goal is not a production-ready editor, but a small working prototype that proves the workflow:

**geolocation → photo → local AI suggestion → human verification → OSM-ready tags**

## Working name

Suggested repository name:

**`osm-field-tagger-ai`**

Alternative names:

- `osm-ai-field-survey`
- `osm-photo-tagger`
- `osm-local-ai-tagger`
- `map-scout-ai`
- `fieldmapper-ai`

Recommended: **`osm-field-tagger-ai`**  
Reason: clear, searchable, practical, and directly describes the purpose without overpromising full automated mapping.

## Core idea

The application helps a user update OpenStreetMap by taking a photo of a real-world object and receiving suggested OSM tags.

Example:

A user photographs a bus shelter.  
The app analyzes the image locally and suggests tags such as:

```text
public_transport=platform
bus=yes
shelter=yes
shelter_type=public_transport
bench=yes
bin=no
lit=yes
```

The user then verifies:

- whether the suggested object type is correct,
- whether the proposed tags are correct,
- whether the map position is correct,
- whether the pin should be moved,
- whether the object should update an existing OSM feature or create a new one.

Only after human confirmation can data be prepared for upload to OpenStreetMap.

## Guiding principles

### Human in the loop

The app must never blindly upload AI-generated data.

The user always verifies and accepts:

- object type,
- OSM tags,
- attribute values,
- geometry/location,
- final upload action.

### On-device first

The first Proof of Concept should work locally on the device.

The photo should be used only temporarily for analysis and should not be uploaded anywhere in the local mode.

### Privacy by design

Default assumptions:

- photos are temporary,
- photos are not uploaded,
- photos are not stored permanently unless the user explicitly chooses that,
- logs should store only tag proposals and metadata, not images,
- sensitive elements such as faces or license plates should be considered for local redaction in later versions.

### OSM community compatibility

The app should support responsible OpenStreetMap editing.

Important assumptions:

- no automatic mass edits,
- no blind AI imports,
- every edit is based on field survey,
- every edit is human-verified,
- changeset comments should clearly state that the edit was field-survey based and AI-assisted.

Example changeset comment:

```text
Field survey; AI-assisted on-device tagging; human verified.
```

Possible source tag:

```text
source=survey
```

## Proposed workflow

1. **Geolocation**
   - The app reads the user’s current location.
   - A map view shows the current position and an editable pin.

2. **Photo capture**
   - The user takes a photo of an object.
   - The photo is used as temporary input for analysis.

3. **Local AI analysis**
   - A local model suggests the most likely object class.
   - A local preset catalogue maps object classes and attributes to OSM tags.
   - The first PoC can use a stub engine before real on-device AI is connected.

4. **Tag proposal**
   - The app presents suggested tags and confidence values.
   - The user can accept, remove, or edit tags.

5. **Position verification**
   - The user verifies the object location.
   - The pin can be moved manually on the map.

6. **OSM preparation**
   - The app prepares an OSM-ready object.
   - In early PoC versions this can be a dry-run JSON/XML preview.
   - Later versions may support OAuth2 and real upload to OSM.

## Local AI approach

The preferred local approach is:

**MobileCLIP / CLIP-style zero-shot classification + local OSM presets**

Instead of training a custom model at the beginning, the app can compare the image against text prompts such as:

- `bus shelter`
- `park bench`
- `waste basket`
- `bicycle rack`
- `hydrant`
- `information board`
- `bollard`

The best matching class is then mapped to OSM tags using a local `presets.json` file.

This allows the preset catalogue to grow over time without retraining the model.

## Optional future Turbo mode

A later version may support an optional cloud mode:

**Turbo mode: OpenAI-assisted tagging**

Rules for this mode:

- disabled by default,
- requires explicit user consent,
- requires the user’s own API key or configured backend,
- sends only a reduced/redacted image,
- falls back to local mode when offline or when the user has no key,
- still requires human verification before any OSM upload.

This is not part of the first PoC.

## First PoC scope

The first prototype should be intentionally small.

### Included

- iOS app prototype in Swift / SwiftUI
- photo selection or camera capture
- current location / map view
- editable pin
- local tag proposal engine
- simple preset catalogue
- review screen
- dry-run output of proposed OSM tags

### Not included initially

- real OSM upload
- OAuth2 login
- editing existing OSM objects
- ways / polygons
- relation handling
- advanced conflict detection
- cloud AI mode
- production-level privacy policy
- App Store distribution

## Initial object classes

Suggested first preset classes:

| Class ID | Object | Example OSM tags |
|---|---|---|
| `bus_shelter` | Bus shelter | `public_transport=platform`, `bus=yes`, `shelter=yes` |
| `bench` | Bench | `amenity=bench` |
| `waste_bin` | Waste basket | `amenity=waste_basket` |
| `bicycle_rack` | Bicycle parking | `amenity=bicycle_parking` |
| `information_board` | Information board | `tourism=information` |
| `bollard` | Bollard | `barrier=bollard` |
| `hydrant` | Fire hydrant | `emergency=fire_hydrant` |

The list should remain small at first and grow only after the workflow is tested.

## Example preset structure

```json
{
  "classes": {
    "bus_shelter": {
      "aliases": [
        "bus shelter",
        "public transport shelter",
        "covered bus stop"
      ],
      "tags": [
        {"k": "public_transport", "v": "platform"},
        {"k": "bus", "v": "yes"},
        {"k": "shelter", "v": "yes"},
        {"k": "shelter_type", "v": "public_transport"},
        {"k": "bench", "v": "{has_bench}"},
        {"k": "bin", "v": "{has_bin}"},
        {"k": "lit", "v": "{is_lit}"}
      ]
    }
  },
  "attributes": [
    "has_bench",
    "has_bin",
    "is_lit",
    "has_roof",
    "glass_panels",
    "advertising"
  ]
}
```

## Suggested architecture

```text
App
├── Views
│   ├── CaptureView
│   ├── ReviewView
│   └── ReviewMapView
├── Services
│   ├── TaggingEngine
│   ├── LocalCLIPEngine
│   ├── PresetMapper
│   └── OSMDryRunExporter
├── Models
│   ├── TagProposal
│   ├── OSMTag
│   └── Preset
└── Resources
    └── presets.json
```

## Key interfaces

### TaggingEngine

```swift
protocol TaggingEngine {
    func proposeTags(for image: CGImage) async throws -> TagProposal
}
```

### TagProposal

```swift
struct TagProposal: Codable {
    let class_id: String
    let confidence: Float
    var attributes: TagAttributes
    var proposed_tags: [OSMTag]
}
```

## Development approach

This project should be developed step by step.

Recommended order:

1. Create a minimal SwiftUI app.
2. Add photo selection.
3. Add a stub local tagging engine.
4. Add `presets.json`.
5. Add a review screen.
6. Add a map with movable pin.
7. Add dry-run export of OSM tags.
8. Replace the stub engine with a local CLIP/MobileCLIP engine.
9. Add detection of nearby existing OSM objects.
10. Add optional OSM OAuth2 upload only after the workflow is stable.

## Important design decision

The preset catalogue can be developed later.

The first priority is to prove the complete loop:

```text
photo → local suggestion → human review → corrected location → OSM-ready output
```

A small, reliable workflow is more important than a large preset list.

## Long-term possibilities

Potential future features:

- optional OpenAI Turbo mode,
- offline queue for field survey edits,
- comparison with nearby existing OSM objects,
- update existing nodes instead of creating new ones,
- support for ways and simple polygons,
- user-defined presets,
- preset packs by theme,
- quality scoring,
- local history of accepted edits,
- export to `.osm`, GeoJSON, or JOSM-compatible formats,
- integration ideas with existing mobile OSM editors.

## License

To be decided.

Possible options:

- MIT for code,
- separate documentation license,
- careful review if OSM-related preset data is copied from existing sources.

## Current status summary

This is currently an idea and Proof of Concept plan.

The next practical step is to create a minimal SwiftUI repository with:

- `README.md`,
- `LICENSE`,
- `.gitignore`,
- basic folder structure,
- a simple app skeleton,
- a stub local tagging engine,
- initial `presets.json`.

