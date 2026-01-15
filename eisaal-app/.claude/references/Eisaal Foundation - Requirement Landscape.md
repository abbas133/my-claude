# Esaal Foundation – App Landscape

V 1.2 – 8th January 2026

## Overview

The Eisaal Foundation aims to develop a comprehensive digital ecosystem
tailored to the specific spiritual, jurisprudential, and social needs of
the Shia Muslim community.

While the digital Islamic market is saturated with generic applications,
there remains a significant void in platforms that address the nuanced
theological requirements of the Jafari school of thought (Fiqh-e-Ja'faria)
while simultaneously integrating essential community services.

The proposed application is not merely a utility for prayer times or
recitations; it is envisioned as a lifecycle companion, supporting the
user from birth (baby names) to daily practice (Namaz, Quran), annual
obligations (Khums), social milestones (Matrimonial), and end-of-life
planning (Wasiyat/Aamaal).

## Philosophy

- Propagate message of Ahlul Bait (a.s)
- Accessible, Beneficial and Friendly to all
- Paving a path to structured community services benefitting the community

The app is designed to be divided into 3 major categories. Every module,
directly or indirectly shall fit within at least one of the below
categories

### Today (Home page/Landing page)

- Hijri Calendar
- Namaz Timing
- Reminders
- Daily aamaal (To-Dos)
- TBD

### Recitations/Tools (Tab)

- Quran
- Duas
- Ziyarat
- Books
- Fiqh
- TBD

### Community

- Online classes
- Matrimony
- Blood donation
- TBD

## Goals

To develop a modern, cross-platform, highly maintainable, modular open to
extension and infrastructure cost-effective app on modern app development
tech-stack with a provision of backend admin panel.

To create a "Digital Sanctuary" where technology serves theology, ensuring
that every feature adheres strictly to Sharia compliance while utilizing
best-in-class modern technology.

## Tech Stack

### Front-end

#### Flutter (Dart)

Single Codebase: Delivers iOS, Android, and Web apps from one source.

Text Rendering: Superior handling of complex Arabic ligatures (Quran/Duas)
compared to React Native.

Performance: Compiles to native ARM machine code, essential for the smooth
animation of the Mohr-e-Ameen counter.

### Back-end

#### Supabase

PostgreSQL Power: Unlike Firebase (NoSQL), Supabase is built on Postgres.
This is non-negotiable for the Shajra (Family Tree) which requires
recursive SQL queries, and Matrimonial filters (e.g., "Find Female AND
Age 20-25 AND City=London").

No Vendor Lock-in: You can self-host this stack if costs rise, vital for
a non-profit.

### APIs

TBD

### Storage

#### PostgreSQL

Relational Integrity: Essential for the complex many-to-many relationships
in genealogy and teacher-student booking systems.

PostGIS: Native geospatial support for "Blood Donors Near Me" and Qibla
direction.

### Admin Panel

#### Flutter Web

Not finalized yet, still looking for better suited options

### Chatbot

#### Flowise + pgvector

RAG Architecture: Uses Supabase's pgvector to store non-English texts.
The bot retrieves answers from verified Shia sources rather than
hallucinating. Flowise provides a drag-and-drop builder for this logic.

### AI

#### Google ML Kit

On-Device Privacy: For the Mohr-e-Ameen (Sajda Counter), ML Kit processes
pose detection locally on the phone. No camera feed is ever sent to the
cloud, preserving absolute privacy during prayer.

## Core Features

### Multi-lingual

The app shall be completely operable with below languages. This is not
only for translation but for the whole app interface and interaction.

1. English
2. Urdu
3. Gujrati
4. Hindi

### Cross-Platform

The app shall be made available on below devices and platforms. All the
subsequent updates, unless not platform dependent, shall be pushed at
the same time.

1. Android
2. iOS
3. iPad OS
4. Web App

### Notifications

The app shall provide notifications for events including, but not limited
to the below list.

1. Namaz time (time of the day + app level pre-config)
   1. Upcoming namaz time
   2. Awwal waqt remaining
   3. Ada time remaining
2. Important Dates (day of the month + database)
   1. Shahadat
   2. Wiladat
   3. Historic event
   4. Prominent figure related
3. Daily Hadees (time of the day + database)
4. Geolocation based events (push notification set by admin for specific
   city/state/country)
   1. Namaze aayaat
   2. Moon sight (new Hijri month start)

These notifications are linked to either Hijri Calendar or time of the
day. Time dependent notifications are solely reliant upon time of the day
and pre-configured read-only configs at the app level; for example, 30
minutes after Azan for awwal waqt count down.

### Chat

Users shall be able to chat throughout the app from below chat-enabled
modules – the chat option will show up only for these modules

1. Hajj/Umrah guidance
2. TBD

Initial chat shall happen with a bot which will gather data regarding what
the user is trying to reach and then try to provide a solution based on
knowledge it already has. For the rest of the cases, it is going to
escalate to human agents.

Human-agent routing shall be skill based routing. For example, matrimonial
queries go to matrimony moderator, fiqh questions go to an aalim online
etc.

### Video Call

Video call integration shall be provided for below modules

1. Hajj/Umrah guidance
2. Qirat check/correction (human)
3. Online classes
   1. Najaf classes
   2. Quran/deeniyat teacher-student

### Reminders

#### Namaz Reminder

If a user has not yet acknowledged to have prayed a particular namaz, a
snoozable reminder shall be given to the user along with time remaining
for ada.

#### Daily Hadees

A hadees from authentic sources, chosen randomly shall be displayed to
the user

- Option in setting to on/off
- Option in setting to configure how many hadees user wants to read daily

#### Daily Salawat

A reminder to recite N (configurable number) of salawat daily

## User

Below information to be collected from the user for profile creation to
support different features throughout the app.

| Name | Identification |
| :--- | :--- |
| Gender | Qaza calculator, Nifas calculator |
| City/State/Country (Geolocation option) | Namaz timing, Hijri date, etc. |
| Preferred language for Matan | Quran language, Dua language |
| Preferred language for description | Descriptions Translations |
| Khums Date | Khums calculation/payment reminder |

### User Registration

Users can register using their Google email address. Additional
information, required for app operation will be collected upon profile
creation/user registration

### Login

Users will be able to login using Google profile using Google OAuth,
passwords should not be stored.

### Profiles

A single user may have more than one profile

- Blood donor
- Matrimonial
- Service provider (Qaza namaz/roza/quran)
- Teacher
- Student

## Digital Mohr-e-Ameen

A utility to count the number of sajda for a namaz which will help elderly
and people who doubt in namaz rakat.

- Option 1: Use proximity sensor
- Option 2: Use camera + AI

### Link To Namaz

Ask the user if they are offering –

- Current ada namaz
  - Mark complete if yes
- Qaza namaz of self/marhomeen
  - Reduce number from the qaza counter

### Algorithm

The screen turns black (battery saver) when the sensor is covered.

Event Trigger: Distance < 1cm.

Debounce Logic: To prevent false counts (e.g., adjusting the phone), the
sensor must be covered for > 1.5 seconds (duration of a Sajdah) and
uncovered for > 1 second before the next count is registered.

Rakat Increment: Every two valid Sajdah counts = 1 Rakat.

Fall-back: If the proximity sensor is absent (some modern tablets), use
the Camera/Face Detection API (Google ML Kit) to detect the "Sajdah" pose,
though this consumes more battery

## Quran

The Quran will be available in following languages.

| Language Support | Translation | Translitration |
| :--- | :--- | :--- |
| Arabic | - | - |
| Urdu | ✅ | ⛔ |
| Hindi | ✅ | ✅ |
| Gujrati | ✅ | ✅ |
| English | ✅ | ✅ |

### Data Source

The text must be sourced from the Tanzil Project, which provides verified
Uthmani script XML/SQL dumps.

### Verification Protocol

Upon first launch/download, the app calculates the SHA-256 checksum of
the local SQLite database containing the verses.

This checksum is compared against a secure API endpoint hosted by Eisaal.
If they do not match (indicating corruption), the database is forcibly
re-downloaded.

### Font & Font-size

Arabic font and font-size can be changed by the user. Possibility to
achieve this directly on read-page and fallback to come from user profile.

### Bookmark

Readers can bookmark their last read part to resume on subsequent reading
sessions.

### Tajweed Engine

Utilizing Tanzil's metadata, the rendering engine (Flutter CustomPainter)
will apply specific color codes to letters requiring Ghunna, Qalqala, or
Ikhfa.

## Recitations (Duas/Ziyarat)

The app shall include all Duas and Ziyarat from [Duas.org](http://Duas.org)
and other sources. The app can also be extended to include major sources
of Duas and Ziyarat including but not limited to -

- Mafatihul Jinan
- Sahifa Sajjadia
- Tohfatul Awam
- Sawabul aamal
- TBD

### Daily Recitation

- Based on Hijri data, the app will suggest a To-Do style list of Duas,
  Ziyarat and Aamal for that day.
- Based on day of the week, the app shall suggest
  - Dua for the day
  - Ziyarat for the day

### Event Notification

Based on Hijri date and/or Georgian date, the app will notify of important
event of that day including but not limited to -

- Wiladat notification
- Shahadat notification
- Historic event
- Prominent figure

The notification shall have the option to read more about that event.

## Fiqhi Masail

The app will host Fiqhi Masail based on authentic sources (official
websites of the Maraje – [sistani.org](http://sistani.org),
[makarem.ir](http://makarem.ir), [leader.ir](http://leader.ir) etc).
All Masail will be listed in two style –

### Categorized

Masail are listed in categorized format, matching how those are listed
in the Tauzihul Masail.

### Question Answer Format

Masail explained on official websites in the form of question answers
will be listed under this section. Both questions and answers are readable.

### Ask A Question

Users will have the ability to ask questions if they can't find what they
are looking for. These questions, after getting their answers will also be
listed under the Question Answer format section.

### Masail Search

Ability to search what the user is looking for on a keyword basis
including categories and questions answers both.

When nothing is found, suggest asking a question with pre-populated
information about the question.

### Video Link

A video link which explains a masla or a topic can be added which will
show up to the user and they can watch the video if they prefer.

- No native video play-back, the link will open Instagram/YouTube to
  continue to play the video

## Khums Calculator

Khums is a 20% tax on surplus income. However, the definition of "surplus"
and "deductible expenses" varies significantly between Maraja.

### Inputs

Different sections for which Khums is wajib is taken as an input from the
user (rough estimated amount) and the khums is calculated based on it.

- Annual income
- Total savings
- TBS

### Deduction Logic (Strategy Pattern)

Marja == Sistani, Marja == Makarem Sirazi etc.

### Output

- Obligated Khums amount
  - Sahme-Imam a.s
  - Sahme-Sadata
- Option to pay directly from the app.
- Link to show Ijaza

## Qaza Namaz Calculator

To keep track of users qaza namaz. Users will input an estimated number
of days of missed namaz and the App is going to calculate, present and
store the number of missed prayers information from there.

The calculation would work differently for male and female users (will
exclude estimated menses period from the missed prayers number of days).

### Namaz Completion Target

The app can suggest strategies to complete these missed prayers.

- Time based completion – if a user wants to complete missed prayers for
  example in the next 90 days, how many days of qaza prayers need to be
  completed per day to achieve this.
- Missed day based – If a user wants to complete based on availability.
  For example, a user says they can pray 2 days of missed prayers daily
  then the app should show how many days it will take them to complete
  the missed prayers.

### Namaz Self or Other

When entering missed namaz information, the app is going to ask if these
are for self or for someone else (father, mother, other momeneen).

User can track qaza prayers of multiple entities (self, mother, father,
other momin)

## Qaza Roza Calculator

To keep track of users qaza roza. The user will input an estimated number
of years/days of missed roza and the app is going to calculate, present
and store the number of missed roza information from there.

The app will provide information to female users in notes format that roza
missed during pregnancy is ruled differently (as per tauzihul masail)

### Roza Completion Target

The app can suggest strategies to complete missed roza

- Time based completion – if a user wants to complete missed roza for
  example in next 90 days, the app should suggest roza pattern (alternate
  day, weekend, daily etc)
- Long term completion – if the user wants to complete missed roza by
  observing it on days of recommended fasting, the app should mark which
  days the user should observe roza and provide estimates by when missed
  roza will be completed.

### Roza Self or Other

When entering missed roza information, the app is going to ask if these
are for self or for someone else (father, mother, other momeneen).

User can track qaza roza of multiple entities (self, mother, father,
other momin)

## Nifas Calculator

Nifas calculator based on tauzihul masael

## Qibla Direction

A Qibla compass to help users determine Qibla direction from their
location.

## Wasiyat (Will)

### Phase I

- Provide model will, based on wasiyat nama of Ayatullah Marashi Najafi
- Include a section for qaza namaz automatically if there's any for self
- Include a section for qaza roza automatically if there's any for self.
- Give provision of including link to external documents, attachments
- Accept a short video of the user stating that it is their valid will
  and if possible include 2 witness's testimony also.
- A disclaimer to be accepted by the user "this is a shariya-valid will,
  however it may not stand as a legal document in the court of law."

### Phase II

- Explore options to validate the will online
  - Aadhar OTP
  - E-sign
- Provide contacts to community trusted legal advisers who can help user
  - Register the will
  - Notarize the will

## Matrimonial Services

This shall be a paid-only service to keep it within the reach of authentic
users only.

### Profile

- Enable profile creation only after subscription
- If girls are creating their own profile, send a verification link to
  "wali". The profile shall be searchable only after wali's verification
  via link
- Option to select "open for"
  - Nikah-e-daimi
  - Nikah-e-muta
- Both of the above options can be selected, this is just another filter
  which can be used while searching/browsing

### Matrimonial Search

- Filter based on –
  - Area (State/City)
  - Age group
  - Open for (Nikah-e-daimi/Nikah-e-muta)

### Privacy Tiers

Tier 1 (Nikah-e-Daimi): Profile photo blurred until mutual match.

Tier 2 (Nikah-e-Mutah): High privacy. Photos blurred. Name masked.
Searchable only by specific criteria.

### Add-on

- Feature to request background verification (chargeable service)
- TBD

## Baby Names Suggestions

Suggest Shia baby names

- Ask (optional) for the complete Georgian date of birth or Hijri date
  of birth
  - Based on date of birth, check if that day is of any historic
    significance and then prioritize name suggestions matching the
    historic significance.
- Option to suggest name
  - A form to be filled with information
    - Name
    - Meaning
    - Origin (if any)
  - This form will be submitted for the backend team's approval and will
    show up in suggestions post approval.

## Blood Donors

A community safety net for emergency blood requirements.

- Option to register as a blood donor
  - Indicate that they may be called during need based on blood type match
- Option to search for blood donors
  - City/State
  - Blood group

### Functional Flow

1. Registration: User selects Blood Group and consents to "Emergency
   Notifications."
2. The "Uber" Model for Privacy
   1. Requester posts a need (e.g., "O+ needed at City Hospital").
   2. The app sends Push Notification to all verified O+ donors within
      a 10km radius (using PostGIS/Geofencing).
   3. Donor accepts request.
   4. The app opens a masked chat or VoIP call. Real phone numbers are
      not exchanged until the donor chooses to share them.

## Quran/Deeniyat Teacher

- Option to register as teacher
  - Create teacher profile of the user
  - Information needed TBD
- Option to search for a teacher
  - Offline/online
  - city/state
  - male/female
- Option to enroll as a student

## Live Classes

- Google Meet links of Live Najaf classes
- Links to historic classes
- How to provide access TBD

## Aamaal for Marhomeen

A paid community service, where a user can request for –

- Quran recitation
- Qaza Namaz
- Qaza Roza

### Service Provider

Users can register themselves as a service provider for these recitations.
A verification letter from local aalim along with the form shall be
submitted to the admin and upon approval the service provider will show up
for different requests.

## Shajra (Genealogy)

This feature is to provide the family tree, roots and origins of different
Sayyed families.

- Option to submit family tree
- Option to explore existing family tree (graph format)
  - Present any description information if present

## Non-Functional Requirements (NFRs)

### Data Privacy & GDPR Compliance

#### Right to Forgotten

Users must have a "Delete Account" button that creates a cascading delete
of their profile, chat history, and Khums logs.

#### Financial Privacy

Khums records are strictly private. Admins cannot view a user's income,
only the transaction ID of the final donation.

#### Matrimonial Photos

Images for Mutah profiles must be stored in a private Supabase Storage
bucket with Time-Limited Signed URLs to prevent scraping.

### Offline Capability

#### Quran/Duas

The SQLite database of text and translation is packaged inside the app
bundle (approx. 15-20MB). It does not require network access to read.

#### Sync Logic

User data (Bookmarks, Qaza counts) is stored in a local Hive (NoSQL)
database on the device and syncs to the cloud whenever connectivity is
restored.

## Implementation Roadmap

### Phase 1: The Foundation (Months 1-3)

#### Phase 1 Goal

Release a high-quality utility app to build user trust.

#### Phase 1 Features

- Prayer Times (default + overrides)
- Quran (Verified Tanzil text, offline mode)
- Duas & Ziyarat (Data from [Duas.org](http://Duas.org))
- Digital Mohr-e-Ameen (Proximity sensor)
- Hijri Calendar with Events

#### Phase 1 Technical Focus

Setting up Flutter environment, Supabase auth, and local SQLite
integration.

### Phase 2: The Lifecycle Utilities (Months 4-7)

#### Phase 2 Goal

Introduce complex logic tools.

#### Phase 2 Features

- Khums Calculator (Marja-specific logic wizard)
- Qaza Tracker (With female biological exclusion logic)
- Wasiyat Generator (PDF export)
- Baby Names (Linked to historic events API)

#### Phase 2 Technical Focus

Edge Functions for complex calculations and PDF generation. Payment
gateway integration.

### Phase 3: The Community Ecosystem (Months 8-12)

#### Phase 3 Goal

Connect users and launch social features

#### Phase 3 Features

- Matrimonial Service (Wali verification, Identity checks)
- Blood Donation (Geo-location notification system)
- Shajra (Family Tree graph visualization)
- Teacher/Class Directory

#### Phase 3 Technical Focus

PostGIS implementation for location services. Graph database optimization.
Admin panels for verification workflows

## List of Screens

### Onboarding & Authentication

1. Splash Screen
   1. Initial branding load with Salawat (Possible audio in background)
2. Language Selection
   1. Initial setup for app-wide language which can be changed later from
      profile settings
3. Login/Sign Up
   1. Option to login or sign up
   2. Login with Google option (via OAuth)
4. User Profile Setup
   1. Basic profile changes like language, city, gender etc

### Main Navigation Tabs

#### Tab 1: Today (Home Dashboard)

Displays below components/widgets on the page in active state

- User Avatar/Pic (circular) at top left corner
- Notification button with count badge at top right corner
- Favorite button at top right, left to notification button
- Hijri Date and Year (bold), and below georgian date and year
- A widget to show prayer time
  - Current prayer and time
  - Upcoming prayer and time
- Widget for daily aamaal (recurring, set by the user)
  - To do style
  - Visible percentage completion percentage
- A widget for today's aamaal (today's specific aamaal based on hijri
  date)
  - To do style
  - Visible percentage completion percentage
- Verse of the day
  - A Quranic verse with option to read more
- Hadees for the day
  - A quote from masoom
  - Option to share

#### Tab 2: Resources

Hub for accessing Quran, Recitations, and Calculators.

#### Tab 3: Community

Dashboard grid for social services like Matrimony, Blood Donation, etc..

### Resources (Recitation and Worship)

Quran Reader: Text view with translation/transliteration toggles and
Tajweed highlighting.

Dua & Ziyarat Library: Categorized list view (e.g., Sahifa Sajjadiya,
Mafatihul Jinan, Special Occasions, Special Purpose)

Recitation Player/Reader: The actual reading view for a specific
Dua/Ziyarat.

Digital Mohr-e-Ameen: Full-screen tool for counting Sajdahs using the
proximity sensor or camera.

Qibla Compass: Direction finder using geolocation.

### Calculators & Trackers

#### Khums Calculator Screen

Input Screen: Fields for Annual Income, Savings, and Expenses.

Result Screen: Shows "Sehme-Imam" and "Sehme-Sadat" amounts with a
payment option.

#### Qaza Namaz Tracker

Dashboard: Grid showing missed prayers for Self, Father, Mother, etc..

Input/Log: Form to record missed or performed qaza prayers.

#### Qaza Roza Tracker

Similar dashboard and input screen for missed fasts.

#### Nifas Calculator Screen

Input screen for dates and calculation result based on Tauzihul Masail.

### Knowledge Base (Fiqh)

#### Fiqhi Masail Listing

Categorized list of religious rulings.

#### Q&A Detail View

"Question & Answer" format for specific rulings.

#### Ask a Question Form

Form to submit new questions to scholars.

#### Fiqh Search Page

Dedicated search for Fiqh topics.

### Community Services

#### Matrimonial

Profile Browsing: List of profiles with blurred photos (Privacy Tiers).

Filters: Search by City, Age, Nikah type (Daimi/Mutah).

My Profile: Edit/Create profile screen (requires subscription).

#### Blood Donation

Map/List View: Search for donors by blood group and location.

Donor Registration: Form to sign up as a donor.

#### Teachers & Classes

Directory: List of Quran/Deeniyat teachers.

Teacher Profile: Details of a specific teacher with an "Enroll" option.

Live Classes: List of links (Google Meet/YouTube) for active sessions.

#### Aamaal for Marhomeen Screen

Request Service: Form to request prayers/recitations for deceased.

Provider Dashboard: For verified users to accept requests.

#### Shajra (Genealogy) Screen

Tree View: Interactive family tree graph.

Submission Form: Form to upload/submit family lineage.

#### Lifecycle Features

Baby Names

Search/List: Database of Shia baby names.

Suggestion Form: "Suggest a Name" input form.

#### Wasiyat (Will) Screen

Generator: Form to input assets and wishes.

Preview/Export: View the generated PDF will.

### Settings & Info

Settings: Notification toggles (Namaz, Events), Theme selection, Font
size adjustments.

About & Privacy: Standard legal and app info pages.

## Themes

### Divine Reflection

Best for: Standard/Default App Theme This captures the physical beauty
of the shrine: the interplay between the cool silver of the mirror work
and the warm royalty of the gold grating. It feels "Heavenly" and "Clean."

Primary (The Zareeh): Royal Gold (HEX: #C5A059) - Taken from the gold
grating.

Background (The Mirrors): Crystal White / Pale Silver (HEX: #F5F7FA) -
Represents the mirror mosaics.

Secondary/Text (The Script): Slate Grey (HEX: #37474F) - Represents the
silver metalwork and shadows.

Vibe: Bright, Holy, Spacious, Reflection.

### The Crimson Tear (Muharram / Aza Mode)

Best for: Muharram & Safar (Dynamic Theme) The user noted the lighting
changes. This palette captures the red glow seen in the images (from
the lights and flowers) set against the black mourning cloths often
draped on the Zareeh.

Primary (The Blood): Deep Garnet Red (HEX: #7f1d1d or #880E4F) - From
the red lighting and floral wreaths.

Background (The Mourning): Midnight Black / Dark Charcoal (HEX: #121212)

- Represents the "Sham-e-Ghariban" atmosphere.

Accent (The Legacy): Muted Bronze (HEX: #A1887F) - The gold of the Zareeh
seeing through the darkness.

Vibe: Solemn, Revolutionary, Emotional, Focused.

### The Inner Sanctum (Warmth & Prayer)

Best for: Reading Mode (Quran/Duas) This pulls from the warm, amber glow
of the chandeliers reflecting off the gold, visible in the upper parts of
the images. It creates a "cozy" spiritual feeling, like sitting near the
Zareeh.

Primary (The Glow): Warm Amber (HEX: #FFCA28 - muted to #FFB300) - The
chandelier light.

Background (The Marble): Warm Cream / Limestone (HEX: #FFF8E1) - The
marble flooring of the Haram.

Accent (The Detail): Deep Wood / Dark Brown (HEX: #4E342E) - Grounding
element.

Vibe: Intimate, Warm, Comforting, Historic.

### The Grounded Earth

A completely neutral, modern, and highly accessible palette. It steps
away from "color" to focus purely on content, using earth tones to create
a sense of stability.

Primary: Warm Taupe / Cocoa (Hex: #795548)

Secondary/Surface: Cream/Off-White (Hex: #FFF8E1)

Accent: Muted Gold (Hex: #BCAAA4)

### The Persian Sky

Primary: Dusty Slate Blue (Hex: #546E7A)

Secondary/Surface: Cream/Off-White (Hex: #FFFDF5)

Accent: Muted Gold (Hex: #C5A059)

### UI Texture

Subtle usage of geometric patterns (Islemi/Arabesque) in the background
with very low opacity (5%) to mimic the shrine's grating without
cluttering the interface.
