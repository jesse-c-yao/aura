# App Aura Design

## Purpose

App Aura is a read-only visualization and inspection environment for
applications defined by the Aura App Document Model.

It allows business stakeholders, product teams, designers, engineers, and QA to
experience what an application specification actually means before a production
implementation is complete.

App Aura does not own product requirements. The canonical app documents remain
the single source of truth. Aura loads those documents, validates them, compiles
them into a normalized runtime model, and presents that model through
role-specific views.

The App Document Model is described separately in
[App Document Model Design](app-docs/app-doc-design.md).

## Product Goals

1. Make complex application documents understandable without reading every raw
   file.
2. Let people experience complete user journeys, not just isolated screens.
3. Give different audiences the information and level of detail they need.
4. Expose inconsistencies and missing information early.
5. Ensure humans and AI work from the same canonical application definition.
6. Allow a conforming application to be visualized without app-specific code.

## Non-Goals

App Aura is not:

- A production application runtime.
- A replacement for the canonical app documents.
- A system that executes real payments, messages, account changes, or other
  consequential external operations.
- A fully faithful implementation of every platform-specific behavior in the
  initial version.
- An editor that silently changes source documents. Document authoring can be
  added later as a separate, explicit capability.

## Target Audiences

### Business Stakeholders

Need to understand the customer journey, visible behavior, content, and
business outcomes without navigating technical implementation details.

### Product and Design

Need to validate screen composition, interaction flows, state transitions, and
consistency between the intended experience and the documented model.

### Engineering and QA

Need to inspect stable IDs, component mappings, event definitions, locales,
validation rules, integrations, and the canonical data behind every visible
element.

## System Architecture

```text
Canonical app documents
        ↓
Document loader
        ↓
Schema and semantic validation
        ↓
Model compiler
        ↓
Normalized app model
        ↓
App Aura runtime
├── UI Journey mode
└── Inspector mode
    ├── Business view
    └── Technical view
```

### Document Loader

Loads an app instance and its referenced documents from the configured app
directory. The loader should not interpret business meaning; it only discovers
and reads canonical resources.

### Validator

Runs JSON Schema validation followed by semantic validation. Rendering should
not begin when required documents or references are invalid.

Diagnostics must identify the source document and, where possible, the exact
object or property responsible for the problem.

### Model Compiler

Resolves stable IDs and relationships into a normalized model. The compiler
produces the read-only data consumed by Aura, but never writes back to the
canonical documents.

The generated model is an artifact, not a second source of truth.

### Projection Engine

Transforms the normalized model into audience-specific projections:

- screen rendering data;
- journey navigation data;
- business-facing descriptions;
- technical inspection data;
- validation and integration summaries.

All projections must be traceable to canonical object IDs.

### Component Registry

Maps canonical component types to visual renderers and inspector descriptors.
Adding a new component requires:

1. a component definition in the schema and catalogs;
2. a renderer adapter;
3. an inspector descriptor;
4. representative tests for valid, invalid, and localized states.

Application documents may select and configure supported components, but they
must not inject arbitrary executable code into Aura.

### Aura Runtime

Maintains only ephemeral preview state, such as the current screen, selected
locale, journey history, and current inspector mode. Runtime state does not
modify canonical app documents unless a future authoring feature is explicitly
introduced.

## Experience Modes

### UI Journey Mode

UI Journey mode presents an executable simulation of the documented
application.

It supports:

- navigation between screens;
- local form input and validation states;
- component interactions such as taps, selections, and toggles;
- modal, loading, empty, error, and success states defined by the model;
- switching between supported locales;
- reviewing a complete user journey rather than a single screen.

Interactions with external systems must be simulated by default. If an action
would create, update, delete, send, pay, publish, or otherwise affect real
external state, Aura shows the expected behavior and clearly marks it as a
simulation.

UI Journey mode must never imply that a simulated action has occurred in the
real world.

### Inspector Mode

Inspector mode explains the model behind the current screen. It preserves the
same selected screen and journey context as UI Journey mode so users can move
between experiencing and inspecting without losing their place.

#### Business View

The business view should contain:

- the purpose of the current screen;
- user-facing labels, messages, and expected behavior;
- the current point in the user journey;
- business rules and visible outcomes;
- links to relevant requirements and acceptance criteria.

It should hide technical identifiers unless they are necessary to explain a
business rule.

#### Technical View

The technical view should expose:

- screen, component, and element IDs;
- component types and resolved properties;
- accessibility IDs and semantics;
- locale keys and resolved values;
- event bindings, actions, conditions, and transitions;
- references to models, integrations, assets, and design tokens;
- validation findings related to the current screen;
- source-document references for each inspected object.

Technical details should use progressive disclosure. The initial view should
show the most important facts, with deeper structures available on demand.

## Rendering Pipeline

1. Load the app manifest and referenced canonical documents.
2. Validate document structure and schema versions.
3. Validate IDs, routes, references, resources, and graph integrity.
4. Compile the documents into a normalized app model.
5. Build projections for the selected mode and audience.
6. Render the current screen using registered component adapters.
7. Update only ephemeral runtime state in response to user interaction.
8. Surface diagnostics whenever behavior cannot be represented faithfully.

## Missing or Unsupported Information

Aura must make uncertainty visible. It must not silently invent missing content.

When information is missing or unsupported, Aura should show:

- the source document and object ID;
- what the document requires;
- what Aura cannot resolve or render;
- whether the issue blocks rendering or only limits fidelity.

This behavior is essential to Aura's role as a validation tool.

## Security and Trust Boundaries

- Canonical documents are treated as untrusted data.
- Aura does not execute arbitrary code embedded in app documents.
- External operations are simulated unless a future sandbox explicitly permits
  a safe integration.
- Credentials and production secrets must never be required for visualization.
- Every generated projection is derived from validated canonical data.
- The interface must make preview and simulation boundaries visible.

## Extensibility

Aura should separate canonical data from platform-specific rendering through
adapters.

Potential adapters include:

- a mobile preview adapter;
- a web preview adapter;
- platform-specific component renderers;
- alternative inspector projections.

Adapters may change presentation, but they must not redefine application
business rules, content, assets, or data models.

## v0.1 Functional Scope

The first usable version should support:

- loading one valid application instance;
- schema and semantic diagnostics;
- rendering screens from the canonical model;
- navigating through a complete user journey;
- basic interaction and local state simulation;
- business and technical inspector views;
- locale switching;
- source references from inspected elements back to canonical object IDs;
- clearly marked simulated external actions.

## Acceptance Criteria

- A conforming app can be opened without application-specific rendering code.
- Invalid references block affected screens and produce actionable diagnostics.
- The same canonical object resolves consistently across all modes.
- UI Journey mode never performs a real external side effect.
- Business and technical views never become separate stores of product truth.
- Switching modes preserves the current screen and journey context.
- Every visible element can be traced to its canonical definition.

## Future Capabilities

Potential follow-up capabilities include:

- live document reload during authoring;
- visual annotations and review comments linked to object IDs;
- side-by-side version comparison;
- generated journey tests from acceptance criteria;
- visual regression snapshots;
- guided authoring that writes proposed changes back through an explicit review
  workflow.

These capabilities should extend the projection model without weakening the
single-source-of-truth boundary.
