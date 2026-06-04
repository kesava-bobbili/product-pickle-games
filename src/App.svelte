<script>
  import {
    ArrowRight,
    Check,
    Clock3,
    Link,
    ListChecks,
    RotateCcw,
    ScissorsLineDashed,
    Sparkles,
    Trophy,
  } from "lucide-svelte";
  import Rules from "./Rules.svelte";

  const games = [
    { id: "fold", name: "Fold", icon: ScissorsLineDashed, tagline: "Think before you unfold." },
    { id: "chain", name: "ChainLink", icon: Link, tagline: "Connect ideas. Find the shortest chain." },
    { id: "logic", name: "Logic Line", icon: ListChecks, tagline: "Arrange pieces. Satisfy every clue." },
  ];

  const foldPuzzle = {
    title: "Daily Fold #001",
    gridSize: 4,
    folds: ["Fold left half onto right", "Fold top half down", "Punch two holes"],
    punches: [
      [2, 2],
      [3, 3],
    ],
    answer: new Set(["0,0", "0,2", "1,1", "1,3", "2,0", "2,2", "3,1", "3,3"]),
    maxMistakes: 3,
  };

  const foldRules = [
    "The sheet is a 4 x 4 grid. Each selected tile means a visible hole after unfolding.",
    "Follow the folds in order. All layers are punched at the shown folded positions.",
    "To win, mark every final hole and no extra tiles.",
    "Three wrong submissions end the round. Hints reveal one missed tile and add a 20 second penalty.",
    "Daily score = solve time + penalties. Lower score is better.",
  ];

  const chainPuzzle = {
    start: "Rain",
    end: "Cinema",
    maxSteps: 5,
    wordBank: [
      "cloud",
      "weather",
      "forecast",
      "screen",
      "movie",
      "cinema",
      "umbrella",
      "storm",
      "camera",
      "ticket",
    ],
    links: new Set([
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
      "ticket>cinema",
    ]),
    best: 4,
  };

  const chainRules = [
    "Build a chain from the start word to the end word using only the word bank.",
    "Every neighboring pair must have a clear allowed relationship from the puzzle graph.",
    "No repeated words. The chain must finish within the step limit.",
    "A shorter valid chain ranks higher. Ties are sorted by time.",
    "The puzzle editor controls allowed links, so joke or vague associations cannot slip through.",
  ];

  const logicPuzzle = {
    pieces: ["Red", "Blue", "Green", "Yellow", "Purple"],
    colors: {
      Red: "#ef4444",
      Blue: "#2563eb",
      Green: "#16a34a",
      Yellow: "#f59e0b",
      Purple: "#7c3aed",
    },
    solution: ["Green", "Red", "Purple", "Blue", "Yellow"],
    maxChecks: 5,
  };

  const logicClues = [
    "Green is in the first position.",
    "Red is immediately left of Purple.",
    "Blue is somewhere to the right of Purple.",
    "Yellow is not in the middle.",
    "Purple is not at either end.",
  ];

  const logicRules = [
    "Place each color exactly once in the five slots.",
    "A check tells you how many clues are currently satisfied, not which ones.",
    "You win only when all clues are satisfied and all slots are filled.",
    "You have five checks. Fewer checks means a better efficiency score.",
    "If more than one solution exists, the daily puzzle is rejected before launch.",
  ];

  let active = "fold";
  let foldSelected = new Set();
  let foldMistakes = 0;
  let foldPenalty = 0;
  let foldMessage = "Mark the holes you predict after the paper unfolds.";
  let foldSolved = false;

  let chain = ["Rain", "", "", "", "", "Cinema"];
  let chainMessage = "Choose words that make a valid semantic chain.";
  let chainSolved = false;

  let logicSlots = ["", "", "", "", ""];
  let logicTray = [...logicPuzzle.pieces];
  let logicChecks = 0;
  let logicMessage = "Arrange the colors, then check the clue score.";
  let logicSolved = false;

  $: activeGame = games.find((game) => game.id === active);
  $: foldProgress = `${foldPuzzle.answer.size - missingFoldCount()} / ${foldPuzzle.answer.size}`;
  $: chainStepsUsed = chain.filter(Boolean).length - 1;
  $: logicFilled = logicSlots.filter(Boolean).length;

  function keyOf(row, col) {
    return `${row},${col}`;
  }

  function copySet(set) {
    return new Set([...set]);
  }

  function toggleFoldCell(row, col) {
    if (foldSolved) return;
    const next = copySet(foldSelected);
    const key = keyOf(row, col);
    if (next.has(key)) {
      next.delete(key);
    } else {
      next.add(key);
    }
    foldSelected = next;
    foldMessage = "Selection updated. Submit when every unfolded hole is marked.";
  }

  function missingFoldCount() {
    let missing = 0;
    foldPuzzle.answer.forEach((cell) => {
      if (!foldSelected.has(cell)) missing += 1;
    });
    return missing;
  }

  function extraFoldCount() {
    let extra = 0;
    foldSelected.forEach((cell) => {
      if (!foldPuzzle.answer.has(cell)) extra += 1;
    });
    return extra;
  }

  function submitFold() {
    if (foldSolved) return;
    const missing = missingFoldCount();
    const extra = extraFoldCount();
    if (missing === 0 && extra === 0) {
      foldSolved = true;
      foldMessage = "Perfect unfold. This would count toward the daily streak.";
      return;
    }
    foldMistakes += 1;
    foldPenalty += 15;
    foldMessage = `Not yet: ${missing} missing, ${extra} extra.`;
    if (foldMistakes >= foldPuzzle.maxMistakes) {
      foldMessage = "Round locked after 3 misses. Reset to try again.";
    }
  }

  function hintFold() {
    if (foldSolved || foldMistakes >= foldPuzzle.maxMistakes) return;
    const missing = [...foldPuzzle.answer].find((cell) => !foldSelected.has(cell));
    if (!missing) return;
    const next = copySet(foldSelected);
    next.add(missing);
    foldSelected = next;
    foldPenalty += 20;
    foldMessage = "Hint used: one correct hole was revealed.";
  }

  function resetFold() {
    foldSelected = new Set();
    foldMistakes = 0;
    foldPenalty = 0;
    foldMessage = "Mark the holes you predict after the paper unfolds.";
    foldSolved = false;
  }

  function setChainWord(index, value) {
    chain[index] = value;
    chain = [...chain];
    chainMessage = "Chain updated. Validate when ready.";
    chainSolved = false;
  }

  function validateChain() {
    const clean = chain.filter(Boolean).map((word) => word.toLowerCase());
    const unique = new Set(clean);
    if (clean[0] !== chainPuzzle.start.toLowerCase() || clean[clean.length - 1] !== chainPuzzle.end.toLowerCase()) {
      chainMessage = "A valid chain must start at Rain and finish at Cinema.";
      return;
    }
    if (unique.size !== clean.length) {
      chainMessage = "No repeated words are allowed.";
      return;
    }
    if (clean.length - 1 > chainPuzzle.maxSteps) {
      chainMessage = "Too many links. Shorten the chain.";
      return;
    }
    for (let i = 0; i < clean.length - 1; i += 1) {
      if (!chainPuzzle.links.has(`${clean[i]}>${clean[i + 1]}`)) {
        chainMessage = `${clean[i]} to ${clean[i + 1]} is not an approved link.`;
        return;
      }
    }
    chainSolved = true;
    const score = clean.length - 1;
    chainMessage = score <= chainPuzzle.best ? "Excellent. You matched the target shortest chain." : "Valid chain. Try fewer links for the leaderboard.";
  }

  function resetChain() {
    chain = ["Rain", "", "", "", "", "Cinema"];
    chainMessage = "Choose words that make a valid semantic chain.";
    chainSolved = false;
  }

  function placePiece(slotIndex, piece) {
    if (logicSolved || !piece) return;
    const nextSlots = [...logicSlots];
    const existingIndex = nextSlots.indexOf(piece);
    if (existingIndex >= 0) nextSlots[existingIndex] = "";
    nextSlots[slotIndex] = piece;
    logicSlots = nextSlots;
    logicTray = logicPuzzle.pieces.filter((item) => !logicSlots.includes(item));
    logicMessage = "Piece placed. Check when your deduction feels solid.";
  }

  function clearSlot(slotIndex) {
    if (logicSolved) return;
    logicSlots[slotIndex] = "";
    logicSlots = [...logicSlots];
    logicTray = logicPuzzle.pieces.filter((item) => !logicSlots.includes(item));
  }

  function satisfiedClues() {
    const [a, b, c, d, e] = logicSlots;
    const red = logicSlots.indexOf("Red");
    const purple = logicSlots.indexOf("Purple");
    const blue = logicSlots.indexOf("Blue");
    return [
      a === "Green",
      red >= 0 && purple === red + 1,
      blue > purple && purple >= 0,
      c !== "Yellow",
      purple > 0 && purple < 4,
    ].filter(Boolean).length;
  }

  function checkLogic() {
    if (logicSolved || logicChecks >= logicPuzzle.maxChecks) return;
    logicChecks += 1;
    const score = satisfiedClues();
    if (logicFilled < logicPuzzle.pieces.length) {
      logicMessage = `Only ${logicFilled}/5 slots filled. ${score}/5 clues currently pass.`;
      return;
    }
    if (score === logicClues.length) {
      logicSolved = true;
      logicMessage = `Solved in ${logicChecks} clue checks. Clean deduction.`;
      return;
    }
    logicMessage = `${score}/5 clues satisfied. Keep rearranging.`;
  }

  function resetLogic() {
    logicSlots = ["", "", "", "", ""];
    logicTray = [...logicPuzzle.pieces];
    logicChecks = 0;
    logicMessage = "Arrange the colors, then check the clue score.";
    logicSolved = false;
  }
