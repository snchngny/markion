## ADDED Requirements

### Requirement: Keep inline syntax hidden UI chrome SHALL be localized

The system SHALL route every user-visible string for Keep inline syntax hidden through the i18n layer, including the Preferences panel toggle label, on/off status feedback when the preference is toggled, the edit affordance on a rendered link, and the floating editor chrome for inline math and for an exact authored token. Existing link-editor strings MAY be reused for the Markdown and simple HTML-anchor fields. Adding the new message keys SHALL require translations in every supported language or the build SHALL fail.

#### Scenario: Preferences panel label reflects the active language

- **WHEN** the active interface language changes
- **THEN** the Keep inline syntax hidden toggle label in the Preferences panel renders in the active language

#### Scenario: Toggle status feedback reflects the active language

- **WHEN** the user toggles Keep inline syntax hidden
- **THEN** any status bar message indicating the new on or off state is produced through the active language translation

#### Scenario: Floating editor chrome reflects the active language

- **WHEN** the user opens the inline-math editor or the exact-token editor while Keep inline syntax hidden is on
- **THEN** that editor's labels, confirm action, and cancel action render in the active language

#### Scenario: New message keys require all-language translations

- **WHEN** a developer adds the Keep inline syntax hidden message variants to the message enum
- **THEN** the project fails to compile until every supported language covers them via the exhaustive match
