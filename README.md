# My Nix presentations

* Create reveal-md shell
  (FIXME: use `nixpkgs-unstable` once https://github.com/NixOS/nixpkgs/pull/321453 is merged)

  ```bash
  nix shell github:imincik/nixpkgs/reveal-md-6.1.2#{reveal-md,chromium}
  ```

* Run presentation

  ```bash
  export PRESENTATION_DIR=<DIRECTORY>

  reveal-md $PRESENTATION_DIR/presentation.md -w
  ```

* Export presentation to PDF

  ```bash
  reveal-md $PRESENTATION_DIR/presentation.md --print-size A4 --print $PRESENTATION_DIR/presentation.pdf --puppeteer-chromium-executable $(which chromium)
  ```
