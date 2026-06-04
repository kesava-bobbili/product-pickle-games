<script>
  // @ts-nocheck
  const cells = Array.from({ length: 16 }, (_, i) => i);
  const tabs = ["Fold", "ChainLink", "Logic Line"];
  let active = $state("Fold");

  const foldAnswer = ["0,0", "0,2", "1,1", "1,3", "2,0", "2,2", "3,1", "3,3"];
  let foldSelected = $state([]);
  let foldPenalty = $state(0);
  let foldMistakes = $state(0);
  let foldSolved = $state(false);
  let foldMessage = $state("Mark every hole that will appear after the paper unfolds.");

  const chainWords = ["cloud", "forecast", "screen", "movie", "weather", "umbrella", "storm", "camera", "ticket"];
  const validLinks = [
    "rain>cloud",
    "rain>forecast",
    "cloud>weather",
    "weather>forecast",
    "forecast>screen",
    "screen>movie",
    "movie>cinema",
    "rain>umbrella",
    "umbrella>storm",
    "storm>weather",
    "screen>camera",
    "camera>movie",
    "movie>ticket",
    "ticket>cinema"
  ];
  let chain = $state(["Rain", "", "", "", "", "Cinema"]);
  let chainMessage = $state("Create the shortest approved idea chain.");
  let chainSolved = $state(false);

  const pieces = ["Red", "Blue", "Green", "Yellow", "Purple"];
  const colors = {
    Red: "#ef4444",
    Blue: "#2563eb",
    Green: "#16a34a",
    Yellow: "#f59e0b",
    Purple: "#7c3aed"
  };
  let logicSlots = $state(["", "", "", "", ""]);
  let logicChecks = $state(0);
  let logicSolved = $state(false);
  let logicMessage = $state("Place each color once, then check the clues.");

  let logicTray = $derived(pieces.filter((p) => !logicSlots.includes(p)));
  let foldCorrect = $derived(foldAnswer.filter((c) => foldSelected.includes(c)).length);
  let chainLinks = $derived(Math.max(chain.filter(Boolean).length - 1, 0));

  function cellKey(i) {
    return `${Math.floor(i / 4)},${i % 4}`;
  }

  function toggleFold(key) {
    if (foldSolved || foldMistakes >= 3) return;
    foldSelected = foldSelected.includes(key)
      ? foldSelected.filter((x) => x !== key)
      : [...foldSelected, key];
    foldMessage = "Selection updated. Submit when your prediction is complete.";
  }

  function submitFold() {
    const missing = foldAnswer.filter((x) => !foldSelected.includes(x)).length;
    const extra = foldSelected.filter((x) => !foldAnswer.includes(x)).length;

    if (missing === 0 && extra === 0) {
      foldSolved = true;
      foldMessage = "Perfect unfold. Daily streak earned.";
      return;
    }

    foldMistakes += 1;
    foldPenalty += 15;
    foldMessage = foldMistakes >= 3
      ? "Round locked after 3 misses. Reset to try again."
      : `Not yet: ${missing} missing, ${extra} extra.`;
  }

  function hintFold() {
    const next = foldAnswer.find((x) => !foldSelected.includes(x));
    if (!next || foldSolved || foldMistakes >= 3) return;
    foldSelected = [...foldSelected, next];
    foldPenalty += 20;
    foldMessage = "Hint used: one correct hole was revealed.";
  }

  function resetFold() {
    foldSelected = [];
    foldPenalty = 0;
    foldMistakes = 0;
    foldSolved = false;
    foldMessage = "Mark every hole that will appear after the paper unfolds.";
  }

  function updateChain(index, value) {
    chain[index] = value;
    chain = [...chain];
    chainSolved = false;
    chainMessage = "Chain updated. Validate when ready.";
  }

  function validateChain() {
    const clean = chain.filter(Boolean).map((x) => x.toLowerCase());

    if (clean[0] !== "rain" || clean[clean.length - 1] !== "cinema") {
      chainMessage = "The chain must start at Rain and end at Cinema.";
      return;
    }

    if (new Set(clean).size !== clean.length) {
      chainMessage = "No repeated words are allowed.";
      return;
    }

    if (clean.length - 1 > 5) {
      chainMessage = "Too many links. Use 5 links or fewer.";
      return;
    }

    for (let i = 0; i < clean.length - 1; i += 1) {
      if (!validLinks.includes(`${clean[i]}>${clean[i + 1]}`)) {
        chainMessage = `${clean[i]} to ${clean[i + 1]} is not an approved link.`;
        return;
      }
    }

    chainSolved = true;
    chainMessage = clean.length - 1 <= 4
      ? "Excellent. You matched the target shortest chain."
      : "Valid chain. Try fewer links for the leaderboard.";
  }

  function resetChain() {
    chain = ["Rain", "", "", "", "", "Cinema"];
    chainSolved = false;
    chainMessage = "Create the shortest approved idea chain.";
  }

  function placePiece(piece) {
    const empty = logicSlots.indexOf("");
    if (empty === -1 || logicSolved) return;
    logicSlots[empty] = piece;
    logicSlots = [...logicSlots];
    logicMessage = "Piece placed. Check when ready.";
  }

  function clearSlot(index) {
    if (logicSolved) return;
    logicSlots[index] = "";
    logicSlots = [...logicSlots];
  }

  function clueScore() {
    const red = logicSlots.indexOf("Red");
    const purple = logicSlots.indexOf("Purple");
    const blue = logicSlots.indexOf("Blue");

    return [
      logicSlots[0] === "Green",
      red >= 0 && purple === red + 1,
      blue > purple && purple >= 0,
      logicSlots[2] !== "Yellow",
      purple > 0 && purple < 4
    ].filter(Boolean).length;
  }

  function checkLogic() {
    if (logicSolved || logicChecks >= 5) return;

    logicChecks += 1;
    const score = clueScore();
    const filled = logicSlots.filter(Boolean).length;

    if (filled < 5) {
      logicMessage = `Only ${filled}/5 slots filled. ${score}/5 clues currently pass.`;
    } else if (score === 5) {
      logicSolved = true;
      logicMessage = `Solved in ${logicChecks} clue checks.`;
    } else {
      logicMessage = `${score}/5 clues satisfied. Keep deducing.`;
    }
  }

  function resetLogic() {
    logicSlots = ["", "", "", "", ""];
    logicChecks = 0;
    logicSolved = false;
    logicMessage = "Place each color once, then check the clues.";
  }
