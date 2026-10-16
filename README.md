# Language Learning App

**A Java application for generating bilingual, comic-based language lessons with AI-assisted dialogue, translations, and speech audio.**

The project brings vocabulary, verb conjugation, and conversational dialogue into a shared lesson format. It processes structured comic scenes, generates dialogue from character poses and settings, translates the text, and supports text-to-speech generation for listening practice. Lessons are exported as XML, with Spanish examples and MP3 audio included in the repository.

The current application is a lesson-generation backend launched through a Java entry point. Its default workflow assembles the existing lesson resources into a combined comic lesson; AI content and audio generation are separate workflows in the code.

## Features

* **Comic-based lesson assembly:** combines conjugation exercises, vocabulary vignettes, transition panels, and dialogue scenes according to a configurable schedule in Java.
* **AI-assisted dialogue:** converts XML scene descriptions into prompts that use the characters, poses, and settings to generate conversational text.
* **Translation support:** generates target-language translations and stores source/translation pairs in a TSV-backed dictionary.
* **Speech audio generation:** sends dialogue to a text-to-speech endpoint, saves MP3 files, and records text-to-audio mappings in TSV format.
* **XML processing:** reads scene specifications with Java DOM APIs and writes structured lesson content.
* **Example learning materials:** includes Spanish lesson XML, character-pose data, and audio for original and translated dialogue.

## Technology

|Technology|Purpose|
|-|-|
|Java|Lesson generation and application logic|
|Java HTTP Client|Requests to text-generation and speech services|
|`org.json`|API request and response handling|
|Java DOM / JAXP|XML parsing and transformation|
|JUnit 5|Tests for parsing, prompts, translations, and scene handling|
|XML, TSV, MP3|Lesson definitions, mappings, and audio assets|

## How it works

The code supports two related workflows:

1. **Assemble an existing lesson.** `controllers.Main` invokes `LessonCreationController.createLesson()`. The controller reads the supplied Spanish lesson components, selects scenes using `LESSON\_SCHEDULE`, and writes `src/main/resources/Spanish/ComicLesson.XML`.
2. **Generate learning content.** The vignette and dialogue classes describe comic panels, request dialogue and translations, and write lesson content. `TTSHandler` can then generate audio from dialogue XML and save the corresponding audio map.

AI generation is not enabled by the default entry point. The generation methods are invoked from Java code rather than through a graphical interface or command-line options.

## Getting started

### Requirements

* JDK 17 or later as a setup baseline; the test sources use Java text blocks.
* A Java IDE such as IntelliJ IDEA.
* The `org.json:json` library for application compilation.
* JUnit Jupiter / JUnit 5 to run the tests.
* Compatible text-generation and text-to-speech services only if generating new AI content or audio.

This source snapshot does not include a Maven or Gradle build file, pinned dependency versions, or a compiled JAR. Dependencies must be configured in the IDE.

### Run the supplied lesson assembly

1. Open the repository as a Java project and select the project JDK.
2. Configure these source roots:

|Directory|IDE source-root type|
|-|-|
|`src/main/java`|Sources|
|`src/main/resources`|Resources|
|`src/test/java`|Test Sources|

3. Add `org.json:json` as a module dependency and JUnit 5 as a test dependency.
4. Set the run configuration's working directory to the repository root.
5. Run `controllers.Main`.

The default workflow uses the bundled XML and does not make API requests. It writes to **`src/main/resources/Spanish/ComicLesson.XML`**, replacing that file if it already exists. Open the output in an XML editor to inspect the assembled lesson; this repository does not include a comic-rendering frontend.

## AI generation configuration

For generation workflows, create a local `config.properties` file in the repository root. `ConfigurationFile` reads this file relative to the working directory.

```properties
# Replace placeholders with values for your compatible API service.
API\_KEY=YOUR\_API\_KEY
COMPLETIONS\_URL=YOUR\_CHAT\_COMPLETIONS\_ENDPOINT
MODEL=YOUR\_TEXT\_MODEL

TTS\_URL=YOUR\_SPEECH\_ENDPOINT
TTS\_MODEL=YOUR\_SPEECH\_MODEL
TTS\_VOICE=YOUR\_VOICE

# Translation target and local data paths
LANGUAGE=Spanish
INPUT\_FILE=src/main/resources/pose\_pairings\_with\_backgrounds.txt
TRANSLATION\_FILE=translations.tsv

# These values select audio subdirectories in the existing implementation.
START\_LANGUAGE=OriginalLanguage
END\_LANGUAGE=TargetLanguage
TTS\_PATH=src/main/resources/audio
```

The text API is expected to accept `model` and `messages`, returning `choices\[].message.content`. The speech API is expected to accept `model`, `input`, and `voice`, returning audio bytes. The audio-directory labels above do not set a speech API language parameter.

The translation loader skips the first TSV row as a header. If creating a new `translations.tsv`, begin it with two tab-separated column names, such as `source` and `translation`.

Keep credentials local. Add the following entries to your `.gitignore` before committing:

```gitignore
config.properties
.idea/
\*.iml
out/
\*.class
```

## Code organisation

|Package|Responsibility|
|-|-|
|`controllers`|Entry point, configuration, and lesson scheduling|
|`dialogue`|Scene dialogue generation, translation coordination, and speech generation|
|`prompts`|HTTP requests, JSON handling, response processing, and audio mappings|
|`translations`|Translation dictionary, phrase extraction, and TSV persistence|
|`vignettes`|Character-pose schemas and scene descriptions|
|`xml`|Parsing source lessons and generating XML output|

Example lesson files are under `src/main/resources/Spanish/`. Speech samples are under `src/main/resources/audio/`, with mappings in `src/main/resources/titleToAudioMap.tsv`.

## Tests

The repository includes JUnit 5 tests under `src/test/java`. Run individual test classes through the IDE after configuring dependencies and resource roots.

Some tests require local configuration, and the translation and dialogue tests include live API calls. These need valid service access and may incur usage charges. They are not a fully isolated offline test suite, and generated responses can vary.

## Current scope and limitations

* Lesson assembly currently uses Spanish resource paths and a fixed schedule defined in `LessonCreationController`.
* AI dialogue generation currently assumes ten scenes; shorter inputs, including the two-scene dialogue test fixture, require adjustment.
* Some generation methods write to `src/main/resources/DialogueLesson.xml`, while assembly and speech generation read `src/main/resources/Spanish/DialogueLesson.xml`. Align these paths when connecting the generation workflows.
* API calls assume successful responses in the expected format; retry handling and robust response validation are areas for improvement.

Potential extensions include a lesson viewer, language-selectable resource paths, a reproducible build, and mocked API tests.

