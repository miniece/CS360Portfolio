# Weight Tracker — CS 360 Mobile Architecture and Programming

An Android application for logging daily weight against a personal goal, built in Java
with a local SQLite database. Submitted as the Project Three artifact for CS 360.

## App Requirements and Goals

The app was built for a client scenario calling for a weight-tracking tool. The
requirements were a database with at least three tables (user credentials, dated
weight entries, and a goal weight), a login screen that doubles as account creation,
a grid displaying every entry with its date, the ability to add and manage entries
and a goal, and a notification when a logged weight reaches that goal.

The underlying user need is not the individual number but the record. A single weight
reading means very little in isolation; a sequence of dated entries shows a direction.
I identified three user types by goal: someone working toward a target, someone
maintaining one they have already reached, and someone logging at a clinician's
request who needs an accurate dated history to bring to an appointment. All three fit
the app into an existing daily routine, which meant a single entry had to take seconds.

## Screens, Features, and UI Design

The app uses four screens. A combined **login and registration** screen validates
credentials against the users table or creates a new account if the username is free,
with inline errors beside the field that caused them. The **weight list** is the home
screen and the one users see most: a summary card showing goal and latest weight, a
fixed header row, and a scrolling list of entries with date, weight, and change since
the previous reading. An **add/edit** screen handles both creating and updating an
entry. A **goal** screen stores the target weight and the phone number alerts are
sent to.

The design decisions all follow from the speed requirement. A floating action button
puts the most common task one tap from the home screen. The date field defaults to
today, so the usual case requires no input at all. Goal setting lives behind the
overflow menu rather than in the daily path, because setting a goal is rare and does
not belong in a flow repeated every morning. Navigation is deliberately shallow —
one top-level destination, with every other screen one step away and returning
directly to it. The app uses standard Material components rather than custom
controls, which keeps it consistent with the rest of the Android system and
predictable across versions, as the Android core app quality guidelines recommend.

## Coding Approach

I planned the data layer before writing any UI logic. Every read and write goes
through a single `DatabaseHelper` class extending `SQLiteOpenHelper`, so no activity
issues raw SQL and the schema lives in one place. Writing out the screen-to-method
mapping ahead of time surfaced decisions I would otherwise have hit halfway through
the build — most importantly that every query had to be scoped by user ID so two
accounts on the same device never see each other's data.

Two choices proved worth the up-front thought. Dates are stored in ISO format
(`yyyy-MM-dd`) so they sort correctly in SQL, then reformatted for display, and the
date field opens a picker rather than accepting typed text so a malformed date cannot
reach the database at all. The change column is computed from adjacent rows rather
than stored, since derived values that are persisted can fall out of sync with the
data they are derived from.

Both strategies generalize: isolate data access behind one interface, and prevent
invalid input at the point of entry rather than validating it after the fact.

## Testing

I tested continuously in the Android Emulator rather than at the end. Each CRUD
operation was verified independently, and persistence was confirmed by logging
entries, logging out, logging back in, and checking the data survived. The SMS
permission was tested on both branches — granted, where the alert sends as a text,
and denied, where the app continues to function and shows the alert on screen
instead. I also used breakpoints to watch values arrive and confirm where state
changed.

This process matters because compiling cleanly is not the same as working. The most
stubborn defects in this project produced no errors at all: an overflow menu that
rendered in a dark color against a dark toolbar, and a toolbar drawing underneath
the status bar because newer Android versions enforce edge-to-edge layout, which
sent taps to the system bar instead of the button. Neither was a flaw in the logic,
and neither would have been found without running the app and interacting with it.

## Innovation and Challenges

The hardest design problem was fitting new information onto a screen that was already
full. In a separate redesign exercise I solved it by auditing which elements earned
the space they occupied, and the same reasoning applied here: the summary card shows
only the two numbers a user checks first, and everything else is one tap away.

The SMS feature posed a different challenge. Android treats runtime permissions as
revocable at any time, so an app that becomes unusable after a denial fails the
platform's expected behavior. Rather than gating the feature, I built the denial path
as a first-class case — the alert falls back to an on-screen message, and the rest of
the app is untouched.

## Strongest Component

The `DatabaseHelper` class is the component I would point to. It implements all four
CRUD operations plus the credential checks, scopes every query by user ID, returns
typed model objects instead of raw cursors, and closes every cursor it opens. It is
the piece that made the rest of the app straightforward: because data access was
solved in one place, the activities stayed short and readable.

## Known Limitations

Passwords are stored as entered rather than hashed. This is adequate for a local-only
course project where no data leaves the device, but it would require a hashing
function with a per-user salt before any production use.
