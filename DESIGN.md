# Disaster Risk Intelligence Platform — Frontend Design System

## Purpose

This document is the single source of truth for the frontend visual language and interaction design of the Disaster Risk Intelligence Platform.

Every page, component, dashboard, map, form, table, modal, notification, mobile screen, and future feature must follow these rules unless a deliberate product decision explicitly overrides them.

The goal is to produce a **professional, operational, trustworthy, map-first platform** for government authorities, field teams, and citizens.

The UI must feel like:

- a serious emergency/disaster operations product
- a modern GIS application
- an enterprise decision-support system
- calm and information-dense
- consistent across desktop, tablet, and mobile
- multilingual by design
- accessible and usable under stressful field conditions

It must **not** feel like a generic AI-generated SaaS dashboard.

---

# 1. Core Design Principles

## 1.1 Complexity belongs in the system, not the interface

The underlying workflow is complex: ingestion, GIS processing, risk engines, satellite analysis, field evidence, impact analysis, prioritization, response, alerts, and feedback loops.

Users should not have to understand that architecture to use the product.

Every page should answer:

1. What is happening?
2. Where is it happening?
3. How serious is it?
4. Why is it happening?
5. What needs action?

## 1.2 Map-first intelligence

Spatial information is central to the platform.

Maps should be treated as a primary application surface, not a decorative widget.

Risk, incidents, roads, villages, hospitals, schools, rainfall, satellite changes, and field reports should connect visually to geographic context whenever possible.

## 1.3 Operational clarity over decoration

Prefer:

- strong hierarchy
- restrained surfaces
- compact information blocks
- clear status indicators
- readable tables
- useful maps
- obvious actions

Avoid:

- excessive gradients
- glassmorphism
- oversized illustrations
- ornamental blobs
- excessive shadows
- floating cards for every section
- unnecessary animations
- decorative AI imagery
- giant dashboard numbers with little context

## 1.4 Calm visual system

The interface itself should be visually calm.

Risk colors are reserved for semantic meaning. They should not become the general brand palette.

The default application should mainly use navy/blue, white, slate neutrals, and a restrained natural green/teal secondary tone.

## 1.5 Actionable information

Every important data visualization should allow the user to move from:

**signal → context → evidence → action**

Example:

`High Risk` → `Village A` → `78% probability, heavy rainfall + slope movement` → `View incident / dispatch team`

## 1.6 Trust through explainability

Risk outputs must never feel like unexplained AI scores.

When showing a risk result, expose where appropriate:

- probability
- severity
- confidence
- last updated time
- evidence
- contributing factors
- affected assets
- recommended action

## 1.7 Role-based simplicity

Different roles should see different amounts of information.

Government admins require broad situational awareness.

Field teams require fast action and mobile usability.

Citizens require simple local-risk information, reporting, and assistance.

Do not expose admin complexity to citizens.

---

# 2. Product Roles

## Government / Administrator

Primary experience:

- command-center dashboard
- risk map
- data monitoring
- incident management
- alerts
- infrastructure impact
- response coordination
- analytics
- reports

Typical layout:

`sidebar + top bar + dense desktop workspace`

## Field Officer / Field Team

Primary experience:

- assigned incidents
- nearby risk
- navigation/map
- incident verification
- evidence capture
- status updates
- alerts
- offline operation

Typical layout:

`mobile/tablet first + large touch targets`

## Citizen / Public User

Primary experience:

- current/local risk
- report incident
- upload photo/video
- receive alerts
- ask assistant
- view nearby information

Typical layout:

`mobile first + very clear primary actions`

---

# 3. Brand Character

The product should communicate:

- trustworthy
- civic
- intelligent
- precise
- resilient
- calm
- modern
- practical

It should NOT communicate:

- playful
- futuristic-for-the-sake-of-futuristic
- gaming
- crypto
- generic AI startup
- consumer social media

The visual tone should be closer to **professional GIS + emergency operations + enterprise software**.

---

# 4. Color System

## 4.1 Brand palette

### Primary Navy

```text
Primary 900: #0F2747
Primary 800: #163A63
Primary 700: #1D4F85
Primary 600: #2563A8
Primary 500: #2F73C9
Primary 100: #E8F1FB
Primary 50:  #F4F8FC
```

Use Primary 900/800 for navigation and important headings. Use Primary 600/500 for primary interactive controls.

