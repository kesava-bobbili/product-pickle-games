# Product Pickle Games

Svelte prototype for the three Week 1 puzzle concepts from the PDF:

- Fold: paper-fold hole prediction with strict answer validation.
- ChainLink: semantic word-chain puzzle with editor-approved links.
- Logic Line: color sequence deduction with clue-check scoring.

## Run the Svelte app

Install dependencies in a normal Node environment:

```bash
npm install
npm run dev
```

Then open the local URL printed by Vite.

## Deploy on GitHub Pages

This project is configured for a GitHub repository named `product-pickle-games`.
After pushing to the `main` branch, GitHub Actions builds the Svelte app and deploys
the `dist` folder to GitHub Pages.

Expected live URL:

```text
https://<your-github-username>.github.io/product-pickle-games/
```

## No-install demo

If npm is not available, open:

```text
outputs/product-pickle-games-demo.html
```

That file is a self-contained playable demo with the same practical launch rules.
