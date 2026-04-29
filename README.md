# Substitution Cipher Demo

An interactive demo built for a computer class to show how a monoalphabetic
substitution cipher works. Type a message, optionally provide a keyword or
adjust the rotation, and watch the encryption update live alongside a visual
mapping between the source and cipher alphabets.

Built with [Svelte 5](https://svelte.dev/) + [Vite](https://vite.dev/) and
[Bun](https://bun.sh/). AI (Claude) was used to help build it.

## Features

- **Live encryption** — encrypts as you type.
- **Optional keyword** — type any word and the cipher alphabet is built from
  it (deduplicated, with the remaining letters of the alphabet appended).
- **Rotation slider** — when no keyword is given, a 0–25 slider acts as a
  Caesar-style shift.
- **Alphabet mapping visualization** — both alphabets are drawn as boxed rows
  with curved arrows showing every letter's mapping. Click any letter in the
  top row to highlight its mapping in yellow.
- **Light & dark mode** — colors automatically adapt to the system theme.

## Running locally

Requires [Bun](https://bun.sh/).

```bash
bun install
bun run dev
```

Then open the URL Vite prints (typically <http://localhost:5173>).

### Other scripts

```bash
bun run build      # production build to dist/
bun run preview    # preview the production build
bun run check      # type-check with svelte-check + tsc
```

## Project layout

```
src/
  App.svelte                    # root component, just mounts the demo
  main.ts                       # Svelte 5 entry point
  app.css                       # global styles + theme tokens
  lib/
    SubstitutionCipher.svelte   # the demo (inputs + SVG visualization)
```

## How the cipher is built

Given a keyword, the cipher alphabet is constructed by:

1. Uppercasing and stripping non-letters from the keyword.
2. Removing duplicate letters (keeping first occurrence).
3. Appending the remaining letters of the alphabet in order.

For example, the keyword `SECRET` produces the cipher alphabet
`SECRTABDFGHIJKLMNOPQUVWXYZ`.

If the keyword is empty, the rotation slider's value is used as a Caesar shift
instead — `rotation = 3` produces `DEFGHIJKLMNOPQRSTUVWXYZABC`.

Case is preserved on output and any non-letter characters (spaces, digits,
punctuation) pass through unchanged.