### Secondary Natural Teal/Green

```text
Secondary 800: #0F5B56
Secondary 700: #11736C
Secondary 600: #158A7D
Secondary 500: #20A18F
Secondary 100: #DFF4F0
Secondary 50:  #F1FAF8
```

Use the secondary palette for environmental context, healthy/normal states, supporting highlights, and secondary product identity.

### Neutral palette

```text
Neutral 950: #0B1220
Neutral 900: #172033
Neutral 800: #263247
Neutral 700: #3B4759
Neutral 600: #596577
Neutral 500: #727E8F
Neutral 400: #9AA4B2
Neutral 300: #CBD2DB
Neutral 200: #E2E7ED
Neutral 100: #F0F3F6
Neutral 50:  #F7F9FB
White:       #FFFFFF
```

Most pages should use Neutral 50 / white as the background system.

## 4.2 Semantic status colors

These colors communicate state, not branding.

### Success

```text
Success 700: #157347
Success 600: #198754
Success 100: #DDF4E6
```

### Info

```text
Info 700: #155E9D
Info 600: #2F80C9
Info 100: #E1F0FB
```

### Warning

```text
Warning 700: #8A5A00
Warning 600: #D99A14
Warning 100: #FFF2D6
```

### Danger

```text
Danger 700: #A61B1B
Danger 600: #D92D20
Danger 100: #FCE3E1
```

## 4.3 Risk scale

Risk status must always use the same semantic scale across the entire product.

```text
LOW       #159447
MODERATE  #D9A441
HIGH      #E57A17
CRITICAL  #C92A2A
```

Recommended lighter backgrounds:

```text
Low       #E4F5EA
Moderate  #FFF3D9
High      #FCE8D3
Critical  #FBE1E1
```

Rules:

- Do not invent new risk colors per page.
- Do not use purple/pink for risk.
- Do not rely on color alone; always pair with a text label, icon, or pattern.
- Critical states may use stronger emphasis and motion, but should not flash continuously.

## 4.4 Risk legend

Use a consistent legend:

`Low → Moderate → High → Critical`

Use circular map markers or compact chips, not large decorative badges.

---

# 5. Color Usage Rules

Do not use every brand color on every page.

A typical page should contain:

- 1 dominant neutral background
- 1 primary brand color
- 1 supporting secondary color at most
- semantic colors only where needed

Example dashboard:

- background: Neutral 50
- cards: White
- navigation: Primary 900
- primary buttons: Primary 600
- secondary accents: Secondary 600
- alerts: semantic/risk colors

Never create a dashboard containing equal amounts of purple, blue, green, orange, red, and teal.

---

# 6. Typography

## Font

Preferred UI font:

**Inter**

Use a local/system fallback stack:

```css
font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
```

The font must support the languages used by the product. Verify glyph coverage for each enabled locale.

## Type scale

```text
Display:      32px / 40px / 700
Page title:   24px / 32px / 700
Section:      18px / 26px / 650
Subtitle:     16px / 24px / 600
Body:         14px / 22px / 400
Body strong:  14px / 22px / 600
Small:        13px / 20px / 400
Caption:      12px / 18px / 400
Micro:        11px / 16px / 600
```

Do not create dozens of font sizes.

Use weight to establish hierarchy before introducing a new size.

---

# 7. Spacing System

Use a 4px base spacing system.

```text
4   → micro gap
8   → compact gap
12  → standard control gap
16  → default component spacing
24  → section spacing
32  → major section spacing
40  → large separation
48  → page section spacing
64  → major page separation
```

Use consistent spacing across components.

Do not manually invent values such as 13px, 19px, 27px, 37px unless technically necessary.

---

# 8. Radius System

Keep corners professional and restrained.

```text
XS: 4px
SM: 6px
MD: 8px
LG: 12px
XL: 16px
```

Defaults:

- buttons: 6–8px
- inputs: 6–8px
- cards: 10–12px
- dialogs: 12–16px
- full-screen sheets: 16px top corners where applicable

Avoid excessive pill-shaped UI.

Pills are mainly for statuses, tags, filters, and compact metadata.

---

# 9. Borders and Shadows

## Borders

Default border:

```text
#E2E7ED
```

Strong border:

```text
#CBD2DB
```

Borders should be subtle and consistent.

