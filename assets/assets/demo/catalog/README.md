# Demo catalog — translatable course data

The multi-language demo (`--dart-define=I18N_DEMO=true`) never calls the real
backend. `lib/demo/demo_backend.dart` answers every API request on the device
and reads **all course text** from these files:

| File | Role |
|---|---|
| `ar.json` | Source (Arabic). Every key must exist here. |
| `<code>.json` | Translation for one language (`en.json`, `fr.json`, …). Any key that is missing or empty falls back to `ar.json`. |

`<code>` is the Mokhtasr API code used everywhere in the app (the list is in
`assets/demo/content/_languages.json`): `ar en fr ur id ky zh es hi pt bn ru de
pa ja sw tr it ta fa as ha th vi uz az ku sq tl bs am or ne tg ml ug sl mg ak km aa`
(`pa` = Pashto, `ku` = Kurdish Sorani, `sl` = Sinhala, `or` = Oromo).

Structure that is *not* text (ids, which option is correct, Quran pages, lock
rules, pass mark 60 %) lives in `lib/demo/demo_catalog.dart`, so translators
only ever touch strings.

## Format

A flat JSON object: `"dotted.key": "text"`. No nesting, no arrays.

```json
{
  "course.adab.title": "Manners and Morals Course",
  "course.adab.lesson.1.q1.text": "Why is Surah Al-Fatihah called by this name?",
  "course.adab.lesson.1.q1.a": "Because the Book of Allah opens with it"
}
```

## Key schema

| Key | Shown as |
|---|---|
| `course.adab.title` | Course title (cards, course page, top of the lesson list) |
| `course.adab.subtitle` | Course subtitle |
| `course.adab.track` | Track label on the cards |
| `course.adab.short_description` | Card blurb |
| `course.adab.description` | Long description on the course page |
| `course.adab.lesson.<n>.title` | Lesson `<n>` title (`n` = 1, 2, 3) |
| `course.adab.lesson.<n>.subtitle` | Lesson `<n>` subtitle |
| `course.adab.lesson.<n>.q<k>.text` | Lesson `<n>` comprehension question `<k>` (`k` = 1–3) |
| `course.adab.lesson.<n>.q<k>.a` / `.b` / `.c` | Its three options, in this order |
| `course.adab.quiz.title` | Title of every lesson quiz |
| `course.adab.exam.title` | Final exam title |
| `course.adab.exam.q<k>.text` | Final exam question `<k>` (`k` = 1–5) |
| `course.adab.exam.q<k>.a` / `.b` / `.c` | Its options |
| `cards.<i>.title` | Title of demo card `<i>` (`i` = 0–8; these cards open the real course) |
| `cards.track` | Track label of the demo cards |
| `cards.short_description` | Blurb of the demo cards |
| `chat.course.title` | Course discussion room title |
| `chat.course.m1`, `chat.course.m2` | The two team messages in it |
| `chat.support.title` | Support room title |
| `chat.support.m1` | Its welcome message |
| `user.display_name` | Name of the local demo user |

Rules for translators:

- Keep the option order: the correct answer is fixed by letter in
  `lib/demo/demo_catalog.dart` (lesson 1: a, b, a · lesson 2: a, a, c ·
  lesson 3: b, a, c · exam: a, c, b, a, b). Do not reorder `a/b/c`.
- Quran quotations inside `{…}` stay in Uthmani Arabic, untranslated.
- `ﷺ` stays as is.
- Lessons: 1 = page 1 (Al-Fatihah 1–7), 2 = page 411 (Luqman 1–11),
  3 = page 515 (Al-Hujurat 1–4). Questions must be answerable from the
  Mokhtasar tafsir of those verses.
- Check a file: `python3 -c "import json;a=json.load(open('assets/demo/catalog/ar.json'));t=json.load(open('assets/demo/catalog/xx.json'));print(set(a)-set(t))"`
  prints the keys still missing (they will show in Arabic).

## Lesson content (tafsir)

Not in this folder: `assets/demo/content/<code>.json` holds the Mokhtasar
tafsir translation for the three pages, read by `lib/demo/demo_content.dart`
and served as `GET /tafsir/mokhtasr?page=…&lang=…` so the lesson screen renders
it through the normal app path:

```json
{"code": "en", "rtl": false, "book_title": "…",
 "pages": {"1": [{"sura": 1, "aya": 1, "ayah": "<Uthmani verse>",
                  "tafsir": "…", "surah_intro": "Name\nType\n• Heading:\nText"}]}}
```

Verses are always taken from `ar.json` (Uthmani script); a missing page or
verse falls back to Arabic. Page audio is the Arabic page recitation for every
language for now.

## UI strings

Interface strings are separate: `assets/i18n/<code>.json`, keyed by the Arabic
source text (see `docs/i18n-extraction.md`).
