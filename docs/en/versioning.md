# Versioning

## Introduction

Versioning in Bonyan serves two purposes: declaring a plugin's compatibility with the architecture, and communicating changes within the plugin itself. These two are separate and follow different formats.

## Architecture Version

The Bonyan architecture version follows a two-segment format:

```
MAJOR.MINOR
```

**MAJOR** changes when a fundamental change to the architecture has been made that is incompatible with previous plugins.

**MINOR** changes when a new capability has been added to the architecture that is compatible with previous versions.

Each plugin declares in its manifest the architecture version it was written for. The Core compares this value against its own version at startup.

### Compatibility Rules

If MAJOR is the same and the plugin's MINOR is less than or equal to the Core's MINOR, the plugin is compatible and will start.

If MAJOR differs, the plugin is incompatible and will not start.

If the plugin's MINOR is greater than the Core's MINOR, the plugin was written for a newer version of the architecture. The Core will not start it and logs the reason.

## Plugin Version

Each plugin's version follows a three-segment format:

```
MAJOR.MINOR.PATCH
```

**MAJOR** changes when a breaking change has been made to the plugin. This means plugins that depend on this plugin's service must be updated.

**MINOR** changes when a new capability has been added to the plugin that is compatible with previous versions.

**PATCH** changes when a bug has been fixed without changing the plugin's overall behavior.

## Versioning Rules

**Every change must update the version.** A plugin that has changed but whose version has not changed cannot be trusted.

**Versions never decrease.** A version only increases.

**A MAJOR change resets MINOR and PATCH to zero.** Example: from `1.4.2` to `2.0.0`.

**A MINOR change resets PATCH to zero.** Example: from `1.4.2` to `1.5.0`.

**A plugin's initial version is `1.0.0`.** The `0.x.x` range is for plugins that are still experimental and whose contracts may change.