## Shadows

Use shadows sparingly.

```text
Shadow XS: 0 1px 2px rgba(15, 39, 71, 0.05)
Shadow SM: 0 2px 8px rgba(15, 39, 71, 0.08)
Shadow MD: 0 8px 24px rgba(15, 39, 71, 0.10)
```

Most cards should be separated using border + background rather than a large shadow.

---

# 10. Icons

## Required icon strategy

**Use icons, never generic emojis, for product UI.**

Preferred icon library:

**Lucide Icons**

Use Lucide consistently throughout the product.

Do not mix random icon packs unless a specific map/GIS library requires a specialized symbol.

## Icon rules

- default size: 18–20px
- compact metadata: 14–16px
- primary action: 18–20px
- large empty-state icon: 32–48px
- mobile primary action: 20–24px

Use stroke icons with consistent visual weight.

Do not use emojis such as `🚨`, `🌧️`, `📍`, `🤖`, `🔥`, or `✅` as UI icons.

Use semantic Lucide icons instead:

```text
AlertTriangle
Map
MapPinned
CloudRain
Satellite
RadioTower
Smartphone
Upload
Camera
Video
MessageCircle
Bot
LocateFixed
WifiOff
RefreshCw
Bell
ShieldCheck
TriangleAlert
Route
Hospital
School
Building2
Users
FileText
BarChart3
ClipboardCheck
Send
Search
Filter
Settings
```

Icon choice must communicate the same concept across the whole application.

Example: use the same map-pin icon family for geographic locations everywhere.

---

# 11. Icon + Text Rules

For important actions, use:

`[Icon] Label`

not icon-only unless the action is universally recognizable and has an accessible tooltip.

Examples:

- `[Plus] Add Incident`
- `[MapPinned] View Risk Map`
- `[Bell] Notifications`
- `[Download] Export Report`
- `[Navigation] Navigate`

Icon-only controls must have:

- accessible label
- tooltip on desktop where useful
- sufficient touch target

---

# 12. Application Shell

The desktop application uses a persistent left navigation rail/sidebar.

## Sidebar

Default width:

```text
Expanded: 240–256px
Collapsed: 72px
```

Background:

`Primary 900`

Sidebar should contain:

```text
Brand / Logo

Overview
Risk Map
Monitoring
Incidents
Infrastructure
Alerts
Response
Reports

---
Settings
Help / Support
User profile
```

Use a consistent active state:

- slightly lighter navy background
- thin primary/secondary accent where useful
- white icon
- white/bolder text

Do not create a completely different visual treatment for every menu item.

## Top bar

The top bar contains:

- breadcrumb/page title
- global search where appropriate
- language selector
- notifications
- user profile

Height:

`64–72px`

It should remain visually quiet compared with the actual workspace.

---

# 13. Page Layout

Standard desktop content layout:

```text
Page shell
├── Sidebar
├── Top bar
└── Main content
    ├── Page header
    ├── Filters/actions
    ├── Primary content
    └── Supporting content
```

Page content max width:

Use a flexible layout with a practical maximum of roughly `1440–1600px` where useful.

Do not force narrow content into unnecessary cards.

Use a 12-column grid for complex desktop screens.

Typical gutters:

```text
Desktop: 24–32px
Tablet: 20–24px
Mobile: 16px
```

---

# 14. Dashboard Design

The government overview dashboard should prioritize situational awareness.

Recommended hierarchy:

1. page context
2. critical KPIs
3. live risk map
4. critical alerts / active incidents
5. trends
6. recent activity
7. secondary analytics

## KPI cards

Use compact, information-rich cards.

Example:

```text
12
Active Alerts
+2 since last update
```

A KPI card may include:

- icon
- value
- label
- small trend
- status

Avoid giant numbers taking up half the screen.

---

# 15. Cards

Cards are containers for meaningful information, not the default wrapper for every element.

Use cards for:

- summary metrics
- alerts
- grouped analytical content
- selected location details
- action panels

Do not wrap every heading, button, and paragraph in a separate card.

Default:

```text
background: white
border: 1px solid Neutral 200
radius: 10–12px
shadow: none or XS
padding: 16–20px
```

---

# 16. Risk Map Design

The map page is a primary product surface.

## Map structure

```text
Map workspace
├── Search
├── Location/region selector
├── Layer controls
├── Map
├── Legend
├── Selected feature panel
└── Map actions
```

