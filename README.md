# LRC Translator

**A free and open-source command-line tool for translating synchronized `.lrc` lyrics while preserving timestamps and safely backing up the original lyrics.**

LRC Translator is designed primarily for personal music libraries and media servers running Linux, with particular attention to compatibility with **Jellyfin**.

---

## Overview

LRC Translator scans a media library, finds `.lrc` lyric files associated with music files, translates the lyrics, and writes the translated text back into the original `.lrc` file.

The original `.lrc` file is preserved through an automatic backup before any modification.

The main goals are:

* Free to use.
* Open source.
* Command-line based.
* No web server required.
* No PHP required.
* Designed to run directly on a Linux media server.
* Compatible with Jellyfin's standard lyric file naming.
* Preserve LRC timestamps.
* Preserve the original lyrics through automatic backups.
* Resume interrupted translation jobs.
* Avoid translating files unnecessarily.
* Keep the music library visually clean.
* Make the project easy for other developers to extend.

---

## Why LRC Translator?

Many music libraries contain synchronized lyrics in `.lrc` format.

For example:

```text
01 - Comfortably Numb.flac
01 - Comfortably Numb.lrc
```

Jellyfin can associate the `.lrc` file with the corresponding music file because they share the same filename.

For this reason, LRC Translator does **not** create a second file such as:

```text
01 - Comfortably Numb.pt-BR.lrc
```

Instead, the existing `.lrc` file is updated.

The translated lyrics remain synchronized with the original timestamps.

Example:

```text
[00:12.30]Hello
[00:12.30]Olá

[00:18.42]Is there anybody in there?
[00:18.42]Tem alguém aí?
```

This allows the translated lyrics to continue working with existing media players and media servers.

---

# Safety First

The original lyrics are valuable data.

For that reason, LRC Translator follows a simple principle:

> **Never modify an original LRC file without first preserving a recoverable copy.**

Before modifying an `.lrc` file, the program creates a backup inside the hidden `.backup` directory.

For example:

```text
/srv/media/
├── musicas/
│   └── Pink Floyd/
│       └── The Wall/
│           └── 01 - In the Flesh.lrc
│
└── lrctranslator/
    └── .backup/
        └── musicas/
            └── Pink Floyd/
                └── The Wall/
                    └── 01 - In the Flesh.lrc
```

The backup represents the original file before translation.

If the translation is executed again, an existing original backup must **never be overwritten by an already translated file**.

This ensures that the backup always remains the true original version.

---

# Designed for Media Servers

LRC Translator is designed to live alongside the media directories.

Example:

```text
/srv/media/
├── musicas/
├── filmes/
├── livros/
├── series/
├── show/
└── lrctranslator/
```

The program determines its own location and automatically identifies its parent directory.

It can then inspect directories located alongside `lrctranslator`.

This means there is no need to hard-code the media path into the source code.

If the entire media directory is moved, for example:

```text
/mnt/storage/media/
```

the program can continue to work as long as its directory structure remains the same.

---

# Automatic Media Discovery

The program does not require the user to manually configure every music directory.

If the program is located at:

```text
/srv/media/lrctranslator/
```

it considers:

```text
/srv/media/
```

its media root.

It can then discover directories such as:

```text
musicas
filmes
livros
series
show
```

Only relevant files are processed.

The initial implementation will primarily look for:

```text
*.lrc
```

and verify whether a corresponding music file exists.

For example:

```text
01 - Time.flac
01 - Time.lrc
```

is a valid pair.

An orphaned file such as:

```text
01 - Time.lrc
```

without a corresponding music file can be reported but should not automatically be modified.

---

# Command-Line Interface

The application is intended to run directly from the Linux terminal.

Example:

```bash
cd /srv/media/lrctranslator
python3 lrc_translator.py
```

No PHP, Apache, Nginx, or other web server is required.

A typical session may look like:

```text
========================================
             LRC TRANSLATOR
========================================

Program directory:
 /srv/media/lrctranslator

Media root:
 /srv/media

Scanning media directories...

music.............. 1,284 LRC files
movies............. 0
books.............. 0
series............. 0
shows.............. 0

----------------------------------------
Total LRC files:     1,284
Matching music:      1,271
Without music:          13
----------------------------------------

Start translation? [Y/N]:
```

