## ADDED Requirements

### Requirement: Preferences panel SHALL expose Keep inline syntax hidden

The Preferences panel SHALL include a Keep inline syntax hidden toggle in its non-theme display settings, reflecting the current enabled or disabled state. Activating the toggle SHALL apply the Visual Edit inline-projection behavior on the next render and SHALL persist the preference. The toggle SHALL default to off. Activating it SHALL NOT change document text, document version, dirty state, undo history, or derived Markdown caches.

#### Scenario: Toggle appears in the Preferences panel

- **WHEN** the Preferences panel is open
- **THEN** it shows a Keep inline syntax hidden toggle in the display-settings area, rendered as an actionable control reflecting the current state

#### Scenario: Toggling applies on the next Visual Edit render

- **WHEN** the user activates Keep inline syntax hidden while a Visual Edit document is open
- **THEN** the active app state updates immediately
- **AND** the next Visual Edit render uses the new inline-projection behavior
- **AND** document text and document version stay unchanged

#### Scenario: Toggling persists

- **WHEN** the user toggles Keep inline syntax hidden
- **THEN** the preferences file is updated with the new boolean value

### Requirement: Keep inline syntax hidden SHALL persist safely

The editor SHALL persist Keep inline syntax hidden in `config.toml` as the boolean `stable_visual_inline`, defaulting to off. A missing field SHALL default to off, and a non-boolean field SHALL degrade to off rather than preventing the editor from starting. The preference SHALL be included in preferences reset behavior.

#### Scenario: Missing preference defaults to off

- **WHEN** `config.toml` omits `stable_visual_inline`
- **THEN** the editor starts with Keep inline syntax hidden off
- **AND** Visual Edit keeps the existing caret-triggered inline reveal behavior

#### Scenario: Invalid value defaults to off

- **WHEN** `stable_visual_inline` is present but not a valid boolean
- **THEN** the editor treats the value as off and the preferences file does not prevent the editor from starting

#### Scenario: Value round-trips

- **WHEN** preferences with `stable_visual_inline` enabled are saved and reloaded
- **THEN** the enabled state is restored exactly and reflected by the Preferences control

#### Scenario: Reset restores off

- **WHEN** the user resets preferences after enabling Keep inline syntax hidden
- **THEN** the preference returns to off
- **AND** Visual Edit returns to caret-triggered inline reveal on the next render
