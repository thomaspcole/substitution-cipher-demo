<script lang="ts">
  let message = $state('Hello, World!');
  let keyword = $state('');
  let rotation = $state(0);

  const ALPHABET = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';

  function rotatedAlphabet(shift: number): string {
    const n = ((shift % 26) + 26) % 26;
    return ALPHABET.slice(n) + ALPHABET.slice(0, n);
  }

  function buildCipher(key: string, shift: number): string {
    const cleaned = key.toUpperCase().replace(/[^A-Z]/g, '');
    const seen = new Set<string>();
    let result = '';
    for (const ch of cleaned) {
      if (!seen.has(ch)) {
        seen.add(ch);
        result += ch;
      }
    }
    if (result.length === 0) {
      return rotatedAlphabet(shift);
    }
    for (const ch of ALPHABET) {
      if (!seen.has(ch)) result += ch;
    }
    return result;
  }

  function translate(text: string, from: string, to: string): string {
    let out = '';
    for (const ch of text) {
      const upper = ch.toUpperCase();
      const idx = from.indexOf(upper);
      if (idx === -1) {
        out += ch;
      } else {
        const mapped = to[idx];
        out += ch === upper ? mapped : mapped.toLowerCase();
      }
    }
    return out;
  }

  let usingKeyword = $derived(keyword.replace(/[^A-Za-z]/g, '').length > 0);
  let cipher = $derived(buildCipher(keyword, rotation));
  let encrypted = $derived(translate(message, ALPHABET, cipher));

  let selectedIdx = $state(0);
  let highlightTargetIdx = $derived(ALPHABET.indexOf(cipher[selectedIdx]));

  const CELL_W = 26;
  const CELL_H = 30;
  const GAP = 2;
  const PAD_X = 8;
  const TOP_Y = 10;
  const BOTTOM_Y = 160;
  const SVG_W = PAD_X * 2 + 26 * CELL_W + 25 * GAP;
  const SVG_H = BOTTOM_Y + CELL_H + 10;

  function cellX(i: number): number {
    return PAD_X + i * (CELL_W + GAP);
  }
</script>

