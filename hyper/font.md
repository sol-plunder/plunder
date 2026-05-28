# A Grid-Native Writing System
### Founding design document — rationale and rubric

---

## 0. What this is

This document describes the design of a writing system and text encoding intended as the foundation of a computing system: a single substrate on which all source code, all system text, and every document in a library is stored, displayed, printed, and archived.

It is not an adaptation of an existing script to the computer. It is a script *designed for* the computer the way Latin was designed for the pen and runes were designed for the chisel — a full commitment to one medium, the addressable binary grid, with every inherited assumption of the ink age deliberately discarded.

The design was reached almost entirely by *subtraction*. At each step, a feature was removed, and the removal turned out to unlock something — because the thing given up had been paying for a constraint the system had decided not to honor. The result is a stack in which every layer has the same shape: a phoneme is a byte, a byte is a cell, a cell is a scannable bitmap, a line is 128 cells, a page is 128 lines, and a printed page is a lossless bridge back to the digital. Nothing is translated between layers because the layers are the same decision seen from different distances.

---

## 1. The founding bet

Every great writing system is ruthlessly specialized to its production medium, and that specialization is the source of its quality, not a limitation of it. Latin's serifs and stroke modulation are a broad-nib pen made visible; it is genuinely poor for carving. Runes are angular because curves split wood and crack stone. Chinese is brush dynamics fossilized into form. Cuneiform *is* the wedge of a stylus in clay. In each case the script and the medium are one artifact, and the script's greatness is how completely it surrendered to its medium.

A system that hedges — that tries to serve every medium at once — is native to none. This is the diagnosis of Unicode: its sprawl, its variable-width decoding, its normalization forms, its combining-character complexity, and its surrogate pairs are all scar tissue from refusing to commit to a medium. It serves the computer *through translation layers* rather than *being* computer-native, because it was built to carry every prior script and every prior medium forward simultaneously.

This system makes the opposite bet: **that the addressable binary grid is the medium of the present age, and a civilization should have a writing system as native to it as Latin was to ink.** Not text displayed on a computer — text that *is* computational at its foundation. The wager is unprovable in advance, as all foundational bets are. The design is what it looks like to commit to it honestly and follow it all the way down.

---

## 2. The two independent properties: count and density

The central technical insight is that **character count and per-glyph information density are independent properties that history made appear linked.**

- Latin, Greek, Cyrillic: *few* characters, *low* density (simple shapes, much empty space in each cell), and — being computationally pleasant — single-byte and fixed-width.
- Chinese, Japanese, Korean: *high* density (rich signal per cell) but *enormous* count, which forces multi-byte variable-width encodings and all the machine-hostile consequences that follow.

The misery of CJK for computers comes entirely from the **count** (tens of thousands of characters → multi-byte → no byte=character, decoding required, input methods, encoding wars, huge fonts). It does **not** come from the density. The pleasantness of ASCII comes entirely from the small fixed **count** (→ one byte per character → byte = character = everything). Its only real weakness is the low density and the Latin-specific proportions.

Because count and density are independent, a system can take **ASCII's count and data model** together with **CJK's spatial density**. Nobody had occupied that corner of the design space, because the two dense scripts also happened to be the two huge-count scripts — a historical correlation, not a necessity. This system breaks the correlation: a small, closed, single-byte, fixed-width set of *dense* glyphs. Computer-friendly like ASCII because the count is small; space-efficient like CJK because the glyphs are dense.

What it does **not** inherit from CJK is logographic coverage. It remains an alphabet — sounds, not meanings. It gets CJK's ink-efficiency at the *cell* level while staying alphabetic at the *word* level. Conflating those would be an overclaim; keeping them separate is what preserves the computer-friendliness.

---

## 3. The phonemic alphabet

The system is built on a phonemic alphabet — one symbol per contrastive sound, shared across every supported language — rather than on merged orthographies.

