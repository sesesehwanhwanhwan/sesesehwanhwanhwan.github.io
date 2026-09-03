SEIHWAN JUNG — PERSONAL ACADEMIC WEBSITE
=========================================

FILES
-----
  index.html                      Home (hero banner, interests, projects,
                                  education, awards, activities)
  cv.html                         CV page — download button + inline preview
  research-passive-abd-hand.html  Project detail page, filled in as a worked
                                  example you can copy the style from
  research-project-2.html         BLANK TEMPLATE — duplicate this per project
  style.css                       All the design. Colors live in the first
                                  block at the top; change them there and the
                                  whole site re-themes.
  robots.txt, sitemap.xml         So Google can find and index the site
  media/                          Photos, charts, demo video

  You still need to add:
    cv.pdf             -> next to index.html; the CV page picks it up
    media/profile.jpg  -> your photo, fills the circle on the home page


═══════════════════════════════════════════════════════════════
STEP 1 — PUT IT ONLINE (you need a URL before Google can find it)
═══════════════════════════════════════════════════════════════

EASIEST — Netlify, no account needed to start:
  1. Go to  https://app.netlify.com/drop
  2. Drag this whole folder onto the page.
  3. You instantly get a URL like  https://random-name.netlify.app
  4. Make a free account to rename it to something like
     seihwan-jung.netlify.app

BEST FOR A CLEAN URL — GitHub Pages:
  1. Make a GitHub account.
  2. Create a repository named exactly:  <your-username>.github.io
  3. Upload every file in this folder (github.com > "Add file" >
     "Upload files" — drag and drop works). Keep media/ as a folder.
  4. Settings > Pages > Source: main branch, / (root). Save.
  5. Live a minute later at  https://<your-username>.github.io

Best of all: buy a domain (about 15,000 KRW/year) like seihwanjung.com
and point it at either host. A personal-name domain ranks fastest for
your own name on Google.


═══════════════════════════════════════════════════════════════
STEP 2 — MAKE YOUR SITE SEARCHABLE ON GOOGLE
═══════════════════════════════════════════════════════════════

2a. REPLACE THE PLACEHOLDER URL
    Every page contains the text  REPLACE-WITH-YOUR-SITE-URL
    Open each file in a text editor and Find & Replace it with your real
    address (no trailing slash), e.g.  seihwan-jung.github.io

    Files to fix:  index.html, cv.html,
                   research-passive-abd-hand.html,
                   research-project-2.html,
                   robots.txt, sitemap.xml

    (In VS Code you can do this across all files at once:
     Edit > Replace in Files, Ctrl+Shift+H.)

    This matters — those tags are what Google reads to know the page
    title, description, and that the site is about a PERSON named
    Seihwan Jung. That's what makes your name match the search.

2b. TELL GOOGLE THE SITE EXISTS
    1. Go to  https://search.google.com/search-console
    2. Sign in and click "Add property" > "URL prefix" > paste your URL.
    3. Verify ownership. The easiest method for GitHub Pages/Netlify is
       "HTML tag": Google gives you a line like
         <meta name="google-site-verification" content="....">
       Paste it into index.html just below the <title> line, re-upload,
       then click Verify.
    4. Once verified, open "Sitemaps" in the left menu and submit:
         sitemap.xml
    5. Also use "URL Inspection" at the top, paste your homepage URL,
       and click "Request indexing".

2c. HELP IT ALONG
    Google trusts pages that other sites link to. Put your site URL in:
      - your GitHub profile
      - your LinkedIn profile
      - your email signature
      - the lab's member page, if they'll add it
    Each of those makes Google index you faster.

HOW LONG DOES IT TAKE?
    Usually a few days to two weeks before "Seihwan Jung" turns up your
    site. To check whether Google has indexed you at all, search:
        site:your-url-here
    If results appear, you're indexed. Ranking for a common name takes
    longer — a personal domain and a few incoming links are the two
    things that speed it up most.


═══════════════════════════════════════════════════════════════
STEP 3 — FILL IN YOUR CONTENT
═══════════════════════════════════════════════════════════════

Every spot that needs your input has a YELLOW DASHED BOX on the page.
Open the .html file in a text editor, write your content, then delete
the whole <div class="todo">...</div> block. The list:

  index.html
    - your real enrollment years (currently "20XX")
    - what you actually did at BRL — replace the three bullet points
    - the second project card (or delete it if you only have one)
    - the GitHub / LinkedIn chips: fill in href="#" or delete them
    - optional extra activities

  cv.html
    - just add cv.pdf to the folder

  research-project-2.html
    - everything; it's the blank template


ADDING ANOTHER PROJECT
----------------------
  1. Copy research-project-2.html to a new name,
     e.g. research-soft-gripper.html
  2. Fill it in (use research-passive-abd-hand.html as the reference).
  3. On index.html, copy one <article class="work">...</article> block
     in the "Research Works" section and point "More Detail" at the new
     file.
  4. Add one <li> to the <ul class="subnav"> dropdown. That menu is
     repeated in EVERY .html file, so paste it into all of them.
  5. Add the new page to sitemap.xml.


CHANGING THE COLORS
-------------------
Open style.css. The first block (:root) has every color:
  --navy    nav bar, hero banner and footer background
  --accent  links, buttons, highlights
  --tint    the pale background used on alternating sections
Change those three and the entire site follows.


NOTES
-----
  - Photos, charts and the shaking demo video came out of your
    "Passive ABD Prosthetic Hand.pptx", resized for the web.
  - All paths are relative, so the site works from any folder as long
    as the files stay together.
  - To preview locally, just double-click index.html.