<div class="layout">
  <div class="cipher panel">
    <label>
      Message
      <textarea bind:value={message} rows="3" placeholder="Type your message..."></textarea>
    </label>

    <label>
      Cipher keyword <span class="hint">(optional — leave blank to use rotation)</span>
      <input type="text" bind:value={keyword} placeholder="e.g. SECRET" />
    </label>

    <label class:disabled={usingKeyword}>
      Rotation: {rotation}
      <span class="hint">{usingKeyword ? '(disabled while a keyword is set)' : ''}</span>
      <input
        type="range"
        min="0"
        max="25"
        bind:value={rotation}
        disabled={usingKeyword}
      />
    </label>

    <div class="mapping">
      <div class="row"><span class="lbl">Plain:</span> <code>{ALPHABET}</code></div>
      <div class="row"><span class="lbl">Cipher:</span> <code>{cipher}</code></div>
    </div>

    <div class="output">
      <h3>Encrypted</h3>
      <pre>{encrypted}</pre>
    </div>
  </div>

  <div class="viz panel">
    <p class="hint">Click a letter in the top row. <code>{ALPHABET[selectedIdx]}</code> → <code>{cipher[selectedIdx]}</code></p>
    <svg viewBox="0 0 {SVG_W} {SVG_H}" xmlns="http://www.w3.org/2000/svg">
      {#each ALPHABET as _letter, i}
        {@const targetIdx = ALPHABET.indexOf(cipher[i])}
        {@const x1 = cellX(i) + CELL_W / 2}
        {@const y1 = TOP_Y + CELL_H}
        {@const x2 = cellX(targetIdx) + CELL_W / 2}
        {@const y2 = BOTTOM_Y}
        {@const active = i === selectedIdx}
        <path
          class:link={true}
          class:active
          d="M {x1} {y1} C {x1} {(y1 + y2) / 2}, {x2} {(y1 + y2) / 2}, {x2} {y2}"
          marker-end={active ? 'url(#arrow-active)' : 'url(#arrow-dim)'}
        />
      {/each}

      <defs>
        <marker id="arrow-active" viewBox="0 0 10 10" refX="8" refY="5"
                markerWidth="6" markerHeight="6" orient="auto-start-reverse">
          <path d="M 0 0 L 10 5 L 0 10 z" class="arrow-active" />
        </marker>
        <marker id="arrow-dim" viewBox="0 0 10 10" refX="8" refY="5"
                markerWidth="5" markerHeight="5" orient="auto-start-reverse">
          <path d="M 0 0 L 10 5 L 0 10 z" class="arrow-dim" />
        </marker>
      </defs>

      {#each ALPHABET as letter, i}
        {@const active = i === selectedIdx}
        <g class="clickable" class:active onclick={() => (selectedIdx = i)} role="button" tabindex="0">
          <rect class="cell" x={cellX(i)} y={TOP_Y} width={CELL_W} height={CELL_H} />
          <text
            class="letter"
            x={cellX(i) + CELL_W / 2}
            y={TOP_Y + CELL_H / 2 + 5}
            text-anchor="middle"
          >{letter}</text>
        </g>
      {/each}

      {#each ALPHABET as letter, i}
        {@const active = i === highlightTargetIdx}
        <g class:active>
          <rect class="cell" x={cellX(i)} y={BOTTOM_Y} width={CELL_W} height={CELL_H} />
          <text
            class="letter"
            x={cellX(i) + CELL_W / 2}
            y={BOTTOM_Y + CELL_H / 2 + 5}
            text-anchor="middle"
          >{letter}</text>
        </g>
      {/each}
    </svg>
  </div>
</div>

<style>
  /* Theme tokens — adapt to light/dark via @media */
  .layout {
    --panel-border: #d4d4d8;
    --panel-bg: transparent;
    --field-border: #a1a1aa;
    --field-bg: #fff;
    --field-text: #18181b;
    --code-bg: rgba(0, 0, 0, 0.06);
    --hint-color: #52525b;
    --svg-dim: #a1a1aa;
    --svg-stroke: #18181b;
    --svg-text-dim: #71717a;
    --svg-text: #18181b;
    --svg-active-fill: #bcdcea;
    --accent: #f5c400;
    --hover-bg: rgba(0, 0, 0, 0.04);
  }
  @media (prefers-color-scheme: dark) {
    .layout {
      --panel-border: #3f3f46;
      --field-border: #52525b;
      --field-bg: #1f1f23;
      --field-text: #f4f4f5;
      --code-bg: rgba(255, 255, 255, 0.08);
      --hint-color: #a1a1aa;
      --svg-dim: #52525b;
      --svg-stroke: #f4f4f5;
      --svg-text-dim: #a1a1aa;
      --svg-text: #f4f4f5;
      --svg-active-fill: #1d4e6b;
      --hover-bg: rgba(255, 255, 255, 0.06);
    }
  }

  .layout {
    display: flex;
    gap: 1.5rem;
    align-items: flex-start;
    flex-wrap: nowrap;
    margin: 2rem 0;
    width: 100%;
    padding: 0 1rem;
    box-sizing: border-box;
    color: var(--field-text);
  }
  .panel {
    padding: 1.5rem;
    border: 1px solid var(--panel-border);
    border-radius: 8px;
    text-align: left;
    font-family: system-ui, sans-serif;
    box-sizing: border-box;
  }
  .cipher { flex: 0 0 460px; }
  .viz { flex: 1 1 auto; min-width: 0; }
  .viz svg { width: 100%; height: auto; display: block; margin-top: 0.5rem; }

  label {
    display: block;
    margin: 1rem 0;
    font-weight: 600;
    color: var(--field-text);
  }
  label.disabled { opacity: 0.5; }
  .hint { font-weight: 400; color: var(--hint-color); font-size: 0.85em; }

  textarea, input[type="text"] {
    display: block;
    width: 100%;
    margin-top: 0.35rem;
    padding: 0.5rem;
    font-size: 1rem;
    font-family: inherit;
    box-sizing: border-box;
    border: 1px solid var(--field-border);
    border-radius: 4px;
    background: var(--field-bg);
    color: var(--field-text);
  }
  textarea:focus, input[type="text"]:focus {
    outline: 2px solid var(--accent);
    outline-offset: 1px;
    border-color: var(--accent);
  }
  input[type="range"] {
    display: block;
    width: 100%;
    margin-top: 0.35rem;
    accent-color: var(--accent);
  }

  .mapping {
    background: var(--code-bg);
    padding: 0.75rem;
    border-radius: 4px;
    font-size: 0.95rem;
  }
  .row { display: flex; gap: 0.5rem; }
  .lbl { width: 4rem; font-weight: 600; }
  code { letter-spacing: 2px; color: var(--field-text); }
  .output pre {
    background: var(--code-bg);
    color: var(--field-text);
    padding: 0.75rem;
    border-radius: 4px;
    white-space: pre-wrap;
    word-break: break-word;
    margin: 0.25rem 0 1rem;
  }
  .output h3 { color: var(--field-text); }

  /* SVG styling */
  svg .link {
    fill: none;
    stroke: var(--svg-dim);
    stroke-width: 1;
    stroke-dasharray: 2 3;
  }
  svg .link.active {
    stroke: var(--accent);
    stroke-width: 2.5;
    stroke-dasharray: 0;
  }
  svg .arrow-active { fill: var(--accent); }
  svg .arrow-dim { fill: var(--svg-dim); }

  svg .cell {
    fill: transparent;
    stroke: var(--svg-dim);
    stroke-width: 1;
    transition: fill 0.1s, stroke 0.1s;
  }
  svg g.active .cell {
    fill: var(--svg-active-fill);
    stroke: var(--svg-stroke);
    stroke-width: 1.5;
  }
  svg .letter {
    font-family: system-ui, sans-serif;
    font-size: 15px;
    font-weight: 400;
    fill: var(--svg-text-dim);
  }
  svg g.active .letter {
    fill: var(--svg-text);
    font-weight: 700;
  }

  svg g.clickable { cursor: pointer; }
  svg g.clickable:hover .cell {
    stroke: var(--svg-stroke);
    fill: var(--hover-bg);
  }
  svg g.clickable:hover .letter {
    fill: var(--svg-text);
  }
  svg g.clickable:focus { outline: none; }
  svg g.clickable:focus-visible .cell {
    stroke: var(--accent);
    stroke-width: 2;
  }
</style>
