Living document — update these diagrams when adding features.

# Bible — Architecture

**Relationship to bible-project:** this repo is the earlier, superseded partial copy of the Bible Project. The repo description on GitHub states: "Earlier partial Bible-database copy; current work continues in the bible-project repository." bible-project is the live continuation (U-8/U-9 repairs, YLT word alignment, Flask website). See `bible-project/architecture.md` for the current system.

## 1. Context diagram (level 0)

```mermaid
flowchart LR
    E1["Kit developer"]
    E2["Spreadsheet files"]
    E3["OSHB Hebrew text"]
    E4["KJV word data"]
    E5["YLT verse data"]
    E6["BDB lexicon data"]
    P0("Bible database build system")
    D0[("bible db SQLite file")]
    E1 -->|"runs build scripts"| P0
    E2 -->|"Hebrew word lists"| P0
    E3 -->|"text plus Strongs plus morphology"| P0
    E4 -->|"KJV words with Strongs tags"| P0
    E5 -->|"verse level translation"| P0
    E6 -->|"dictionary glosses"| P0
    P0 -->|"assembled corpus"| D0
    P0 -->|"verification reports"| E1
    D0 -->|"read only corpus"| E1
```

## 2. Level-1 data flow diagram

```mermaid
flowchart LR
    E1["Kit developer"]
    E2["Spreadsheet files"]
    E3["OSHB Hebrew text"]
    E4["KJV word data"]
    E5["YLT verse data"]
    E6["BDB lexicon data"]
    P1("1.0 Ingest canonical word list")
    P2("2.0 Crosswalk words to OSHB")
    P3("3.0 Load renderings and glosses")
    P4("4.0 Build root numbering")
    P5("5.0 Build lexicon entries")
    P6("6.0 Assemble SQLite database")
    D1[("D1 canonical word files")]
    D2[("D2 bible db SQLite file")]
    E1 -->|"runs scripts"| P1
    E2 -->|"word lists"| P1
    P1 -->|"canonical words"| D1
    D1 -->|"canonical words"| P2
    E3 -->|"text plus Strongs"| P2
    P2 -->|"aligned words"| P6
    E4 -->|"KJV words"| P3
    E5 -->|"YLT verses"| P3
    E6 -->|"glosses"| P3
    P3 -->|"renderings and glosses"| P6
    D1 -->|"unpointed base words"| P4
    P4 -->|"root entry form vowel tiers"| P6
    P4 -->|"numbered forms"| P5
    P6 -->|"word frequency data"| P5
    P5 -->|"lexicon rows"| P6
    P6 -->|"assembled corpus"| D2
    P6 -->|"assembly reports"| E1
    D2 -->|"read only corpus"| E1
```

## 3. Entity–relationship diagram

```mermaid
erDiagram
    BOOK {
        int book_id PK
        string english_name
        string hebrew_name
    }
    WORD {
        int word_id PK
        int book_id FK
        string text
        string root_code
        string strongs
    }
    KJV_WORD {
        int kjv_word_id PK
        string english_text
        string strongs
    }
    KJV_RENDERING {
        int rendering_id PK
        int word_id FK
        string english_text
    }
    YLT_VERSE {
        int verse_id PK
        string text
    }
    GLOSS {
        string strongs PK
        string gloss_text
    }
    ROOT_ENTRY {
        int root_id PK
        string base_word
    }
    ROOT_FORM {
        int form_id PK
        int root_id FK
        string affix_pattern
    }
    ROOT_VOWEL {
        int vowel_id PK
        int root_id FK
        int form_id FK
        string pointed_form
    }
    LEXICON {
        int lexicon_id PK
        string kjv_renderings
        string ylt_contexts
        string verse_list
    }
    BOOK ||--o{ WORD : contains
    ROOT_ENTRY ||--o{ ROOT_FORM : has
    ROOT_FORM ||--o{ ROOT_VOWEL : has
    ROOT_VOWEL ||--|| LEXICON : describes
    WORD ||--o{ KJV_RENDERING : rendered_as
    WORD }o--|| GLOSS : glossed_by
```

## Grounding notes

- Observed: README.md table of the 11 SQLite tables with exact row counts (words 264,217; kjv_renderings 409,930; kjv_words 610,324; ylt_verses 23,145; glosses 17,347; books 39; root_entry 30,087; root_form 57,724; root_vowel 126,869; lexicon 126,869); the script list (ingest_canonical.py, assemble_bible.py, build_roots.py/2/3, build_lexicon.py, fixup_notes.py, diff_corrected.py); the three-tier root-based numbering `root.fix.vowel`; public-domain source list (Kit's spreadsheets, OSHB/morphhb, Crosswire KJV, eng-ylt USFM, Open Scriptures HebrewLexicon); README status line "Database built; website not yet started".
- Observed: repo has no website/ directory and no ylt_renderings table (both present only in bible-project); no subdirectories at all — 22 files, all top-level.
- INFERRED: the internal wiring between scripts — README says assemble_bible.py does "crosswalk to OSHB, KJV/YLT/gloss loading, SQLite build" and build_roots*.py builds the tiers, so the DFD process split follows README's own decomposition but the file-level data handoffs between numbered processes are inferred from script purposes.
- INFERRED: primary/foreign keys and attribute names in the ERD are plausible column names inferred from the README's descriptions, not quoted schema (no schema file was observed in this repo); relationship cardinalities are inferred from the described one-per-word and one-per-form structure.
- INFERRED: that `bible.db` is rebuilt by the developer locally rather than shipped, from README "(SQLite, git-ignored — rebuild with assemble_bible.py)".