</script>

<svelte:head>
  <title>Product Pickle Games</title>
</svelte:head>

<main class="shell">
  <section class="topbar">
    <div>
      <p class="eyebrow">Week 1 puzzle concepts</p>
      <h1>Product Pickle Games</h1>
    </div>
    <div class="daily-badge">
      <Clock3 size={18} />
      <span>2-5 min daily demos</span>
    </div>
  </section>

  <nav class="tabs" aria-label="Game switcher">
    {#each games as game}
      <button class:active={active === game.id} on:click={() => (active = game.id)} aria-label={`Open ${game.name}`}>
        <svelte:component this={game.icon} size={18} />
        <span>{game.name}</span>
      </button>
    {/each}
  </nav>

  <section class="game-head">
    <div>
      <p class="eyebrow">{activeGame.tagline}</p>
      <h2>{activeGame.name}</h2>
    </div>
    <div class="score-pills">
      {#if active === "fold"}
        <span><Trophy size={16} /> Holes {foldProgress}</span>
        <span>Penalty {foldPenalty}s</span>
      {:else if active === "chain"}
        <span><Trophy size={16} /> Links {Math.max(chainStepsUsed, 0)}</span>
        <span>Best {chainPuzzle.best}</span>
      {:else}
        <span><Trophy size={16} /> Checks {logicChecks}/{logicPuzzle.maxChecks}</span>
        <span>Filled {logicFilled}/5</span>
      {/if}
    </div>
  </section>

  {#if active === "fold"}
    <section class="play-area fold-layout">
      <div class="panel">
        <div class="panel-title">
          <ScissorsLineDashed />
          <div>
            <h3>{foldPuzzle.title}</h3>
            <p>Predict the final holes after all folds open.</p>
          </div>
        </div>

        <div class="fold-steps">
          {#each foldPuzzle.folds as step, index}
            <div>
              <strong>{index + 1}</strong>
              <span>{step}</span>
            </div>
          {/each}
        </div>

        <div class="fold-preview" aria-label="Folded punch preview">
          {#each Array(4) as _, row}
            {#each Array(4) as _, col}
              <span class:crease={row === 1 || col === 1} class:punch={foldPuzzle.punches.some(([r, c]) => r === row && c === col)}></span>
            {/each}
          {/each}
        </div>
      </div>

      <div class="panel board-panel">
        <h3>Unfolded answer board</h3>
        <div class="grid fold-grid" style={`--size:${foldPuzzle.gridSize}`}>
          {#each Array(foldPuzzle.gridSize) as _, row}
            {#each Array(foldPuzzle.gridSize) as _, col}
              <button
                class:selected={foldSelected.has(keyOf(row, col))}
                class:revealed={foldSolved && foldPuzzle.answer.has(keyOf(row, col))}
                disabled={foldMistakes >= foldPuzzle.maxMistakes && !foldSolved}
                on:click={() => toggleFoldCell(row, col)}
                aria-label={`Cell ${row + 1}, ${col + 1}`}
              >
                {#if foldSelected.has(keyOf(row, col)) || (foldSolved && foldPuzzle.answer.has(keyOf(row, col)))}
                  ●
                {/if}
              </button>
            {/each}
          {/each}
        </div>
        <p class="status" class:win={foldSolved}>{foldMessage}</p>
        <div class="actions">
          <button class="primary" on:click={submitFold} disabled={foldSolved || foldMistakes >= foldPuzzle.maxMistakes}>
            <Check size={18} /> Submit
          </button>
          <button on:click={hintFold} disabled={foldSolved || foldMistakes >= foldPuzzle.maxMistakes}>
            <Sparkles size={18} /> Hint
          </button>
          <button on:click={resetFold}>
            <RotateCcw size={18} /> Reset
          </button>
        </div>
      </div>
    </section>

    <Rules title="Fold launch rules" rules={foldRules} />
  {:else if active === "chain"}
    <section class="play-area">
      <div class="panel">
        <div class="chain-goal">
          <strong>{chainPuzzle.start}</strong>
          <ArrowRight size={20} />
          <strong>{chainPuzzle.end}</strong>
        </div>
        <div class="chain-inputs">
          {#each chain as word, index}
            {#if index === 0 || index === chain.length - 1}
              <div class="locked-word">{word}</div>
            {:else}
              <select value={word} on:change={(event) => setChainWord(index, event.target.value)}>
                <option value="">Pick link</option>
                {#each chainPuzzle.wordBank.filter((item) => item !== "cinema") as option}
                  <option value={option}>{option}</option>
                {/each}
              </select>
            {/if}
          {/each}
        </div>
        <p class="status" class:win={chainSolved}>{chainMessage}</p>
        <div class="actions">
          <button class="primary" on:click={validateChain}><Check size={18} /> Validate</button>
          <button on:click={resetChain}><RotateCcw size={18} /> Reset</button>
        </div>
      </div>

      <div class="panel">
        <h3>Approved word bank</h3>
        <div class="word-bank">
          {#each chainPuzzle.wordBank as word}
            <span>{word}</span>
          {/each}
        </div>
      </div>
    </section>

    <Rules title="ChainLink launch rules" rules={chainRules} />
  {:else}
    <section class="play-area logic-layout">
      <div class="panel">
        <h3>Clues</h3>
        <ol class="clues">
          {#each logicClues as clue}
            <li>{clue}</li>
          {/each}
        </ol>
      </div>

      <div class="panel">
        <h3>Sequence</h3>
        <div class="logic-slots">
          {#each logicSlots as piece, index}
            <button class="slot" on:click={() => clearSlot(index)} style={`--piece:${piece ? logicPuzzle.colors[piece] : "#d1d5db"}`}>
              <span>{index + 1}</span>
              <strong>{piece || "Empty"}</strong>
            </button>
          {/each}
        </div>
        <div class="tray">
          {#each logicTray as piece}
            <button style={`--piece:${logicPuzzle.colors[piece]}`} on:click={() => placePiece(logicSlots.findIndex((slot) => !slot), piece)} disabled={!logicSlots.includes("")}>
              {piece}
            </button>
          {/each}
        </div>
        <p class="status" class:win={logicSolved}>{logicMessage}</p>
        <div class="actions">
          <button class="primary" on:click={checkLogic} disabled={logicSolved || logicChecks >= logicPuzzle.maxChecks}>
            <Check size={18} /> Check
          </button>
          <button on:click={resetLogic}><RotateCcw size={18} /> Reset</button>
        </div>
      </div>
    </section>

    <Rules title="Logic Line launch rules" rules={logicRules} />
  {/if}
</main>