Recommended layers:

- Risk heatmap
- Rainfall
- Satellite
- Roads
- Villages
- Hospitals
- Schools
- Sensors
- Incident locations
- Field teams

Do not show every layer by default.

Use a clear layer panel with checkboxes/toggles.

## Selected location panel

A selected village/road/incident should show:

```text
Village A

HIGH RISK

Probability    78%
Confidence     91%
Updated        18 min ago

Contributing factors
• Heavy rainfall
• Slope movement
• Satellite surface change

[View Details]
[Create Action]
```

The map should connect to the rest of the application through contextual actions.

---

# 17. Risk Visualization

Use risk color consistently.

Recommended visual hierarchy:

```text
Critical → strongest emphasis
High     → strong emphasis
Moderate → medium emphasis
Low      → calm / low visual weight
```

Never use a risk heatmap as the only source of meaning.

Provide text labels and legends.

When multiple points overlap, use clustering rather than unreadable marker stacks.

---

# 18. Alerts

Alerts must be immediately scannable.

Each alert should expose:

- severity/risk
- title
- location
- timestamp
- status
- action

Example:

```text
[TriangleAlert] Landslide risk increased
Village A, District X
2 min ago
HIGH

[View Alert]
```

Critical alerts may have stronger borders/backgrounds, but avoid making the entire page red.

---

# 19. Notification System

Support channels:

- in-app
- push
- SMS
- dashboard
- community-facing notifications

The web notification center should unify status regardless of delivery channel.

Notification states:

- unread
- read
- acknowledged
- resolved

Do not rely solely on color. Use text and icons.

---

# 20. Incident Management

Incident lists should prioritize operational usefulness.

Recommended table columns:

```text
ID
Type
Location
Risk
Reported
Status
Assigned team
Action
```

On mobile, convert table rows into compact incident cards rather than forcing horizontal scrolling where possible.

Incident detail should show:

- summary
- map location
- evidence
- risk assessment
- timeline
- verification
- assignment
- response actions
- audit trail

---

# 21. Incident Reporting — Citizen / Field

This is a high-priority workflow.

The interaction should be linear and simple.

Suggested steps:

```text
1. What happened?
2. Where?
3. Evidence
4. Description
5. Submit
```

Use large touch targets.

The form should minimize typing.

Prefer:

- selectable incident types
- automatic GPS
- camera capture
- gallery upload
- optional voice/input assistance if implemented

Avoid long forms.

---

# 22. Offline UX

Offline is a product state, not an error.

Always communicate it clearly.

Example:

```text
[WifiOff] Offline
Reports will be saved on this device and synced automatically when connected.
```

Pending report status:

```text
Pending Sync
Syncing
Synced
Sync Failed
```

Never tell users a report was successfully submitted to the server when the device is offline.

---

# 23. Field Workflow

Field users need fast context.

The primary mobile screen should prioritize:

1. nearby risk
2. current assignment
3. urgent alerts
4. report/verify action
5. map/navigation

Use bottom navigation where appropriate for mobile.

Recommended mobile navigation:

```text
Home
Map
Reports
Alerts
Profile
```

The most important action can also be a prominent central action button, but do not use excessive floating buttons.

---

# 24. AI Assistant UX

AI is an intelligence layer, not the identity of the entire application.

Use the AI assistant in contextual places.

Useful actions:

- Explain this risk
- Summarize incident
- What changed today?
- Why is this area high risk?
- Suggest response
- Answer location-related questions

Preferred visual language:

- standard product colors
- Bot icon from Lucide
- compact assistant panel
- clear distinction between AI-generated insight and verified operational data

Do not use purple gradients just because a feature is AI.

Do not make the entire product look like a chatbot.

AI-generated outputs should show:

- source/context where applicable
- time generated
- confidence/caveat where required
- whether information is verified or advisory

---

# 25. Explainable Risk Intelligence

Every risk result should be interpretable.

Recommended component:

```text
Risk Assessment

HIGH
78% probability
91% confidence

Contributing evidence
• 142 mm rainfall in 24h
• Increased slope movement
• Satellite change detected
• Vulnerable road nearby

Last updated
18 minutes ago

[View Evidence]
[Create Response]
```

The user must be able to distinguish:

