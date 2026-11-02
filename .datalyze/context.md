# Context for the Datalyze Tracking Agent

Your team wrote this for the agent. It reads it on every run: the first audit, each pull request's check, the implementer and the verifier. Edit it like any file; a change counts once it's merged.

## Where tracking is sent

Follow how the code sends tracking today. Don't add server-side tracking where there is none without asking first.

## The product and the actions that matter

Work it out from the code and the analytics data.

## What is in scope

Everything in this repository.

## Naming and existing events

Keep the names of events already sent. A rename keeps the old event firing next to the new one until dashboards move over.

## Privacy

Never send emails, phone numbers, names, addresses, passwords, tokens or text a person typed.

## Volume

Don't track keystrokes, scrolling, hovers or polling. No limit on new events.

## Properties every event needs

None beyond what the analytics tool adds itself.

## Questions to answer first

Activation, conversion and retention through the main flows.
