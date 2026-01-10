# My Nix presentations

* Create reveal-md shell

  ```bash
  nix shell nixpkgs/nixos-unstable#{reveal-md,chromium}
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