- measured data
- system-derived analysis
- AI explanation
- human verification

These must not blur into one visual category.

---

# 26. Response & Action UX

Response prioritization should be action-oriented.

Example priority row:

```text
CRITICAL
Village A
Road access threatened

Affected population: 4,200
Nearest response team: 6.2 km

[Dispatch Team]
[View Details]
```

Recommended statuses:

```text
New
Acknowledged
Assigned
In Progress
Resolved
Closed
```

Use a consistent status treatment across incidents and response actions.

---

# 27. Feedback / Verification Loop

The product must visually represent the closed loop where necessary.

Example incident timeline:

```text
Reported
   ↓
Evidence received
   ↓
Field verification
   ↓
Road status updated
   ↓
Risk reassessed
   ↓
Resolved
```

Use a timeline/stepper rather than another collection of cards.

---

# 28. Reports & Analytics

Analytics should be restrained and legible.

Use:

- line charts
- bar charts
- area charts
- simple donut charts only where useful
- geographic visualizations for spatial data

Charts must have:

- clear labels
- units
- time ranges
- legend
- accessible contrast
- meaningful empty/loading states

Never use chart colors randomly. The same metric category should retain the same visual identity across pages.

---

# 29. Tables

Tables are important for government workflows.

Rules:

- concise headers
- sufficient row height
- sticky header when useful
- clear hover state
- status chips
- predictable alignment
- pagination or virtualization for large datasets
- column visibility controls where appropriate

Avoid overly rounded table cells and excessive card styling around each row.

---

# 30. Forms & Inputs

Inputs should be simple and professional.

Default:

```text
Height: 40–44px desktop
Height: 44–48px mobile
Radius: 6–8px
Border: Neutral 300
Background: White
```

Focus state:

- Primary blue focus ring
- clear contrast
- never rely only on color change

Labels belong above fields.

Error messages must be close to the affected field and explain how to fix the problem.

---

# 31. Buttons

## Primary

Use Primary 600.

Example:

`[Send] Submit Report`

## Secondary

White/neutral surface with border.

## Tertiary / Ghost

For low-emphasis actions.

## Destructive

Use Danger only for destructive/irreversible actions.

Do not use red buttons for ordinary warnings.

Button hierarchy should be obvious:

`Primary > Secondary > Tertiary`

Avoid having multiple visually equal primary buttons in one context.

---

# 32. Status Chips

Use compact semantic chips.

Examples:

```text
[HIGH]
[CRITICAL]
[UNDER REVIEW]
[VERIFIED]
[ASSIGNED]
[IN PROGRESS]
[RESOLVED]
```

Status chips should use subtle tinted backgrounds and strong readable text.

Avoid neon styling.

---

# 33. Search & Filtering

Search should be available where users manage large datasets.

Use predictable placement:

- page header or table toolbar
- map search near map top edge

Filter controls should be grouped.

Example:

```text
Search location...
[Risk Level] [Date] [District] [Status]
```

Avoid scattered filters around the page.

Provide clear reset behavior.

---

# 34. Loading, Empty, Error, and Success States

Every major feature must define four states.

## Loading

Use:

- skeletons
- subtle progress indicators
- map loading overlays

Do not use full-screen spinners for small updates.

## Empty

Explain what is empty and what action is available.

Use a Lucide illustration/icon, not an emoji.

## Error

Explain:

- what failed
- whether data is affected
- what the user can do next

## Success

Use concise confirmation.

Do not over-celebrate operational events.

---

# 35. Accessibility

Target WCAG 2.2 AA where practical.

Minimum requirements:

- keyboard navigation
- visible focus state
- semantic HTML
- labels for inputs
- accessible names for icon-only controls
- sufficient contrast
- no color-only meaning
- large enough touch targets
- readable font sizes
- screen-reader-friendly status messages

Recommended touch target:

`44px × 44px` minimum for mobile controls.

---

# 36. Responsive Design

## Desktop

Primary target for government admin.

Use:

- persistent sidebar
- multi-column layouts
- map + side panel combinations
- dense tables
- dashboards

## Tablet

Primary target for field teams.

Use:

- collapsible navigation
- touch-friendly controls
- two-column layouts where practical
- tablet-optimized map workflows

## Mobile

Primary target for citizen/field interaction.

Use:

