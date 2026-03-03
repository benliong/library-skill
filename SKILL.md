---
name: check-library
description: >
  Use this skill when the user asks to check if a book is available at a
  UK public library, wants to borrow a book from a UK library, or asks
  "which library near me has this book" with a UK postcode or UK city.
  This skill covers England, Scotland, Wales, and Northern Ireland.
user-invocable: true
argument-hint: '"[book title(s)]" near [UK postcode or city]'
allowed-tools:
  - WebFetch
  - Bash
---

# UK Library Availability Checker

You help the user find books at UK public libraries. Follow these steps exactly.

## Step 1 — Parse Input

Extract from the user's message:
- One or more **book titles** (and authors if provided)
- A **UK location**: postcode (e.g. `SW1A 2AA`, `M1 1AE`) or city name (London, Manchester, Edinburgh…)

If no location is given, ask: "What's your UK postcode or nearest city?"

If the user provides a non-UK location, politely note: "This skill is for UK public libraries only. For other countries, visit worldcat.org and enter your location."

---

## Step 2 — Book Metadata Lookup (repeat for each book)

Call Open Library Search API:

```
GET https://openlibrary.org/search.json?title={URL_ENCODED_TITLE}&fields=key,title,author_name,isbn,ebook_access,ia&limit=3
```

If an author was provided, add `&author={URL_ENCODED_AUTHOR}`.

From the results, pick the best match (exact title + correct author). Extract:
- `title`, `author_name[0]`
- Best ISBN-13 (prefer 13-digit ISBNs starting with 978)
- `ebook_access` field
- `ia` identifier (first element, if present)
- `key` (work key, e.g. `/works/OL19664718W`)

---

## Step 3 — Digital Availability Check

If the book has an `ia` identifier, call the Internet Archive availability API:

```
GET https://archive.org/services/loans/loan/?action=availability&identifier={IA_IDENTIFIER}
```

Interpret the `lending_status` response:
- `available_to_borrow: true` → **Borrowable now** (1-hour loan via Internet Archive)
- `available_to_browse: true` → **Browsable now** (14-day borrow)
- `available_to_waitlist: true` → **Join waitlist** (currently checked out)
- `is_printdisabled: true` AND all others false → **Print-disabled users only** (not available for general public)
- No `ia` field or `ebook_access: "no_ebook"` → **No digital copy available**

If borrowable or browsable, the direct link is:
```
https://openlibrary.org/works/{WORK_KEY}
```

---

## Step 4 — Physical Library Lookup

### 4a — WorldCat.org link (always include)

Generate a WorldCat.org search link for each book — covers all UK libraries:

```
https://search.worldcat.org/isbn/{ISBN}
```

Tell the user: "Click this link and enter your full postcode (e.g. `M1 1AE`) in the 'Libraries near you' box to see which local libraries hold a copy."

---

### 4b — City-specific catalog link

Detect the city or postcode area from the user's input and provide the matching direct catalog link:

| City / Region | Catalog link pattern |
|---|---|
| **London** (any London borough, or postcodes starting with E, EC, N, NW, SE, SW, W, WC) | `https://llc.ent.sirsidynix.net.uk/client/en_GB/default/search/results?qu={URL_ENCODED_TITLE}&te=ILS` — covers Westminster, Camden, Islington, Hackney, Lambeth, and other borough consortium libraries |
| **Manchester** | `https://manchester.spydus.co.uk/cgi-bin/spydus.exe/SRQB/WPAC/BIBENQ?WBCLEF=TITL&ENTRY={URL_ENCODED_TITLE}` |
| **Birmingham** | `https://birmingham.spydus.co.uk/cgi-bin/spydus.exe/SRQB/WPAC/BIBENQ?WBCLEF=TITL&ENTRY={URL_ENCODED_TITLE}` |
| **Leeds** | `https://leeds.spydus.co.uk/cgi-bin/spydus.exe/SRQB/WPAC/BIBENQ?WBCLEF=TITL&ENTRY={URL_ENCODED_TITLE}` |
| **Bristol / South West England** (Libraries West) | `https://www.librarieswest.org.uk/search/results?query={URL_ENCODED_TITLE}` |
| **Edinburgh** | `https://yourlibrary.edinburgh.gov.uk/web/arena/search#/?query={URL_ENCODED_TITLE}` |
| **Glasgow** | `https://encore.glasgow.gov.uk/iii/encore/search/C__S{URL_ENCODED_TITLE}` |
| **Liverpool** | `https://liverpool.spydus.co.uk/cgi-bin/spydus.exe/SRQB/WPAC/BIBENQ?WBCLEF=TITL&ENTRY={URL_ENCODED_TITLE}` |
| **Cardiff / Wales** | `https://cardiff.spydus.co.uk/cgi-bin/spydus.exe/SRQB/WPAC/BIBENQ?WBCLEF=TITL&ENTRY={URL_ENCODED_TITLE}` |
| **Northern Ireland** | `https://www.librariesni.org.uk/Search?term={URL_ENCODED_TITLE}` |
| **Any other UK city or postcode** | Skip — WorldCat.org (step 4a) is sufficient |

If the city cannot be matched above, omit this link and note: "For your specific library, use the WorldCat link above and enter your postcode."

---

### 4c — Libby / OverDrive (always include)

Provide the Libby link for digital borrowing via library card — works across all UK library authorities:

```
https://libbyapp.com/search/
```

Note: In Libby, search for your local council or library authority by name to check eBook and audiobook availability with your library card.

---

## Step 5 — Present Results

Format your output clearly. For each book:

```
## 📚 [Book Title]
**Author:** [Author Name]
**ISBN:** [ISBN-13]

### Digital Availability (Internet Archive)
[Status: Borrowable now / Browsable now / Join waitlist / Print-disabled users only / Not available]
[If available: 🔗 Borrow on Open Library: https://openlibrary.org/works/...]

### Find a Physical Copy at a UK Library
🔗 WorldCat — enter your postcode: https://search.worldcat.org/isbn/...
🔗 [City] Libraries catalog: https://...   (omit this line if city not matched above)

### Borrow Digitally via Your UK Library Card
🔗 Search Libby/OverDrive: https://libbyapp.com/search/
(Sign in with your library card to check eBook/audiobook availability)
```

---

## Important Notes

- **This skill is UK-only.** If a non-UK location is given, say so and redirect to worldcat.org.
- **Always use ISBN-13** when generating WorldCat.org links (more reliable than ISBN-10).
- If Open Library returns no match for a title, say so and suggest the user check WorldCat.org directly.
- `print-disabled` status means restricted to users with print disabilities — do NOT describe it as generally borrowable.
- WorldCat.org covers all UK libraries and is the most reliable physical availability resource.
- Libby/OverDrive uses the user's public library card — it is separate from Internet Archive.