</script>

<main>
  <section class="hero">
    <div>
      <p>Week 1 puzzle concepts</p>
      <h1>Product Pickle Games</h1>
    </div>
    <span>2-5 min daily demos</span>
  </section>

  <nav>
    {#each tabs as tab}
      <button class:active={active === tab} onclick={() => active = tab}>
        {tab}
      </button>
    {/each}
  </nav>

  {#if active === "Fold"}
    <section class="head">
      <div>
        <p>Think before you unfold.</p>
        <h2>Fold</h2>
      </div>
      <div class="scores">
        <b>Holes {foldCorrect} / 8</b>
        <b>Penalty {foldPenalty}s</b>
      </div>
    </section>

    <section class="layout">
      <article class="card">
        <h3>Daily Fold #001</h3>
        <p>Predict the final holes after all folds open.</p>

        <div class="steps">
          <div><b>1</b> Fold left half onto right</div>
          <div><b>2</b> Fold top half down</div>
          <div><b>3</b> Punch two holes</div>
        </div>

        <div class="mini-grid">
          {#each cells as i}
            <span class:shade={Math.floor(i / 4) === 1 || i % 4 === 1} class:punch={i === 10 || i === 15}></span>
          {/each}
        </div>
      </article>

      <article class="card">
        <h3>Unfolded answer board</h3>

        <div class="fold-grid">
          {#each cells as i}
            <button
              class:selected={foldSelected.includes(cellKey(i))}
              class:revealed={foldSolved && foldAnswer.includes(cellKey(i))}
              onclick={() => toggleFold(cellKey(i))}
            >
              {foldSelected.includes(cellKey(i)) || foldSolved && foldAnswer.includes(cellKey(i)) ? "●" : ""}
            </button>
          {/each}
        </div>

        <p class:win={foldSolved} class="status">{foldMessage}</p>

        <div class="actions">
          <button class="primary" onclick={submitFold}>Submit</button>
          <button onclick={hintFold}>Hint</button>
          <button onclick={resetFold}>Reset</button>
        </div>
      </article>
    </section>

    <section class="rules">
      <h3>Fold rules</h3>
      <p>The sheet is a 4 x 4 grid. Each selected tile means a visible hole after unfolding.</p>
      <p>Follow folds in order. Every layer is punched at the visible punch location.</p>
      <p>Win by marking all final holes with no extra holes.</p>
      <p>Three wrong submissions lock the round. Hints add 20 seconds.</p>
    </section>
  {:else if active === "ChainLink"}
    <section class="head">
      <div>
        <p>Connect ideas. Find the shortest chain.</p>
        <h2>ChainLink</h2>
      </div>
      <div class="scores">
        <b>Links {chainLinks}</b>
        <b>Best 4</b>
      </div>
    </section>

    <section class="layout">
      <article class="card wide">
        <div class="goal"><strong>Rain</strong><span>to</span><strong>Cinema</strong></div>

        <div class="chain">
          {#each chain as word, i}
            {#if i === 0 || i === chain.length - 1}
              <div class="locked">{word}</div>
            {:else}
              <select value={word} onchange={(e) => updateChain(i, e.target.value)}>
                <option value="">Pick link</option>
                {#each chainWords as option}
                  <option value={option}>{option}</option>
                {/each}
              </select>
            {/if}
          {/each}
        </div>

        <p class:win={chainSolved} class="status">{chainMessage}</p>

        <div class="actions">
          <button class="primary" onclick={validateChain}>Validate</button>
          <button onclick={resetChain}>Reset</button>
        </div>
      </article>

      <article class="card">
        <h3>Approved word bank</h3>
        <div class="bank">
          {#each [...chainWords, "cinema"] as word}
            <span>{word}</span>
          {/each}
        </div>
      </article>
    </section>

    <section class="rules">
      <h3>ChainLink rules</h3>
      <p>Start at Rain and finish at Cinema.</p>
      <p>Use only the word bank. No repeated words.</p>
      <p>Every neighboring pair must be an approved semantic link.</p>
      <p>Shortest valid chain wins. Ties are ranked by time.</p>
    </section>
  {:else}
    <section class="head">
      <div>
        <p>Arrange pieces. Satisfy every clue.</p>
        <h2>Logic Line</h2>
      </div>
      <div class="scores">
        <b>Checks {logicChecks} / 5</b>
        <b>Filled {logicSlots.filter(Boolean).length} / 5</b>
      </div>
    </section>

    <section class="layout">
      <article class="card">
        <h3>Clues</h3>
        <ol>
          <li>Green is in the first position.</li>
          <li>Red is immediately left of Purple.</li>
          <li>Blue is somewhere to the right of Purple.</li>
          <li>Yellow is not in the middle.</li>
          <li>Purple is not at either end.</li>
        </ol>
      </article>

      <article class="card wide">
        <h3>Sequence</h3>

        <div class="slots">
          {#each logicSlots as piece, i}
            <button class="slot" onclick={() => clearSlot(i)} style={`--c:${piece ? colors[piece] : "#d1d5db"}`}>
              <i></i>
              <b>{piece || "Empty"}</b>
              <small>{i + 1}</small>
            </button>
          {/each}
        </div>

        <div class="tray">
          {#each logicTray as piece}
            <button onclick={() => placePiece(piece)} style={`--c:${colors[piece]}`}>
              {piece}
            </button>
          {/each}
        </div>

        <p class:win={logicSolved} class="status">{logicMessage}</p>

        <div class="actions">
          <button class="primary" onclick={checkLogic}>Check</button>
          <button onclick={resetLogic}>Reset</button>
        </div>
      </article>
    </section>

    <section class="rules">
      <h3>Logic Line rules</h3>
      <p>Place each color exactly once in the five slots.</p>
      <p>A check reveals only how many clues are satisfied.</p>
      <p>Win when all five clues pass.</p>
      <p>Fewer clue checks means a better efficiency score.</p>
    </section>
  {/if}
</main>

<style>
  :global(body) {
    margin: 0;
    background: #f7f7f4;
    color: #111827;
    font-family: Inter, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  }

  * {
    box-sizing: border-box;
  }

  main {
    width: min(1120px, 100%);
    margin: auto;
    padding: 24px;
  }

  .hero {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 16px;
    margin-bottom: 14px;
  }

  .hero p,
  .head p {
    margin: 0 0 4px;
    color: #64748b;
    font-size: 0.78rem;
    font-weight: 900;
    text-transform: uppercase;
  }

  h1 {
    margin: 0;
    font-size: clamp(2.2rem, 8vw, 4.8rem);
    line-height: 0.95;
  }

  h2 {
    margin: 0;
    font-size: clamp(2rem, 7vw, 3.5rem);
  }

  h3 {
    margin-top: 0;
  }

  .hero span,
  .scores b {
    padding: 10px 14px;
    border-radius: 999px;
    background: #e6f4f1;
    color: #0f766e;
    font-weight: 900;
    white-space: nowrap;
  }

  nav {
    display: flex;
    gap: 8px;
    margin-bottom: 24px;
    overflow-x: auto;
    padding-bottom: 4px;
  }

  button,
  select {
    min-height: 44px;
    border: 1px solid #d1d5db;
    border-radius: 8px;
    background: white;
    color: #111827;
    font: inherit;
    font-weight: 850;
    cursor: pointer;
  }

  nav button,
  .actions button {
    padding: 10px 14px;
  }

  nav button.active,
  .primary {
    border-color: #111827;
    background: #111827;
    color: white;
  }

  .head {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 16px;
    margin-bottom: 18px;
  }

  .scores {
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-end;
    gap: 8px;
  }

  .layout {
    display: flex;
    gap: 18px;
    align-items: stretch;
  }

  .card {
    flex: 1;
    min-width: 0;
    padding: 18px;
    border: 1px solid #dedbd1;
    border-radius: 8px;
    background: white;
    box-shadow: 0 18px 50px rgba(17, 24, 39, 0.08);
  }

  .wide {
    flex: 1.35;
  }

  .steps {
    display: grid;
    gap: 10px;
    margin: 18px 0;
  }

  .steps div {
    display: grid;
    grid-template-columns: 32px 1fr;
    align-items: center;
    gap: 10px;
    padding: 10px;
    border-radius: 8px;
    background: #f1f1ec;
  }

  .steps b {
    display: grid;
    width: 32px;
    height: 32px;
    place-items: center;
    border-radius: 50%;
    background: #111827;
    color: white;
  }

  .mini-grid,
  .fold-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    aspect-ratio: 1;
    border: 2px solid #111827;
  }

  .mini-grid {
    width: min(300px, 100%);
  }

  .mini-grid span,
  .fold-grid button {
    border: 1px solid #c7c3b8;
  }

  .mini-grid .shade {
    background: #eee8dc;
  }

  .mini-grid .punch {
    position: relative;
  }

  .mini-grid .punch::after {
    position: absolute;
    inset: 32%;
    border-radius: 50%;
    background: #111827;
    content: "";
  }

  .fold-grid {
    width: min(430px, 100%);
    margin: 0 auto 14px;
  }

  .fold-grid button {
    display: grid;
    place-items: center;
    min-height: 60px;
    background: #fffdf8;
    font-size: 1.5rem;
  }

  .fold-grid .selected {
    background: #fef3c7;
  }

  .fold-grid .revealed {
    background: #ccfbf1;
  }

  .status {
    min-height: 48px;
    margin: 0 0 14px;
    padding: 12px;
    border-radius: 8px;
    background: #f1f5f9;
    color: #334155;
    font-weight: 800;
  }

  .status.win {
    background: #dcfce7;
    color: #166534;
  }

  .actions {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
  }

  .rules {
    margin-top: 18px;
    padding: 18px;
    border-radius: 8px;
    background: #111827;
    color: white;
  }

  .rules p {
    margin: 8px 0 0;
    color: #e5e7eb;
  }

  .goal {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 14px;
    margin-bottom: 18px;
    font-size: clamp(1.5rem, 6vw, 3rem);
  }

  .goal span {
    color: #64748b;
    font-size: 1rem;
    font-weight: 900;
    text-transform: uppercase;
  }

  .chain {
    display: grid;
    grid-template-columns: repeat(6, 1fr);
    gap: 8px;
    margin-bottom: 14px;
  }

  .locked,
  select {
    display: grid;
    min-height: 52px;
    place-items: center;
    width: 100%;
    border-radius: 8px;
    text-align: center;
    font-weight: 900;
  }

  .locked {
    background: #fef3c7;
    border: 1px solid #d1d5db;
  }

  .bank,
  .tray {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
  }

  .bank span {
    padding: 8px 10px;
    border-radius: 999px;
    background: #eef2ff;
    color: #3730a3;
    font-weight: 900;
  }

  ol {
    display: grid;
    gap: 10px;
    margin: 0;
    padding-left: 22px;
  }

  li {
    padding: 10px;
    border-radius: 8px;
    background: #f1f1ec;
    font-weight: 750;
  }

  .slots {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 8px;
    margin-bottom: 14px;
  }

  .slot {
    display: grid;
    place-items: center;
    gap: 7px;
    min-height: 112px;
    padding: 10px;
  }

  .slot i {
    width: 34px;
    height: 34px;
    border-radius: 50%;
    background: var(--c);
  }

  .slot small {
    color: #64748b;
    font-weight: 900;
  }

  .tray {
    margin-bottom: 14px;
  }

  .tray button {
    padding: 8px 12px;
    border-color: var(--c);
    box-shadow: inset 0 -4px 0 var(--c);
  }

  @media (max-width: 780px) {
    main {
      padding: 16px;
    }

    .hero,
    .head,
    .layout {
      flex-direction: column;
      align-items: stretch;
    }

    .scores {
      justify-content: flex-start;
    }

    .chain {
      grid-template-columns: 1fr;
    }

    .slots {
      grid-template-columns: repeat(2, 1fr);
    }
  }
</style>
