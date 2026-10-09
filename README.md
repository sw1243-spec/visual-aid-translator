# Visual Aid Translator

An offline desktop app that translates factory documents — work instructions, inspection sheets, process flow charts, training decks — while keeping the original layout. Built as a contract project for an automotive driveshaft plant in the US whose operators read Spanish, Bengali, Korean and French.

> **Source code is private** (client project). This page describes what the app does and how it was built.

공장 작업 지침서·검사 시트·공정 흐름도를 원래 레이아웃 그대로 번역하는 **오프라인** 데스크톱 앱입니다. 미국 자동차 부품 공장 계약 프로젝트로 개발했고, 소스 코드는 고객 프로젝트라 비공개입니다.

## The problem

- Plant documents are Excel sheets with merged cells and text boxes, Word manuals, PowerPoint training decks and PDF flow charts — not plain text.
- Documents may not leave the plant network: **no cloud translation API** is allowed.
- A wrong word on the line is a quality incident: part numbers, gauge IDs, "must not", GO / NO-GO must never be mistranslated.

## What it does

- **Translates Excel, Word, PowerPoint and PDF** (old .xls / .doc / .ppt are converted on the fly) into Spanish, Bengali, Korean, French, and Korean → English — and writes the translation back **into the same cells, boxes and runs**, keeping fonts, colors and positions.
- **Runs fully offline** on a normal plant PC: MADLAD-400 3B (CTranslate2, int8) on the CPU.
- **Protects what must not change**: part numbers, document numbers, gauge IDs, units, LH / RH, GO / NO-GO, run formatting markers.
- **Plant glossary** with a feedback loop: when a reviewer corrects a term, the app offers to add it, and the next file uses it.
- **Review screen** that flags only lines worth a look (numbers changed, negation lost, glossary term missing, untranslated), with bulk approval for the rest.
- **Watermarks** exported copies (TRANSLATED COPY / UNCONTROLLED COPY with the document's own date) for document control (IATF 16949).
- Skips hidden sheets, rows and columns; detects the document's source language; keeps step numbers inside PDF circles in place.

## How quality was measured

Translation quality was not judged by feel. Native speakers at the plant graded real document lines, and every correction became a rule or a glossary entry with a test that pins it:

- Spanish, Korean, Bengali and Korean → English grading sheets (≈ 360 lines) and a handwritten review of a translated flow chart by Bengali-speaking operators.
- A blind A/B comparison against a newer open model (TranslateGemma 4B) on the same 360 graded lines — the current engine stayed (265 vs 244 acceptable lines), so the engine was kept and the effort went into document handling instead.

## Stack

| Layer | Tech |
|---|---|
| Desktop shell | Python, pywebview |
| UI | Next.js (static export), TypeScript |
| Translation engine | MADLAD-400 3B · CTranslate2 int8 · SentencePiece |
| Documents | openpyxl, python-docx, python-pptx, lxml, pdfminer.six + PyMuPDF |
| Storage | SQLite (peewee) — jobs, segments, translation memory, glossary |
| Packaging | PyInstaller + Inno Setup (no admin rights needed) |
| Tests | pytest — ~950 tests, including layout round-trips on recreated plant documents |

## My role

Sole developer: requirements with the plant's quality team, architecture, the translation pipeline, all document handlers, UI, packaging, pilot rollout and the native-speaker review loop.

