# Security review

The following issues were identified in the application. Reproduce them only
in a local or otherwise authorized instance. Each requires a signed-in account.

## 1. Users can modify other users' listings

**Severity:** Medium  
**Affected code:** `market/views.py` (`listing_edit`, `listing_sold`,
`listing_delete`); `market/forms.py` (`ListingForm`)

The listing edit, mark-sold, and remove views fetch a listing by ID without
checking that the signed-in user owns it. The edit form also includes
`seller_note`, which is not displayed on the public listing page.

**How to trigger:**

1. Sign in as an account that does not own a listing (or register a new
   account).
2. Open `/listings/1/edit/` for an existing listing. The edit page exposes its
   seller note.
3. Submit changes to modify another user's listing. The same missing ownership
   check affects the POST endpoints `/listings/1/sold/` and
   `/listings/1/delete/`; submitting those changes the listing's status.

**Impact:** An authenticated user can read private seller notes and tamper with
or remove another user's listing.

## 2. Users can access other users' message threads

**Severity:** Medium  
**Affected code:** `market/views.py` (`thread_detail`)

The thread view checks only that a user is signed in. It does not verify that
they are the thread's buyer or the listing's owner before displaying messages
or accepting a new one.

**How to trigger:**

1. Sign in as an account that is neither participant in an existing thread.
2. Open `/threads/1/` (or another existing thread ID) to read its messages.
3. Submit a POST to the same URL with a non-empty `body` field to add a message
   to the thread as the signed-in account.

**Impact:** An authenticated nonparticipant can read private conversation
content and post messages into it.

## 3. Users can grant themselves department-desk access

**Severity:** Medium  
**Affected code:** `market/forms.py` (`ProfileForm`); `market/views.py`
(`profile`, `desk`)

The profile form writes the `is_staff` field on the user, although it is
presented as a department-board display preference. The desk view uses that
same field to authorize access to department information.

**How to trigger:**

1. Sign in as a non-staff account and open `/profile/`.
2. Select “Show my name on the department board” and save the profile.
3. Open `/desk/`; the page now displays the department hold-closet location and
   reimbursement code.

**Impact:** Any authenticated user can promote their account to staff through
the profile form and access information intended for department board
accounts.
