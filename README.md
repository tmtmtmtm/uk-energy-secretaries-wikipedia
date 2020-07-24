Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the
[Secretary of State for Energy](https://www.wikidata.org/wiki/Q19968043)
had all of the information expected.

Step 2: Tracking page
=====================

I created a new PositionHolderHistory table. The initial version at
https://www.wikidata.org/w/index.php?title=Talk:Q19968043&oldid=1237735506
had 3 dated memberships, 5 undated ones, and 9 warnings.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js) 
to configure the Item ID and source URL.

Step 4: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Step 5: Scrape
==============

Comparison/source = [Department of Energy (United Kingdom)](https://en.wikipedia.org/wiki/Department_of_Energy_(United_Kingdom))

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

Worked straightaway, other than an extra row at the bottom, which I
simply removed from the output.

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

No new additions required.

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

 added as https://tools.wmflabs.org/editgroups/b/wikibase-cli/02fcca3650d5b

Plus two date precision improvements:

    Q335038$E7CD4675-94DD-4176-AF81-2B31A96F40EB P580 1974-01-01 1974-01-08
    Q335038$E7CD4675-94DD-4176-AF81-2B31A96F40EB P582 1974-03-01 1974-03-04

    pbpaste | wd uq --batch --summary "Add higher precision dates from (wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

Step 8: Refresh the Tracking Page
=================================

Final version at https://www.wikidata.org/w/index.php?title=Talk:Q19968043&oldid=1237771905