**Why phonemic.** An earlier approach merged Latin, Greek, and Cyrillic into one table. It strained against 256 slots and forced the dropping of languages, because it was paying for *orthography*: multiple spellings of one sound, accented capitals, etymological letters. The phonemic approach pays only for *contrastive sounds*, which are finite and enumerable, and the same sound is written the same way in every language. The result covers far more languages in roughly half the space.

**The reach exceeds the name.** The project began as "Germanic and Slavic," but a phoneme inventory adequate to European phonology covers most of the world's phonologically *modest* languages for free — Romance, Finnic, Japanese, Portuguese — because what those languages share is a segmental, non-tonal, consonant-and-vowel *shape*, not any common ancestry. The system's true boundary is phonological *type*, not geography or genealogy.

**What it deliberately cannot reach**, by design and by cost, are languages built on a different phonological axis: tonal languages (Mandarin, Vietnamese, Thai, Yoruba), Semitic languages with pharyngeals and emphatics (Arabic, Hebrew), and languages with clicks, ejectives, or implosives (Khoisan, Georgian, Quechua). These are not omissions to be patched; they are the renounced remainder that makes a closed, fixed-width encoding possible.

**Low information per character is a deliberate choice.** Diphthongs are written as sequences of two vowels, not as unit letters. Features (palatalization, length, nasalization) are written as modifiers, not as precomposed variants. This keeps the character set small and regular, and pushes all richness into *strings* — which is exactly the shape a computer wants: a tiny alphabet and composition by concatenation. The decision that is good for the encoding's size is also the decision that is good for the machine's parsing; a modifier is a regular, searchable, sortable suffix where a precomposed variant would be an irregular special case.

(The detailed sound inventory — ~49 consonants, ~21 vowels, modifiers for palatalization/length/nasalization, and suprasegmentals for stress/pitch-accent/stød — is maintained as a separate specification.)

---

## 4. The encoding model

**Closed, not extensible.** There is no escape mechanism, no shift state, no extension plane. The internationalization pressure that forced ASCII into Unicode is treated as a deliberate anti-goal: the system serves a fixed set of languages and renounces the rest on purpose. Internationalization was the *only* force that ever truly required text-encoding extensibility — and renouncing it is what buys every simplification below. Closing the set is not a limitation tolerated; it is the mechanism that delivers the guarantees.

**Byte = character = cell.** The encoding is fixed-width: one byte is one character is one displayed cell. This restores the single most computer-friendly property a text encoding can have. Length, index, slice, reverse, sort, and compare are all trivial byte operations. The entire class of bugs that comes from "a character is not a byte" simply does not exist.

**No combining marks.** All marks are either their own standalone cell (a modifier letter) or a motif designed into a single precomposed glyph. Nothing stacks within a cell as a combining accent. This preserves byte=cell and the fixed-width guarantee.

**256 characters, one byte.** The lower portion holds the phonemic alphabet; the upper portion holds digits, punctuation, box-drawing/block elements, and the symbol set that all source code and formal notation must express themselves *within* — exactly as every programming language has always accepted the symbols its substrate provided. Source code adapts to the available symbols; this is the normal and correct order, and it means the symbol set can be frozen deliberately and generously rather than predicted.

---

## 5. The grid and the glyph

**No newline.** Text is a stream of fixed-width rows; position is structure. There is no newline character. "Move forward ten lines" is "move forward N bytes." Anything more structured than rows of glyphs — pagination, headings, tables, diagrams-as-documents — belongs to a markup layer above the encoding, not inside it.

**The rigid grid makes position a trusted signal.** In handwriting and proportional type, absolute position within a glyph is noise the reader learns to discard, so shape must carry everything. On a rigid grid, every glyph is registered to a known cell at a known position, so *which part of the cell is inked* becomes a perfectly reliable channel — and it is the most degradation-robust channel available, because position is exactly what a grid preserves losslessly. A left-column stroke versus a right-column stroke share no ink and cannot be confused at any resolution; this is why the b/d problem is a Latin problem and need not be ours.