- bottom navigation
- stacked content
- large tap targets
- simple forms
- map as full-width interaction
- bottom sheets instead of large desktop side panels

Never merely shrink the desktop dashboard onto mobile.

---

# 37. Multilingual / i18n Requirements

Multilingual support must be built into the component architecture from the beginning.

Potential initial locales may include:

```text
English
Hindi
Nepali
Odia
Malayalam
```

Add new languages without rewriting components.

## Rules

- never hardcode user-visible text inside reusable components
- use translation keys
- keep labels concise but not cryptic
- allow for significant text expansion
- avoid fixed-width buttons
- avoid text baked into images
- ensure data tables can handle longer translated labels
- localize dates
- localize times
- localize number formatting where appropriate
- localize accessibility labels

## Language selector

Place in the global top bar and mobile profile/settings area.

Use a language/globe icon from Lucide.

Example:

`[Languages] English`

Do not use country flags as the primary language identifier. Languages are not countries.

## Translation architecture

Recommended concept:

```text
/locales
  /en
  /hi
  /ne
  /or
  /ml
```

All components consume translated keys.

## Dynamic text

Translation strings must support variables:

```text
"lastUpdated": "Updated {{time}} ago"
```

Avoid string concatenation such as:

`"Updated " + time + " ago"`

because word order varies by language.

---

# 38. Localization of Data

The platform should localize:

- date formats
- time formats
- number separators
- percentages where appropriate
- measurement units if region-specific
- calendar conventions if required

Store timestamps in a standard server representation and format them for the user's locale/timezone at the UI layer.

---

# 39. RTL Readiness

The design system should remain structurally compatible with RTL languages.

Prefer logical CSS properties such as:

```css
margin-inline
padding-inline
inset-inline-start
inset-inline-end
text-align: start
```

Avoid unnecessary assumptions that left always means navigation and right always means content.

---

# 40. Motion & Interaction

Motion should communicate change, not decorate the interface.

Use:

- 120–180ms for micro-interactions
- 180–250ms for panels/modals
- subtle map transitions
- clear success/error transitions

Avoid:

- bouncing cards
- continuous floating animation
- excessive parallax
- decorative gradients moving around
- flashing critical alerts

Reduced-motion preferences must be respected.

---

# 41. Data Freshness

Operational data must show freshness where relevant.

Use:

```text
Updated 4 min ago
Last verified 18 min ago
Live
Sync pending
```

Do not display "Live" unless the data pipeline actually supports a live/current state.

Use subdued metadata styling rather than making timestamps dominate the page.

---

# 42. Trust Boundaries in the UI

The interface must visually differentiate:

### Observed data

Example:

`Rainfall: 142 mm`

### System analysis

Example:

`Landslide probability: 78%`

### AI-generated explanation

Example:

`Potential contributing factors...`

### Human verification

Example:

`Verified by Field Team 03`

Do not present AI-generated suggestions as confirmed facts.

---

# 43. Government Dashboard Information Hierarchy

Default overview should prioritize:

```text
CRITICAL / HIGH RISK
        ↓
ACTIVE INCIDENTS
        ↓
CURRENT MAP / LOCATION CONTEXT
        ↓
RESPONSE STATUS
        ↓
TREND / ANALYTICS
        ↓
HISTORICAL INFORMATION
```

The user should not need to scroll through decorative summary content before reaching critical operational data.

---

# 44. Citizen Dashboard Information Hierarchy

Default citizen home should prioritize:

```text
CURRENT LOCATION
        ↓
LOCAL RISK
        ↓
WHAT TO DO
        ↓
REPORT INCIDENT
        ↓
ALERTS
        ↓
AI ASSISTANCE
```

Keep language plain and direct.

---

# 45. Mobile Incident Reporting Layout

Recommended sequence:

```text
Header
Progress indicator

Question / field
Large selectable options

Location confirmation
Evidence upload
Description

Offline/sync state
Primary submit action
```

Use one primary action per step.

Do not overload the screen with secondary actions.

---

# 46. Map Interaction Principles

Map controls should remain discoverable without covering the main map unnecessarily.

Recommended controls:

- zoom
- current location
- layers
- search
- map type if needed
- legend
- route/navigation where relevant

Selected objects should open a panel or bottom sheet with details rather than a cluttered permanent map tooltip.

---

# 47. Illustrations & Decorative Graphics

