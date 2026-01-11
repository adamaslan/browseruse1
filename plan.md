# Plan: Improve Art Opening Detection

## Current Issues Identified

1. **Vague Task Prompt**: The current task prompt is generic and doesn't specifically guide the agent to look for "opening receptions" vs general exhibitions
2. **No Distinction Between Opening vs Exhibition**: The agent conflates "exhibition dates" with "opening reception" - these are different things
3. **Missing Event Type Classification**: No way to filter for actual opening night events vs ongoing shows
4. **Inconsistent Date Parsing**: Dates like "Jan - Jun 2025" are ambiguous - is that the opening or the run?
5. **No Opening-Specific Keywords**: Agent doesn't prioritize looking for "opening reception", "vernissage", "first Thursday", etc.
6. **Brooklyn Art Cave False Negative**: The agent missed events possibly because it didn't look for the right patterns

## Proposed Changes

### 1. Enhance the Task Prompt with Opening-Specific Instructions

Update `make_agent()` to include explicit guidance:
- Prioritize finding **opening receptions** (the specific event date/time when the show opens)
- Look for keywords: "opening reception", "vernissage", "first Thursday", "opening night", "artist reception"
- Distinguish between: `opening_date` (the one-time event) vs `exhibition_dates` (the run of the show)

### 2. Add Structured Schema for Better Extraction

Define explicit fields:
```python
{
    "gallery_name": str,
    "address": str,
    "exhibition_title": str,
    "opening_reception_date": str,  # The specific opening event date
    "opening_reception_time": str,  # The specific time (e.g., "6-8PM")
    "exhibition_start_date": str,   # When exhibition starts
    "exhibition_end_date": str,     # When exhibition ends
    "artists": list[str],
    "event_type": str,  # "opening_reception", "closing_reception", "artist_talk", "group_show", etc.
    "is_free": bool,
    "rsvp_required": bool,
    "source_url": str
}
```

### 3. Improve the Prompt with Gallery-Specific Patterns

Many galleries have predictable patterns:
- First Thursdays (Bushwick Open Studios)
- Saturday openings
- Evening receptions (typically 6-9PM)

Add hints to the agent:
```
Look for opening reception patterns:
- Time ranges like "6-8PM", "7-9PM"
- Days like "Thursday", "Saturday"
- Phrases containing "reception", "opening", "vernissage"
- Event pages, calendar entries, or "upcoming" sections
```

### 4. Add Fallback Navigation Strategies

When the primary URL fails or shows no events:
- Check `/exhibitions`, `/events`, `/upcoming`, `/calendar` paths
- Look for "Current" and "Upcoming" sections
- Check social media links (Instagram often has opening announcements)

### 5. Add Date Validation Logic

```python
from datetime import datetime, timedelta

def is_upcoming_opening(date_str: str) -> bool:
    """Validate that the opening is in the future (within next 3 months)."""
    # Parse various date formats
    # Return True only if it's a future date
```

### 6. Create Opening Detection Keywords List

```python
OPENING_KEYWORDS = [
    "opening reception",
    "vernissage",
    "artist reception",
    "opening night",
    "first thursday",
    "opening party",
    "preview",
    "private view",
    "launch party",
]

EXCLUDE_KEYWORDS = [
    "closed",
    "past exhibition",
    "archive",
    "previous",
]
```

## Implementation Steps

1. **Update `make_agent()` function** with improved task prompt
2. **Add structured output schema** as a parameter to the Agent
3. **Create helper functions** for date validation and keyword detection
4. **Update `append_structured()`** to validate and normalize data
5. **Add retry logic** for galleries that return no results
6. **Add logging** to track which galleries found openings vs not

## Updated Task Prompt (Draft)

```python
task = f"""
You are searching for ART OPENING RECEPTIONS at Brooklyn galleries.

IMPORTANT: An "opening reception" is the specific one-time event when a new exhibition
launches - typically an evening event with wine/refreshments where guests meet the artist.
This is DIFFERENT from the exhibition dates (when the show runs).

For each gallery, find:
1. The OPENING RECEPTION date and time (e.g., "Saturday, January 18, 6-8PM")
2. The exhibition title
3. The artist(s) name(s)
4. The gallery address

LOOK FOR these keywords: "opening reception", "vernissage", "artist reception",
"opening night", "first Thursday", "preview"

NAVIGATION TIPS:
- Check "Exhibitions", "Events", "Upcoming", or "Calendar" pages
- Look for dates in the NEXT 3 MONTHS (current date: {datetime.now().strftime('%B %d, %Y')})
- If a page shows "current exhibition" with no future events, note that

OUTPUT FORMAT (JSON array):
{{
    "gallery_name": "...",
    "exhibition_title": "...",
    "opening_reception_date": "...",  // MUST be a specific date like "January 18, 2025"
    "opening_reception_time": "...",  // Like "6-8PM"
    "artists": ["..."],
    "address": "...",
    "source_url": "..."
}}

If no upcoming opening reception is found, return:
{{"gallery_name": "...", "status": "no_upcoming_openings", "source_url": "..."}}

Galleries to check: {', '.join(remaining_sources)}
"""
```

## Success Metrics

- Correctly identifies opening reception dates (not just exhibition runs)
- Extracts specific times (e.g., "6-8PM" not just "Not available")
- Reduces false negatives (galleries marked as "no events" that actually have openings)
- Consistent date format in output