**No gaps — segmentation is structural, not visual.** Latin spends part of every cell on side-bearings (the inter-letter gap) and on baseline-relative spacing, because its boundaries are *unpredictable* and must be drawn. On a fixed grid the boundary is at every Nth pixel by definition, so the gap is pure waste. Glyphs fill the cell edge-to-edge, horizontally and vertically, and the grid does the separating. This recovers monospace's fixed-width tax on both axes at once. Logical segmentation is never ambiguous; the only remaining concern is perceptual *crowding* (flanking ink interfering with foveal recognition), managed by concentrating each glyph's high-signal detail toward its center and keeping its edges quiet.

**Square cells.** The cell renders square, making the grid an isotropic raster: circles are round, diagonals are true, and the coordinate model has no privileged axis. Latin's proportions assume a tall cell (ascenders, descenders, x-height rhythm); purpose-built glyphs have no such inheritance and can be designed *for* the square. Where reading wants a tall-narrow rhythm, that rhythm is carried by the *ink within* the square cell rather than by the cell's aspect ratio — the square serves the canvas and the coordinate model; the internal ink serves reading.

**Featural construction.** Glyphs are built from a small set of base skeletons (one family per place of articulation, conceptually) plus systematic marks (voicing, manner, palatalization, etc.), so the alphabet is far smaller to learn than its character count suggests — a reader who knows the system can decode a glyph never seen before. This applies Hangul's *principle* (compose at the level where the language is regular) at the unit where it is true here — the **segment-plus-feature** — rather than at the syllable, which the target languages' wildly varying consonant clusters make impossible.

**Ink-free channels.** Freeing letterforms from ink does not merely permit different shapes; it adds entire perceptual *dimensions* that ink-bound writing could never use cheaply: fill texture (solid / dotted / dashed / hatched), line weight (graded), position within the cell, stroke style, and figure-ground inversion (ink-on-paper vs. paper-on-ink). Each is an independent channel. Distinguishing glyphs across *multiple orthogonal channels* — rather than cramming more shape detail into one — is what lets a 256-glyph set stay legible while small and gaplessly packed, because separate channels add their distinctness rather than competing for the same visual resource. The channels differ in robustness (position and figure-ground survive degradation best; fine texture is fragile), so the most frequent and important distinctions are assigned to the most robust channels and the rarer, finer distinctions to the fragile ones.

---

## 6. Unified text and graphics

Because cells are gapless and the grid is rigid, the text plane *is* a raster surface: each cell is one controllable super-pixel, fully inked cells form genuinely solid regions, and diagonal-bearing cells connect into continuous lines across the seam. This revives and generalizes the box-drawing / PETSCII tradition so that *every* glyph tiles seamlessly, not just a special block of drawing characters. Text and graphics share one model, one coordinate system, one buffer. The drawing capability is not a feature that was added; it is an emergent property of the gapless-grid commitment. The square cell keeps the geometry honest, so diagrams and illustrations in the library's texts are geometrically true rather than ASCII-art-stretched.

---

## 7. The page, and the archive

**One frame, every medium.** A line is 128 cells. A page is 128 lines — 16,384 bytes, exactly 16KB, a power of two that is simultaneously a memory page and a paper page. The terminal is 128×128; the screen is 128×128; the printed sheet is 128×128. The same 16KB renders identically to glass or to paper with zero difference. A document does not mutate as it moves between screen, storage, and print, as it does in the ink age; it is a fixed object that media merely present. Long documents are sequences of 16KB pages, page *N* at offset *N* × 16384 — the same power-of-two addressing rhyme one level up.

**Lossless paper round-trip.** Because a glyph is a small bitmap on a rigid known grid, and there are exactly 256 possible glyphs each occupying a known cell, **scanning is not OCR — it is bit-reading.** No interpretation, no probabilistic recovery, no ambiguity: each cell is sampled at known positions and matched against 256 known patterns. The multi-channel distinctness engineered for screen legibility is the same property that makes the scan robust; the legibility engineering and the scan-fidelity engineering are one and the same.

