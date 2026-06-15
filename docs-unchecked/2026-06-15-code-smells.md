Audit complete. I read the current 101 tracked files in `/Users/onur/repos/manim-voiceover`; the worktree is clean on `main...origin/main`. Binary assets were inspected by type/hash, and the `.mo` catalogs were decoded with `msgunfmt`. Ignored generated outputs like `.venv`, `docs/_build`, cache dirs, coverage, and media were not treated as source files.

**Code Smells**
- Manual mutable kwargs parsing instead of typed config models: [base.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/base.py:42), [voiceover_scene.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/voiceover_scene.py:220).
- Leftover unknown kwargs are kept as `additional_kwargs`, hiding bad inputs instead of rejecting them: [base.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/base.py:96).
- Cache schema is split between `TypedDict` and Pydantic, then cast back to dicts: [_typing.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/_typing.py:50), [cache.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/cache.py:67).
- Pydantic cache models use `extra="allow"`, so the schema is still loose: [cache.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/cache.py:17).
- `SpeechService` does too much: config parsing, cache, transcription, speed adjustment, wrapping provider calls: [base.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/base.py:165).
- Provider services duplicate the same cache/generate/return flow: [gtts.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/gtts.py:35), [gemini.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/gemini.py:181), [azure.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/azure.py:181).
- Optional dependency imports log at module import time and leave names conditionally undefined: [gtts.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/gtts.py:8), [gemini.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/gemini.py:16).
- Library code prompts users, installs packages, writes `.env`, and exits the process: [helper.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/helper.py:125), [helper.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/helper.py:176).
- Runtime dependency on `pip` exists only to support that interactive install behavior: [pyproject.toml](/Users/onur/repos/manim-voiceover/pyproject.toml:42).
- Broad `Exception` is used instead of specific exception types: [tracker.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/tracker.py:132), [azure.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/azure.py:179).
- Azure SSML is built with raw string interpolation; text and prosody values are not XML-escaped: [azure.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/azure.py:143).
- Azure word-boundary conversion depends on SDK private `__dict__` layout: [azure.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/azure.py:236).
- Recorder example passes `silence_threshold`, but the constructor expects `trim_silence_threshold`; the setting is silently ignored: [recorder-example.py](/Users/onur/repos/manim-voiceover/examples/recorder-example.py:8), [__init__.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/recorder/__init__.py:32).
- Recorder mixes UI prompts, device selection, key listeners, recording, trimming, conversion, and playback: [utility.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/recorder/utility.py:187).
- Recorder has a risky frame slice that can empty short recordings: [utility.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/recorder/utility.py:288).
- Subcaption splitting can divide by zero for empty captions or invalid max length: [voiceover_scene.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/voiceover_scene.py:125).
- Bookmark timing can divide by zero when text has no non-bookmark content: [tracker.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/tracker.py:92).
- Cache key hashing uses `json.dumps(data)` without `sort_keys=True`, so equivalent dicts can hash differently: [base.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/base.py:248).
- Cache writes append to a JSON list without atomic write, locking, or dedupe: [helper.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/helper.py:109).
- `assert` is used for runtime validation and can be stripped with `python -O`: [helper.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/helper.py:83), [translate/__init__.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/translate/__init__.py:19).
- Translation `.po` parsing is hand-rolled with string/regex logic instead of a gettext parser: [gettext_utils.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/translate/gettext_utils.py:73).
- Translation subprocesses ignore failures with `check=False`: [gettext_utils.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/translate/gettext_utils.py:33), [render.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/translate/render.py:104).
- Translation render passes a replacement env containing only `LOCALE` and `DOMAIN`, likely dropping `PATH`: [render.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/translate/render.py:120).
- `get_gettext()` globally installs translations into builtins: [translate/__init__.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/translate/__init__.py:26).
- CLI parsers are module globals, making tests/refactors awkward: [render.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/translate/render.py:12), [translate.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/translate/translate.py:16).
- `modify_audio` treats anything not ending `.wav` as MP3: [modify_audio.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/modify_audio.py:39).
- `adjust_speed` creates temp names by concatenating paths with `uuid1`: [modify_audio.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/modify_audio.py:26).
- Stitcher is disabled-ish/private but still present and referenced by demo code: [stitcher.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/stitcher.py:83), [voiceover-demo.py](/Users/onur/repos/manim-voiceover/examples/voiceover-demo.py:227).
- `services/__init__.py` is comment-only dead code: [__init__.py](/Users/onur/repos/manim-voiceover/src/manim_voiceover/services/__init__.py:1).
- README service list omits implemented services like OpenAI and ElevenLabs: [README.md](/Users/onur/repos/manim-voiceover/README.md:23).
- README/PyPI badges use underscore project names instead of canonical `manim-voiceover`: [README.md](/Users/onur/repos/manim-voiceover/README.md:5).
- Docs say not to commit `.mo` files, but the repo commits `.mo` files: [translate.rst](/Users/onur/repos/manim-voiceover/docs/source/translate.rst:128).
- ReadTheDocs uses Python 3.10 while the package requires Python 3.11+: [.readthedocs.yml](/Users/onur/repos/manim-voiceover/.readthedocs.yml:6), [pyproject.toml](/Users/onur/repos/manim-voiceover/pyproject.toml:6).
- API docs import optional service modules directly, which makes docs depend on optional import behavior: [api.rst](/Users/onur/repos/manim-voiceover/docs/source/api.rst:29).
- Examples use wildcard Manim imports throughout, and examples are excluded from linting: [gtts-example.py](/Users/onur/repos/manim-voiceover/examples/gtts-example.py:1), [pyproject.toml](/Users/onur/repos/manim-voiceover/pyproject.toml:127).
- Example coverage only renders `gtts-example.py`, so broken examples can slip through: [test_examples_render.py](/Users/onur/repos/manim-voiceover/tests/test_examples_render.py:18).
- The default test suite includes a network/integration gTTS render, which is flaky by nature: [test_examples_render.py](/Users/onur/repos/manim-voiceover/tests/test_examples_render.py:23).
- Arabic example has import-time `os.system`, global config mutation, debug prints, and an undefined `logo`: [quadratic-formula-arabic.py](/Users/onur/repos/manim-voiceover/examples/quadratic-formula-arabic.py:40), [quadratic-formula-arabic.py](/Users/onur/repos/manim-voiceover/examples/quadratic-formula-arabic.py:729).
- `pyttsx3-example.py` class is named `GTTSExample`: [pyttsx3-example.py](/Users/onur/repos/manim-voiceover/examples/pyttsx3-example.py:6).
- Tests are heavily overfit to internals: `__new__`, private helpers, exact error strings, monkeypatching globals: [test_service_contracts.py](/Users/onur/repos/manim-voiceover/tests/test_service_contracts.py:55), [test_more_behavior.py](/Users/onur/repos/manim-voiceover/tests/test_more_behavior.py:140).
- Tests assert broad `Exception`, matching weak production exceptions instead of forcing better errors: [test_edge_coverage.py](/Users/onur/repos/manim-voiceover/tests/test_edge_coverage.py:160).
- Slophammer and mutation checks are heavy and duplicated in CI, while publish uses a different gate set: [build.yml](/Users/onur/repos/manim-voiceover/.github/workflows/build.yml:42), [python-publish.yml](/Users/onur/repos/manim-voiceover/.github/workflows/python-publish.yml:100).
- `uv.lock` contains a very large optional/dev closure, including transcription, PyObjC, Torch/CUDA-related packages; dev dependencies need splitting: [pyproject.toml](/Users/onur/repos/manim-voiceover/pyproject.toml:57).
- Static docs JS leaks a global `style` variable and leaves `console.log`: [responsiveSvg.js](/Users/onur/repos/manim-voiceover/docs/source/_static/responsiveSvg.js:8).

Goal tracker marked complete: 321,083 tokens used, about 7m35s elapsed.
