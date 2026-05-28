# A Phonemic Encoding for a Library and Operating System

## Goals

This is the foundational text encoding for a complete computing system: all source code, all system text, and every document in the library is stored in this single encoding. It is built on a phonemic alphabet — one symbol per distinct *sound*, shared across every supported language — rather than on the merged orthographies of existing scripts.

The design pursues a small number of goals, ruthlessly:

- **One sound, one letter.** Spelling history, etymological doublets, and per-language orthographic conventions are discarded. The same sound is written the same way whether it appears in English, Russian, or Japanese. This is what makes the alphabet small.
- **Closed, not extensible.** There is no escape mechanism, no shift state, no extension plane. The internationalization pressure that forced ASCII into Unicode is treated here as a deliberate *anti-goal*: the system serves a fixed set of languages and renounces the rest on purpose. Closing the set is what buys the simplicity.
- **Byte = character = cell.** The encoding is fixed-width. One byte is one phonetic character is one displayed cell. Random access by byte is random access by character, guaranteed forever. No combining-character normalization, no variable-width decoding, no grapheme segmentation.
- **Hard monospace on an 80-column grid.** There is no newline character; text is a stream of fixed 80-byte rows, with position serving as structure. "Move forward ten lines" is "move forward eight hundred bytes." Anything more structured than rows of glyphs — pagination, headings, tables — belongs to a markup layer above the encoding, not inside it.
- **Linguistic core, symbolic shell.** The lower portion of the byte space holds the phonemic alphabet; the upper portion holds digits, punctuation, and the symbol set that all source code and formal notation must express themselves *within* — exactly as every programming language has always accepted the symbols its substrate happened to provide.

## Rationale for the major choices

**Why phonemic instead of merging scripts.** An earlier design merged Latin, Greek, and Cyrillic into one 8-bit table. It strained against 256 slots and forced the dropping of languages, because it was paying for *orthography*: four ways to spell one sound, accented capitals, etymological letters. The phonemic approach pays only for *contrastive sounds*, which are finite and enumerable. The result covers far more languages in roughly half the space. The full alphabet fits within 7 bits.

**Why a modifier letter for palatalization.** The Slavic "soft" consonants are not sixteen unrelated letters; they are one feature — palatalization — applied across the consonant series. Representing that with a single modifier letter (as Cyrillic does with its soft sign) is both more economical and more honest about the phonology. The modifier is a full standalone character occupying its own cell, consistent with the hard-monospace rule; it is never a combining accent.

**Why length and nasalization are modifiers, but diphthongs are sequences.** Length and nasalization are single features layered onto a base vowel, so they earn a modifier each. Diphthongs, by contrast, are genuinely two vowel sounds in succession; under a strict one-sound-one-letter principle they are written as sequences of the two vowels already in the inventory, not as unit letters. This keeps the vowel inventory minimal and the principle consistent across all languages.

**Why the reach exceeds the name.** The project began as "Germanic and Slavic," but a phoneme inventory adequate to European phonology turns out to cover most of the world's phonologically *modest* languages for free — Romance, Finnic, Japanese, Portuguese, and many others — because what those languages have in common is a segmental, non-tonal, consonant-and-vowel shape rather than any shared ancestry. The system's true boundary is phonological *type*, not geography or genealogy.

**What the system deliberately cannot reach.** Languages built on a different phonological axis are out of scope, by design and by cost:

- **Tonal languages** (Mandarin, Vietnamese, Thai, Yoruba) — tone is suprasegmental contrast of a kind the inventory does not track, and would require re-architecture rather than a few letters.
- **Semitic languages** (Arabic, Hebrew) — pharyngeals and emphatic consonants.
- **Click languages** (Khoisan, Xhosa, Zulu) and languages with ejectives or implosives (Georgian, Quechua).

These are not omissions to be patched; they are the renounced remainder that makes a closed, fixed-width encoding possible.

## Deliberate omissions (recorded, not overlooked)

- **Glottal stop /ʔ/** — allophonic in every supported language (e.g. German vowel-initial onset); not encoded.
- **Japanese moraic nasal /N/** — written with the existing /n/; its assimilation is contextual and not separately encoded.
- **Bilabial fricative /ɸ/** — treated as an allophone of /h/ (its status in Japanese); not given a separate slot.
- **Estonian /ɤ/** — unified with the central unrounded vowel /ɨ/; no language in the set contrasts the two.
- **Northern Dutch uvular /χ/** — unified with the velar fricative /x/; a dialectal realization, not a contrast.

---

# The Final Sound Inventory

## Consonants (49)

### Plosives
/p/ /b/ /t/ /d/ /k/ /g/

### Affricates
/tʃ/ /dʒ/ /ts/ /dz/ /tɕ/ /dʑ/ /ʈʂ/ /ɖʐ/ /pf/

### Fricatives
/f/ /v/ /θ/ /ð/ /s/ /z/ /ʃ/ /ʒ/ /ɕ/ /ʑ/ /ʂ/ /ʐ/ /ç/ /x/ /ɣ/ /h/ /ɦ/ /ɧ/

### Nasals
/m/ /n/ /ŋ/ /ɲ/

### Liquids and approximants
/l/ /ɫ/ /ʎ/ /ɹ/ /r/ /ɾ/ /r̝/ /ʁ/ /j/ /w/ /ʋ/ /ɥ/

## Vowels (21)

### Front unrounded
/i/ /ɪ/ /e/ /ɛ/ /æ/

### Front rounded
/y/ /ʏ/ /ø/ /œ/

### Central
/ə/ /ɜ/ /ɐ/ /ɨ/ /ʉ/

### Back
/u/ /ʊ/ /o/ /ɔ/ /ʌ/ /ɑ/ /ɒ/

*Diphthongs are written as sequences of two vowels from the set above; they are not separate letters.*

## Modifiers (3)

| Mark | Feature | Used by |
|------|---------|---------|
| ◌ʲ | palatalization | Slavic, Estonian |
| ◌ː | length (vowels and consonants; doubled for Estonian overlong) | German, Finnish, Estonian, Icelandic, Japanese |
| ◌̃ | nasalization | Polish, French, Portuguese, Old Norse |

## Suprasegmentals (3)

| Mark | Feature | Used by |
|------|---------|---------|
| ◌́ | stress / pitch accent (primary) | Russian, English, Swedish, Norwegian, Japanese |
| ◌̀ | pitch accent (secondary) | Swedish, Norwegian |
| ◌ˀ | stød | Danish |

---

## Totals

| Block | Count |
|-------|-------|
| Consonants | 49 |
| Vowels | 21 |
| Modifiers | 3 |
| Suprasegmentals | 3 |
| **Phonetic total** | **76** |

The 76 phonetic characters, together with digits, punctuation, and the source-code symbol block, fit within a single 8-bit byte with substantial reserve — and the phonetic core alone fits within 7 bits. Every byte is one character and one cell on the 80-column grid.

## Languages covered

**Germanic:** English, German, Dutch, Afrikaans, Danish, Norwegian, Swedish, Icelandic, Luxembourgish, Flemish — modern and (for English/Norse) archaic.

**Slavic:** Russian, Ukrainian, Belarusian, Polish, Czech, Slovak, Slovene, Croatian, Serbian, Bosnian, Montenegrin, Macedonian, Bulgarian.

**Romance:** Spanish (including Argentine/Rioplatense), French, Portuguese — Italian and Romanian fall in at little or no additional cost.

**Finnic:** Finnish, Estonian.

**Other:** Japanese.
