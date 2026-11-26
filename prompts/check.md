You are the final checker for a cleaned transactions spreadsheet. Columns: id, date, vendor, amount, category, note. A reference sample of clean rows is appended below; rows that already match it in form must come back unchanged.

Re-check every row against these rules and fix what is still wrong:
- date: YYYY-MM-DD, always in 2026. A slash date whose first number is over 12 is day/month/year; otherwise it is month/day/year, and a two-digit year means 2026. Also convert the 2026.03.14, 14 Mar 2026, March 14, 2026 and 20260314 forms.
- amount: a plain number with exactly two decimals, no $ sign, no USD, no commas. 12.3 becomes 12.30 and 12.340 becomes 12.34.
- vendor: spelled exactly as in this table, which also decides the category:
  groceries: Whole Harvest Market, Corner Grocer, Fresh Fields, Daily Pantry
  dining: Blue Fork Bistro, Taco Lane, Noodle House, Sunrise Diner
  transport: Metro Transit, City Cab Co, Rapid Rail, Blue Line Scooters
  utilities: Northside Power, Clearwater Utility, Fiber Net, Metro Gas
  rent: Maple Court Apartments, Riverside Lofts, Oak Street Property
  entertainment: Starlight Cinema, Vinyl Vault, Game Den, Riverside Theater
  health: Greenleaf Pharmacy, Summit Dental, Wellness Clinic, Clear Vision Optics
  office: Paper Trail Supply, Desk Depot, Print Hub, Cable Depot
- category: exactly one of groceries, dining, transport, utilities, rent, entertainment, health, office. Rewrite synonyms (food shopping, restaurants, Travel, bills, housing, fun, medical, work and their capitalised forms) to the table word. If the category is empty or disagrees with the vendor, use the vendor's category from the table.
- note: trim spaces, collapse repeated spaces to one, remove curly quotes. Never reword a note; an empty note stays empty.
- If amount and category are swapped, swap them back.

Add "flag": true to a row you had to guess at, so it gets one more pass. Add "drop": true only to a row whose id is empty, or to one copy of two rows in this batch that share the same id (keep the copy whose vendor matches the table spelling; if the copies are identical, keep the first).
Every row with an id is real: never drop it for any other reason, never change an id. Return every row, in order, as a JSON array only, no commentary.
