# New Listing Alert Matcher (n8n)

An n8n automation for real estate agents and agencies. Every 3 hours it checks a Google Sheet for new property listings, matches them against each buyer's saved search criteria, and sends the matching buyers a professionally formatted email alert.

<img width="1780" height="430" alt="re-workflow" src="https://github.com/user-attachments/assets/7750b17d-1ab3-4ca1-baf4-1f3a26692252" />


## The problem it solves

An agent usually manages many buyers, and every buyer wants something different (city, property type, budget, bedrooms). When a new listing comes in, someone has to remember which buyers it suits and email them by hand. That is slow, easy to get wrong, and the agent who alerts first usually wins the deal.

This workflow does the matching and the emailing automatically, so every buyer hears about a suitable property as soon as it is added.

## How it works

1. **Every 3 Hours** - a schedule trigger starts the workflow.
2. **Get New Listings** - reads all rows from the `Listings` tab of a Google Sheet.
3. **Get Buyer Criteria** - reads all rows from the `Buyer Criteria` tab (set to *Execute Once*, so buyers are read a single time).
4. **Match Listings to Buyers** - a Code node that:
   - keeps only listings with `status = Active` and `notified` not equal to `Yes`,
   - keeps only buyers whose `active` is not `No`,
   - compares each listing against each buyer on city, property type, price range and minimum bedrooms (an empty buyer field means "any"),
   - builds an HTML email for every match.
5. **Send Match Email** - sends the email from Gmail (Email Type: HTML).
6. **Mark Listing Notified** - sets `notified = Yes` on the listing row so it is never sent twice.

## Requirements

- An [n8n](https://n8n.io) instance (cloud or self-hosted)
- A Google account with Google Sheets access
- A Gmail account to send the alerts from

## Google Sheet setup

Create one spreadsheet with two tabs. Header names must match exactly, with no extra spaces.

**Tab 1: `Listings`**

| listingId | address | city | propertyType | price | bedrooms | listingUrl | status | addedAt | notified |
|---|---|---|---|---|---|---|---|---|---|
| L201 | 56 Cactus Ridge Road | Phoenix | House | 365000 | 3 | https://example.com/l201 | Active | 2026-10-17 | |

- `listingId` must be unique, because the last node uses it to find the row to update.
- Leave `notified` empty for new listings.
- Set `status` to `Sold` or `Off Market` when a property is no longer available.

<img width="1920" height="830" alt="re-listings" src="https://github.com/user-attachments/assets/b0cfa2f2-651a-4a41-891e-2c18731b35e3" />

**Tab 2: `Buyer Criteria`**

| buyerName | buyerEmail | city | propertyType | minPrice | maxPrice | minBedrooms | active |
|---|---|---|---|---|---|---|---|
| Ayesha | buyer@example.com | Phoenix | House | 300000 | 400000 | 3 | Yes |

- Any criteria field can be left empty to mean "any".
- Set `active` to `No` to pause alerts for a buyer.

<img width="1920" height="823" alt="re-buyers" src="https://github.com/user-attachments/assets/020593d7-d8d6-43fc-9494-9f8082109fc1" />



## Installation

1. In n8n choose **Import from File** and select `workflow.json`.
2. Open **Get New Listings**, **Get Buyer Criteria** and **Mark Listing Notified**. In each one, select your Google Sheets credential and your spreadsheet (by ID).
3. In **Mark Listing Notified**, use manual column mapping with:
   - match column: `listingId`
   - `listingId` = `{{ $('Match Listings to Buyers').item.json.listingId }}`
   - `notified` = `Yes`
4. Open **Send Match Email**, select your Gmail credential and set **Email Type** to **HTML**. Optionally turn off *Append n8n Attribution*.
5. Open **Match Listings to Buyers** and set the `AGENCY` constant at the top of the code to your business name.
6. Activate the workflow.

## Testing

1. Add a test listing (`status = Active`, `notified` empty) and a buyer whose criteria match it.
2. Click **Execute Workflow**.
3. Check that the email arrives and that `notified` becomes `Yes` in the sheet.
4. Run it again. No second email should be sent.


<img width="1527" height="671" alt="re-email(1)" src="https://github.com/user-attachments/assets/782dc736-6a19-4c5e-b144-005071efa233" />

<img width="1405" height="745" alt="re-email(2)" src="https://github.com/user-attachments/assets/bc778164-1098-4ef3-94e3-59cab9175859" />


## Customisation

- **More criteria** (bathrooms, area, and so on): edit the matching logic in the **Match Listings to Buyers** code node.
- **Email design and wording**: edit the HTML template in the same node.
- **Currency**: change the `money()` helper in the code node.
- **Schedule**: change the interval in the **Every 3 Hours** node.

## Known limitations

- Listings are added to the sheet manually. An MLS or listing-site feed could be connected to import them automatically.
- A listing is marked as notified after it is sent, so a buyer added later will not receive older listings automatically.
- A listing that matches several buyers sends a separate email to each of them, but is marked as notified once.

## Tech stack

n8n, Google Sheets, Gmail, JavaScript (Code node)