Use illustration sparingly.

Preferred visual language:

- clean line illustrations
- geographic shapes
- restrained environmental imagery
- simple diagrams

Never use generic AI illustrations of robots to represent the AI feature.

The product identity should come primarily from UI consistency and information design.

---

# 48. Logo / Brand Mark Placement

Use the product logo/wordmark consistently in:

- authentication screens
- desktop sidebar
- mobile top bar
- system-generated reports where appropriate

Do not place the logo as a giant hero element on normal dashboard pages.

---

# 49. Authentication Screens

Login and signup must feel part of the same product.

Structure:

```text
Brand
Heading
Supporting statement
Form
Primary action
Secondary actions
Language selector
Support/help
```

Avoid over-designed landing-page aesthetics.

The user should get to the operational system quickly.

---

# 50. Professionalism Checklist

Before considering a page finished, check:

- Is there a clear visual hierarchy?
- Does the page use the established palette?
- Are risk colors used semantically only?
- Are icons from Lucide rather than emojis?
- Are spacing values consistent?
- Are border radii consistent?
- Are there unnecessary cards?
- Is the most important information visible first?
- Can a user understand what action to take?
- Does the design work in another language?
- Does the layout survive longer translated strings?
- Does it work on mobile/tablet where required?
- Are loading, empty, and error states defined?
- Is color supplemented by text/icon meaning?
- Does the UI feel operational rather than decorative?

---

# 51. Anti-Patterns — Explicitly Forbidden

The following should not appear unless there is a very specific reason:

1. Generic emoji icons in buttons, cards, navigation, or alerts.
2. Random icon libraries mixed together.
3. Purple gradient backgrounds for every AI feature.
4. Glassmorphism as the default component style.
5. Excessive 20–32px border radii.
6. Huge floating cards with no information hierarchy.
7. Multiple competing accent colors in one screen.
8. Heavy shadows around every component.
9. Giant numbers without context.
10. Decorative charts with no actionable meaning.
11. Fixed-width UI that breaks under translation.
12. Text embedded into images.
13. Desktop dashboards simply scaled down for mobile.
14. Critical alerts represented by color alone.
15. AI responses presented as verified operational facts.
16. Constant animation intended to make the product "feel alive".
17. Fake real-time labels such as "Live" when data is stale.
18. Excessive use of pills and chips.
19. Unnecessary side panels on simple screens.
20. Inconsistent wording for the same state.

---

# 52. Naming Consistency

Use one term for one concept everywhere.

Preferred examples:

```text
Incident
Risk Level
Probability
Confidence
Field Team
Verification
Response
Alert
Report
Location
Risk Map
```

Do not switch between synonyms such as:

`Issue / Case / Event / Incident`

unless they represent genuinely different domain entities.

---

# 53. Content Style

Interface copy should be:

- direct
- concise
- plain-language
- action-oriented
- neutral and professional

Examples:

Prefer:

`No active critical alerts`

over:

`Great news! Everything looks safe right now!`

Prefer:

`Report Incident`

over:

`Tell Us What Happened!`

Prefer:

`Risk increased in Village A`

over:

`Oh no! Something changed!`

The product is operational, not conversationally playful.

---

# 54. Suggested Component Library

Build and reuse these primitives before implementing large pages.

## Foundation

- Typography
- Color tokens
- Spacing tokens
- Icon wrapper
- Focus ring
- Surface

## Navigation

- Sidebar
- Mobile bottom navigation
- TopBar
- Breadcrumbs
- Tabs

## Inputs

- Button
- Input
- Select
- Combobox
- Date picker
- Checkbox
- Radio
- Switch
- Textarea
- File uploader

## Feedback

- Alert
- Toast
- Status chip
- Skeleton
- Empty state
- Error state
- Progress

## Data

- KPI card
- Data table
- Timeline
- Chart wrapper
- Stat block
- Evidence list

## Map

- Map shell
- Layer panel
- Map legend
- Map controls
- Location marker
- Risk marker
- Cluster marker
- Selected location sheet

## Domain

- Risk badge
- Risk assessment panel
- Incident card
- Incident timeline
- Response action card
- Alert item
- Field report card
- Verification status
- AI insight panel

---

# 55. Recommended Design Tokens

These tokens should be centralized in the frontend theme/design-token layer.

