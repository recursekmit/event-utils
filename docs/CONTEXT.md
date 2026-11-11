# Idea and problem statement

## Problem

Clubs and institutions run the same kinds of events every year: hackathons, recruitment drives, workshops, and fests. Each time, someone builds a one-off website and a pile of forms and spreadsheets for registration, teams, submissions, and judging. That work is thrown away after the event, the next organizers start again from nothing, and participant data ends up scattered across tools nobody maintains.

## Idea

event-utils is one reusable, open-source, self-hosted platform that runs all of an institution's events from a single installation, so a new event is configured, not rebuilt.

## Who it is for

- **Organizers** set up an event, give it its own look, and choose which features it uses.
- **Participants** find an event, register, form teams, and submit their work.
- **Judges and staff** score submissions and help run the event on the day.

## What an event needs

- Its own public page and branding.
- Registration and team formation.
- Problem statements and submissions (for hackathons).
- Judging and published results.
- Day-of and follow-up tasks such as check-in and certificates.

Not every event needs all of these; organizers pick what applies.

## Plug and play

This is the main idea. Every feature is a switch, and the event moderator can turn it on or off for their event at any time without a developer and without rebuilding anything.

The moderator should be able to do this in whichever way suits them:

- from the admin console,
- from a command-line tool, or
- by editing a JSON or YAML file.

All three must give the same result. A feature that is off should simply not be there for that event; a feature that is on should work without extra setup beyond its own settings.

This is a requirement on what the moderator experiences, not a decision about how the platform is built.

## Status of this document

This is the only document describing the project. There is no code, architecture, roadmap, or delivery sequence.
