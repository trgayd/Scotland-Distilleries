# Requirements Document
## Scotland Whisky Platform

Version: 0.1 (Draft)
Date: 2026-05-16

## 1. Project Overview

A web platform (with a future mobile app) for exploring Scottish whisky distilleries and whiskies. Users can discover distilleries and whiskies, track visits, log tastings, curate personal collections, and build a wishlist. The platform is data-driven, with distillery and whisky records maintained and periodically enriched by AI.

## 2. Goals

- Provide the most accurate and comprehensive database of Scottish distilleries and whiskies
- Give whisky enthusiasts a personal logbook for visits, tastings, and wishlists
- Build a foundation that scales from a web prototype to a mobile app and a large user base
- Keep the data living and updatable
- Enable community contribution (user-submitted whiskies) with quality control

## 3. User Personas

The Explorer: Casual whisky drinker. Browses by region, flavour profile, age. No strong prior knowledge.

The Enthusiast: Tracks every dram. Logs tastings, rates whiskies, maintains wishlists and buy lists.

The Traveller: Plans trips to Scotland. Marks distilleries to visit, tracks visits, plans routes.

The Contributor: Submits whiskies or corrections not yet in the database.

The Administrator / Moderator: Manages the database, approves submissions, runs AI enrichment tasks.

## 4. Functional Requirements

### Distillery Features

| ID | Requirement |
|----|-------------|
| D1 | Browse all distilleries, filterable by region, founding year, production style |
| D2 | Search by name or town |
| D3 | Detail page: name, region, town, founding year, history, expressions, flavour profile, map location |
| D4 | Sort by founding year, name, region |
| D5 | Each distillery links to its whiskies |

### Whisky Features

| ID | Requirement |
|----|-------------|
| W1 | Browse whiskies, filterable by distillery, region, age, cask type, flavour profile, price range |
| W2 | Search by name |
| W3 | Detail page: name, distillery, age, ABV, cask type, tasting notes, availability |
| W4 | Sort by age, ABV, distillery, user rating |
| W5 | Users can submit a whisky not in the database |
| W6 | Submissions enter a moderation queue before publishing |

### User Account

| ID | Requirement |
|----|-------------|
| U1 | Register with email and password |
| U2 | Log in / log out |
| U3 | View and edit profile |
| U4 | Delete account and all data |

### User - Distillery Interactions

| ID | Requirement |
|----|-------------|
| UD1 | Mark as liked |
| UD2 | Mark as visited (with optional date and notes) |
| UD3 | Mark as want to visit |
| UD4 | Personal distillery dashboard |

### User - Whisky Interactions

| ID | Requirement |
|----|-------------|
| UW1 | Mark as tasted (with optional date and notes) |
| UW2 | Rate: love it / like it / neutral / don't like it |
| UW3 | Mark as want to taste |
| UW4 | Mark as want to buy |
| UW5 | Mark as owned |
| UW6 | Personal whisky dashboard |

### AI Data Enrichment

| ID | Requirement |
|----|-------------|
| AI1 | Admins trigger enrichment jobs for distillery data |
| AI2 | Admins trigger enrichment jobs to add new whiskies |
| AI3 | All AI-generated content flagged and reviewed before publishing |
| AI4 | Enrichment jobs logged with timestamp, source, fields updated |

### Moderation

| ID | Requirement |
|----|-------------|
| M1 | Admins view queue of user-submitted whiskies |
| M2 | Admins approve, reject, or edit before publishing |
| M3 | Submitting user notified of outcome |

## 5. Non-Functional Requirements

Performance: Page load under 2s, API responses under 300ms, real-time search (debounced)

Scalability: Local Docker to AWS with config changes only; schema supports 10k+ whiskies, 100k+ users

Security: bcrypt passwords, JWT auth, role-based access control, input validation

Data Integrity: Source flags (manual/AI/user), soft deletes, moderation before publish

Portability: Fully Docker-local, .env config, no hardcoded environment assumptions

Mobile Readiness: Token-based auth compatible with mobile clients

Accessibility: WCAG 2.1 AA

## 6. Use Cases

UC1 - New user discovers a distillery: Browses, filters Islay, clicks Lagavulin, reads detail, clicks Like, prompted to register, like saved.

UC2 - Enthusiast logs a tasting: Searches Ardbeg Uigeadail, marks tasted, adds date/notes, rates love it, adds to want-to-buy list.

UC3 - Traveller plans visits: Filters Highland, marks several as want-to-visit, later marks Glenmorangie as visited with date.

UC4 - Contributor submits a whisky: Searches, not found, fills submission form, confirmation shown, moderator approves, user notified.

UC5 - Admin runs AI enrichment: Selects distilleries with gaps, triggers job, reviews diff, approves, records updated and flagged.

## 7. Out of Scope (v1)

- Distillery map / route planning (planned later)
- Social features (following users, sharing tastings)
- E-commerce / purchase links / price tracking
- Native mobile app (backend designed to support it)
- Multi-language support

## 8. Constraints

- Local-only development (Docker); no cloud costs during prototyping
- AI enrichment is opt-in and logged - no automatic background updates
- User data must be exportable (GDPR)
- Scotland-specific only - no non-Scottish whiskies in v1