```ts
colors = {
  primary: {
    50: '#F4F8FC',
    100: '#E8F1FB',
    500: '#2F73C9',
    600: '#2563A8',
    700: '#1D4F85',
    800: '#163A63',
    900: '#0F2747',
  },

  secondary: {
    50: '#F1FAF8',
    100: '#DFF4F0',
    500: '#20A18F',
    600: '#158A7D',
    700: '#11736C',
    800: '#0F5B56',
  },

  neutral: {
    50: '#F7F9FB',
    100: '#F0F3F6',
    200: '#E2E7ED',
    300: '#CBD2DB',
    400: '#9AA4B2',
    500: '#727E8F',
    600: '#596577',
    700: '#3B4759',
    800: '#263247',
    900: '#172033',
    950: '#0B1220',
  },

  risk: {
    low: '#159447',
    moderate: '#D9A441',
    high: '#E57A17',
    critical: '#C92A2A',
  },

  status: {
    success: '#198754',
    info: '#2F80C9',
    warning: '#D99A14',
    danger: '#D92D20',
  },
}
```

---

# 56. Recommended Breakpoints

Use standard responsive ranges, adjustable to the project framework:

```text
sm: 640px
md: 768px
lg: 1024px
xl: 1280px
2xl: 1536px
```

Design intentionally around these ranges rather than many custom breakpoints.

---

# 57. Page Templates

## Template A — Government Overview

```text
Sidebar
Top bar
Page header
KPI strip
Live risk map + critical alerts
Analytics
Recent incidents
```

## Template B — Map Workspace

```text
Sidebar
Top bar
Map toolbar
Map + layers
Selected location panel
Legend
```

## Template C — Data Management

```text
Sidebar
Top bar
Page header
Search/filter toolbar
Table
Pagination
Details drawer
```

## Template D — Incident Detail

```text
Header
Risk summary
Map/evidence
Timeline
Verification
Response
Activity/audit history
```

## Template E — Mobile Citizen Home

```text
Top bar
Current location
Local risk
Primary actions
Alerts
AI assistance
Bottom navigation
```

## Template F — Mobile Report

```text
Step indicator
Question
Options
GPS
Evidence
Description
Offline state
Submit
```

---

# 58. Implementation Guidance for AI / Coding Agents

When an AI coding/design agent receives a page request:

1. Read this file before creating UI.
2. Reuse existing components before creating new components.
3. Reuse tokens rather than inventing hex values.
4. Reuse Lucide icons rather than emojis.
5. Follow the role-specific information hierarchy.
6. Check mobile behavior before finalizing the component.
7. Check multilingual expansion before finalizing fixed widths.
8. Preserve consistent states: loading, empty, error, success.
9. Avoid introducing a new visual pattern if an existing pattern can be reused.
10. Do not modify the global palette for a single page.
11. Do not introduce gradients unless a specific brand-level decision is made later.
12. Keep risk colors semantically consistent.
13. Treat maps as functional workspaces, not decorative images.
14. Clearly separate observed data, system analysis, AI output, and human verification.

If a design decision conflicts with this document, prefer this document unless the product owner explicitly changes the design system.

---

# 59. Definition of Done for Any UI Screen

A screen is ready when:

```text
[ ] Uses design tokens
[ ] Uses consistent typography
[ ] Uses consistent spacing
[ ] Uses Lucide icons
[ ] Contains no generic emojis
[ ] Uses risk/status colors correctly
[ ] Has a clear primary action
[ ] Has appropriate responsive behavior
[ ] Has multilingual-safe layouts
[ ] Has loading/empty/error states where applicable
[ ] Is accessible
[ ] Uses consistent component patterns
[ ] Does not introduce unnecessary visual decoration
[ ] Clearly communicates data freshness where relevant
[ ] Distinguishes analysis from verified facts
```

---

# 60. Final Visual Direction

The final product should look like a **mature disaster-risk intelligence platform**.

The visual formula is:

```text
Professional GIS
        +
Government operations
        +
Modern enterprise UX
        +
Natural/environmental visual language
        +
Accessible multilingual design
        +
Evidence-driven AI assistance
```

The product should feel credible enough for an emergency operations center and simple enough for a citizen using a phone in the field.

**Consistency is more important than novelty.**

When in doubt, reduce decoration, increase hierarchy, clarify the action, and reuse an existing component pattern.
