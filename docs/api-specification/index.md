---
hide:
  - navigation
  - toc
---

<!-- Page title for SEO/accessibility; visually hidden -->
<h1 class="sr-only">MKE4k 4.1.2 API documentation</h1>

<!-- Page-specific CSS -->
<style>
  /* Remove default margins so ReDoc can use full height */
  .md-main__inner { margin-top: 0 !important; }
  .md-content { margin-top: 0 !important; }
  html, body { height: 100%; }

  /* Hide the header palette toggle (light/dark switch) on THIS page only */
  .md-header__option[data-md-component="palette"],
  form[data-md-component="palette"] {
    display: none !important;
  }

  /* Give ReDoc a full-viewport canvas */
  #redoc-container { height: 100vh; }

  /* Smallify function-name headers in case someone loads this page on a small screen and they hide under the examples pane */
  #redoc-container h2 {
    font-size: 1.3em !important;
  }
</style>

<!-- Force light theme on THIS page only (non-persistent) -->
<script>
  (function () {
    function forceLight() {
      document.body.setAttribute("data-md-color-scheme", "default");
      document.body.setAttribute("data-md-color-primary", "indigo");
      document.body.setAttribute("data-md-color-accent", "indigo");
      // Reflect "light" radio without touching localStorage
      var light = document.querySelector('input.md-option#\\_\\_palette_0');
      if (light) light.checked = true;
    }
    forceLight();
    document.addEventListener("DOMContentLoaded", forceLight);
  })();
</script>

<!-- ReDoc mount -->
<div id="redoc-container"></div>

<!-- ReDoc bundle -->
<script src="https://cdn.redoc.ly/redoc/latest/bundles/redoc.standalone.js"></script>

<!-- Initialize ReDoc with a robust spec URL (so this script works for all versions) -->
<script>
  (function () {
    // Resolve "../../openapi/accounts.spec.yml" relative to THIS page, safely
    var specUrl = new URL("../openapi/k0rdent-api-1.5.0.json", window.location.href).pathname;

    Redoc.init(specUrl, { scrollYOffset: 0 }, document.getElementById("redoc-container"));
  })();
</script>
