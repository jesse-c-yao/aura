# App Document Model Design

Define Your App should not be merely a collection of explanatory documents. It
should be the single, validated data model for the app. Every downstream view,
including Visualize for Human, is a read-only projection of that model.

The model is organized into two layers:

```text
App document schemas   # Define the rules
        ↓
App instance documents # Describe a specific app
        ↓
Compiler               # Validate references and produce a model index
        ↓
Renderer / Inspecter   # Present the compiled model
```

Schemas define the shared contract and should not be duplicated inside every
app. The app instance contains only the data that describes that app.

## v0.1 Structure

```text
app-docs/
├── schemas/
│   └── v1/
│       ├── common.schema.json
│       ├── app.schema.json
│       ├── navigation.schema.json
│       ├── requirements.schema.json
│       ├── interaction.schema.json
│       ├── screen.schema.json
│       ├── domain-model.schema.json
│       ├── component.schema.json
│       ├── tokens.schema.json
│       ├── assets.schema.json
│       ├── content.schema.json
│       └── integration.schema.json
│
└── sample-app/
    ├── app.json
    ├── navigation.json
    ├── requirements.json
    ├── interactions.json
    │
    ├── screens/
    │   └── splash-screen.json
    │
    ├── models/
    │   ├── user.json
    │   └── session.json
    │
    ├── design/
    │   ├── tokens.json
    │   └── components.json
    │
    ├── assets/
    │   ├── catalog.json
    │   └── files/
    │       ├── splash-logo.png
    │       └── app-icon.png
    │
    ├── content/
    │   └── en-US.json
    │
    └── integrations/
        └── authentication.json
```

This is the starting structure. Split a document only when it grows large and
has a clear ownership boundary. For example, `interactions.json` may eventually
become an `interactions/` directory grouped by business domain. Each action
must still be defined exactly once.

## Document Responsibilities

| Document | Owns | Does not own |
| --- | --- | --- |
| `app.json` | App identity, version, entry screen, supported platforms, default locale, and model version | Screen contents, component definitions, or business rules |
| `navigation.json` | Global navigation containers, tabs, route groups, guards, and deep links | Screen layout or the complete behavior of an action |
| `requirements.json` | User roles, journeys, acceptance criteria, and non-functional requirements | UI implementation or component properties |
| `interactions.json` | Actions, conditions, business rules, side effects, and navigation targets | Field types, visual styles, or message content |
| `screens/<id>.json` | Screen composition, component instances, states, and event bindings | Token values, asset paths, API endpoints, or complete business rules |
| `models/<entity>.json` | Entities, fields, relationships, defaults, and field-level validation | UI layout, message content, or cross-entity workflows |
| `design/tokens.json` | Colors, typography, spacing, radii, shadows, and other design values | Component structure or screen layout |
| `design/components.json` | Reusable component structure, supported properties, and defaults | A specific component instance within a screen |
| `assets/catalog.json` | Asset IDs, relative paths, media types, hashes, dimensions, and sources | Where and how an asset appears in a screen |
| `content/<locale>.json` | User-facing message keys and their localized values | Screen structure or business logic |
| `integrations/<id>.json` | APIs, authentication, request and response contracts, error mappings, and external system boundaries | UI presentation or duplicate domain entity definitions |

## Stable IDs and References

The file location is not the source of truth. The stable ID is. Every
referenceable object has a globally unique, path-independent ID:

```text
app.sample
screen.splash
component.primary-button
token.color.surface.base
asset.splash.logo
message.screen.splash.title
action.splash.continue
entity.user
integration.authentication
```

Documents compose and reference these objects instead of redefining them. A
screen should contain references, not copies of tokens, messages, assets, or
API details:

```json
{
  "$schema": "../../schemas/v1/screen.schema.json",
  "id": "screen.splash",
  "route": "/splash",
  "title": {
    "textRef": "message.screen.splash.title"
  },
  "background": {
    "tokenRef": "token.color.surface.base"
  },
  "elements": [
    {
      "id": "element.splash.logo",
      "componentRef": "component.image",
      "assetRef": "asset.splash.logo"
    }
  ],
  "events": [
    {
      "event": "primaryAction",
      "actionRef": "action.splash.continue"
    }
  ]
}
```

Screens bind UI events to action IDs. `interactions.json` is the owner of action
behavior:

```json
{
  "actions": {
    "action.splash.continue": {
      "trigger": "screen.splash.primaryAction",
      "condition": "session.isAuthenticated",
      "effects": [
        {
          "type": "navigate",
          "screenRef": "screen.home"
        }
      ]
    }
  }
}
```

Each piece of information has exactly one owner:

- Field types and field-level validation belong in `models/`.
- Cross-field and workflow rules belong in `interactions.json`.
- User-visible error messages belong in `content/`.
- External request formats belong in `integrations/`.
- A screen references the relevant IDs rather than copying the definitions.

## Validation

The model is not complete until it has three validation layers:

1. **JSON Schema validation** checks document structure, field formats, and
   required properties.
2. **Semantic validation** checks duplicate IDs, unresolved references,
   duplicate routes, unreachable screens, invalid transitions, missing locale
   keys, missing assets, and invalid integration operations.
3. **Projection validation** checks the compiler output consumed by the
   renderer.

The compiler produces a generated app model, conceptually:

```text
build/app-model.json
```

This generated artifact is not hand-edited. The renderer consumes the compiled
model and must not independently combine rules from scattered documents.

The model therefore follows four invariants:

1. Schemas and app instances are separate.
2. Files are grouped by ownership, not by visual position in the UI.
3. Cross-document relationships use stable IDs.
4. Renderers and inspectors are projections; compilers and validators are the
   formal entry points to the canonical model.
