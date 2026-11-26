You clean rows of a personal transactions spreadsheet. Every row has exactly these keys: id, date, vendor, amount, category, note.

Target format, exactly like these clean rows:
{sample}

Convert every row to the target format using the rules and tables below. The tables are complete: every vendor and every category word you will meet is in them.

DATES - always output YYYY-MM-DD, and every date is in the year 2026. Inputs arrive in one of these forms:
- 03/14/2026 or 3/14/26: month/day/year (a two-digit year means 2026)
- 14/03/2026: day/month/year. This form is used only when the first number is over 12; if the first number is 12 or less, the form is month/day/year
- 2026.03.14: year.month.day
- 14 Mar 2026: day, short month name, year
- March 14, 2026: long month name, day, year
- 20260314: year, month and day run together

AMOUNTS - always a plain number with exactly two decimals: no $ sign, no USD, no commas, no spaces. $1,234.56, USD 12.34, 12.34 USD and $ 12.34 all become digits only. A one-decimal amount gains a trailing zero: 12.3 becomes 12.30. A three-decimal amount keeps two: 12.340 becomes 12.34.

VENDORS AND CATEGORIES - the vendor must be spelled exactly as in this table (fix letter case, extra spaces, tabs and one swapped pair of letters to match it), and the vendor decides the category:
- groceries: Whole Harvest Market, Corner Grocer, Fresh Fields, Daily Pantry
- dining: Blue Fork Bistro, Taco Lane, Noodle House, Sunrise Diner
- transport: Metro Transit, City Cab Co, Rapid Rail, Blue Line Scooters
- utilities: Northside Power, Clearwater Utility, Fiber Net, Metro Gas
- rent: Maple Court Apartments, Riverside Lofts, Oak Street Property
- entertainment: Starlight Cinema, Vinyl Vault, Game Den, Riverside Theater
- health: Greenleaf Pharmacy, Summit Dental, Wellness Clinic, Clear Vision Optics
- office: Paper Trail Supply, Desk Depot, Print Hub, Cable Depot

CATEGORY WORDS - the category is exactly one of groceries, dining, transport, utilities, rent, entertainment, health, office. Rewrite any of these forms to the word on the left:
- groceries: Groceries, GROCERY, food shopping, supermarket
- dining: Dining, restaurants, Food & Dining, eating out
- transport: Transport, transportation, Travel, commute
- utilities: Utilities, bills, UTILITY, power/water
- rent: Rent, housing, RENT, apartment
- entertainment: Entertainment, fun, leisure, Movies & Music
- health: Health, medical, Healthcare, pharmacy
- office: Office, office supplies, work, Supplies
If the category is empty, set it from the vendor table above. If amount and category have been swapped (the amount field holds a category word and the category field holds a number), swap them back, then format both by the rules above.

NOTES - only tidy them: trim leading and trailing spaces, collapse repeated spaces to one, remove curly quotes. Never reword a note. An empty note stays empty.

DUPLICATES AND JUNK - add "drop": true ONLY to:
- a row whose id is empty (a repeated header row, a TOTAL row, a row of dashes, a blank row), or
- one copy of two rows in this batch that share the same id: keep the copy whose vendor matches the table spelling and drop the other copy; if the copies are identical, keep the first and drop the second.
Every other row is real: fix it, never drop it, never change its id, never invent rows. Return every row you were given, in the same order, with every value as a string. Change nothing that is already correct.

Reply with only the JSON array.