---

# Analysis Before Translation

The program should never immediately start modifying files.

The first stage is an analysis.

The user should be able to see:

* Number of LRC files found.
* Number of matching music files.
* Number of orphaned LRC files.
* Number of files already processed.
* Number of files pending translation.

Only after the user explicitly confirms should the translation begin.

Example:

```text
1,271 files are ready for translation.

Start translation? [Y/N]:
```

---

# One File at a Time

Translation should be performed sequentially.

Example:

```text
========================================
File 1 of 1271
========================================

Artist : Pink Floyd
Album  : The Wall
Track  : Comfortably Numb

Translating...
```

Then:

```text
File 2 of 1271
```

and so on.

This makes the process easy to monitor and avoids unnecessarily high resource consumption.

---

# Resume Interrupted Jobs

Large music libraries can contain thousands of tracks.

The program must therefore be able to resume an interrupted translation job.

For example:

```text
Translated: 734
Pending:     537
```

If the server is restarted, the program should determine which files have already been processed and continue with the remaining files.

A lightweight local database such as **SQLite** is planned for this purpose.

No MySQL, PostgreSQL, or other database server should be required.

---

# Translation

The translation engine must be **free of charge**.

The project is intended to avoid mandatory paid APIs such as commercial translation services.

The preferred approach is a local translation engine running on the user's own server.

A local model or another free translation mechanism may be integrated through a dedicated translation interface.

The translation engine should be replaceable.

Conceptually:

```text
                 LRC Translator
                       |
                       v
                Translation Layer
                       |
              +--------+--------+
              |                 |
              v                 v
        Local engine       Alternative
        / model            free engine
```

This architecture allows contributors to implement additional translation backends without modifying the rest of the application.

---

# Context-Aware Translation

Lyrics should not necessarily be translated one line at a time.

LRC files frequently divide a sentence across multiple synchronized lines.

For example:

```text
[01:20.00]I don't
[01:21.20]know what
[01:22.40]you're talking about
```

Translating each line independently can produce poor results.

The translation system should therefore be able to send groups of lyric lines as a contextual block while preserving the original timestamps.

The resulting LRC can then be reconstructed.

Example:

```text
[01:20.00]I don't
[01:20.00]Eu não

[01:21.20]know what
[01:21.20]sei o que

[01:22.40]you're talking about
[01:22.40]você está falando
```

The exact translation formatting may evolve during development.

---

# LRC Preservation

The following information must remain intact whenever possible:

* Timestamps.
* Track synchronization.
* LRC metadata.
* Artist information.
* Album information.
* Other valid LRC tags.

The translation process should modify the lyric text rather than unnecessarily reconstructing or discarding unrelated LRC metadata.

---

# Backup Structure

Backups are stored separately from the media library.

They are hidden to avoid visually cluttering the user's media directories.

Planned structure:

```text
lrctranslator/
├── .backup/
├── .logs/
└── .database/
```

The backup directory mirrors the original media directory structure.

Example:

```text
.backup/
└── musicas/
    └── Artist/
        └── Album/
            └── Track.lrc
```

This makes restoration straightforward.

---

# Restore

A future version should provide a simple way to restore an original lyric.

For example:

```text
[1] Analyze library
[2] Translate
[3] Restore backup
[4] View history
[5] Configuration
[0] Exit
```

The restore function should never delete the backup unless explicitly requested by the user.

---

# Project Structure

The project is intended to evolve toward a modular structure:

```text
LRCTranslator/
│
├── lrc_translator.py
├── config.py
├── requirements.txt
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── CHANGELOG.md
│
├── src/
│   ├── scanner.py
│   ├── lrc.py
│   ├── translator.py
│   ├── backup.py
│   └── database.py
│
└── tests/
```

The initial implementation may be simpler.

The project structure can evolve as functionality is added.

---

# Planned Features

## Core

* [ ] Recursive LRC scanner.
* [ ] Automatic media-root detection.
* [ ] Music/LRC matching.
* [ ] LRC timestamp preservation.
* [ ] Context-aware lyric translation.
* [ ] Portuguese (Brazil) translation.
* [ ] Automatic backup.
* [ ] Backup integrity protection.
* [ ] SQLite processing history.
* [ ] Resume interrupted jobs.
* [ ] Translation progress display.
* [ ] Restore original LRC files.

