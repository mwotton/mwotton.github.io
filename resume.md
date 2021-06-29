# Resume for Mark Wotton

## Skills

  - primarily Haskell and PostgreSQL
  - have also worked professionally in Python, Ruby, C, shell, perl (both standalone and PL/pgSQL)
  - distributed systems design and implementation
  - backend web development
  - some embedded systems work

## Work History

### SimSpace (Lead Software Engineer, Oct 2017-present)

  - authored a resource management web app for deciding when a given network
	could be built, given the resources available. Squeal/Servant
  - authored a query language & compiler for content search.
  - wrote [squealgen](https://github.com/mwotton/squealgen) in order to automatically
	generate haskell-level types from a given database and integrated it into
	the SimSpace build process.
  - extensive database query profiling and optimisation.
  - added infrastructure for database-level testing with automatic migration.

### BetterTeam (Senior Software Engineer, Oct 2015-Oct 2017 )

  - initiated a Haskell web app for betterteam.com, including email
    processing, analytics integration with segment/intercom, and
    extensive healthchecks & monitoring. yesod/postgresql/ghcjs.

### MeanPath/LeadStage (CTO, Oct 2013-Oct 2015)

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
    - domain extractor (very fast stream filter for finding domain
      names in data)
    - arin crawler
    - fault-tolerant elasticsearch ingester
    - parallelised SMTP query engine
    - elasticsearch alternative (directly querying sqlite database in
      parallel)

### NinjaBlocks (Co-founder/CTO, 2012)
  - IoT startup - Haskell backend, C on the device, communication
    through zeromq.

### X-Men Origins: Wolverine movie (solder monkey, 2007)
  - soldered the blinkenlights for Wolverine's adamantium injection scene.

### Doha Asian Games ceremonies (programmer, 2006-2007)
  - Implemented a simple operating system for the PIC chip on a custom
board, allowing it to be programmed from a handheld device for complex
lighting tasks.


## OSS patches
  - squeal, yesod, intero, mandrill binding, slack binding, sqlite3-lz4

## selection of OSS solo projects

- [roboservant](https://github.com/mwotton/roboservant) : a type-driven fuzzer
- [squealgen](https://github.com/mwotton/squealgen) : a type generator for postgresql databases
- [robots.txt](https://github.com/meanpath/robots) : a parser library for robots.txt files
- [dnsmadeeasy](https://github.com/mwotton/dnsmadeeasy) binding for Haskell
- [dustme](https://github.com/mwotton/dustme): selecta reimplementation in Haskell
- [segment.com](https://github.com/mwotton/segment-api) API binding for Haskell
- [lz4](https://github.com/mwotton/lz4hs) binding for Haskell
- [Hubris](https://github.com/mwotton/Hubris), a Haskell/Ruby binding.
- [hscmph](https://github.com/mwotton/hscmph), a CMPH binding for computing perfect hashes.

## Other employers

  Bigcommerce (2013), Upguard (2012-2013), Key Options (2010-2012), Westfield (2009-2010), Optus

## Education

  BSc (Hons) University of Sydney

## Contact

   - +1 734 239 0390
   - mwotton@gmail.com
   - https://shimweasel.com
   - https://twitter.com/mwotton
   - https://hackage.haskell.org/user/MarkWotton
