---
name: find-workers
description: Guides finding workers on the Instawork platform for business partners via the Instawork Partner MCP server. Use this to create shift bookings, find pricing, search locations or positions, or ask questions about finding staffing.
---

# Booking Shifts

Use the Instawork Partner MCP server to book shifts on behalf of business partners using natural language.

For connection and authentication setup, see [setup.md](setup.md).

## Available Tools

| Tool                                    | Description                                                                               |
| --------------------------------------- | ----------------------------------------------------------------------------------------- |
| `searchCompanyLocations`                | Search for company locations, optionally filtered by a search query                       |
| `getPositions`                          | Get a list of all available positions                                                     |
| `getPositionTiers`                      | Get the available skill tiers for a position                                              |
| `getPositionTemplates`                  | Get all available position templates for a company                                        |
| `getPositionTemplateDetails`            | Get the details of a position template                                                    |
| `getDefaultPositionTemplateForPosition` | Get the default position template for a position                                          |
| `getInstructionsOptions`                | Get available attire, grooming, and requirements options for a position                   |
| `getBreakLength`                        | Get applicable break rules for a given state and shift length                             |
| `getPaymentOptions`                     | Get available payment options for the partner, optionally filtered by location            |
| `getOnsiteContacts`                     | Get team members that can be added as onsite contacts                                     |
| `getBookingPricing`                     | Get initial, floor, recommended, and maximum bill rates for a booking                     |
| `getBookingTemplates`                   | Get booking templates available for rebooking, filtered by location and/or position       |
| `getBookingTemplateDetailsForRebooking` | Get pre-filled booking data from a booking template                                       |
| `getShiftgroupDetailsForRebooking`      | Get pre-filled booking data from a previous shift group                                   |
| `searchShiftgroups`                     | Search for shift groups by location, position, timeframe, and date range                  |
| `searchRosterLists`                     | Find saved roster lists, such as Favorites or a named crew                                |
| `searchRosteredPros`                    | Search for individual rostered professionals by location, name, or position               |
| `createDraftBooking`                    | Create a draft booking with positions, shifts, pricing, and roster details                |
| `createBooking`                         | Create and confirm a booking with positions, shifts, pricing, payment, and roster details |

## Booking Workflow

Use this checklist for a new booking (one position, one schedule at a time):

```
- [ ] 1. Confirm location — searchCompanyLocations
- [ ] 2. Confirm position — getPositions
- [ ] 3. Confirm skill tier when the position offers tiers — getPositionTiers
- [ ] 4. Get position instructions — getPositionTemplates → getPositionTemplateDetails
         (if no templates: getDefaultPositionTemplateForPosition)
- [ ] 5. Discuss attire / requirements if needed — getInstructionsOptions
- [ ] 6. Confirm onsite contact — getOnsiteContacts
- [ ] 7. Confirm staffing option (default: Smart assign)
         (named list: searchRosterLists; individual pros: searchRosteredPros)
- [ ] 8. Confirm shift date, start time, end time, headcount
- [ ] 9. Get break rules — getBreakLength
- [ ] 10. Get pricing — getBookingPricing → set bill rate to initial rate (unless overridden)
- [ ] 11. Confirm payment — getPaymentOptions (silent if only one; ask if two or more)
- [ ] 12. Present booking summary and wait for explicit confirmation
- [ ] 13. Create booking — createBooking (or createDraftBooking if payment is ambiguous)
```

**Rebooking from a template:** use `getBookingTemplates` → `getBookingTemplateDetailsForRebooking` to pre-fill data, then resume from step 8.

**Rebooking from a previous shift:** use `searchShiftgroups` → `getShiftgroupDetailsForRebooking` to pre-fill data, then resume from step 8.

## Booking Summary Template

Always present this before creating a booking and wait for explicit confirmation:

```
---
**Booking Summary**
- Location: [Location name]
- Position: [Position] (x[number] pros)
- Skill level: [Tier display title; omit when no tier is selected]
- Date: [Date]
- Time: [Start] – [End] ([Duration] hours)
- Break: [Length] minutes
- Rate/hour: $[Rate]
- Requested staffing: [Smart assign | Requested only — names or roster lists]
- Special requests: [Any instructions]
---
Does this look correct? Reply 'confirm' to book or let me know what to change.
```

## Rules & Constraints

### Limitations

- **Only USA and Canada are supported.** Do not attempt to book shifts in any other country.
- Long-term staffing and permanent positions are **not supported**. Explain this clearly if requested.

### One at a Time

- Book **one schedule or position at a time**. Do not attempt to create multiple positions or schedules in a single booking flow.

### Position and Instructions Setup

> **Critical:** Attire, job duties, and on-site instructions directly affect shift outcomes. Do **not** post a shift until these fields are captured (see steps 4–5 of the workflow above).

### Position Tiering

- After resolving the numeric position ID with `getPositions`, call `getPositionTiers`.
- If tiers are returned, ask the partner to choose by display name, such as Basic, Intermediate, or Expert.
- Pass the selected tier's `id` as `positions[].position_tiering` to `createBooking` or `createDraftBooking`.
- Do not show the tier ID to the partner. If no tiers are returned, omit `position_tiering`.

### Staffing Options

| Option                  | Behaviour                                                                                                                   |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Smart assign**        | Prioritises requested pros → rostered pros → best market pros → all other pros. ML-driven to balance quality and fill rate. |
| **Requested pros only** | Only opens the shift to explicitly requested professionals or members of selected roster lists.                            |
| **New pros**            | Opens the shift to professionals who have not previously worked with this partner.                                          |

Default to **Smart assign** unless the partner specifies otherwise.

- When the partner names a saved group, such as Favorites or Event Staff, call `searchRosterLists` and pass the selected list `id` in `roster.requested_roster_ids`.
- When the partner names individual professionals, call `searchRosteredPros` and pass their numeric IDs in `roster.requested_worker_ids`.
- Never send both `requested_roster_ids` and `requested_worker_ids` with values in the same booking request.
- Set `roster.is_requested_worker_only` to `true` only when the partner wants the booking restricted to the selected list members or individuals. Keep it `false` to prioritize them while allowing other pros to book.

### Pricing

- **Always set the bill rate to the initial rate** returned by `getBookingPricing`, unless the partner explicitly requests a different rate.

### Payment Options

- If there is **only one** payment option, use it silently.
- If there are **two or more**, ask which they should use, eg.: _"I see a Visa card and a Mastercard — which one should I use?"_
- If the payment option is ambiguous, save a **draft booking** rather than a confirmed booking.

### Shift Validation

Enforce the following before submitting. Explain any violation clearly:

- Shifts must start **at least 30 minutes** from the current time.
- Maximum shift duration is **12 hours**.
- Shift end time must be **after** the start time.
- Unpaid break cannot **exceed** total shift duration.
- Minimum unpaid break must comply with **legal requirements** from `getBreakLength`.

### Error Handling

- Recover gracefully when tool calls fail, information is missing, or a request is unsupported.
- Never abandon the conversation — explain what went wrong and offer a path forward.

### Other

- Do not display internal IDs like position_id or booking_id to the user during the conversation, reference everything by name.

## Reference Documents

- [Booking a Shift](https://help.instawork.com/en/collections/1892545-booking-a-shift)
- [Editing and Preparing for Shifts](https://help.instawork.com/en/collections/10902644-editing-and-preparing-for-shifts)
- [Instawork 101](https://help.instawork.com/en/collections/10902458-instawork-101)
