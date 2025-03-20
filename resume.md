# Resume for Mark Wotton

I'm a highly experienced freelancer and consultant with significant experience solving technical problems with whatever tools suit the work to be done, with a particular focus on algorithmically complex backend systems and end-to-end correctness via types (from HTTP to database) and generative testing.

## Skills
  - LLM-powered backend systems
  - general web development
  - generative testing
  - embedded systems work
  - have worked professionally in Haskell, Postgres, Java, Python, TypeScript, Rust, C, shell, perl (both standalone and PL/pgSQL) and Ruby for the past 25 years.

## Work History

### LambdaLabs OU (solo operator/consultant, 2020-present)
  - provided web crawling services & data (Python, Haskell).
  - optimised critical code for a Sydney Haskell startup getting it down from minutes to half a second.
  - built a timetable solver for a medical startup using Java and TypeScript that produced significantly better results while respecting existing semantics (using generative testing with MiniTSis, which was written for this.).
  - implemented a favicon healthcheck site (TypeScript, Rust).
  - built a system for automatically generating Scrapy web crawlers from a base URL using LLMs (Haskell).
  - built a decision tree generator using LLMs to answer legal questions (Python).

### SimSpace (Lead Software Engineer, Oct 2017-2020)
  - authored a resource management web app for deciding when a given network could be built, given the resources available. Squeal/Servant
  - authored a query language & compiler for content search.
  - wrote [squealgen](https://github.com/mwotton/squealgen) in order to automatically generate haskell-level types from a given database and integrated it into	the SimSpace build process.
  - extensive database query profiling and optimisation.
  - added infrastructure for ephemeral database-level testing with automatic migration.

### BetterTeam (Founding Engineer, Oct 2015-Oct 2017 )

  - founding engineer for betterteam.com, including email
    processing, analytics integration with segment/intercom, and
    extensive healthchecks & monitoring. App is still running with over a million jobs posted and 40 million candidates. yesod/postgresql/ghcjs.

### MeanPath/LeadStage (CTO, Oct 2013-Oct 2015)

  - wrote a crawler that collected the front page from 160 million domains daily, engineered a system to avoid crushing the domain servers involved.
  - Haskell stack + C zeromq agent to distribute domains for fetching. Sqlite backend for temporary storage + elasticsearch for search.
  - under the leadstage brand, took meanpath crawl results and enriched them with social media info, lead scoring, Alexa rank, IP sharing information etc.

    The Haxl framework enabled runtime selection of source information and clear & efficient compound queries.
  - Other meanpath/leadstage projects
    - domain extractor (very fast stream filter for finding domain names in data)
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
  - squeal, yesod, intero, mandrill binding, slack binding, sqlite3-lz4, quickjs-hs

## selection of OSS solo projects

- [miniTSis](https://github.com/lambdamechanic/miniTSis): TypeScript implementation of Hypothesis-style generative testing (ported from minithesis)
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

   - +84 038 568 9863
   - mark@lambdamechanic.com
   - https://shimweasel.com
   - https://twitter.com/mwotton
   - https://hackage.haskell.org/user/MarkWotton
   - https://www.npmjs.com/~mwotton
   - https://lib.rs/~mwotton
