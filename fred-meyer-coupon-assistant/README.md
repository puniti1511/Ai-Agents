# Fred Meyer Digital Coupon Assistant — Business Case & Scope

## Overview

This is a personal AI agent built on Claude that checks Fred Meyer's digital coupons against a grocery list on request, clips the relevant ones with the user's confirmation, and reports back what was clipped and what it's worth. It exists to remove the manual work of scrolling through hundreds of live coupons every time a grocery run comes up.

**MVP capabilities:** read the current grocery list · search Fred Meyer's live coupon catalog for matches · surface matches for the user to review · **clip confirmed coupons directly on the account** · report clipped items, expirations, and total savings in chat.

## Impact

This saves roughly 20–25 minutes before every Fred Meyer trip — the time it used to take to manually scroll the coupon catalog, cross-check it against a shopping list, and clip everything by hand. That time is now a single ad hoc request with a chat summary at the end.

## The problem

Fred Meyer's digital coupon catalog typically holds 300+ live offers at any given time, spanning everything from weekly produce deals to household goods. Working out which of those match an actual shopping list, and clipping them before they expire, is the kind of task that's easy to skip under time pressure — and a meaningful share of the best offers ("Weekly Digital Deals") expire the same day they're first seen, so skipping the check has a real cost in missed savings.

## Scope

On request — inside a conversation with Claude, not on a fixed schedule — the assistant reads the user's current grocery list from a private, local spreadsheet that is never uploaded or shared anywhere. It opens Fred Meyer's site in the user's own already-authenticated browser session and searches the site's coupon catalog for matches against each grocery item, using judgment-based matching rather than exact text matching (for example, recognizing that a coupon titled "Honeycrisp or SugarBee Apples" matches a grocery list entry of "Apples Gala or Honeycrisp"). Matches are presented back to the user; on confirmation, the assistant clips the selected coupons directly on the account — a simple, reversible click, not a purchase — and returns a savings summary in the chat.

## Out of scope

Scheduled or fully unattended runs are out of scope by design (a scheduled version was piloted and then retired — see Design decisions below). Email delivery is out of scope; results are shown in the chat only, per the user's preference. Automated login or credential entry is permanently out of scope: the assistant will not type a password into any site under any circumstance, so it depends on the browser already being signed in. No purchasing, checkout, or payment action of any kind is in scope.

## How it works

A run starts with the assistant re-reading the grocery list from the source spreadsheet, since the list changes between runs and a stale copy would produce wrong matches. It then opens Fred Meyer's Digital Coupons page in the user's browser and checks that the account is already signed in — if it isn't, the assistant stops and asks the user to sign in themselves rather than entering credentials on their behalf. Each grocery item is searched against the live coupon catalog, and any reasonable matches are collected along with their discount amount and expiration date. The assistant flags anything expiring the same day, since those are the offers a user is most likely to lose by waiting. It then shows the full match list in chat and clips only the coupons the user confirms, rather than clipping everything automatically.

## Output format

Every run ends with a summary in this shape:

| Product | Price with Coupon / Savings | Coupon Expires | Notes |
|---|---|---|---|
| Honeycrisp or SugarBee Apples | $2.49/lb | Aug 25 (today) | Already clipped |
| Dove Bodywash or Bar Soap | Save $2.00 | Sep 1 | Clipped this run |
| 4X Points on Purchases (Fri 8/28 only) | Points multiplier, excludes gift cards | Aug 28 | Clipped this run |

**Total estimated savings this run:** sum of all newly clipped coupon values, called out at the end of the chat summary, alongside a separate note for any item expiring same-day so it doesn't get missed.

## Design decisions

The assistant clips only with the user's confirmation each time, rather than acting fully autonomously. This keeps a person in the loop before any action that changes something on a real retail account, and avoids the pattern of automated, unattended account actions that tends to trigger bot detection on retail sites.

A scheduled version (Wednesday and Friday mornings) was built and then deliberately removed in favor of ad hoc requests. A scheduled cloud run needs the user's computer and browser bound and reachable at the exact moment it fires, which is unreliable in practice, and logging into a retail account with nobody present carries more risk than doing the same thing with the user there to catch anything unexpected.

Credential handling was never in scope. The assistant is not able to enter a password into a web form under any circumstance, including the user's own accounts, so every run depends entirely on the browser's existing signed-in session. That keeps a real secret — the account password — outside of any automated flow entirely, rather than depending on careful handling of it.

The source grocery list and the account credentials it contains are intentionally excluded from this repository. Only the process and its outcomes are documented here.

## Risks and limitations

Coupon matching is keyword- and judgment-based against Fred Meyer's own site search, not exhaustive machine-verified matching — a coupon worded very differently from the grocery list entry could be missed, and a loosely related result is sometimes surfaced for the user to judge rather than assumed to be correct. The assistant depends on Fred Meyer's website structure, which can change without notice and break the flow. Because login is never automated, a run that starts after the browser session has expired will pause and ask the user to sign back in rather than completing on its own.

## Status

Working, in active ad hoc use as of August 2026.
