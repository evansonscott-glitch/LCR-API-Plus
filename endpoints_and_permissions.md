# LCR-API-Plus Endpoints and Permissions Analysis

Based on the code review of the `LCR-API-Plus` repository, here is a breakdown of the available endpoints, how authentication works, and what we'll need to test access for an Area-level calling.

## Authentication Flow

The API uses a Selenium-based web scraping approach to authenticate because the Church's LCR system does not offer a public-facing API.

1.  **Selenium WebDriver**: It spins up a headless Chrome browser.
2.  **Login Page**: It navigates to `lcr.churchofjesuschrist.org`.
3.  **Credential Entry**: It inputs the provided username and password into the standard login form.
4.  **Cookie Extraction**: Once logged in, it waits for the main header to load, then extracts the `appSession` cookie.
5.  **Requests Session**: It passes this cookie into a standard Python `requests.Session()` object, which is then used for all subsequent, faster API calls directly to the backend endpoints.

## Available Endpoints & Required Data

The following methods are mapped directly to LCR backend endpoints. Most of them require a `unitNumber` parameter, which is passed in during API initialization.

| Method | Endpoint | Description | Expected Permissions |
| :--- | :--- | :--- | :--- |
| `birthday_list(month, months)` | `/api/report/birthday-list` | Gets a list of birthdays for a given month range. | Typically Ward/Stake leadership and clerks. |
| `members_moved_in(months)` | `/api/report/members-moved-in/unit/{unit_number}/{months}` | Gets members moved into the unit. | Ward/Stake leadership and clerks. |
| `members_moved_out(months)` | `/api/report/members-moved-out/unit/{unit_number}/{months}` | Gets members moved out of the unit. | Ward/Stake leadership and clerks. |
| `member_list()` | `/api/umlu/report/member-list` | Gets the full member directory for the unit. | Ward/Stake leadership and clerks. |
| `individual_photo(member_id)` | `/individual-photo/{member_id}` | Gets the photo for a specific member. | Same access as the member list. |
| `callings()` | `/services/orgs/sub-orgs-with-callings` | Gets callings for all organizations. | Ward/Stake leadership and clerks. |
| `ministering(organization)` | `/api/umlu/v1/ministering/data-full` | Gets ministering data (requires "EQ" or "RS" param). | EQ/RS Presidencies, Bishopric, Clerks. |
| `recommend_status()` | `/api/recommend/recommend-status` | Gets temple recommend statuses. | Bishopric, Stake Presidency, Executive Secretaries, Clerks. |
| `quarterly_report(unit_number, quarter, year)`| `/api/report/quarterly-report` | Gets the detailed quarterly report. | Bishopric, Stake Presidency, Clerks. |
| `access_table()` | `/services/access-table` | Gets the user's role ID and allowed methods. | Open to any authenticated user to check their own access. |

## Testing with an Area-Level Calling

Because Scott has an Area-level calling (Utah Area) rather than a Ward or Stake calling, his permissions in LCR will likely be different.

**The primary challenge:** Most of these endpoints explicitly require a `unitNumber` (Ward or Stake number). Area-level callings often have broader access, but the API calls themselves are structured to query *specific* units.

**What we need to test:**
1.  **Can he query specific wards/stakes?** We need to know if his Area access allows him to pass a specific Ward or Stake `unitNumber` into these endpoints and successfully retrieve data, or if he gets a `403 Forbidden` error.
2.  **Does he have an Area `unitNumber`?** We need to determine if there is a specific "Area" unit number that can be passed to these endpoints, and if the LCR backend supports returning aggregated Area data for things like `members_moved_in`.
3.  **The `access_table` endpoint**: The most valuable first step will be to hit the `access_table()` endpoint with his credentials. This should return a JSON object detailing exactly what his role ID is and what data access he has been granted by the system.

## Setup Requirements for Testing

To proceed with testing without hardcoding credentials, I have created an `.env.example` file in the repository.

Scott will need to provide:
1.  His LDS Username
2.  His LDS Password
3.  A target `unit_number` to test against (e.g., his home ward, a stake in his area, or his area number if one exists).

Once we have these securely loaded into a local `.env` file (which is ignored by Git), we can write a quick test script to hit the `access_table()` and a few benign endpoints like `birthday_list()` to map out his exact permission boundaries.
