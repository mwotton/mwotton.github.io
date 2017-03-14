# Resume for Mark Wotton

## Skills

  - Haskell, C, bash
  - distributed systems design and implementation

## Work History

### betterteam (Senior Software Engineer)

  - initiated a Haskell web app for betterteam.com, including email
    processing, analytics integration with segment/intercom, and
    extensive healthchecks & monitoring. yesod/postgresql/ghcjs

### meanpath/leadstage (CTO)

  - wrote a crawler that collected the front page from 160
    million domains daily, engineered a system to avoid
    crushing the domain servers involved.

  - Haskell stack + C zeromq agent to
    distribute domains for fetching. Sqlite backend for temporary
    storage + elasticsearch for search.

  - under the leadstage brand, took meanpath crawl results and
    enriched them with social media info, lead
    scoring, Alexa rank, IP sharing information etc.

    The Haxl framework enabled runtime selection of source
    information and clear & efficient compound queries.

  - Other meanpath/leadstage projects
    - OSS robots.txt parsing library (https://hackage.haskell.org/package/robots-txt)
    - arin crawler
    - domain extractor (very fast stream filter for finding domain
      names in data)
    - fault-tolerant elasticsearch ingester
    - parallelised SMTP query engine
    - elasticsearch alternative (directly querying sqlite database in
      parallel)

### ninjablocks (Co-founder/CTO)
  - IoT startup - Haskell backend, C on the device, communication
    through zeromq.

### X-Men Origins: Wolverine movie (solder monkey)
  - soldered the blinkenlights for Wolverine's adamantium injection scene

### Doha Asian Games ceremonies (programmer)
  - Implemented a simple operating system for the PIC chip on a custom
board, allowing it to be programmed from a handheld device for complex
lighting tasks

## OSS work

  - numerous patches (yesod, intero, mandrill binding, slack binding, sqlite3-lz4)
  - selection of solo projects

    - dnsmadeeasy binding for Haskell
          https://github.com/mwotton/dnsmadeeasy

    - dustme: selecta reimplementation in Haskell
          https://github.com/mwotton/dustme

    - binding to segment.com API
          https://github.com/mwotton/segment-api

    - lz4 binding
          https://github.com/mwotton/lz4hs

    - Hubris: Haskell/Ruby binding
          https://github.com/mwotton/Hubris

    - hscmph - CMPH binding for computing perfect hashes
          https://github.com/mwotton/hscmph


## Other employers

  Bigcommerce, Westfield, Optus, Upguard

## Education

  BSc (Hons) University of Sydney

## Contact

  - +1 734 239 0390
  - mwotton@gmail.com
  - http://www.shimweasel.com
  - https://twitter.com/mwotton
  - https://hackage.haskell.org/user/MarkWotton