## Translation

* [ ] Free local translation engine.
* [ ] Pluggable translation backends.
* [ ] Automatic language detection.
* [ ] Manual source-language selection.
* [ ] Target-language configuration.

## User Experience

* [ ] Dry-run / analysis mode.
* [ ] Translation confirmation.
* [ ] Detailed progress information.
* [ ] Error reporting.
* [ ] Log files.
* [ ] Library statistics.

## Future Possibilities

* [ ] Multiple target languages.
* [ ] Windows support.
* [ ] Docker support.
* [ ] Optional web interface.
* [ ] Jellyfin integration.
* [ ] Automatic album processing.
* [ ] Parallel processing.
* [ ] Improved lyric segmentation.
* [ ] Translation quality review.

---

# Privacy

The project is intended to support local processing.

Whenever possible, lyrics should remain on the user's own server.

The default translation implementation should not require sending an entire music library to an external commercial service.

Any external translation backend should be explicitly configured by the user.

---

# Open Source

LRC Translator is intended to be a community-driven open-source project.

Contributions are welcome.

Developers can create branches to:

* Fix bugs.
* Improve translation quality.
* Add translation engines.
* Improve LRC parsing.
* Add new languages.
* Improve performance.
* Add operating-system support.
* Improve documentation.
* Add tests.
* Implement new features.

The project should favor simple, understandable code over unnecessary complexity.

---

# Development Philosophy

The project follows a few basic principles:

### 1. Do not destroy user data

Original LRC files must be recoverable.

### 2. Keep the media library clean

Temporary files, databases, logs, and backups should remain outside the normal media directories whenever practical.

### 3. Keep dependencies reasonable

The program should not require a complete web stack or database server.

### 4. Prefer local processing

When technically feasible, translation should happen locally.

### 5. Keep the translator modular

The translation engine should be replaceable without rewriting the LRC processing system.

### 6. Make it understandable

The project should remain accessible to contributors who are not experts in Python or machine learning.

---

# Installation

The project is currently under development.

The planned installation process is intentionally simple.

Example:

```bash
cd /srv/media
git clone <repository-url> lrctranslator
cd lrctranslator
python3 lrc_translator.py
```

The final installation procedure will be documented once the initial version is complete.

---

# Requirements

The initial target platform is:

* Linux
* Python 3
* Standard filesystem access
* Read/write access to the media library

Additional requirements will depend on the selected translation engine.

No web server is required.

No PHP installation is required.

No MySQL/PostgreSQL server is required.

---

# Status

**Early development**

The project is currently being designed.

The initial development priorities are:

1. Media directory discovery.
2. LRC scanning.
3. FLAC/music matching.
4. Safe backup system.
5. LRC parser.
6. SQLite processing history.
7. Translation engine integration.
8. Resume support.
9. Restore functionality.

---

# Contributing

Contributions are welcome.

Before implementing a major feature, opening an issue to discuss the proposed approach is recommended.

For code contributions:

1. Fork the repository.
2. Create a feature branch.
3. Implement the change.
4. Test it against real LRC files.
5. Document relevant changes.
6. Open a Pull Request.

Example:

```bash
git checkout -b feature/new-translation-engine
```

Please avoid committing:

* Personal music files.
* Copyrighted lyrics.
* `.lrc` files from commercial music.
* Personal configuration files.
* Databases.
* Backup files.
* Logs containing personal information.

Use synthetic or public-domain test data whenever possible.

---

# License

This project is licensed under the **MIT License**.

See `LICENSE` for the complete license text.

---

# Disclaimer

LRC Translator is intended for use with lyric files that the user is legally entitled to access, modify, and store.

Users are responsible for complying with applicable copyright laws and the terms governing their music and lyric files.

---

## Project Goal

> **Provide a free, safe, practical, and open-source way to translate synchronized lyrics in personal music libraries without breaking existing media-server compatibility.**

If you find the project useful, contributions, bug reports, testing, and documentation improvements are welcome.