This yields a capability no ink-age text stack possesses honestly: **storage that is simultaneously archival-permanent and perfectly digital.** Paper survives centuries without power, hardware, or format support; normally that means surrendering digital exactness. This system refuses the choice. A document printed as a 128×128 glyph grid is paper-permanent *and* bit-exact, recoverable by eye or by scanner, with no surviving computer required to interpret it — because the interpretation rule ("match the grid against 256 known shapes") can be reconstructed from a single reference sheet. The codec itself fits on one page. **The first page of any archive can be the page that teaches a future reader how to read all the others.**

---

## 8. The rubric — the principles in order

The design can be applied and extended by holding to these, roughly in priority:

1. **Commit to the medium.** The medium is the addressable binary grid. When a choice would serve the grid or serve a legacy of ink, serve the grid. Fitness to medium is the source of quality, not a limitation.
2. **Subtract before adding.** Prefer removing an inherited assumption to adding a feature. Most gains in this design came from renunciation, because the renounced thing was paying for a constraint not worth honoring.
3. **Keep the count small and closed.** No extensibility, no escape, no shift. The closed set is what delivers byte=character and every simplification downstream.
4. **One unit, every layer.** Phoneme = byte = cell = bitmap; line = 128 cells; page = 128 lines = 16KB = one screen = one sheet. Keep the layers the same shape so nothing must be translated between them.
5. **Push richness into sequences, not glyphs.** Modifiers over variants; diphthongs over diphthong-letters. Small regular characters; complexity in strings. This serves both encoding size and machine parsing.
6. **Let the grid carry what the grid carries.** Position for segmentation (no gaps) and for signal (position channel). Do not redraw, in the glyph, information the rigid grid already guarantees.
7. **Distinguish across orthogonal channels.** Shape, position, texture, weight, figure-ground. Assign frequent/important contrasts to robust channels, rare/fine contrasts to fragile ones. Spread distinctness across dimensions rather than cramming one.
8. **Square the cell.** Isotropic canvas, clean coordinates; carry reading rhythm in the ink, not the aspect ratio.
9. **Unify text and graphics.** Every glyph tiles seamlessly; the text plane is a drawing surface; one model serves prose, code, notation, UI, diagrams, and art.
10. **Design for the archive.** Glyph distinctness must survive print-and-scan as bit-reading, not OCR. The codec must be expressible on a single self-bootstrapping page.

---

## 9. What remains open

Everything designable has been reasoned through; the system is internally coherent across all layers. The one load-bearing question that remains is **empirical, not derivable**: where the human crowding-and-acuity floor sits for a real, distinctness-optimized, gaplessly-packed glyph set at small sizes — and how each ink-free channel (position, texture, weight, figure-ground) degrades independently under blur and scanning.

The resolution is a measurement, not an argument: design a real featural glyph set against the phoneme grid, render it gapless at descending sizes, compute pairwise and in-context distinctness under blur and under simulated scanning, and find the size at which crowding begins to break recognition. That floor is the single number on which the strong form of the thesis ("CJK-dense *and* ASCII-clean at once") depends. If the floor is low enough, the thesis is fully realized; if it is higher than hoped, the system still lands at "denser than Latin, fully computer-native, losslessly archivable" — which is already a win, merely a smaller one.

---

## 10. The character of the thing

This system achieves its coherence by being exquisitely specialized — and the specialization is its identity, not its flaw. The same choices that make it beautiful make it unable to leave its world: it cannot internationalize, cannot escape the grid, cannot be handwritten, cannot render gracefully in proportional type. This is the exact mirror of Latin being unable to leave ink — and Latin's commitment to ink was never held against it. Elegance and specialization are the same fact.

It is a writing system that stops porting the ink age onto silicon and starts from silicon: a commitment to the computer era as the foundation of a culture, abandoning the world of ink and cursive, fully embracing the world of the binary. Whether the binary grid is the substrate of the next long age the way ink was for the last is not knowable in advance. But that is the nature of a foundational bet. You do not get to know; you get to commit. This is what the commitment looks like, followed honestly to the foundation.
