# Rise Eleven: código completo do site (rise11-site)

Gerado em 13/11/2026 a partir da branch `main` (commit f156157 Site do Rise Eleven).
Só arquivos versionados no Git; sem imagens e fontes.

## Arquivos

```
.nojekyll
assets/backgrounds/backgrounds.css
assets/backgrounds/backgrounds.js
assets/icons/icon.css
assets/layout/layout.css
assets/layout/layout.js
assets/moments/lib.js
assets/moments/moments.js
assets/moments/pinned-chapters/meta.json
assets/moments/pinned-chapters/moment.css
assets/moments/pinned-chapters/moment.js
assets/motion/motion.css
assets/motion/motion.js
assets/system.css
assets/system.js
assets/ui/base.css
assets/ui/cards.css
assets/ui/chat/fragment.css
assets/ui/chat/fragment.js
assets/ui/checklist/fragment.css
assets/ui/checklist/fragment.js
assets/ui/notify/fragment.css
assets/ui/notify/fragment.js
assets/ui/search/fragment.css
assets/ui/search/fragment.js
assets/ui/toggles/fragment.css
assets/ui/toggles/fragment.js
assets/ui/ui.js
assets/ui/window/fragment.css
assets/ui/window/fragment.js
index.html
```

## .nojekyll

```

```

## assets/backgrounds/backgrounds.css

```css
/*
  taste-skill backgrounds.css - 16 section and page grounds, drawn in CSS (no WebGL, no images except the
  photograph ground's own). Link after system.css and assets/ui/cards.css. Rules: references/backgrounds.md.

  A ground is one class on a <section> (or on <body> / a wrapper for the two page grounds, or on a panel for
  .bg-stage):
    <section class="section bg-halo"> ... </section>
  Every ground sets the page tokens for its own children (--bg --surface --ink --muted --accent --card
  --action --line, --fx-*), so the system's cards, buttons and fragments work on it unchanged. It draws on
  the page palette (--p-*) and the lit ground (--ground-*, system.css). Body text keeps 7:1 and muted text
  4.5:1 on its text zone (index.json "textZone"; check.mjs measures it on every palette). The one
  exception is .bg-stage, a panel for a visual: words on it sit on a card, never on the stage itself.

  Knobs, all optional, on :root or on one section:
    --g-hue      the colour the tints lean to (default: the accent, hue-guarded: never violet or lavender)
    --g-hue-b    its neighbour, 30 degrees round the wheel (default derived, and guarded the same way)
    --g-k, --g-kl, --g-kd   strength of the tints, of the lines, of the dark grounds: 1 at most; backgrounds.js
                 sets them from the palette's contrast headroom (see there)
    --tone       the second brand tone of .bg-band / .bg-field (default: the lit ground, system.css)
    --tone-ink   its text colour (default: the lit ground's ink)
    --g-photo    url(...) for .bg-photo
  <html data-mode="light|dark"> says whether the page ink is dark or light; <html data-tint="away"> marks a
  palette whose bg is a strong colour or a mid tone (tints then move away from the ink); <html
  data-lowchroma> marks a grey accent (the stage then takes the lit ground's colour). backgrounds.js sets
  them and --g-k / --g-kl / --g-kd from the palette, unless the page already has (load it plain in <head>).
*/

/* ---------------------------------------------------------------- tokens */
:root {
  /* the hue the tints lean to: the accent, with an oklch hue in 262-338 (violet, lavender, purple) moved
     to azure (262-300) or pink (300-338), so a cobalt or plum accent never tints a ground violet */
  --g-hue-src: var(--p-accent);
  /* the tint's lightness is held between these, so a pale lime on a dark page or a navy on a white one
     tints as deeply as a mid blue would */
  --g-L-lo: 0.62; --g-L-hi: 1;
  --g-hue: oklch(from var(--g-hue-src) clamp(var(--g-L-lo), l, var(--g-L-hi)) c
    calc(h + clamp(0, (h - 262) * 1000, 1) * clamp(0, (338 - h) * 1000, 1)
      * (250 + clamp(0, (h - 300) * 1000, 1) * 95 - h)));
  /* the neighbour: 30 degrees toward the cooler side (toward the warmer side for a red or a pink, hues under 40
     or over 330, so it never turns rose-violet). Any colour's own hue goes through this:
     oklch(from X l c calc(h + var(--g-hs))) */
  --g-hs: calc(30 - 60 * clamp(0, (h - 40) * 1000, 1) * clamp(0, (330 - h) * 1000, 1));
  --g-hue-b: oklch(from var(--g-hue) l c calc(h + var(--g-hs)));
  /* the hue as a pale pool, its lightness fixed: for the second-tone grounds, whose ink is dark whatever the page is */
  --g-pool: oklch(from var(--g-hue) 0.8 calc(c * 0.6) h);
  /* a second, different tone for two-colour fields: the lit ground (sand, sky, sage, painting) */
  --g-hue-2: var(--ground-bg, #d9ccb7);
  /* ...as a tint: its lightness held to the same band as --g-hue, so the contrast headroom (--g-k) counts for it too.
     The soft grounds take it as their SECOND hue (not the accent's 30-degree neighbour), so a blue page is not blue
     throughout: a blue wash with a sand light in the corner, as on Figma's colour fields */
  --g-hue-t: oklch(from var(--g-hue-2) clamp(var(--g-L-lo), l, var(--g-L-hi)) c h);
  /* the accent's own hue with the same guard: the stage uses it on a strong-colour page (data-tint="away"),
     where --g-hue is the page's own colour and a stage in it would be a muddy tan on the same colour */
  --g-acc: oklch(from var(--p-accent) 0.7 c calc(h + clamp(0, (h - 262) * 1000, 1) * clamp(0, (338 - h) * 1000, 1)
    * (250 + clamp(0, (h - 300) * 1000, 1) * 95 - h)));
  /* black when the page ink is light, white when it is dark: the way a shade moves away from the ink */
  --g-away: oklab(from var(--p-ink) calc(1 - round(l)) 0 0);
  --g-k: 1; --g-kl: 1; --g-kd: 1; --g-kp: 1;

  /* the dark side: the page's inverse on a light page, its own bg on a dark one (data-mode) */
  --g-dark-bg: var(--p-inv-bg);          --g-dark-surface: var(--p-inv-surface);
  --g-dark-ink: var(--p-inv-ink);        --g-dark-muted: var(--p-inv-muted);
  --g-dark-accent: var(--p-inv-accent);  --g-dark-accent-ink: var(--p-inv-accent-ink);
  --g-dark-action: var(--p-inv-action);  --g-dark-action-ink: var(--p-inv-action-ink);
  --g-light-bg: var(--p-bg);             --g-light-ink: var(--p-ink);        --g-light-muted: var(--p-muted);
  --g-light-action: var(--p-action);     --g-light-action-ink: var(--p-action-ink);
  --g-light-accent: var(--p-accent);     --g-light-accent-ink: var(--p-accent-ink);
  --g-card: var(--p-bg);                 /* a card on a tinted light ground is the page's own white */
  --g-grain: var(--g-noise-mul); --g-grain-mode: multiply;   /* grain on a light page darkens, on a dark page lightens */

  /* a cover over the grain, so it thins out with --g-k (--g-kd on the dark grounds) */
  --g-cover: color-mix(in oklab, var(--bg) calc(100% - 100% * var(--g-k)), transparent);
  --g-cover-d: color-mix(in oklab, var(--bg) calc(100% - 100% * var(--g-kd)), transparent);
  --g-fade: 120px;                       /* the soft grounds fade into the page over this much at top and bottom */
  --g-edge: 160px;                       /* .bg-dusk: how long the tone change takes */

  /* grain: SVG fractal noise as a data URI, 240px tile, two octaves at 1.2 so it is fine, like film, never
     like sand. soft is mean grey (neutral under soft-light, for the mid-tone grounds), mul sits near white
     (multiply, for light pages), scr near black (screen, for dark) */
  --g-noise-soft: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='240' height='240'%3E%3Cfilter id='n' x='0' y='0' width='100%25' height='100%25' color-interpolation-filters='sRGB'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='1.2' numOctaves='2' stitchTiles='stitch'/%3E%3CfeColorMatrix values='1 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 1'/%3E%3CfeComponentTransfer%3E%3CfeFuncR type='linear' slope='0.6' intercept='.2'/%3E%3CfeFuncG type='linear' slope='0.6' intercept='.2'/%3E%3CfeFuncB type='linear' slope='0.6' intercept='.2'/%3E%3C/feComponentTransfer%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
  --g-noise-mul: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='240' height='240'%3E%3Cfilter id='n' x='0' y='0' width='100%25' height='100%25' color-interpolation-filters='sRGB'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='1.2' numOctaves='2' stitchTiles='stitch'/%3E%3CfeColorMatrix values='1 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 1'/%3E%3CfeComponentTransfer%3E%3CfeFuncR type='linear' slope='.26' intercept='.84'/%3E%3CfeFuncG type='linear' slope='.26' intercept='.84'/%3E%3CfeFuncB type='linear' slope='.26' intercept='.84'/%3E%3C/feComponentTransfer%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
  --g-noise-scr: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='240' height='240'%3E%3Cfilter id='n' x='0' y='0' width='100%25' height='100%25' color-interpolation-filters='sRGB'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='1.2' numOctaves='2' stitchTiles='stitch'/%3E%3CfeColorMatrix values='1 0 0 0 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 1'/%3E%3CfeComponentTransfer%3E%3CfeFuncR type='linear' slope='.17' intercept='-.05'/%3E%3CfeFuncG type='linear' slope='.17' intercept='-.05'/%3E%3CfeFuncB type='linear' slope='.17' intercept='-.05'/%3E%3C/feComponentTransfer%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
}
html[data-mode="dark"] {
  --g-L-lo: 0; --g-L-hi: 0.55;
  --g-dark-bg: var(--p-bg);              --g-dark-surface: var(--p-surface);
  --g-dark-ink: var(--p-ink);            --g-dark-muted: var(--p-muted);
  --g-dark-accent: var(--p-accent);      --g-dark-accent-ink: var(--p-accent-ink);
  --g-dark-action: var(--p-action);      --g-dark-action-ink: var(--p-action-ink);
  --g-light-bg: var(--p-inv-bg);         --g-light-ink: var(--p-inv-ink);    --g-light-muted: var(--p-inv-muted);
  --g-light-action: var(--p-inv-action); --g-light-action-ink: var(--p-inv-action-ink);
  --g-light-accent: var(--p-inv-accent); --g-light-accent-ink: var(--p-inv-accent-ink);
  --g-card: var(--p-surface);            /* a card on a dark ground is one step lighter than it */
  --g-grain: var(--g-noise-scr); --g-grain-mode: screen;
}
/* a palette whose bg is a strong colour (cobalt, mustard, pink) or a mid tone (stone, slate): the accent is
   another hue, and mixing the two turns the ground to mud, so the tints are the bg's own colour pushed
   away from the ink: a deeper cobalt on white text, a paler mustard on black text */
html[data-tint="away"][data-mode="light"] { --g-grain: var(--g-noise-scr); --g-grain-mode: screen; }
html[data-tint="away"][data-mode="dark"] { --g-grain: var(--g-noise-mul); --g-grain-mode: multiply; }
html[data-tint="away"] { --g-hue-src: color-mix(in oklab, var(--p-bg) 58%, var(--g-away)); --g-hue-2: color-mix(in oklab, color-mix(in oklab, var(--p-bg) 66%, var(--g-away)) 90%, var(--p-accent)); }

/* ---------------------------------------------------------------- the contract of every ground */
.bg-mist, .bg-halo, .bg-grain, .bg-dusk, .bg-ruled, .bg-dots, .bg-arcs,
.bg-band, .bg-field, .bg-stage, .bg-photo, .bg-horizon, .bg-deep, .bg-liquid {
  position: relative; isolation: isolate; overflow: clip;
  --bg: var(--p-bg); --surface: var(--p-surface); --ink: var(--p-ink);
  /* muted leans to the ink (60/40), so a tint under it never costs the 4.5:1 (backgrounds.js counts on 60/40) */
  --muted: color-mix(in oklab, var(--p-ink) 60%, var(--p-muted));
  --accent: var(--p-accent); --accent-ink: var(--p-accent-ink);
  --action: var(--p-action); --action-ink: var(--p-action-ink);
  --card: var(--g-card);
  --line: color-mix(in oklab, var(--ink) 11%, transparent);
  --line-strong: color-mix(in oklab, var(--ink) 20%, transparent);
  --fx-bg: var(--bg); --fx-surface: var(--surface); --fx-ink: var(--ink); --fx-muted: var(--muted);
  --fx-accent: var(--accent); --fx-accent-ink: var(--accent-ink);
  --ui-font: var(--font-text); --ui-mono: var(--font-mono); --ui-radius: var(--r-md);
  background-color: var(--bg); color: var(--ink);
}
/* the two page grounds (.bg-paper on <body>, .bg-topglow on a wrapper) set NO tokens: the page keeps its own --card,
   --muted, --line and --accent (a ground that restyled the body would turn every card flat). They paint only
   background-image and ::before. */
.bg-paper, .bg-topglow { position: relative; isolation: isolate; }
.bg-paper { background-color: var(--bg); }   /* the base the grain blends with */
/* the soft grounds are overlays: they paint no colour of their own, so they sit on the page ground (plain or .bg-paper) */
:is(.bg-mist, .bg-halo, .bg-ruled, .bg-dots, .bg-arcs, .bg-liquid) { background-color: transparent; }
/* the decorations of the soft grounds sit on ::before, behind the content, and fade into the page at
   the top and bottom, so a soft ground can neighbour any other */
:is(.bg-mist, .bg-halo, .bg-ruled, .bg-dots, .bg-arcs)::before {
  content: ""; position: absolute; inset: 0; z-index: -1; pointer-events: none;
  -webkit-mask-image: linear-gradient(to bottom, transparent, #000 var(--g-fade), #000 calc(100% - var(--g-fade)), transparent);
          mask-image: linear-gradient(to bottom, transparent, #000 var(--g-fade), #000 calc(100% - var(--g-fade)), transparent);
}

/* dark grounds take the dark side's tokens */
.bg-horizon, .bg-deep, .bg-photo {
  --bg: color-mix(in oklab, var(--g-dark-bg) 80%, black);
  --surface: color-mix(in oklab, var(--g-dark-surface) 80%, black);
  --ink: var(--g-dark-ink);
  --muted: color-mix(in oklab, var(--g-dark-ink) 60%, var(--g-dark-muted));
  --accent: var(--g-dark-accent); --accent-ink: var(--g-dark-accent-ink);
  --action: var(--g-dark-action); --action-ink: var(--g-dark-action-ink);
  --card: color-mix(in oklab, var(--bg) 88%, var(--ink));
  /* the light on a dark ground: the ground's own colour and the hue, then pulled toward the ink */
  --lit: color-mix(in oklab, color-mix(in oklab, var(--bg) 55%, var(--g-hue)) 68%, var(--ink));
}
/* the second-tone grounds take the lit ground's tokens (sand, sky, sage, painting), or --tone */
.bg-band, .bg-field {
  /* a little lighter than the lit ground, so its dark ink keeps 7:1 on all four (painting is the darkest) */
  --bg: var(--tone, color-mix(in oklab, var(--ground-bg) 55%, var(--ground-surface)));
  --surface: var(--ground-surface);
  --ink: var(--tone-ink, color-mix(in oklab, var(--ground-ink) 85%, black));
  --muted: color-mix(in oklab, var(--ink) 60%, var(--ground-muted));
  --accent: var(--ground-accent); --accent-ink: var(--ground-surface);
  --action: var(--ink); --action-ink: var(--bg);
  --card: color-mix(in oklab, var(--ground-surface) 70%, white);
  /* the tone's own neighbours, so a field is one family of colour and never two that turn to mud: a richer
     neighbour 30 degrees round the wheel (a peach beside sand, cyan beside sky) and a deeper shade of itself */
  --t-b: oklch(from var(--bg) l clamp(0, c * 1.6 + 0.022, 0.075) calc(h + var(--g-hs)));
  --t-c: oklch(from var(--bg) calc(l - 0.032) calc(c * 1.3 + 0.012) h);
}

/* .bg-inset: a hard ground as a rounded panel on the page's grid, its left and right edges on the container's
   (Figma's colour panels, x.ai's video card). Add it next to the ground class: class="section bg-field bg-inset".
   On a dark page a band or a field is always inset (a pale full-bleed slab is the brightest thing on a dark page and
   harsh at a phone's width); .bg-bleed opts out. */
.bg-inset, html[data-mode="dark"] :is(.bg-band, .bg-field):not(.bg-bleed) { width: min(calc(100% - 2 * var(--pad)), var(--container)); margin-inline: auto; margin-block: var(--s-5); border-radius: var(--r-lg); }

/* Spacing. A ground that changes the colour is spaced like the system's own bands, and a little more; a soft one a half
   step more than a plain section; and each holds a screen of its own (min-height, the words centred), so a shot of one
   section never shows the next. The system's own variants keep what they set: the hero (.section--hero: its top, its
   screen-high box), .section--flush-top and .section--rule are never touched. --g-screen on a section sets its minimum
   height (0px = the words' own height). */
.section:is(.bg-band, .bg-field, .bg-grain, .bg-horizon, .bg-deep, .bg-photo):not(.section--hero, .section--flush-top, .section--rule) { padding-block: calc(var(--section-band) * 1.25); }
.section:is(.bg-mist, .bg-halo, .bg-ruled, .bg-dots, .bg-arcs, .bg-liquid):not(.section--hero, .section--flush-top, .section--rule) { padding-block: calc(var(--section) * 1.5); }
/* a hero on a ground keeps its system top and its screen height and gets the ground's room under it */
.section--hero:is(.bg-band, .bg-field, .bg-grain, .bg-horizon, .bg-deep, .bg-photo) { padding-bottom: calc(var(--section-band) * 1.25); }
.section--hero:is(.bg-mist, .bg-halo, .bg-ruled, .bg-dots, .bg-arcs, .bg-liquid) { padding-bottom: calc(var(--section) * 1.5); }
.section:is(.bg-band, .bg-field, .bg-grain, .bg-dusk, .bg-horizon, .bg-deep):not(.section--hero) { min-height: var(--g-screen, min(88svh, 860px)); display: grid; align-content: center; }
.section:is(.bg-mist, .bg-halo, .bg-ruled, .bg-dots, .bg-liquid):not(.section--hero) { min-height: var(--g-screen, min(76svh, 760px)); display: grid; align-content: center; }
/* on a phone the stacked words already fill the screen: the sections keep their padding and no minimum height */
@media (max-width: 767px) {
  .section:is(.bg-band, .bg-field, .bg-grain, .bg-dusk, .bg-horizon, .bg-deep, .bg-mist, .bg-halo, .bg-ruled, .bg-dots, .bg-liquid):not(.section--hero) { min-height: var(--g-screen, 0px); }
}
/* system.css forces the last section before the footer to one plain bottom padding (!important): a ground that closes the
   page keeps its own (the horizon's dome needs its 150px; a band stays even top and bottom) */
main > section.section:is(.bg-band, .bg-field, .bg-grain, .bg-deep, .bg-photo):not(.section--hero):last-of-type { padding-bottom: calc(var(--section-band) * 1.25) !important; }
main > section.section:is(.bg-mist, .bg-halo, .bg-ruled, .bg-dots, .bg-arcs, .bg-liquid):not(.section--hero):last-of-type { padding-bottom: calc(var(--section) * 1.5) !important; }
main > section.section.bg-horizon:last-of-type { padding-bottom: calc(var(--section-band) * 1.25 + 150px) !important; }

/* ---------------------------------------------------------------- 1 mist: a soft gradient field */
.bg-mist::before {
  background:
    radial-gradient(54% 70% at 4% 6%, color-mix(in oklab, var(--g-hue) calc(42% * var(--g-k)), transparent), transparent),
    radial-gradient(46% 62% at 98% 16%, color-mix(in oklab, var(--g-hue-t) calc(48% * var(--g-k)), transparent), transparent),
    radial-gradient(64% 64% at 66% 116%, color-mix(in oklab, var(--g-hue) calc(24% * var(--g-k)), transparent), transparent),
    radial-gradient(34% 46% at 22% 104%, color-mix(in oklab, var(--g-hue-b) calc(18% * var(--g-k)), transparent), transparent);
}

/* ---------------------------------------------------------------- 2 halo: a radial light behind the heading */
.bg-halo { --halo-x: 50%; --halo-y: 38%; }
.bg-halo::before {
  background:
    radial-gradient(34% 40% at var(--halo-x) var(--halo-y), color-mix(in oklab, var(--g-hue) calc(30% * var(--g-k)), transparent), transparent),
    radial-gradient(70% 76% at var(--halo-x) var(--halo-y), color-mix(in oklab, var(--g-hue) calc(15% * var(--g-k)), transparent), transparent),
    radial-gradient(30% 34% at calc(var(--halo-x) + 18%) calc(var(--halo-y) + 16%), color-mix(in oklab, var(--g-hue-t) calc(32% * var(--g-k)), transparent), transparent);
}

/* ---------------------------------------------------------------- 3 grain: a tint with fine grain on it */
.bg-grain {
  --bg: color-mix(in oklab, var(--p-bg) calc(100% - 11% * var(--g-k)), var(--g-hue));
  --surface: color-mix(in oklab, var(--p-surface) calc(100% - 11% * var(--g-k)), var(--g-hue));
  background-image:
    radial-gradient(90% 100% at 0% 0%, color-mix(in oklab, var(--p-bg) 74%, transparent), transparent 70%),
    radial-gradient(54% 80% at 100% 0%, color-mix(in oklab, var(--g-hue-b) calc(20% * var(--g-k)), transparent), transparent 72%),
    radial-gradient(80% 90% at 100% 100%, color-mix(in oklab, var(--g-hue) calc(26% * var(--g-k)), transparent), transparent 70%),
    radial-gradient(46% 70% at 0% 100%, color-mix(in oklab, var(--g-hue-b) calc(12% * var(--g-k)), transparent), transparent 72%),
    linear-gradient(var(--g-cover), var(--g-cover)),
    var(--g-grain);
  background-blend-mode: normal, normal, normal, normal, normal, var(--g-grain-mode);
}

/* ---------------------------------------------------------------- 4 paper: the page ground, never flat */
.bg-paper {
  overflow: visible;
  background-image:
    radial-gradient(120% 60vh at 50% 0, color-mix(in oklab, var(--g-hue) calc(9% * var(--g-kp)), transparent), transparent),
    radial-gradient(60% 50vh at 100% 0, color-mix(in oklab, var(--g-hue-b) calc(6% * var(--g-kp)), transparent), transparent),
    linear-gradient(to bottom, transparent 70%, color-mix(in oklab, var(--g-hue) calc(6% * var(--g-kp)), transparent)),
    /* the page ground carries every word of the page, so its grain is a quarter of the grain ground's */
    linear-gradient(color-mix(in oklab, var(--bg) calc(100% - 28% * var(--g-kp)), transparent), color-mix(in oklab, var(--bg) calc(100% - 28% * var(--g-kp)), transparent)),
    var(--g-grain);
  background-blend-mode: normal, normal, normal, normal, var(--g-grain-mode);
}

/* ---------------------------------------------------------------- 5 dusk: one tone moves into the next, no edge */
.bg-dusk {
  /* the two ends are transparent, so the tone melts into whatever the page really paints there (plain, .bg-paper's grain, a
     neighbouring tint) and never meets a page colour it was not. Set --g-from / --g-next to a colour to melt from or into one. */
  --g-from: transparent; --g-to: color-mix(in oklab, var(--p-bg) calc(100% - 15% * var(--g-k)), var(--g-hue)); --g-next: transparent;
  --bg: var(--g-to);
  --surface: color-mix(in oklab, var(--p-surface) calc(100% - 15% * var(--g-k)), var(--g-hue));
  background-color: transparent;
  background-repeat: no-repeat;
  /* the two melting edges and, between them (a pixel of overlap each side), the solid tone */
  background-size: 100% var(--g-edge), 100% var(--g-edge), 100% calc(100% - 2 * var(--g-edge) + 2px);
  background-position: top, bottom, center;
  /* a smoothstep: 0, .04, .16, .32, .5, .68, .84, .96, 1 of the way */
  background-image:
    linear-gradient(to bottom,
      var(--g-from), color-mix(in oklab, var(--g-to) 4%, var(--g-from)) 12.5%, color-mix(in oklab, var(--g-to) 16%, var(--g-from)) 25%,
      color-mix(in oklab, var(--g-to) 32%, var(--g-from)) 37.5%, color-mix(in oklab, var(--g-to) 50%, var(--g-from)) 50%,
      color-mix(in oklab, var(--g-to) 68%, var(--g-from)) 62.5%, color-mix(in oklab, var(--g-to) 84%, var(--g-from)) 75%,
      color-mix(in oklab, var(--g-to) 96%, var(--g-from)) 87.5%, var(--g-to)),
    linear-gradient(to top,
      var(--g-next), color-mix(in oklab, var(--g-to) 4%, var(--g-next)) 12.5%, color-mix(in oklab, var(--g-to) 16%, var(--g-next)) 25%,
      color-mix(in oklab, var(--g-to) 32%, var(--g-next)) 37.5%, color-mix(in oklab, var(--g-to) 50%, var(--g-next)) 50%,
      color-mix(in oklab, var(--g-to) 68%, var(--g-next)) 62.5%, color-mix(in oklab, var(--g-to) 84%, var(--g-next)) 75%,
      color-mix(in oklab, var(--g-to) 96%, var(--g-next)) 87.5%, var(--g-to)),
    linear-gradient(var(--g-to), var(--g-to));
}
.section.bg-dusk:not(.section--hero) { padding-block: calc(var(--section) + var(--g-edge) * .5); }

/* ---------------------------------------------------------------- 6 topglow: light under the nav, the page lifts out of it */
/* on <body> (with bg-paper) or a wrapper that holds the nav and the hero: a pool of light and a thin lit line just
   under the nav bar, fading out by 700px down, a second tone leaning in from the right. It draws on ::before, so it
   stacks with bg-paper on one element. */
.bg-topglow { overflow: visible; }
.bg-topglow::before {
  content: ""; position: absolute; z-index: -1; top: 0; left: 0; right: 0; height: min(100%, 720px); pointer-events: none;
  background:
    linear-gradient(90deg, transparent, color-mix(in oklab, var(--g-hue) calc(60% * var(--g-kp)), transparent) 50%, transparent) 50% var(--nav-h, 64px) / 62% 1px no-repeat,
    radial-gradient(62% 470px at 50% var(--nav-h, 64px), color-mix(in oklab, var(--g-hue) calc(27% * var(--g-kp)), transparent), transparent),
    radial-gradient(42% 380px at 86% calc(var(--nav-h, 64px) + 60px), color-mix(in oklab, var(--g-hue-b) calc(18% * var(--g-kp)), transparent), transparent),
    radial-gradient(120% 300px at 50% var(--nav-h, 64px), color-mix(in oklab, var(--g-hue) calc(12% * var(--g-kp)), transparent), transparent);
}

/* ---------------------------------------------------------------- 7 ruled: a fine grid that fades out */
/* square rules (Vercel's and Linear's heroes). The glow is ::before, full width; the grid is ::after, exactly as wide as the
   container: its first rule lies on the container's left edge, the cells are spaced to fit, and its right edge is a rule too, so
   the grid is the page's own frame and never a second system across it. It is clear behind the heading (a hole in the mask)
   and fades toward the top and bottom. For a section with no row rules of its own: never over a table, pricing or a list
   (index.json: ruled is dealt only to the sections that list it). */
.bg-ruled::before {
  background: radial-gradient(50% 44% at 50% 0%, color-mix(in oklab, var(--g-hue) calc(20% * var(--g-k)), transparent), transparent);
}
.bg-ruled::after {
  content: ""; position: absolute; z-index: -1; top: 0; bottom: 0; pointer-events: none; box-sizing: border-box;
  --rule: color-mix(in oklab, var(--ink) calc(8% * var(--g-kl)), transparent);
  --cell: 72px;
  left: max(var(--pad), calc((100% - var(--container)) / 2)); right: max(var(--pad), calc((100% - var(--container)) / 2));
  border-right: 1px solid var(--rule);
  background:
    linear-gradient(90deg, var(--rule) 1px, transparent 1px) 0 0 / var(--cell) var(--cell) space repeat,
    linear-gradient(var(--rule) 1px, transparent 1px) 0 0 / var(--cell) var(--cell) repeat;
  -webkit-mask-image: linear-gradient(to bottom, transparent, #000 var(--g-fade), #000 calc(100% - var(--g-fade)), transparent), radial-gradient(ellipse 120% 90% at 50% 40%, #000 30%, transparent 92%), radial-gradient(ellipse 46% 34% at 50% 38%, transparent 40%, #000 100%);
          mask-image: linear-gradient(to bottom, transparent, #000 var(--g-fade), #000 calc(100% - var(--g-fade)), transparent), radial-gradient(ellipse 120% 90% at 50% 40%, #000 30%, transparent 92%), radial-gradient(ellipse 46% 34% at 50% 38%, transparent 40%, #000 100%);
  -webkit-mask-composite: source-in; mask-composite: intersect;
}
@media (max-width: 767px) { .bg-ruled::after { --cell: 48px; } }

/* ---------------------------------------------------------------- 8 dots: a dot field that fades out */
/* the dots live in the margins and the corners and are clear behind the heading and the words (a hole in the middle of the mask,
   not a spotlight on it), at 24% of the ink, so no line of body copy ever sits on them */
.bg-dots::before {
  background:
    radial-gradient(34% 46% at 6% 34%, color-mix(in oklab, var(--g-hue) calc(16% * var(--g-k)), transparent), transparent),
    radial-gradient(30% 40% at 96% 70%, color-mix(in oklab, var(--g-hue-t) calc(24% * var(--g-k)), transparent), transparent),
    radial-gradient(circle at 11px 11px, color-mix(in oklab, var(--ink) calc(26% * var(--g-kl)), transparent) 1.15px, transparent 1.9px) 0 0 / 22px 22px;
  -webkit-mask-image: linear-gradient(to bottom, transparent, #000 var(--g-fade), #000 calc(100% - var(--g-fade)), transparent), radial-gradient(ellipse 58% 70% at 50% 40%, transparent 24%, #000 78%);
          mask-image: linear-gradient(to bottom, transparent, #000 var(--g-fade), #000 calc(100% - var(--g-fade)), transparent), radial-gradient(ellipse 58% 70% at 50% 40%, transparent 24%, #000 78%);
  -webkit-mask-composite: source-in; mask-composite: intersect;
}

/* ---------------------------------------------------------------- 9 arcs: rings that rise from below, x.ai's footer */
/* the rings need room under the words: a minimum height (the words centred), not more padding, so the gaps stay the page's */
.section.bg-arcs { min-height: clamp(420px, 36vw, 540px); display: grid; align-content: center; }
.bg-arcs::before {
  background:
    radial-gradient(56% 56% at 50% 100%, color-mix(in oklab, var(--g-hue) calc(22% * var(--g-k)), transparent), transparent),
    radial-gradient(30% 40% at 78% 100%, color-mix(in oklab, var(--g-hue-t) calc(24% * var(--g-k)), transparent), transparent),
    repeating-radial-gradient(circle at 50% 140%, transparent 0 79px, color-mix(in oklab, var(--ink) calc(22% * var(--g-kl)), transparent) 79px 80px);
  -webkit-mask-image: linear-gradient(to bottom, transparent, #000 var(--g-fade), #000 calc(100% - var(--g-fade)), transparent), radial-gradient(ellipse 84% 94% at 50% 100%, #000 30%, transparent 82%);
          mask-image: linear-gradient(to bottom, transparent, #000 var(--g-fade), #000 calc(100% - var(--g-fade)), transparent), radial-gradient(ellipse 84% 94% at 50% 100%, #000 30%, transparent 82%);
  -webkit-mask-composite: source-in; mask-composite: intersect;
}

/* ---------------------------------------------------------------- 10 band: a coloured band in a second tone */
/* the lit ground's tone with a light corner, a richer neighbour of the same colour leaning in from the right and a
   deeper shade of itself in the far corner, under fine grain: one family of colour, never flat */
.bg-band {
  background-image:
    var(--g-noise-soft),
    radial-gradient(66% 120% at 0% 0%, color-mix(in oklab, var(--bg) 42%, var(--g-hi, white)), transparent 72%),
    radial-gradient(52% 96% at 100% 0%, color-mix(in oklab, var(--t-b) 62%, transparent), transparent 76%),
    radial-gradient(64% 104% at 100% 100%, color-mix(in oklab, var(--t-c) 70%, transparent), transparent 78%);
  background-blend-mode: soft-light, normal, normal, normal;
}

/* .bg-tint, with .bg-band or .bg-field: the second tone is a pale tint of the accent's own colour (a mint for a teal accent,
   an azure for a blue one, Figma's flat mint panel) instead of the lit ground, so the band is coloured on a sand page too */
.bg-tint {
  --tone: oklch(from var(--g-hue) 0.93 clamp(0.035, c * 0.32, 0.075) h);
  --tone-ink: color-mix(in oklab, var(--g-light-ink) 58%, black);
  --accent: var(--g-light-accent); --accent-ink: var(--g-light-accent-ink);
  --surface: color-mix(in oklab, var(--tone) 60%, white);
  --card: color-mix(in oklab, var(--g-light-bg) 80%, white);
}
/* on a dark page a pale tinted slab is the brightest thing there: the tint is a deep tone of the accent's colour with light words */
html[data-mode="dark"] .bg-tint {
  --tone: oklch(from var(--g-hue) 0.24 clamp(0.04, c * 0.42, 0.09) h);
  --tone-ink: color-mix(in oklab, var(--p-ink) 92%, white);
  --muted: color-mix(in oklab, var(--tone-ink) 82%, var(--tone));
  --accent: var(--p-accent); --accent-ink: var(--p-accent-ink);
  --surface: color-mix(in oklab, var(--tone) 80%, white);
  --card: color-mix(in oklab, var(--tone) 86%, black);
  --action: var(--tone-ink); --action-ink: var(--tone);
  --g-hi: color-mix(in oklab, var(--tone) 88%, white);   /* the light corner of the band and the field stays deep, under light words */
}

/* ---------------------------------------------------------------- 11 field: a grainy gradient in the lit tone, Figma's panel */
.bg-field {
  background-image:
    var(--g-noise-soft),
    linear-gradient(112deg, transparent 38%, color-mix(in oklab, var(--g-hi, white) 26%, transparent) 50%, transparent 62%),
    radial-gradient(58% 86% at 8% 0%, color-mix(in oklab, var(--g-hi, white) 58%, transparent), transparent),
    radial-gradient(54% 80% at 82% 6%, color-mix(in oklab, var(--t-b) 80%, transparent), transparent),
    radial-gradient(60% 96% at 100% 100%, color-mix(in oklab, var(--t-c) 76%, transparent), transparent),
    radial-gradient(40% 60% at 0% 100%, color-mix(in oklab, var(--t-b) 50%, transparent), transparent);
  background-blend-mode: soft-light, normal, normal, normal, normal, normal;
}

/* ---------------------------------------------------------------- 12 stage: a vivid panel for a visual, Figma's tiles */
/* a rounded panel (not a section) behind one card or window: the accent's colour as a mesh with a light corner and a
   deep one, and grain. It has NO text of its own: every word sits on a card on it (that is what lets it be vivid).
     <div class="bg-stage"><div class="mini">...</div></div>
   A grey accent (data-lowchroma) takes the lit ground's colour. --stage-pad is the room round the card. */
.bg-stage {
  --st-src: var(--g-hue);
  /* yellows go olive at mid lightness, so around hue 90 the mid and deep tones are lifted (+.12, +.1) */
  --st-1: oklch(from var(--st-src) calc(0.64 + 0.12 * clamp(0, 1 - max(h - 90, 90 - h) / 45, 1)) clamp(0.07, c * 0.8, 0.14) calc(h - 8 * clamp(0, 1 - max(h - 90, 90 - h) / 45, 1)));
  --st-2: oklch(from var(--st-src) 0.8 clamp(0.05, c * 0.6, 0.1) calc(h + var(--g-hs)));
  --st-3: oklch(from var(--st-src) 0.95 0.035 h);
  /* a yellow darkened turns olive: its deep tone leans to amber instead (hue -24 at yellow) */
  --st-4: oklch(from var(--st-src) calc(0.44 + 0.1 * clamp(0, 1 - max(h - 90, 90 - h) / 45, 1)) clamp(0.05, c * 0.7, 0.12) calc(h - 24 * clamp(0, 1 - max(h - 90, 90 - h) / 45, 1)));
  --stage-pad: clamp(20px, 4.5vw, 64px);
  display: grid; place-items: center; padding: var(--stage-pad); border-radius: var(--r-lg);
  background-color: var(--st-1);
  background-image:
    var(--g-noise-soft),
    radial-gradient(58% 78% at 10% 4%, color-mix(in oklab, var(--st-3) 90%, transparent), transparent 72%),
    radial-gradient(48% 70% at 90% 12%, color-mix(in oklab, var(--st-2) 92%, transparent), transparent 74%),
    radial-gradient(70% 90% at 78% 112%, color-mix(in oklab, var(--st-4) 94%, transparent), transparent 74%),
    radial-gradient(46% 62% at 4% 100%, color-mix(in oklab, var(--st-2) 60%, transparent), transparent 72%);
  background-blend-mode: soft-light, normal, normal, normal, normal;
}
html[data-tint="away"] .bg-stage { --st-src: var(--g-acc); }
html[data-lowchroma] .bg-stage { --st-src: oklch(from var(--ground-accent) l c h); }
.bg-stage > * { position: relative; width: min(100%, var(--stage-w, 520px)); }

/* ---------------------------------------------------------------- 13 photo: a full-bleed photograph under a calm scrim */
/* The photograph is an <img class="bg-photo__img" src alt> child (alt text, lazy loading, found early by the
   browser), or --g-photo: url(...) when the section cannot hold one. The scrim is the dark side's tone and is at
   least .76 deep from the bottom edge up to --scrim-h (the top of the words; backgrounds.js measures it from the
   section's own text and sets it, 66% by default, 86% on a phone), so white words keep 7:1 even on a white
   photograph (a scrim of .76 over white is darker than #595959). A pale photograph takes .bg-photo--light: the scrim is
   the light side's tone and the words are dark. The scrim is the section's ::before, above the photograph.
   --g-photo-pos: where the subject sits (object-position, e.g. "70% 30%"); --g-photo-pos-m the same on a phone, where the
   words take the lower part and the subject belongs in the upper one. */
.bg-photo {
  --scrim: color-mix(in oklab, var(--g-dark-bg) 70%, black);
  /* the scrim deepens as the palette's dark side loses headroom (--g-kd): .88 / .80 / .76 at 1, solid at 0 */
  --scrim-a: color-mix(in oklab, var(--scrim) calc(100% - 12% * var(--g-kd)), transparent);
  --scrim-b: color-mix(in oklab, var(--scrim) calc(100% - 20% * var(--g-kd)), transparent);
  --scrim-c: color-mix(in oklab, var(--scrim) calc(100% - 24% * var(--g-kd)), transparent);
  --scrim-h: 66%; --scrim-w: 56%;
  min-height: clamp(520px, 52vw, 760px); display: grid; align-content: end;
  background: var(--g-photo, none) var(--g-photo-pos, center) / cover no-repeat, var(--scrim);
}
.bg-photo__img { position: absolute; inset: 0; z-index: -2; width: 100%; height: 100%; object-fit: cover; object-position: var(--g-photo-pos, 50% 50%); }
.bg-photo::before {
  content: ""; position: absolute; inset: 0; z-index: -1; pointer-events: none;
  background: linear-gradient(to top, var(--scrim-a), var(--scrim-b) calc(var(--scrim-h) * .45), var(--scrim-c) var(--scrim-h), transparent calc(var(--scrim-h) + 26%));
}
.bg-photo--left { align-content: center; }
.bg-photo--left::before { background: linear-gradient(to right, var(--scrim-a), var(--scrim-b) calc(var(--scrim-w) * .64), var(--scrim-c) var(--scrim-w), transparent calc(var(--scrim-w) + 26%)); }
.bg-photo--light {
  --scrim: var(--g-light-bg);
  --scrim-a: color-mix(in oklab, var(--scrim) 94%, transparent);
  --scrim-b: color-mix(in oklab, var(--scrim) 88%, transparent);
  --scrim-c: color-mix(in oklab, var(--scrim) 76%, transparent);
  --bg: color-mix(in oklab, var(--g-light-bg) 90%, white); --surface: var(--p-surface); --ink: var(--g-light-ink);
  --muted: color-mix(in oklab, var(--g-light-ink) 60%, var(--g-light-muted));
  --accent: var(--g-light-accent); --accent-ink: var(--g-light-accent-ink);
  --action: var(--g-light-action); --action-ink: var(--g-light-action-ink);
  --card: var(--g-light-bg);
}
@media (max-width: 767px) {
  .bg-photo { --scrim-h: 86%; min-height: 640px; background-position: var(--g-photo-pos-m, var(--g-photo-pos, center)); }
  .bg-photo__img { object-position: var(--g-photo-pos-m, var(--g-photo-pos, 50% 50%)); }
  .bg-photo--left { align-content: end; }
  .bg-photo--left::before { background: linear-gradient(to top, var(--scrim-a), var(--scrim-b) calc(var(--scrim-h) * .45), var(--scrim-c) var(--scrim-h), transparent calc(var(--scrim-h) + 12%)); }
}

/* ---------------------------------------------------------------- 14 horizon: a dark band with a lit horizon */
/* a dome of the page's dark tone rising from the bottom edge, its rim a thin line of light that is brightest at the
   top and fades to both sides, with the glow above it (the planet's edge in x.ai's dark pages) */
.section.bg-horizon:not(.section--flush-top, .section--rule) { padding-bottom: calc(var(--section-band) * 1.25 + 150px); }   /* (0,3,0), after the spacing block: keeps the dome's room, hero or not */
.bg-horizon::before {
  content: ""; position: absolute; z-index: -1; left: 0; right: 0; bottom: 0; height: 640px; pointer-events: none;
  background:
    radial-gradient(26% 74px at 50% calc(100% - 140px), color-mix(in oklab, var(--lit) calc(70% * var(--g-kd)), transparent), transparent),
    radial-gradient(54% 76% at 50% 100%, color-mix(in oklab, var(--lit) calc(44% * var(--g-kd)), transparent), transparent 74%),
    radial-gradient(120% 84% at 50% 100%, color-mix(in oklab, var(--g-hue) calc(22% * var(--g-kd)), transparent), transparent 76%);
}
.bg-horizon::after {
  /* a dome as wide as 170% of the section and never less than 1200px, so its curve stays a flat planet's rim on a phone (a
     dome as wide as the phone is a pointed arch) */
  content: ""; position: absolute; z-index: -1; left: 50%; translate: -50% 0; width: max(170%, 1200px); bottom: -560px; height: 700px; pointer-events: none;
  border-radius: 50% 50% 0 0 / 100% 100% 0 0;
  background: radial-gradient(60% 100% at 50% 0, color-mix(in oklab, var(--lit) 34%, var(--bg)), var(--bg) 72%);
  border-top: 2px solid color-mix(in oklab, var(--lit) 90%, white);
  box-shadow: 0 -12px 56px -8px color-mix(in oklab, var(--lit) 62%, transparent), 0 -1px 16px color-mix(in oklab, var(--lit) 70%, transparent);
  -webkit-mask-image: linear-gradient(90deg, transparent 5%, #000 36%, #000 64%, transparent 95%);
          mask-image: linear-gradient(90deg, transparent 5%, #000 36%, #000 64%, transparent 95%);
}

/* ---------------------------------------------------------------- 15 deep: near-black with a spotlight and grain, x.ai's dark */
.bg-deep {
  --bg: color-mix(in oklab, var(--g-dark-bg) 70%, black);
  background-image:
    radial-gradient(64% 70% at 50% 0%, color-mix(in oklab, var(--g-hue) calc(21% * var(--g-kd)), transparent), transparent 76%),
    radial-gradient(48% 60% at 100% 100%, color-mix(in oklab, var(--g-hue-t) calc(18% * var(--g-kd)), transparent), transparent 72%),
    linear-gradient(180deg, color-mix(in oklab, var(--g-dark-surface) calc(40% * var(--g-kd)), var(--bg)), var(--bg) 70%),
    linear-gradient(var(--g-cover-d), var(--g-cover-d)),
    var(--g-noise-scr);
  background-blend-mode: normal, normal, normal, normal, screen;
}
/* a hairline of light along the top edge, brightest in the middle (a lit lip, as on x.ai's cards) */
.bg-deep::before {
  content: ""; position: absolute; z-index: -1; left: 0; right: 0; top: 0; height: 1px; pointer-events: none;
  background: linear-gradient(90deg, transparent 6%, color-mix(in oklab, var(--ink) calc(28% * var(--g-kd)), transparent) 50%, transparent 94%);
}

/* ---------------------------------------------------------------- 16 liquid: a slow mesh of colour, transform only */
/* two layers, each wider than the section (160%), each holding two wide ellipses of colour that fade to nothing inside the
   layer's own height (their radii end before the section's top and bottom, so nothing is clipped into an edge). They drift,
   skew and stretch against each other (translate, skewX, scaleY: transforms only, nothing repaints), so the mesh keeps
   changing shape. Three hues: the accent, the lit ground's tone (a second hue, so the wash is not blue all through) and
   the accent's neighbour, all low in chroma. Held still, it is the same picture. */
.bg-liquid::before, .bg-liquid::after {
  content: ""; position: absolute; z-index: -1; pointer-events: none; top: 0; bottom: 0; left: -30%; right: -30%; will-change: transform;
}
.bg-liquid::before {
  background:
    radial-gradient(ellipse 30% 36% at 30% 44%, color-mix(in oklab, var(--g-hue) calc(40% * var(--g-k)), transparent), transparent),
    radial-gradient(ellipse 22% 24% at 54% 74%, color-mix(in oklab, var(--g-hue-t) calc(44% * var(--g-k)), transparent), transparent);
  animation: g-liq-a 26s ease-in-out infinite alternate;
}
.bg-liquid::after {
  background:
    radial-gradient(ellipse 28% 30% at 68% 38%, color-mix(in oklab, var(--g-hue-b) calc(32% * var(--g-k)), transparent), transparent),
    radial-gradient(ellipse 20% 20% at 44% 26%, color-mix(in oklab, var(--g-hue-t) calc(36% * var(--g-k)), transparent), transparent);
  animation: g-liq-b 34s ease-in-out infinite alternate;
}
@keyframes g-liq-a { from { transform: translate3d(-7%, 0, 0) skewX(-7deg) scaleY(.94); } to { transform: translate3d(8%, 0, 0) skewX(6deg) scaleY(1.05); } }
@keyframes g-liq-b { from { transform: translate3d(8%, 0, 0) skewX(6deg) scaleY(1.04); } to { transform: translate3d(-9%, 0, 0) skewX(-6deg) scaleY(.92); } }
.bg-liquid.is-off::before, .bg-liquid.is-off::after, .bg-liquid.is-held::before, .bg-liquid.is-held::after { animation-play-state: paused; }
/* the pause disc sits on the container's right edge, not the section's corner */
.bg-liquid > .loop-pause { right: max(var(--pad), calc((100% - var(--container)) / 2)); bottom: 16px; }

/* ---------------------------------------------------------------- motion and print */
@media (prefers-reduced-motion: reduce) {
  .bg-liquid::before, .bg-liquid::after { animation: none; }
}
@media print { .bg-liquid::before, .bg-liquid::after { display: none; } }
```

## assets/backgrounds/backgrounds.js

```js
/*
  taste-skill backgrounds.js - optional, about 2 KB of work. Load it plain (not deferred) at the end of <head>,
  right after the :root palette block, so a dark page never paints one frame with the light page's grain.
  It reads the page palette (--p-*) and sets, on <html>, what backgrounds.css cannot know in CSS:
    data-mode  light | dark    is the page ink lighter than the page bg?
    data-tint  away            (only when) the bg is a strong colour or a mid tone: tints move away from the
                               ink, which only ever adds contrast
    data-lowchroma             (only when) the accent is a grey or near-grey: .bg-stage then takes the lit
                               ground's colour instead of a hue that does not exist
    --g-k      0.08 to 1       how strongly the tints may show on this palette: the contrast headroom of the
                               ink (7:1) and muted (4.5:1) text, in lightness, over the depth the grounds are
                               drawn at. A palette whose ink is only just 7:1 on its bg gets almost flat grounds
                               (an away palette keeps 1: its tints only add contrast)
    --g-kl     the same headroom, kept for the grounds that draw in the ink's own direction (the rules, dots, rings)
    --g-kp     the same, counted on the raw muted text, for the two page grounds (paper, topglow), which set no tokens
    --g-kd     the same for the dark grounds (horizon, deep, photo)
  An attribute or variable the page (or start.mjs) already set is kept (--g-k, --g-kl, --g-kd each on its own: inline on
  <html> or in a stylesheet). It also pauses the .bg-liquid blobs while their section is off screen, and measures each
  .bg-photo's words to set how high its scrim stays deep (--scrim-h, or --scrim-w for .bg-photo--left on a wide screen).
  Every other ground is plain CSS.
*/
(() => {
  const root = document.documentElement;
  const get = n => getComputedStyle(root).getPropertyValue(n).trim();
  const hex = n => {
    let v = get(n).replace('#', '');
    if (v.length === 3) v = [...v].map(c => c + c).join('');
    return /^[0-9a-f]{6}$/i.test(v) ? [0, 2, 4].map(i => parseInt(v.slice(i, i + 2), 16) / 255) : null;
  };
  const lin = c => (c <= 0.04045 ? c / 12.92 : ((c + 0.055) / 1.055) ** 2.4);
  const Y = ([r, g, b]) => 0.2126 * lin(r) + 0.7152 * lin(g) + 0.0722 * lin(b);
  const oklab = rgb => {
    const [r, g, b] = rgb.map(lin);
    const l = Math.cbrt(0.4122214708 * r + 0.5363325363 * g + 0.0514459929 * b);
    const m = Math.cbrt(0.2119034982 * r + 0.6806995451 * g + 0.1073969566 * b);
    const s = Math.cbrt(0.0883024619 * r + 0.2817188376 * g + 0.6299787005 * b);
    return [0.2104542553 * l + 0.793617785 * m - 0.0040720468 * s, 1.9779984951 * l - 2.428592205 * m + 0.4505937099 * s, 0.0259040371 * l + 0.7827717662 * m - 0.808675766 * s];
  };
  const REF = 0.22;  // the deepest a tint is drawn at k = 1, in oklab lightness
  /* how far (oklab L) a ground of colour bg may move toward the ink and keep ink 7:1 and the muted
     text a ground uses (60% ink, 40% palette muted) 4.5:1 */
  const room = (ink, muted, bg, bgScale = 1, mixInk = 0.6) => {
    const Li = oklab(ink)[0], Lm = mixInk * Li + (1 - mixInk) * oklab(muted)[0], Lb = oklab(bg)[0] * bgScale, Yi = Y(ink), Ym = Lm ** 3;
    if (Li > Lb) {
      const max = Math.min((Yi + 0.05) / 7 - 0.05, (Ym + 0.05) / 4.5 - 0.05) * 0.8;
      return max > 0 ? Math.cbrt(max) - Lb : 0;
    }
    const min = Math.max(7 * (Yi + 0.05) - 0.05, 4.5 * (Ym + 0.05) - 0.05) * 1.15;
    return min < 1 ? Lb - Math.cbrt(min) : 0;
  };
  const clamp = (v, a, b) => Math.min(b, Math.max(a, v));

  const detect = force => {
    const bg = hex('--p-bg'), ink = hex('--p-ink'), muted = hex('--p-muted');
    if (!bg || !ink) return;
    const a = oklab(bg), b = oklab(ink);
    if (force || !root.dataset.mode) root.dataset.mode = b[0] > a[0] ? 'dark' : 'light';
    if (force) { root.removeAttribute('data-tint'); ['--g-k', '--g-kl', '--g-kd', '--g-kp'].forEach(p => root.style.removeProperty(p)); }
    const ac = hex('--p-accent');
    if (ac) root.toggleAttribute('data-lowchroma', Math.hypot(oklab(ac)[1], oklab(ac)[2]) < 0.045);
    const away = Math.hypot(a[1], a[2]) > 0.09 || (a[0] > 0.3 && a[0] < 0.84);
    if (away && !root.hasAttribute('data-tint')) root.setAttribute('data-tint', 'away');
    const m = muted || ink;
    /* a strength the page already set (inline on <html>, or in a stylesheet: anything but the CSS default of 1) is kept, each on its own */
    const mine = n => force || (!root.dataset.k && root.style.getPropertyValue(n).trim() === '' && ['', '1'].includes(get(n)));
    const kl = clamp(room(ink, m, bg) / REF, 0.08, 1);
    if (mine('--g-kl')) root.style.setProperty('--g-kl', kl.toFixed(3));
    if (mine('--g-k')) root.style.setProperty('--g-k', root.dataset.tint === 'away' ? '1' : kl.toFixed(3));
    /* the page grounds (paper, topglow) sit under the page's own muted text, which they do not re-mix: their headroom is counted on the palette's raw muted */
    if (mine('--g-kp')) root.style.setProperty('--g-kp', root.dataset.tint === 'away' ? '1' : clamp(room(ink, m, bg, 1, 0) / REF, 0.08, 1).toFixed(3));
    const db = hex('--g-dark-bg'), di = hex('--g-dark-ink'), dm = hex('--g-dark-muted');
    if (db && di && dm && mine('--g-kd')) root.style.setProperty('--g-kd', clamp(room(di, dm, db, 0.8) / REF, 0.08, 1).toFixed(3));
  };
  detect(false);
  window.tasteBackgrounds = { detect: () => detect(true) };

  /* the pause disc (WCAG 2.2.2: it moves for more than 5 s): the system's own .loop-pause, same icons, in the section's
     bottom-right corner; it holds the blobs with .is-held. Nothing under prefers-reduced-motion, where they hold still. */
  const ICON_PAUSE = '<svg viewBox="0 0 16 16" aria-hidden="true"><rect x="4.25" y="3.25" width="2.75" height="9.5" rx="0.9"/><rect x="9" y="3.25" width="2.75" height="9.5" rx="0.9"/></svg>';
  const ICON_PLAY = '<svg viewBox="0 0 16 16" aria-hidden="true"><path d="M5.25 3.7v8.6c0 .62.68 1 1.2.66l6.5-4.3c.47-.31.47-1 0-1.32l-6.5-4.3c-.52-.34-1.2.04-1.2.66Z"/></svg>';
  const hold = el => {
    if (el.hasAttribute('data-no-pause') || matchMedia('(prefers-reduced-motion: reduce)').matches || el.querySelector(':scope > .loop-pause')) return;
    const b = document.createElement('button');
    b.type = 'button'; b.className = 'loop-pause';
    const show = () => { const held = el.classList.contains('is-held'), label = (held ? 'Play' : 'Pause') + ' animation'; b.innerHTML = held ? ICON_PLAY : ICON_PAUSE; b.setAttribute('aria-label', label); b.title = label; };
    b.addEventListener('click', () => { el.classList.toggle('is-held'); show(); });
    show(); el.append(b);
  };
  const watch = () => {
    const live = document.querySelectorAll('.bg-liquid');
    live.forEach(hold);
    if (live.length && 'IntersectionObserver' in window) {
      const io = new IntersectionObserver(es => es.forEach(e => e.target.classList.toggle('is-off', !e.isIntersecting)), { rootMargin: '80px' });
      live.forEach(el => { el.classList.add('is-off'); io.observe(el); });
    }
  };
  /* the scrim of a photograph must stay deep under every word: measure where the words start (their top edge, as a share of the
     section's height, from the bottom; their right edge for .bg-photo--left on a wide screen) and keep the deep part of the
     scrim 8% past it. A value the section sets itself (inline) is kept. */
  const fit = el => {
    const r = el.getBoundingClientRect();
    if (!r.height || !r.width) return;
    const words = [...el.querySelectorAll('h1, h2, h3, h4, p, a, button, li, blockquote, figcaption, .btn-row')].filter(w => !w.closest('.bg-photo__img') && w.getClientRects().length);
    if (!words.length) return;
    const top = Math.min(...words.map(w => w.getBoundingClientRect().top)), right = Math.max(...words.map(w => w.getBoundingClientRect().right));
    if (!el.dataset.scrimHSet) {
      const h = Math.min(100, Math.max(40, ((r.bottom - top) / r.height) * 100 + 8));
      el.style.setProperty('--scrim-h', h.toFixed(1) + '%');
    }
    if (!el.dataset.scrimWSet) el.style.setProperty('--scrim-w', Math.min(100, Math.max(30, ((right - r.left) / r.width) * 100 + 8)).toFixed(1) + '%');
  };
  const photos = () => {
    document.querySelectorAll('.bg-photo').forEach(el => {
      /* a page-set inline value is kept: remember it before the first measure */
      if (el.dataset.scrimHSet == null) el.dataset.scrimHSet = el.style.getPropertyValue('--scrim-h') ? '1' : '';
      if (el.dataset.scrimWSet == null) el.dataset.scrimWSet = el.style.getPropertyValue('--scrim-w') ? '1' : '';
      fit(el);
      if ('ResizeObserver' in window && !el.__fit) { el.__fit = new ResizeObserver(() => fit(el)); el.__fit.observe(el); }
    });
  };
  if (document.readyState === 'loading') document.addEventListener('DOMContentLoaded', () => { watch(); photos(); }, { once: true }); else { watch(); photos(); }
  if (document.fonts && document.fonts.ready) document.fonts.ready.then(photos);
})();
```

## assets/icons/icon.css

```css
/*
  taste-skill icons/icon.css - how a real icon sits on the page: bare, in a tile, in a list row, in a chip,
  in an avatar stack, in an integration grid, on a file card. Link it after system.css. It reads the
  ground tokens (--bg --surface --card --ink --muted --accent --line) and the radius set, so the same
  markup is right on a light page, on .ground-surface and on .ground-inverse. Nothing here animates.

  Markup comes from `node scripts/icons.mjs fetch ... --out assets/icons`: a colour icon is an
  <img class="ic ic-24">, a glyph (ui:) or any --mono icon is an inline <svg class="ic ic-svg ic-24"> that
  takes the text colour (works from file:// too; --mask prints a <span class="ic-mask"> instead, which needs
  http). Sizes: ic-16 ic-20 ic-24 ic-28 ic-32 ic-36 ic-40 ic-48 ic-56.
*/

/* the only colours outside the ground tokens: a white plate for logos, and the connected dot */
:root { --ic-white: #fff; --ic-ok: var(--ok, #2f9e5b); }

/* ------------------------------------------------------------ the icon itself */
.ic { --ic-s: 24px; display: inline-block; flex: none; height: var(--ic-s); width: auto; max-width: none; vertical-align: middle; object-fit: contain; }
.ic-16 { --ic-s: 16px; } .ic-20 { --ic-s: 20px; } .ic-24 { --ic-s: 24px; } .ic-28 { --ic-s: 28px; } .ic-32 { --ic-s: 32px; }
.ic-36 { --ic-s: 36px; } .ic-40 { --ic-s: 40px; } .ic-48 { --ic-s: 48px; } .ic-56 { --ic-s: 56px; }
.ic-svg { width: var(--ic-s); }
.ic-mask { width: var(--ic-s); background-color: currentColor; -webkit-mask-size: contain; mask-size: contain; -webkit-mask-repeat: no-repeat; mask-repeat: no-repeat; -webkit-mask-position: center; mask-position: center; }
.ic-muted { color: var(--muted); }
.ic-accent { color: var(--accent); }

/* a wordmark (aspect over 1.8, flagged ic-wide by the tool) is drawn smaller than a square mark of the same
   size, so a row of both has an even weight; inside a tile it is fitted by the tile rule below */
.ic-wide { height: calc(var(--ic-s) * 0.7); max-width: min(100%, calc(var(--ic-s) * 4.5)); }
.ic-svg.ic-wide { width: auto; }

/* One-tone marks. ic-ink is a black mark (GitHub, Apple, X): white on a dark ground. ic-paper is a white mark
   (Notion): it would vanish on a light ground, so it is shown black there, bare or in a light tile. A tile with
   a dark or brand fill keeps a white mark as drawn and turns a black one white (.ic-tile--ink, --brand). */
.ic-paper { filter: invert(1); }
:is(.ground-inverse, .ic-dark) .ic-ink { filter: invert(1); }
:is(.ground-inverse, .ic-dark) .ic-paper { filter: none; }
.ic-keep { filter: none !important; }

/* ------------------------------------------------------------ the tile behind a logo */
.ic-tile {
  --tile: 40px;
  --tile-bg: color-mix(in oklab, var(--ic-white) 72%, var(--bg));
  --tile-line: var(--line);
  display: inline-grid; place-items: center; flex: none;
  width: var(--tile); height: var(--tile); border-radius: calc(var(--tile) * 0.28);
  background: var(--tile-bg); color: var(--ink);
  box-shadow: inset 0 0 0 1px var(--tile-line), 0 1px 2px color-mix(in oklab, var(--ink) 7%, transparent);
}
.ic-tile > .ic { --ic-s: calc(var(--tile) * 0.6); }
/* a wordmark or an odd-shaped logo is fitted into the square box, never allowed to spill out of its tile */
.ic-tile > img.ic, .ic-file > img.ic, .ic-point > img.ic { width: var(--ic-s); object-fit: contain; }
/* a landscape mark (Salesforce, AWS, YouTube: ic-land) and a portrait one (Google Docs, Figma: ic-tall) are drawn on their long
   side a little larger than a square mark, so a row of marks has an even weight and none reads small in its tile */
.ic-tile > img.ic.ic-land, .ic-tile > .ic-svg.ic-land { width: calc(var(--tile) * 0.7); height: auto; max-height: 80%; }
.ic-tile > img.ic.ic-tall, .ic-tile > .ic-svg.ic-tall { height: calc(var(--tile) * 0.68); width: auto; max-width: 80%; }
.ic-tile--24 { --tile: 24px; } .ic-tile--32 { --tile: 32px; } .ic-tile--40 { --tile: 40px; }
.ic-tile--48 { --tile: 48px; } .ic-tile--56 { --tile: 56px; } .ic-tile--72 { --tile: 72px; }
.ic-tile--round { border-radius: 50%; }
/* a wordmark-only logo (Stripe, Visa): a wide tile, the mark enlarged to fill it (about 1.7 tiles wide, never
   taller than 58% of the tile), at its own proportions */
.ic-tile--wide { width: calc(var(--tile) * 2.4); }
.ic-tile.ic-tile--wide > .ic { width: calc(var(--tile) * 1.7); height: auto; max-width: 80%; max-height: 58%; object-fit: contain; }
.ic-tile--flat { box-shadow: inset 0 0 0 1px var(--tile-line); }
/* a glyph in a tile: the surface colour, ink strokes, or the accent */
.ic-tile--glyph { --tile-bg: color-mix(in oklab, var(--ink) 5%, var(--bg)); box-shadow: none; }
.ic-tile--accent { --tile-bg: color-mix(in oklab, var(--accent) 12%, var(--bg)); color: var(--accent); box-shadow: none; }
.ic-tile--ink { --tile-bg: var(--ink); color: var(--bg); box-shadow: none; }
/* on a dark ground --ink is light, so an ink tile is a dark plate (the ground lifted a little) instead (its black marks are
   inverted to white either way) */
:is(.ground-inverse, .ic-dark) .ic-tile--ink { --tile-bg: color-mix(in oklab, var(--ink) 10%, var(--bg)); color: var(--ink); box-shadow: inset 0 0 0 1px var(--line); }
/* a tile in a brand colour (set --tile-bg inline): white marks stay white, black marks turn white */
.ic-tile--brand { --tile-bg: var(--accent); color: var(--ic-white); box-shadow: none; }
:is(.ic-tile--ink, .ic-tile--brand) .ic-paper { filter: none; }
:is(.ic-tile--ink, .ic-tile--brand) .ic-ink { filter: invert(1); }
/* dark grounds: a light tile, so every brand colour reads and black marks need no inversion */
:is(.ground-inverse, .ic-dark) .ic-tile:not(.ic-tile--glyph, .ic-tile--accent, .ic-tile--ink) {
  --tile-bg: color-mix(in oklab, var(--ink) 94%, var(--bg)); --tile-line: transparent; color: var(--bg);
  box-shadow: 0 1px 2px color-mix(in oklab, var(--bg) 70%, transparent);
}
:is(.ground-inverse, .ic-dark) .ic-tile:not(.ic-tile--ink, .ic-tile--brand) .ic-ink { filter: none; }
:is(.ground-inverse, .ic-dark) .ic-tile:not(.ic-tile--ink, .ic-tile--brand) .ic-paper { filter: invert(1); }
:is(.ground-inverse, .ic-dark) .ic-tile--glyph { --tile-bg: color-mix(in oklab, var(--ink) 9%, var(--bg)); }
:is(.ground-inverse, .ic-dark) .ic-tile--accent { --tile-bg: color-mix(in oklab, var(--accent) 22%, var(--bg)); }
/* a tile of a dark-ground brand, glass: for one-colour marks only */
.ic-tile--ghost { --tile-bg: color-mix(in oklab, var(--ink) 6%, transparent); box-shadow: inset 0 0 0 1px var(--line); }

/* ------------------------------------------------------------ chip: icon and a name */
.ic-chip {
  display: inline-flex; align-items: center; gap: 8px; height: 32px; padding: 0 12px 0 8px;
  border-radius: 999px; background: var(--card, var(--surface)); color: var(--ink);
  box-shadow: inset 0 0 0 1px var(--line); font-size: var(--t-small, 14px); line-height: 1; white-space: nowrap;
}
.ic-chip > .ic { --ic-s: 18px; }
.ic-chip--lg { height: 40px; padding: 0 16px 0 10px; gap: 10px; font-size: var(--t-body, 16px); }
.ic-chip--lg > .ic { --ic-s: 22px; }
.ic-chip > .ic-wide { height: auto; width: calc(var(--ic-s) * 2.2); max-height: calc(var(--ic-s) * 0.9); }
/* on a dark ground a brand colour (Salesforce blue, Dropbox, Jira) or a dark file icon (InDesign) sits close to the
   ground: a bare mark in a chip or on a file card gets a small light plate, as a tile does */
:is(.ground-inverse, .ic-dark) :is(.ic-chip, .ic-file) > img.ic { box-sizing: content-box; padding: 3px; border-radius: 7px; background: color-mix(in oklab, var(--ink) 94%, var(--bg)); }
:is(.ground-inverse, .ic-dark) .ic-file > img.ic { padding: 4px; border-radius: 10px; }
:is(.ground-inverse, .ic-dark) :is(.ic-chip, .ic-file) > img.ic-ink { filter: none; }
:is(.ground-inverse, .ic-dark) :is(.ic-chip, .ic-file) > img.ic-paper { filter: invert(1); }
.ic-chip-row { display: flex; flex-wrap: wrap; gap: 8px; }
a.ic-chip:hover { box-shadow: inset 0 0 0 1px var(--line-strong); }

/* ------------------------------------------------------------ list row: tile, text, a value at the end */
.ic-list { display: grid; }
.ic-row { display: flex; align-items: center; gap: 14px; padding: 12px 0; border-top: 1px solid var(--line); min-width: 0; }
.ic-row:first-child { border-top: 0; }
.ic-row__main { display: grid; gap: 2px; min-width: 0; flex: 1; }
.ic-row__title { font-size: var(--t-body, 16px); font-weight: 500; line-height: 1.3; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.ic-row__meta { font-size: var(--t-small, 14px); color: var(--muted); line-height: 1.3; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.ic-row__end { flex: none; font-size: var(--t-small, 14px); color: var(--muted); display: inline-flex; align-items: center; gap: 8px; }
.ic-row__end .ic { --ic-s: 16px; }
/* a connected / off state as a dot with its word */
.ic-state { display: inline-flex; align-items: center; gap: 6px; }
.ic-state::before { content: ""; width: 6px; height: 6px; border-radius: 50%; background: var(--muted); }
.ic-state--on { color: var(--ink); }
.ic-state--on::before { background: var(--ic-ok); }

/* ------------------------------------------------------------ avatar stack: overlapping tiles */
.ic-stack { display: inline-flex; align-items: center; }
.ic-stack > * { margin-left: -10px; box-shadow: 0 0 0 2px var(--bg); }
.ic-stack > :first-child { margin-left: 0; }
.ic-stack > .ic-tile { box-shadow: inset 0 0 0 1px var(--tile-line), 0 0 0 2px var(--bg); }
.ic-stack__more {
  display: inline-grid; place-items: center; height: 40px; min-width: 40px; padding: 0 10px; border-radius: 999px;
  background: color-mix(in oklab, var(--ink) 8%, var(--bg)); color: var(--muted); font-size: var(--t-label, 13px); font-weight: 500;
}
.ic-stack--sm > .ic-tile { --tile: 32px; } .ic-stack--sm .ic-stack__more { height: 32px; min-width: 32px; }

/* ------------------------------------------------------------ integration grid */
.ic-grid { display: grid; gap: var(--card-gap, 16px); grid-template-columns: repeat(auto-fill, minmax(168px, 1fr)); }
.ic-cell {
  display: grid; align-content: start; gap: 14px; padding: 20px; border-radius: var(--r-md, 12px);
  background: var(--card, var(--surface)); box-shadow: inset 0 0 0 1px var(--line); min-width: 0;
}
.ic-cell__name { font-size: var(--t-body, 16px); font-weight: 500; line-height: 1.25; }
.ic-cell__meta { font-size: var(--t-small, 14px); color: var(--muted); line-height: 1.4; margin-top: 2px; }
a.ic-cell:hover { box-shadow: inset 0 0 0 1px var(--line-strong); }
/* the mosaic: one outline, cells divided by hairlines */
.ic-grid--mosaic { gap: 1px; background: var(--line); border-radius: var(--r-lg, 20px); overflow: hidden; border: 1px solid var(--line); }
.ic-grid--mosaic > .ic-cell { border-radius: 0; box-shadow: none; background: var(--bg); }
.ic-grid--mosaic > .ic-cell:hover { background: color-mix(in oklab, var(--ink) 3%, var(--bg)); }
/* a logo wall: tiles only, no names */
.ic-wall { display: grid; gap: 12px; grid-template-columns: repeat(auto-fill, minmax(64px, 1fr)); }
.ic-wall > .ic-tile { --tile: 64px; justify-self: start; }
.ic-wall > .ic-tile--wide { grid-column: span 2; }
.ic-grid--cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
@media (min-width: 640px) { .ic-grid--cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); } .ic-grid--cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); } }
@media (max-width: 639px) { .ic-grid--cols-3, .ic-grid--cols-4 { grid-template-columns: repeat(2, minmax(0, 1fr)); } }

/* ------------------------------------------------------------ file card */
.ic-file {
  display: flex; align-items: center; gap: 14px; padding: 14px 16px; min-width: 0;
  border-radius: var(--r-md, 12px); background: var(--card, var(--surface)); box-shadow: inset 0 0 0 1px var(--line);
}
.ic-file > .ic { --ic-s: 36px; }
/* a wordmark on a file card (a YAML, a Visa) keeps its own proportions at a readable width, never a strip in a 36px square */
.ic-file > img.ic.ic-wide, .ic-file > .ic-svg.ic-wide { width: calc(var(--ic-s) * 1.9); height: auto; max-height: var(--ic-s); }
.ic-file > img.ic.ic-land, .ic-file > .ic-svg.ic-land { width: calc(var(--ic-s) * 1.15); height: auto; }
.ic-file > img.ic.ic-tall, .ic-file > .ic-svg.ic-tall { width: auto; height: var(--ic-s); }
.ic-file__main { display: grid; gap: 2px; min-width: 0; flex: 1; }
.ic-file__name { font-size: var(--t-body, 16px); font-weight: 500; line-height: 1.3; overflow-wrap: anywhere; }
.ic-file__meta { font-size: var(--t-small, 14px); color: var(--muted); line-height: 1.3; overflow-wrap: anywhere; }
.ic-file__end { flex: none; color: var(--muted); font-size: var(--t-small, 14px); }
.ic-file-list { display: grid; gap: 10px; }
/* a file as a compact tile: icon on top, name under it */
.ic-file--tile { display: grid; justify-items: start; gap: 12px; padding: 18px; }
.ic-file--tile > .ic { --ic-s: 40px; }
.ic-file--tile > img.ic.ic-wide, .ic-file--tile > .ic-svg.ic-wide { width: calc(var(--ic-s) * 2.1); }
.ic-file--tile .ic-file__main { width: 100%; }

/* ------------------------------------------------------------ feature list with glyphs */
.ic-points { display: grid; gap: 16px; }
.ic-point { display: grid; grid-template-columns: auto 1fr; gap: 14px; align-items: start; }
.ic-point > .ic-tile { --tile: 36px; }
.ic-point > .ic-tile > .ic { --ic-s: 18px; }
.ic-point__title { font-weight: 500; line-height: 1.3; }
.ic-point__text { color: var(--muted); font-size: var(--t-small, 14px); line-height: 1.5; margin-top: 2px; }

/* ------------------------------------------------------------ a phone: names wrap, file tiles stack */
@media (max-width: 479px) {
  .ic-row__title, .ic-row__meta { white-space: normal; overflow: visible; text-overflow: clip; }
  .ic-grid--cols-2:has(> .ic-file--tile) { grid-template-columns: 1fr; }
  .ic-file--tile { grid-template-columns: auto 1fr; justify-items: start; align-items: center; }
}
```

## assets/layout/layout.css

```css
/*
  taste-skill layout.css - layout v4 (references/layout-v4.md): one section per screen, equal left and
  right edges, the ten hero types (heroes.json) and the display tokens a font pairing sets (fonts.json).
  Link it after system.css, ui/cards.css and backgrounds/backgrounds.css; load layout.js after system.js.

  Section   system.css sets the rhythm: every main > section exactly one or two screens (--h), its h2
            first, --padt under its top; this file adds the heroes and the display tokens.
  Hero      <section class="hero hero--<type>"><div class="wrap"><div class="hero-text ..">..</div></div>
            first in main: exactly the first screen, the fixed header over its top.
  Edges     one container; no one-sided bleed (a block stands on the lines or runs edge to edge).
  Type      start.mjs deals the pairing (VI-02) and prints --font-*, --display-weight (400; 400 to 500
            on tech) and --display-tracking; a page may add --display-stretch, --display-axes and
            --ly-scale (the face's optical size against the default grotesk, 1 when unsure).
  Place     a place's hero is never dealt: always hero--place (Recipe 3: the photograph under the
            header, the words under it on the ground). Nothing is laid over a photograph to make
            words readable, on any type: no scrim, gradient or veil (PH-31, VI-49).

  Every number here is in references/layout-v4.md with what it was measured on; change it there first.
*/

/* ---------------------------------------------------------------- tokens */
:root {
  --ly-nav: var(--nav, 72px);
  --ly-screen: calc(100svh - var(--ly-nav));      /* one screen under the fixed header: 828 at 1440x900, 788 at 390x844 */
  --ly-pad: var(--padt, 120px);
  --ly-hero-pad: var(--hgap, 48px);
  --ly-head-gap: var(--head-gap, 48px);            /* a section's heading block to its visual (system.css) */
  /* the content box of one screen (708 at 1440x900) and of the hero (716 = 900 - 72 - 48 - 64) */
  --ly-fit: calc(100svh - var(--padt, 120px) - var(--padb, 72px) - var(--bar, 0px));
  --ly-hero-h: calc(100svh - var(--nav, 72px) - var(--hgap, 48px) - var(--hpadb, 64px) - var(--bar, 0px));
  --ly-scale: 1;
  /* hero sizes: straight lines from 390 to 1440, times the pairing's --ly-scale (layout-v4.md, section 3) */
  --ly-h1:       calc(var(--ly-scale) * clamp(40px, 3.048vw + 28.11px, 72px));    /* 72 / 40: split, product, film, centred */
  --ly-h1-xl:    calc(var(--ly-scale) * clamp(44px, 3.429vw + 30.63px, 80px));    /* 80 / 44: photo */
  --ly-h1-col:   calc(var(--ly-scale) * clamp(40px, 2.286vw + 31.09px, 64px));    /* 64 / 40: collage */
  --ly-h1-mega:  calc(var(--ly-scale) * clamp(56px, 9.524vw + 18.86px, 156px));   /* 156 / 56: editorial */
  --ly-h1-word:  calc(var(--ly-scale) * clamp(54px, 8.571vw + 20.57px, 144px));   /* 144 / 54: word */
  --ly-h1-quiet: calc(var(--ly-scale) * clamp(34px, 2.095vw + 25.83px, 56px));    /* 56 / 34: statement */
  --ly-h1-side:  calc(var(--ly-scale) * clamp(40px, 1.143vw + 35.54px, 52px));    /* 52 / 40: triptych */
  --ly-lead: clamp(17px, .286vw + 15.89px, 20px);                                  /* 20 / 17 */
}

/* the pairing's display axes (fonts.json): width through font-stretch, the rest as settings */
h1, h2, h3, .ly-display { font-stretch: var(--display-stretch, 100%); font-variation-settings: var(--display-axes, normal); }
.card h3, .card h4, .tile h3 { font-stretch: 100%; font-variation-settings: normal; }

/* ---------------------------------------------------------------- one section per screen
   system.css makes every main > section exactly one or two screens (--h) with its h2 first; nothing
   here centres a section's content (a short block centred in a tall screen hides that it is short) */
main > .section .section-head { margin-bottom: var(--ly-head-gap); }
/* a visual sized to what one screen leaves after its heading (--ly-head, about 160px for a two-line
   h2 and a sub-line): or put .fill on it inside the section's .wrap */
.fit-screen { height: clamp(280px, calc(var(--ly-fit) - var(--ly-head, 160px) - var(--ly-head-gap)), 720px); }
@media (max-width: 1023px) { .fit-screen { height: auto; } }

/* ---------------------------------------------------------------- edges */
/* no one-sided bleed: a block either stands on the container's lines or runs edge to edge (a
   full-bleed photograph or ground). layout-check.js flags a block that touches one viewport edge. */

/* ---------------------------------------------------------------- the hero (every type) */
/* the box (exactly the first screen, padded under the fixed header) is system.css's; .hero-text fills
   what is left, and a type that grows a picture down to the fold lets it take the rest */
.hero { position: relative; isolation: isolate; }
.hero > .wrap > .hero-text { height: 100%; }
.hero__in { display: flex; flex-direction: column; }
.hero h1 { font-size: var(--ly-h1); line-height: 1.02; letter-spacing: var(--display-tracking-xl, var(--display-tracking)); text-wrap: balance; }
.hero .hero__lead { font-size: var(--ly-lead); line-height: 1.5; max-width: 42ch; text-wrap: pretty; color: var(--ink); }
.hero .hero__lead .muted, .hero__lead.muted { color: var(--muted); }
.hero__kicker { display: flex; align-items: center; gap: var(--s-2); font-size: var(--t-small); color: var(--muted); }
.hero__kicker .dot { width: 8px; height: 8px; border-radius: 50%; background: var(--accent); flex: none; }
.hero__text > * + * { margin-top: var(--s-5); }
.hero__text > .btn-row { margin-top: var(--s-6); }
.hero__text > .hero__kicker + h1 { margin-top: var(--s-4); }
.hero__grid { align-items: center; align-content: center; }
/* a hero grid with a picture as tall as the hero (split, collage, triptych): one row, the whole height */
.hero-text.hero__grid { grid-template-rows: minmax(0, 1fr); align-content: stretch; }
.hero__head { display: grid; grid-template-columns: repeat(12, minmax(0, 1fr)); column-gap: var(--gutter); row-gap: var(--s-5); align-items: end; }
.hero__head > :first-child { grid-column: 1 / span 7; }
.hero__head > :first-child > * + * { margin-top: var(--s-4); }
.hero__head > .btn-row { grid-column: 9 / -1; justify-content: flex-end; }
.hero__foot { margin-top: var(--s-6); align-items: end; }
.hero__foot > .btn-row { justify-content: flex-end; }
.hero__fill { flex: 1 1 auto; position: relative; min-height: 0; }
.hero__media { position: relative; overflow: hidden; border-radius: var(--r-lg); background: var(--surface); }
.hero__media > img, .hero__media > video, .hero__media > picture > img {
  position: absolute; inset: 0; width: 100%; height: 100%; max-width: none; object-fit: cover; object-position: var(--ly-pos, 50% 50%);
}
@media (max-width: 1023px) {
  .hero__head > :first-child, .hero__head > .btn-row { grid-column: 1 / -1; justify-content: flex-start; }
  .hero__foot > .btn-row { justify-content: flex-start; }
  .hero__media > img { object-position: var(--ly-pos-phone, var(--ly-pos, 50% 50%)); }
}
/* facts docked to the hero's foot: a hairline and four short facts on the column lines */
.hero__facts { display: grid; grid-template-columns: repeat(12, minmax(0, 1fr)); column-gap: var(--gutter); row-gap: var(--s-5);
               border-top: 1px solid var(--line); padding-top: var(--s-5); margin: 0; }
.hero__facts > div { grid-column: auto / span 3; display: grid; gap: var(--s-1); }
.hero__facts dt { font-size: var(--t-small); color: var(--muted); }
.hero__facts dd { margin: 0; font-size: var(--t-body); font-weight: 500; }
@media (max-width: 639px) { .hero__facts > div { grid-column: auto / span 6; } }
/* a proof strip (logos, a rating) docked to the hero's foot instead of a section of its own */
.hero__dock { margin-top: auto; padding-top: var(--s-7); }

/* a small card laid on a hero picture, 24px in from a corner (the opening hours, today's bake,
   an order's state): one, opaque, in the page's colours */
.ly-float {
  position: absolute; z-index: 2; left: var(--s-5); bottom: var(--s-5); width: min(300px, calc(100% - 2 * var(--s-5)));
  display: grid; gap: var(--s-1); padding: var(--s-4); border-radius: var(--r-md);
  background: var(--bg); color: var(--ink); border: 1px solid var(--line);   /* flat: no shadow at rest (ST-97) */
}
.ly-float--tr { left: auto; bottom: auto; right: var(--s-5); top: var(--s-5); }
.ly-float__row { display: flex; align-items: center; justify-content: space-between; gap: var(--s-3); font-size: var(--t-small); color: var(--muted); }
.ly-float__title { font-weight: 500; font-size: var(--t-body); }
.ly-live { display: inline-flex; align-items: center; gap: 6px; font-size: var(--t-label); color: var(--ink); }
.ly-live::before { content: ""; width: 7px; height: 7px; border-radius: 50%; background: var(--accent); box-shadow: 0 0 0 0 color-mix(in oklab, var(--accent) 50%, transparent); animation: ly-pulse 2.4s ease-out infinite; }
@keyframes ly-pulse { 70% { box-shadow: 0 0 0 7px transparent; } 100% { box-shadow: 0 0 0 0 transparent; } }
/* kind other (a cafe, a shop, a studio) and a place: nothing moves by itself (MO-07) */
:root:is([data-type="place"], :has(meta[name="site-kind"][content="other"])) .ly-live::before { animation: none; }

/* 1. hero--split: text on span-5, the picture on span-6 start-7, ending on the right line (never out
      to the viewport edge: the page's two edges stay equal), as tall as the screen less the hero's
      padding: 628 x 740 at 1440x900. A phone stacks text then picture, and the picture takes what the
      first screen leaves (at least 240px), so the hero is still one screen */
.hero--split .hero__media { height: 100%; min-height: 0; }
.hero--split .hero__grid { align-items: stretch; }
.hero--split .hero__text { align-self: start; }
.hero--collage .hero__text { align-self: start; }
/* the words' first line on the picture's top line: an h1 first in its column starts at its capitals */
:is(.hero--split, .hero--collage) .hero__text > h1:first-child { text-box: trim-start cap alphabetic; }
@media (max-width: 1023px) {
  .hero--split .hero__grid, .hero--collage .hero__grid { display: flex; flex-direction: column; align-items: stretch; }
  .hero--split .hero__media { height: auto; flex: 1 1 0; min-height: 240px; margin-top: var(--s-6); }
}

/* 2. hero--photo (product, service, software): the photograph over the whole first screen and the
      words on its calm side, in a solid box in the page's own colours (the default), or with
      data-box="none" straight on a calm part of the photograph that is dark (data-tone="dark") or
      light (data-tone="light") and empty at the desktop crop. Nothing is laid over the photograph:
      no scrim, gradient or veil (PH-31, VI-49). From 1023 down the photograph is a layer under the
      header (--photo) and the words sit under it on the ground. On a place see hero--place. */
.hero--photo { color: var(--ink); background: #1b1b1b; }
.hero--photo > .hero__bg { position: absolute; inset: 0; z-index: 0; width: 100%; height: 100%; max-width: none; object-fit: cover; object-position: var(--ly-pos, 50% 50%); }
.hero--photo[data-box="none"][data-tone="dark"] { --ink: #ffffff; --muted: #ffffff; --line: rgb(255 255 255 / .4); --line-strong: rgb(255 255 255 / .7); color: #fff; }
.hero--photo[data-box="none"][data-tone="light"] { --ink: #111111; --muted: #111111; --line: rgb(17 17 17 / .24); --line-strong: rgb(17 17 17 / .5); color: #111; }
.hero--photo:not([data-box="none"]) .hero__text {
  padding: var(--s-7); background: var(--p-bg); color: var(--p-ink); border-radius: var(--r-lg);
  --ink: var(--p-ink); --muted: var(--p-muted); --line: color-mix(in oklab, var(--p-ink) 11%, transparent); --line-strong: color-mix(in oklab, var(--p-ink) 20%, transparent);
}
/* the facts are small words: on the photograph only on a calm part (data-box="none"), never beside a box */
.hero--photo:not([data-box="none"]) .hero__facts { display: none; }
.hero--photo > .wrap > .hero-text { display: flex; flex-direction: column; justify-content: flex-end; }
.hero--photo[data-calm="left"] > .wrap > .hero-text, .hero--photo[data-calm="right"] > .wrap > .hero-text { justify-content: center; }
.hero--photo[data-calm="top"] > .wrap > .hero-text { justify-content: flex-start; }
.hero--photo .hero__text { width: calc(50% - var(--gutter) / 2); }   /* six of the twelve columns: 628 at 1440 */
@media (max-width: 1023px) { .hero--photo .hero__text { width: 100%; } }
.hero--photo[data-calm="right"] .hero__text { margin-left: auto; }
.hero--photo h1 { font-size: var(--ly-h1-xl); max-width: 14ch; }
.hero--photo .hero__facts { margin-top: var(--s-8); }
.hero--photo[data-calm="left"] > .wrap > .hero-text, .hero--photo[data-calm="right"] > .wrap > .hero-text { position: relative; }
.hero--photo[data-calm="left"] .hero__facts, .hero--photo[data-calm="right"] .hero__facts { position: absolute; left: 0; right: 0; bottom: 0; margin: 0; }
/* the page's own button pair (never a third style) */
.hero--photo .btn-row { --ink: var(--p-ink); --bg: var(--p-bg); --action: var(--p-action); --action-ink: var(--p-action-ink); --line-strong: color-mix(in oklab, var(--p-ink) 20%, transparent); }
html:is([data-buttons="arrow"], [data-buttons="block"]) .hero--photo[data-box="none"][data-tone="dark"] .btn--secondary { color: #fff; text-decoration-color: rgb(255 255 255 / .6); }
/* a tablet and a phone: no calm part survives every crop, so the photograph is a layer under the
   header (--photo) and the words sit --hgap under it on the page ground, in the page's own colours */
@media (max-width: 1023px) {
  .hero--photo, .hero--photo[data-tone] { background: var(--bg); color: var(--p-ink); --ink: var(--p-ink); --muted: var(--p-muted);
    --line: color-mix(in oklab, var(--p-ink) 11%, transparent); --line-strong: color-mix(in oklab, var(--p-ink) 20%, transparent); }
  .hero--photo { --photo: min(46svh, 56vw); }
  .hero--photo > .hero__bg { top: var(--nav); bottom: auto; height: var(--photo); object-position: var(--ly-pos-phone, var(--ly-pos, 50% 50%)); }
  .hero--photo .hero__facts { display: none; }
  .hero--photo > .wrap > .hero-text, .hero--photo[data-calm] > .wrap > .hero-text { justify-content: flex-start; }
  .hero--photo .hero__text, .hero--photo:not([data-box="none"]) .hero__text { padding: 0; background: none; border-radius: 0; }
  .hero--photo[data-calm="right"] .hero__text { margin-left: 0; }
  html:is([data-buttons="arrow"], [data-buttons="block"]) .hero--photo[data-tone="dark"] .btn--secondary { color: var(--p-ink); text-decoration-color: var(--line-strong); }
}
@media (max-width: 639px) { .hero--photo { --photo: 42svh; } }

/* 0. hero--place: the only place hero, never dealt (local-business Recipe 3; ST-53, PH-47, PH-81).
      The client's photograph full-bleed under the header (--photo: 480 at 1440x900, y 72-552, at most
      53% of the screen; a phone's 4:3 crop, 390x292), nothing on it. On the ground below, --hgap
      (40, 24 on a phone) under it: the h1 on columns 1-6 (48/53, 32/37 on a phone), and from the
      split line the sub-line (the kind of place, where, today's hours) and one filled 48px button.
      html[data-type="place"] .hero--photo is drawn the same way (the older markup).
      <section class="hero hero--place"><img class="hero__bg" src=".." alt=".." fetchpriority="high">
        <div class="wrap"><div class="hero-text"><div class="hero__text">
          <h1>..</h1><p class="hero__lead">..</p><div class="btn-row"><a class="btn btn--primary">..</a></div>
        </div></div></div></section> */
.hero--place, :root[data-type="place"] .hero--photo { --photo: min(480px, 53svh); background: var(--bg); color: var(--ink); }
.hero--place > .hero__bg, :root[data-type="place"] .hero--photo > .hero__bg {
  position: absolute; left: 0; right: 0; top: var(--nav); bottom: auto; z-index: 0; width: 100%; height: var(--photo); max-width: none;
  object-fit: cover; object-position: var(--ly-pos, 50% 50%); border-radius: 0;
}
/* the hero as photos.mjs fetch prints it: <picture class="hero__bg"><source media="(max-width: 639px)" ..><img ..></picture>.
   The picture is the layer (placed as img.hero__bg above, out of the flow beside the .wrap); its img fills it */
:is(.hero--photo, .hero--place) > picture.hero__bg { display: block; overflow: hidden; }
:is(.hero--photo, .hero--place) > picture.hero__bg > img {
  display: block; width: 100%; height: 100%; max-width: none; border-radius: 0;
  object-fit: cover; object-position: var(--ly-pos, 50% 50%);
}
@media (max-width: 1023px) {
  :is(.hero--photo, .hero--place) > picture.hero__bg > img { object-position: var(--ly-pos-phone, var(--ly-pos, 50% 50%)); }
}
:root[data-type="place"] .hero--photo, :root[data-type="place"] .hero--photo[data-tone] { --ink: var(--p-ink); --muted: var(--p-muted); color: var(--p-ink); }
:is(.hero--place, :root[data-type="place"] .hero--photo) > .wrap > .hero-text { display: block; }
:is(.hero--place, :root[data-type="place"] .hero--photo) .hero__text {
  display: grid; grid-template-columns: repeat(12, minmax(0, 1fr)); column-gap: var(--gutter); align-content: start; align-items: start;
  width: 100%; margin: 0; padding: 0; background: none; border-radius: 0;
}
:is(.hero--place, :root[data-type="place"] .hero--photo) .hero__text > * { grid-column: 7 / -1; margin: 0; }
:is(.hero--place, :root[data-type="place"] .hero--photo) .hero__text > h1 {
  grid-column: 1 / span 6; grid-row: 1 / span 3; max-width: none; font-size: var(--t-h1); line-height: 1.104; text-box: trim-start cap alphabetic;
}
:is(.hero--place, :root[data-type="place"] .hero--photo) .hero__lead { font-size: var(--t-lead); line-height: 1.5; max-width: 40ch; color: var(--ink); }
:is(.hero--place, :root[data-type="place"] .hero--photo) .hero__text > .btn-row { margin-top: var(--s-5); }
:is(.hero--place, :root[data-type="place"] .hero--photo) .btn { height: 48px; }
:is(.hero--place, :root[data-type="place"] .hero--photo) .hero__facts { display: none; }
@media (max-width: 1023px) {
  .hero--place, :root[data-type="place"] .hero--photo { --photo: min(56vw, 46svh); }
  :is(.hero--place, :root[data-type="place"] .hero--photo) .hero__text { display: flex; flex-direction: column; gap: var(--s-5); }
  :is(.hero--place, :root[data-type="place"] .hero--photo) .hero__text > .btn-row { margin-top: 0; }
}
/* a tablet held upright (820x1180): a 56vw photograph would leave a third of the screen empty under
   the button, so the photograph grows to a little over half the screen (656 at 820x1180) */
@media (min-width: 640px) and (max-width: 1023px) and (orientation: portrait) {
  .hero--place, :root[data-type="place"] .hero--photo { --photo: min(80vw, 56svh); }
}
@media (max-width: 639px) {
  .hero--place, :root[data-type="place"] .hero--photo { --photo: 75vw; }
  :is(.hero--place, :root[data-type="place"] .hero--photo) .hero__text > h1 { line-height: 1.156; }
  :is(.hero--place, :root[data-type="place"] .hero--photo) .hero__lead { font-size: 18px; line-height: 1.556; }
  :is(.hero--place, :root[data-type="place"] .hero--photo) .hero__text > .btn-row { margin-top: 0; }
}

/* 3. hero--product: the head row (h1 and line left, the buttons right on the line's baseline), then
      one object on a stage that fills the rest of the screen. The lit, rounded stage below is for a print
      or a scene (tech, games); the product itself (a render, a cut-out, a photograph) takes 3b: no box */
.hero--product .hero__stage { margin-top: var(--s-7); }
.hero__stage {
  position: relative; overflow: hidden; border-radius: var(--r-lg); min-height: clamp(260px, 40svh, 400px); flex: 1 1 auto;
  background:
    radial-gradient(60% 70% at 50% 42%, color-mix(in oklab, var(--ground-surface, #e9e0d2) 90%, white), transparent 72%),
    linear-gradient(to bottom, var(--ground-bg, #d9ccb7) 0 72%, color-mix(in oklab, var(--ground-bg, #d9ccb7) 86%, black) 100%);
}
.hero__stage > img { position: absolute; inset: 0; width: 100%; height: 100%; max-width: none; object-fit: cover; object-position: var(--ly-pos, 50% 50%); }
.hero__stage > img.is-cutout { object-fit: contain; padding: 5% 0 4%; }
.hero__stage > [data-print], .hero__stage > [data-scene] { position: absolute; inset: 0; }
/* 3b. the product itself on the stage, never boxed (in testing: a render at 44% of a rounded lit box
      read as "small and boxed"). The lit box above stays for a print or a world scene (tech, games).
      - a render (a [data-product] frame) or a cut-out (img.is-cutout): no box, no gradient, no radius;
        the page's own ground. The stage runs across the wrap and down to --s-5 above the fold, the
        head row's gap is --s-6, and a render's frame reaches 7% past the stage's top and bottom, so
        the product's long side fills the stage's height (the place's 0.86-0.88 margin taken back):
        a wide product's widest view (a bike's side) spans 80% or more of the wrap at 1440.
      - a product photograph (any other img): edge to edge across the screen, square corners, no ground;
        crop it (--ly-pos) so the product spans 80% or more of the wrap.
      <div class="hero__stage"><div class="frame" data-product="…/specs/<id>.json" data-product-place="stage"
        data-product-view="side"></div> one to three .ly-spot </div> */
.hero--product .hero__stage:is(:has(> [data-product]), :has(> img.is-cutout)) {
  margin-top: var(--s-6); margin-bottom: calc(var(--s-5) - var(--hpadb, 64px));
  border-radius: 0; background: none; overflow: visible;
}
.hero--product .hero__stage > [data-product] {
  position: absolute; inset: -7% 0; width: auto; height: auto; min-height: 0;
  padding: 0; border: 0; border-radius: 0; background: none; overflow: visible;
}
.hero--product .hero__stage > [data-product]::before { content: none; }
/* the head row gives the product its height: the h1 on cols 1-7 (two lines), the lead and the buttons
   stacked on cols 9-12, their foot on the h1's foot (the head ends at y 278 at 1440x900, against y 373
   with the lead under the h1); the stage takes the rest: 1280x566 at y 310 */
@media (min-width: 1024px) {
  .hero--product:has(.hero__stage > :is([data-product], img, picture)) .hero__head { grid-template-rows: auto auto; row-gap: var(--s-5); }
  .hero--product:has(.hero__stage > :is([data-product], img, picture)) .hero__head > :first-child { display: contents; }
  .hero--product:has(.hero__stage > :is([data-product], img, picture)) .hero__head h1 { grid-column: 1 / span 7; grid-row: 1 / span 2; align-self: end; margin: 0; }
  .hero--product:has(.hero__stage > :is([data-product], img, picture)) .hero__head .hero__lead { grid-column: 9 / -1; grid-row: 1; align-self: end; margin: 0; max-width: none; }
  .hero--product:has(.hero__stage > :is([data-product], img, picture)) .hero__head > .btn-row { grid-column: 9 / -1; grid-row: 2; justify-content: flex-start; }
}
/* a small laptop (1024-1279): h1 on cols 1-6, the lead and the buttons on cols 7-12, so the two
   buttons stay on one line and the lead on three */
@media (min-width: 1024px) and (max-width: 1279px) {
  .hero--product:has(.hero__stage > :is([data-product], img, picture)) .hero__head h1 { grid-column: 1 / span 6; }
  .hero--product:has(.hero__stage > :is([data-product], img, picture)) .hero__head .hero__lead,
  .hero--product:has(.hero__stage > :is([data-product], img, picture)) .hero__head > .btn-row { grid-column: 7 / -1; }
}
.hero--product .hero__stage:has(> img:not(.is-cutout), > picture) {
  border-radius: 0; background: none; margin-inline: calc(50% - 50vw); margin-bottom: calc(0px - var(--hpadb, 64px));
}
@media (max-width: 639px) { .hero--product .hero__stage:has(> img:not(.is-cutout), > picture) { margin-bottom: 0; } }
/* every product render frame is the product on its section's ground: never a grey panel (sections.md
   Product, rule 1); the frame keeps its size and clips a close-up at its edges */
.frame[data-product] { background: none; border: 0; padding: 0; }
/* a hotspot on the product: a disc that opens one short spec card on hover, focus or tap */
.ly-spot { position: absolute; z-index: 2; left: var(--x); top: var(--y); transform: translate(-50%, -50%); }
.ly-spot > button {
  display: grid; place-items: center; width: 32px; height: 32px; padding: 0; border-radius: 50%; cursor: pointer;
  border: 0; background: var(--p-bg, #fff); color: var(--p-ink, #111); box-shadow: 0 0 0 6px rgb(255 255 255 / .28), 0 6px 18px -6px rgb(0 0 0 / .45);
  transition: transform var(--dur) var(--ease), box-shadow var(--dur) var(--ease);
}
.ly-spot > button svg { width: 12px; height: 12px; transition: transform var(--dur) var(--ease); }
.ly-spot > button:hover, .ly-spot.is-open > button { transform: scale(1.08); box-shadow: 0 0 0 9px rgb(255 255 255 / .32), 0 6px 18px -6px rgb(0 0 0 / .45); }
.ly-spot.is-open > button svg { transform: rotate(45deg); }
.ly-spot__card {
  position: absolute; left: calc(100% + 12px); top: 50%; width: 230px; padding: var(--s-3) var(--s-4); border-radius: var(--r-md);
  background: var(--p-bg, #fff); color: var(--p-ink, #111); font-size: var(--t-small); line-height: 1.45;
  box-shadow: 0 16px 36px -16px rgb(0 0 0 / .45); opacity: 0; visibility: hidden; transform: translate(-6px, -50%);
  transition: opacity var(--dur) var(--ease), transform var(--dur) var(--ease), visibility 0s linear var(--dur);
}
.ly-spot__card strong { display: block; font-size: var(--t-body); font-weight: 500; margin-bottom: 2px; }
.ly-spot--left .ly-spot__card { left: auto; right: calc(100% + 12px); transform: translate(6px, -50%); }
.ly-spot.is-open .ly-spot__card, .ly-spot:hover .ly-spot__card, .ly-spot:focus-within .ly-spot__card { opacity: 1; visibility: visible; transform: translate(0, -50%); transition-delay: 0s; }
/* a phone: the card opens under its disc, centred on it and nudged by layout.js (--dx) so it stays
   16px inside the stage; never beside the disc, where the stage would cut it */
@media (max-width: 639px) {
  .hero--product .hero__stage { margin-top: var(--s-6); flex: none; min-height: 0; aspect-ratio: 4 / 5; }
  /* a render or a cut-out takes what the head leaves, at least 240 tall: the product's width is the bound */
  .hero--product .hero__stage:is(:has(> [data-product]), :has(> img.is-cutout)) { flex: 1 1 auto; aspect-ratio: auto; min-height: 240px; margin-bottom: 0; }
  .ly-spot__card, .ly-spot--left .ly-spot__card { left: 50%; right: auto; top: calc(100% + 10px); width: 230px;
    transform: translate(calc(-50% + var(--dx, 0px)), -4px); }
  .ly-spot.is-open .ly-spot__card, .ly-spot:hover .ly-spot__card, .ly-spot:focus-within .ly-spot__card { transform: translate(calc(-50% + var(--dx, 0px)), 0); }
}

/* 4. hero--editorial: the headline IS the picture. Three lines of big left-aligned type (128px at
      1440, 52 at 390) across the whole container, with one or two photographs set in the line as
      pills (.ly-pic, 0.8em tall, 1.9em wide), whole on the first frame (no opening, no settle: the
      h1 never reflows after load, MO-62); the lead and the buttons dock to the foot under a
      hairline. No frame under it: the type fills the screen. */
.hero--editorial .hero__in { justify-content: space-between; }
.hero--editorial h1 { font-size: var(--ly-h1-mega); line-height: .98; letter-spacing: var(--display-tracking-xl, -0.04em); max-width: none; margin-block: auto; padding-block: var(--s-6); text-wrap: initial; }
.hero--editorial .hero__foot { margin-top: 0; padding-top: var(--s-5); border-top: 1px solid var(--line); align-items: end; }
.ly-pic {
  display: inline-block; position: relative; overflow: hidden; vertical-align: -0.04em;
  height: .8em; width: var(--w, 1.9em); margin-inline: .04em; border-radius: 999px; background: var(--surface);
}
.ly-pic > img { position: absolute; inset: 0; width: 100%; height: 100%; max-width: none; object-fit: cover; object-position: var(--ly-pos, 50% 50%); }
.ly-pic--sq { border-radius: .16em; }
@media (max-width: 639px) { .ly-pic { --w: 1.5em; } .hero--editorial h1 { padding-block: var(--s-5); } }

/* 5. hero--collage: text on span-5, a UI collage from column 7 out through the right viewport edge;
      the main window starts 56px in from its left and top and runs out through the right and bottom */
.hero--collage h1 { font-size: var(--ly-h1-col); }
.hero__art {
  position: relative; overflow: hidden; height: 100%; min-height: 0;
  border-radius: var(--r-lg); background: var(--ground-bg, #d9ccb7);
  --fx-bg: var(--ground-bg); --fx-surface: var(--ground-surface); --fx-ink: var(--ground-ink); --fx-accent: var(--ground-accent);
}
/* the frame stands on the container's right line; the window inside is cut by the frame's right and
   bottom edges, never by the viewport. It runs --ly-cut (28px) past the right edge and keeps that much
   plus 20px of its own empty margin at its right (--ly-cut-room, on its content), so the cut falls
   after the last column a visitor reads (the status chip), never through it. A window's widest
   columns (times, ids) are left out or hidden (.ly-cut-hide). */
.hero__art > .ly-piece { position: absolute; }
.hero__art { --ly-cut: 28px; --ly-cut-room: calc(var(--ly-cut) + 20px); }
.hero__art > .ly-piece--main { left: var(--s-8); top: var(--s-8); width: calc(100% - var(--s-8) + var(--ly-cut)); }
.hero__art .ly-cut-hide { display: none; }
@media (max-width: 1023px) {
  .hero__art { height: auto; flex: 1 1 0; min-height: 220px; margin-top: var(--s-6); }
  .hero__art > .ly-piece--main { left: var(--s-6); top: var(--s-6); width: calc(100% - var(--s-6) + var(--ly-cut)); }
}
/* a phone: the window alone, its side bar hidden, 16px in; the small card waits for a wider screen
   (a 300px card over a 340px frame covers the window and leaves a sliver of cut rows) */
@media (max-width: 639px) {
  .hero__art > .ly-piece--main { left: var(--s-4); top: var(--s-4); width: calc(100% - var(--s-4) + var(--ly-cut)); }
  .hero__art > .ly-piece:not(.ly-piece--main) { display: none; }
}

/* 6. hero--word: an oversized headline (144px) across the container with one live word last, then a
      foot row: the line and the buttons on span-6, the four facts as a column on span-5 start-8
      (the right half has a job: the prices and promises a visitor checks first) */
.hero--word, .hero--editorial { overflow-x: clip; }
.hero--word .hero__in { justify-content: center; }
.hero--word h1 { font-size: var(--ly-h1-word); line-height: .92; max-width: none; }
.hero--word .hero__foot { margin-top: var(--s-8); align-items: start; }
.hero--word .hero__foot .btn-row { margin-top: var(--s-6); }
.hero--word .hero__facts { margin-top: var(--s-8); }
/* facts as a column: one row each, the label left and the value right, hairlines between */
.hero__facts.hero__facts--col { display: grid; grid-template-columns: minmax(0, 1fr); row-gap: 0; padding-top: 0; border-top: 0; margin-top: 0; }
.hero__facts--col > div { grid-column: auto; display: flex; align-items: baseline; justify-content: space-between; gap: var(--s-4);
                          padding-block: var(--s-3); border-top: 1px solid var(--line); }
.hero__facts--col > div:last-child { border-bottom: 1px solid var(--line); }
.hero__facts--col dd { text-align: right; }
@media (max-width: 1023px) {
  .hero--word .hero__foot > .hero__facts--col { margin-top: var(--s-7); grid-template-columns: repeat(2, minmax(0, 1fr)); column-gap: var(--gutter); }
  .hero__facts--col > div { display: grid; gap: var(--s-1); justify-content: start; }
  .hero__facts--col > div:nth-last-child(2) { border-bottom: 1px solid var(--line); }
  .hero__facts--col dd { text-align: left; }
}
/* the live word: one slot in the headline as wide as its widest word (the grid's one cell holds them
   all), so a change never moves another word or breaks a line; each word carries its own underline in
   the accent and brings it with it */
.ly-word { position: relative; display: inline-grid; vertical-align: bottom; overflow: clip; overflow-clip-margin: .14em; margin-bottom: -.1em; }
.ly-word > span { grid-area: 1 / 1; justify-self: start; white-space: nowrap; padding-bottom: .1em;
                  background: linear-gradient(var(--accent), var(--accent)) 0 calc(100% - .05em) / 100% max(3px, .05em) no-repeat;
                  transition: transform .6s var(--ease, ease), opacity .6s var(--ease, ease); }
.ly-word > span.is-out { transform: translateY(-105%); opacity: 0; }
.ly-word > span.is-next { transform: translateY(105%); opacity: 0; }
/* letters that answer the pointer: each letter's weight (and width, on a face with a wdth axis). The
   headline's lines are locked first (layout.js puts a break where each line ends at rest and stops
   wrapping), so a heavier, wider letter never sends a word to the next line */
.ly-letters .ly-l { display: inline-block; font-weight: var(--w, inherit); font-stretch: var(--wd, inherit); transition: font-weight .32s var(--ease, ease), font-stretch .32s var(--ease, ease); }
h1.ly-locked { white-space: nowrap; text-wrap: nowrap; }

/* 7. hero--film: a cinema frame. The film's poster fills one rounded frame on the container's lines,
      the whole first screen less 16px (1280 x 804 at 1440x900); the headline, the line and the
      buttons stand in a solid box of the page's inverse ground at the frame's lower left, the play
      control and its label (a solid pill) at its right foot (x.ai's film card). Nothing is laid
      over the film to make words readable: no scrim or gradient (PH-31). */
.hero--film { padding-block: var(--s-4); }
.hero--film .hero__media { flex: 1 1 auto; min-height: 520px; background: #000; color: #fff; display: flex; flex-direction: column; justify-content: flex-end; }
.hero__over { position: relative; z-index: 2; display: grid; grid-template-columns: minmax(0, 1fr) auto; align-items: end; gap: var(--s-6); padding: var(--s-6); }
.hero__over > :first-child {
  justify-self: start; max-width: 600px; padding: var(--s-7); border-radius: var(--r-md);
  background: var(--p-inv-bg); color: var(--p-inv-ink);
  --bg: var(--p-inv-bg); --ink: var(--p-inv-ink); --muted: var(--p-inv-muted); --action: var(--p-inv-action); --action-ink: var(--p-inv-action-ink);
  --line: color-mix(in oklab, var(--p-inv-ink) 14%, transparent); --line-strong: color-mix(in oklab, var(--p-inv-ink) 30%, transparent);
}
.hero__over h1 { max-width: 15ch; }
.hero__over .hero__lead { color: var(--p-inv-ink); margin-top: var(--s-4); }
.hero__over .btn-row { margin-top: var(--s-6); }
.hero--film .ly-play { position: relative; left: auto; bottom: auto; }
.hero--film .ly-play__label { padding: var(--s-3) var(--s-4); border-radius: var(--r-md); background: var(--p-inv-bg); color: var(--p-inv-ink); }
/* the running time under the play label (before the media block below, which hides it on a phone) */
.ly-play__label small { display: block; margin-top: 2px; font-size: var(--t-small); color: var(--p-inv-muted); font-weight: 400; }
@media (max-width: 1023px) {
  .hero__over { grid-template-columns: 1fr; gap: var(--s-4); padding: var(--s-5); }
  .hero__over > :first-child { max-width: none; justify-self: stretch; padding: var(--s-6); }
  .hero--film .ly-play { order: -1; justify-self: start; gap: var(--s-3); font-size: var(--t-body); }
  .ly-play__label small { display: none; }
}
@media (max-width: 639px) {
  .hero__over { padding: var(--s-3); }
  .hero__over > :first-child { padding: var(--s-5); }
  .hero--film .hero__media { min-height: 0; }
}
.ly-play { position: absolute; z-index: 2; left: var(--s-6); bottom: var(--s-6); display: inline-flex; align-items: center; gap: var(--s-4);
           padding: 0; border: 0; background: none; color: #fff; cursor: pointer; font: 500 var(--t-lead)/1.2 var(--font-text); text-align: left; }
.ly-play__disc { display: grid; place-items: center; width: 64px; height: 64px; border-radius: 50%; background: #fff; color: #111; flex: none;
                 transition: transform var(--dur) var(--ease); }
.ly-play__disc svg { width: 18px; height: 18px; margin-left: 3px; }
.ly-play:hover .ly-play__disc, .ly-play:focus-visible .ly-play__disc { transform: scale(1.06); }
@media (max-width: 639px) { .ly-play__disc { width: 52px; height: 52px; } }

/* 8. hero--statement: a quiet statement over a ground (assets/backgrounds), facts docked at the foot */
.hero--statement .hero__in { justify-content: space-between; }   /* the h1 top-aligned at y 120 (layout-v4.md section 3), the facts docked at the foot */
.hero--statement h1 { font-size: var(--ly-h1-quiet); line-height: 1.1; max-width: 21ch; }
.hero--statement .hero__lead { margin-top: var(--s-5); }
.hero--statement .btn-row { margin-top: var(--s-6); }
.hero--statement .hero__facts { margin-top: var(--s-8); }
/* on a phone the facts of a photo or statement hero are a desktop line: the hero stays one screen and the
   ground's foot (a horizon's rim, the photograph) stays clear; the next section carries them */
@media (max-width: 639px) { .hero--statement .hero__facts { display: none; } }

/* 9. hero--triptych: the headline on span-3, a collage of three to five pieces on span-6 start-4
      that changes set every 2.8 s, one large button on span-3 at the right line (Figma's hero).
      Every set has one piece (a photograph or a piece with a quiet corner) that ends at 96% of the
      collage's width and height, so the pause disc sits on that piece's corner, never in empty ground;
      no piece covers a line of another piece's text */
.hero--triptych .hero__grid > :first-child { align-self: center; }
.hero--triptych h1 { font-size: var(--ly-h1-side); line-height: 1.04; }
.hero--triptych .hero__lead { font-size: var(--t-body); max-width: 30ch; margin-top: var(--s-4); }
.hero--triptych .hero__cta { display: flex; justify-content: flex-end; }
.hero--triptych .hero__cta .btn { height: 64px; padding-inline: var(--s-6); font-size: var(--t-lead); }
/* 1024 to 1279: three columns get too narrow, so it becomes a split: the headline and the button on
   span-4, the collage on span-7 start-6 to the right line */
@media (min-width: 1024px) and (max-width: 1279px) {
  .hero--triptych .hero__grid { grid-template-rows: 1fr auto 1fr; }
  .hero--triptych .hero__grid > :first-child { grid-column: 1 / span 4; grid-row: 1; align-self: end; }
  .hero--triptych .ly-collage { grid-column: 6 / span 7; grid-row: 1 / span 3; }
  .hero--triptych .hero__cta { grid-column: 1 / span 4; grid-row: 2; justify-content: flex-start; margin-top: var(--s-6); }
  .hero--triptych .hero__cta .btn { height: 56px; font-size: var(--t-body); }
}
.ly-collage { position: relative; height: 100%; min-height: 0; }
.ly-collage__set { position: absolute; inset: 0; transition: opacity .7s var(--ease, ease); }
.ly-collage__set:not(.is-on) { opacity: 0; pointer-events: none; }
/* a copy of the markup that lost its .is-on: the first set shows rather than an empty hole */
.ly-collage:not(:has(> .ly-collage__set.is-on)) > .ly-collage__set:first-child { opacity: 1; pointer-events: auto; }
.ly-collage .ly-piece { position: absolute; overflow: hidden; border-radius: var(--r-md); background: var(--surface);
                        box-shadow: 0 1px 2px rgb(0 0 0 / .08), 0 18px 40px -18px rgb(0 0 0 / .32); }
.ly-collage .ly-piece > img { width: 100%; height: 100%; object-fit: cover; max-width: none; }
.ly-collage > button.loop-pause.loop-pause { right: calc(4% + 12px); bottom: calc(4% + 12px); }
@media (max-width: 1023px) {
  .hero--triptych .hero__cta { justify-content: stretch; } .hero--triptych .hero__cta .btn { width: 100%; height: 56px; }
  .hero--triptych .hero__grid { display: flex; flex-direction: column; align-items: stretch; }
  .hero--triptych .hero__grid > :first-child { align-self: stretch; }
  .ly-collage { height: auto; flex: 1 1 0; min-height: 300px; margin-block: var(--s-6); }
}

/* 10. hero--centred: news pill, centred headline and two buttons, one product window rising out of
       a lit frame that takes the rest of the screen and fades at the hero's foot (x.ai's hero). The
       window's story (the question, the work, the answer, its actions) sits in its top 260px, so it
       stands whole above the fade, which starts at 82% of the frame */
.hero--centred .hero__c { display: grid; justify-items: center; text-align: center; gap: var(--s-4); max-width: 880px; margin-inline: auto; }
.hero--centred h1 { max-width: 16ch; }
.hero--centred .hero__lead { max-width: 52ch; color: var(--muted); }
.hero--centred .btn-row { justify-content: center; margin-top: var(--s-3); }
.hero--centred > .wrap > .hero-text { display: flex; flex-direction: column; }
.hero--centred .hero__window { flex: 1 1 0; min-height: clamp(220px, 32svh, 340px);
  -webkit-mask-image: linear-gradient(to bottom, #000 82%, transparent); mask-image: linear-gradient(to bottom, #000 82%, transparent);
  position: relative; overflow: hidden; margin-top: var(--s-6);
  border-radius: var(--r-lg); background: var(--ground-bg, #d9ccb7);
  --fx-bg: var(--ground-bg); --fx-surface: var(--ground-surface); --fx-ink: var(--ground-ink); --fx-accent: var(--ground-accent); }
.hero--centred .hero__window > .ly-piece { position: absolute; z-index: 1; left: 50%; top: var(--s-6); width: min(880px, calc(100% - 2 * var(--s-6))); transform: translateX(-50%); }
.ly-mark { display: inline-block; width: .8em; height: .8em; vertical-align: -.06em; margin-inline: .06em; border-radius: 50%; background: var(--ink); position: relative; }
.ly-mark::before, .ly-mark::after { content: ""; position: absolute; top: 27%; width: 13%; height: 42%; border-radius: 99px; background: var(--bg); transform: skewX(-12deg); }
.ly-mark::before { left: 31%; } .ly-mark::after { left: 56%; }
@media (max-width: 639px) {
  .hero--centred .btn-row { width: 100%; }
  .hero--centred .hero__window { margin-top: var(--s-6); min-height: 280px; }
  .hero--centred .hero__window > .ly-piece { top: var(--s-4); width: calc(100% - 2 * var(--s-4)); left: var(--s-4); transform: none; }
}

/* ---------------------------------------------------------------- micro-interactions the heroes share */
/* buttons keep the page's one hover (system.css, motion.css: the fill steps, the arrow moves 3px, the
   press sinks 1px; no shadow on a button, ST-105); a text link draws its underline from the left */
.ly-link { position: relative; font-weight: 500; text-decoration: none; background: linear-gradient(currentColor, currentColor) 0 100% / 0 1px no-repeat; transition: background-size .28s var(--ease); }
.ly-link:hover, .ly-link:focus-visible { background-size: 100% 1px; }

/* steps inside a piece of UI (layout.js [data-ly-steps]): each arrives in place, nothing travels */
[data-ly-steps] [data-step] { transition: opacity .35s var(--ease, ease), transform .35s var(--ease, ease); }
[data-ly-steps] [data-step].is-off { opacity: 0; transform: translateY(6px); }

/* ---------------------------------------------------------------- no entrance
   the hero is set on its first frame on every kind: no entrance fade, rise or opening (MO-62) */
@media (prefers-reduced-motion: reduce) {
  .ly-live::before { animation: none; }
  .ly-word > span, .ly-collage__set, .ly-letters .ly-l { transition: none; }
}
```

## assets/layout/layout.js

```js
/*
  taste-skill layout.js - the small moments of the layout v4 heroes (references/layout-v4.md, section 3).
  Load it once, after system.js:  <script type="module" src="assets/layout/layout.js"></script>
  Every loop runs through ui.loop (it rests off screen, carries the kit's pause button and restarts
  when the pointer enters its box); on a phone (under 640px), on kind other and on a place, and under
  reduced motion each shows its finished picture and holds.

  - [data-ly-word='["offices","pianos"]'] on a span.ly-word in the h1 (hero--word): one live word.
    The first word is in the markup; it changes every 2 s for two rounds, rests on the first, and
    plays a round again when the pointer enters the headline. The slot is as wide as the widest word
    (CSS grid, one cell), so a change never moves another word or breaks a line differently.
  - h1[data-ly-letters] (hero--word, hero--editorial): letters near the pointer take a heavier weight
    (+200), and a wider width (+10%) with data-ly-letters="wdth" on a face with a wdth axis (type.json
    "letters"). Fine pointers only; the h1 keeps one aria-label.
  - Lines locked: an h1 with a live word or letters keeps the lines it has at rest (a break where each
    line ends, then no wrapping), measured again on resize; a wider letter never re-breaks a line.
  - .ly-collage with two to four .ly-collage__set (hero--triptych): the sets crossfade every 2.8 s;
    nothing moves inside a set.
  - [data-ly-steps] on a box (a chat, a list, a status): it shows the finished picture first, holds
    it 1.8 s (data-ly-hold), then its [data-step] children arrive one by one 0.7 s apart
    (data-ly-steps="900" sets the step time) and it holds again; the pointer entering restarts it.
  - .ly-pic in an h1 (hero--editorial): a photograph set in the headline as a pill, whole on the first
    frame (layout.css), so nothing here; letters() leaves it whole.
  - .ly-spot (hero--product): the disc opens its spec card on a tap or Enter; hover and focus open it
    in CSS. Escape or a tap elsewhere closes it.
  - .ly-play[data-film="film.mp4"] (hero--film): plays the film in its frame with controls. Without
    data-film a <button> control is hidden; an <a class="ly-play" href> goes to the film's page.
*/
import ui from '../ui/ui.js';

const reduced = () => matchMedia('(prefers-reduced-motion: reduce)').matches;
const phone = () => innerWidth < 640;
// kind other (a cafe, a shop, a studio) and a place: nothing moves by itself (MO-07); the first word,
// the first set and the finished steps show, still
const other = () => { const m = document.querySelector('meta[name="site-kind"]'); return document.documentElement.dataset.type === 'place' || (!!m && m.content.trim().toLowerCase() === 'other'); };

/* ---------------------------------------------------------------- the live word */
function liveWord(slot) {
  let words;
  try { words = JSON.parse(slot.dataset.lyWord); } catch { return; }
  const first = slot.textContent.trim();
  words = [first, ...words.filter(w => w !== first)];
  if (words.length < 2) return;
  const h1 = slot.closest('h1');
  if (h1 && !h1.hasAttribute('aria-label')) h1.setAttribute('aria-label', h1.textContent.replace(/\s+/g, ' ').trim());
  slot.textContent = '';
  const spans = words.map((w, i) => {
    const s = document.createElement('span');
    s.textContent = w;
    if (i) s.className = 'is-next';
    s.setAttribute('aria-hidden', 'true');
    slot.append(s);
    return s;
  });
  let at = 0;
  const show = i => {
    spans.forEach((s, k) => { s.style.transition = 'none'; s.className = k === i ? '' : 'is-next'; void s.offsetWidth; s.style.transition = ''; });
    at = i;
  };
  if (reduced() || phone() || other()) { show(0); return; }
  const ctl = ui.loop(slot, async api => {
    for (let n = 1; n <= words.length; n++) {
      await api.wait(2000);
      const next = n % words.length, cur = at;
      spans[cur].className = 'is-out';
      spans[next].className = '';
      at = next;
      await api.wait(620);
      spans[cur].style.transition = 'none'; spans[cur].className = 'is-next'; void spans[cur].offsetWidth; spans[cur].style.transition = '';
    }
  }, { still: () => show(0), rest: 900, cycles: 2, pause: false });
  if (h1) h1.addEventListener('pointerenter', e => { if (e.pointerType !== 'touch') ctl.restart(); });
}

/* ---------------------------------------------------------------- letters that answer the pointer */
function letters(h1) {
  if (reduced() || !matchMedia('(pointer: fine)').matches) return;
  const axis = h1.dataset.lyLetters === 'wdth';
  const cs = getComputedStyle(h1);
  const base = parseFloat(cs.fontWeight) || 400;
  const heavy = Math.min(900, base + 200);
  const baseW = parseFloat(cs.fontStretch) || 100;
  const label = h1.getAttribute('aria-label') || h1.textContent.replace(/\s+/g, ' ').trim();
  const walk = node => {
    for (const n of [...node.childNodes]) {
      if (n.nodeType === 3) {
        const frag = document.createDocumentFragment();
        for (const part of n.textContent.split(/(\s+)/)) {
          if (!part) continue;
          if (/^\s+$/.test(part)) { frag.append(document.createTextNode(' ')); continue; }
          const wd = document.createElement('span');
          wd.style.whiteSpace = 'nowrap';
          for (const ch of part) { const l = document.createElement('span'); l.className = 'ly-l'; l.textContent = ch; wd.append(l); }
          frag.append(wd);
        }
        n.replaceWith(frag);
      } else if (n.nodeType === 1 && !n.matches('.ly-word, .ly-mark, .ly-pic')) walk(n);
    }
  };
  walk(h1);
  h1.classList.add('ly-letters');
  h1.setAttribute('aria-label', label);
  [...h1.childNodes].forEach(c => { if (c.nodeType === 1) c.setAttribute('aria-hidden', 'true'); });
  const ls = [...h1.querySelectorAll('.ly-l')];
  let px = 0, py = 0, raf = 0, on = false, boxes = null;
  const measure = () => { boxes = ls.map(l => { const r = l.getBoundingClientRect(); return [r.left + r.width / 2 + scrollX, r.top + r.height / 2 + scrollY]; }); };
  const draw = () => {
    raf = 0;
    if (!boxes) measure();
    const reach = parseFloat(getComputedStyle(h1).fontSize) * 2.4;
    ls.forEach((l, i) => {
      const d = on ? Math.hypot(px + scrollX - boxes[i][0], py + scrollY - boxes[i][1]) : Infinity;
      const k = Math.max(0, 1 - d / reach), e = k * k * (3 - 2 * k);
      l.style.setProperty('--w', Math.round(base + (heavy - base) * e));
      if (axis) l.style.setProperty('--wd', (baseW + 10 * e).toFixed(1) + '%');
    });
  };
  addEventListener('resize', () => { boxes = null; });
  document.fonts?.ready.then(() => { boxes = null; });
  const host = h1.closest('.hero') || h1;
  host.addEventListener('pointermove', e => { if (e.pointerType === 'touch') return; on = true; px = e.clientX; py = e.clientY; if (!raf) raf = requestAnimationFrame(draw); });
  host.addEventListener('pointerleave', () => { on = false; boxes = null; if (!raf) raf = requestAnimationFrame(draw); });
}

/* ---------------------------------------------------------------- lines locked */
// The h1's words as units: a text word becomes a nowrap span (letters() already made them); the live
// word, the mark and a pill are units of their own. At rest it reads where each line starts, puts a
// <br class="ly-br"> there and stops wrapping (h1.ly-locked); on resize it unlocks, waits a frame and
// reads again. Nothing moves when it locks: the breaks are where the browser had them.
function lockLines(h1) {
  const units = () => {
    const out = [];
    const walk = node => {
      for (const n of [...node.childNodes]) {
        if (n.nodeType === 3) {
          if (!n.textContent.trim()) continue;
          const frag = document.createDocumentFragment();
          for (const part of n.textContent.split(/(\s+)/)) {
            if (!part) continue;
            if (/^\s+$/.test(part)) { frag.append(document.createTextNode(' ')); continue; }
            const w = document.createElement('span'); w.className = 'ly-u'; w.style.whiteSpace = 'nowrap'; w.textContent = part; frag.append(w);
          }
          n.replaceWith(frag);
        } else if (n.nodeType === 1) {
          if (n.matches('.ly-word, .ly-mark, .ly-pic, .ly-u, br')) { if (!n.matches('br')) out.push(n); continue; }
          if (n.style.whiteSpace === 'nowrap' && n.querySelector('.ly-l')) { out.push(n); continue; }   // a word of letters()
          walk(n);
        }
      }
    };
    walk(h1);
    return out.length ? out : [...h1.querySelectorAll('.ly-u, .ly-word, .ly-mark, .ly-pic')];
  };
  let list = units();
  const unlock = () => { h1.querySelectorAll('br.ly-br').forEach(b => b.remove()); h1.classList.remove('ly-locked'); };
  const lock = () => {
    unlock();
    list = list.filter(u => u.isConnected);
    let top = null;
    const fs = parseFloat(getComputedStyle(h1).fontSize) || 16;
    for (const u of list) {
      const t = u.getBoundingClientRect().top;
      if (top !== null && t - top > fs * 0.5) { const br = document.createElement('br'); br.className = 'ly-br'; u.before(br); }
      top = t;
    }
    h1.classList.add('ly-locked');
    // a lock that no longer fits (read before the web font or the phone width arrived) is let go
    if (h1.scrollWidth > h1.clientWidth + 1) unlock();
  };
  // read the lines at rest: after the fonts, and after the pills in the line have opened to their width
  const opened = () => Promise.all([...h1.querySelectorAll('.ly-pic, .ly-pic > img')].flatMap(p => (p.getAnimations ? p.getAnimations() : []).map(a => a.finished.catch(() => {}))));
  const ready = document.fonts ? document.fonts.ready : Promise.resolve();
  ready.then(opened).then(() => requestAnimationFrame(lock));
  // the h1 changing width (a resize, the phone width arriving after the first paint) reads the lines again,
  // after the fonts; a phone's bars sliding (height only) changes nothing
  let t = 0, lastW = h1.clientWidth;
  const again = () => {
    if (h1.clientWidth === lastW) return;
    lastW = h1.clientWidth;
    clearTimeout(t); unlock();
    t = setTimeout(() => ready.then(() => requestAnimationFrame(lock)), 150);
  };
  if (typeof ResizeObserver === 'function') new ResizeObserver(again).observe(h1); else addEventListener('resize', again);
}

/* ---------------------------------------------------------------- the collage */
function collage(box) {
  const sets = [...box.querySelectorAll(':scope > .ly-collage__set')];
  if (!sets.length) return;
  const show = i => sets.forEach((s, k) => { s.classList.toggle('is-on', k === i); s.toggleAttribute('inert', k !== i); s.setAttribute('aria-hidden', String(k !== i)); });
  show(0);
  if (sets.length < 2 || phone() || other()) return;
  box.setAttribute('data-ui-pause-host', '');
  ui.loop(box, async api => {
    for (let n = 1; n <= sets.length; n++) { await api.wait(2800); show(n % sets.length); await api.wait(700); }
  }, { still: () => show(0), rest: 600 });
}

/* ---------------------------------------------------------------- steps inside a piece of UI */
function steps(box) {
  // data-step-at="n" puts a step at that place in the order (0 is first); the rest keep document order
  const items = [...box.querySelectorAll('[data-step]')]
    .map((el, i) => ({ el, at: el.dataset.stepAt != null ? parseFloat(el.dataset.stepAt) : i }))
    .sort((a, b) => a.at - b.at).map(o => o.el);
  if (!items.length) return;
  // house rule: the loops inside cards a little faster, and repeating. The first picture is the
  // finished one (a shot or a first look never meets an empty card); then it plays and holds again.
  const every = parseInt(box.dataset.lySteps, 10) || 700;
  const hold = parseInt(box.dataset.lyHold, 10) || 1800;
  const set = n => items.forEach((s, k) => s.classList.toggle('is-off', k >= n));
  const done = () => set(items.length);
  done();
  if (reduced() || phone() || other()) return;
  ui.loop(box, async api => {
    await api.wait(hold);
    set(0);
    for (let n = 1; n <= items.length; n++) { await api.wait(n === 1 ? 380 : every); set(n); }
  }, { still: done, rest: 1200 });
}

/* ---------------------------------------------------------------- hotspots */
function spot(el) {
  const btn = el.querySelector(':scope > button');
  if (!btn) return;
  const card = el.querySelector('.ly-spot__card');
  const stage = el.closest('.hero__stage') || el.parentElement;
  // the card opens on the side with room: a disc right of the middle opens it to its left (desktop);
  // on a phone it opens under the disc, nudged (--dx) to stay 16px inside the stage
  const place = () => {
    if (!card) return;
    const x = parseFloat(getComputedStyle(el).getPropertyValue('--x')) || 0;
    if (!el.dataset.lySide) el.classList.toggle('ly-spot--left', x > 50);
    el.style.setProperty('--dx', '0px');
    if (!phone()) return;
    const s = stage.getBoundingClientRect(), d = el.getBoundingClientRect();
    const w = card.offsetWidth, mid = d.left + d.width / 2, m = 16;
    const left = mid - w / 2, right = mid + w / 2;
    const dx = left < s.left + m ? s.left + m - left : right > s.right - m ? s.right - m - right : 0;
    el.style.setProperty('--dx', Math.round(dx) + 'px');
  };
  if (el.classList.contains('ly-spot--left')) el.dataset.lySide = 'left';
  place();
  addEventListener('resize', place);
  const set = on => { if (on) place(); el.classList.toggle('is-open', on); btn.setAttribute('aria-expanded', String(on)); };
  set(false);
  el.addEventListener('pointerenter', place);
  el.addEventListener('focusin', place);
  btn.addEventListener('click', () => set(!el.classList.contains('is-open')));
  document.addEventListener('pointerdown', e => { if (!el.contains(e.target)) set(false); });
  el.addEventListener('keydown', e => { if (e.key === 'Escape') { set(false); btn.focus(); } });
}

/* ---------------------------------------------------------------- the film */
function film(btn) {
  const src = btn.dataset.film;
  if (!src) { if (btn.tagName === 'BUTTON') btn.hidden = true; return; }
  btn.addEventListener('click', () => {
    const media = btn.closest('.hero__media');
    const v = document.createElement('video');
    Object.assign(v, { src, controls: true, autoplay: true, playsInline: true });
    const poster = media.querySelector('img');
    if (poster) v.poster = poster.currentSrc || poster.src;
    media.querySelectorAll('img, .ly-play').forEach(n => n.remove());
    media.append(v);
    v.focus();
  });
}

function init() {
  document.querySelectorAll('[data-ly-word]').forEach(liveWord);
  document.querySelectorAll('h1[data-ly-letters]').forEach(letters);
  document.querySelectorAll('h1[data-ly-letters], h1:has(.ly-word)').forEach(lockLines);
  document.querySelectorAll('.ly-collage').forEach(collage);
  document.querySelectorAll('[data-ly-steps]').forEach(steps);
  document.querySelectorAll('.ly-spot').forEach(spot);
  document.querySelectorAll('.ly-play').forEach(film);
}
if (document.readyState === 'loading') document.addEventListener('DOMContentLoaded', init, { once: true });
else init();

export default { init };
```

## assets/moments/lib.js

```js
/*
  moments/lib.js - the few helpers every moment shares (taste-skill). One ES module, no dependencies.

  reduced()            true under prefers-reduced-motion: a moment then shows its finished state only
  phone()              true on a phone: under 640px wide, or a touch screen in landscape under 500px tall
                       (a phone turned sideways). A moment shows its finished state there too.
  still()              reduced() || phone(): the one question every moment asks. The loader (moments.js)
                       hands each moment a lib whose reduced() answers still(), so a moment written
                       against reduced() is already static on a phone, and sets data-moment-still on its
                       host ("reduced" or "phone"). The answer is taken once, when the page loads.
  coarse()             true on a touch screen (no hover moments there)
  inCard(el)           true when el is, or sits in, a card, a plan, a result or a bento cell: nothing
                       on a card ever moves on scroll, so the loader leaves such a moment still
  scrub(el, fn, opts)  calls fn(progress, rect) from a requestAnimationFrame while el is near the
                       screen. mode 'hold' (default): 0 when el's top reaches the viewport top, 1 when
                       its bottom reaches the viewport bottom (a pinned stage's run). mode 'view': 0 when
                       el's top enters at the bottom edge, 1 when its bottom leaves at the top.
                       Returns { destroy, update }. Nothing is captured: the page scrolls natively.
  once(el, fn, opts)   fn once: when `threshold` of el is on screen, or el covers that share of the screen (a
                       target taller than the screen can never show it), or part of it has stayed on screen
                       for opts.rescue ms (3000), or the observer has not reported at all after 1.5 s of a
                       visible page. Nothing waits for ever. Returns { destroy }.
  split(el, unit)      wraps the words ('words') or letters ('chars') of el in spans
                       (.mo-w, .mo-c, each with an inner .mo-i that moves) and keeps the text readable:
                       the host gets aria-label, the spans aria-hidden. Returns the inner spans.
  lines(words)         groups word spans into lines by their offsetTop; returns an array of arrays.
  fontsReady(ms)       resolves when the fonts are loaded, or after ms.
  clamp, lerp, ease    the maths.
*/
const rq = typeof matchMedia === 'function' ? matchMedia('(prefers-reduced-motion: reduce)') : null;
export const reduced = () => !!(rq && rq.matches);
const pq = typeof matchMedia === 'function' ? [matchMedia('(max-width: 639px)'), matchMedia('(pointer: coarse) and (max-height: 499px) and (max-width: 959px)')] : [];
export const phone = () => pq.some(q => q.matches);
export const still = () => reduced() || phone();
export const stillWhy = () => (reduced() ? 'reduced' : phone() ? 'phone' : '');
export const inCard = el => !!(el && el.closest && el.closest('.card, .plan, .result, .knock, [data-card], .bento > *'));
export const coarse = () => typeof matchMedia === 'function' && matchMedia('(pointer: coarse)').matches;
export const clamp = (v, a = 0, b = 1) => Math.min(b, Math.max(a, v));
export const lerp = (a, b, t) => a + (b - a) * t;
export const ease = {
  out: t => 1 - Math.pow(1 - t, 3),
  quint: t => 1 - Math.pow(1 - t, 5),
  expo: t => (t >= 1 ? 1 : 1 - Math.pow(2, -10 * t)),
  inOut: t => (t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2),
  smooth: (a, b, x) => { const t = clamp((x - a) / (b - a)); return t * t * (3 - 2 * t); },
};

export function scrub(el, fn, { mode = 'hold', margin = '25% 0px' } = {}) {
  // on from the start, with the scroll listener: an observer that never reports (a dead stub, a page it cannot reach) leaves the
  // moment following the scroll instead of frozen at its first state; the observer turns it off when the target is far away
  let on = true, raf = 0, last = -1;
  const tick = () => {
    raf = 0;
    if (!on) return;
    const r = el.getBoundingClientRect(), vh = innerHeight;
    let p = mode === 'view' ? (vh - r.top) / (vh + r.height) : -r.top / Math.max(1, r.height - vh);
    p = clamp(p);
    if (p !== last) { last = p; fn(p, r); }
  };
  const req = () => { if (!raf) raf = requestAnimationFrame(tick); };
  const io = new IntersectionObserver(es => {
    on = es[es.length - 1].isIntersecting;
    removeEventListener('scroll', req); removeEventListener('resize', req);
    if (on) { addEventListener('scroll', req, { passive: true }); addEventListener('resize', req); req(); }
  }, { rootMargin: margin });
  io.observe(el);
  addEventListener('scroll', req, { passive: true }); addEventListener('resize', req); req();
  return {
    update: () => { last = -1; req(); },
    destroy() { io.disconnect(); removeEventListener('scroll', req); removeEventListener('resize', req); if (raf) cancelAnimationFrame(raf); },
  };
}

export function once(el, fn, { threshold = 0.35, margin = '0px', rescue = 3000 } = {}) {
  // fn runs once when `threshold` of el is on screen, or el covers that share of the screen (a target taller
  // than the screen can never show `threshold` of itself), or any part of it has stayed on screen for `rescue`
  // ms. An observer that has not reported at all after 1.5 s of a visible page runs fn too: nothing waits for ever
  let timer = 0, starve = 0, done = false;
  const go = () => { if (done) return; done = true; clearTimeout(timer); clearTimeout(starve); io.disconnect(); fn(); };
  const steps = [0];
  for (let t = 0.1; t < threshold - 1e-6; t += 0.1) steps.push(+t.toFixed(2));
  steps.push(threshold);
  const io = new IntersectionObserver(es => {
    clearTimeout(starve);
    const e = es[es.length - 1];
    if (!e.isIntersecting) { clearTimeout(timer); timer = 0; return; }
    const vh = e.rootBounds ? e.rootBounds.height : innerHeight;
    if (e.intersectionRatio >= threshold || e.intersectionRect.height >= vh * threshold) return go();
    if (!timer) timer = setTimeout(go, rescue);
  }, { threshold: steps, rootMargin: margin });
  const wait = () => { starve = setTimeout(() => (document.hidden ? wait() : go()), 1500); };
  wait();
  io.observe(el);
  return { destroy() { done = true; clearTimeout(timer); clearTimeout(starve); io.disconnect(); } };
}

export function split(el, unit = 'words') {
  if (!el.getAttribute('aria-label')) el.setAttribute('aria-label', el.textContent.replace(/\s+/g, ' ').trim());
  const inners = [];
  const wrap = (text, cls) => {
    const s = document.createElement('span');
    s.className = cls;
    const i = document.createElement('span');
    i.className = 'mo-i';
    i.textContent = text;
    s.append(i);
    inners.push(i);
    return s;
  };
  const walk = node => {
    [...node.childNodes].forEach(n => {
      if (n.nodeType === 3) {
        const frag = document.createDocumentFragment();
        n.textContent.split(/(\s+)/).forEach(part => {
          if (!part) return;
          if (/^\s+$/.test(part)) { frag.append(' '); return; }
          if (unit === 'chars') {
            const w = document.createElement('span');
            w.className = 'mo-w';
            for (const ch of part) w.append(wrap(ch, 'mo-c'));
            frag.append(w);
          } else frag.append(wrap(part, 'mo-w'));
        });
        n.replaceWith(frag);
      } else if (n.nodeType === 1 && !n.classList.contains('mo-w')) walk(n);
    });
  };
  walk(el);
  el.querySelectorAll('.mo-w').forEach(w => w.setAttribute('aria-hidden', 'true'));
  return inners;
}

export function lines(words) {
  const out = [];
  let top = null;
  for (const w of words) {
    const t = w.offsetTop;
    if (top === null || Math.abs(t - top) > 2) { out.push([]); top = t; }
    out[out.length - 1].push(w);
  }
  return out;
}

export const fontsReady = (ms = 900) => Promise.race([document.fonts ? document.fonts.ready : Promise.resolve(), new Promise(r => setTimeout(r, ms))]);

/*
  blocks(root, skip)   the section's words and pictures as whole blocks, for a moment that uncovers a section with a
                       clip (a disc, a line): a block is a heading, paragraph, list item, button, link, picture, UI
                       window or any element that carries text of its own, outermost first (a frame with its own
                       text is one block). A clip that slices through a block cuts its words mid-move, so such a
                       moment keeps each block invisible until the clip has passed the whole of it, then fades it
                       in whole. Returns { list, measure, set, release }:
    list      [{ el, l, t, r, b }] with the box of each block in root's own coordinates (set by measure)
    measure() reads the boxes again (call when the root's size changes or the fonts arrive)
    set(fn)   fn(block) -> opacity 0-1 for every block; 1 puts the element's own inline opacity back
    release() every block as it was
  skip: elements (the moment's own layers) that are never a block.
*/
const BLOCK = 'h1,h2,h3,h4,h5,h6,p,li,dt,dd,blockquote,figcaption,label,a,button,img,picture,video,canvas,svg,table,figure,.frame,.dw,[data-moment-block]';
export function blocks(root, skip = []) {
  const list = [];
  const walk = el => {
    for (const c of el.children) {
      if (skip.includes(c) || /^(SCRIPT|STYLE|NOSCRIPT|TEMPLATE)$/.test(c.tagName)) continue;
      const own = [...c.childNodes].some(n => n.nodeType === 3 && n.textContent.trim());
      if (c.matches(BLOCK) || own) list.push({ el: c, was: c.style.opacity, l: 0, t: 0, r: 0, b: 0 });
      else walk(c);
    }
  };
  walk(root);
  const measure = () => {
    const rr = root.getBoundingClientRect();
    for (const b of list) {
      const r = b.el.getBoundingClientRect();
      b.l = r.left - rr.left; b.t = r.top - rr.top; b.r = r.right - rr.left; b.b = r.bottom - rr.top;
    }
  };
  measure();
  const set = fn => { for (const b of list) { const o = fn(b); b.el.style.opacity = o >= 0.999 ? b.was : (o <= 0 ? '0' : o.toFixed(3)); } };
  const release = () => { for (const b of list) b.el.style.opacity = b.was; };
  return { list, measure, set, release };
}
```

## assets/moments/moments.js

```js
/*
  moments.js - the loader for the page's one or two creative moments (taste-skill, references/moments.md).

    <section data-moment="pinned-chapters" data-moment-opts='{"hold": 2.4}'>…</section>

    in system.js:   const m = await import(new URL('moments/moments.js', HERE).href); await m.ready;
    on its own:     <script type="module" src="assets/moments/moments.js"></script>

  It finds every [data-moment], links <id>/moment.css, imports <id>/moment.js and calls its default
  export mount(root, opts, lib) -> { destroy }. The page keeps scrolling natively; every moment shows
  its finished state under prefers-reduced-motion, without script, and when its module fails.

  Mounting is idempotent: an element mounts once however many times mount() or mountAll() is called
  (the mount promise is kept on the element as __momentP, before any await), mountAll(document) is
  one cached promise, and `ready` is that promise: it resolves with the mounted elements. Importing
  this module starts mountAll(document) on its own unless window.__momentsManual is set first.

  Marks on <html>: .mo-js as soon as this runs (a moment's CSS may hide what it will reveal only under
  .mo-js, and .mo-safe arrives 2.5 s later as a fallback that shows everything); data-moments-ready="n"
  when every moment is mounted (shoot scripts wait for it); data-moments-over="n" when a page holds
  more than two, and data-moments-repeat when an id is used twice (both are faults: one or two a page,
  an id once). A moment whose module fails gets data-moment-failed and the class mo-safe on its own
  element (its CSS shows the finished state at once). The 2.5 s timer and .mo-js can also be set
  earlier by one line in <head>, with modulepreload links for this file, lib.js and the dealt moment:
    <script>document.documentElement.classList.add('mo-js');setTimeout(()=>document.documentElement.classList.add('mo-safe'),2500)</script>
    <link rel="modulepreload" href="assets/moments/moments.js"><link rel="modulepreload" href="assets/moments/lib.js">
*/
import * as lib from './lib.js';

const BASE = new URL('./', import.meta.url);
const html = document.documentElement;
const linked = new Map();
function css(href) {
  if (!linked.has(href)) {
    const done = [...document.querySelectorAll('link[rel="stylesheet"]')].some(l => l.href === href);
    linked.set(href, done ? Promise.resolve() : new Promise(res => {
      const l = document.createElement('link');
      l.rel = 'stylesheet';
      l.href = href;
      l.onload = l.onerror = () => res();
      document.head.appendChild(l);
    }));
  }
  return linked.get(href);
}

export function mount(el) {
  if (el.__momentP) return el.__momentP;
  const id = el.dataset.moment;
  if (!id) return Promise.resolve(null);
  el.__momentP = (async () => {
    let opts = {};
    try { opts = JSON.parse(el.dataset.momentOpts || '{}'); } catch { console.warn(`moments: bad data-moment-opts on ${id}`); }
    const why = lib.stillWhy(), card = lib.inCard(el);
    if (card) { el.dataset.momentStill = 'card'; html.dataset.momentsCard = [html.dataset.momentsCard, id].filter(Boolean).join(' '); console.warn(`moments: ${id} sits in a card; it stays still`); }
    else if (why) el.dataset.momentStill = why;
    const mlib = { ...lib, reduced: () => card || lib.still() };
    const [, mod] = await Promise.all([css(new URL(`${id}/moment.css`, BASE).href), import(new URL(`${id}/moment.js`, BASE).href)]);
    const api = mod.default(el, opts, mlib) || {};
    el.__moment = api;
    el.dataset.momentMounted = '1';
    return api;
  })();
  return el.__momentP;
}

let docP = null, resolveReady;
export const ready = new Promise(r => { resolveReady = r; });

export function mountAll(root = document) {
  if (root === document && docP) return docP;
  const p = (async () => {
    html.classList.add('mo-js');
    const els = [...root.querySelectorAll('[data-moment]')].filter(el => !el.parentElement?.closest('[data-moment]'));
    const ids = els.map(el => el.dataset.moment);
    const repeat = ids.filter((id, i) => ids.indexOf(id) !== i);
    if (repeat.length) { html.dataset.momentsRepeat = [...new Set(repeat)].join(' '); console.warn(`moments: an id twice on one page: ${repeat.join(', ')}`); }
    if (ids.length > 2) { html.dataset.momentsOver = String(ids.length); console.warn(`moments: ${ids.length} moments on one page (one or two)`); }
    await Promise.all(els.map(el => mount(el).catch(e => {
      if (!el.dataset.momentFailed) console.warn(`moments: ${el.dataset.moment} failed: ${e.message}`);
      el.dataset.momentFailed = e.message;
      el.classList.add('mo-safe');
      el.classList.remove('is-live');   // the hide rules are written under .is-live and ignore .mo-safe: a failed moment shows as written
      el.classList.add('is-done');
    })));
    if (root === document) html.dataset.momentsReady = String(els.length);
    return els;
  })();
  if (root === document) { docP = p; p.then(resolveReady, () => resolveReady([])); }
  return p;
}

export function unmountAll(root = document) {
  root.querySelectorAll('[data-moment-mounted]').forEach(el => { try { el.__moment?.destroy?.(); } catch { /* gone */ } delete el.__moment; delete el.__momentP; el.removeAttribute('data-moment-mounted'); });
  if (root === document) docP = null;
}

html.classList.add('mo-js');
setTimeout(() => html.classList.add('mo-safe'), 2500);
const start = () => { if (!window.__momentsManual) mountAll(document); };
if (document.readyState === 'loading') document.addEventListener('DOMContentLoaded', start, { once: true });
else start();

export default { mount, mountAll, unmountAll, ready, lib };
```

## assets/moments/pinned-chapters/meta.json

```json
{
  "id": "pinned-chapters",
  "name": "Pinned chapters",
  "kind": "scroll",
  "fits": [
    "section"
  ],
  "trigger": "scroll",
  "cost": "low",
  "webgl": 0,
  "pages": [
    "tech",
    "gaming",
    "software"
  ],
  "look": "A feature section holds on screen for two screens of scroll while its three steps open one at a time at the left and one frame at the right changes to each step's picture with a short crossfade.",
  "reduced": "Nothing is pinned: from 640px each step is a static row, its text at the left and its visual at the right (at most 30svh tall); on a phone each step stands with its 16:9 visual under it. Two to four steps fit two screens at 1440x900, 1024x768, 820x1180 and 390x844.",
  "phone": "Nothing is pinned: each step stands with its title, two lines and its visual under it.",
  "maxPerPage": 1,
  "weight": 1,
  "needs": {
    "words": "2 to 4 steps, a title and two lines each",
    "pictures": "one picture or screen per step"
  },
  "avoidWith": [
    "closing-lift",
    "line-carries-colour"
  ],
  "states": {
    "start": {
      "p": 0.08,
      "wait": 900
    },
    "middle": {
      "p": 0.5,
      "wait": 900
    },
    "end": {
      "p": 0.92,
      "wait": 900
    }
  }
}
```

## assets/moments/pinned-chapters/moment.css

```css
/* pinned-chapters: at rest (phones, tablets, a short screen, reduced motion, no script, a failed
   module) each step is a title, two lines and its visual, and the section is its --h screens (two) or
   grows past them, never over the next section. From 640px that rest is a row per step: the text at the
   left, the visual at the right, at most 30svh tall; a phone stacks them with a 16:9 visual, so two to
   four steps fit two screens at 390x844. Live (.is-live, from 1024px on a
   screen 640px or taller) the section is (hold + 1) screens tall, its stage sticks for the hold,
   the steps open one at a time and the visuals share one frame. */
[data-moment="pinned-chapters"] { --mo-hold: 1; }
main [data-moment="pinned-chapters"][data-moment] > .mo-pc__grid { display: grid; gap: var(--s-7); align-items: center; align-content: start; height: auto; }
[data-moment="pinned-chapters"] .mo-pc__text { display: grid; gap: var(--s-6); align-content: start; }
[data-moment="pinned-chapters"] .mo-pc__steps { display: grid; gap: var(--s-5); }
[data-moment="pinned-chapters"] .mo-pc__step { display: grid; gap: var(--s-2); padding-left: var(--s-5); border-left: 2px solid var(--line); }
[data-moment="pinned-chapters"] .mo-pc__step h3 { font-family: var(--font-text); font-weight: 500; font-size: var(--t-lead); line-height: 1.3; letter-spacing: -0.01em; color: var(--ink); }
[data-moment="pinned-chapters"] .mo-pc__step p { color: var(--ink); max-width: 46ch; }
[data-moment="pinned-chapters"] .mo-pc__step p > .mo-pc__p-in { display: block; }
[data-moment="pinned-chapters"] .mo-pc__visual { margin: var(--s-4) 0 0; border-radius: var(--r-md); overflow: hidden; }
[data-moment="pinned-chapters"] .mo-pc__visual > img { width: 100%; aspect-ratio: 4 / 3; object-fit: cover; }
[data-moment="pinned-chapters"]:not(.is-live) .mo-pc__visuals { display: none; }

/* the static rows from 640px: the step's text left, its visual right, hairlines between the steps */
@media (min-width: 640px) {
  [data-moment="pinned-chapters"]:not(.is-live) .mo-pc__steps { gap: var(--s-6); }
  [data-moment="pinned-chapters"]:not(.is-live) .mo-pc__step {
    grid-template-columns: 5fr 6fr; column-gap: var(--s-8); row-gap: var(--s-2); align-items: center;
    border-left: 0; border-top: 1px solid var(--line); padding: var(--s-6) 0 0; }
  [data-moment="pinned-chapters"]:not(.is-live) .mo-pc__step h3 { grid-column: 1; align-self: end; }
  [data-moment="pinned-chapters"]:not(.is-live) .mo-pc__step p { grid-column: 1; align-self: start; }
  [data-moment="pinned-chapters"]:not(.is-live) .mo-pc__step .mo-pc__visual { grid-column: 2; grid-row: 1 / span 2; margin: 0; }
  [data-moment="pinned-chapters"]:not(.is-live) .mo-pc__visual > img { max-height: 30svh; }
}

/* a phone: shorter visuals, and a section that grows past its screens rather than run over the next one */
@media (max-width: 639px) {
  [data-moment="pinned-chapters"]:not(.is-live) .mo-pc__visual > img { aspect-ratio: 16 / 9; max-height: 26svh; }
  main > [data-moment="pinned-chapters"]:not(.is-live) { height: auto; min-height: calc(var(--hm, var(--h, 1)) * 100lvh); }
}

/* live */
[data-moment="pinned-chapters"].is-live { height: calc((var(--mo-hold) + 1) * 100svh); padding: 0; }
main [data-moment="pinned-chapters"][data-moment].is-live > .mo-pc__grid { position: sticky; top: 0; height: 100svh; box-sizing: border-box; padding-top: var(--nav, var(--nav-h, 72px)); grid-template-columns: 5fr 6fr; gap: var(--s-8); align-items: center; align-content: center; overflow: clip; }
[data-moment="pinned-chapters"].is-live .mo-pc__step { border-left-color: var(--line); transition: border-color 300ms ease; }
[data-moment="pinned-chapters"].is-live .mo-pc__step h3 { color: var(--muted); transition: color 300ms ease; }
/* a closed step's lines collapse to nothing: the grid row goes to 0fr and the inner span (min-height 0,
   overflow hidden) lets it; the text sits in that span, never as an anonymous item with its own height */
[data-moment="pinned-chapters"].is-live .mo-pc__step p { display: grid; grid-template-rows: 0fr; opacity: 0; transition: grid-template-rows 420ms cubic-bezier(.2, .7, .2, 1), opacity 300ms ease; }
[data-moment="pinned-chapters"].is-live .mo-pc__step p > .mo-pc__p-in { min-height: 0; overflow: hidden; }
[data-moment="pinned-chapters"].is-live .mo-pc__step.is-on { border-left-color: var(--accent); }
[data-moment="pinned-chapters"].is-live .mo-pc__step.is-on h3 { color: var(--ink); }
[data-moment="pinned-chapters"].is-live .mo-pc__step.is-on p { grid-template-rows: 1fr; opacity: 1; }
[data-moment="pinned-chapters"].is-live .mo-pc__visuals { position: relative; display: block; width: 100%; aspect-ratio: 4 / 3; max-height: calc(100svh - 128px); padding: 0; }
[data-moment="pinned-chapters"].is-live .mo-pc__visual { position: absolute; inset: 0; margin: 0; border-radius: 0; opacity: 0; transform: scale(1.03); transition: opacity 480ms ease, transform 900ms cubic-bezier(.16, 1, .3, 1); }
[data-moment="pinned-chapters"].is-live .mo-pc__visual > img { width: 100%; height: 100%; aspect-ratio: auto; }
[data-moment="pinned-chapters"].is-live .mo-pc__visual.is-on { opacity: 1; transform: none; z-index: 1; }
[data-moment="pinned-chapters"].is-live .mo-pc__visual.is-past { transform: scale(1); }
/* a drawn window inside a visual sits on the lit ground with a margin */
[data-moment="pinned-chapters"] .mo-pc__visual.mo-pc__visual--lit { display: flex; align-items: center; justify-content: center; padding: clamp(16px, 4vw, 40px); background: var(--ground-bg); aspect-ratio: 4 / 3; }
@media print {
  [data-moment="pinned-chapters"].is-live { height: auto; }
  main [data-moment="pinned-chapters"][data-moment].is-live > .mo-pc__grid { position: static; height: auto; display: block; overflow: visible; }   /* same selector as the live rule, so print wins: nothing clips */
  [data-moment="pinned-chapters"].is-live .mo-pc__step p { grid-template-rows: 1fr; opacity: 1; transition: none; }
  [data-moment="pinned-chapters"].is-live .mo-pc__step h3 { color: var(--ink); }
  [data-moment="pinned-chapters"].is-live .mo-pc__visuals { position: static; aspect-ratio: auto; max-height: none; display: grid; gap: var(--s-5); }
  [data-moment="pinned-chapters"].is-live .mo-pc__visual { position: static; opacity: 1; transform: none; aspect-ratio: 4 / 3; }
  [data-moment="pinned-chapters"].is-live .mo-pc__visual > img { height: auto; aspect-ratio: 4 / 3; }
}
@media (prefers-reduced-motion: reduce) {
  [data-moment="pinned-chapters"].is-live { height: auto; }
  main [data-moment="pinned-chapters"][data-moment].is-live > .mo-pc__grid { position: static; height: auto; }
  [data-moment="pinned-chapters"] .mo-pc__step p, [data-moment="pinned-chapters"] .mo-pc__visual { transition: none !important; }
}
```

## assets/moments/pinned-chapters/moment.js

```js
/*
  pinned-chapters - one section holds on screen while the visitor scrolls through its three (2 to 4)
  steps: the step in hand opens its two lines at the left, the one visual at the right changes to that
  step's picture or screen with a short crossfade and settle. The page scrolls natively (the stage is
  position: sticky); nothing else on the page moves. From 1024px on a screen at least 640px tall;
  otherwise, under reduced motion and without script the section is static: from 640px a row per
  step (text left, visual right, at most 30svh tall), below that every step stands with its visual
  under it (16:9).

    <section class="section" data-moment="pinned-chapters">
      <div class="container mo-pc__grid">
        <div class="mo-pc__text">
          <h2>How editors work with it</h2>
          <ol class="mo-pc__steps">
            <li class="mo-pc__step">
              <h3>Step title</h3><p>Two lines.</p>
              <figure class="mo-pc__visual"><img …></figure>          its picture, or a drawn window
            </li>
            …
          </ol>
        </div>
        <div class="mo-pc__visuals frame"></div>                     the shared frame, empty in the HTML
      </div>
    </section>

  The stage holds for one screen of scroll, so the live section is exactly two screens (layout-v4.md);
  with 2 to 4 steps each step gets a quarter to a half of a screen of scroll.
*/
export default function (root, opts, lib) {
  const steps = [...root.querySelectorAll('.mo-pc__step')];
  const frame = root.querySelector('.mo-pc__visuals');
  const figures = steps.map(s => s.querySelector('.mo-pc__visual'));
  if (steps.length < 2 || !frame || figures.some(f => !f)) return {};
  const homes = figures.map(f => f.parentElement);
  // the section stays exactly two screens (layout-v4.md: 1H or 2H), so the stage holds for one screen of scroll
  const hold = 1;
  root.style.setProperty('--mo-hold', String(hold));
  // each step's lines in a span of their own, so a closed step can collapse to nothing (moment.css)
  steps.forEach(s => {
    const p = s.querySelector('p');
    if (!p || p.querySelector('.mo-pc__p-in')) return;
    const w = document.createElement('span');
    w.className = 'mo-pc__p-in';
    while (p.firstChild) w.append(p.firstChild);
    p.append(w);
  });
  const grid = root.querySelector('.mo-pc__grid');
  let live = false, scrubber = null, k = -1;
  const wide = () => matchMedia('(min-width: 1024px)').matches;
  const fits = () => !lib.reduced() && wide() && innerHeight >= 640;
  const show = i => {
    if (i === k) return;
    k = i;
    steps.forEach((s, n) => s.classList.toggle('is-on', n === i));
    figures.forEach((f, n) => { f.classList.toggle('is-on', n === i); f.classList.toggle('is-past', n < i); });
  };
  const go = () => {
    if (live) return;
    live = true;
    // the scrub first: if it throws (no IntersectionObserver), nothing has been moved or hidden yet
    scrubber = lib.scrub(root, p => show(Math.min(steps.length - 1, Math.floor(p * steps.length))));
    root.classList.add('is-live');
    grid?.setAttribute('data-edge-free', '');   // live the grid is one screen of a two-screen section: the layout check skips it (the rest rows are checked)
    figures.forEach(f => frame.append(f));
    show(0);   // step 1 open before the first scroll tick: a scrub that never ticks still shows a step and its visual
  };
  const rest = () => {
    if (!live) return;
    live = false;
    root.classList.remove('is-live');
    grid?.removeAttribute('data-edge-free');
    scrubber?.destroy(); scrubber = null;
    figures.forEach((f, n) => { homes[n].append(f); f.classList.remove('is-on', 'is-past'); });
    steps.forEach(s => s.classList.remove('is-on'));
    k = -1;
  };
  const sync = () => { const f = fits(); if (f) go(); else rest(); root.classList.toggle('is-static', !f && wide()); };
  sync();
  let t = 0;
  const onResize = () => { clearTimeout(t); t = setTimeout(sync, 120); };
  addEventListener('resize', onResize);
  const rq = matchMedia('(prefers-reduced-motion: reduce)');
  rq.addEventListener('change', sync);   // a preference changed after load: go static (or live) at once
  return { destroy() { removeEventListener('resize', onResize); rq.removeEventListener('change', sync); rest(); root.classList.remove('is-static'); } };
}
```

## assets/motion/motion.css

```css
/*
  taste-skill motion.css - the micro-interactions a company site has (figma.com, x.ai): one small
  opt-in layer over system.css and ui/cards.css. Link it after them (data-motion on <html> is optional);
  load motion.js once (it adds the tab indicator, the number tick and the card loops).
  The layer is ON as soon as this file is linked: no attribute is needed (a hover shot of a linked card or photo
  shows the 2px lift and the 1.03 zoom). <html data-motion="off"> switches the whole layer off; data-motion="live"
  is the number tick in a headless shot (motion.js).
  references/motion.md says what moves and how fast. Everything here answers the pointer, the
  keyboard or a number coming into view once; nothing moves because the page scrolled.

  Hover rules sit under (hover: hover) and (pointer: fine): a phone never keeps a stuck hover.
  Under prefers-reduced-motion colours still change; nothing moves or scales.
*/

:root {
  --m-in: 140ms;                              /* hover in: buttons, links, nav, tabs */
  --m-out: 200ms;                             /* hover out, panel close */
  --m-press: 90ms;                            /* a press */
  --m-panel: 200ms;                           /* a menu or sheet opening */
  --m-slide: 240ms;                           /* an indicator travelling between two tabs or links */
  --m-image: 600ms;                           /* a photograph scaling inside its mask */
  --m-ease: cubic-bezier(.2, .8, .2, 1);      /* ease-out: fast start, soft landing */
  --m-ease-io: cubic-bezier(.65, 0, .35, 1);  /* the indicator: leaves and lands softly */
  --m-cycle: 3.2s;                            /* one cycle of a card's loop (2.5-4 s) */
  --m-lift: -2px;
}

/* ---------------------------------------------------------------- buttons */
html:not([data-motion="off"]) .btn {
  position: relative;
  transition: background-color var(--m-out) var(--m-ease), color var(--m-out) var(--m-ease),
              border-color var(--m-out) var(--m-ease), box-shadow var(--m-out) var(--m-ease),
              transform var(--m-press) var(--m-ease);
}
/* the tone moves toward the ground: a black button greys a little on white, a white one on black */
html:not([data-motion="off"]) .btn--primary:hover { background: color-mix(in oklab, var(--action) 84%, var(--bg)); }
html:not([data-motion="off"]) .btn--secondary:hover {
  background: color-mix(in oklab, var(--bg) 92%, var(--ink));
  border-color: color-mix(in oklab, var(--ink) 32%, transparent);
}
html:not([data-motion="off"]) .btn:hover { transition-duration: var(--m-in), var(--m-in), var(--m-in), var(--m-in), var(--m-press); }
html:not([data-motion="off"]) .btn:active { transform: translateY(1px) scale(.985); }
/* the step: a button rises 1px on hover (2px on the large size) and presses 1px down with a 1.5% shrink on
   click; pointer only. The text-link secondary (arrow and block looks) keeps its underline and does not step.
   Reduced motion keeps the colour change only (see the end of this file). */
@media (hover: hover) and (pointer: fine) {
  html:not([data-motion="off"]) .btn:hover { transform: translateY(-1px); }
  html:not([data-motion="off"]) .btn--xl:hover { transform: translateY(-2px); }
  html:not([data-motion="off"]):is([data-buttons="arrow"], [data-buttons="block"]) .btn--secondary:hover { transform: none; }
  html:not([data-motion="off"]) .btn:active { transform: translateY(1px) scale(.985); }
  html:not([data-motion="off"]):is([data-buttons="arrow"], [data-buttons="block"]) .btn--secondary:active { transform: scale(.985); }
}

/* the arrow: the text arrow (.btn--arrow, .link) nudges 3px; the drawn arrow (.m-arrow) grows its
   stem and moves its head 3px, the way a good product site's "Learn more" does */
html:not([data-motion="off"]) :is(.btn--arrow, .link)::after { transition: transform var(--m-out) var(--m-ease); }
html:not([data-motion="off"]) :is(.btn--arrow, .link):hover::after { transform: translateX(3px); transition-duration: var(--m-in); }
.m-arrow { width: 12px; height: 12px; flex: none; overflow: visible; fill: none; stroke: currentColor; stroke-width: 1.6;
           stroke-linecap: round; stroke-linejoin: round; }
.m-arrow__stem { opacity: 0; transform: scaleX(.2); transform-origin: 1px 50%; transform-box: fill-box;
                 transition: opacity var(--m-out) var(--m-ease), transform var(--m-out) var(--m-ease); }
.m-arrow__head { transition: transform var(--m-out) var(--m-ease); }
:is(a, button):hover > .m-arrow .m-arrow__stem,
:is(a, button):focus-visible > .m-arrow .m-arrow__stem { opacity: 1; transform: none; transition-duration: var(--m-in); }
:is(a, button):hover > .m-arrow .m-arrow__head,
:is(a, button):focus-visible > .m-arrow .m-arrow__head { transform: translateX(3px); transition-duration: var(--m-in); }

/* a pill that is a link (the line above a hero headline) */
html:not([data-motion="off"]) a.pill { transition: background-color var(--m-out) var(--m-ease), border-color var(--m-out) var(--m-ease); }
html:not([data-motion="off"]) a.pill:hover { background: color-mix(in oklab, var(--card) 90%, var(--ink)); border-color: var(--line-strong); transition-duration: var(--m-in); }

/* ---------------------------------------------------------------- links: an underline that draws */
/* .link and links in running text: a rest line, and the ink line draws left to right on hover and
   leaves to the right. A link in a sentence rests on its own colour at 45% (about 3.3:1 on its
   ground), so it reads as a link even when it is the same colour as the text (WCAG 1.4.1); .link
   and .m-link carry an arrow or sit alone and rest on the faint hairline */
html:not([data-motion="off"]) :is(.link, .m-link, p a:not([class]), .footer ul a, .footer .link) {
  text-decoration: none;
  background-image: linear-gradient(currentColor, currentColor), linear-gradient(var(--m-rest, var(--line-strong)), var(--m-rest, var(--line-strong)));
  background-repeat: no-repeat; background-size: 0% 1px, 100% 1px;
  background-position: right 0 bottom 0, 0 100%;
  transition: background-size var(--m-out) var(--m-ease), color var(--m-out) var(--m-ease);
  padding-bottom: 2px;
}
/* set on its own rule: the shared rule above is (0,3,1) through :is(.footer .link) and would win */
html:not([data-motion="off"]) p a:not([class]) { --m-rest: color-mix(in oklab, currentColor 45%, transparent); }
html:not([data-motion="off"]) :is(.footer ul a, .m-link--bare):not(:hover, :focus-visible) { background-size: 0% 1px, 0% 1px; }
html:not([data-motion="off"]) :is(.link, .m-link, p a:not([class]), .footer ul a, .footer .link):is(:hover, :focus-visible) {
  background-size: 100% 1px, 100% 1px; background-position: left 0 bottom 0, 0 100%; transition-duration: var(--m-in);
}
/* the link's arrow is not underlined: the line runs under the words only */
html:not([data-motion="off"]) .link { background-size: 0% 1px, calc(100% - 1em - var(--s-2)) 1px; }
html:not([data-motion="off"]) .link:is(:hover, :focus-visible) { background-size: calc(100% - 1em - var(--s-2)) 1px, calc(100% - 1em - var(--s-2)) 1px; }
html:not([data-motion="off"]) .link, html:not([data-motion="off"]) .link:is(:hover, :focus-visible) { background-position: 0 100%, 0 100%; }
html:not([data-motion="off"]) .footer ul a:hover { color: var(--ink); }

/* ---------------------------------------------------------------- navigation */
/* the nav words take no pill (ST-08, ST-11): system.css draws the underline under a word from the left on hover and focus, and
   takes it away to the right; the page you are on keeps its still bar (system.css draws it, at the same inset) */
@media (min-width: 1024px) {
  /* the full-width panel: fades and settles 6px in 200ms, its columns follow 30ms apart; it closes
     in 120ms. While one menu is open, the next word swaps the panel at once (motion.js .m-menu-open) */
  html:not([data-motion="off"]) .nav__menu {
    transform: translateY(-6px);
    transition: opacity 120ms var(--m-ease), transform 120ms var(--m-ease), visibility 0s linear 120ms;
  }
  html:not([data-motion="off"]) .nav__item:is(:hover, :focus-within) > .nav__menu {
    transition: opacity var(--m-panel) var(--m-ease) 150ms, transform var(--m-panel) var(--m-ease) 150ms, visibility 0s linear 150ms;
  }
  /* .m-menu-open comes on when the first panel starts to show (150ms): from then the next word swaps
     the panel at once. Duration and delay go to 0, the properties stay listed, so the panel that is
     still fading in finishes its own 200ms instead of snapping */
  html:not([data-motion="off"]) .nav.m-menu-open .nav__menu,
  html:not([data-motion="off"]) .nav.m-menu-open .nav__item:is(:hover, :focus-within) > .nav__menu { transition-duration: 0s; transition-delay: 0s; }
  html:not([data-motion="off"]) .nav__menu .grid > * { opacity: 0; transform: translateY(4px); transition: opacity 100ms var(--m-ease), transform 100ms var(--m-ease); }
  html:not([data-motion="off"]) .nav__item:is(:hover, :focus-within) > .nav__menu .grid > * {
    opacity: 1; transform: none; transition: opacity 260ms var(--m-ease), transform 260ms var(--m-ease);
    transition-delay: calc(170ms + var(--m-i, 0) * 30ms);
  }
  html:not([data-motion="off"]) .nav.m-menu-open .nav__item:is(:hover, :focus-within) > .nav__menu .grid > * { transition-delay: calc(var(--m-i, 0) * 30ms); }
  html:not([data-motion="off"]) .nav__menu .grid > :nth-child(2) { --m-i: 1; }
  html:not([data-motion="off"]) .nav__menu .grid > :nth-child(3) { --m-i: 2; }
  html:not([data-motion="off"]) .nav__menu .grid > :nth-child(4) { --m-i: 3; }
  html:not([data-motion="off"]) .nav__menu .grid > :nth-child(n + 5) { --m-i: 4; }
  html:not([data-motion="off"]) .menu-col a { transition: color var(--m-out) var(--m-ease); }
  html:not([data-motion="off"]) .menu-col a:hover { transition-duration: var(--m-in); }
  html:not([data-motion="off"]) .menu-card .panel { transition: border-color var(--m-out) var(--m-ease); }
}
/* the phone sheet slides in 8px and fades (system.js opens it) */
@media (max-width: 1023px) {
  html:not([data-motion="off"]) .nav.is-open .nav__links { transition: opacity var(--m-panel) var(--m-ease), transform var(--m-panel) var(--m-ease); }
  @starting-style { html:not([data-motion="off"]) .nav.is-open .nav__links { opacity: 0; transform: translateY(-8px); } }
}

/* ---------------------------------------------------------------- tabs: the indicator slides */
html:not([data-motion="off"]) .tabs.m-has-ind { position: relative; isolation: isolate; }
html:not([data-motion="off"]) .tabs.m-has-ind .tab { position: relative; z-index: 1; }
html:not([data-motion="off"]) .tabs.m-has-ind .tab[aria-selected="true"] { background: transparent; border-color: transparent; }
.m-tab-ind {
  position: absolute; left: 0; top: 0; z-index: 0; width: var(--w, 0px); height: var(--h, 0px);
  border-radius: var(--r-btn); background: var(--ink); pointer-events: none;
  transform: translate(var(--x, 0px), var(--y, 0px));
  transition: transform var(--m-slide) var(--m-ease-io), width var(--m-slide) var(--m-ease-io), height var(--m-slide) var(--m-ease-io);
}
.m-tab-ind.is-jump { transition: none; }
html:not([data-motion="off"]) .tab { transition: color var(--m-slide) var(--m-ease), border-color var(--m-out) var(--m-ease), background-color var(--m-out) var(--m-ease); }

/* ---------------------------------------------------------------- focus: the ring settles in */
/* only on the plain controls, never on a part of a loop, and not on an element that runs its own
   animation (mark it .m-anim): an animation set here would replace that element's own */
html:not([data-motion="off"]) :is(a, button, input, select, textarea, summary, [tabindex]):focus-visible:not([data-m-loop] *, .m-anim, .loop-pause) {
  animation: m-focus var(--m-out) var(--m-ease);
}
@keyframes m-focus { from { outline-offset: 6px; outline-color: transparent; } }

/* ---------------------------------------------------------------- cards: a 2px lift */
/* the card rises 2px, its border and shadow deepen; a card without its own box (.card--open) lifts
   its panel. Never tilts, flips or follows the pointer */
html:not([data-motion="off"]) .card:not(.card--open, .card--plan) {
  transition: transform var(--m-out) var(--m-ease), border-color var(--m-out) var(--m-ease), box-shadow var(--m-out) var(--m-ease);
}
html:not([data-motion="off"]) .card--open > .panel { transition: transform var(--m-out) var(--m-ease), box-shadow var(--m-out) var(--m-ease); }
@media (hover: hover) and (pointer: fine) {
  html:not([data-motion="off"]) .card:not(.card--open, .card--plan, .m-nolift):hover {
    transform: translateY(var(--m-lift));
    border-color: color-mix(in oklab, var(--ink) 16%, transparent);
    /* the shadow is deeper on a dark card (its lightness l): about .2 on white, .64 on near-black */
    box-shadow: inset 0 1px 0 var(--c-hi, transparent), 0 1px 2px rgb(0 0 0 / .06),
                0 14px 32px -18px oklch(from var(--card, #fff) 0 0 0 / calc(0.75 - l * 0.55));
    transition-duration: 180ms;
  }
  html:not([data-motion="off"]) .card--open:not(.m-nolift):hover > .panel {
    transform: translateY(var(--m-lift)); box-shadow: 0 14px 32px -18px oklch(from var(--bg, #fff) 0 0 0 / calc(0.75 - l * 0.55)); transition-duration: 180ms;
  }
}

/* ---------------------------------------------------------------- photographs: 1.03 inside the mask */
/* a photograph in a link or a linked card grows 3% inside its own mask (the frame, the tile, a .m-zoom wrapper);
   the mask clips it, so the box never moves. .m-zoom is the hook for any other photograph: wrap it. */
.m-zoom { overflow: clip; }
html:not([data-motion="off"]) :is(.frame__img, .tile__media > img, .m-zoom img, .m-zoom > video) {
  transition: transform var(--m-image) var(--m-ease);
}
@media (hover: hover) and (pointer: fine) {
  html:not([data-motion="off"]) :is(a, .card:has(a), .m-zoom):hover :is(.frame__img, .tile__media > img),
  html:not([data-motion="off"]) .m-zoom:hover :is(img, video) { transform: scale(1.03); }
}

/* ---------------------------------------------------------------- inputs, switches, checks */
html:not([data-motion="off"]) .field :is(input, select, textarea), .m-input {
  transition: border-color var(--m-out) var(--m-ease), box-shadow var(--m-out) var(--m-ease), background-color var(--m-out) var(--m-ease);
}
html:not([data-motion="off"]) .field :is(input, select, textarea):hover, .m-input:hover { border-color: color-mix(in oklab, var(--ink) 48%, transparent); }
html:not([data-motion="off"]) .field :is(input, select, textarea):focus, .m-input:focus {
  outline: none; border-color: var(--action);
  box-shadow: 0 0 0 3px color-mix(in oklab, var(--action) 20%, transparent); transition-duration: var(--m-in);
}
/* a switch: <button class="m-switch" role="switch" aria-checked="false" aria-label="..."></button>
   (motion.js flips aria-checked on click unless the page handles it). The knob slides in 180ms and
   stretches 4px while pressed. The off track is ink at 48% (3.3:1 or more against the knob and the
   ground, the 3:1 a control needs) */
.m-switch {
  position: relative; flex: none; width: 40px; height: 24px; padding: 0; border: 0; border-radius: 999px; cursor: pointer;
  background: color-mix(in oklab, var(--ink) 48%, var(--bg));
  transition: background-color 180ms var(--m-ease);
}
.m-switch::before {
  content: ""; position: absolute; left: 3px; top: 3px; width: 18px; height: 18px; border-radius: 999px;
  background: var(--bg); box-shadow: 0 1px 3px rgb(0 0 0 / .25);
  transition: transform 180ms var(--m-ease), width var(--m-press) var(--m-ease);
}
.m-switch[aria-checked="true"] { background: var(--action); }
.m-switch[aria-checked="true"]::before { transform: translateX(16px); background: var(--action-ink); }
.m-switch:active::before { width: 22px; }
.m-switch[aria-checked="true"]:active::before { transform: translateX(12px); }
.m-switch:hover:not([aria-checked="true"]) { background: color-mix(in oklab, var(--ink) 58%, var(--bg)); }
/* a check box: <input type="checkbox" class="m-check">, the tick draws in 160ms */
.m-check {
  appearance: none; -webkit-appearance: none; flex: none; margin: 0; width: 20px; height: 20px; border-radius: 6px; cursor: pointer;
  border: 1px solid color-mix(in oklab, var(--ink) 48%, var(--bg)); background: var(--bg) no-repeat center / 14px 14px;
  display: inline-grid; place-items: center;
  transition: background-color var(--m-in) var(--m-ease), border-color var(--m-in) var(--m-ease);
}
.m-check::before {
  content: ""; width: 10px; height: 6px; margin-top: -2px; border: solid var(--action-ink); border-width: 0 0 2px 2px;
  transform: rotate(-45deg) scale(0); transform-origin: 50% 50%; transition: transform 160ms var(--m-ease);
}
.m-check:hover:not(:checked) { border-color: color-mix(in oklab, var(--ink) 64%, var(--bg)); }
.m-check:checked { background-color: var(--action); border-color: var(--action); }
.m-check:checked::before { transform: rotate(-45deg) scale(1); }

/* ---------------------------------------------------------------- FAQ
   Every answer stays open (ST-47, system.css): nothing here opens, closes or animates its height. */

/* ---------------------------------------------------------------- numbers that tick once */
[data-m-count] { font-variant-numeric: tabular-nums; }

/* ---------------------------------------------------------------- loops inside a card's panel */
/* a CSS loop the page writes inside [data-m-loop] runs one cycle in --m-cycle (2.5-4 s) and repeats
   while on screen; motion.js pauses it off screen and restarts it on hover (motion.md) */
[data-m-loop] { --m-cycle: 3.2s; --m-still: 0.6; --m-safe: 48px; }
/* the pause button (a 32px disc 12px in from the bottom-right corner) gets a clear band: content in
   the flow ends --m-safe (48px) above the bottom edge, and a fading stage dissolves above that band,
   never under the button. Size the panel so the loop's subject sits above the fade */
[data-m-loop]:not(:has(> .stage)) { padding-bottom: var(--m-safe); }
[data-m-loop] > .stage:is(.stage--fade-b, .stage--fade-br) { bottom: var(--m-safe); }
/* on a phone (as ui.js does) and under reduced motion a loop holds its finished picture: the frame at
   --m-still (0-1) of the cycle, set on the loop where the picture is complete */
@media (max-width: 639px) {
  [data-m-loop] * { animation-duration: var(--m-cycle) !important; animation-delay: calc(-1 * var(--m-still) * var(--m-cycle)) !important;
                    animation-play-state: paused !important; }
}
[data-m-loop].is-m-off *, [data-m-loop].is-m-paused * { animation-play-state: paused !important; }

/* ---------------------------------------------------------------- reduced motion */
@media (prefers-reduced-motion: reduce) {
  html:not([data-motion="off"]) .card:hover, html:not([data-motion="off"]) .card--open:hover > .panel,
  html:not([data-motion="off"]) :is(a, .card:has(a), .m-zoom):hover :is(.frame__img, .tile__media > img, img, video),
  html:not([data-motion="off"]) .btn:is(:hover, :active), html:not([data-motion="off"]) :is(.btn--arrow, .link):hover::after,
  :is(a, button):hover > .m-arrow :is(.m-arrow__head, .m-arrow__stem) { transform: none !important; }
  .m-arrow__stem { opacity: 1; transform: none; }
  html:not([data-motion="off"]) .nav__menu, html:not([data-motion="off"]) .nav__menu .grid > * { transform: none !important; }
  html:not([data-motion="off"]) :is(a, button, input, select, textarea, summary, [tabindex]):focus-visible { animation: none; }
  [data-m-loop] * { animation-duration: var(--m-cycle) !important; animation-iteration-count: infinite !important;
                    animation-delay: calc(-1 * var(--m-still) * var(--m-cycle)) !important; animation-play-state: paused !important; }
}
```

## assets/motion/motion.js

```js
/*
  taste-skill motion.js - the moving half of the micro-interaction layer (motion.css is the rest).
  Load it once, after system.js (motion.css needs no attribute; <html data-motion="off"> turns the layer off, JS too):
    <script type="module" src="assets/motion/motion.js"></script>
  It starts by itself. references/motion.md says what moves and how fast.

  - nav: .m-menu-open from the moment a menu starts to show (150ms of intent), so the next word swaps the panel at once
    instead of waiting 150ms again; it stays while the pointer is on the word or its panel.
  - tabs: every .tabs[role=tablist] gets one indicator that slides to the selected tab (system.js
    still selects; this follows aria-selected).
  - numbers: [data-m-count] on a stat ticks from 0 to its own text once, when 60% of it is on
    screen (1.2 s, ease-out); prefix, suffix, decimals and thousands separators are kept. The
    number's width is measured once the web fonts are in and again when it is first seen (a stat
    in a hidden tab), so nothing beside it moves; a stat the observer never reports shows its own
    text after 6 s on screen. A headless screenshot (HeadlessChrome) sees the final number unless
    <html data-motion="live">.
  - loops: [data-m-loop] marks a panel (or any box) whose CSS keyframe loops the page wrote itself;
    they pause off screen, restart when the pointer enters the card around them, and get the
    system's pause button (one per panel, shared with ui.js). ui.js restarts its own fragments.
  - switches: a click on .m-switch flips aria-checked unless a handler before it called
    preventDefault().
  Under prefers-reduced-motion: no indicator travel, no tick, no loops, no restart.
*/
const html = document.documentElement;
const reducedQ = matchMedia('(prefers-reduced-motion: reduce)');
const reduced = () => reducedQ.matches;
const shot = () => /HeadlessChrome/.test(navigator.userAgent) && html.dataset.motion !== 'live';
const jump = (el, on) => { el.classList.toggle('is-jump', on); };

/* ---------------------------------------------------------------- nav */
function nav() {
  const bar = document.querySelector('.nav');
  const links = bar?.querySelector('.nav__links');
  if (!links || bar.dataset.mNav) return;
  bar.dataset.mNav = '1';   // no pill under the words (ST-08, ST-11): system.css draws their underline

  // one menu open: the next word swaps the panel at once (figma.com); leaving the bar closes softly
  let t = 0;
  bar.querySelectorAll('.nav__item').forEach(item => {
    const has = !!item.querySelector('.nav__menu');
    item.addEventListener('pointerenter', () => {
      clearTimeout(t);
      // 150ms: the panel has started to show (motion.css lets it finish its own fade)
      if (has) t = setTimeout(() => bar.classList.add('m-menu-open'), 150);
      else bar.classList.remove('m-menu-open');
    });
  });
  bar.addEventListener('pointerleave', () => { clearTimeout(t); bar.classList.remove('m-menu-open'); });
}

/* ---------------------------------------------------------------- tabs */
function tabs() {
  document.querySelectorAll('.tabs[role="tablist"], [role="tablist"].m-tabs').forEach(list => {
    if (!list.querySelector('[role="tab"]') || list.querySelector(':scope > .m-tab-ind')) return;
    const ind = document.createElement('span');
    ind.className = 'm-tab-ind';
    ind.setAttribute('aria-hidden', 'true');
    list.prepend(ind);
    list.classList.add('m-has-ind');
    let instant = true;
    const place = () => {
      const sel = list.querySelector('[role="tab"][aria-selected="true"]');
      if (!sel) { ind.style.opacity = '0'; return; }
      ind.style.opacity = '';
      const now = instant || reduced();
      jump(ind, now);
      ind.style.setProperty('--x', `${sel.offsetLeft}px`);
      ind.style.setProperty('--y', `${sel.offsetTop}px`);
      ind.style.setProperty('--w', `${sel.offsetWidth}px`);
      ind.style.setProperty('--h', `${sel.offsetHeight}px`);
      if (now) { ind.getBoundingClientRect(); jump(ind, false); }
      instant = false;
    };
    place();
    new MutationObserver(place).observe(list, { subtree: true, attributes: true, attributeFilter: ['aria-selected'] });
    new ResizeObserver(() => { instant = true; place(); }).observe(list);
    document.fonts?.ready.then(() => { instant = true; place(); });
  });
}

/* ---------------------------------------------------------------- numbers */
// the first number in the element's text: "8,402", "4.2", "64%", "EUR 1.200", "12k"
// separators: . , and the no-break spaces (U+00A0, U+202F); a plain space ends the number ("12 4 h" is 12)
const NUM = /\d+(?:[.,\u00A0\u202F]\d+)*/;
function parse(text) {
  const m = text.match(NUM);
  if (!m) return null;
  const raw = m[0];
  const seps = raw.match(/[^\d]/g) || [];
  let dec = '', group = '', decimals = 0;
  const lastSep = raw.search(/[^\d](?=\d+$)/);
  if (lastSep >= 0) {
    const tail = raw.length - lastSep - 1;
    // a last separator before exactly three digits groups thousands; otherwise it is the decimal point
    if (tail === 3 && !(seps.length === 1 && raw[lastSep] === '.' && /^0/.test(raw))) group = raw[lastSep];
    else { dec = raw[lastSep]; decimals = tail; group = seps.length > 1 ? seps[0] : ''; }
  }
  const intPart = dec ? raw.slice(0, lastSep) : raw;
  const value = parseFloat(intPart.replace(/[^\d]/g, '') + (dec ? '.' + raw.slice(lastSep + 1) : ''));
  return { index: m.index, raw, value, dec, group, decimals };
}
function format(v, p) {
  let [i, d] = v.toFixed(p.decimals).split('.');
  if (p.group) i = i.replace(/\B(?=(\d{3})+(?!\d))/g, p.group);
  return d ? i + p.dec + d : i;
}
// the web fonts, or 3 s at most: a number measured in the fallback font comes out narrower
const fontsIn = () => Promise.race([document.fonts?.ready ?? Promise.resolve(), new Promise(r => setTimeout(r, 3000))]);
function counts() {
  const els = [...document.querySelectorAll('[data-m-count]')].filter(el => !el.__mCount);
  if (!els.length || reduced() || shot()) return;
  const io = new IntersectionObserver(es => es.forEach(e => { if (e.isIntersecting) e.target.__mCount?.hit(); }), { threshold: 0.6 });
  const ro = new ResizeObserver(es => es.forEach(e => e.target.__mCount?.arm()));
  const items = [];
  els.forEach(el => {
    const walk = document.createTreeWalker(el, NodeFilter.SHOW_TEXT);
    let node, p;
    while ((node = walk.nextNode()) && !(p = parse(node.data))) { /* next */ }
    if (!node || !p) return;
    // the number gets its own span, as wide as the final number and right-aligned, so nothing beside it
    // moves. Until it is measured it shows its own text, never a 0
    const after = node.splitText(p.index);
    after.splitText(p.raw.length);
    const span = document.createElement('span');
    span.className = 'm-count__n';
    span.style.display = 'inline-block';
    span.style.textAlign = 'right';
    span.textContent = p.raw;
    after.replaceWith(span);
    const ms = +el.dataset.mCount || 1200;
    let state = 'wait', timer = 0;          // wait (own text) -> armed (0, width held) -> run -> done
    const stop = () => { clearTimeout(timer); io.unobserve(el); ro.unobserve(el); };
    const onScreen = () => {
      const r = span.getBoundingClientRect();
      return r.width > 0 && r.bottom > 0 && r.top < innerHeight && r.right > 0 && r.left < innerWidth && document.visibilityState === 'visible';
    };
    // the safety net: a stat on screen for 6 s that the observer never reported shows its own text
    const watch = () => {
      clearTimeout(timer);
      timer = setTimeout(() => {
        if (state !== 'armed') return;
        if (onScreen()) { state = 'done'; stop(); span.textContent = p.raw; } else watch();
      }, 6000);
    };
    const ctl = {
      /** measure the final number in the page's font and hold its width; false while it has none (hidden) */
      arm() {
        if (state === 'run' || state === 'done') return false;
        span.textContent = p.raw;
        span.style.minWidth = '';
        const w = span.getBoundingClientRect().width;
        if (!w) { state = 'wait'; clearTimeout(timer); return false; }
        span.style.minWidth = `${w}px`;
        span.textContent = format(0, p);
        if (state !== 'armed') { state = 'armed'; watch(); }
        return true;
      },
      /** print: stop waiting and show the written number */
      show() { if (state === 'done') return; state = 'done'; stop(); span.textContent = p.raw; },
      /** first seen: measure again (a font or a tab that changed since), then tick once */
      hit() {
        if (state === 'run' || state === 'done' || !ctl.arm()) return;
        state = 'run';
        stop();
        const t0 = performance.now();
        const step = now => {
          const k = Math.min(1, (now - t0) / ms);
          span.textContent = k < 1 ? format(p.value * (1 - Math.pow(1 - k, 4)), p) : p.raw;
          if (k < 1) requestAnimationFrame(step); else state = 'done';
        };
        requestAnimationFrame(step);
      },
    };
    el.__mCount = ctl;
    items.push(el);
  });
  // measured only once the fonts are in; the resize observer measures again when a hidden stat shows
  fontsIn().then(() => items.forEach(el => { el.__mCount.arm(); ro.observe(el); io.observe(el); }));
  // a printed page never scrolls the stats into view: each prints its own number
  addEventListener('beforeprint', () => items.forEach(el => el.__mCount.show()));
}

/* ---------------------------------------------------------------- loops inside cards */
const ICON_PAUSE = '<svg viewBox="0 0 16 16" aria-hidden="true"><rect x="4.25" y="3.25" width="2.75" height="9.5" rx="0.9"/><rect x="9" y="3.25" width="2.75" height="9.5" rx="0.9"/></svg>';
const ICON_PLAY = '<svg viewBox="0 0 16 16" aria-hidden="true"><path d="M5.25 3.7v8.6c0 .62.68 1 1.2.66l6.5-4.3c.47-.31.47-1 0-1.32l-6.5-4.3c-.52-.34-1.2.04-1.2.66Z"/></svg>';
const boxOf = el => el.closest('[data-ui-pause-host], .panel__screen, .panel, .frame') || el;
const hostOf = el => el.closest('.card, .tile, .menu-card, a') || boxOf(el);

function restart(loop) {
  if (reduced() || loop.classList.contains('is-m-paused')) return;
  // the page's keyframe loops only: a hover transition in the panel (the pause button's) is no cycle.
  // ui.js fragments in the card restart by themselves
  const anims = loop.getAnimations({ subtree: true }).filter(a => typeof CSSAnimation === 'function' && a instanceof CSSAnimation);
  if (!anims.length) return;
  const dur = a => Number(a.effect?.getComputedTiming().duration) || 0;
  const main = anims.reduce((m, a) => (dur(a) > dur(m) ? a : m));
  const d = dur(main), t = Number(main.currentTime) || 0;
  // restart only when the cycle is past its first quarter, so a quick in-and-out never stutters
  if (!d || (t % d) / d > 0.25) anims.forEach(a => { a.currentTime = 0; });
}
function setPaused(box, paused) {
  box.querySelectorAll('[data-m-loop]').forEach(l => l.classList.toggle('is-m-paused', paused));
  if (box.matches('[data-m-loop]')) box.classList.toggle('is-m-paused', paused);
}
function pauseButton(loop) {
  const box = boxOf(loop);
  if (box.querySelector(':scope > .loop-pause')) return;   // ui.js or the page put one there: it is shared
  const btn = document.createElement('button');
  btn.type = 'button';
  btn.className = 'loop-pause m-own';
  const show = paused => {
    btn.innerHTML = paused ? ICON_PLAY : ICON_PAUSE;
    btn.setAttribute('aria-label', `${paused ? 'Play' : 'Pause'} animation`);
    btn.title = btn.getAttribute('aria-label');
    btn.classList.toggle('is-paused', paused);
  };
  show(false);
  if (getComputedStyle(box).position === 'static') box.style.position = 'relative';
  box.append(btn);
  btn.__show = show;
}
function loops() {
  const els = [...document.querySelectorAll('[data-m-loop]')].filter(el => !el.__mLoop);
  if (!els.length) return;
  if (reduced() || matchMedia('(max-width: 639px)').matches) return;   // motion.css holds them on their finished picture
  const io = new IntersectionObserver(es => es.forEach(e => e.target.classList.toggle('is-m-off', !e.isIntersecting)), { rootMargin: '48px' });
  els.forEach(el => {
    el.__mLoop = true;
    io.observe(el);
    pauseButton(el);
    hostOf(el).addEventListener('pointerenter', e => { if (e.pointerType !== 'touch') restart(el); });   // a mouse or pen, never a tap
    hostOf(el).addEventListener('focusin', e => { if (!e.target.closest('.loop-pause')) restart(el); });
  });
}
// one listener for every pause button: its own (m-own) toggles; a ui.js one is mirrored after ui.js ran
document.addEventListener('click', e => {
  const btn = e.target.closest('.loop-pause');
  if (!btn) return;
  const box = btn.parentElement;
  if (btn.classList.contains('m-own')) {
    const paused = !btn.classList.contains('is-paused');
    btn.__show(paused);
    setPaused(box, paused);
    box.querySelectorAll('[data-ui]').forEach(r => { try { paused ? r.__ui?.pause() : r.__ui?.resume(); } catch { /* none */ } });
  } else setPaused(box, btn.classList.contains('is-paused'));
});

/* ---------------------------------------------------------------- switches */
document.addEventListener('click', e => {
  const sw = e.target.closest('.m-switch');
  if (!sw || e.defaultPrevented) return;
  sw.setAttribute('aria-checked', String(sw.getAttribute('aria-checked') !== 'true'));
});

/* ---------------------------------------------------------------- start */
export function init() {
  if (html.dataset.motion === 'off') return;
  if (!html.hasAttribute('data-motion')) html.setAttribute('data-motion', '');
  try { nav(); } catch (e) { console.warn('motion.js: nav', e); }
  try { tabs(); } catch (e) { console.warn('motion.js: tabs', e); }
  try { counts(); } catch (e) { console.warn('motion.js: counts', e); }
  try { loops(); } catch (e) { console.warn('motion.js: loops', e); }
}
/** call again after the page adds markup (a tab list, a stat, a loop) */
export const refresh = init;
export default { init, refresh };

if (document.readyState === 'loading') document.addEventListener('DOMContentLoaded', init, { once: true });
else init();
```

## assets/system.css

```css
/*
  taste-skill system.css - the one system for the page. Link it first, then assets/ui/cards.css (the
  feature card, its panel and the fragment placements), then set the palette (--p-*), the fonts and
  the radius set on :root in the page (node scripts/start.mjs prints that block).
  Do not add a second grid, spacing scale, type scale, radius scale, button or card style.
*/

/* ---------------------------------------------------------------- tokens */
:root {
  /* palette (hex only; fx.js reads --bg --surface --ink --muted --accent --accent-ink) */
  --p-bg: #ffffff;
  --p-surface: #f4f4f4;
  --p-ink: #111111;
  --p-muted: #6b6b6b;
  --p-accent: #1a4be0;
  --p-accent-ink: #ffffff;
  --p-action: #1a4be0;      /* buttons, links, focus: the accent, or the ink when the accent is too pale */
  --p-action-ink: #ffffff;
  --p-inv-bg: #111111;
  --p-inv-surface: #1c1c1c;
  --p-inv-ink: #f4f4f4;
  --p-inv-muted: #a3a3a3;
  --p-inv-accent: #ffffff;
  --p-inv-accent-ink: #111111;
  --p-inv-action: #ffffff;
  --p-inv-action-ink: #111111;
  /* the lit ground behind UI and in the closing band (references/cards.md), one per page, tech pages only
     (none on kind other, which is flat): <html data-ground="sand|sky|sage|painting">. system.js gives it to every ground
     that holds UI, on light and dark pages alike; --hue on one panel changes only its accent */
  --ground-bg: #d9ccb7; --ground-surface: #e9e0d2; --ground-ink: #2a2622; --ground-muted: #5a534a; --ground-accent: #a8794a;

  /* fonts */
  --font-display: system-ui, -apple-system, "Segoe UI", sans-serif;
  --font-text: system-ui, -apple-system, "Segoe UI", sans-serif;
  --font-mono: ui-monospace, "SFMono-Regular", Menlo, Consolas, monospace;
  --display-weight: 400;    /* headlines 400; 500 on a tech page (software, gaming, hardware) */
  --display-tracking: -0.025em;

  /* the page rhythm (references/layout-v4.md): one section per screen. The header is fixed and
     --nav tall; every section is padded --padt above and --padb (plus the phone action bar, --bar)
     below, and is exactly one or two screens tall (--h: 1 or 2; --hm on a phone). The hero's words
     start --hgap under the header and its content ends --hpadb above the fold. */
  --nav: 72px;
  --padt: 120px;
  --padb: 72px;
  --hgap: 48px;
  --hpadb: 64px;
  --bar: 0px;
  /* grid: one .wrap (alias .container) for the header, every section and the footer, the same
     --gut on both sides: 1280 wide at x 80-1360 at 1440, 342 at x 24-366 at 390 */
  --container: 1280px;
  --gut: 80px;
  --pad: var(--gut);
  --gutter: 24px;

  /* spacing: 4/8 scale */
  --s-1: 4px;  --s-2: 8px;  --s-3: 12px; --s-4: 16px; --s-5: 24px;
  --s-6: 32px; --s-7: 48px; --s-8: 64px; --s-9: 96px; --s-10: 128px;
  --section: 40px;      /* only a .section outside main (the kit's demo pages); every main > section and
                           footer.closing take --padt and --padb */
  --section-band: 56px; /* the same for an older footer outside footer.closing */
  --head-gap: 48px;     /* the one heading-to-content gap, 32 on a phone (layout.css reads it as --ly-head-gap) */
  --card-pad: 16px;     /* card padding (cards.css sets its own for cards with a panel) */
  --card-gap: 16px;     /* gap between cards */

  /* type scale (the only sizes on the page) */
  --t-label: 13px;
  --t-small: 14px;
  --t-body: 16px;
  --t-lead: 18px;
  --t-card: 18px;
  --t-h3: 24px;
  /* straight lines from the phone size at 390 to the desktop size at 1440 (system.md's table) */
  --t-h2: clamp(30px, 1.72vw + 23.3px, 48px);
  --t-h1: clamp(38px, 3.24vw + 25.4px, 72px);
  /* a big number stays a step under the h2 (VI-11): 38 at 1440, 28 at 390, so it never reads as a
     section heading; a stats row and a price set their own, smaller still */
  --t-stat: clamp(28px, .952vw + 24.29px, 38px);

  /* radius: one scale (soft by default; data-radius on <html> switches it) */
  --r-sm: 8px;  --r-md: 12px; --r-lg: 20px; --r-btn: 8px;

  /* controls (fixed here; a page never adds its own) */
  --btn-h: 44px;
  --btn-h-sm: 36px;
  --nav-h: var(--nav);  /* the older name, kept for the kit's files */
  --icon: 16px;         /* small icons: socials, menu chevrons, mono brand marks */

  /* motion */
  --ease: cubic-bezier(.2, .7, .2, 1);
  --dur: 180ms;
}
html[data-radius="sharp"] { --r-sm: 4px; --r-md: 6px;  --r-lg: 8px;  --r-btn: 4px; }
html[data-radius="soft"]  { --r-sm: 8px; --r-md: 12px; --r-lg: 20px; --r-btn: 8px; }
html[data-radius="round"] { --r-sm: 10px; --r-md: 16px; --r-lg: 24px; --r-btn: 999px; }
/* the four lit grounds (references/cards.md); sand is the default above. painting goes with
   painterly-landscape (x.ai's daylight sky over olive hills) */
html[data-ground="sky"]      { --ground-bg: #c6d3df; --ground-surface: #d6e0e9; --ground-ink: #25303a; --ground-muted: #4d5964; --ground-accent: #3f6a92; }
html[data-ground="sage"]     { --ground-bg: #cdd6c6; --ground-surface: #dce3d6; --ground-ink: #262d24; --ground-muted: #535b51; --ground-accent: #6c8e63; }
html[data-ground="painting"] { --ground-bg: #79a8d6; --ground-surface: #a6c4e2; --ground-ink: #16212c; --ground-muted: #273949; --ground-accent: #9c9b3c; }
/* cards.css takes its card and panel radii from this scale */
html[data-radius] .card { --c-r: var(--r-lg); }
html[data-radius] .panel { --p-r: var(--r-md); }

/* a phone: the header 56, the sections padded 96 / 48, the hero's words 40 under the header and its
   content ending 24 above the fold, 24 at both sides, 12 between columns */
@media (max-width: 639px) {
  :root { --nav: 56px; --padt: 96px; --padb: 48px; --hgap: 40px; --hpadb: 24px; --gut: 24px; --gutter: 12px; --card-gap: 12px; --head-gap: 32px; }
}
@media (min-width: 640px) {
  :root { --gut: 48px; --section: 48px; --section-band: 72px; --card-pad: 24px; }
}
@media (min-width: 1024px) {
  :root { --gut: 80px; --section: 56px; --section-band: 96px; --card-gap: 24px;
          --t-lead: 20px; --t-card: 20px; --t-h3: 28px; }
  /* a place (and a local service): the 1200 column at x 120-1320 */
  html[data-type="place"] { --gut: 120px; --container: 1200px; }
}
/* a place (VI-06, ST-96, PH-23): its own smaller type (h1 48/53 and 32/37, h2 32/40 and 24/30, titles
   and the sub-line 20 and 18), the Recipe 3 hero's gap under its photograph (40, 24 on a phone),
   square photographs and cards and 2px buttons, whatever data-radius and data-buttons say */
:root[data-type="place"] {
  --hgap: 40px;
  --t-h1: clamp(32px, 1.524vw + 26.06px, 48px);
  --t-h2: clamp(24px, .762vw + 21px, 32px);
  --t-h3: clamp(20px, .381vw + 18.51px, 24px);
  --r-sm: 2px; --r-md: 0px; --r-lg: 0px; --r-btn: 2px;
}
:root[data-type="place"] :is(h1, .t-h1) { line-height: 1.104; }
:root[data-type="place"] :is(h2, .t-h2) { line-height: 1.25; }
@media (max-width: 639px) {
  :root[data-type="place"] { --hgap: 24px; }
  :root[data-type="place"] :is(h1, .t-h1) { line-height: 1.156; }
}

/* ---------------------------------------------------------------- grounds (at most three) */
:root, .ground-base {
  --bg: var(--p-bg); --surface: var(--p-surface); --ink: var(--p-ink); --muted: var(--p-muted);
  --accent: var(--p-accent); --accent-ink: var(--p-accent-ink); --card: var(--p-surface);
  --action: var(--p-action); --action-ink: var(--p-action-ink);
}
.ground-surface {
  --bg: var(--p-surface); --surface: var(--p-bg); --ink: var(--p-ink); --muted: var(--p-muted);
  --accent: var(--p-accent); --accent-ink: var(--p-accent-ink); --card: var(--p-bg);
  --action: var(--p-action); --action-ink: var(--p-action-ink);
}
.ground-inverse {
  --bg: var(--p-inv-bg); --surface: var(--p-inv-surface); --ink: var(--p-inv-ink); --muted: var(--p-inv-muted);
  --accent: var(--p-inv-accent); --accent-ink: var(--p-inv-accent-ink); --card: var(--p-inv-surface);
  --action: var(--p-inv-action); --action-ink: var(--p-inv-action-ink);
}
:root, .ground-base, .ground-surface, .ground-inverse {
  --line: color-mix(in oklab, var(--ink) 11%, transparent);
  --line-strong: color-mix(in oklab, var(--ink) 20%, transparent);
  /* effects (fx.js) and UI fragments (assets/ui) take the colours of the ground they sit on */
  --fx-bg: var(--bg); --fx-surface: var(--surface); --fx-ink: var(--ink); --fx-muted: var(--muted);
  --fx-accent: var(--accent); --fx-accent-ink: var(--accent-ink);
  --ui-font: var(--font-text); --ui-mono: var(--font-mono); --ui-radius: var(--r-md);
}
.ground-base, .ground-surface, .ground-inverse { background: var(--bg); color: var(--ink); }

/* ---------------------------------------------------------------- base */
*, *::before, *::after { box-sizing: border-box; }
/* an anchor lands on the section's top (the judge's frame): the fixed header covers --nav of its --padt */
html { -webkit-text-size-adjust: 100%; scroll-padding-top: 0; scroll-behavior: auto; margin: 0; }
body {
  margin: 0; background: var(--bg); color: var(--ink);
  font: 400 var(--t-body)/1.55 var(--font-text);
  -webkit-font-smoothing: antialiased; text-rendering: optimizeLegibility; overflow-x: clip;
}
img, svg, video, canvas { display: block; max-width: 100%; }
/* hidden means hidden, also on an img the line above makes a block (a menu tab's photograph, a peek photograph) */
[hidden] { display: none !important; }
img, video { height: auto; }
a { color: inherit; text-decoration: none; }
button { font: inherit; color: inherit; }
h1, h2, h3, h4, p, ul, ol, figure, blockquote { margin: 0; }
ul, ol { padding: 0; list-style: none; }
h1, h2, h3, h4 {
  font-family: var(--font-display); font-weight: var(--display-weight);
  letter-spacing: var(--display-tracking); text-wrap: balance; overflow-wrap: break-word;
}
h1, .t-h1 { font-size: var(--t-h1); line-height: 1.04; }
h2, .t-h2 { font-size: var(--t-h2); line-height: 1.1; }
h3, .t-h3 { font-size: var(--t-h3); line-height: 1.2; }
h4, .t-card { font-family: var(--font-text); font-weight: 500; font-size: var(--t-card); line-height: 1.3; letter-spacing: -0.01em; }
p { text-wrap: pretty; }
.t-lead  { font-size: var(--t-lead); line-height: 1.5; }
.t-body  { font-size: var(--t-body); line-height: 1.55; }
.t-small { font-size: var(--t-small); line-height: 1.5; }
.t-label { font-size: var(--t-label); font-weight: 500; line-height: 1.4; letter-spacing: 0.01em; }
.t-stat  { font-family: var(--font-display); font-weight: var(--display-weight); font-size: var(--t-stat);
           line-height: 1; letter-spacing: var(--display-tracking); font-variant-numeric: tabular-nums; }
.muted { color: var(--muted); }
.mono  { font-family: var(--font-mono); }
.measure { max-width: 62ch; }
:focus-visible { outline: 2px solid var(--action); outline-offset: 2px; border-radius: 2px; }
::selection { background: color-mix(in oklab, var(--accent) 30%, transparent); }

/* ---------------------------------------------------------------- grid */
.wrap, .container { width: min(100% - 2 * var(--gut), var(--container)); margin-inline: auto; padding-inline: 0; }
.grid { display: grid; grid-template-columns: repeat(12, minmax(0, 1fr)); column-gap: var(--gutter); row-gap: var(--card-gap); }
.grid > * { min-width: 0; grid-column: 1 / -1; }
.section { padding-block: var(--section); }

/* ---------------------------------------------------------------- one section per screen
   The page is a stack of screens. Framed at scrollY = k x the screen height, a shot shows exactly one
   section: the hero alone on the first screen, every other section (and footer.closing, the close and
   the footer in one) exactly one or two screens tall, its h2 first, --padt under its top. Heights
   and padding do it: no pinning, snapping or script.
     <section class="section" id=".." style="--h: 2">     (2 screens; --hm sets a phone's count alone)
       <div class="wrap"> the h2 first, then the content </div>     (the one in-flow child)
     </section>
   A layer (a full-bleed photograph or ground behind the words) is an <img class="layer" aria-hidden="true">
   beside the .wrap. Content must fit: one screen holds 708px of content at 1440x900 (900 - 120 - 72),
   two hold 1,608 (the second screen's part starts --padt under its top with .g2). From 640 to 1279 a
   section keeps at least its screens and grows past them rather than overlap the next. */
main > :is(section, .section), footer.closing {
  position: relative; box-sizing: border-box;
  height: calc(var(--h, 1) * 100vh); height: calc(var(--h, 1) * 100svh);
  padding: var(--padt) 0 calc(var(--padb) + var(--bar));
}
main > :is(section, .section) > :is(.wrap, .container), footer.closing > .wrap { position: relative; z-index: 1; height: 100%; }
main > :is(section, .section) > :is(.wrap, .container):not(.grid) { display: flex; flex-direction: column; }
/* a .wrap that is the section's grid too (class="wrap grid"): its rows start at the top */
main > :is(section, .section) > :is(.wrap, .container).grid { align-content: start; }
/* .fill: the block that takes what the screen leaves under the heading (a photograph, a frame) */
main > :is(section, .section) > .wrap > .fill { flex: 1 1 0; min-height: 0; }
/* two screens as two groups: row 1 the h2 and the first group, row 2 the rest (no second h2), starting
   --padt under the second screen's top, so the second shot looks like any other section */
main > :is(section, .section).g2, footer.closing.g2 { --h: 2; }
main > :is(section, .section).g2 > :is(.wrap, .container), footer.closing.g2 > .wrap { display: grid; grid-template-rows: minmax(0, 1fr) minmax(0, 1fr); row-gap: calc(var(--padt) + var(--padb) + var(--bar)); }
main > :is(section, .section).g2 > :is(.wrap, .container) > *, footer.closing.g2 > .wrap > * { min-height: 0; align-self: start; }
main > :is(section, .section).g2 > :is(.wrap, .container) > :is(.group, .fill) { align-self: stretch; }
/* a group: one screen's part of a .g2 section (its h2 and first rows, or the rest) */
.group { display: flex; flex-direction: column; min-height: 0; }
/* each screen's card row runs down to that screen's foot (the cards stretch, their panels grow), so
   the second screen of a .g2 is never a short row over an empty ground; both rows end on the same line */
@media (min-width: 1024px) {
  main > :is(section, .section).g2 > .wrap > .grid:last-child,
  main > :is(section, .section).g2 > .wrap > .group > .grid:last-child { align-self: stretch; flex: 1 1 auto; min-height: 0; }
}
.group > .fill { flex: 1 1 0; min-height: 0; }
@media (min-width: 640px) and (max-width: 1279px) {
  main > :is(section, .section):not(.hero, .section--hero), footer.closing {
    height: auto; min-height: calc(var(--h, 1) * 100vh); min-height: calc(var(--h, 1) * 100svh);
  }
  /* the section is a column and its .wrap takes all of it, so a .fill (a photograph, a frame) still
     grows to the section's foot (a min-height percentage would not resolve, leaving it at its least) */
  main > :is(section, .section):not(.hero, .section--hero), footer.closing { display: flex; flex-direction: column; }
  main > :is(section, .section):not(.hero, .section--hero) > :is(.wrap, .container), footer.closing > .wrap { flex: 1 0 auto; height: auto; min-height: 0; }
}
/* a phone: lvh (the screen with its bars hidden) for every section but the hero; --hm when stacking
   makes the content taller than the desktop count */
@media (max-width: 639px) {
  main > :is(section, .section):not(.hero, .section--hero), footer.closing {
    height: calc(var(--hm, var(--h, 1)) * 100vh); height: calc(var(--hm, var(--h, 1)) * 100lvh);
  }
  main > :is(section, .section).g2m, footer.closing.g2m { --hm: 2; }
  main > :is(section, .section).g2m > :is(.wrap, .container), footer.closing.g2m > .wrap { display: grid; grid-template-rows: minmax(0, 1fr) minmax(0, 1fr); row-gap: calc(var(--padt) + var(--padb) + var(--bar)); }
  main > :is(section, .section).g2:not(.g2m) > :is(.wrap, .container) { display: flex; row-gap: 0; }
  main > :is(section, .section).g2:not(.g2m) > :is(.wrap, .container) > * { align-self: stretch; }
}
/* a layer: the photograph or ground of a whole section, under its words, edge to edge */
main > :is(section, .section) > :is(img, video, canvas, picture).layer,
footer.closing > :is(img, video, canvas, picture).layer {
  position: absolute; inset: 0; z-index: 0; width: 100%; height: 100%; max-width: none; object-fit: cover; object-position: var(--pos, 50% 50%);
}
main > :is(section, .section) > picture.layer > img { width: 100%; height: 100%; object-fit: cover; }

/* the hero: exactly the first screen, the fixed header over its top. Its words start --hgap under the
   header (--photo adds a full-bleed photograph's height above them on a place); its content ends
   --hpadb above the fold, and nothing of the next section shows, at 1440, 1024, 820 and 390.
     <section class="hero hero--<type>"><div class="wrap"><div class="hero-text"> .. </div>[one strip]</div></section>
   .hero-text holds the whole composition (the words, and the picture or window beside or under them);
   one docked strip (logos, three facts) may follow it, at most 96px, desktop only. */
main > :is(.hero, .section--hero) {
  height: 100vh; height: 100svh; min-height: 0;
  padding: calc(var(--nav) + var(--photo, 0px) + var(--hgap)) 0 calc(var(--hpadb) + var(--bar));
}
:is(.hero, .section--hero) > .wrap { height: 100%; display: grid; grid-template-rows: minmax(0, 1fr) auto; row-gap: var(--s-7); }
:is(.hero, .section--hero) > .wrap > .hero-text { min-height: 0; }
:is(.hero, .section--hero) > .wrap > .hero-strip { grid-row: 2; max-height: 96px; }
@media (max-width: 639px) { :is(.hero, .section--hero) > .wrap > .hero-strip { display: none; } }
.section--hero .section-head { max-width: 860px; }
/* the starter's own hero (hero-split): the words on span-5 top-aligned, the picture on span-6 start-7
   as tall as the hero's content box (at most 716 at 1440x900); a phone stacks words, then the picture
   in what is left */
.hero-text.hero-split { height: 100%; align-items: stretch; }
.hero-split > .hero-copy { align-self: start; }
.hero-split > .hero-visual { position: relative; min-height: 0; height: 100%; }
@media (max-width: 1023px) {
  .hero-text.hero-split { display: flex; flex-direction: column; gap: var(--s-6); }
  .hero-split > .hero-visual { flex: 1 1 0; min-height: 200px; height: auto; }
}
.hero-copy > * + * { margin-top: var(--s-5); }
/* the h1 first in the words' column: its capitals start on the picture's top line, not the line box's
   (the leading above the capitals would put the words about 12px under the photograph's top) */
.hero-copy > h1:first-child { text-box: trim-start cap alphabetic; }
.hero-copy > .btn-row { margin-top: var(--s-6); }
/* three short facts at the foot of the words' column, on the picture's bottom line (no dead space
   under the buttons): <dl class="hero-facts"><div><dt>..</dt><dd>..</dd></div> x3</dl> */
@media (min-width: 1024px) {
  .hero-split > .hero-copy { height: 100%; display: flex; flex-direction: column; }
  .hero-split > .hero-copy > .hero-facts { margin-top: auto; }
}
.hero-facts { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: var(--s-5) var(--gutter); margin: 0; }
.hero-facts > div { display: grid; gap: var(--s-1); align-content: start; }
.hero-facts dt { font-size: var(--t-small); color: var(--muted); }
.hero-facts dd { margin: 0; font-weight: 500; }
@media (max-width: 1023px) { .hero-facts { display: none; } }
/* a notice line above the h1 (changed hours, a delivery cut-off): one line with a number, static,
   starting on the left line like the h1 under it */
.hero-notice { font-size: var(--t-small); font-weight: 500; color: var(--ink); }
.hero-notice b, .hero-notice strong { font-weight: 500; color: var(--accent); }
.hero-copy > .hero-notice + h1 { margin-top: var(--s-5); }

/* a phone: open tiles from a row of three or four (rooms, goods, beans) go two-up, an odd last one
   across the row, so a row of three fits one 844px screen (stacked one under another they need two) */
@media (max-width: 639px) {
  .grid > .tile:is(.span-3, .span-4) { grid-column: auto / span 6; }
  .grid > .tile:is(.span-3, .span-4):nth-child(odd of .tile):nth-last-child(1 of .tile) { grid-column: 1 / -1; }
  .grid > .tile:is(.span-3, .span-4) .tile__head { flex-wrap: wrap; gap: 0 var(--s-2); }
}
/* a phone: open cards (words on the ground, the panel under them) stacked one under another keep 32px
   between a panel's foot and the next card's title, so a title never reads as the panel's caption */
@media (max-width: 639px) { .grid:has(> .card--open) { row-gap: var(--s-6); } }
/* spans: desktop >= 1024. Tablet 640-1023: text cards on span-3 and span-6 go two-up; cards with a
   panel (a visual needs the width; the one exception is a span-6 pair, below)
   and everything else (split text and visuals, span-4 cards, span-5/7/8 blocks) take the full row, so a split stacks and no row is left half empty. Phone < 640: one column. */
@media (min-width: 640px) and (max-width: 1023px) {
  .grid > .card.span-3:not(:has(> .panel)), .grid > .card.span-6:not(:has(> .panel)) { grid-column: auto / span 6; }
  /* the one exception with a panel: a 2x2 (or a pair) of span-6 cards stays two-up from 720px (a panel
     of about 310px and more, wide enough for a kit fragment), so four cards take one or two screens
     at 768-820 instead of a card per row (four stacked panels run past two screens). Split cards and
     every other span keep the full row. Under 720 a card per row. */
  @media (min-width: 720px) {
    .grid > .card.span-6:has(> .panel):not(.card--split) { grid-column: auto / span 6; }
    .grid > .card.span-6:has(> .panel):not(.card--split):nth-child(odd of .card.span-6):nth-last-child(1 of .card.span-6) { grid-column: 1 / -1; }
  }
  /* open tiles (rooms, models, people, a photo pair) go two-up; an odd last one takes the row */
  .grid > .tile:is(.span-3, .span-4, .span-6) { grid-column: auto / span 6; }
  .grid > .tile:is(.span-3, .span-4, .span-6):nth-child(odd of .tile):nth-last-child(1 of .tile) { grid-column: 1 / -1; }
}
@media (min-width: 1024px) {
  .span-2  { grid-column: auto / span 2; }
  .span-3  { grid-column: auto / span 3; }
  .span-4  { grid-column: auto / span 4; }
  .span-5  { grid-column: auto / span 5; }
  .span-6  { grid-column: auto / span 6; }
  .span-7  { grid-column: auto / span 7; }
  .span-8  { grid-column: auto / span 8; }
  .span-9  { grid-column: auto / span 9; }
  .span-10 { grid-column: auto / span 10; }
  .span-12 { grid-column: auto / span 12; }
  .start-1 { grid-column-start: 1; } .start-2 { grid-column-start: 2; } .start-3 { grid-column-start: 3; }
  .start-5 { grid-column-start: 5; } .start-6 { grid-column-start: 6; } .start-7 { grid-column-start: 7; }
  .start-4 { grid-column-start: 4; } .start-8 { grid-column-start: 8; } .start-9 { grid-column-start: 9; }
  .start-10 { grid-column-start: 10; }
  /* a split is top-aligned: the words' first line and the picture's top on one line (never centred
     against each other, never mirrored: text left, the visual right, on every split of the page) */
  .grid--middle, .grid--top { align-items: start; }
}
/* card rows: every card in a row the same height (a plain .grid stretches them) */
.grid:is(.grid--middle, .grid--top) > .card { align-self: stretch; }
/* 1024-1279: a span-4 card's panel would be about 220px wide, too narrow for real UI. Cards with a
   panel on span-4 go two-up, a span-8 one takes the row, and an odd last one takes the row too
   (steps, proof row: two, then one wide; bento: the wide card, then the four small ones two by two) */
@media (min-width: 1024px) and (max-width: 1279px) {
  .grid > .card.span-4:has(> .panel) { grid-column: auto / span 6; }
  .grid > .card.span-8:has(> .panel),
  .grid > .card.span-4:has(> .panel):nth-child(odd of .card.span-4):nth-last-child(1 of .card.span-4) { grid-column: 1 / -1; }
}

/* ---------------------------------------------------------------- section heading */
.section-head { max-width: 760px; margin-bottom: var(--head-gap); display: grid; gap: var(--s-4); }
/* every heading starts on the left line (a centred head among left ones reads as a mistake): the
   old centred head is left-aligned too */
.section-head--center { text-align: left; justify-items: start; }
/* a head row: the h2 (and its line) left, one link or the section's button at the right line */
.section-head--split { max-width: none; display: flex; align-items: end; justify-content: space-between; gap: var(--s-5); flex-wrap: wrap; }
.section-head--split > div { display: grid; gap: var(--s-4); max-width: 760px; }
.section-head p { color: var(--ink); font-size: var(--t-lead); line-height: 1.5; max-width: 60ch; }
.section-head--row { max-width: none; display: flex; align-items: end; justify-content: space-between; gap: var(--s-5); flex-wrap: wrap; }
.section-head--row > div { display: grid; gap: var(--s-4); max-width: 760px; }
/* the hero row (hero-product, hero-cycle): the h1 and its line left, the buttons right, across 1280 */
.section--hero .section-head--row { max-width: none; }

/* ---------------------------------------------------------------- buttons (one pair, a dealt look)
   One pair a page, .btn--primary and .btn--secondary, in the look dealt with the page (sections.md)
   and set once on <html data-buttons>; the markup never changes:
     solid  filled primary, outlined secondary, the radius scale (Figma; the default)
     pill   pills, a chevron after the primary's label, a tinted secondary (x.ai)
     tonal  the radius scale, a tinted secondary with no outline (Linear, Vercel)
     arrow  an arrow after the primary's label; the secondary is a text link with an arrow (Stripe)
     block  near-square corners, 48px tall; the secondary is an underlined text link (shops, hotels) */
.btn {
  display: inline-flex; align-items: center; justify-content: center; gap: var(--s-2);
  height: var(--btn-h); padding: 0 var(--s-5); border-radius: var(--r-btn); border: 1px solid transparent;
  font: 500 var(--t-body)/1 var(--font-text); letter-spacing: 0; white-space: nowrap; cursor: pointer;
  transition: background-color var(--dur) var(--ease), color var(--dur) var(--ease), border-color var(--dur) var(--ease), text-decoration-color var(--dur) var(--ease);
}
.btn--primary { background: var(--action); color: var(--action-ink); }
.btn--primary:hover { background: color-mix(in oklab, var(--action) 86%, var(--ink)); }
/* an inverted primary (the action is the ground's ink: a brand band, a lit ground): its hover steps toward the ground, as the filled one steps toward the ink */
.ground-brand .btn--primary:hover { background: color-mix(in oklab, var(--action) 88%, var(--bg)); }
/* the ground's own colour with a clear border: never grey on grey, readable on an effect too */
.btn--secondary { background: var(--bg); color: var(--ink); border-color: var(--line-strong); }
.btn--secondary:hover { background: color-mix(in oklab, var(--bg) 94%, var(--ink)); border-color: color-mix(in oklab, var(--ink) 35%, transparent); }
.btn--sm { height: var(--btn-h-sm); padding: 0 var(--s-4); font-size: var(--t-small); }
html[data-buttons="pill"] { --r-btn: 999px; }
html[data-buttons="block"] { --r-btn: 2px; --btn-h: 48px; }
html:is([data-buttons="pill"], [data-buttons="tonal"]) .btn--secondary { background: color-mix(in oklab, var(--ink) 8%, var(--bg)); border-color: transparent; }
html:is([data-buttons="pill"], [data-buttons="tonal"]) .btn--secondary:hover { background: color-mix(in oklab, var(--ink) 14%, var(--bg)); border-color: transparent; }
/* arrow and block: the secondary is a text link at the button's height (no box: one button look) */
html:is([data-buttons="arrow"], [data-buttons="block"]) .btn--secondary {
  background: none; border-color: transparent; padding-inline: var(--s-2);
  text-decoration: underline; text-underline-offset: 4px; text-decoration-color: var(--line-strong);   /* the .link look */
}
html:is([data-buttons="arrow"], [data-buttons="block"]) .btn--secondary:hover { background: none; border-color: transparent; text-decoration-color: currentColor; }
/* the glyph after a label: a chevron (pill) or an arrow (arrow), drawn so every font shows the same */
html[data-buttons="pill"] .btn--primary::after,
html[data-buttons="arrow"] :is(.btn--primary, .btn--secondary)::after {
  content: ""; flex: none; width: 12px; height: 12px; margin-right: -2px; background: currentColor;
  -webkit-mask: var(--btn-glyph) center / contain no-repeat; mask: var(--btn-glyph) center / contain no-repeat;
  transition: transform var(--dur) var(--ease);
}
html[data-buttons="pill"] { --btn-glyph: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 12 12'%3E%3Cpath d='M4.5 2.5 8 6l-3.5 3.5' fill='none' stroke='black' stroke-width='1.6' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/svg%3E"); }
html[data-buttons="arrow"] { --btn-glyph: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 12 12'%3E%3Cpath d='M1.5 6h8.5M6.5 2.5 10 6l-3.5 3.5' fill='none' stroke='black' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/svg%3E"); }
html:is([data-buttons="pill"], [data-buttons="arrow"]) .btn:hover::after { transform: translateX(3px); }
.btn:active { transform: translateY(1px) scale(.985); }
.btn--arrow::after { content: "\2192"; transition: transform var(--dur) var(--ease); }
.btn--arrow:hover::after { transform: translateX(3px); }
/* the drawn arrow (assets/motion: its stem grows and its head moves 3px on hover), the last child of
   a primary button, a pill or a link: <svg class="m-arrow" viewBox="0 0 12 12" aria-hidden="true">
   <path class="m-arrow__stem" d="M1 6h8"/><path class="m-arrow__head" d="M4.5 2.5 8 6l-3.5 3.5"/></svg>.
   It replaces the dealt glyph, so a button never shows two */
.m-arrow { flex: none; width: 12px; height: 12px; margin-right: -2px; overflow: visible;
           fill: none; stroke: currentColor; stroke-width: 1.6; stroke-linecap: round; stroke-linejoin: round; }
.btn--xl .m-arrow { width: 0.62em; height: 0.62em; stroke-width: 1.4; }
:is(.btn, .link, .pill):has(> .m-arrow)::after { content: none; display: none; }
.btn-row { display: flex; flex-wrap: wrap; gap: var(--s-3); align-items: center; }
.section-head--center .btn-row { justify-content: center; }
/* on the phone every button row stacks full width */
@media (max-width: 639px) {
  .btn-row { flex-direction: column; align-items: stretch; width: 100%; justify-self: stretch; }
  /* full width, the second action is a text link at the button's height under the filled one (an
     outline across the whole screen reads as a rule, a pale fill as a switched-off button) */
  .btn-row > .btn--secondary { background: none; border-color: transparent; text-decoration: underline; text-underline-offset: 4px; text-decoration-color: var(--line-strong); }
  .section-head--split > .btn { width: 100%; }
}
.link { display: inline-flex; align-items: center; gap: var(--s-2); font-weight: 500; color: var(--ink);
        text-decoration: underline; text-decoration-color: var(--line-strong); text-underline-offset: 4px;
        transition: text-decoration-color var(--dur) var(--ease); }
.link:hover { text-decoration-color: currentColor; }
.link::after { content: "\2192"; display: inline-block; text-decoration: none; transition: transform var(--dur) var(--ease); }
.link:hover::after { transform: translateX(3px); }
.pill { display: inline-flex; align-items: center; gap: var(--s-2); height: 32px; padding: 0 var(--s-3);
        border-radius: 999px; border: 1px solid var(--line); background: var(--card);
        font-size: var(--t-label); color: var(--ink); }
.pill .muted { color: var(--muted); }

/* ---------------------------------------------------------------- card (one style) */
.card {
  display: flex; flex-direction: column; gap: var(--s-2); min-width: 0;
  background: var(--card); border: 1px solid var(--line); border-radius: var(--r-lg); padding: var(--card-pad);
}
/* small titles in the text face: a display face at 20px (a condensed or hairline serif) reads smaller than the body */
.card > h3, .card > h4, .card--plan h3 { font-family: var(--font-text); font-weight: 500; font-size: var(--t-card); line-height: 1.3; letter-spacing: -0.01em; }
/* the type floor (system.md): text people read is 16px or more in ink; cards.css sets it for its
   card text, this for a plain card's lines */
.card > p { color: var(--ink); font-size: var(--t-body); max-width: 52ch; }
/* at most two boxes one inside the other (sections.md): a card whose panel holds UI drops its own
   box, so its title and lines sit on the page ground over the panel (Figma's tiles), or runs the
   panel flush to its sides and bottom (cards.css .panel--flush) */
.card.card--open { --c-pad: 0px; background: none; border: 0; box-shadow: none; padding: 0; overflow: visible; }
.card.card--open > .panel { margin: var(--s-5) 0 0; border-radius: var(--r-lg); }
.card.card--open > .panel--top { margin: 0 0 var(--s-5); }
.card--plan.is-featured { border-color: var(--action); box-shadow: inset 0 0 0 1px var(--action); }
/* cards.css (linked after this file) sets .card { gap: 0 } for feature cards; a plan keeps its 16px rows */
.card.card--plan { gap: var(--s-4); }

/* ---------------------------------------------------------------- visuals: panels and frames
   A card's .panel (the inset visual), .stage and the placements (.at-corner, .at-rise, .at-over,
   .at-right, .at-center, .stage--fade-*) come from assets/ui/cards.css (references/cards.md).
   This file adds the frames (hero, split, square, closing band) and lets a .stage work inside them.
   The grounds' colours: system.js lights every ground that holds UI (see there). */
:not(.card) > .panel { border-radius: var(--p-r, var(--r-md)); }   /* a panel on its own: four round corners */
.menu-card .panel { --p-h: 160px; }
.frame {
  position: relative; isolation: isolate; overflow: clip; display: flex; align-items: center; justify-content: center;
  background: var(--surface); border-radius: var(--r-lg); border: 1px solid var(--line);
  padding: clamp(var(--s-4), 4vw, var(--s-7));
}
/* a zero-width spacer keeps --ratio (height / width) as the frame's minimum height */
.frame::before { content: ""; flex: none; width: 0; padding-top: var(--ratio, 0px); }
/* an object or scene (assets/objects) and a photograph (.frame__img) fill the frame themselves */
.frame > :not(canvas, .stage, .loop-pause, .frame__fx, .frame__img, [data-print], [data-scene], .print3d) { position: relative; max-width: 100%; min-width: 0; }
/* a photograph is a plain rectangle in its frame: no border, no print, no filter */
.frame--photo { padding: 0; border: 0; }
/* .frame--fill: as tall as its row (a split's picture beside the words, down to the section's foot);
   stacked on a tablet or a phone it takes what the screen leaves, at least 200px */
.frame--fill { height: 100%; }
@media (max-width: 1023px) {
  .grid.fill:has(> .frame--fill) { display: flex; flex-direction: column; gap: var(--s-6); }
  .grid.fill > .frame--fill { flex: 1 1 0; height: auto; min-height: 200px; }
}
/* a split's words as a column: the last block (stats, a link) on the picture's bottom line */
.split-col { display: flex; flex-direction: column; gap: var(--s-5); }
@media (min-width: 1024px) { .split-col { height: 100%; } .split-col > .split-col__end { margin-top: auto; } }
:is(.frame, .panel) > .frame__img { position: absolute; inset: 0; z-index: 0; width: 100%; height: 100%; max-width: none; object-fit: cover; }
.frame--portrait { --ratio: 125%; }
/* the product stage (hero-product): wide, the product whole on it; a phone gets a tall stage */
.frame--stage { --ratio: 50%; }
@media (max-width: 639px) { .frame--stage, .frame--portrait { --ratio: 120%; } }
.frame--hero { padding: clamp(24px, 5vw, 64px); }
.frame--split { --ratio: 80%; }
.frame--square { --ratio: 100%; }
/* the closing band: the hero's echo, lit with the page's ground (sand, sky, sage or painting) on light
   and dark pages alike, so its text and buttons take that ground's ink. Its effect sits on a layer of
   its own, <div class="frame__fx" data-fx="..">, held still and calm where the text sits (system.js) */
.frame--cta { min-height: 420px; }
/* .frame--lit: the same for an object's frame that holds a few words in its clear half (the
   product statement's facts in a .frame__text, lower left; the object stands right of centre) */
.frame--cta, .frame--lit {
  border-color: transparent; background: var(--ground-bg); color: var(--ground-ink);
  --bg: var(--ground-surface); --surface: var(--ground-surface); --ink: var(--ground-ink); --muted: var(--ground-muted);
  --action: var(--ground-ink); --action-ink: var(--ground-surface);
  --line: color-mix(in oklab, var(--ground-ink) 12%, transparent); --line-strong: color-mix(in oklab, var(--ground-ink) 24%, transparent);
}
.frame > .frame__fx { position: absolute; inset: 0; z-index: 0; }
/* on a light page the buttons on a lit ground keep the page's own action colour: one action, one look
   (a dark page keeps the lit ground's ink, which reads on the bright ground) */
html:not([data-tone="dark"]) :is(.frame--cta, .frame--lit, .footer--statement) { --action: var(--p-action); --action-ink: var(--p-action-ink); }
/* a card on a photograph (a booking form over the room): the card holds the page's colours */
.frame--photo:has(> .card) { padding: clamp(var(--s-5), 5vw, var(--s-8)) clamp(var(--s-4), 4vw, var(--s-7)); }
.frame--photo > .card { position: relative; z-index: 1; width: min(100%, 480px); }   /* flat: no shadow at rest (ST-97) */
.frame--cta > .frame__fx { opacity: .85; }
.frame--lit > .frame__text { z-index: 2; align-self: flex-end; margin-right: auto; width: min(100%, 360px); display: grid; gap: var(--s-5); }
/* 2:1 on a desktop; a phone gets 2:3, the words at the top and the object smaller and low under
   them (its hero preset), so the facts never touch the object */
.frame--lit { --ratio: 50%; }
@media (max-width: 639px) { .frame--lit { --ratio: 150%; } .frame--lit > .frame__text { align-self: flex-start; } }
@media (max-width: 639px) { .frame--split { --ratio: 100%; } }
/* A hero, wide or split frame (x.ai's hero window): --w on the rising fragment's slot (its true width)
   centres it while it fits; anchored 24px in and cut by the right edge once the frame is narrower.
   --pair on the .stage (the main fragment's width plus the part of the second that sticks out)
   centres the pair; below 1024 the main one alone. In a card's panel cards.css places it (.at-corner,
   .at-rise, .at-right; a centred .at-rise.fit narrows with the panel), as its recipe says. */
.frame > .stage > .at-rise { --ax: max(24px, calc(50% - var(--w, 64%) / 2)); }
.frame--hero > .stage > .at-rise { --ax: max(24px, calc(50% - var(--pair, var(--w, 64%)) / 2)); }
.frame--hero > .stage > .at-over { --bx: max(24px, calc(50% - var(--pair, 800px) / 2)); }
@media (max-width: 1023px) { .frame--hero > .stage > .at-rise { --ax: max(24px, calc(50% - var(--w, 64%) / 2)); } }
/* a frame holding a cards.css .stage: the stage fills it and places the fragments (references/cards.md);
   the hero and wide frames are 16:7 and square on a phone; below 1024 only the main fragment shows */
.frame:has(> .stage) { padding: 0; }
.frame--hero:has(> .stage) { aspect-ratio: 16 / 7; }
@media (max-width: 1023px) {
  .frame--hero:has(> .stage) { aspect-ratio: 3 / 2; }   /* a tablet: room for the main fragment's rows */
}
/* a tablet held upright (820x1180): a taller frame, so the hero fills the screen without a gap */
@media (min-width: 640px) and (max-width: 1023px) and (orientation: portrait) {
  .frame--hero:has(> .stage) { aspect-ratio: 4 / 3; }
}
@media (max-width: 1023px) {
  .frame--hero > .stage > :is(.at-over, .at-over-l) { display: none; }
}
/* a split frame is about 440-560px wide from 1024 to 1279: the main piece alone there */
@media (min-width: 1024px) and (max-width: 1279px) {
  .frame--split > .stage > :is(.at-over, .at-over-l) { display: none; }
}
@media (max-width: 639px) {
  .frame--hero:has(> .stage) { aspect-ratio: 1; }
  .frame > .stage > :is(.at-over, .at-over-l) { display: none; }
  /* phone: cards.css sets a fragment 16px in on both sides, and system.js holds every loop on its
     finished picture, so no pause button lands on the UI */
}
/* a small object on a focal tile (.at-center): its true width (--w, 288px for the slider), narrowed
   on a small panel so 24px of ground always shows on both sides (a span-4 panel is 304px at 1280) */
.stage > .at-center { width: calc(100% - 48px); max-width: var(--w, 288px); }
.stage > .at-center > :is([data-ui], .ui) { width: 100%; max-width: none; }
/* .to-edge stretches a fragment's surface past the bottom (cards.css); doc's surface is its page */
.stage > .to-edge > .ui-doc > .ui-doc__page { min-height: 100%; }
/* .hero-stage: whole fragments side by side in a frame, never overlapping or cut (a row of app
   screens); the first is the main one, and the ones that no longer fit drop out from the end */
.hero-stage { container: hero-stage / inline-size; width: 100%; }
.hero-stage__row { display: flex; align-items: center; justify-content: center; gap: var(--s-5); }
.hero-stage__row > * { flex: none; max-width: 100%; }
@container hero-stage (max-width: 1060px) { .hero-stage__row > :nth-child(n + 3) { display: none; } }
@container hero-stage (max-width: 740px)  { .hero-stage__row > :nth-child(n + 2) { display: none; } }
/* the pause button on every visual that moves by itself for more than 5 s (WCAG 2.2.2): ui.js adds it
   to a loop when it starts to move, system.js to the hero's ground. Write a button.loop-pause into
   the frame only for a loop of your own (hero-cycle). The same solid 32px disc in the page colours as
   assets/ui/base.css's, 12px in from the bottom-right corner, with a 44px hit area. */
.loop-pause {
  position: absolute; right: 12px; bottom: 12px; z-index: 6; display: grid; place-items: center;
  width: 32px; height: 32px; margin: 0; padding: 0; border-radius: 50%; cursor: pointer;
  border: 1px solid color-mix(in oklab, var(--ink) 16%, transparent); background: var(--bg); color: var(--ink);
  box-shadow: 0 1px 2px rgb(0 0 0 / .12), 0 4px 12px -4px rgb(0 0 0 / .28);
}
.loop-pause::before { content: ""; position: absolute; inset: -6px; border-radius: 50%; }
.loop-pause svg { width: 14px; height: 14px; fill: currentColor; }
@media (prefers-reduced-motion: reduce) { .loop-pause { display: none; } }

/* ---------------------------------------------------------------- navigation */
/* The header: fixed, opaque, one hairline under it, the same at every depth (no blur, no shadow, no
   floating pill, never hidden on scroll). Its .wrap is the page's, so the logo starts on the left line
   and the main button ends on the right one. Plain words, 15px, sentence case; one filled button. */
.nav {
  position: fixed; inset: 0 0 auto 0; z-index: 50; height: var(--nav);
  background: var(--bg); color: var(--ink); border-bottom: 1px solid var(--line);
}
.nav__bar { height: 100%; display: flex; align-items: center; gap: var(--s-8); }
/* a text logo: the display face at 500, 22px (the client's SVG logo replaces it when there is one) */
.nav__logo { display: inline-flex; align-items: center; gap: var(--s-2); font: 500 22px/1 var(--font-display); letter-spacing: -0.02em; flex: none; }
@media (max-width: 639px) { .nav__logo { font-size: 20px; } }
.nav__logo svg, .nav__logo img { height: 24px; width: auto; }
/* the brand's mark beside its name, in the accent (the client's logo replaces both). Dealt shapes:
   <span class="mark" data-mark="dots|arc|ring|petal|leaf|stack" aria-hidden="true"></span> */
.mark { display: inline-block; flex: none; width: 22px; height: 22px; background: var(--accent);
        -webkit-mask: var(--mark) center / contain no-repeat; mask: var(--mark) center / contain no-repeat; }
.mark[data-mark="dots"]  { --mark: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Ccircle cx='6.5' cy='6.5' r='4.5'/%3E%3Ccircle cx='17.5' cy='6.5' r='4.5'/%3E%3Ccircle cx='6.5' cy='17.5' r='4.5'/%3E%3Crect x='13' y='13' width='9' height='9' rx='1.5'/%3E%3C/svg%3E"); }
.mark[data-mark="arc"]   { --mark: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath d='M2.5 21.5V12a9.5 9.5 0 0 1 19 0v9.5h-6V12a3.5 3.5 0 0 0-7 0v9.5z'/%3E%3C/svg%3E"); }
.mark[data-mark="ring"]  { --mark: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill-rule='evenodd' d='M12 2a10 10 0 1 1 0 20 10 10 0 0 1 0-20zm0 5.5a4.5 4.5 0 1 0 0 9 4.5 4.5 0 0 0 0-9z'/%3E%3C/svg%3E"); }
.mark[data-mark="petal"] { --mark: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Ccircle cx='12' cy='6.6' r='5'/%3E%3Ccircle cx='17.4' cy='12' r='5'/%3E%3Ccircle cx='12' cy='17.4' r='5'/%3E%3Ccircle cx='6.6' cy='12' r='5'/%3E%3Ccircle cx='12' cy='12' r='4'/%3E%3C/svg%3E"); }
.mark[data-mark="leaf"]  { --mark: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath d='M3 21C3 10 10 3 21 3c0 11-7 18-18 18z'/%3E%3C/svg%3E"); }
.mark[data-mark="stack"] { --mark: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Crect x='2' y='2.5' width='20' height='5.5' rx='2.75'/%3E%3Crect x='2' y='9.25' width='14' height='5.5' rx='2.75'/%3E%3Crect x='2' y='16' width='8' height='5.5' rx='2.75'/%3E%3C/svg%3E"); }
/* initial: the name's first letter on a small accent tile (<span class="mark mark--initial" aria-hidden="true">K</span>);
   wordmark: no mark, the name alone */
.mark.mark--initial { -webkit-mask: none; mask: none; display: inline-grid; place-items: center; border-radius: calc(var(--r-sm) * 0.75);
  color: var(--accent-ink); font: var(--display-weight, 400) 13px/1 var(--font-display); letter-spacing: 0; }
.footer__mark .mark, .footer .nav__logo .mark { width: 0.8em; height: 0.8em; margin-right: 0.18em; vertical-align: -0.04em; }
.footer__mark .mark--initial, .footer .nav__logo .mark--initial { width: 1em; height: 1em; font-size: 0.6em; }
.nav--split .nav__logo .mark--initial { width: 26px; height: 26px; font-size: 15px; }
.nav__links { display: flex; align-self: stretch; gap: var(--s-1); flex: 1; min-width: 0; }
.nav__item { display: flex; align-items: center; height: 100%; }
.nav__link {
  position: relative; display: inline-flex; align-items: center; gap: var(--s-1); height: 36px; padding: 0 var(--s-3); border-radius: var(--r-sm);
  font-size: 15px; font-weight: 500; color: var(--ink); white-space: nowrap; transition: background-color var(--dur) var(--ease), color var(--dur) var(--ease);
}
/* a small laptop (1024-1279): the words sit closer, so a bar of four words never wraps or reaches the logo */
@media (min-width: 1024px) and (max-width: 1279px) {
  .nav__bar { gap: var(--s-6); }
  .nav__link { padding: 0 var(--s-2); }
  .nav__link::before, .nav__link[aria-current="page"]::after { left: var(--s-2); right: var(--s-2); }
  .nav__links { gap: 0; }
}
/* the words take no pill or fill (ST-08, ST-11): a 2px line draws under the word from the left on hover and focus and leaves to the right */
.nav__link::before { content: ""; position: absolute; left: var(--s-3); right: var(--s-3); bottom: -2px; height: 2px; border-radius: 2px; background: currentColor;
  transform: scaleX(0); transform-origin: right; transition: transform var(--dur) var(--ease); pointer-events: none; }
.nav__link:hover::before, .nav__link:focus-visible::before, .nav__item:hover > .nav__link::before, .nav__item:focus-within > .nav__link::before { transform: scaleX(1); transform-origin: left; }
.nav__actions > .nav__link::before { left: var(--s-2); right: var(--s-2); }
.nav__link .chev { width: 12px; height: 12px; opacity: .6; transition: transform var(--dur) var(--ease); }
/* the page you are on: a 2px bar under its word (still, the same inset as the hover line) */
.nav__link[aria-current="page"]::after { content: ""; position: absolute; left: var(--s-3); right: var(--s-3); bottom: -2px; height: 2px; border-radius: 2px; background: currentColor; }
.nav__expand { display: none; }
.nav__actions { display: flex; align-items: center; gap: var(--s-4); flex: none; }
/* the bar's button: the page's primary at the one button height (44, 48 with data-buttons="block"; ST-09) */
.nav__actions .btn { height: var(--btn-h); padding-inline: var(--s-5); font-size: 15px; }
.nav__actions > .nav__link { padding-inline: var(--s-2); }
.nav__toggle { display: none; }

.nav__menu {
  position: absolute; left: 0; right: 0; top: 100%; background: var(--bg); color: var(--ink);
  border-top: 1px solid var(--line); border-bottom: 1px solid var(--line);
  padding: var(--s-7) 0 var(--s-8); opacity: 0; visibility: hidden; transform: translateY(-6px);
  transition: opacity 120ms var(--ease), transform 120ms var(--ease), visibility 0s linear 120ms;
}
@media (min-width: 1024px) {
  /* hover opens the full-width panel after 150ms of intent; a click on the word goes to its page */
  .nav__item:hover > .nav__menu, .nav__item:focus-within > .nav__menu {
    opacity: 1; visibility: visible; transform: none; transition-duration: 200ms, 200ms, 0s; transition-delay: 150ms, 150ms, 150ms;
  }
  .nav__item:hover > .nav__link .chev, .nav__item:focus-within > .nav__link .chev { transform: rotate(180deg); }
}
.menu-col { display: grid; gap: var(--s-2); align-content: start; }
.menu-col > .t-label { color: var(--muted); margin-bottom: var(--s-2); }
.menu-col a { display: grid; gap: var(--s-1); padding: var(--s-2) 0; font-weight: 500; }
.menu-col a span { font-weight: 400; color: var(--muted); font-size: var(--t-small); }
.menu-col a:hover { color: var(--action); }
.menu-card { display: grid; gap: var(--s-3); }
.menu-card .panel { --ratio: 56.25%; }

@media (max-width: 1023px) {
  .nav__links, .nav__actions { display: none; }
  /* the menu button takes the secondary button's look on a 44px tap area */
  .nav__toggle {
    display: inline-grid; place-items: center; margin-left: auto; width: 44px; height: 44px; border-radius: var(--r-btn);
    border: 1px solid var(--line-strong); background: transparent; color: var(--ink); cursor: pointer; flex: none;
  }
  .nav.is-open .nav__links {
    display: flex; flex-direction: column; position: fixed; top: var(--nav); left: 0; right: 0; bottom: 0;
    padding: var(--s-4) var(--gut) calc(var(--btn-h) * 2 + 56px); background: var(--bg); overflow-y: auto; gap: 0;
  }
  .nav.is-open .nav__item { display: grid; grid-template-columns: 1fr auto; height: auto; border-bottom: 1px solid var(--line); }
  .nav.is-open .nav__link { height: 56px; padding: 0; font-size: 20px; background: none; }
  .nav.is-open .nav__link[aria-current="page"]::after { left: 0; right: auto; width: 24px; bottom: 10px; }
  .nav.is-open .nav__link::before { display: none; }
  .nav.is-open .nav__link .chev { display: none; }
  .nav.is-open .nav__expand {
    display: inline-grid; place-items: center; width: 44px; height: 56px; border: 0; background: none; color: var(--ink); cursor: pointer;
  }
  .nav__expand svg { width: 16px; height: 16px; transition: transform var(--dur) var(--ease); }
  .nav__item.is-expanded .nav__expand svg { transform: rotate(180deg); }
  .nav.is-open .nav__menu {
    position: static; grid-column: 1 / -1; display: none; opacity: 1; visibility: visible; transform: none;
    box-shadow: none; border: 0; padding: 0 0 var(--s-4);
  }
  .nav.is-open .nav__item.is-expanded .nav__menu { display: block; }
  .nav.is-open .nav__menu :is(.container, .wrap) { width: 100%; }
  .nav.is-open .nav__menu .grid { row-gap: var(--s-4); }
  /* menu cards become plain rows in the sheet (their title and line), the picture panel hidden */
  .nav.is-open .menu-card { display: grid; gap: var(--s-1); padding: var(--s-2) 0; }
  .nav.is-open .menu-card .panel { display: none; }
  .nav.is-open .nav__actions {
    display: grid; gap: var(--s-2); position: fixed; left: 0; right: 0; bottom: 0; z-index: 51;
    padding: var(--s-4) var(--gut) calc(var(--s-4) + env(safe-area-inset-bottom)); background: var(--bg); border-top: 1px solid var(--line);
  }
  .nav.is-open .nav__actions .btn { width: 100%; height: 48px; font-size: var(--t-body); }
  /* a text action (Sign in, a phone line) under the buttons, centred at their height */
  .nav.is-open .nav__actions .nav__link { display: inline-flex; order: 2; justify-content: center; height: var(--btn-h); font-size: var(--t-body); }
  body.nav-locked { overflow: hidden; }
}
/* a tablet and a phone (1023 and below, ST-15): the main button stays in the bar, at the right line
   with the menu button 12px after it; the plain words go to the sheet */
@media (max-width: 1023px) {
  .nav__bar { gap: var(--s-3); }
  .nav:not(.is-open) .nav__actions { display: flex; margin-left: auto; }
  .nav:not(.is-open) .nav__actions > :not(.btn--primary) { display: none; }
  .nav:not(.is-open) .nav__actions + .nav__toggle { margin-left: 0; }
}
@media (max-width: 639px) {
  /* a long name (15 characters, "Verdigris Works"), the main button and the menu button fit 342px (390 wide) on
     one line each, with no page CSS: the name steps down with the screen (18px at 390), the button is the small
     size, and the three sit 8px apart. A name of 17 characters or more is the client's to shorten to its mark. */
  .nav__bar { gap: var(--s-2); }
  .nav__logo { font-size: clamp(16px, 4.6vw, 20px); gap: 6px; white-space: nowrap; }
  .nav__logo .mark:not(.mark--initial) { width: 20px; height: 20px; }
  .nav:not(.is-open) .nav__actions .btn { height: 44px; padding-inline: var(--s-3); font-size: var(--t-small); }
}
/* no script: the sheet cannot open, so the words stay in the bar (it scrolls sideways) and the dead menu button goes */
@media (scripting: none) and (max-width: 1023px) {
  .nav__links { display: flex; overflow-x: auto; scrollbar-width: none; margin-right: var(--s-3); padding-right: var(--s-2); border-right: 1px solid var(--line); }   /* stops short of the button, and a hairline marks where the row is cut (it scrolls) */
  .nav__toggle { display: none; }
}

/* nav types (dealt, sections.md). Every one is the same fixed bar on the page's .wrap; the full-width
   hover panel on a desktop and the full-screen sheet on a phone are shared, so only the bar's order
   and its panel's contents differ.
   (default) nav-mega: the logo, the words 64px right of it, the actions at the right line; the panel
              holds two or three link columns and one picture card (Figma)
   nav-cards: the same bar; its panel holds three or four picture cards (a.menu-card, span-4 or span-3)
   .nav--centre: the words in the middle of the bar (Linear, Vercel)
   .nav--split: the words left, the name in the middle, the actions right (shops, hotels, restaurants)
   .nav--tone: the bar on the page's inverse ground, a band of colour over a light page (x.ai's black
              bar); the panel under it stays on the page ground */
@media (min-width: 1024px) {
  .nav--split .nav__bar, .nav--centre .nav__bar { display: grid; grid-template-columns: minmax(0, 1fr) auto minmax(0, 1fr); }
  .nav--split .nav__logo { grid-column: 2; grid-row: 1; font-size: 26px; }
  .nav--split .nav__links { grid-column: 1; grid-row: 1; }
  /* the first word starts on the left line (its box, its letters and its underline) */
  .nav--split .nav__links > .nav__item:nth-child(1 of .nav__item) > .nav__link { padding-left: 0; }
  .nav--split .nav__links > .nav__item:nth-child(1 of .nav__item) > .nav__link::before,
  .nav--split .nav__links > .nav__item:nth-child(1 of .nav__item) > .nav__link[aria-current="page"]::after { left: 0; }
  .nav--split .nav__actions, .nav--centre .nav__actions { grid-column: 3; grid-row: 1; justify-self: end; }
  .nav--centre .nav__logo { grid-column: 1; grid-row: 1; justify-self: start; }
  .nav--centre .nav__links { grid-column: 2; grid-row: 1; justify-content: center; flex: none; }
}
.nav--tone { --bg: var(--p-inv-bg); --ink: var(--p-inv-ink); --muted: var(--p-inv-muted); --accent: var(--p-inv-accent); --accent-ink: var(--p-inv-accent-ink);
             --action: var(--p-inv-action); --action-ink: var(--p-inv-action-ink); --card: var(--p-inv-surface);
             --line: color-mix(in oklab, var(--p-inv-ink) 14%, transparent); --line-strong: color-mix(in oklab, var(--p-inv-ink) 26%, transparent); }
.nav--tone .nav__menu { --bg: var(--p-bg); --ink: var(--p-ink); --muted: var(--p-muted); --action: var(--p-action); --line: color-mix(in oklab, var(--p-ink) 11%, transparent); }
/* the older float type is the plain bar now (a floating, frosted pill is not a header) */

/* ---------------------------------------------------------------- logo row */
.logos { display: flex; flex-wrap: wrap; justify-content: space-between; align-items: center; gap: var(--s-5) var(--s-6); }
.logos > * { height: 28px; width: auto; max-width: 100%; color: var(--ink); }
.logos .wordmark { height: auto; font: 500 var(--t-card)/1 var(--font-display); letter-spacing: -0.02em; white-space: nowrap; }
/* --mono marks in a logo row: 28px tall, each in the same box (the mask keeps its shape) */
.logos .icon-mono { width: 120px; height: 28px; }
/* tablet and phone: a grid, so a wrapped row never leaves one logo on its own */
@media (max-width: 1023px) { .logos { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); justify-items: center; } }
@media (max-width: 639px)  { .logos { grid-template-columns: repeat(2, minmax(0, 1fr)); } }

/* ---------------------------------------------------------------- lists, quote, steps */
.checks { display: grid; gap: var(--s-3); }
.checks li { display: grid; grid-template-columns: 20px 1fr; gap: var(--s-3); align-items: start; }
.checks li::before {
  content: ""; width: 20px; height: 20px; margin-top: 2px; background: var(--action);
  -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'%3E%3Cmask id='m'%3E%3Crect width='20' height='20' fill='white'/%3E%3Cpath d='M6 10.2l2.6 2.6L14 7.4' fill='none' stroke='black' stroke-width='1.8' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/mask%3E%3Ccircle cx='10' cy='10' r='10' mask='url(%23m)'/%3E%3C/svg%3E") center / 20px no-repeat;
          mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'%3E%3Cmask id='m'%3E%3Crect width='20' height='20' fill='white'/%3E%3Cpath d='M6 10.2l2.6 2.6L14 7.4' fill='none' stroke='black' stroke-width='1.8' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/mask%3E%3Ccircle cx='10' cy='10' r='10' mask='url(%23m)'/%3E%3C/svg%3E") center / 20px no-repeat;
}
.quote blockquote { font-family: var(--font-display); font-weight: var(--display-weight); font-size: var(--t-h3);
                    line-height: 1.3; letter-spacing: var(--display-tracking); text-wrap: pretty; }
/* cards.css styles .quote as quote text; who said it stays in the text face */
.quote figcaption, .quote .person { font: 400 var(--t-small)/1.45 var(--font-text); letter-spacing: 0; color: var(--ink); }
.person { display: flex; align-items: center; gap: var(--s-3); }
.person img, .person .avatar { width: 40px; height: 40px; border-radius: 999px; object-fit: cover; background: var(--surface); flex: none; }
.person .avatar { display: grid; place-items: center; font: 500 var(--t-small)/1 var(--font-text); color: var(--muted); }
/* the number beside a quote (quote-stat): below 1024 it sits under the quote, by space alone (ST-43) */
@media (max-width: 1023px) { .stat-aside { margin-top: var(--s-7); } }
/* a stats row: two to four short facts, the number above its label (data-m-count on a count) */
.stats { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: var(--s-5) var(--gutter); margin: 0; }
.stats > div { display: flex; flex-direction: column-reverse; justify-content: flex-end; gap: var(--s-2); }
.stats dd { margin: 0; }
.stats .t-stat { font-size: clamp(26px, 1vw + 22px, 36px); }
@media (max-width: 639px) { .stats .t-stat { font-size: 26px; } }
.step-num { display: inline-grid; place-items: center; width: 32px; height: 32px; border-radius: 999px;
            border: 1px solid var(--line-strong); font: 500 var(--t-small)/1 var(--font-text); font-variant-numeric: tabular-nums; }

/* ---------------------------------------------------------------- tabs */
.tabs { display: flex; flex-wrap: wrap; gap: var(--s-2); }
.tab {
  height: var(--btn-h-sm); padding: 0 var(--s-4); border-radius: var(--r-btn); border: 1px solid var(--line);
  background: transparent; color: var(--muted); font-size: var(--t-small); font-weight: 500; cursor: pointer;
  transition: background-color var(--dur) var(--ease), color var(--dur) var(--ease), border-color var(--dur) var(--ease);
}
.tab:hover { color: var(--ink); border-color: var(--line-strong); }
.tab[aria-selected="true"] { background: var(--ink); color: var(--bg); border-color: var(--ink); }
/* the panel shown at load is there on the first frame (MO-62); system.js fades a chosen one in */
[role="tabpanel"][hidden] { display: none; }

/* ---------------------------------------------------------------- pricing */
.price { display: flex; align-items: baseline; gap: var(--s-2); }
.price .t-stat { font-size: var(--t-stat); }
.card--plan { gap: var(--s-4); }
.card--plan .btn { width: 100%; }
.card--plan .checks { margin-top: var(--s-2); }
.badge { display: inline-flex; align-items: center; height: 24px; padding: 0 var(--s-3); border-radius: 999px; vertical-align: middle;
         background: var(--action); color: var(--action-ink); font: 500 var(--t-label)/1 var(--font-text); letter-spacing: 0; }
.card--plan h3 .badge { margin-left: var(--s-2); }

/* ---------------------------------------------------------------- FAQ
   Every answer is open, always (ST-47, MO-70): no accordion, no +/- marker, nothing changes height.
     <div class="faq"><div class="faq__item"><h3>[Question]</h3><p>[Answer]</p></div> ...</div>
   Older <details> markup shows its answer open too (system.js also sets [open]). */
.faq > :is(.faq__item, details) { display: grid; gap: var(--s-2); padding: var(--s-5) 0; border-bottom: 1px solid var(--line); }   /* each row closes with its rule (ST-43) */
.faq > :is(.faq__item, details):first-child { padding-top: 0; }
.faq :is(.faq__item > h3, summary) { display: block; font: 500 var(--t-lead)/1.4 var(--font-text); letter-spacing: 0; list-style: none; cursor: text; }
.faq summary { pointer-events: none; }
.faq summary::-webkit-details-marker { display: none; }
.faq details::details-content { content-visibility: visible; }
.faq :is(.faq__item, details) p { color: var(--ink); max-width: 64ch; }
@media (max-width: 639px) { .faq > :is(.faq__item, details) { padding: var(--s-4) 0; } }

/* ---------------------------------------------------------------- footer */
.footer { padding-block: var(--section) var(--s-6); border-top: 1px solid var(--line); }   /* an older footer after the last section; footer.closing (below) replaces it */
.footer .grid { row-gap: var(--s-7); }
.footer h4 { font: 500 var(--t-small)/1.4 var(--font-text); letter-spacing: 0; color: var(--muted); margin-bottom: var(--s-4); }
.footer ul { display: grid; gap: var(--s-3); font-size: var(--t-small); }
.footer ul a:hover { color: var(--action); }
.footer__legal { display: flex; flex-wrap: wrap; gap: var(--s-4) var(--s-6); justify-content: space-between;
                 margin-top: var(--s-8); padding-top: var(--s-5); border-top: 1px solid var(--line);
                 font-size: var(--t-label); color: var(--muted); }
.socials { display: flex; gap: var(--s-2); }
.socials a { display: inline-grid; place-items: center; width: 36px; height: 36px; border-radius: 999px; border: 1px solid var(--line); color: var(--ink); }
.socials a:hover { border-color: var(--line-strong); }
.socials svg, .socials img, .socials .icon-mono { width: var(--icon); height: var(--icon); }
/* a brand mark in the text colour (icons.mjs --mono prints these): the SVG is a mask, so it
   follows currentColor on every ground. The mask image is set in the tag's style attribute. */
.icon-mono { display: inline-block; flex: none; width: var(--icon); height: var(--icon); background: currentColor;
             -webkit-mask: center / contain no-repeat; mask: center / contain no-repeat; }
@media (max-width: 1023px) { .footer .footer__col { grid-column: auto / span 6; } .footer .grid > .span-4 { grid-column: 1 / -1; } }
@media (min-width: 640px) and (max-width: 1023px) { .footer .footer__col { grid-column: auto / span 3; } }

/* the company block's one line, set in the display face (footer-columns) */
.footer__lede { font: var(--display-weight) var(--t-h3)/1.25 var(--font-display); letter-spacing: var(--display-tracking); max-width: 22ch; }
/* the legal row's end: the socials sit right of the copyright line */
.footer__legal .socials a { width: 32px; height: 32px; }

/* footer types (dealt, sections.md).
   .footer--dark (Figma, x.ai): a dark ground (with .ground-inverse), the name set large top left, the
   link columns on the right. On a dark page it stays dark: a step darker than the page, a hairline
   on top (system.js sets <html data-tone="dark">), never a light slab under a dark page. */
.footer--dark { border-top: 0; padding-block: var(--section-band) var(--s-6); }   /* a change of ground: the gap a band gets */
html[data-tone="dark"] .footer--dark {
  --bg: color-mix(in oklab, var(--p-bg) 72%, #000); --surface: var(--p-surface); --ink: var(--p-ink); --muted: var(--p-muted);
  --accent: var(--p-accent); --accent-ink: var(--p-accent-ink); --action: var(--p-action); --action-ink: var(--p-action-ink);
  border-top: 1px solid var(--line);
}
.footer__mark { display: inline-flex; align-items: center; font: var(--display-weight) var(--t-h2)/1 var(--font-display); letter-spacing: var(--display-tracking); }
/* .footer--visit (a place): the address set large with the way there, the hours beside it, then
   one row of the nav's words; no third ask for the main action */
.footer__address { font: var(--display-weight) var(--t-h2)/1.1 var(--font-display); letter-spacing: var(--display-tracking); font-style: normal; }
.footer__nav { display: flex; flex-wrap: wrap; align-items: center; gap: var(--s-3) var(--s-6); margin-top: var(--s-8); padding-top: var(--s-5); border-top: 1px solid var(--line); font-weight: 500; }
.footer__nav a:hover { color: var(--action); }
.footer__nav .socials { margin-left: auto; }
.footer--visit .footer__legal { margin-top: var(--s-5); border-top: 0; padding-top: 0; }
/* .footer--statement: the close and the footer in one inset slab on the page's lit ground (sand,
   sky, sage or painting): one big line, one full-width button, then the columns */
.footer--statement {
  margin: 0; border: 0; border-radius: 0; padding-block: var(--s-10) var(--s-6);
  background: var(--ground-bg); color: var(--ground-ink);
  --bg: var(--ground-bg); --surface: var(--ground-surface); --ink: var(--ground-ink); --muted: var(--ground-muted);
  --action: var(--ground-ink); --action-ink: var(--ground-surface); --accent: var(--ground-accent);
  --line: color-mix(in oklab, var(--ground-ink) 14%, transparent); --line-strong: color-mix(in oklab, var(--ground-ink) 28%, transparent);
}
.footer--statement .cta-type { padding-bottom: var(--s-9); margin-bottom: var(--s-8); border-bottom: 1px solid var(--line); }
@media (max-width: 639px) { .footer--statement:not(.closing) { padding-top: var(--s-9); } }
/* opening hours: <dl class="hours" data-tz="Europe/Berlin">, one dt (its days: "Mon to Fri", "Sat and
   Sun", or data-days="mon-fri") and one dd a row. system.js marks today's row .is-today in the venue's
   time zone (never the visitor's clock); the labels are read, so they stay in ink */
.hours { display: grid; grid-template-columns: auto 1fr; gap: var(--s-2) var(--s-5); font-variant-numeric: tabular-nums; }
.hours dt { color: var(--ink); } .hours dd { margin: 0; }
.hours .is-today { font-weight: 500; }
.hours dt.is-today::after { content: ""; display: inline-block; width: 6px; height: 6px; margin-left: var(--s-2); border-radius: 999px; background: var(--accent); vertical-align: 0.2em; }
/* the visit block (a place, sections.md), on the container's lines. Open by default: the words on the page ground,
   no box (a box as wide as the section on a ground of its own reads as a band to the rhythm check, and a phone's
   padding moves its words off the left line). .visit--card is the panel on --g-alt for the full page, where it
   stands in a split column beside the client's photograph: from 1024px up only, at most half the container,
   48px inside; under 1024 it is open type again. .visit--cols is the short page's: address, hours and access as
   three columns, then the menu as .rows under it in the same section.
   Its head is the label "Visit" left and the open-now line right; under it the address column and the hours column 24
   apart; under both at most two 44px rows (access, a guesthouse's check-in, "By appointment").
   <section class="section" data-place="visit" aria-label="Visit"><div class="wrap"><div class="visit">
     <div class="visit__head"><p class="t-label muted">Visit</p><p class="t-small" data-open-now></p></div>
     <div class="visit__cols">
       <div class="visit__col stack-4"><address class="visit__address">Street<br>District</address>
         <a class="link" href="https://www.openstreetmap.org/search?query=..">Get directions</a></div>
       <div class="visit__col"><dl class="hours" data-tz="Europe/Berlin"><dt>Mon to Fri</dt><dd>8 to 18</dd></dl></div>
     </div>
     <ul class="visit__rows"><li><span>Access</span><span>Step-free at the front door</span></li></ul>
   </div></div></section> */
.visit { display: grid; gap: var(--s-5); color: var(--ink); }
.visit__head { display: flex; align-items: baseline; justify-content: space-between; gap: var(--s-4); flex-wrap: wrap; }
.visit__head > * { margin: 0; }
.visit__cols { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); column-gap: var(--s-5); row-gap: var(--s-5); }   /* the columns stretch to one height (row-lines) */
.visit--cols .visit__cols { grid-template-columns: repeat(3, minmax(0, 1fr)); }
.visit__col { min-width: 0; }
.visit__col > .t-label { margin-bottom: var(--s-3); color: var(--muted); }
.visit__address { font: 400 20px/30px var(--font-text); font-style: normal; }
.visit__rows { display: grid; margin: 0; padding: 0; list-style: none; }
.visit__rows > li { display: flex; align-items: center; justify-content: space-between; gap: var(--s-5); min-height: 44px; border-top: 1px solid var(--line); }
.visit__rows > li > :first-child { color: var(--muted); }
.visit__rows > li > :last-child { text-align: right; }
@media (min-width: 1024px) {
  .visit--card { --bg: var(--p-surface); --surface: var(--p-bg); --card: var(--p-bg); background: var(--bg); border-radius: var(--r-lg); padding: var(--s-7); }
}
@media (max-width: 1023px) { .visit--cols .visit__cols { grid-template-columns: repeat(2, minmax(0, 1fr)); } }
@media (max-width: 639px) {
  .visit__cols, .visit--cols .visit__cols { grid-template-columns: minmax(0, 1fr); }
  .visit__address { font-size: 18px; line-height: 28px; }
  .visit__rows > li { align-items: flex-start; flex-direction: column; justify-content: center; gap: 0; padding-block: var(--s-2); }
  .visit__rows > li > :last-child { text-align: left; }
}
/* photo credits: one short line that opens the full list (copy.md), at the end of the legal row:
   <details class="credits"><summary>Photographs via Unsplash and Wikimedia Commons</summary>
   <p>one line a photograph, as photos.mjs credits prints them</p></details> */
.credits { flex-basis: 100%; font-size: var(--t-label); color: var(--muted); }
.credits > summary { display: inline; cursor: pointer; list-style: none; }
.credits > summary::-webkit-details-marker { display: none; }
.credits > summary::after { content: "Credits"; margin-left: var(--s-2); color: var(--ink); text-decoration: underline; text-decoration-color: var(--line-strong); text-underline-offset: 3px; }
.credits[open] > summary::after { content: "Hide"; }
.credits > :not(summary) { margin-top: var(--s-2); max-width: 110ch; }
.credits a { text-decoration: underline; text-decoration-color: var(--line-strong); text-underline-offset: 3px; }

/* ---------------------------------------------------------------- footer.closing: the close and the footer, one screen
   The page ends on one more section: <footer class="footer closing [footer--dark | footer--statement |
   footer--visit]"> whose .wrap holds exactly two children, the close (.closing__cta, row 1, the
   rest of the screen) and the links (.closing__links, row 2: the columns, then the legal row). The
   close sits on the footer's own ground, so it is never a slab flush on a footer under it. Its three
   looks (dealt, sections.md):
     .close--photo  the h2 and the button on one head row, then the client's photograph across the
                    .wrap down to the links (never the hero's photograph)
     .close--type   the h2 and a line, then the big button across the .wrap (Figma's close)
     .close--word   the name set across the .wrap in the display face under the h2 and the button
   A phone stacks it in one screen; add .g2m when the links need a screen of their own. */
footer.closing { border-top: 0; }
footer.closing > .wrap { display: grid; grid-template-rows: minmax(0, 1fr) auto; row-gap: var(--s-8); }
footer.closing.g2 > .wrap { grid-template-rows: minmax(0, 1fr) minmax(0, 1fr); }
.closing__cta { min-height: 0; display: flex; flex-direction: column; gap: var(--s-6); }
.closing__cta > .section-head--split, .closing__cta > .section-head { margin-bottom: 0; }
.closing__cta h2 { max-width: 20ch; }
.closing__cta > .close__media { flex: 1 1 0; min-height: clamp(120px, 18svh, 160px); position: relative; overflow: clip; border-radius: var(--r-lg); background: var(--surface); }
.close__media > img { position: absolute; inset: 0; width: 100%; height: 100%; max-width: none; object-fit: cover; object-position: var(--pos, 50% 50%); }
.closing__cta.close--type > .btn--xl { align-self: stretch; }
/* the name across the .wrap: sized in layout.js-free CSS from the container (8 letters fill 1280) */
.close__word { margin-top: auto; font: var(--display-weight) clamp(64px, 14.5vw, 216px)/.86 var(--font-display); letter-spacing: -0.045em;
               white-space: nowrap; overflow: clip; color: var(--ink); }
.closing__links { display: grid; gap: var(--s-7); }
.closing__links > .footer__legal { margin-top: 0; }
.closing__links .footer__lede { font-size: var(--t-lead); max-width: 30ch; }
/* a phone: a close with a photograph takes a screen of its own (the photograph fills what the words
   leave) and the links the next; a close in type stays one screen with its links */
@media (max-width: 639px) {
  footer.closing:has(> .wrap > .closing__cta > .close__media) { --hm: 2; }
  footer.closing:has(> .wrap > .closing__cta > .close__media) > .wrap { grid-template-rows: minmax(0, 1fr) minmax(0, 1fr); row-gap: calc(var(--padt) + var(--padb) + var(--bar)); }
  footer.closing:has(> .wrap > .closing__cta > .close__media) .closing__links { align-self: start; }
}
@media (max-width: 639px) {
  footer.closing > .wrap { row-gap: var(--s-7); }
  .closing__links { gap: var(--s-5); }
  .closing__links > .grid { row-gap: var(--s-5); }
  .closing__links .footer__col { grid-column: auto / span 6; }
  .closing__cta > .close__media { min-height: 140px; }
  .close__word { font-size: clamp(56px, 19vw, 96px); }
}

/* ---------------------------------------------------------------- the big close (closing-type) */
/* the close starts on the left line like every other heading; the big button runs across the .wrap */
.cta-type { display: grid; justify-items: start; gap: var(--s-6); text-align: left; }
.cta-type h2 { max-width: 22ch; }
/* the primary button at its one large size, once a page: 640px wide and centred; .btn--wide runs it
   across the container (Figma's close), on a software page only (system.md) */
.btn--xl { width: min(100%, 640px); height: clamp(64px, 1.4vw + 48px, 72px); font: var(--display-weight) clamp(20px, 0.6vw + 16px, 26px)/1 var(--font-display); letter-spacing: var(--display-tracking); }
.btn--xl.btn--wide { width: 100%; height: clamp(64px, 3vw + 40px, 88px); font-size: clamp(22px, 0.95vw + 18px, 32px); justify-content: space-between; padding-inline: var(--s-7); }
html[data-buttons] .btn--xl::after { width: 0.6em; height: 0.6em; }
/* on a phone it is the page's own button at full width, one step taller, so the pair stays one look */
@media (max-width: 639px) { .btn--xl, .btn--xl.btn--wide { width: 100%; height: 48px; font: 500 18px/1 var(--font-text); letter-spacing: 0; } }

/* ---------------------------------------------------------------- photographs
   .photo-band: one photograph (or film) and a few words. Three uses: the older photo hero
   (.photo-band--hero.photo-band--bleed), one media band mid-page (rhythm media-band) and the close
   (closing-photo). Nothing is laid over the photograph to make words readable: no scrim, gradient,
   veil or blurred panel (PH-31, VI-49). From 640 the words stand in a solid box in the section's own
   colours, lower left on the photograph (.photo-band--left: centred on the left side; --light is the
   same box, kept for old markup). On a phone, and on a place at every width (a place photograph
   carries no words), the photograph stands alone and the words go under it on the ground. */
.photo-band { position: relative; isolation: isolate; overflow: clip; display: grid; align-items: end;
              min-height: clamp(440px, 46vw, 720px); border-radius: var(--r-lg); background: var(--surface); }
.photo-band--bleed { border-radius: 0; }
.photo-band--hero { min-height: calc(100svh - var(--nav-h)); }
/* the older photo hero first in main (layout/hero-photo.html replaces it): the first screen, under the fixed header */
main > .photo-band--hero:first-child { height: 100svh; min-height: 0; }
main > .photo-band--hero:first-child > .photo-band__text { padding-top: calc(var(--nav) + var(--hgap)); }
.photo-band > :is(img, video, picture) { position: absolute; inset: 0; z-index: -2; width: 100%; height: 100%; max-width: none; object-fit: cover; }
.photo-band > picture > img { width: 100%; height: 100%; object-fit: cover; }
.photo-band__text { position: relative; display: grid; gap: var(--s-4); justify-items: start; color: var(--ink); }
.photo-band__text > :is(h1, h2, p) { max-width: 18ch; }
.photo-band__text > p { max-width: 44ch; font-size: var(--t-lead); }
/* from 640: a solid box in the section's colours, 32px in from the photograph's lower-left corner;
   on a full-bleed band it starts on the page's left line */
@media (min-width: 640px) {
  :root:not([data-type="place"]) .photo-band__text {
    width: fit-content; max-width: min(560px, calc(100% - 2 * var(--s-6))); margin: var(--s-6);
    padding: var(--s-6); background: var(--bg); border-radius: var(--r-md);
  }
  :root:not([data-type="place"]) .photo-band--bleed > .photo-band__text { margin: var(--s-8) max(var(--gut), (100% - var(--container)) / 2); }
  :root:not([data-type="place"]) main > .photo-band--hero:first-child > .photo-band__text { margin-top: calc(var(--nav) + var(--hgap)); padding-top: var(--s-6); }
  .photo-band--left { align-items: center; }
}
@media (max-width: 639px) {
  .photo-band { display: block; min-height: 0; overflow: visible; background: none; border-radius: 0; }
  .photo-band > :is(img, video, picture) { position: relative; inset: auto; height: auto; aspect-ratio: 4 / 5; border-radius: var(--r-lg); }
  .photo-band--bleed > :is(img, video, picture) { border-radius: 0; }
  .photo-band__text { padding: var(--s-5) 0 0; }
  .photo-band--bleed > .photo-band__text { padding: var(--s-5) var(--pad) 0; }
  .photo-band--hero > .photo-band__text { padding-bottom: var(--section); }
  /* the hero's photograph grows to fill the first screen above its words */
  .photo-band--hero { display: flex; flex-direction: column; min-height: calc(100svh - var(--nav-h)); }
  .photo-band--hero > :is(img, video, picture) { flex: 1 0 0; min-height: 100vw; aspect-ratio: auto; object-fit: cover; }
}
:root[data-type="place"] .photo-band { display: flex; flex-direction: column; min-height: 0; overflow: visible; background: none; border-radius: 0; }
:root[data-type="place"] .photo-band > :is(img, video, picture) { position: relative; inset: auto; flex: 1 1 auto; min-height: 0; height: auto; aspect-ratio: 4 / 3; }
:root[data-type="place"] .photo-band__text { padding: var(--s-6) 0 0; }
:root[data-type="place"] .photo-band--bleed > .photo-band__text { padding-inline: max(var(--gut), (100% - var(--container)) / 2); }

/* .pl-photo: every single in-flow photograph on a place (visit, menu, story, booking, close; PH-80).
   It keeps its ratio (--pl-r as a number: 1.3333 for 4:3, .8 for 4:5, 1 for 1:1) and its column, and
   when the screen is short it gets shorter, narrowing from its column's outer edge (.pl-photo--end
   for a photograph in the right-hand column). Square corners, nothing on it.
     <figure class="pl-photo" style="--pl-r: .8"><img src=".." alt=".." loading="lazy"></figure> */
:root { --pl-room: calc(100svh - var(--padt) - var(--padb) - var(--bar)); }
.pl-photo { container-type: inline-size; margin: 0; min-width: 0; }
.pl-photo > img, .pl-photo > picture > img {
  display: block; aspect-ratio: var(--pl-r, 1.3333); width: auto; max-width: 100%;
  height: min(calc(100cqw / var(--pl-r, 1.3333)), var(--pl-cap, calc(var(--pl-room) - var(--pl-head, 0px))));
  object-fit: cover; object-position: var(--pos, 50% 50%); border-radius: var(--r-lg);
}
.pl-photo--end > img, .pl-photo--end > picture > img { margin-left: auto; }
@media (max-width: 1023px) { .pl-photo > img, .pl-photo > picture > img { width: 100%; height: auto; } }

/* .tile: an open tile, the picture on top and its words on the page ground under it, no box
   (a room, a model, a person, a case). One --tile-ratio per row so the pictures line up. */
.tile { display: grid; gap: var(--s-4); align-content: start; color: inherit; }
.tile__media { position: relative; overflow: clip; aspect-ratio: var(--tile-ratio, 4 / 3); border-radius: var(--r-lg); background: var(--surface); }
.tile__media > img { width: 100%; height: 100%; object-fit: cover; transition: transform 600ms var(--ease); }
a.tile:hover .tile__media > img { transform: scale(1.03); }   /* inside its mask; the tile never grows (motion.css has the same) */
.tile--portrait { --tile-ratio: 4 / 5; }
.tile--wide { --tile-ratio: 16 / 9; }
.tile__head { display: flex; align-items: baseline; justify-content: space-between; gap: var(--s-4); }
.tile__head > :is(h3, h4) { font: 500 var(--t-card)/1.3 var(--font-text); letter-spacing: -0.01em; }
.tile__price { flex: none; font-variant-numeric: tabular-nums; white-space: nowrap; }
.tile__facts { color: var(--muted); font-size: var(--t-small); margin-top: calc(-1 * var(--s-3)); }
.tile > p { color: var(--ink); max-width: 52ch; }
/* an object render or a drawing in a tile (line-up): on the lit ground, the object filling the tile */
.tile__media:has(> [data-print], > .drawing) { display: grid; place-items: center; background: var(--ground-bg); }
.tile__media > [data-print] { position: absolute; inset: 0; }
.tile__media > .drawing { width: min(84%, calc(84% * var(--ar, 1.5))); }

/* .drawing: a product with no photograph and no object of its kind, drawn (sections.md, the weak
   case): one inline SVG in true proportions, its lines a fixed 1.5px at every size, one part in the
   accent, dimension lines with their figures as HTML labels placed in percent of the figure.
   <figure class="drawing" style="--ar: 1.6" role="img" aria-label="..">
     <svg viewBox="0 0 1600 1000" aria-hidden="true">
       <g class="part"><path class="fill" d=".."/><path class="ln" d=".."/></g>
       <g class="dim"><path d="M100 960H1500"/><path d="M100 945v30M1500 945v30"/></g>
     </svg>
     <span class="drawing__dim" style="left: 50%; top: 96%">1,400 mm</span>
   </figure>
   In a frame it fits the frame whole (--ar: the viewBox's width over its height); a crop for a
   feature section is the same SVG with the viewBox set round the part. */
.drawing { position: relative; margin: 0; flex: none; aspect-ratio: var(--ar, 1.5); color: var(--ink); }
.frame > .drawing { width: min(100%, calc(var(--ratio, 50%) * var(--ar, 1.5) * 0.86)); }
/* a crop (a feature section's close-up of one part): it fills the frame and runs out of its edges,
   <svg preserveAspectRatio="xMidYMid slice">, no figures on it */
.frame > .drawing.drawing--crop { position: absolute; inset: 0; width: 100%; height: 100%; aspect-ratio: auto; }
.drawing > svg { display: block; width: 100%; height: 100%; overflow: visible; }
.drawing :is(.ln, .fill, .acc, .dim) { vector-effect: non-scaling-stroke; stroke-linecap: round; stroke-linejoin: round; }
.drawing .ln { fill: none; stroke: currentColor; stroke-width: 1.5; }
.drawing .fill { fill: color-mix(in oklab, var(--ink) 6%, var(--bg)); stroke: currentColor; stroke-width: 1.5; }
.drawing .acc { fill: var(--accent); stroke: currentColor; stroke-width: 1.5; }
/* a dark part (a black steel frame, a rubber foot) solid in ink, its seams .ink-ln */
.drawing .ink { fill: var(--ink); stroke: var(--ink); stroke-width: 1.5; }
.drawing .ink-ln { fill: none; stroke: color-mix(in oklab, var(--bg) 45%, var(--ink)); stroke-width: 1; vector-effect: non-scaling-stroke; }
.drawing .dim, .drawing .dim * { fill: none; stroke: var(--muted); stroke-width: 1; }
.drawing .part { pointer-events: bounding-box; }
.drawing .part :is(.ln, .fill, .ink, .acc) { transition: stroke var(--dur) var(--ease), fill var(--dur) var(--ease); }
@media (hover: hover) {
  .drawing .part:hover :is(.ln, .fill) { stroke: var(--accent); }
  .drawing .part:hover .fill { fill: color-mix(in oklab, var(--accent) 14%, var(--bg)); }
  .drawing .part:hover .ink { fill: var(--accent); stroke: var(--accent); }
}
.drawing__dim { position: absolute; transform: translate(-50%, -50%); padding: 2px var(--s-2); border-radius: 999px;
  background: var(--bg); color: var(--ink); font: 500 var(--t-label)/1.3 var(--font-text); font-variant-numeric: tabular-nums; white-space: nowrap; }
@media (max-width: 639px) {
  .drawing__dim { font-size: var(--t-label); padding: 1px 6px; }
  /* room for a figure at the drawing's edge, which keeps its 13px while the drawing shrinks */
  .frame > .drawing:not(.drawing--crop) { width: min(84%, calc(var(--ratio, 50%) * var(--ar, 1.5) * 0.8)); }
  /* a whole drawing (wider than tall) in a phone frame: the frame shortens to it, so a desk or a bike is
     not a small line across a tall empty box (a hero stage still grows to the first screen) */
  .frame:not(:has(> .frame__text)):has(> .drawing:not(.drawing--crop)) { --ratio: 80%; }
}

/* .peek: a list beside one photograph that changes to the row under the pointer (a place's or a
   service's one moment: rooms, spaces, classes, services, the menu's kinds). system.js runs it:
   <div class="grid grid--middle" data-peek>
     <div class="frame frame--photo frame--portrait span-5">
       <img class="frame__img" data-peek-for="garden" src=".." alt="..">
       <img class="frame__img" data-peek-for="kitchen" src=".." alt=".." hidden> ...</div>
     <ul class="rows span-6 start-7"><li data-peek-to="garden">..</li><li data-peek-to="kitchen">..</li></ul>
   </div> */
/* every row's words stay on the left line; the row on show darkens its rule to ink and marks its name
   with a dot after it. The photograph swaps at once (MO-07). The list runs as tall as the photograph
   (the row's two boxes match), its rows from the top. */
[data-peek] > .rows { align-self: stretch; }
[data-peek-to] { cursor: default; transition: border-color var(--dur) var(--ease); }
[data-peek-to].is-on { border-bottom-color: var(--ink); }
[data-peek-to] .rows__name::after { content: ""; display: inline-block; width: 8px; height: 8px; margin-left: var(--s-2); vertical-align: .1em;
  border-radius: 999px; background: var(--accent); opacity: 0; }
[data-peek-to].is-on .rows__name::after { opacity: 1; }
@media (max-width: 1023px) { [data-peek-to] .rows__name::after { display: none; } }
/* a tablet shows the photograph under the list, full width: 5:4, not a portrait taller than the screen */
@media (min-width: 640px) and (max-width: 1023px) { [data-peek] .frame--portrait { --ratio: 80%; } }

/* .rows: a list read across, a name (and a line) left, a value right, hairlines between: a menu, a
   price list, services with prices, specs, hours. Never cards for these. */
.rows { border-top: 0; }   /* each row closes with its own rule; no rule above the first (ST-43) */
.rows > li { display: grid; grid-template-columns: minmax(0, 1fr) auto; gap: var(--s-1) var(--s-5); padding-block: var(--s-4); border-bottom: 1px solid var(--line); }
/* no rule above the first row, so no padding either: its words start on the line beside it (a photograph's top) */
.rows > li:first-child { padding-top: 0; }
.rows__name { font-weight: 500; }
.rows__value { grid-column: 2; grid-row: 1; text-align: right; font-variant-numeric: tabular-nums; white-space: nowrap; }
.rows > li > p { grid-column: 1; color: var(--ink); max-width: 56ch; }
.rows--spec .rows__name { font-weight: 400; color: var(--muted); }

/* forms (booking-form): fields on the system's control height, one radius, labels above */
.form { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: var(--s-4); }
.form > :is(.field--full, .btn-row, p) { grid-column: 1 / -1; }
.field { display: grid; gap: var(--s-2); font-size: var(--t-small); font-weight: 500; }
.field :is(input, select, textarea) {
  width: 100%; height: var(--btn-h); padding: 0 var(--s-3); border: 1px solid var(--line-strong); border-radius: var(--r-sm);
  background: var(--bg); color: var(--ink); font: 400 var(--t-body)/1.4 var(--font-text);
}
.field textarea { height: auto; min-height: 120px; padding-block: var(--s-3); resize: vertical; }
@media (max-width: 639px) { .form { grid-template-columns: minmax(0, 1fr); } }

/* sections part by a change of ground or by space only: no rule between them (.section--rule is plain) */

/* the phone action bar (a place's directions and booking; other types may use it): fixed at the foot
   under 640px, one 342x48 button or two of 165, and every section's foot pads for it (--bar) */
.actionbar { display: none; }
@media (max-width: 639px) {
  :root:has(> body > nav.actionbar) { --bar: calc(64px + env(safe-area-inset-bottom)); }
  .actionbar {
    position: fixed; left: 0; right: 0; bottom: 0; z-index: 40; display: grid; grid-auto-flow: column; grid-auto-columns: minmax(0, 1fr);
    gap: var(--s-3); align-items: center; height: calc(64px + env(safe-area-inset-bottom));
    padding: var(--s-2) var(--gut) env(safe-area-inset-bottom); background: var(--bg); border-top: 1px solid var(--line);
  }
  .actionbar .btn { width: 100%; height: 48px; }
}

/* a brand-colour band (product and place may add one, in the page's second half): the accent as the
   section's ground, its words in the accent's ink */
.ground-brand { --bg: var(--p-accent); --surface: color-mix(in oklab, var(--p-accent) 88%, var(--p-accent-ink)); --ink: var(--p-accent-ink);
  --muted: color-mix(in oklab, var(--p-accent-ink) 82%, var(--p-accent)); --card: var(--surface);
  --action: var(--p-accent-ink); --action-ink: var(--p-accent); background: var(--bg); color: var(--ink); }
.ground-brand { --line: color-mix(in oklab, var(--ink) 16%, transparent); --line-strong: color-mix(in oklab, var(--ink) 30%, transparent); }

/* a [bracketed placeholder] or a "(sample)" tag left in visible text: system.js marks it so it shows
   in every screenshot; nothing like it ships (copy.md) */
[data-placeholder] { outline: 2px dashed #d92d20; outline-offset: 2px; }

/* ---------------------------------------------------------------- highlight + entrance + motion */
/* data-hl (older markup) is plain ink from the first frame: nothing on the page changes colour on load
   (MO-62); a heading's second half in .muted is the one highlight */
/* the hero's words are set on the first frame: no entrance (.enter stays a no-op for old markup) */
.enter { animation: none; }
@keyframes fade { from { opacity: 0; } }
@media (prefers-reduced-motion: reduce) {
  /* a card loop of motion.css ([data-m-loop]) holds its own finished frame */
  :not([data-m-loop], [data-m-loop] *), ::before, ::after { animation-duration: .001ms !important; animation-iteration-count: 1 !important;
                           transition-duration: .001ms !important; scroll-behavior: auto !important; }
}

/* ---------------------------------------------------------------- helpers */
.sr-only { position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap; border: 0; }
.stack-2 > * + * { margin-top: var(--s-2); }
.stack-4 > * + * { margin-top: var(--s-4); }
.stack-5 > * + * { margin-top: var(--s-5); }
.stack-6 > * + * { margin-top: var(--s-6); }
```

## assets/system.js

```js
/*
  taste-skill system.js - the page behaviour every build shares. Load it once, at the end of <body>:
    <script type="module" src="assets/system.js"></script>
  Write no second script for any of this.

  - nav: hover menus are CSS; this adds the phone sheet, the accordion, Escape and aria-expanded, and
    closes the sheet after a tap on any link in it.
  - tabs: [role=tablist] with [role=tab][aria-controls] and [role=tabpanel]. aria-controls may name
    several ids (the tab's text and its screen in a frame); they are shown and hidden together.
  - text highlight: none. <p data-hl> (older markup) stays plain ink from the first frame; nothing
    changes colour on load (MO-62).
  - FAQ: every .faq answer is open (ST-47): a <details> in .faq is opened and stays open.
  - opening hours: <dl class="hours" data-tz="Europe/Berlin"> marks today's dt and dd .is-today in the
    venue's time zone (each dt names its days, "Mon to Fri", "Sat and Sun", "Daily", or carries
    data-days="mon-fri" / "sat,sun"). Without data-tz no row is marked: never the visitor's clock,
    and an .is-today written in the markup is taken out (it would be wrong six days a week). A
    [data-open-now] line reads "Open now, until 16:00" or "Closed, opens Tuesday at 7:30" from them.
  - moments: a page with [data-moment] loads assets/moments/moments.js (references/moments.md), which
    mounts the dealt moments once; data-ready waits for them.
  - peek: [data-peek] holds rows [data-peek-to="key"] and photographs [data-peek-for="key"] (all but
    the first hidden); the photograph changes at once (MO-07) to the row under the pointer, the
    keyboard focus or a tap. A place's or a service's one moment (references/sections.md).
  - visuals (assets/effects/fx.js and assets/ui/ui.js are loaded only when the page has any):
    * mounts every [data-fx] ground; only a ground inside the hero (.hero) moves, and not when its frame
      holds an object or a scene ([data-print], [data-scene], assets/objects). Every other one is
      drawn once when it nears the screen, kept as a plain picture and its WebGL context freed
      (ui.grounds), so a page holds at most
      3 live WebGL contexts (4 while an object moves on screen); a ground a fragment drives (the
      slider) stays live near the screen. The featured plan's lit border (data-glow="edge-glow",
      data-glow-params) rests on a quiet hairline and runs one light round its edge on hover or
      focus;
    * lights every ground that holds UI (a .panel, .panel__screen or .frame) and the closing band's
      (.frame--cta > .frame__fx) with the page's lit ground: <html data-ground="sand|sky|sage|painting">
      (system.css --ground-*), on light and dark pages alike; without data-ground the page palette
      picks one. data-fx-palette on the element wins; --hue on it changes only its accent;
    * starts every pasted [data-ui] fragment (assets/ui/<name>/fragment.html, its names, numbers and
      logos rewritten to the client's) and loads every [data-ui-load] one (the kit's sample text: a
      draft only, never shipped). Each holds its
      finished picture (system.js marks it data-ui-still, so ui.js holds it the moment its own load
      resolves) unless it, or an element around it, carries data-ui-loop (one per card group) or
      data-ui-cycles. On a phone (under 640px), on kind other (<meta name="site-kind"
      content="other">) and on a place nothing loops, voice included, and the hero's ground is drawn
      once: every fragment holds its finished picture and plays only while the pointer is on its card;
    * pause buttons: ui.js puts one on every loop when it starts to move (a 32px disc 12px in from
      the bottom-right corner of its panel or frame); system.js gives the hero's moving ground the
      same button, shared with a loop in that frame. A box that already holds a .loop-pause (a loop
      the page runs itself) gets none;
    * sets <html data-ready="1"> when all of it is drawn (shoot.mjs and consistency.mjs wait for it).
  - page tone: <html data-tone="dark|light"> from the palette's base (a dark footer stays dark).
  - pictures: lazy images are fetched once the page has loaded, so no photograph shows as an empty
    frame; one that fails to load is outlined in red and counted (<html data-broken>).
  - guards: a [bracketed placeholder], a "(sample)" tag, a line that explains or apologises ("Names
    and figures on this page are examples") or a printed .example address gets a red dashed outline
    and <html data-placeholders="n">; an effect id in more than two sections sets <html data-fx-repeat>.
*/
const reduced = () => window.matchMedia('(prefers-reduced-motion: reduce)').matches;
const HERE = new URL('./', import.meta.url);
/* kind other (a cafe, a shop, a studio, a service firm) and a place: nothing moves by itself (MO-07) */
const kindOther = () => {
  const m = document.querySelector('meta[name="site-kind"]');
  return document.documentElement.dataset.type === 'place' || (!!m && m.content.trim().toLowerCase() === 'other');
};

function nav() {
  const nav = document.querySelector('.nav');
  if (!nav) return;
  const toggle = nav.querySelector('.nav__toggle');
  const bars = '<svg width="18" height="18" viewBox="0 0 18 18" aria-hidden="true"><path d="M2 5h14M2 13h14" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/></svg>';
  const cross = '<svg width="18" height="18" viewBox="0 0 18 18" aria-hidden="true"><path d="M4 4l10 10M14 4L4 14" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/></svg>';
  const setOpen = open => {
    nav.classList.toggle('is-open', open);
    document.body.classList.toggle('nav-locked', open);
    if (toggle) { toggle.setAttribute('aria-expanded', String(open)); toggle.innerHTML = open ? cross : bars; }
  };
  toggle?.addEventListener('click', () => setOpen(!nav.classList.contains('is-open')));
  nav.querySelectorAll('.nav__item').forEach(item => {
    if (!item.querySelector('.nav__menu')) return;
    let btn = item.querySelector('.nav__expand');
    if (!btn) {
      btn = document.createElement('button');
      btn.className = 'nav__expand';
      btn.type = 'button';
      btn.innerHTML = '<svg viewBox="0 0 16 16" aria-hidden="true"><path d="M4 6l4 4 4-4" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>';
      item.querySelector('.nav__link').after(btn);
    }
    const label = item.querySelector('.nav__link')?.textContent.trim() || 'section';
    btn.setAttribute('aria-label', `Show ${label} links`);
    btn.setAttribute('aria-expanded', 'false');
    btn.addEventListener('click', () => {
      const open = !item.classList.contains('is-expanded');
      item.classList.toggle('is-expanded', open);
      btn.setAttribute('aria-expanded', String(open));
    });
  });
  document.addEventListener('keydown', e => {
    if (e.key !== 'Escape') return;
    if (nav.classList.contains('is-open')) { setOpen(false); toggle?.focus(); return; }
    if (nav.contains(document.activeElement)) document.activeElement.blur();
  });
  // a tap on any link in the open sheet closes it (a top-level word, a menu link, an action, an
  // in-page #anchor alike), so the page it leads to is never left under the sheet
  nav.addEventListener('click', e => {
    if (nav.classList.contains('is-open') && e.target.closest?.('a[href]')) setOpen(false);
  });
  window.matchMedia('(min-width: 1024px)').addEventListener('change', e => { if (e.matches) setOpen(false); });
}

function tabs() {
  const parts = t => (t.getAttribute('aria-controls') || '').split(/\s+/).filter(Boolean).map(id => document.getElementById(id)).filter(Boolean);
  document.querySelectorAll('[role="tablist"]').forEach(list => {
    const tabs = [...list.querySelectorAll('[role="tab"]')];
    let first = true;
    const select = (tab, focus) => {
      tabs.forEach(t => {
        const on = t === tab;
        t.setAttribute('aria-selected', String(on));
        t.tabIndex = on ? 0 : -1;
        parts(t).forEach(el => {
          const was = el.hidden;
          el.hidden = !on;
          // a panel or a screen in a frame fades in on a chosen tab only (opacity only, nothing moves;
          // the panel shown at load is there on the first frame, MO-62)
          if (on && was && !first && !reduced()) el.animate([{ opacity: 0 }, { opacity: 1 }], { duration: 280, easing: 'ease-out' });
        });
      });
      if (focus) tab.focus();
    };
    tabs.forEach((t, i) => {
      t.addEventListener('click', () => select(t));
      t.addEventListener('keydown', e => {
        const d = e.key === 'ArrowRight' || e.key === 'ArrowDown' ? 1 : e.key === 'ArrowLeft' || e.key === 'ArrowUp' ? -1 : 0;
        if (!d) return;
        e.preventDefault();
        select(tabs[(i + d + tabs.length) % tabs.length], true);
      });
    });
    select(tabs.find(t => t.getAttribute('aria-selected') === 'true') || tabs[0]);
    first = false;
  });
}

/* every FAQ answer open, for good: a <details> in .faq is opened and its summary no longer closes it */
function faq() {
  document.querySelectorAll('.faq details').forEach(d => {
    d.open = true;
    d.addEventListener('toggle', () => { if (!d.open) d.open = true; });
    const s = d.querySelector(':scope > summary');
    if (s) s.addEventListener('click', e => e.preventDefault());
  });
}

/* ------------------------------------------------------------------ opening hours
   today's row in the venue's time zone, from data-tz on the dl; never the visitor's clock */
const DAYS = ['sun', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat'];
function daySet(text) {
  const t = String(text || '').toLowerCase().replace(/\bweekdays\b/g, 'mon to fri').replace(/\bweekends?\b/g, 'sat and sun');
  if (/\b(daily|every day|all week|mon(day)?\s*(?:-|–|to)\s*sun(day)?)\b/.test(t)) return new Set(DAYS);
  const hits = [...t.matchAll(/\b(sun|mon|tue|wed|thu|fri|sat)[a-z]*\.?/g)];
  if (!hits.length) return null;
  const set = new Set();
  for (let i = 0; i < hits.length; i++) {
    const a = DAYS.indexOf(hits[i][1]);
    const next = hits[i + 1];
    const between = next ? t.slice(hits[i].index + hits[i][0].length, next.index) : '';
    if (next && /^\s*(?:-|–|to|till|until|through)\s*$/.test(between)) {
      const b = DAYS.indexOf(next[1]);
      for (let d = a; ; d = (d + 1) % 7) { set.add(DAYS[d]); if (d === b) break; }
      i++;
    } else set.add(DAYS[a]);
  }
  return set;
}
function hours() {
  document.querySelectorAll('dl.hours').forEach(dl => {
    dl.querySelectorAll('.is-today').forEach(el => el.classList.remove('is-today'));
    const tz = dl.dataset.tz;
    if (!tz) return;
    let today;
    try { today = new Intl.DateTimeFormat('en-US', { weekday: 'short', timeZone: tz }).format(new Date()).slice(0, 3).toLowerCase(); }
    catch { console.warn(`system.js: hours: unknown time zone ${tz}`); return; }
    for (const dt of dl.querySelectorAll('dt')) {
      const days = daySet(dt.dataset.days ? dt.dataset.days.replace(/,/g, ' and ') : dt.textContent);
      if (!days?.has(today)) continue;
      dt.classList.add('is-today');
      for (let dd = dt.nextElementSibling; dd && dd.tagName === 'DD'; dd = dd.nextElementSibling) dd.classList.add('is-today');
      break;
    }
  });
  openNow();
}
/* [data-open-now] (a line in the visit block or the hero's notice) reads "Open now, until 16:00" or
   "Closed, opens Tuesday at 7:30" from the first dl.hours with data-tz, in the venue's time zone; its
   own text stays when the hours cannot be read (no data-tz, a row without times) */
const DAY_NAMES = { sun: 'Sunday', mon: 'Monday', tue: 'Tuesday', wed: 'Wednesday', thu: 'Thursday', fri: 'Friday', sat: 'Saturday' };
function openNow() {
  const outs = document.querySelectorAll('[data-open-now]');
  const dl = document.querySelector('dl.hours[data-tz]');
  if (!outs.length || !dl) return;
  let parts;
  try {
    parts = Object.fromEntries(new Intl.DateTimeFormat('en-US', { weekday: 'short', hour: '2-digit', minute: '2-digit', hourCycle: 'h23', timeZone: dl.dataset.tz })
      .formatToParts(new Date()).map(p => [p.type, p.value]));
  } catch { return; }
  const today = DAYS.indexOf(parts.weekday.slice(0, 3).toLowerCase());
  const nowMin = (+parts.hour % 24) * 60 + +parts.minute;
  const span = day => {
    for (const dt of dl.querySelectorAll('dt')) {
      const days = daySet(dt.dataset.days ? dt.dataset.days.replace(/,/g, ' and ') : dt.textContent);
      if (!days?.has(DAYS[day])) continue;
      const dd = dt.nextElementSibling;
      const m = dd && dd.tagName === 'DD' && dd.textContent.match(/(\d{1,2})(?:[:.](\d{2}))?\s*(?:to|-|–|until)\s*(\d{1,2})(?:[:.](\d{2}))?/);
      return m ? { open: +m[1] * 60 + (+m[2] || 0), close: +m[3] * 60 + (+m[4] || 0), from: m[1] + (m[2] ? ':' + m[2] : ''), to: m[3] + (m[4] ? ':' + m[4] : ':00') } : { closed: true };
    }
    return null;
  };
  const t = span(today);
  let text = null;
  if (t && !t.closed && nowMin >= t.open && nowMin < t.close) text = `Open now, until ${t.to}`;
  else {
    for (let k = 0; k < 8 && !text; k++) {
      const d = (today + k) % 7, s2 = span(d);
      if (!s2 || s2.closed || (k === 0 && nowMin >= s2.open)) continue;
      text = `Closed, opens ${k === 0 ? 'today' : k === 1 ? 'tomorrow' : DAY_NAMES[DAYS[d]]} at ${s2.from}`;
    }
  }
  if (text) outs.forEach(o => { o.textContent = text; });
}

/* ------------------------------------------------------------------ peek
   a list beside one photograph that changes to the row under the pointer, the focus or a tap */
function peek() {
  document.querySelectorAll('[data-peek]').forEach(root => {
    const rows = [...root.querySelectorAll('[data-peek-to]')];
    const pics = [...root.querySelectorAll('[data-peek-for]')];
    if (!rows.length || !pics.length) return;
    let cur = null;
    // the swap is instant (MO-07): the row's photograph shows, the others hide, nothing fades
    const show = row => {
      const pic = pics.find(p => p.dataset.peekFor === row.dataset.peekTo);
      if (!pic || row === cur) return;
      cur = row;
      // the row on show is marked beside its photograph only (from 1024); stacked, the list stays plain
      const wide = matchMedia('(min-width: 1024px)').matches;
      rows.forEach(r => r.classList.toggle('is-on', wide && r === row));
      pics.forEach(p => { p.hidden = p !== pic; });
    };
    // every photograph is fetched now, so a swap never shows an empty frame
    pics.forEach(p => { if (p.loading === 'lazy') p.loading = 'eager'; });
    show(rows.find(r => pics.some(p => !p.hidden && p.dataset.peekFor === r.dataset.peekTo)) || rows[0]);
    rows.forEach(r => {
      r.addEventListener('pointerenter', e => { if (e.pointerType === 'mouse') show(r); });
      r.addEventListener('focusin', () => show(r));
      r.addEventListener('click', e => { if (!e.target.closest('a, button, input, select, textarea')) show(r); });
    });
  });
}

/* ------------------------------------------------------------------ visuals */
let probe;
function rgb(value) {
  value = (value || '').trim();
  if (!value) return null;
  probe = probe || document.createElement('canvas').getContext('2d', { willReadFrequently: true });
  probe.canvas.width = probe.canvas.height = 1;
  probe.clearRect(0, 0, 1, 1);
  probe.fillStyle = '#000';
  probe.fillStyle = value;
  probe.fillRect(0, 0, 1, 1);
  const d = probe.getImageData(0, 0, 1, 1).data;
  return d[3] ? [d[0], d[1], d[2]] : null;
}
const hex = c => '#' + c.map(v => Math.round(v).toString(16).padStart(2, '0')).join('');

// The page's lit ground (references/cards.md: sand, sky, sage or painting, <html data-ground>, picked at
// random from the three on a tech page; a page of kind other has none, it is flat). Without data-ground the
// page palette picks one, so two sites never share it by default.
const GROUNDS = ['sand', 'sky', 'sage'];
function groundTokens() {
  const html = document.documentElement;
  if (!html.dataset.ground) {
    const first = document.querySelector('.card [data-fx], .frame [data-fx], .frame[data-fx]');
    const root = getComputedStyle(html);
    const key = `${root.getPropertyValue('--p-bg')}${root.getPropertyValue('--p-accent')}`.replace(/\s/g, '');
    let h = 0;
    for (const c of key) h = (h * 31 + c.charCodeAt(0)) >>> 0;
    html.dataset.ground = first?.dataset.fx === 'painterly-landscape' ? 'painting' : GROUNDS[h % GROUNDS.length];
  }
  const cs = getComputedStyle(html);
  const t = k => cs.getPropertyValue(`--ground-${k}`).trim();
  return { bg: t('bg'), surface: t('surface'), ink: t('ink'), muted: t('muted'), accent: t('accent') };
}
// every ground that holds UI, and the closing band's, takes the lit ground on light and dark pages
// alike: clearly lighter than a dark card, a clear step darker than white UI, so page, card, panel and
// UI are never four near-black layers. data-fx-palette on the element wins; --hue changes its accent.
function lightGrounds() {
  const g = groundTokens();
  document.querySelectorAll('[data-fx]').forEach(el => {
    if (el.dataset.fxPalette) return;
    const band = el.matches('.frame--cta > .frame__fx');
    if (!band && !el.matches('.panel, .panel__screen, .frame')) return;
    const hue = rgb(getComputedStyle(el).getPropertyValue('--hue'));
    el.dataset.fxPalette = JSON.stringify({ ...g, ...(hue ? { accent: hex(hue) } : {}) });
    // the box itself takes the lit colour too: an effect that keeps its host's own background (a
    // frame's dark surface on a dark page) then still draws on the lit ground, and nothing flashes dark
    if (!band) el.style.backgroundColor = g.bg;
  });
}

// the featured plan's lit border (cards.css .plan.is-featured): it rests on a quiet accent hairline;
// on hover or keyboard focus one light runs round its edge (edge-glow, mounted for that lap, then
// faded and freed), so nothing moves on its own or because the page scrolled
function glowOnHover(fx, els) {
  for (const el of els) {
    let busy = false;
    const lap = async () => {
      if (busy || reduced()) return;
      busy = true;
      try {
        let params = {};
        try { params = JSON.parse(el.dataset.glowParams || '{}'); } catch { /* defaults */ }
        const c = await fx.mount(el, el.dataset.glow, { palette: fx.readPalette(el), params, seed: 2 });
        const r = el.getBoundingClientRect(), per = 2 * (r.width + r.height);
        await new Promise(res => setTimeout(res, per / Math.max(40, Math.min(c.params?.speed || 400, per / 4)) * 1000));
        el.classList.add('is-fading');
        await new Promise(res => setTimeout(res, 650));
        c.destroy();
      } catch (e) { console.warn('system.js: glow', e); }
      el.classList.remove('is-fading');
      busy = false;
    };
    el.addEventListener('pointerenter', lap);
    el.addEventListener('focusin', lap);
    fx.load(el.dataset.glow).catch(() => {});
  }
}

async function visuals() {
  // a lit border written as data-fx="edge-glow" is the hover light (data-glow), never a ground
  document.querySelectorAll('[data-fx="edge-glow"]').forEach(el => {
    el.dataset.glow = 'edge-glow';
    if (el.dataset.fxParams && !el.dataset.glowParams) el.dataset.glowParams = el.dataset.fxParams;
    el.removeAttribute('data-fx');
  });
  const grounds = [...document.querySelectorAll('[data-fx]')];
  const glows = [...document.querySelectorAll('[data-glow]')];
  const frags = [...document.querySelectorAll('[data-ui-load], [data-ui]')].filter(el => !el.parentElement?.closest('[data-ui-load]'));
  if (!grounds.length && !frags.length && !glows.length) return;
  // only the hero's ground moves, and not when its frame holds an object (assets/objects: data-print),
  // which is then that visual's one motion and its pause button the only one in the corner. On a phone
  // (under 640px), on kind other and on a place nothing loops: the hero's ground is drawn once like the
  // rest and every fragment holds its finished picture (as most good sites cut motion there), so no
  // pause button covers the UI. A held fragment still plays while the pointer is on its card.
  const phone = window.matchMedia('(max-width: 639px)').matches || kindOther();
  const box = el => el.closest('.frame, .panel__screen, .panel') || el;
  const moving = phone ? [] : grounds.filter(el => el.closest('.hero, .section--hero') && !box(el).querySelector('[data-print], [data-scene]'));
  const live = el => moving.includes(el);
  const [fx, ui] = await Promise.all([
    grounds.length || glows.length ? import(new URL('effects/fx.js', HERE).href).then(m => m.default) : null,
    import(new URL('ui/ui.js', HERE).href).then(m => m.default),
  ]);
  const name = el => el.dataset.uiLoad || el.dataset.ui || '';
  const loops = el => !phone && !!el.closest('[data-ui-loop], [data-ui-cycles]');
  // a fragment that does not loop is held on its finished picture as soon as its own load resolves
  frags.forEach(el => { if (!loops(el)) el.setAttribute('data-ui-still', ''); });
  const opts = el => { try { return JSON.parse(el.dataset.uiOpts || '{}'); } catch { return {}; } };
  const run = el => (el.dataset.uiLoad ? ui.load(el, el.dataset.uiLoad, opts(el)) : ui.start(el)).catch(e => console.warn(e));

  if (fx) lightGrounds();
  await Promise.all([
    // the hero's ground moves and keeps its context; every other ground is baked to a picture
    fx && Promise.all(moving.map(el => fx.mountAll(el))),
    fx && grounds.length && ui.grounds(document, fx, { skip: live }),
    ...frags.map(run),
  ]);
  // the hero's ground moves, so it gets the pause button every loop has (one per frame, shared)
  if (fx && ui) moving.forEach(el => { const c = fx.get(el); if (c) ui.pausable(el.closest('.panel__screen, .panel, .frame') || el, c); });
  if (fx && glows.length) glowOnHover(fx, glows);
}

/* ------------------------------------------------------------------ page tone
   <html data-tone="dark|light"> from the palette's base, so a dark footer stays dark on a dark page
   (system.css .footer--dark) instead of turning into the light inverse */
function tone() {
  const c = rgb(getComputedStyle(document.documentElement).getPropertyValue('--p-bg'));
  if (!c) return;
  const lin = v => { v /= 255; return v <= 0.04045 ? v / 12.92 : ((v + 0.055) / 1.055) ** 2.4; };
  const L = 0.2126 * lin(c[0]) + 0.7152 * lin(c[1]) + 0.0722 * lin(c[2]);
  document.documentElement.dataset.tone = L < 0.18 ? 'dark' : 'light';
}

/* ------------------------------------------------------------------ pictures always draw
   A photograph never shows as an empty frame: once the page has loaded, every lazy image is fetched
   (so one far down the page is there when a visitor or a screenshot reaches it), and one that fails
   to load is outlined in red and counted (<html data-broken="n">), like a placeholder. */
function pictures() {
  const html = document.documentElement;
  let broken = 0;
  const fail = img => {
    if (img.dataset.broken != null) return;
    img.dataset.broken = '';
    img.setAttribute('data-placeholder', '');
    html.dataset.broken = String(++broken);
    console.warn(`system.js: a picture did not load: ${img.currentSrc || img.src}`);
  };
  document.querySelectorAll('img').forEach(img => {
    // an SVG with only a viewBox reports no natural width though it draws: decode() tells
    if (img.complete && img.getAttribute('src') && !img.naturalWidth) img.decode().catch(() => fail(img));
    else img.addEventListener('error', () => fail(img), { once: true });
  });
  const eager = () => document.querySelectorAll('img[loading="lazy"]').forEach(img => { img.loading = 'eager'; });
  const idle = window.requestIdleCallback || (f => setTimeout(f, 200));
  if (document.readyState === 'complete') idle(eager);
  else window.addEventListener('load', () => idle(eager), { once: true });
}

/* ------------------------------------------------------------------ guards
   Faults that shipped before: a [bracketed placeholder] or a "(sample)" tag in visible text, a line that
   explains or apologises ("Names and figures on this page are examples", "made in code", "photographs
   follow"), a printed .example address, and one effect in more than two sections. Each is marked so it
   shows in every screenshot and the checker can read it: [data-placeholder] (a red dashed outline,
   system.css) and <html data-placeholders="n">; <html data-fx-repeat="id ...">. */
const PLACEHOLDER = /\[[^\]\n]{1,60}\]|\((?:sample|placeholder|example)[^)]{0,30}\)|\bsample (?:figure|price|name|quote|number|photo|page)s?\b|\bplaceholder\b|\bnames and figures\b|\b(?:are|is) (?:examples?|illustrative|invented)\b|\bfor illustration\b|\b(?:made|drawn|rendered) in code\b|\bphotographs? (?:follow|to follow|coming)\b|\bto be replaced\b|[\w.+-]+@[\w-]+\.example\b|\bwww\.[\w-]+\.example\b/i;
function guards() {
  const html = document.documentElement;
  let n = 0;
  const walk = document.createTreeWalker(document.body, NodeFilter.SHOW_TEXT, {
    acceptNode: t => (t.parentElement?.closest('script, style, template, noscript') ? NodeFilter.FILTER_REJECT : NodeFilter.FILTER_ACCEPT),
  });
  for (let t = walk.nextNode(); t; t = walk.nextNode()) {
    if (PLACEHOLDER.test(t.textContent)) { t.parentElement.setAttribute('data-placeholder', ''); n++; }
  }
  document.querySelectorAll('img[alt]').forEach(img => { if (PLACEHOLDER.test(img.alt)) { img.setAttribute('data-placeholder', ''); n++; } });
  // the old figures note, whatever its words
  document.querySelectorAll('.figures-note').forEach(el => { if (!el.hasAttribute('data-placeholder')) { el.setAttribute('data-placeholder', ''); n++; } });
  if (n) { html.dataset.placeholders = String(n); console.warn(`system.js: ${n} placeholder, sample tag, apology or .example address left in visible text`); }
  // an effect id in at most two sections (a card group counts once; the closing band is a section)
  const seen = new Map();
  document.querySelectorAll('[data-fx]').forEach(el => {
    if (el.closest('.nav')) return;   // the menu card's panel is hidden until hover: not counted
    const sec = el.closest('section, footer') || el;
    if (!seen.has(el.dataset.fx)) seen.set(el.dataset.fx, new Set());
    seen.get(el.dataset.fx).add(sec);
  });
  const repeat = [...seen].filter(([, s]) => s.size > 2).map(([id]) => id);
  if (repeat.length) { html.dataset.fxRepeat = repeat.join(' '); console.warn(`system.js: effect in more than two sections: ${repeat.join(', ')}`); }
}

/* ------------------------------------------------------------------ moments
   The page's one or two dealt moments (assets/moments, references/moments.md): loaded only when the page
   holds a [data-moment]; data-ready waits for them. mount is idempotent, so nothing mounts twice. */
async function moments() {
  if (!document.querySelector('[data-moment]')) return;
  const m = await import(new URL('moments/moments.js', HERE).href);
  await m.ready;
}

function init() {
  try { tone(); } catch (e) { console.warn('system.js: tone', e); }
  nav();
  tabs();
  try { faq(); } catch (e) { console.warn('system.js: faq', e); }
  try { hours(); } catch (e) { console.warn('system.js: hours', e); }
  try { peek(); } catch (e) { console.warn('system.js: peek', e); }
  try { pictures(); } catch (e) { console.warn('system.js: pictures', e); }
  try { guards(); } catch (e) { console.warn('system.js: guards', e); }
  Promise.all([
    visuals().catch(e => console.warn('system.js: visuals', e)),
    moments().catch(e => console.warn('system.js: moments', e)),
  ]).finally(() => { document.documentElement.dataset.ready = '1'; });
}
if (document.readyState === 'loading') document.addEventListener('DOMContentLoaded', init, { once: true });
else init();

export default { init };
```

## assets/ui/base.css

```css
/*
  taste-skill UI fragment kit - shared base.
  Small pieces of real product UI, set on an effect ground inside a card's inset panel.
  The standard (references/cards.md): true product scale (13px text, 12px meta, never under 11px).
  A fragment is a picture of a product's screen, so the page's 13px floor (VI-05) does not apply
  inside .ui; every word the page itself says (titles, lines, labels, captions) keeps 13px or more.
  The rest:
  real-looking data, crisp 1px borders, radius 10-12, a soft layered shadow, one accent touch
  (the agent's mark is the ink, so the accent stays free for that one touch).

  Styled only by custom properties set by the page:
    --ui-font, --ui-mono, --ui-radius
    --fx-bg (the fragment's own surface), --fx-surface, --fx-ink, --fx-muted, --fx-accent, --fx-accent-ink
  Light or dark is read from --fx-bg itself (relative colour), so a fragment needs no theme class.
  Every size sits on a 4px grid. Loops stop under prefers-reduced-motion (the JS shows a still state).
*/
:where(.ui) {
  --u-bg: var(--fx-bg, #ffffff);
  --u-surface: var(--fx-surface, #f4f4f3);
  --u-ink: var(--fx-ink, #111111);
  --u-muted: var(--fx-muted, #6b6b6b);
  --u-accent: var(--fx-accent, #e8571f);
  --u-accent-ink: var(--fx-accent-ink, #ffffff);
  --u-r: var(--ui-radius, 12px);
  --u-r2: max(4px, calc(var(--u-r) - 4px));
  --u-r3: max(4px, calc(var(--u-r) - 6px));

  /* lines and fills, all mixed from the palette */
  --u-line: color-mix(in oklab, var(--u-ink) 9%, transparent);
  --u-line-2: color-mix(in oklab, var(--u-ink) 15%, transparent);
  --u-border: color-mix(in oklab, var(--u-ink) 12%, transparent);
  --u-fill: color-mix(in oklab, var(--u-ink) 3.5%, var(--u-bg));
  --u-fill-2: color-mix(in oklab, var(--u-ink) 7%, var(--u-bg));
  --u-fill-3: color-mix(in oklab, var(--u-ink) 11%, var(--u-bg));
  --u-text-2: color-mix(in oklab, var(--u-ink) 80%, var(--u-bg));
  --u-soft: color-mix(in oklab, var(--u-muted) 80%, var(--u-bg));
  --u-tint: color-mix(in oklab, var(--u-accent) 13%, var(--u-bg));
  --u-tint-2: color-mix(in oklab, var(--u-accent) 24%, var(--u-bg));
  --u-accent-text: color-mix(in oklab, var(--u-accent) 74%, var(--u-ink));
  --u-skel: color-mix(in oklab, var(--u-ink) 10%, var(--u-bg));
  /* a brand tile is light on every surface, so a logo keeps its own colours (references/icons.md) */
  --u-tile: #ffffff;
  --u-tile-edge: inset 0 0 0 1px rgb(0 0 0 / 0.09), 0 1px 2px rgb(0 0 0 / 0.12);

  /* depth: shadows grow and a top highlight appears as the fragment surface gets darker */
  --u-sh: rgb(0 0 0 / 0.14);
  --u-sh: oklch(from var(--u-bg) 0 0 0 / calc(0.5 - l * 0.38));
  --u-hi: transparent;
  --u-hi: oklch(from var(--u-bg) 1 0 0 / clamp(0, calc(0.11 - l * 0.11), 0.09));
  --u-shadow:
    inset 0 1px 0 var(--u-hi),
    0 1px 1px color-mix(in oklab, var(--u-sh) 40%, transparent),
    0 2px 4px -1px color-mix(in oklab, var(--u-sh) 45%, transparent),
    0 12px 24px -8px color-mix(in oklab, var(--u-sh) 80%, transparent),
    0 32px 64px -24px var(--u-sh);
  --u-shadow-sm:
    inset 0 1px 0 var(--u-hi),
    0 1px 2px color-mix(in oklab, var(--u-sh) 55%, transparent),
    0 6px 16px -6px color-mix(in oklab, var(--u-sh) 80%, transparent);
  --u-ease: cubic-bezier(0.2, 0.7, 0.2, 1);
  --u-spring: cubic-bezier(0.3, 1.3, 0.5, 1);

  font-family: var(--ui-font, system-ui, -apple-system, "Segoe UI", sans-serif);
  font-size: 13px;
  line-height: 20px;
  font-weight: 400;
  color: var(--u-ink);
  -webkit-font-smoothing: antialiased;
  font-feature-settings: "tnum" 1, "cv11" 1;
  text-align: left;
  letter-spacing: -0.003em;
  box-sizing: border-box;
}
:where(.ui) *, :where(.ui) *::before, :where(.ui) *::after { box-sizing: border-box; }
/* a fragment never runs out of its panel on its own: on a narrow panel it narrows; a placement that
   crops it on purpose (cards.css, from 640px: every placement but .at-center) lifts this */
[data-ui] { max-width: 100%; }
/* fragments use div and span for text so page rules like `.card p` cannot reach them */
.ui :where(ul, ol, li, figure, blockquote, article, aside, nav) { margin: 0; padding: 0; list-style: none; }
.ui :where(mark) { background: none; color: inherit; }
:where(.ui) button { font: inherit; color: inherit; background: none; border: 0; padding: 0; margin: 0; cursor: default; }
:where(.ui) svg { display: block; flex: none; }

/* ---------------------------------------------------------------- the window every fragment sits in */
.ui-panel {
  position: relative;
  background: var(--u-bg);
  border: 1px solid var(--u-border);
  border-radius: var(--u-r);
  box-shadow: var(--u-shadow);
  max-width: 100%;
}
.ui-head {
  display: flex; align-items: center; gap: 8px;
  height: 44px; padding: 0 12px 0 16px;
  border-bottom: 1px solid var(--u-line);
}
.ui-title { font-size: 13px; font-weight: 500; line-height: 20px; white-space: nowrap; }
.ui-meta { font-size: 13px; line-height: 16px; color: var(--u-muted); white-space: nowrap; }
.ui-spacer { flex: 1; }
.ui-sep { width: 1px; height: 16px; background: var(--u-line-2); flex: none; }

/* window chrome: three quiet dots and a centred title */
.ui-dots { display: flex; gap: 6px; flex: none; }
.ui-dots i { width: 10px; height: 10px; border-radius: 50%; background: var(--u-fill-3); box-shadow: inset 0 0 0 0.5px var(--u-line); }

/* ---------------------------------------------------------------- controls */
/* the primary button is the ink (as on figma.com and x.ai); the accent is kept for one touch */
.ui-btn {
  display: inline-flex; align-items: center; justify-content: center; gap: 6px; flex: none;
  height: 28px; padding: 0 10px; border-radius: max(6px, var(--u-r3));
  font-size: 13px; font-weight: 500; line-height: 1; white-space: nowrap;
  background: var(--u-ink); color: var(--u-bg);
  box-shadow: inset 0 1px 0 color-mix(in oklab, var(--u-bg) 14%, transparent), 0 1px 2px color-mix(in oklab, var(--u-sh) 60%, transparent);
  transition: transform 160ms var(--u-ease), background-color 200ms var(--u-ease), color 200ms var(--u-ease);
}
.ui-btn--accent { background: var(--u-accent); color: var(--u-accent-ink); }
.ui-btn--ghost { background: var(--u-bg); color: var(--u-ink); box-shadow: inset 0 0 0 1px var(--u-line-2), 0 1px 2px color-mix(in oklab, var(--u-sh) 30%, transparent); }
.ui-btn--quiet { background: transparent; color: var(--u-muted); box-shadow: none; }
.ui-btn--sm { height: 24px; padding: 0 8px; font-size: 13px; }
.ui-btn .ui-icon { width: 14px; height: 14px; }
.ui-btn.is-pressed { transform: scale(0.96); }

/* status: a dot and a word; quiet by default, the accent only where something is happening */
.ui-chip {
  display: inline-flex; align-items: center; gap: 6px; flex: none;
  height: 22px; padding: 0 8px; border-radius: 6px;
  font-size: 13px; font-weight: 500; line-height: 1; white-space: nowrap;
  background: var(--u-fill); color: var(--u-muted); box-shadow: inset 0 0 0 1px var(--u-line);
}
.ui-chip::before { content: ""; width: 6px; height: 6px; border-radius: 50%; background: currentColor; opacity: 0.55; flex: none; }
.ui-chip--plain::before { display: none; }
.ui-chip--accent { background: var(--u-tint); color: var(--u-accent-text); box-shadow: inset 0 0 0 1px color-mix(in oklab, var(--u-accent) 22%, transparent); }
.ui-chip--accent::before { background: var(--u-accent); opacity: 1; }
.ui-chip--ink { color: var(--u-ink); }
.ui-chip--ink::before { opacity: 1; }
.ui-chip--live::before { animation: ui-live 1.6s var(--u-ease) infinite; }
.ui-chip svg { width: 12px; height: 12px; margin: 0 -1px 0 -2px; }
.ui-chip:has(svg)::before { display: none; }
@keyframes ui-live { 0%, 100% { opacity: 1; } 50% { opacity: 0.35; } }

.ui-dot { width: 6px; height: 6px; border-radius: 50%; background: var(--u-accent); flex: none; }
.ui-kbd {
  display: inline-grid; place-items: center; min-width: 20px; height: 20px; padding: 0 5px; border-radius: 5px;
  font: 500 13px/1 var(--ui-font, inherit); color: var(--u-muted);
  background: var(--u-bg); box-shadow: inset 0 0 0 1px var(--u-line-2), inset 0 -1px 0 var(--u-line);
}
.ui-icon { width: 16px; height: 16px; fill: none; stroke: currentColor; stroke-width: 1.5; stroke-linecap: round; stroke-linejoin: round; }

/* people: initials on a quiet tone, five tones that never reach violet */
.ui-avatar {
  --h: 60;
  display: grid; place-items: center; flex: none;
  width: 24px; height: 24px; border-radius: 50%;
  font-size: 13px; font-weight: 600; line-height: 1; letter-spacing: -0.02em;
  background: color-mix(in oklab, oklch(0.74 0.08 var(--h)) 34%, var(--u-bg));
  color: color-mix(in oklab, oklch(0.5 0.09 var(--h)) 70%, var(--u-ink));
  box-shadow: inset 0 0 0 1px color-mix(in oklab, oklch(0.6 0.08 var(--h)) 22%, transparent);
}
.ui-avatar[data-tone="1"] { --h: 40; }
.ui-avatar[data-tone="2"] { --h: 85; }
.ui-avatar[data-tone="3"] { --h: 150; }
.ui-avatar[data-tone="4"] { --h: 220; }
.ui-avatar[data-tone="5"] { --h: 20; }
.ui-avatar--md { width: 32px; height: 32px; font-size: 13px; }
.ui-avatar--accent { background: var(--u-accent); color: var(--u-accent-ink); box-shadow: none; }

/* the product's own mark (an assistant, a bot): an ink disc with a four-point spark. It is the ink,
   not the accent, so a fragment's one accent touch stays free for its status */
.ui-agent {
  position: relative; display: block; flex: none; width: 24px; height: 24px; border-radius: 50%;
  background: var(--u-ink);
  box-shadow: inset 0 0 0 1px color-mix(in oklab, var(--u-bg) 12%, transparent), inset 0 1px 0 color-mix(in oklab, #fff 16%, transparent);
}
.ui-agent::after {
  content: ""; position: absolute; inset: 26%; background: var(--u-bg);
  -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'%3E%3Cpath d='M8 .8c.5 4.3 2.9 6.7 7.2 7.2-4.3.5-6.7 2.9-7.2 7.2C7.5 10.9 5.1 8.5.8 8 5.1 7.5 7.5 5.1 8 .8z'/%3E%3C/svg%3E") center / contain no-repeat;
          mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'%3E%3Cpath d='M8 .8c.5 4.3 2.9 6.7 7.2 7.2-4.3.5-6.7 2.9-7.2 7.2C7.5 10.9 5.1 8.5.8 8 5.1 7.5 7.5 5.1 8 .8z'/%3E%3C/svg%3E") center / contain no-repeat;
}

/* skeleton lines for the parts of a window that are only context */
.ui-skel { display: block; height: 6px; border-radius: 3px; background: var(--u-skel); }

/* ---------------------------------------------------------------- brand marks */
/* Real brand icons are local files (icons/, written by scripts/icons.mjs fetch; icons-credits.json lists them): no page calls
   a CDN for an icon. A mark on a tile (.ui-logo, .ui-file__icon, .ui-field__tile, .ui-diff__tool, .ui-pipe__tool) is a colour
   file as <img src="icons/notion.svg" alt="">, the path as printed by fetch for the page that holds it. A small mark inline beside
   text stays in the ink: the one-colour inline <svg class="ui-brand" aria-hidden="true"> that fetch --mono prints. */
.ui-brand { display: block; flex: none; width: 18px; height: 18px; }
svg.ui-brand { fill: currentColor; color: var(--u-ink); }
.ui-brand--ink { color: var(--u-ink); }
/* a brand logo on a small tile: the tile is light on every surface and the logo in its own colours */
.ui-logo {
  display: grid; place-items: center; flex: none;
  width: 40px; height: 40px; border-radius: var(--u-r2);
  background: var(--u-tile); box-shadow: var(--u-tile-edge);
}
.ui-logo img, .ui-file__icon img, .ui-field__tile img, .ui-diff__tool img, .ui-pipe__tool img { display: block; flex: none; object-fit: contain; }
.ui-logo :is(svg, img, .ui-brand) { width: 20px; height: 20px; }
.ui-logo--sm { width: 32px; height: 32px; border-radius: max(6px, var(--u-r3)); }
.ui-logo--sm :is(svg, img, .ui-brand) { width: 16px; height: 16px; }
.ui-logo--lg { width: 56px; height: 56px; border-radius: var(--u-r); }
.ui-logo--lg :is(svg, img, .ui-brand) { width: 28px; height: 28px; }

/* ---------------------------------------------------------------- the pointer */
.ui-pointer {
  position: absolute; left: 0; top: 0; z-index: 5;
  width: 20px; height: 20px; pointer-events: none;
  filter: drop-shadow(0 1px 1px rgb(0 0 0 / 0.25)) drop-shadow(0 3px 6px rgb(0 0 0 / 0.2));
  will-change: transform;
}
.ui-pointer svg { width: 20px; height: 20px; }
/* a named pointer (a person or the agent working in the product) */
.ui-pointer__name {
  position: absolute; left: 14px; top: 18px; height: 20px; padding: 0 7px; border-radius: 6px 10px 10px 10px;
  font: 500 13px/20px var(--ui-font, inherit); white-space: nowrap;
  background: var(--u-accent); color: var(--u-accent-ink);
}
.ui-ring {
  position: absolute; left: 0; top: 0; width: 28px; height: 28px; margin: -14px 0 0 -14px; border-radius: 50%;
  border: 1.5px solid var(--u-accent); opacity: 0; pointer-events: none; z-index: 4;
}
.ui-ring.is-on { animation: ui-ring 520ms var(--u-ease) forwards; }
@keyframes ui-ring { from { opacity: 0.8; transform: var(--at) scale(0.4); } to { opacity: 0; transform: var(--at) scale(1.5); } }

@keyframes ui-spin { to { transform: rotate(360deg); } }
.ui-spin {
  display: inline-block; width: 12px; height: 12px; border-radius: 50%; flex: none;
  border: 1.5px solid color-mix(in oklab, currentColor 30%, transparent); border-top-color: currentColor;
  animation: ui-spin 700ms linear infinite;
}

/* ---------------------------------------------------------------- held, paused */
/* a fragment held on its finished picture (data-ui-still, its cycles done) shows nothing that moves:
   no caret, spinner or pulse runs on in a still picture */
.is-ui-held, .is-ui-held *, .is-ui-held *::before, .is-ui-held *::after { animation: none !important; }
.is-ui-paused, .is-ui-paused *, .is-ui-paused *::before, .is-ui-paused *::after { animation-play-state: paused !important; }

/* the pause button ui.loop puts by a looping fragment (and an object from assets/objects, as
   .loop-pause): a solid disc in the page colours (never a see-through blur), 32px in a 44px hit area,
   the same as components' .cx-pause and system.css. In a card it sits beside the title (ui.js sets its
   top and right; the title keeps 44px clear), in a panel outside a card 12px in from the bottom-right
   corner, and with no panel under the fragment, in 44px kept free below it. */
button.ui-pause.ui-pause, button.loop-pause {
  position: absolute; right: 12px; bottom: 12px; z-index: 6; display: grid; place-items: center;
  width: 32px; height: 32px; margin: 0; padding: 0; border-radius: 50%; cursor: pointer;
  border: 1px solid color-mix(in oklab, var(--ink, #111) 16%, transparent);
  background: var(--bg, #fff); color: var(--ink, #111);
  box-shadow: 0 1px 2px rgb(0 0 0 / 0.12), 0 4px 12px -4px rgb(0 0 0 / 0.28);
  opacity: 0.88; transition: opacity 200ms ease, border-color 200ms ease;
  -webkit-tap-highlight-color: transparent;
}
button.ui-pause.ui-pause--title { bottom: auto; box-shadow: none; opacity: 1; }
button.ui-pause.ui-pause--below { right: 0; bottom: 6px; box-shadow: none; }
.ui-pause-beside { padding-right: 44px; }
button:is(.ui-pause, .loop-pause)::before { content: ""; position: absolute; inset: -6px; border-radius: 50%; }
button:is(.ui-pause, .loop-pause):is(:hover, :focus-visible) { opacity: 1; border-color: color-mix(in oklab, var(--ink, #111) 32%, transparent); }
button:is(.ui-pause, .loop-pause):focus-visible { outline: 2px solid var(--ink, #111); outline-offset: 2px; }
button:is(.ui-pause, .loop-pause) svg { display: block; width: 14px; height: 14px; fill: currentColor; }
button:is(.ui-pause, .loop-pause).is-paused svg, button.loop-pause[aria-pressed="true"] svg { margin-left: 1px; }
@media (forced-colors: active) { button.ui-pause.ui-pause, button.loop-pause { border-color: ButtonText; } }

@media (prefers-reduced-motion: reduce) {
  :where(.ui) *, :where(.ui) *::before, :where(.ui) *::after { transition-duration: 0ms !important; animation: none !important; }
  button.ui-pause.ui-pause, button.loop-pause { display: none; }
}
```

## assets/ui/cards.css

```css
/*
  taste-skill feature cards - the card, its inset panel and where a fragment sits on it.
  The rules and their reasons are in references/cards.md. Link after system.css (it refines .card
  and .panel there) or on its own with these tokens on a ground:
    --bg (page), --card (one step off the page), --ink, --muted, --accent, --accent-ink,
    --font-text, --font-mono
  Fragments in a panel take --fx-* from .stage (set below from the same tokens).
*/

/* ---------------------------------------------------------------- the card */
.card {
  --c-r: 20px;                      /* card radius */
  --c-pad: 28px;                    /* text padding = the panel's side inset */
  --c-gap: 8px;                     /* an inset panel's margin; its radius is --c-r minus this */
  --c-border: color-mix(in oklab, var(--ink) 8%, transparent);
  --c-text: color-mix(in oklab, var(--ink) 84%, var(--card));   /* card text: the ink, a touch softer than titles */
  --c-hi: oklch(from var(--card) 1 0 0 / clamp(0, calc(0.09 - l * 0.09), 0.07));
  --c-sh: oklch(from var(--card) 0 0 0 / calc(0.07 - l * 0.04));
  position: relative; display: flex; flex-direction: column; min-width: 0;
  padding: var(--c-pad); border-radius: var(--c-r);
  background: var(--card); color: var(--ink);
  border: 1px solid var(--c-border);
  box-shadow: inset 0 1px 0 var(--c-hi), 0 1px 2px var(--c-sh);
  overflow: clip;
  gap: 0;
}
@media (min-width: 1024px) { .card { --c-pad: 32px; } }
@media (max-width: 639px) { .card { --c-r: 18px; --c-pad: 20px; } }

.card > h3, .card__title {
  margin: 0; font-family: var(--font-text); font-size: 18px; line-height: 26px; font-weight: 560;
  letter-spacing: -0.012em; color: var(--ink); text-wrap: balance;
}
/* the two lines under a title: 16px in the ink (a touch softer than the title), never small grey text */
.card > p, .card__text {
  margin: 8px 0 0; font-size: 16px; line-height: 24px; color: var(--c-text);
  max-width: 46ch; text-wrap: pretty;
}
/* two muted lines keep one height in a row so the panels line up; only where cards sit side by side
   (below 1024px a row is one column, and the held line would be an empty gap) */
@media (min-width: 1024px) { .card--row > p { min-height: 2lh; } }
.card__link {
  display: inline-flex; align-items: center; gap: 6px; margin-top: 16px; width: max-content;
  font-size: 15px; font-weight: 500; line-height: 22px; color: var(--ink); text-decoration: none;
}
.card__link svg { width: 14px; height: 14px; transition: transform 200ms cubic-bezier(.2,.7,.2,1); }
.card__link:hover svg { transform: translateX(2px); }

/* ---------------------------------------------------------------- the inset panel */
/* Default (x.ai): aligned with the text column and running off the card's bottom edge.
   The effect canvas mounts on the panel (data-fx) and sits under everything in it. */
.panel {
  --p-r: 12px;
  display: block; padding: 0;
  --p-h: clamp(264px, 22vw + 8px, 340px);
  position: relative; isolation: isolate; overflow: clip;
  min-height: var(--p-h); flex: 1 0 auto;
  border-radius: var(--p-r) var(--p-r) 0 0;
  background: color-mix(in oklab, var(--ink) 4%, var(--card));
}
/* a panel with no effect mounted stands on the page's lit ground (system.css --ground-*: sand, sky,
   sage, painting), never a grey a shade off the white UI on it: flat on kind other (cafes, shops,
   studios); on a tech page (site-kind tech, software or gaming) a soft mesh of the ground's own tones,
   the Figma tile, with no grain (grain or texture would be the page's one effect). A [data-fx] panel
   draws its effect instead. */
.panel:not([data-fx], .panel--device) { background: var(--ground-bg, #d9ccb7); }
:root:has(meta[name="site-kind"]:is([content="tech"], [content="software"], [content="gaming"])) .panel:not([data-fx], .panel--device) {
  background-color: var(--ground-bg, #d9ccb7);
  background-image:
    radial-gradient(70% 90% at 12% 0%, color-mix(in oklab, var(--ground-surface, #e9e0d2) 92%, white), transparent 70%),
    radial-gradient(60% 80% at 100% 100%, color-mix(in oklab, var(--ground-accent, #a8794a) 34%, var(--ground-bg, #d9ccb7)), transparent 72%),
    radial-gradient(50% 60% at 85% 10%, color-mix(in oklab, var(--ground-surface, #e9e0d2) 60%, transparent), transparent 70%);
}
.card > .panel { margin-top: 32px; margin-bottom: calc(-1 * var(--c-pad) - 1px); }
.card > .panel--device { margin-top: 24px; }
/* system.css spaces the text above a panel itself; here the panel carries the 32px */
.card > :has(+ .panel) { margin-bottom: 0; }
/* a crisp 1px edge over the ground and over any fragment it crops */
.panel::after {
  content: ""; position: absolute; inset: 0; z-index: 4; pointer-events: none; border-radius: inherit;
  box-shadow: inset 0 0 0 1px color-mix(in oklab, var(--ink) 7%, transparent), inset 0 1px 0 color-mix(in oklab, var(--ink) 4%, transparent);
}
/* the vignette that calms the ground where the fragment sits (--vx, --vy: its centre) */
.panel::before {
  content: ""; position: absolute; inset: 0; z-index: 0; pointer-events: none; width: auto; padding: 0;
  background: radial-gradient(ellipse 70% 62% at var(--vx, 50%) var(--vy, 62%),
    color-mix(in oklab, var(--calm, var(--card)) var(--calm-k, 34%), transparent), transparent 72%);
}
/* inset on all sides by --c-gap; nested radius */
.card > .panel--inset {
  margin: 32px calc(var(--c-gap) - var(--c-pad)) calc(var(--c-gap) - var(--c-pad));
  border-radius: calc(var(--c-r) - var(--c-gap));
}
/* runs to the card's side and bottom edges (Figma's image band); the card's corners clip it */
.card > .panel--flush {
  margin: 32px calc(-1 * var(--c-pad) - 1px) calc(-1 * var(--c-pad) - 1px);
  border-radius: 0;
}
.panel--flush::after { box-shadow: inset 0 1px 0 color-mix(in oklab, var(--ink) 7%, transparent); }
/* the panel on top, the caption under it (tall cards) */
.card > .panel--top {
  order: -1; margin: calc(var(--c-gap) - var(--c-pad)) calc(var(--c-gap) - var(--c-pad)) 24px;
  border-radius: calc(var(--c-r) - var(--c-gap)); flex: none; min-height: 0; aspect-ratio: 1 / 1;
}
.panel--top.panel--wide { aspect-ratio: 4 / 3; }

/* a device (x.ai's "Computer" card): a status row, then the ground as its screen. The row sits on the
   card itself, not in a frame of its own, so the card stays two levels deep (card > picture > UI) */
.panel--device { display: flex; padding: 0; flex-direction: column; background: none; overflow: visible; isolation: auto; border-radius: 0; }
.panel--device::before, .panel--device::after { display: none; }
.panel__bar {
  display: flex; align-items: center; gap: 8px; height: 32px; margin-bottom: 12px; padding: 0; flex: none;
  font: 500 13px/20px var(--font-text); color: var(--ink);
}
.panel__bar .muted { color: var(--muted); font-weight: 400; }
.panel__bar .sp { flex: 1; }
/* the bar's status is quiet ink: the accent belongs to the one status inside the screen */
.panel__status {
  display: inline-flex; align-items: center; gap: 6px; height: 22px; padding: 0 8px; border-radius: 6px;
  font: 500 var(--t-label)/1 var(--font-text); color: var(--ink);
  background: color-mix(in oklab, var(--ink) 6%, transparent); box-shadow: inset 0 0 0 1px color-mix(in oklab, var(--ink) 8%, transparent);
}
.panel__bar .ui-agent { width: 20px; height: 20px; --u-ink: var(--ink); --u-bg: var(--card); }
/* a still dot: the bar's status never spins on its own (the moving part is the screen's one loop) */
.panel__status i { width: 6px; height: 6px; border-radius: 50%; background: currentColor; opacity: 0.7; }
.panel__screen {
  position: relative; isolation: isolate; overflow: clip; flex: 1;
  margin: 0; border-radius: var(--p-r) var(--p-r) 0 0;
  background: color-mix(in oklab, var(--ink) 4%, var(--card));
}
.panel__screen::after {
  content: ""; position: absolute; inset: 0; z-index: 4; pointer-events: none; border-radius: inherit;
  box-shadow: inset 0 0 0 1px color-mix(in oklab, var(--ink) 7%, transparent);
}
.panel__screen::before {
  content: ""; position: absolute; inset: 0; z-index: 0; pointer-events: none;
  background: radial-gradient(ellipse 70% 62% at var(--vx, 50%) var(--vy, 70%),
    color-mix(in oklab, var(--calm, #000) var(--calm-k, 30%), transparent), transparent 72%);
}

/* ---------------------------------------------------------------- where a fragment sits */
/* .stage holds the fragment(s) and gives them their palette: the fragment surface is one step
   lighter than the card on dark grounds and white on light ones */
.stage, .panel > .stage, .panel__screen > .stage {
  position: absolute; inset: 0; z-index: 1; max-width: none;
  --ui-font: var(--font-text); --ui-mono: var(--font-mono); --ui-radius: 12px;
  /* the fragment surface: white on a light card, one step lighter than a dark card (--frag overrides) */
  --fx-bg: var(--frag, oklch(from var(--card) clamp(calc(l + 0.05), calc((l - 0.5) * 1000), 1) calc(c * 0.5) h));
  --fx-surface: var(--card); --fx-ink: var(--ink); --fx-muted: var(--muted);
  --fx-accent: var(--accent); --fx-accent-ink: var(--accent-ink);
}
.stage > * { position: absolute; max-width: none; }
/* from 640px a fragment keeps its true width and is cropped by the panel, never squeezed to fit
   (on a phone it takes the panel's width instead: see the end of this block) */
@media (min-width: 640px) { .stage > :not(.at-center) > [data-ui] { max-width: none; } }
/* a fragment cut by the panel's edge dissolves into the ground over its last --fade-h (64px; 40px for a
   short panel) instead of being sliced through a line of text (edge-fade-dissolve, 14 of 41 sites).
   The fade only ever covers context: the fragment's point (the row that changes, the number, the
   button) sits above it. */
.stage { --fade-h: 64px; }
.stage--fade-b { -webkit-mask: linear-gradient(to bottom, #000 calc(100% - var(--fade-h)), transparent calc(100% - 4px)); mask: linear-gradient(to bottom, #000 calc(100% - var(--fade-h)), transparent calc(100% - 4px)); }
.stage--fade-r { -webkit-mask: linear-gradient(to right, #000 calc(100% - 72px), transparent calc(100% - 4px)); mask: linear-gradient(to right, #000 calc(100% - 72px), transparent calc(100% - 4px)); }
.stage--fade-br {
  -webkit-mask: linear-gradient(to bottom, #000 calc(100% - var(--fade-h)), transparent calc(100% - 4px)), linear-gradient(to right, #000 calc(100% - 72px), transparent calc(100% - 4px));
  -webkit-mask-composite: source-in;
  mask: linear-gradient(to bottom, #000 calc(100% - var(--fade-h)), transparent calc(100% - 4px)), linear-gradient(to right, #000 calc(100% - 72px), transparent calc(100% - 4px));
  mask-composite: intersect;
}
/* anchored at the top-left inset, cut by the right and bottom edges (19 of 41 sites) */
.at-corner { left: var(--ax, 32px); top: var(--ay, 32px); }
/* anchored at the top-right inset, cut by the bottom only: on a narrow panel it narrows to the panel
   minus both insets rather than run out of the left edge through its text */
.at-right { right: var(--ax, 32px); top: var(--ay, 32px); max-width: calc(100% - 2 * var(--ax, 32px)); }
.stage > .at-right > :is([data-ui], .ui) { max-width: 100%; }
/* rising out of the bottom edge (x.ai) */
.at-rise { left: var(--ax, 18%); top: var(--ay, 48px); }
/* .to-edge: the wrapper reaches --cut (72px) past the panel's bottom edge whatever the panel's height,
   so a fragment that fills its wrapper (stat with opts.fill, a long list) is always cut on purpose and
   never by a few accidental px */
.stage > .to-edge { bottom: calc(-1 * var(--cut, 72px)); }
.stage > .to-edge > :is([data-ui], .ui) { height: 100%; }
/* .fit: the fragment takes the panel's width minus --ax on both sides instead of its own, for a narrow
   panel where a few px of overhang would cut its text (contain it, or cut it by 48px or more).
   --fit-max (its true width, e.g. 400px for chat) keeps it at that width on a wide panel, centred
   between the insets (x.ai's window rising out of the bottom edge), so it only narrows where the
   panel is too small: never a 1-23px sliver, a cut at 1024, or uneven ground at the sides */
.stage > .fit { right: var(--ax, 32px); }
.stage > .fit > :is([data-ui], .ui) { width: 100%; max-width: var(--fit-max, 100%); margin-inline: auto; }
/* .wide: a fluid fragment (files, calendar) set wider than the panel (--w), so it is cut by the right
   edge like a window, not centred with even ground round it; pair with .stage--fade-br. Whatever the
   panel's width it runs at least 64px past the right edge */
.stage > .wide { width: max(var(--w, 680px), 100% - var(--ax, 32px) + 64px); }
.stage > .wide > :is([data-ui], .ui) { width: 100%; max-width: none; }
/* a small control in the calm middle of a focal ground (Figma's Texture tile): never a window or a
   dialog. It never runs past the panel: at most the panel less 24px a side */
.at-center { left: 50%; top: 50%; translate: -50% -50%; max-width: calc(100% - 48px); }
.stage > .at-center > :is([data-ui], .ui) { max-width: 100%; }
/* a second fragment laid over the first one's edge */
.at-over { right: var(--bx, 24px); bottom: var(--by, 24px); z-index: 2; }
.at-over-l { left: var(--bx, 24px); bottom: var(--by, 24px); z-index: 2; }
/* phone: a fragment takes the panel's width minus the inset on both sides (its own width stays the upper
   bound), so it is only ever cut at the bottom, never through a line of text at the side. The desktop
   --ax and --ay are ignored here; --ay-sm sets the top. */
@media (max-width: 639px) {
  .stage > :is(.at-corner, .at-right, .at-rise) { left: var(--ax-sm, 16px); right: var(--ax-sm, 16px); top: var(--ay-sm, 24px); max-width: none; }
  .stage > :is(.at-corner, .at-right, .at-rise) > :is([data-ui], .ui) { max-width: 100%; }
  .stage > .wide { width: auto; }
  .stage > .at-center { max-width: calc(100% - 2 * var(--ax-sm, 16px)); }
  /* a dialog over a window takes the panel's width less 12px a side, never running past it */
  .stage > :is(.at-over, .at-over-l) { left: 12px; right: 12px; bottom: 16px; }
  .stage > :is(.at-over, .at-over-l) > :is([data-ui], .ui) { width: auto; max-width: 100%; }
}

/* ---------------------------------------------------------------- variants */
/* text beside the panel */
.card--split { display: grid; grid-template-columns: minmax(0, 5fr) minmax(0, 7fr); gap: 40px; align-items: stretch; }
.card--split > .card__body { display: flex; flex-direction: column; justify-content: center; padding: 16px 0; }
.card--split > .panel { margin: calc(var(--c-gap) - var(--c-pad)) calc(var(--c-gap) - var(--c-pad)) calc(var(--c-gap) - var(--c-pad)) 0; border-radius: calc(var(--c-r) - var(--c-gap)); min-height: 420px; }
@media (max-width: 899px) {
  .card--split { grid-template-columns: minmax(0, 1fr); gap: 0; }
  .card--split > .card__body { padding: 0; }
  .card--split > .panel { margin: 28px calc(var(--c-gap) - var(--c-pad)) calc(var(--c-gap) - var(--c-pad)); min-height: 360px; }
}
.card__points { display: grid; gap: 0; margin: 24px 0 0; padding: 0; list-style: none; }
.card__points li { display: flex; gap: 12px; padding: 12px 0; border-top: 1px solid var(--c-border); font-size: 16px; line-height: 24px; color: var(--c-text); }
.card__points li b { color: var(--ink); font-weight: 500; }
.card__points svg { width: 18px; height: 18px; margin-top: 3px; flex: none; color: var(--ink); }

/* the big number: filled with an effect through a blend knock-out (see cards.md) */
.knock { position: relative; isolation: isolate; display: inline-block; line-height: 0.92; }
.knock__text {
  position: relative; z-index: 1; display: block;
  font: 600 var(--knock-size, 128px)/0.92 var(--font-display, var(--font-text)); letter-spacing: -0.05em;
  background: var(--card); color: #fff; mix-blend-mode: darken; padding: 0.04em 0.06em 0.08em 0;
}
.knock--light .knock__text { color: #000; mix-blend-mode: lighten; }
/* a plain big number beside a knocked-out one: the same size and the same box, so the rows under both
   start level */
.stat__num {
  display: block; font: 600 var(--knock-size, 128px)/0.92 var(--font-display, var(--font-text)); letter-spacing: -0.05em;
  padding: 0.04em 0.06em 0.08em 0; color: var(--ink); white-space: nowrap;
}
.stat__unit { font-size: 0.32em; letter-spacing: -0.01em; color: var(--muted); font-weight: 500; margin-left: 0.2em; }
.stat__label { font-size: 16px; line-height: 24px; color: var(--c-text); }
.stat__label b { color: var(--ink); font-weight: 500; }

/* testimonial: the quote and who said it, beside one result told as a number and two bars */
.quote { margin: 0; font-family: var(--font-display, var(--font-text)); font-size: clamp(22px, 1.1vw + 12px, 28px); line-height: 1.32; letter-spacing: -0.02em; font-weight: 450; color: var(--ink); text-wrap: pretty; }
.person { display: flex; align-items: center; gap: 12px; margin-top: 28px; }
.person__logo { display: grid; place-items: center; flex: none; width: 40px; height: 40px; border-radius: 10px; color: var(--ink); background: color-mix(in oklab, var(--ink) 5%, var(--card)); box-shadow: inset 0 0 0 1px var(--c-border); }
.person__logo svg { width: 22px; height: 22px; }
.person__name { display: block; font-size: 16px; line-height: 24px; font-weight: 500; color: var(--ink); }
.person__role { display: block; font-size: 16px; line-height: 24px; color: var(--c-text); }
.result { display: flex; flex-direction: column; justify-content: space-between; gap: 32px; padding-left: 40px; border-left: 1px solid var(--c-border); }
.result__fig { font: 600 clamp(64px, 3.4vw + 24px, 88px)/0.95 var(--font-display, var(--font-text)); letter-spacing: -0.05em; color: var(--ink); }
.result__cap { margin: 12px 0 0; max-width: 28ch; font-size: 16px; line-height: 24px; color: var(--c-text); }
.result__bars { display: grid; gap: 14px; margin: 0; padding: 0; list-style: none; }
.result__bars li { display: grid; grid-template-columns: 124px minmax(0, 1fr) auto; align-items: center; gap: 12px; font-size: 16px; line-height: 24px; color: var(--c-text); }
.result__bars i { display: block; height: 8px; width: var(--w); border-radius: 4px; background: color-mix(in oklab, var(--ink) 16%, var(--card)); }
.result__bars li:last-child i { background: var(--accent); }
.result__bars b { font-weight: 500; color: var(--ink); font-variant-numeric: tabular-nums; }
@media (max-width: 1023px) { .result { padding: 28px 0 0; border-left: 0; border-top: 1px solid var(--c-border); } }

/* pricing */
.plan { gap: 0; }
/* every plan's name row is the badge's height, so prices, buttons and lists sit level across the row */
.plan__name { height: 24px; font-size: 16px; line-height: 24px; font-weight: 500; color: var(--ink); display: flex; align-items: center; gap: 8px; }
.plan__price { display: flex; align-items: baseline; gap: 6px; margin-top: 20px; font: 600 44px/1 var(--font-display, var(--font-text)); letter-spacing: -0.04em; }
.plan__price small { font: 400 14px/20px var(--font-text); letter-spacing: 0; color: var(--muted); }
.plan__for { margin-top: 12px; font-size: 16px; line-height: 24px; color: var(--c-text); min-height: 2lh; }
.plan__btn {
  display: inline-flex; align-items: center; justify-content: center; height: 40px; margin-top: 24px; border-radius: 10px;
  font: 500 14px/1 var(--font-text); text-decoration: none; color: var(--ink);
  box-shadow: inset 0 0 0 1px color-mix(in oklab, var(--ink) 16%, transparent);
}
.plan__btn--primary { background: var(--ink); color: var(--card); box-shadow: inset 0 1px 0 color-mix(in oklab, var(--card) 18%, transparent); }
.plan__list { display: grid; gap: 10px; margin: 24px 0 0; padding: 20px 0 0; list-style: none; border-top: 1px solid var(--c-border); font-size: 16px; line-height: 24px; color: var(--c-text); }
.plan__list li { display: flex; gap: 10px; }
.plan__list li::before {
  content: ""; flex: none; width: 16px; height: 16px; margin-top: 4px; background: var(--ink); opacity: 0.85;
  -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'%3E%3Cpath d='m3.5 8.4 2.8 2.8 6.2-6.4' fill='none' stroke='black' stroke-width='1.6' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/svg%3E") center / 16px no-repeat;
          mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'%3E%3Cpath d='m3.5 8.4 2.8 2.8 6.2-6.4' fill='none' stroke='black' stroke-width='1.6' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/svg%3E") center / 16px no-repeat;
}
/* the featured plan: the same --card fill as its neighbours and a quiet accent hairline, its one accent
   touch (so the badge is quiet ink). On hover or focus a light runs once round the edge: an edge-glow
   canvas (fill 1, wash 0) mounted for that lap only, which paints the same fill under a lit border and
   then fades. Never a light that runs because the plan scrolled into view, never a comet left on one edge */
.plan.is-featured { border-color: color-mix(in oklab, var(--accent) 48%, transparent); background: var(--card); }
.plan.is-featured.is-fading > .fx-canvas { opacity: 0 !important; transition: opacity 600ms ease !important; }
.plan.is-featured .plan__badge { display: inline-flex; align-items: center; height: 22px; padding: 0 8px; border-radius: 6px; font: 500 var(--t-label)/1 var(--font-text); color: var(--ink); background: color-mix(in oklab, var(--ink) 7%, transparent); box-shadow: inset 0 0 0 1px color-mix(in oklab, var(--ink) 10%, transparent); }

/* on a phone a fragment fits the panel's width (see the placements), so it is only cut at the bottom:
   the side fades go, the bottom fade stays */
@media (max-width: 639px) {
  .stage--fade-r { -webkit-mask: none; mask: none; }
  .stage--fade-b, .stage--fade-br {
    -webkit-mask: linear-gradient(to bottom, #000 calc(100% - min(56px, var(--fade-h))), transparent calc(100% - 4px));
    mask: linear-gradient(to bottom, #000 calc(100% - min(56px, var(--fade-h))), transparent calc(100% - 4px));
  }
}

@keyframes ui-spin { to { transform: rotate(360deg); } }
```

## assets/ui/chat/fragment.css

```css
/* chat: a customer asks, the assistant answers word by word with the sources it used */
/* in a .to-edge wrapper the body grows to fill it, so the input always sits past the panel's edge */
.ui-chat { width: 400px; overflow: hidden; display: flex; flex-direction: column; }
.ui-chat > .ui-head, .ui-chat__input { flex: none; }
.ui-chat .ui-head .ui-agent { width: 20px; height: 20px; }
.ui-chat__online { display: inline-flex; align-items: center; gap: 5px; font-size: 13px; color: var(--u-muted); }
/* a quiet ink dot, as every status in the kit: no off-palette green */
.ui-chat__online::before { content: ""; width: 6px; height: 6px; border-radius: 50%; background: currentColor; opacity: 0.7; }
.ui-chat__tool { display: grid; place-items: center; width: 28px; height: 28px; border-radius: 6px; color: var(--u-muted); }
/* --chat-body sets the empty room above the input, so a placement can push the input past the panel's
   edge by a clear margin (cut on purpose) or keep it in view */
.ui-chat__body { flex: 1 0 auto; display: grid; gap: 12px; padding: 12px 16px 16px; min-height: var(--chat-body, 216px); align-content: start; }
.ui-chat__day { justify-self: center; font-size: 13px; line-height: 16px; color: var(--u-muted); }
.ui-msg { font-size: 13px; line-height: 20px; transition: opacity 260ms var(--u-ease), transform 360ms var(--u-ease); }
.ui-msg--me {
  justify-self: end; max-width: 280px; padding: 8px 12px; border-radius: 14px 14px 4px 14px;
  background: var(--u-fill-2); color: var(--u-ink); box-shadow: inset 0 0 0 1px var(--u-line);
}
.ui-msg--bot { display: flex; gap: 10px; align-items: flex-start; max-width: 340px; }
.ui-msg__av { width: 22px; height: 22px; margin-top: 1px; }
.ui-msg__col { display: grid; gap: 10px; min-width: 0; color: var(--u-text-2); }
.ui-msg--bot b { font-weight: 500; color: var(--u-ink); }
.ui-msg.is-hidden { opacity: 0; transform: translateY(6px); }
.ui-chat__srcs { display: flex; flex-wrap: wrap; gap: 6px; }
.ui-chat__src {
  display: inline-flex; align-items: center; gap: 6px; height: 24px; padding: 0 8px 0 6px;
  border-radius: 6px; font-size: 13px; font-weight: 500; line-height: 16px; color: var(--u-muted);
  background: var(--u-bg); box-shadow: inset 0 0 0 1px var(--u-line-2);
}
.ui-chat__src :is(.ui-icon, .ui-brand) { width: 13px; height: 13px; }
/* the answer's last sentence is written again in place: its words never drop below 72% ink */
.ui-chat__w { transition: opacity 180ms var(--u-ease); }
.ui-chat__w.is-off { opacity: 0.72; }
.ui-chat__srcs.is-hidden { opacity: 0; }
.ui-chat__src { transition: opacity 240ms var(--u-ease, ease), transform 240ms var(--u-ease, ease); }
.ui-chat__src.is-wait { opacity: 0; transform: translateY(3px); }
/* the typing dots: under the answer where its sources sit, only while the next words are on their way */
.ui-msg__col { position: relative; }
.ui-chat__typing { position: absolute; left: 0; bottom: 4px; display: flex; gap: 4px; opacity: 0; transition: opacity 160ms ease; pointer-events: none; }
.ui-chat__typing i { width: 5px; height: 5px; border-radius: 50%; background: var(--u-muted, #6b6b6b); animation: ui-chat-dot 900ms ease-in-out infinite; }
.ui-chat__typing i:nth-child(2) { animation-delay: 150ms; } .ui-chat__typing i:nth-child(3) { animation-delay: 300ms; }
.ui-chat.is-typing .ui-chat__typing { opacity: 1; }
@keyframes ui-chat-dot { 0%, 60%, 100% { transform: none; opacity: .45; } 30% { transform: translateY(-3px); opacity: 1; } }
@media (prefers-reduced-motion: reduce) { .ui-chat__typing { display: none; } }
.ui-chat__srcs { transition: opacity 320ms var(--u-ease); }
.ui-chat__input {
  display: flex; align-items: center; gap: 8px; height: 40px; margin: 0 12px 12px; padding: 0 4px 0 12px;
  border-radius: 10px; background: var(--u-fill); box-shadow: inset 0 0 0 1px var(--u-line-2);
  font-size: 13px; color: var(--u-soft);
}
.ui-chat__ph { flex: 1; }
.ui-chat__send { display: grid; place-items: center; width: 30px; height: 30px; border-radius: 7px; background: var(--u-ink); color: var(--u-bg); }
```

## assets/ui/chat/fragment.js

```js
// chat: one cycle (about 2.8 s): the question stays; the answer's sources fade, the answer's last sentence
// is written again word by word (its words step back to about 72% ink, then come up in order in under
// 0.8 s, like the end of a streamed reply), its sources come back, and the finished answer holds for the
// rest of the cycle. The rest of the answer never leaves full ink; no word ever drops below 72% (4.5:1 on white).
import { loop } from '../ui.js';

// wrap every word of the answer in a span (inside <b> too), so the words can light up in order
function words(el) {
  const out = [];
  const walk = node => {
    for (const n of [...node.childNodes]) {
      if (n.nodeType === 3) {
        const parts = n.textContent.split(/(\s+)/);
        const frag = document.createDocumentFragment();
        for (const p of parts) {
          if (!p) continue;
          if (/^\s+$/.test(p)) { frag.append(p); continue; }
          const s = document.createElement('span');
          s.className = 'ui-chat__w';
          s.textContent = p;
          out.push(s);
          frag.append(s);
        }
        n.replaceWith(frag);
      } else if (n.nodeType === 1) walk(n);
    }
  };
  walk(el);
  return out;
}

export default function init(root) {
  const answer = root.querySelector('.ui-chat__answer');
  const srcs = root.querySelector('.ui-chat__srcs');
  const all = answer ? words(answer) : [];
  if (!all.length) return null;
  // the last sentence (a word ending . ! or ? before the last word starts it), at most 12 words;
  // a one-sentence answer writes its second half
  let from = 0;
  for (let i = all.length - 2; i >= 0; i--) if (/[.!?]$/.test(all[i].textContent)) { from = i + 1; break; }
  if (from === 0) from = Math.floor(all.length / 2);
  from = Math.max(from, all.length - 12);
  const w = all.slice(from);
  const full = () => { w.forEach(s => s.classList.remove('is-off')); if (srcs) srcs.classList.remove('is-hidden'); };
  const per = Math.max(40, Math.min(90, 720 / w.length));
  const each = srcs ? [...srcs.children] : [];
  // one cycle (about 2.6 s): the sources fade, the sentence steps back, the typing dots run 0.6 s, the
  // sentence is written again in under 0.8 s, the sources come back one by one, the answer holds 0.38 s
  return loop(root, async api => {
    if (srcs) srcs.classList.add('is-hidden');
    await api.wait(200);
    w.forEach(s => s.classList.add('is-off'));
    root.classList.add('is-typing');
    await api.wait(780);
    root.classList.remove('is-typing');
    for (const s of w) { s.classList.remove('is-off'); await api.wait(per); }
    await api.wait(160);
    each.forEach(e => e.classList.add('is-wait'));
    if (srcs) srcs.classList.remove('is-hidden');
    for (const e of each) { e.classList.remove('is-wait'); await api.wait(240); }
    await api.wait(400);
  }, { still: () => { root.classList.remove('is-typing'); each.forEach(e => e.classList.remove('is-wait')); full(); } });
}
```

## assets/ui/checklist/fragment.css

```css
/* checklist: items tick one after another and the progress bar fills */
.ui-checks { width: 340px; padding-bottom: 8px; }
.ui-checks__head { display: flex; align-items: center; height: 48px; padding: 0 16px; }
.ui-checks__bar { position: relative; height: 4px; margin: 0 16px 8px; border-radius: 2px; background: var(--u-line); overflow: hidden; }
.ui-checks__bar i { position: absolute; inset: 0; border-radius: 2px; background: var(--u-accent); transform-origin: left; transform: scaleX(var(--k, 0)); transition: transform 500ms var(--u-ease); }
.ui-check { display: flex; align-items: center; gap: 12px; height: 40px; padding: 0 16px; font-size: 13px; line-height: 20px; }
.ui-check__box {
  display: grid; place-items: center; flex: none; width: 18px; height: 18px; border-radius: 50%;
  box-shadow: inset 0 0 0 1.5px var(--u-line-2); color: var(--u-bg);
  transition: background-color 240ms var(--u-ease), box-shadow 240ms var(--u-ease), transform 300ms var(--u-spring);
}
.ui-check__box svg { width: 12px; height: 12px; fill: none; stroke: currentColor; stroke-width: 2.2; stroke-linecap: round; stroke-linejoin: round; }
.ui-check__box path { stroke-dasharray: 12; stroke-dashoffset: 12; transition: stroke-dashoffset 300ms var(--u-ease) 80ms; }
.ui-check__label { position: relative; transition: color 300ms var(--u-ease); }
.ui-check__label::after {
  content: ""; position: absolute; left: 0; right: 0; top: 50%; height: 1px; background: currentColor;
  transform: scaleX(0); transform-origin: left; transition: transform 360ms var(--u-ease) 120ms;
}
.ui-check.is-done .ui-check__box { background: var(--u-ink); box-shadow: none; }
/* the bar is the one accent; the step that ticks is the ink, a little larger for a moment */
.ui-check.is-now .ui-check__box { background: var(--u-ink); color: var(--u-bg); }
.ui-check.is-done .ui-check__box path { stroke-dashoffset: 0; }
.ui-check.is-done .ui-check__label { color: var(--u-muted); }
.ui-check.is-done .ui-check__label::after { transform: scaleX(1); }
.ui-check.is-now .ui-check__box { transform: scale(1.12); }
.ui-checks.is-reset * { transition-duration: 200ms !important; transition-delay: 0ms !important; }
@media (max-width: 360px) { .ui-checks { width: 100%; } }
```

## assets/ui/checklist/fragment.js

```js
// checklist: one cycle (about 3.2 s): back to the page's own state, then the open items tick in turn
// (check draws, label strikes), the bar fills; it holds on all done, then starts again. The count keeps
// the page's words: only its first figure follows ("2 of 5" reads "3 of 5").
import { loop } from '../ui.js';

export default function init(root) {
  const items = [...root.querySelectorAll('.ui-check')];
  if (!items.length) return null;
  const bar = root.querySelector('[data-checks-bar]'), count = root.querySelector('[data-checks-count]');
  const start = items.filter(i => i.classList.contains('is-done')).length;
  const count0 = count ? count.textContent : '';
  const show = n => {
    items.forEach((it, i) => it.classList.toggle('is-done', i < n));
    if (bar) bar.style.setProperty('--k', n / items.length);
    if (count) count.textContent = n === start ? count0 : count0.replace(/\d+/, String(n));
  };
  const left = items.length - start;
  const step = Math.max(320, Math.min(900, 2000 / Math.max(1, left)));
  return loop(root, async api => {
    root.classList.add('is-reset');
    show(start);
    await api.wait(300);
    root.classList.remove('is-reset');
    for (let n = start + 1; n <= items.length; n++) {
      items[n - 1].classList.add('is-now');
      show(n);
      await api.wait(step);
      items[n - 1].classList.remove('is-now');
    }
    await api.wait(390);
  }, { still: () => { root.classList.remove('is-reset'); items.forEach(i => i.classList.remove('is-now')); show(start); } });
}
```

## assets/ui/notify/fragment.css

```css
/* notify: a short stack of notifications; a new one slides in on top, the oldest leaves */
.ui-notify { width: 380px; }
.ui-notify__list { position: relative; display: grid; gap: 8px; }
.ui-note {
  display: flex; align-items: center; gap: 12px; height: 64px; padding: 0 14px 0 12px;
  border-radius: var(--u-r); background: var(--u-bg); border: 1px solid var(--u-border);
  box-shadow: var(--u-shadow-sm);
  transition: transform 520ms var(--u-ease), opacity 360ms var(--u-ease);
}
.ui-note:first-child { box-shadow: var(--u-shadow); }
.ui-note__av { width: 32px; height: 32px; font-size: 13px; }
.ui-note__ico { display: grid; place-items: center; flex: none; width: 32px; height: 32px; border-radius: 50%; background: var(--u-fill-2); color: var(--u-muted); box-shadow: inset 0 0 0 1px var(--u-line); }
.ui-note__ico svg { width: 16px; height: 16px; }
.ui-note__text { display: grid; flex: 1; min-width: 0; }
.ui-note__title { display: flex; align-items: baseline; gap: 8px; font-size: 13px; font-weight: 500; line-height: 20px; white-space: nowrap; }
.ui-note__title time { margin-left: auto; font-size: 13px; font-weight: 400; color: var(--u-soft); }
.ui-note__body { font-size: 13px; line-height: 16px; color: var(--u-muted); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.ui-note__btn { margin-left: 4px; }
.ui-note:has(.ui-note__btn) .ui-note__title time { display: none; }
.ui-note.is-enter { opacity: 0; transform: translateY(-14px) scale(0.98); }
.ui-note.is-leave { opacity: 0; transform: translateY(8px) scale(0.98); }
/* phone: the stack takes the panel's width (cards.css) and a long line wraps instead of being cut */
@media (max-width: 639px) {
  .ui-note { height: auto; min-height: 64px; padding: 12px 12px 12px 12px; align-items: flex-start; }
  .ui-note__av, .ui-note__ico { margin-top: 2px; }
  .ui-note__title, .ui-note__body { white-space: normal; text-wrap: pretty; }
  .ui-note__btn { align-self: center; }
}
```

## assets/ui/notify/fragment.js

```js
// notify: one cycle (about 3 s): a notification slides in at the top and the oldest fades out at the
// bottom (FLIP for the shift); hold. What slides in is the page's own: opts.items [[icon, title, body]]
// (HTML) when given, else the oldest notification of the stack comes round to the top again. The times
// stay with their places ("now" at the top), and only the top one keeps its button.
import { loop } from '../ui.js';

export default function init(root, opts = {}) {
  const list = root.querySelector('[data-notify-list]');
  if (!list || !list.children.length) return null;
  const keep = list.children.length;
  const list0 = list.innerHTML;
  const times = [...list.children].map(li => li.querySelector('time')?.textContent ?? null);
  const fresh = opts.items ? opts.items.map(([ico, title, body]) => {
    const li = document.createElement('li');
    li.className = 'ui-note';
    li.innerHTML = `${ico}<span class="ui-note__text"><span class="ui-note__title">${title}<time></time></span><span class="ui-note__body">${body}</span></span>`;
    return li;
  }) : null;
  // the finished picture is the stack as written
  let n = 0;
  const still = () => { n = 0; if (list.innerHTML !== list0) list.innerHTML = list0; };
  // one cycle (2.82 s): three notifications slide in in turn, the oldest leaving; the third is held 0.4 s
  return loop(root, async api => { for (let step = 0; step < 3; step++) {
    const kids = [...list.children];
    const before = kids.map(k => k.getBoundingClientRect().top);
    const old = kids[kids.length - 1];
    const li = fresh ? fresh[n++ % fresh.length].cloneNode(true) : old.cloneNode(true);
    li.classList.remove('is-leave');
    li.classList.add('is-enter');
    li.querySelector('.ui-note__btn')?.remove();
    old.classList.add('is-leave');
    await api.wait(260);
    old.style.position = 'absolute';
    old.style.visibility = 'hidden';
    list.prepend(li);
    kids.forEach((k, i) => {
      if (k === old) return;
      const dy = before[i] - k.getBoundingClientRect().top;
      k.style.transition = 'none';
      k.style.transform = `translateY(${dy}px)`;
    });
    li.getBoundingClientRect();
    kids.forEach(k => { if (k !== old) { k.style.transition = ''; k.style.transform = ''; } });
    li.classList.remove('is-enter');
    old.remove();
    while (list.children.length > keep) list.lastElementChild.remove();
    // the times belong to the places: the top one is always the newest; only it keeps a button
    [...list.children].forEach((k, i) => {
      const t = k.querySelector('time');
      if (t && times[i] != null) t.textContent = times[i];
      if (i) k.querySelector('.ui-note__btn')?.remove();
    });
    await api.wait(step < 2 ? 820 : 400);
  } }, { still });
}
```

## assets/ui/search/fragment.css

```css
/* search: a command bar; the query types in and the results follow */
.ui-search { width: 400px; overflow: hidden; }
.ui-search__field { display: flex; align-items: center; gap: 10px; height: 52px; padding: 0 12px 0 16px; border-bottom: 1px solid var(--u-line); color: var(--u-muted); }
.ui-search__field .ui-icon { width: 18px; height: 18px; }
.ui-search__q { flex: 1; display: flex; align-items: center; min-width: 0; font-size: 15px; line-height: 20px; color: var(--u-ink); white-space: pre; }
.ui-search__caret { width: 1.5px; height: 18px; margin-left: 1px; background: var(--u-ink); animation: ui-caret 1s steps(1) infinite; }
@keyframes ui-caret { 50% { opacity: 0; } }
.ui-search kbd {
  display: inline-grid; place-items: center; min-width: 20px; height: 20px; padding: 0 5px; border-radius: 5px;
  font: 500 13px/1 var(--ui-font, inherit); color: var(--u-muted); background: var(--u-fill); box-shadow: inset 0 0 0 1px var(--u-line);
}
.ui-search__group { padding: 12px 16px 4px; font-size: 13px; font-weight: 500; line-height: 16px; color: var(--u-muted); }
.ui-search__list { padding: 0 8px 8px; min-height: 128px; }
.ui-search__row {
  display: flex; align-items: center; gap: 10px; height: 40px; padding: 0 8px; border-radius: var(--u-r3);
  font-size: 13px; line-height: 20px; transition: opacity 240ms var(--u-ease), transform 300ms var(--u-ease), background-color 200ms var(--u-ease);
}
.ui-search__row.is-sel { background: var(--u-fill-2); }
.ui-search__row.is-hidden { opacity: 0; transform: translateY(4px); }
.ui-search__row.is-stale { opacity: 0.38; }
.ui-search__ico { display: grid; place-items: center; width: 28px; height: 28px; border-radius: 7px; background: var(--u-bg); box-shadow: inset 0 0 0 1px var(--u-line); color: var(--u-muted); }
.ui-search__row.is-sel .ui-search__ico { background: var(--u-bg); color: var(--u-ink); box-shadow: inset 0 0 0 1px var(--u-line-2); }
.ui-search__row.is-sel { box-shadow: inset 2px 0 0 var(--u-accent); }
.ui-search__t { flex: 1; min-width: 0; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; color: color-mix(in oklab, var(--u-ink) 75%, var(--u-bg)); }
.ui-search__t b { font-weight: 500; color: var(--u-ink); }
.ui-search__m { font-size: 13px; color: var(--u-muted); }
.ui-search__foot { display: flex; gap: 16px; height: 36px; align-items: center; padding: 0 16px; border-top: 1px solid var(--u-line); font-size: 13px; color: var(--u-muted); }
.ui-search__foot span { display: inline-flex; align-items: center; gap: 4px; }
.ui-search__foot kbd { min-width: 20px; height: 20px; font-size: 13px; }
@media (max-width: 420px) { .ui-search { width: 100%; } }
```

## assets/ui/search/fragment.js

```js
// search: one query per cycle (about 2.7 s): the old results dim, the query is cleared and typed letter by
// letter, results appear one by one, the selection steps down; hold. The queries are the page's own:
// the query and results as written in the markup come first, then opts.queries [{q, rows: [[icon,
// title, meta]]}] if any; with none, it types its own query again.
import { loop } from '../ui.js';

const rowsHTML = rows => rows.map(([ico, t, m]) => `<li class="ui-search__row"><span class="ui-search__ico">${ico}</span><span class="ui-search__t">${t}</span><span class="ui-search__m">${m}</span></li>`).join('');

export default function init(root, opts = {}) {
  const q = root.querySelector('[data-search-q]');
  const list = root.querySelector('[data-search-list]');
  if (!q || !list) return null;
  // the finished picture is the markup as written (its query and its results)
  const q0 = q.textContent, list0 = list.innerHTML;
  const extra = (opts.queries || []).map(it => ({ q: it.q, html: rowsHTML(it.rows || []) })).filter(it => it.q !== q0);
  const all = [{ q: q0, html: list0 }, ...extra];
  let k = 0;
  const still = () => { k = 0; q.textContent = q0; list.innerHTML = list0; };
  const reveal = html => {
    list.innerHTML = html;
    const rows = [...list.children];
    rows.forEach(r => { r.classList.remove('is-sel', 'is-stale'); r.classList.add('is-hidden'); });
    rows.forEach((r, i) => setTimeout(() => r.classList.remove('is-hidden'), 30 + i * 80));
    return rows;
  };
  return loop(root, async api => {
    const item = all[++k % all.length];
    // the old results dim while the query is cleared and stay until the new ones replace them, so the
    // list is never empty
    [...list.children].forEach(r => r.classList.add('is-stale'));
    // a long query types faster, so the cycle stays under 4 s
    const del = Math.min(14, 260 / Math.max(1, q.textContent.length)), per = Math.min(58, 900 / Math.max(1, item.q.length));
    for (let i = q.textContent.length; i >= 0; i--) { q.textContent = q.textContent.slice(0, i); await api.wait(del); }
    await api.wait(160);
    let rows = null;
    for (let i = 1; i <= item.q.length; i++) {
      q.textContent = item.q.slice(0, i);
      await api.wait(Math.round(per * (0.7 + (i % 3) * 0.3)));
      if (i === Math.min(3, item.q.length)) rows = reveal(item.html);
    }
    if (!rows) rows = reveal(item.html);
    if (!rows.length) { await api.wait(1600); return; }
    rows[0].classList.add('is-sel');
    await api.wait(520);
    for (let j = 1; j < rows.length; j++) {
      rows[j - 1].classList.remove('is-sel');
      rows[j].classList.add('is-sel');
      await api.wait(340);
    }
    await api.wait(400);
  }, { still });
}
```

## assets/ui/toggles/fragment.css

```css
/* toggles: a settings list with switches; one flips at a time */
.ui-toggles { width: 380px; overflow: hidden; }
.ui-toggles__ico { color: var(--u-muted); }
.ui-toggles__list { padding: 4px 0; }
.ui-tg { display: flex; align-items: center; gap: 16px; min-height: 58px; padding: 8px 16px; transition: background-color 300ms var(--u-ease); }
.ui-tg + .ui-tg { box-shadow: inset 0 1px 0 var(--u-line); }
.ui-tg__text { display: grid; flex: 1; min-width: 0; }
.ui-tg__label { font-size: 13px; font-weight: 500; line-height: 20px; }
.ui-tg__sub { font-size: 13px; line-height: 16px; color: var(--u-muted); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.ui-switch {
  position: relative; flex: none; width: 32px; height: 20px; border-radius: 999px;
  background: var(--u-fill-3); box-shadow: inset 0 0 0 1px var(--u-line);
  transition: background-color 240ms var(--u-ease), box-shadow 240ms var(--u-ease);
}
.ui-switch i {
  position: absolute; top: 2px; left: 2px; width: 16px; height: 16px; border-radius: 50%;
  background: #fff; box-shadow: 0 1px 2px rgb(0 0 0 / 0.25), 0 0 0 0.5px rgb(0 0 0 / 0.08);
  transition: transform 300ms var(--u-spring);
}
.ui-switch.is-on { background: var(--u-ink); box-shadow: none; }
.ui-switch.is-on i { transform: translateX(12px); background: var(--u-bg); }
/* the row that just changed carries the one accent touch */
.ui-tg.is-active { background: color-mix(in oklab, var(--u-accent) 6%, var(--u-bg)); }
.ui-tg.is-active .ui-switch.is-on { background: var(--u-accent); }
.ui-tg.is-active .ui-switch.is-on i { background: #fff; }
/* phone: the list takes the panel's width (cards.css); a long line wraps instead of being cut */
@media (max-width: 639px) {
  .ui-tg { gap: 12px; padding: 8px 14px; }
  .ui-tg__sub { white-space: normal; }
}
```

## assets/ui/toggles/fragment.js

```js
// toggles: one switch per cycle (about 2.6 s): its row lights, it flips, the count follows; hold. The
// switches flip in a fixed order that comes back to the page's own state every six cycles. The count
// keeps the page's words: only its first figure follows ("4 of 5 on" reads "3 of 5 on").
import { loop } from '../ui.js';

export default function init(root, opts = {}) {
  const rows = [...root.querySelectorAll('.ui-tg')];
  const sw = rows.map(r => r.querySelector('.ui-switch'));
  if (!rows.length || sw.some(s => !s)) return null;
  const count = root.querySelector('[data-tg-count]');
  const count0 = count ? count.textContent : '';
  const initial = sw.map(s => s.classList.contains('is-on'));
  const n = rows.length, a = Math.min(2, n - 1), c = Math.min(3, n - 1);
  const order = (opts.order || [a, 0, a, c, 0, c]).filter(i => i < n);
  const set = (i, on) => { sw[i].classList.toggle('is-on', on); sw[i].setAttribute('aria-checked', String(on)); };
  const ons = () => sw.filter(s => s.classList.contains('is-on')).length;
  const on0 = initial.filter(Boolean).length;
  const tally = () => { if (count) count.textContent = ons() === on0 ? count0 : count0.replace(/\d+/, String(ons())); };
  let k = 0;
  // one cycle (2.62 s): two switches flip in turn and the count follows; the second is held 0.38 s
  return loop(root, async api => {
    await api.wait(240);
    for (let n = 0; n < 2; n++) {
      const i = order[k++ % order.length];
      rows[i].classList.add('is-active');
      set(i, !sw[i].classList.contains('is-on'));
      tally();
      await api.wait(900);
      rows[i].classList.remove('is-active');
      await api.wait(n < 1 ? 200 : 380);
    }
  }, { still: () => { k = 0; rows.forEach(r => r.classList.remove('is-active')); initial.forEach((on, i) => set(i, on)); tally(); } });
}
```

## assets/ui/ui.js

```js
/*
  ui.js - the taste-skill UI fragment kit. One ES module, no dependencies.

  A fragment is a folder next to this file: <name>/fragment.html, fragment.css, meta.json and,
  when it moves, fragment.js (export default function (root, opts) -> { destroy() }).

  Load into a placeholder (the HTML, both stylesheets and the script are fetched once):
    <div data-ui-load="calendar"></div>
    import ui from './assets/ui/ui.js'; ui.loadAll(document);
  Stylesheets: ui.js links base.css and each fragment's CSS from script, so with scripts off (or before it
  runs) a fragment would be unstyled and leave an empty ground. A page therefore links assets/ui/base.css and
  assets/ui/<name>/fragment.css for every fragment it uses in its own <head> (starter.html does); link() here
  skips a stylesheet that is already present, so nothing loads twice.
  Or paste fragment.html into the page, edit its text to the client's content, then
  ui.startAll(document): it links base.css and the fragment's CSS and reads data-ui-opts on the root.

  The fragment takes its colours from the custom properties around it:
    --fx-bg --fx-surface --fx-ink --fx-muted --fx-accent --fx-accent-ink, and --ui-font --ui-mono --ui-radius.

  Motion: one rule, the same as system.js. Put data-ui-loop on the group's one loop (the fragment,
  its placeholder or an element around it); every other fragment holds its finished picture.
  - With system.js the page needs nothing else: it marks every other fragment data-ui-still before
    it loads (and on a phone, under 640px, on kind other and on a place, every fragment).
  - Without system.js (cards.html), ui.loadAll / ui.startAll do the same as soon as the page holds
    any data-ui-loop. A page with none (the kit's index.html) lets every fragment loop.
  - data-ui-still on the placeholder (or any ancestor) holds a fragment by hand, from the moment its
    own load resolves. voice is held like the rest (its orb holds one frame).
  - data-ui-cycles="2" plays that many cycles when first seen, then rests on the finished picture.
  - One cycle is 2.5 to 4 s and repeats while on screen, 0.6 s after the fragment first shows. The
    pointer entering its card restarts it from the top; a held fragment plays while the pointer is on
    its card. ui.restart(el) restarts every fragment in el (a tab shown again).
  - A loop runs while on screen and gets a pause button (a solid 32px disc) when it starts to move:
    in a .card beside the card's title (never over the panel, which a cropped fragment fills to its
    corners); in [data-ui-pause-host], or a .panel/.frame outside a card, 12px in from its bottom-right
    corner; with no panel, under the fragment. Not where the place already holds a .loop-pause. Under prefers-reduced-motion nothing
    moves and no button is shown.
  - A fragment's script never writes words of its own over the page's: every state it shows comes
    from the markup (data-* beside the text it changes) or from opts.
  Grounds: ui.grounds(document, fx) mounts the [data-fx] grounds of a page of cards. A still ground is
  drawn once at the screen's sharpness (up to 2 device pixels) and kept as a plain picture, its WebGL
  context freed; a ground a fragment drives stays live (at 1 device pixel) only near the screen. So
  the page needs one or two contexts. See grounds() below.
*/
const BASE = new URL('./', import.meta.url);
const reducedQuery = typeof matchMedia === 'function' ? matchMedia('(prefers-reduced-motion: reduce)') : null;
export const reduced = () => !!(reducedQuery && reducedQuery.matches);
/** true when el sits under data-ui-still: it shows its finished picture and never cycles */
export const isStill = el => !!(el && el.closest && el.closest('[data-ui-still]'));

const css = new Map();
function link(href) {
  if (!css.has(href)) {
    const done = [...document.querySelectorAll('link[rel="stylesheet"]')].some(l => l.href === href);
    css.set(href, done ? Promise.resolve() : new Promise(res => {
      const l = document.createElement('link');
      l.rel = 'stylesheet';
      l.href = href;
      l.onload = l.onerror = () => res();
      document.head.appendChild(l);
    }));
  }
  return css.get(href);
}
const cache = new Map();
const fetchText = url => {
  if (!cache.has(url)) cache.set(url, fetch(url).then(r => { if (!r.ok) throw new Error(`ui: ${r.status} ${url}`); return r.text(); }));
  return cache.get(url);
};

let uid = 0;
// SVG ids (gradients, masks) must be unique per copy of a fragment
function uniqueIds(root) {
  const n = ++uid;
  for (const el of root.querySelectorAll('[id]')) {
    const old = el.id, id = `${old}-u${n}`;
    el.id = id;
    for (const ref of root.querySelectorAll('*')) {
      for (const a of ['fill', 'stroke', 'mask', 'clip-path', 'filter', 'href', 'xlink:href']) {
        const v = ref.getAttribute(a);
        if (!v || !v.includes('#' + old)) continue;
        const nv = v.split(`url(#${old})`).join(`url(#${id})`);
        ref.setAttribute(a, nv === '#' + old ? '#' + id : nv);
      }
    }
  }
}

const mods = new Map();
function script(name) {
  if (!mods.has(name)) mods.set(name, import(new URL(`${name}/fragment.js`, BASE).href).then(m => m.default));
  return mods.get(name);
}

/** fill el with the fragment `name` and start it; returns its controller (or null) */
export async function load(el, name, opts = {}) {
  if (!/^[a-z0-9-]+$/.test(name)) throw new Error(`ui: bad fragment name "${name}"`);
  const base = new URL(`${name}/`, BASE);
  const [meta, html] = await Promise.all([
    fetchText(new URL('meta.json', base).href).then(JSON.parse),
    fetchText(new URL('fragment.html', base).href),
    link(new URL('base.css', BASE).href),
    link(new URL('fragment.css', base).href),
  ]);
  // the kit's sample marks live in assets/ui/icons: a fragment pasted into the project's index.html asks for assets/ui/icons/ (from the project root),
  // and one loaded here is pointed at the kit's own folder, whatever page loads it
  el.innerHTML = html.replace(/src="(?:assets\/ui\/)?icons\//g, `src="${new URL('icons/', BASE).href}`);
  const root = el.querySelector('[data-ui]') || el.firstElementChild;
  uniqueIds(root);
  return meta.js ? start(root, opts) : null;
}

/** start a fragment already in the page (its markup pasted from fragment.html and edited to the
    client's content); links its stylesheets, reads data-ui-opts='{...}' on the root for its options */
export async function start(root, opts) {
  if (root.__ui) return root.__ui;
  const name = root.dataset.ui;
  if (!/^[a-z0-9-]+$/.test(name || '')) return null;
  if (!opts) { try { opts = root.dataset.uiOpts ? JSON.parse(root.dataset.uiOpts) : {}; } catch { opts = {}; } }
  await Promise.all([link(new URL('base.css', BASE).href), link(new URL(`${name}/fragment.css`, BASE).href)]);
  const init = await script(name).catch(() => null);
  if (!init) return null;
  root.__ui = init(root, opts) || { destroy() {} };
  return root.__ui;
}

/*
  The motion plan, as system.js deals it: once the page marks a loop (data-ui-loop), every fragment
  that is not that loop or inside one is held still; on a phone (under 640px), on kind other
  (<meta name="site-kind" content="other">) and on a place nothing runs by itself, voice included.
  A page with no data-ui-loop anywhere leaves every fragment to loop (a gallery of the kit).
*/
const LOOPS = '[data-ui-loop], [data-ui-cycles]';
export function plan(root = document) {
  const doc = root.ownerDocument || root;
  if (!doc.querySelector || !doc.querySelector('[data-ui-loop]')) return;
  const kind = doc.querySelector('meta[name="site-kind"]');
  const other = (doc.documentElement && doc.documentElement.dataset.type === 'place') || (!!kind && kind.content.trim().toLowerCase() === 'other');
  const phone = other || (typeof matchMedia === 'function' && matchMedia('(max-width: 639px)').matches);
  for (const el of root.querySelectorAll('[data-ui-load], [data-ui]')) {
    if (el.parentElement && el.parentElement.closest('[data-ui-load]')) continue;   // inside a loaded one
    if (phone || !el.closest(LOOPS)) el.setAttribute('data-ui-still', '');
  }
}

/* every placeholder resolves on its own: a still one is held the moment its own load is done */
export function loadAll(root = document) {
  plan(root);
  return Promise.all([...root.querySelectorAll('[data-ui-load]')].map(el => {
    let opts = {};
    try { opts = el.dataset.uiOpts ? JSON.parse(el.dataset.uiOpts) : {}; } catch (e) { console.warn('ui: bad data-ui-opts', el); }
    return load(el, el.dataset.uiLoad, opts).catch(e => { console.warn(e); return null; });
  }));
}
export function startAll(root = document) {
  plan(root);
  return Promise.all([...root.querySelectorAll('[data-ui]')].map(el => start(el)));
}

/** hold a fragment (its placeholder or root) on its finished picture from now on; it still plays while
    the pointer is on its card */
export function hold(el) {
  el.setAttribute('data-ui-still', '');
  const root = el.matches('[data-ui]') ? el : el.querySelector('[data-ui]');
  if (!root || !root.__ui) return;
  if (root.__ui.hold) root.__ui.hold();
  else { root.__ui.destroy(); root.__ui = null; }
}

/* ---------------------------------------------------------------- pause button */
const ICON_PAUSE = '<svg viewBox="0 0 16 16" aria-hidden="true"><rect x="4.25" y="3.25" width="2.75" height="9.5" rx="0.9"/><rect x="9" y="3.25" width="2.75" height="9.5" rx="0.9"/></svg>';
const ICON_PLAY = '<svg viewBox="0 0 16 16" aria-hidden="true"><path d="M5.25 3.7v8.6c0 .62.68 1 1.2.66l6.5-4.3c.47-.31.47-1 0-1.32l-6.5-4.3c-.52-.34-1.2.04-1.2.66Z"/></svg>';
const boxes = new WeakMap();
const boxOf = root => root.closest('[data-ui-pause-host], .panel__screen, .panel, .frame') || root.parentElement;
// a card's title: the first h3 or .card__title in the card that is not inside its panel
const titleOf = card => [...card.querySelectorAll('h3, .card__title')].find(t => !t.closest('.panel, .frame, .ui') && t.closest('.card') === card);

/**
  pausable(box, ctrl): the same pause button for anything else that moves on its own in a panel for
  more than about 5 s (an object from assets/objects, a moving ground): ctrl needs pause() and
  resume(). One button per place, shared by every loop in it. Returns a function that removes it.
  Nothing is added under reduced motion (those hold still by themselves).
  Where it goes, so it never covers what the panel shows (a cropped panel runs its UI into every corner):
  - in a .card with a title: beside the title, at the right of the card's text column (the title keeps
    44px clear for it); one button for the card;
  - a box marked [data-ui-pause-host]: its bottom-right corner, as the page chose;
  - a .panel or .frame outside a card: its bottom-right corner (keep that corner clear);
  - a fragment with no panel round it: under its bottom-right corner, in 44px kept free below it.
*/
export function pausable(box, ctl) {
  if (!box || reduced()) return () => {};
  const explicit = box.matches('[data-ui-pause-host]');
  const card = explicit ? null : box.closest('.card');
  const title = card ? titleOf(card) : null;
  const key = title ? card : box;
  let p = boxes.get(key);
  if (!p) {
    if (box.querySelector(':scope > .loop-pause') || (title && card.querySelector(':scope > .loop-pause'))) return () => {};   // the page put its own there
    const btn = document.createElement('button');
    btn.type = 'button';
    btn.className = 'ui-pause loop-pause';
    p = { btn, ctls: new Set(), paused: false, off: () => {} };
    const show = () => {
      const label = `${p.paused ? 'Play' : 'Pause'} animation`;
      btn.innerHTML = p.paused ? ICON_PLAY : ICON_PAUSE;
      btn.setAttribute('aria-label', label);
      btn.title = label;
      btn.classList.toggle('is-paused', p.paused);
    };
    btn.addEventListener('click', () => { p.paused = !p.paused; show(); p.ctls.forEach(c => (p.paused ? c.pause() : c.resume())); });
    show();
    if (title) {
      // beside the title: top on the title's first line, right on the text column's right edge
      title.classList.add('ui-pause-beside');
      btn.classList.add('ui-pause--title');
      const place = () => {
        const c = card.getBoundingClientRect(), t = title.getBoundingClientRect(), col = title.parentElement;
        const cs = getComputedStyle(col), r = col.getBoundingClientRect();
        const colRight = r.right - (parseFloat(cs.paddingRight) || 0) - (parseFloat(cs.borderRightWidth) || 0);
        const lh = parseFloat(getComputedStyle(title).lineHeight) || 26;
        const k = c.width / (card.offsetWidth || c.width) || 1;      // a scaled card (fit) measures in its own px
        btn.style.top = `${Math.round((t.top - c.top) / k + (lh - 32) / 2)}px`;
        btn.style.right = `${Math.max(8, Math.round((c.right - colRight) / k))}px`;
      };
      if (getComputedStyle(card).position === 'static') card.style.position = 'relative';
      card.append(btn);
      place();
      const ro = typeof ResizeObserver === 'function' ? new ResizeObserver(place) : null;
      if (ro) { ro.observe(card); ro.observe(title); }
      p.off = () => { if (ro) ro.disconnect(); };   // the title keeps its 44px: no reflow when the button goes
    } else {
      const bare = !explicit && !box.matches('.panel__screen, .panel, .frame');
      if (getComputedStyle(box).position === 'static') box.style.position = 'relative';
      if (bare) {
        // no panel round it: the fragment fills its box to every corner, so the button sits under it
        btn.classList.add('ui-pause--below');
        if (!box.dataset.uiPauseRoom) { box.dataset.uiPauseRoom = ''; box.style.paddingBottom = `calc(${getComputedStyle(box).paddingBottom} + 44px)`; }
      }
      box.append(btn);
    }
    boxes.set(key, p);
  }
  p.ctls.add(ctl);
  if (p.paused) ctl.pause();
  return () => { p.ctls.delete(ctl); if (!p.ctls.size) { p.btn.remove(); p.off(); boxes.delete(key); } };
}
const addPause = (root, ctl) => pausable(boxOf(root), ctl);

/* ---------------------------------------------------------------- the finished picture, tidied */
/* A finished picture keeps some text hidden on purpose (the other state of a button, a tag shown on hover, a label cut where it
   would meet the edge). That text is hidden from assistive technology too (aria-hidden), so nothing in a fragment reads as
   present but invisible; it is taken off again when the fragment starts to move. */
const tidying = new WeakMap();
function sweep(root) {
  const walk = (el, up) => {
    if (typeof SVGElement === 'function' && el instanceof SVGElement) return;
    const cs = getComputedStyle(el);
    const self = cs.display === 'none' || cs.visibility === 'hidden' || parseFloat(cs.opacity) < 0.05;
    if (self && !up && cs.display !== 'none' && el.textContent.trim()) {
      if (!el.hasAttribute('aria-hidden')) { el.setAttribute('aria-hidden', 'true'); el.dataset.uiAh = ''; }
    } else if (el.dataset.uiAh != null && !(self && !up)) { el.removeAttribute('aria-hidden'); delete el.dataset.uiAh; }
    for (const c of el.children) walk(c, up || self);
  };
  for (const c of root.children) walk(c, false);
}
function tidy(root) {
  if (tidying.has(root)) return;
  tidying.set(root, requestAnimationFrame(() => requestAnimationFrame(() => { tidying.delete(root); if (root.isConnected) sweep(root); })));
}
function untidy(root) {
  if (tidying.has(root)) { cancelAnimationFrame(tidying.get(root)); tidying.delete(root); }
  for (const el of root.querySelectorAll('[data-ui-ah]')) { el.removeAttribute('aria-hidden'); delete el.dataset.uiAh; }
}

/* ---------------------------------------------------------------- the loop */
const STOP = Symbol('stop'), AGAIN = Symbol('again');
// the card (or tile, or link) a fragment sits in: the pointer entering it restarts the fragment's cycle
const hostOf = root => root.closest('[data-ui-hover], .card, .tile, .menu-card, a') || boxOf(root);
const hosted = new WeakMap();
const QUARTER = 800;        // a restart within this long of a cycle's start changes nothing (no stutter)

/* one set of listeners per card: the pointer (a mouse or pen, never a tap) or keyboard focus entering it
   restarts every loop in it from the top; a card whose fragments are all held plays its top fragment
   (the last in the page, the dialog over a window) while the pointer stays, and rests when it leaves.
   Keyboard focus plays it once only: a held fragment never loops with no pause button while a link in
   its card keeps focus */
function hostListen(host, rec) {
  let set = hosted.get(host);
  if (!set) {
    set = new Set();
    hosted.set(host, set);
    const enter = pointer => {
      const live = [...set].filter(r => r.running());
      if (live.length) { live.forEach(r => r.restart()); return; }
      const top = [...set].sort((a, b) => (a.root.compareDocumentPosition(b.root) & Node.DOCUMENT_POSITION_FOLLOWING ? -1 : 1)).pop();
      if (top) (pointer ? top.hover(true) : top.restart());
    };
    const leave = () => set.forEach(r => r.hover(false));
    host.addEventListener('pointerenter', e => { if (e.pointerType !== 'touch') enter(true); });
    host.addEventListener('pointerleave', leave);
    host.addEventListener('focusin', e => { if (!e.target.closest('.loop-pause')) enter(false); });   // not the pause button itself
    host.addEventListener('focusout', e => { if (!host.contains(e.relatedTarget)) leave(); });
  }
  set.add(rec);
  return () => set.delete(rec);
}

/**
  Run cycle(api) again and again while the fragment is on screen. One cycle is 2.5 to 4 s and ends on
  the finished picture (references/motion.md). api.wait(ms) waits that long of on-screen time;
  api.tween(ms, fn, ease) calls fn(0..1) on frames; api.alive is false after destroy.
  The waits and tweens of a cycle add up on the cycle's own timeline (api.time is its clock): each one
  ends at its place on it, so a busy page (six loops at once, a CPU renderer, a scroll) cannot stretch
  the cycle past its plan, a late step just catches up. The dev warning reads the plan (the sum) and the
  time it took (4.4 s at most); ui-cycle-end on the root says both ({ ms, planned, stalled }).
  A held or reduced-motion fragment hides its intentionally hidden text from assistive technology too.
  Returns { destroy, pause, resume, restart, hold }.
  - still() (the finished picture) is shown at once and held `rest` ms (600) before the first cycle,
    so a visitor or a screenshot never meets an empty window.
  - The pointer entering the fragment's card (.card, .tile, a, or [data-ui-hover]) restarts it from
    the top (not in its first 800 ms, not while paused); restart() does the same for the page (a tab
    shown again). Restarting shows still() and runs the cycle at once, so a cycle must start from the
    finished picture without a jump.
  - Held (data-ui-still, or its `cycles` done): it shows still() and plays its cycle only while the
    pointer is on its card, then rests; restart() plays it once. Under reduced motion nothing plays.
  - `cycles` (or data-ui-cycles on an ancestor) plays that many cycles, then holds; `pause: false` adds
    no button (the page runs its own). Page loops use this as well:
    ui.loop(el, async api => { ... }, { still: () => showFinished(el) })
*/
export function loop(root, cycle, { still, rest = 600, rootMargin = '0px', cycles, pause = true } = {}) {
  const show = () => { if (still) still(); tidy(root); };
  const setHeld = on => root.classList.toggle('is-ui-held', on);
  if (reduced()) {
    setHeld(true);
    show();
    const none = () => {};
    return { destroy: none, pause: none, resume: none, restart: none, hold: none };
  }
  const cyclesHost = root.closest('[data-ui-cycles]');
  const maxCycles = cycles || (cyclesHost ? Math.max(1, parseInt(cyclesHost.dataset.uiCycles, 10) || 2) : 0);
  let mode = isStill(root) ? 'held' : 'run';
  let want = mode === 'run' ? Infinity : 0;      // cycles still to play
  let needRest = mode === 'run' && rest > 0;
  let visible = false, stopped = false, paused = false, hovering = false, unpause = null;
  let gen = 0, cur = 0, began = 0, played = 0;     // gen moves on at every restart; cur is the cycle's own
  let stalled = false, warned = false;             // a cycle that waited off screen or paused is not measured
  const queue = [], aborts = new Set(), wake = [];
  const active = () => visible && !paused && !document.hidden && !stopped;
  const flush = () => { if (!active()) return; while (queue.length) queue.shift()(); };
  const abort = why => { gen++; const fs = [...aborts]; aborts.clear(); queue.length = 0; fs.forEach(f => f(why)); };
  const kick = () => { while (wake.length) wake.shift()(); };
  // the cycle's own clock: the waits of a cycle add up on a timeline that starts when the cycle starts,
  // and each wait ends at its place on that timeline, not a delay after the last one. So a busy main
  // thread (a CPU renderer, six loops on one page, a scroll) cannot stretch the cycle: a wait that
  // comes late is shortened and the cycle catches up. Time spent off screen or paused is not counted.
  let t0 = 0, planned = 0, lost = 0;
  const LAG = 1200;       // behind by more than this (a stalled tab): give up catching up and go on from here
  const gate = () => new Promise((res, rej) => {
    if (stopped) return rej(STOP);
    if (cur !== gen) return rej(AGAIN);
    if (active()) return res();
    stalled = true;
    const from = performance.now();
    const ab = why => rej(why);
    aborts.add(ab);
    queue.push(() => { aborts.delete(ab); lost += performance.now() - from; res(); });
  });
  const sleep = ms => new Promise((res, rej) => {
    const ab = why => { clearTimeout(t); rej(why); };
    const t = setTimeout(() => { aborts.delete(ab); res(); }, ms);
    aborts.add(ab);
  });
  // behind the timeline: give the page one frame (at most 50 ms) to paint what the last step changed
  const frame = () => new Promise((res, rej) => {
    let done = false, t = 0;
    const go = () => { if (done) return; done = true; clearTimeout(t); aborts.delete(ab); res(); };
    const ab = why => { done = true; clearTimeout(t); rej(why); };
    aborts.add(ab);
    t = setTimeout(go, 50);
    requestAnimationFrame(go);
  });
  const api = {
    get alive() { return !stopped; },
    /** the cycle's own clock: ms of cycle time since it started (on screen and not paused) */
    get time() { return began ? performance.now() - t0 - lost : 0; },
    async wait(ms) {
      await gate();
      if (!began) { await sleep(ms); await gate(); return; }       // before the cycle starts (the rest): a plain wait
      planned += ms;
      const due = t0 + planned + lost - performance.now();
      if (due < -LAG) { lost -= due; await frame(); }               // far behind (a stalled tab): go on from here
      else if (due > 1) await sleep(due);
      else await frame();
      await gate();
    },
    // frames count only while the loop is active, so a pause or a scroll away holds the tween where it is
    async tween(ms, fn, ease = t => 1 - Math.pow(1 - t, 3)) {
      await gate();
      if (began) planned += ms;
      await new Promise((res, rej) => {
        let done = 0, last = null, dead = false;
        const ab = why => { dead = true; rej(why); };
        aborts.add(ab);
        const step = now => {
          if (dead) return;
          if (!active()) { last = null; gate().then(() => requestAnimationFrame(step), e => { aborts.delete(ab); rej(e); }); return; }
          if (last != null) done += now - last;
          last = now;
          const k = Math.min(1, done / ms);
          fn(ease(k));
          if (k < 1) requestAnimationFrame(step); else { aborts.delete(ab); res(); }
        };
        requestAnimationFrame(step);
      });
      await gate();
    },
  };
  const io = new IntersectionObserver(es => {
    const was = visible;
    visible = es[es.length - 1].isIntersecting;
    // scrolled away mid-cycle: rest on the finished picture, never on a cleared or half-built frame; the cycle starts again from the top when it is back
    if (was && !visible && began && !paused) { abort(AGAIN); began = 0; show(); if (mode === 'run' && rest > 0) needRest = true; }
    flush();
  }, { rootMargin });
  io.observe(root);
  const onVis = () => flush();
  document.addEventListener('visibilitychange', onVis);
  // effects inside the fragment (the voice orb) stop and start with it
  const inner = on => root.querySelectorAll('[data-fx-mounted]').forEach(el => { try { on ? el.__fx?.resume() : el.__fx?.pause(); } catch {} });
  const ctl = {
    pause() { paused = true; if (began) { abort(AGAIN); began = 0; show(); if (mode === 'run' && rest > 0) needRest = true; } root.classList.add('is-ui-paused'); inner(false); root.dispatchEvent(new CustomEvent('ui-pause', { detail: { paused: true } })); },
    resume() { paused = false; root.classList.remove('is-ui-paused'); inner(true); root.dispatchEvent(new CustomEvent('ui-pause', { detail: { paused: false } })); flush(); },
    /** from the top: a loop starts its cycle again at once; a held fragment plays it once */
    restart() {
      if (stopped || paused) return;
      if (want > 0 && began && performance.now() - began < QUARTER) return;
      abort(AGAIN);
      needRest = false;
      began = 0;
      show();
      if (mode === 'held') want = Math.max(want, 1);
      kick();
    },
    /** from now on the finished picture; it still plays while the pointer is on its card */
    hold() {
      if (stopped) return;
      mode = 'held'; want = 0; hovering = false;
      if (unpause) { unpause(); unpause = null; }
      abort(AGAIN);
      show();
      setHeld(true);
    },
    destroy() {
      if (stopped) return;
      stopped = true;
      io.disconnect();
      document.removeEventListener('visibilitychange', onVis);
      abort(STOP);
      kick();
      unlisten();
      if (unpause) unpause();
      root.classList.remove('is-ui-paused');
      setHeld(true);
      show();
    },
  };
  const rec = {
    root,
    running: () => !stopped && mode === 'run',
    restart: () => ctl.restart(),
    hover(on) {
      if (stopped || mode !== 'held') return;
      hovering = on && !paused;
      if (hovering && want <= 0) { want = 1; kick(); }
      // leaving: the cycle under way plays to its end and rests on the finished picture
    },
  };
  const unlisten = hostListen(hostOf(root), rec);
  setHeld(mode === 'held');
  show();
  (async () => {
    for (;;) {
      if (stopped) return;
      if (want <= 0) { setHeld(true); await new Promise(r => wake.push(r)); continue; }
      cur = gen;
      setHeld(false);
      try {
        await gate();   // a cycle starts only while the fragment is on screen and not paused: its first step clears the picture
        if (needRest) { await api.wait(rest); needRest = false; }
        // the button appears when the picture starts to move; a loop with a cycle count stops by itself
        if (mode === 'run' && pause && !maxCycles && !unpause) unpause = addPause(root, ctl);
        began = performance.now();
        t0 = began; planned = 0; lost = 0;
        untidy(root);
        stalled = false;
        // a page that wants to follow the cycle (a caption, a second piece in step) listens for this
        root.dispatchEvent(new CustomEvent('ui-cycle', { bubbles: true, detail: { held: mode === 'held' } }));
        await cycle(api);
        // a dev warning (references/motion.md): one cycle is 2.5 to 4 s, the finished picture held at most
        // 15% of it; a cycle that waited off screen, paused or was restarted is not measured
        const len = performance.now() - began;
        root.dispatchEvent(new CustomEvent('ui-cycle-end', { bubbles: true, detail: { ms: Math.round(len), planned: Math.round(planned), stalled } }));
        // the plan (the sum of the cycle's waits and tweens) is what the author controls, so it is the
        // measure; the time the cycle took is checked too, for one that ran long on a busy page
        if (!warned && !stalled && (planned < 2400 || planned > 4000 || len > 4400)) {
          warned = true;
          console.warn(`ui.js: a cycle planned at ${(planned / 1000).toFixed(2)} s and run in ${(len / 1000).toFixed(2)} s (keep one cycle 2.5 to 4 s)`, root);
        }
        began = 0;
        want--;
        if (mode === 'run' && maxCycles && ++played >= maxCycles) { mode = 'held'; want = 0; }
        if (mode === 'held' && hovering) want = Math.max(want, 1);     // again while the pointer stays
        if (want <= 0) show();
      } catch (e) {
        began = 0;
        if (e === STOP) return;
        if (e !== AGAIN) { console.error(e); return; }
      }
    }
  })();
  return ctl;
}

/** restart every fragment in el (or el itself): a loop starts its cycle again from the top, a held one
    plays it once. For a tab or a panel the page shows again */
export function restart(el = document) {
  const roots = el.matches && el.matches('[data-ui]') ? [el] : [...el.querySelectorAll('[data-ui]')];
  for (const r of roots) { try { r.__ui?.restart?.(); } catch (e) { console.warn(e); } }
}

/* ---------------------------------------------------------------- grounds */
/*
  grounds(root, fx, opts): mounts every [data-fx] ground under root so a page of cards stays light on a
  laptop's integrated GPU: one or two WebGL contexts in all, never one per panel.
  - A still ground (every ground in a card holds still) is drawn once when it first comes near the
    screen, copied into a plain 2D canvas, and its WebGL context is freed at once. One at a time, so
    baking never holds more than one extra context. It is drawn again after a resize.
  - A ground a fragment drives (data-fx-live on it, or a fragment inside it whose data-ui-opts has a
    "drive": the slider) stays live while it is near the screen, draws a frame only when it changes,
    and is freed when it leaves the screen.
  - Under prefers-reduced-motion nothing drives a ground, so every ground is baked.
  A baked ground is drawn at min(devicePixelRatio, 2) device pixels per CSS pixel (its context lives
  for that one frame), so it stays sharp on a 1.25-2x laptop screen; a live ground draws at 1. It also
  caps fx.config at 3 live contexts; fx.config.maxDpr is left alone, so a moving hero the page mounts
  keeps its own sharpness. opts.skip(el) leaves a ground to the page (that moving hero). Call it again
  after a theme change: every ground is drawn again in the new colours. Resolves when the grounds near
  the screen are drawn.
*/
const grounded = new WeakMap();
const bakeQueue = [];
let bakeBusy = null;
const nextFrame = () => new Promise(r => requestAnimationFrame(() => r()));
const parseJSON = s => { try { return s ? JSON.parse(s) : null; } catch { return null; } };
// live: marked so, or holding a fragment that drives it and is not held still (a still slider never
// moves); never under reduced motion, where no fragment moves
const isLive = el => !reduced() && (el.hasAttribute('data-fx-live') ||
  [...el.querySelectorAll('[data-ui-opts]')].some(f => /"drive"/.test(f.dataset.uiOpts || '') && !f.closest('[data-ui-still]')));
// mount with a device-pixel cap of its own: fx.shader reads fx.config.maxDpr once, when it is made
async function mountAt(fx, el, spec, dpr) {
  await fx.load(el.dataset.fx);
  const cfg = fx.config, keep = cfg ? cfg.maxDpr : null;
  if (cfg) cfg.maxDpr = dpr;
  try { return await fx.mount(el, el.dataset.fx, spec); } finally { if (cfg) cfg.maxDpr = keep; }
}
const bakeDpr = () => Math.min(2, Math.max(1, (typeof devicePixelRatio === 'number' && devicePixelRatio) || 1));

function groundSpec(el, fx, extra) {
  let palette = fx.readPalette(el);
  const over = parseJSON(el.dataset.fxPalette);
  if (over) palette = fx.normalizePalette({ ...palette, ...over });
  // a seed is kept on the element, so a ground drawn again (resize, theme) is the same picture
  if (!el.dataset.fxSeed) el.dataset.fxSeed = String(Math.floor(Math.random() * 10000));
  return { palette, params: { ...(parseJSON(el.dataset.fxParams) || {}), ...(extra || {}) }, seed: el.dataset.fxSeed };
}
function unground(el) { if (el.__fx) { try { el.__fx.destroy(); } catch { /* gone */ } } }

async function bakeOne(el, fx, st) {
  st.queued = false;
  if (!el.isConnected || !st.near || !st.dirty) return;
  const gen = ++st.gen;
  unground(el);
  // a baked picture is kept for good, so it is drawn at full scale: fx's adaptive scale (which lowers the
  // render scale when frames run slow) is off while it draws; bakes run one at a time
  const cfg = fx.config, adaptive = cfg ? cfg.adaptive : undefined;
  if (cfg) cfg.adaptive = false;
  try { await bakeDraw(el, fx, st, gen); } finally { if (cfg) cfg.adaptive = adaptive; }
}
// a ground that could not be drawn (no WebGL, a shader error, too slow) still shows a picture: a still
// field in its own colours (fx.fallbackCSS), never an empty panel; it is tried again when it comes back
function stand(el, fx) {
  if (el.style.background || !fx.fallbackCSS) return;
  try { el.style.background = fx.fallbackCSS(groundSpec(el, fx).palette); el.dataset.uiStandIn = ''; } catch { /* the panel's own fill stays */ }
}
async function bakeDraw(el, fx, st, gen) {
  let api;
  // a second try (after a frame that never came) draws at 1 device pixel: a slower machine still gets a picture
  try { api = await mountAt(fx, el, groundSpec(el, fx), st.tries ? 1 : bakeDpr()); } catch (e) { console.warn('ui: ground', e); stand(el, fx); st.dirty = false; return; }
  // fx makes the context once it sees the element; wait for its first drawn frame (slow on a CPU renderer)
  let src = null;
  for (const t0 = performance.now(); performance.now() - t0 < 10000;) {
    if (el.__fx !== api) return;                                    // replaced meanwhile
    if (gen !== st.gen || !st.near) { api.destroy(); if (st.near) queueBake(el, fx, st); return; }
    if (el.dataset.fxFallback) { st.dirty = false; return; }      // no WebGL: the CSS fallback stays
    const c = el.querySelector(':scope > canvas.fx-canvas:not(.is-baked)');
    if (c && c.style.opacity === '1') { src = c; break; }
    await nextFrame();
  }
  if (!src) {
    // too slow: a still field in its colours for now, and one more try a moment later at 1 device pixel
    api.destroy(); stand(el, fx);
    if (!st.tries) { st.tries = 1; setTimeout(() => { if (st.near && st.dirty) queueBake(el, fx, st); }, 1200); }
    return;
  }
  api.set({});                    // one more frame now, so the drawing buffer is whole when it is copied
  const out = document.createElement('canvas');
  out.width = src.width; out.height = src.height;
  out.getContext('2d').drawImage(src, 0, 0);
  out.className = 'fx-canvas is-baked';
  out.setAttribute('aria-hidden', 'true');
  Object.assign(out.style, { position: 'absolute', inset: '0', width: '100%', height: '100%', display: 'block', zIndex: '-1', pointerEvents: 'none', borderRadius: 'inherit' });
  api.destroy();                                                     // frees the WebGL context
  el.insertBefore(out, el.firstChild);
  if (el.dataset.uiStandIn != null) { el.style.background = ''; delete el.dataset.uiStandIn; }   // the real picture replaces the stand-in
  const id = el.dataset.fx;
  const stub = {
    id, el, baked: true, params: api.params, palette: api.palette, seed: api.seed,
    pause() {}, resume() {}, set() { return stub; }, setPalette() { return stub; },
    destroy() { out.remove(); if (el.__fx === stub) delete el.__fx; el.removeAttribute('data-fx-mounted'); },
    get info() { return { webgl: 0, baked: true, size: [out.width, out.height] }; },
  };
  el.__fx = stub;                 // fx.mountAll skips it; fx.mount on it replaces the picture
  el.setAttribute('data-fx-mounted', id);
  st.size = [el.clientWidth, el.clientHeight];
  st.dirty = false;
}
// the next ground to draw: one on the screen first (the top one), then the nearest to it, so a scroll
// past a group never makes the panel in view wait for the ones it passed
function nextBake() {
  const vh = innerHeight;
  let best = 0, bd = Infinity;
  bakeQueue.forEach(([el], i) => {
    const r = el.getBoundingClientRect();
    const d = r.bottom > 0 && r.top < vh ? Math.max(0, r.top) - 1e6 : r.top >= vh ? r.top - vh : -r.bottom;
    if (d < bd) { bd = d; best = i; }
  });
  return bakeQueue.splice(best, 1)[0];
}
function pump() {
  if (!bakeBusy) bakeBusy = (async () => {
    await null;                   // every ground the same scroll brought near is queued before one is picked
    while (bakeQueue.length) {
      const [el, fx, st] = nextBake();
      try { await bakeOne(el, fx, st); } catch (e) { console.warn('ui: ground', e); }
      await nextFrame();
    }
    bakeBusy = null;
  })();
  return bakeBusy;
}
function queueBake(el, fx, st) {
  if (st.queued || !st.dirty) return;
  st.queued = true;
  bakeQueue.push([el, fx, st]);
  pump();
}
async function liveOn(el, fx, st) {
  if (el.__fx && !el.__fx.baked) return;
  const gen = ++st.gen;
  unground(el);
  const api = await mountAt(fx, el, groundSpec(el, fx, st.extra), 1).catch(e => { console.warn('ui: ground', e); return null; });
  if (!api) return;
  if (gen !== st.gen || !st.near) { api.destroy(); return; }
  api.pause();                    // no frame loop: it draws when shown, resized or set() by its fragment
  const set = api.set;
  api.set = next => { st.extra = { ...st.extra, ...next }; return set(next); };
}

const groundWaiters = [];
const groundIO = typeof IntersectionObserver === 'function' ? new IntersectionObserver(es => {
  for (const e of es) {
    const st = grounded.get(e.target);
    if (!st) continue;
    st.near = e.isIntersecting;
    if (st.live) {
      if (st.near) liveOn(e.target, st.fx, st);
      else { st.gen++; unground(e.target); }                      // its context is freed off screen
    } else if (st.near) queueBake(e.target, st.fx, st);
  }
  for (const r of groundWaiters.splice(0)) r();
}, { rootMargin: '12% 0px' }) : null;   // the same margin fx.js makes its contexts at
const groundRO = typeof ResizeObserver === 'function' ? new ResizeObserver(es => {
  for (const e of es) {
    const st = grounded.get(e.target);
    if (!st || st.live || !st.size) continue;
    const w = e.target.clientWidth, h = e.target.clientHeight;
    if (Math.abs(w - st.size[0]) < 2 && Math.abs(h - st.size[1]) < 2) continue;
    clearTimeout(st.rt);
    st.rt = setTimeout(() => { st.dirty = true; if (st.near) queueBake(e.target, st.fx, st); }, 250);
  }
}) : null;

export async function grounds(root, fx, { skip } = {}) {
  root = root || document;
  if (fx.config) fx.config.maxLive = Math.min(fx.config.maxLive || 3, 3);
  const els = [...root.querySelectorAll('[data-fx]')].filter(el => !(skip && skip(el)));
  if (root.matches && root.matches('[data-fx]') && !(skip && skip(root))) els.unshift(root);
  if (!groundIO) return fx.mountAll(root);
  const seen = new Promise(r => groundWaiters.push(r));
  for (const el of els) {
    let st = grounded.get(el);
    if (st) {
      // again (a theme change): draw it once more in the new colours
      st.dirty = true; st.gen++;
      if (st.live) { unground(el); if (st.near) liveOn(el, fx, st); }
      else if (st.near) queueBake(el, fx, st);
      continue;
    }
    st = { fx, live: isLive(el), near: false, dirty: true, queued: false, gen: 0, size: null, extra: null, tries: 0 };
    grounded.set(el, st);
    groundIO.observe(el);
    if (groundRO) groundRO.observe(el);
  }
  await Promise.race([seen, new Promise(r => setTimeout(r, 300))]);
  if (bakeBusy) await bakeBusy;
}

/** the pointer arrow used by fragments that click */
export const POINTER_SVG = '<svg viewBox="0 0 20 20" aria-hidden="true"><path d="M4.5 2.8v13.4c0 .5.6.7.9.4l3.2-3.1 2.2 4.9c.1.3.5.4.8.3l1.6-.7c.3-.1.4-.5.3-.8l-2.2-4.8 4.4-.3c.5 0 .7-.6.3-.9L5.4 2.4c-.4-.3-.9 0-.9.4Z" fill="#111" stroke="#fff" stroke-width="1.3" stroke-linejoin="round"/></svg>';

/** a controller that also restarts, pauses and holds: for a fragment that returns its own destroy() */
export const withLoop = (ctl, destroy) => ({ pause: ctl.pause, resume: ctl.resume, restart: ctl.restart, hold: ctl.hold, destroy });

const ui = { load, loadAll, start, startAll, plan, loop, restart, hold, pausable, grounds, isStill, reduced, POINTER_SVG };
export default ui;
```

## assets/ui/window/fragment.css

```css
/* window: a helpdesk queue (sidebar, list); the assistant works down the open rows */
.ui-win { width: 600px; overflow: hidden; }
.ui-win__bar { display: flex; align-items: center; gap: 14px; height: 44px; padding: 0 12px 0 14px; border-bottom: 1px solid var(--u-line); background: var(--u-fill); min-width: 0; }
/* the breadcrumb gives way first (an ellipsis), so the search box at the right is never pushed out or cut */
.ui-win__crumb { display: inline-flex; align-items: center; gap: 8px; min-width: 0; flex: 0 1 auto; overflow: hidden; font-size: 13px; line-height: 16px; color: var(--u-muted); white-space: nowrap; }
.ui-win__crumb b { min-width: 0; overflow: hidden; text-overflow: ellipsis; }
.ui-win__crumb .ui-brand, .ui-win__crumb i { flex: none; }
.ui-win__crumb .ui-brand { width: 14px; height: 14px; }
.ui-win__crumb i { font-style: normal; color: var(--u-muted); }
.ui-win__crumb b { font-weight: 500; color: var(--u-ink); }
.ui-win__search { display: inline-flex; flex: none; align-items: center; gap: 8px; height: 28px; width: 168px; padding: 0 4px 0 8px; border-radius: 7px; background: var(--u-bg); box-shadow: inset 0 0 0 1px var(--u-line-2); font-size: 13px; color: var(--u-soft); }
.ui-win__search .ui-icon { width: 14px; height: 14px; color: var(--u-muted); }
.ui-win__search .ui-kbd { margin-left: auto; min-width: 20px; height: 20px; font-size: 13px; }
.ui-win__body { display: grid; grid-template-columns: 152px 1fr; min-height: 312px; }
.ui-win__side { display: grid; align-content: start; gap: 1px; padding: 8px; border-right: 1px solid var(--u-line); }
.ui-win__nav {
  display: flex; align-items: center; gap: 8px; height: 30px; padding: 0 8px; border-radius: 6px;
  font-size: 13px; line-height: 16px; color: var(--u-muted);
}
.ui-win__nav .ui-icon { width: 14px; height: 14px; }
.ui-win__nav em { margin-left: auto; font-style: normal; font-size: 13px; color: var(--u-soft); }
.ui-win__nav.is-on { background: var(--u-fill-2); color: var(--u-ink); font-weight: 500; }
.ui-win__nav.is-on em { color: var(--u-muted); }
.ui-win__label { margin: 12px 8px 4px; font-size: 13px; line-height: 16px; color: var(--u-soft); }
.ui-win__hash { width: 14px; text-align: center; color: var(--u-soft); }
.ui-win__main { min-width: 0; }
.ui-win__cols, .ui-win__row { display: grid; grid-template-columns: 24px 92px minmax(0, 1fr) 96px 28px; align-items: center; gap: 0 12px; padding: 0 16px; }
.ui-win__cols { height: 32px; border-bottom: 1px solid var(--u-line); font-size: 13px; line-height: 16px; color: var(--u-soft); }
.ui-win__cols span:first-child { grid-column: 1 / 3; }
.ui-win__row { height: 44px; font-size: 13px; line-height: 16px; transition: background-color 260ms var(--u-ease); }
.ui-win__row + .ui-win__row { box-shadow: inset 0 1px 0 var(--u-line); }
.ui-win__row.is-sel { background: color-mix(in oklab, var(--u-ink) 4%, var(--u-bg)); }
.ui-win__row .ui-avatar { width: 24px; height: 24px; }
.ui-win__who { font-weight: 500; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.ui-win__sub { color: var(--u-muted); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.ui-win__age { color: var(--u-soft); text-align: right; font-size: 13px; }
.ui-win__st {
  display: inline-flex; align-items: center; gap: 6px; justify-self: start; height: 22px; padding: 0 8px; border-radius: 6px;
  font-size: 13px; font-weight: 500; white-space: nowrap; color: var(--u-muted); box-shadow: inset 0 0 0 1px var(--u-line-2);
  transition: color 240ms var(--u-ease), background-color 240ms var(--u-ease), box-shadow 240ms var(--u-ease);
}
.ui-win__st::before { content: ""; width: 6px; height: 6px; border-radius: 50%; background: currentColor; opacity: 0.6; flex: none; }
/* the row being worked on is the window's one accent touch */
.ui-win__st.is-work { color: var(--u-accent-text); background: var(--u-tint); box-shadow: inset 0 0 0 1px color-mix(in oklab, var(--u-accent) 22%, transparent); }
.ui-win__st.is-work::before { width: 8px; height: 8px; background: none; border: 1.5px solid color-mix(in oklab, currentColor 30%, transparent); border-top-color: currentColor; opacity: 1; animation: ui-spin 800ms linear infinite; }
.ui-win__st.is-done { color: var(--u-soft); box-shadow: inset 0 0 0 1px var(--u-line); }
.ui-win__st.is-you { color: var(--u-ink); box-shadow: inset 0 0 0 1px var(--u-line-2); background: var(--u-fill-2); }
.ui-win__st.is-you::before { background: var(--u-ink); opacity: 1; }
/* compact (opts.compact): a narrower window whose rows stack name over subject, for a card panel
   where the whole width must show; the status stays in view */
.ui-win--compact { width: 456px; }
.ui-win--compact .ui-win__body { grid-template-columns: 128px 1fr; }
.ui-win--compact .ui-win__search { width: 112px; }
.ui-win--compact .ui-win__cols { display: none; }
.ui-win--compact .ui-win__row { grid-template-columns: 24px minmax(0, 1fr) auto; grid-template-rows: 18px 16px; align-content: center; height: 56px; row-gap: 1px; padding: 0 14px; }
.ui-win--compact .ui-win__row .ui-avatar { grid-row: 1 / 3; }
.ui-win--compact .ui-win__who { grid-column: 2; grid-row: 1; line-height: 18px; }
.ui-win--compact .ui-win__sub { grid-column: 2; grid-row: 2; font-size: 13px; }
.ui-win--compact .ui-win__st { grid-column: 3; grid-row: 1 / 3; align-self: center; }
.ui-win--compact .ui-win__age { display: none; }
/* narrow: the window's own width decides its layout (a container query), so a .fit wrapper, a small
   card at 1024 or a phone panel never slices its columns:
   - under 540px the rows stack name over subject, the column heads and the age go (the compact rows);
   - under 420px the sidebar goes too and the search shrinks to its icon. */
.ui-win { container: ui-win / inline-size; }
@container ui-win (max-width: 539px) {
  .ui-win .ui-win__cols { display: none; }
  .ui-win .ui-win__row { grid-template-columns: 24px minmax(0, 1fr) auto; grid-template-rows: 18px 16px; align-content: center; height: 56px; row-gap: 1px; padding: 0 14px; }
  .ui-win .ui-win__row .ui-avatar { grid-row: 1 / 3; }
  .ui-win .ui-win__who { grid-column: 2; grid-row: 1; line-height: 18px; }
  .ui-win .ui-win__sub { grid-column: 2; grid-row: 2; font-size: 13px; }
  .ui-win .ui-win__st { grid-column: 3; grid-row: 1 / 3; align-self: center; }
  .ui-win .ui-win__age { display: none; }
  .ui-win .ui-win__body { grid-template-columns: 128px minmax(0, 1fr); }
  .ui-win .ui-win__search { width: 112px; }
}
@container ui-win (max-width: 419px) {
  .ui-win .ui-win__body { grid-template-columns: minmax(0, 1fr); }
  .ui-win .ui-win__side { display: none; }
  .ui-win .ui-win__search { width: 28px; padding: 0; justify-content: center; gap: 0; }
  .ui-win .ui-win__search :is(.ui-kbd, .ui-win__ph) { display: none; }
  .ui-win .ui-win__row { gap: 0 10px; padding: 0 12px; }
  .ui-win .ui-win__bar { gap: 10px; padding: 0 8px 0 12px; }
}
/* very narrow: the window dots go too, the crumb keeps its last word */
@container ui-win (max-width: 299px) {
  .ui-win .ui-dots { display: none; }
}
/* phone: the window takes the panel's width (cards.css); the queries above lay it out */
@media (max-width: 639px) {
  .ui-win, .ui-win--compact { width: 100%; }
}
```

## assets/ui/window/fragment.js

```js
// window: one row per cycle (about 3 s): the assistant finishes the row it is on (it turns to the done
// word) and takes the next open row (it turns to the working word); after the last open row the queue
// starts over. A row is open when its status reads the open word as written. The words are the page's
// own: data-open, data-work and data-done on the root (or opts.open, opts.work, opts.done); "Open",
// "Working" and "Replied" when it gives none.
import { loop } from '../ui.js';

export default function init(root, opts = {}) {
  const rows = [...root.querySelectorAll('[data-row]')];
  const st = rows.map(r => r.querySelector('[data-st]'));
  if (opts.compact) root.classList.add('ui-win--compact');
  if (!rows.length || st.some(s => !s)) return null;
  const word = (k, d) => opts[k] || root.dataset[k] || d;
  const openWord = word('open', 'Open'), work = word('work', 'Working'), done = word('done', 'Replied');
  const first = st.map(s => ({ text: s.textContent, cls: s.className }));
  const open = rows.map((r, i) => (st[i].textContent.trim() === openWord ? i : -1)).filter(i => i >= 0);
  if (!open.length) return null;
  const set = (i, text, cls) => { st[i].textContent = text; st[i].className = `ui-win__st ${cls || ''}`.trim(); };
  const reset = () => { rows.forEach((r, i) => { r.classList.remove('is-sel'); st[i].textContent = first[i].text; st[i].className = first[i].cls; }); };
  // step j: the first j open rows are done, open row j is being worked on (none when j is past the end)
  const at = j => {
    reset();
    open.forEach((i, n) => { if (n < j) set(i, done, 'is-done'); });
    if (j < open.length) { set(open[j], work, 'is-work'); rows[open[j]].classList.add('is-sel'); }
  };
  // the finished picture: the first open row done, the second being worked on
  const j0 = Math.min(1, open.length);
  let j = j0;
  const still = () => { j = j0; at(j); };
  // one cycle (about 2.6 s): two steps of the queue (a row done and the next taken), the second held 0.38 s
  return loop(root, async api => {
    await api.wait(200);
    for (let n = 0; n < 2; n++) {
      j = (j + 1) % (open.length + 1);
      if (j === 0) {
        // the queue starts over: every row back as written, then the first open row is taken
        reset();
        await api.wait(350);
        at(0);
      } else {
        set(open[j - 1], done, 'is-done');
        rows[open[j - 1]].classList.remove('is-sel');
        await api.wait(350);
        if (j < open.length) { set(open[j], work, 'is-work'); rows[open[j]].classList.add('is-sel'); }
      }
      await api.wait(n < 1 ? 1300 : 380);
    }
  }, { still });
}
```

## index.html

```html
<!doctype html>
<html lang="pt-BR" data-type="software" data-radius="soft" data-ground="pitch" data-buttons="tonal" data-motion>
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Rise Eleven: peneiras de futebol de base no app</title>
<meta name="description" content="Rise Eleven conecta clubes e jogadores de futebol de base no Brasil: clubes publicam peneiras, jogadores se inscrevem e mostram seus vídeos.">
<meta name="site-kind" content="software">
<meta name="page-type" content="software">
<meta name="theme-color" content="#0b0b0c">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Oswald:wght@500;600&family=Barlow:wght@400;500;600&family=JetBrains+Mono:wght@500&display=swap" rel="stylesheet">
<link rel="stylesheet" href="assets/system.css">
<link rel="stylesheet" href="assets/ui/cards.css">
<link rel="stylesheet" href="assets/motion/motion.css">
<link rel="stylesheet" href="assets/backgrounds/backgrounds.css">
<link rel="stylesheet" href="assets/layout/layout.css">
<link rel="stylesheet" href="assets/icons/icon.css">
<style>
:root {
  --p-bg: #0b0b0c; --p-surface: #151517; --p-ink: #f1f1f2; --p-muted: #b6b6bb;
  --p-accent: #F26B2C; --p-accent-ink: #1c0a02; --p-action: #F26B2C; --p-action-ink: #1c0a02;
  --p-inv-bg: #f1f1f2; --p-inv-surface: #ffffff; --p-inv-ink: #0b0b0c; --p-inv-muted: #55555b;
  --p-inv-accent: #C2410C; --p-inv-accent-ink: #ffffff; --p-inv-action: #C2410C; --p-inv-action-ink: #ffffff;
  --font-display: "Oswald", "Arial Narrow", sans-serif;
  --font-text: "Barlow", system-ui, sans-serif;
  --font-mono: "JetBrains Mono", ui-monospace, Consolas, monospace;
  --display-weight: 500;
  --display-tracking: -0.005em;
  --display-tracking-xl: -0.01em;
  --ly-scale: 1.04;
}
/* the lit ground behind UI panels and the closing band: the pitch */
html[data-ground="pitch"] { --ground-bg: #2a2a2e; --ground-surface: #333338; --ground-ink: #f4f4f5; --ground-muted: #d4d4d8; --ground-accent: #FFA26B; }
/* the closing band: a deeper pitch, so its white text reads at 7:1 */
html[data-ground="pitch"] .footer--statement { --ground-bg: #1a1a1d; --ground-surface: #222226; --ground-muted: #dcdce0; }
</style>
<link rel="stylesheet" href="assets/ui/base.css">
<link rel="stylesheet" href="assets/ui/window/fragment.css">
<link rel="stylesheet" href="assets/ui/chat/fragment.css">
<link rel="stylesheet" href="assets/ui/search/fragment.css">
<link rel="stylesheet" href="assets/ui/notify/fragment.css">
<link rel="stylesheet" href="assets/ui/checklist/fragment.css">
<link rel="stylesheet" href="assets/ui/toggles/fragment.css">
<script src="assets/backgrounds/backgrounds.js"></script>
<!-- page CSS: inside visuals only (the hero collage, the app frame), one --p-h for the card group -->
<style>
  /* the RISE11 wordmark (from the brief's brand page) */
  .wordmark { font: 600 24px/1 var(--font-display); letter-spacing: .01em; color: var(--ink); }
  .wordmark b { font-weight: inherit; color: var(--accent); }
  :root { --frag: #17171a; }
  .close__word b { font-weight: inherit; color: var(--ground-accent); }
  h1, h2, .close__word { font-weight: 500; }

  /* hero: the two app screens from the brief (modo jogador, modo olheiro), as dark phones on the collage */
  .ly-collage { container-type: inline-size; }
  /* phones keep a 19.5:9 screen at any size: as wide as fits both the width and the height of the collage */
  .ly-collage__set { container-type: size; display: flex; align-items: center; justify-content: center; gap: 4cqw; }
  .ly-collage .ly-piece.ph { position: relative; flex: none; width: min(45cqw, calc(94cqh * 9 / 19.5)); height: auto; aspect-ratio: 9 / 19.5; }
  /* the phones show the real app: colors from src/constants/theme.ts, system font, logo weight 900 */
  .ly-collage .ly-piece.ph, .ph-stage .ph { --app-bg: #0B0F14; --app-el: #141A21; --app-sel: #1E262F; --app-text: #F2F4F7; --app-muted: #8B95A1; --app-border: #232B34; --app-primary: #F26B2C; --app-on-primary: #FFFFFF;
    --accent: var(--app-primary); --accent-ink: var(--app-on-primary); background: #060607; color: var(--app-text); border-radius: 6cqw; padding: 1.6cqw; border: 1px solid #27272b;
    box-shadow: 0 1px 2px rgb(0 0 0 / .4), 0 30px 60px -24px rgb(0 0 0 / .8); font: 400 max(13px, 2.1cqw)/1.35 -apple-system, system-ui, "Segoe UI", Roboto, sans-serif; }
  .ph__screen { height: 100%; border-radius: 4.6cqw; background: var(--app-bg); overflow: hidden; display: grid; grid-template-rows: auto 1fr auto; }
  .ph__top { display: flex; align-items: center; justify-content: space-between; padding: 4cqw 4cqw 2.4cqw; }
  .ph__top .wordmark { font-family: inherit; font-weight: 900; font-size: max(14px, 3.6cqw); line-height: 1; letter-spacing: -0.02em; color: var(--app-text); }
  .ph__top small { display: block; font: 600 max(13px, 1.7cqw)/1.2 -apple-system, system-ui, "Segoe UI", Roboto, sans-serif; letter-spacing: .04em; color: var(--accent); }
  .ph__top strong { font: 800 max(15px, 4cqw)/1.1 inherit; font-family: inherit; }
  .ph__icons { display: flex; gap: 1.6cqw; }
  .ph__icons i { width: 5cqw; height: 5cqw; border-radius: 50%; border: 1px solid var(--app-border); }
  .ph__body { padding: 0 3.4cqw; display: grid; gap: 2.4cqw; align-content: start; overflow: hidden; }
  .ph-card { background: var(--app-el); border: 1px solid var(--app-border); border-radius: 2.6cqw; padding: 2.6cqw; display: grid; gap: 1.6cqw; }
  .ph-card b, .ph-row b { font-weight: 600; }
  .ph-card small, .ph-row small { color: var(--app-muted); font-size: max(13px, 1.7cqw); }
  .ph-card em { font-style: normal; color: var(--accent); font-weight: 600; }
  .ph-who { display: flex; align-items: center; gap: 2cqw; }
  .ph-av { width: 6cqw; height: 6cqw; border-radius: 50%; background: var(--app-sel); flex: none; display: grid; place-items: center; font: 600 max(12px, 1.7cqw)/1 -apple-system, system-ui, "Segoe UI", Roboto, sans-serif; color: var(--app-text); }
  .ph-av--club { background: var(--accent); color: var(--accent-ink); }
  .ph-video { position: relative; aspect-ratio: 16 / 10; border-radius: 1.8cqw; background: var(--app-sel); overflow: hidden; }
  .ph-video::before { content: ""; position: absolute; inset: 8% 6%; border: 1.5px solid rgb(255 255 255 / .4); border-radius: 1cqw; }
  .ph-video::after { content: ""; position: absolute; left: 50%; top: 8%; bottom: 8%; border-left: 1.5px solid rgb(255 255 255 / .4); }
  .ph-video span { position: absolute; z-index: 1; left: 50%; top: 50%; width: 9cqw; height: 9cqw; transform: translate(-50%, -50%); border-radius: 50%; background: #fff; display: grid; place-items: center; }
  .ph-video span svg { width: 38%; height: 38%; margin-left: 8%; }
  .ph-video small { position: absolute; z-index: 1; left: 2.4cqw; bottom: 2cqw; color: #fff; font-weight: 600; }
  .ph-search { border: 1px solid var(--app-border); border-radius: 99px; padding: 1.6cqw 2.6cqw; color: var(--app-muted); }
  .ph-chips { display: flex; flex-wrap: wrap; gap: 1.2cqw; }
  .ph-chips span { border: 1px solid var(--app-border); border-radius: 99px; padding: .6cqw 2cqw; }
  .ph-chips span.on { background: var(--accent); border-color: var(--accent); color: var(--accent-ink); font-weight: 600; }
  .ph-row { display: grid; grid-template-columns: auto 1fr auto; align-items: center; gap: 2cqw; background: var(--app-el); border: 1px solid var(--app-border); border-radius: 2.6cqw; padding: 2.2cqw 2.6cqw; }
  .ph-row .n { text-align: right; color: var(--accent); font-weight: 600; line-height: 1.1; }
  .ph-row .n small { display: block; font-weight: 400; }
  .ph__tabs { display: grid; grid-template-columns: repeat(5, 1fr); align-items: end; column-gap: .8cqw; padding: 2.4cqw 1.2cqw 3cqw; background: var(--app-el); border-top: 1px solid var(--app-border); color: var(--app-muted); font-size: max(10px, 1.35cqw); text-align: center; }
  .ph__tabs--4 { grid-template-columns: repeat(4, 1fr); }
  .ph__tabs span.on { color: var(--accent); font-weight: 600; }
  .ph__tabs .rec { justify-self: center; width: 9cqw; height: 9cqw; margin-top: -5cqw; border-radius: 50%; background: var(--accent); display: grid; place-items: center; box-shadow: 0 0 0 1.2cqw var(--app-bg); }
  .ph__tabs .rec svg { width: 45%; height: 45%; }
  /* phone: the player screen alone, wide enough for its text */
  @media (max-width: 1023px) { .ph-card + .ph-card, .ph-row:last-child { display: none; } }
  /* Oswald's tall line box: drop the right column to the h2's cap line */
  #peneiras .grid > p.start-7 { margin-top: 10px; }
  #perguntas .faq { margin-top: 12px; }
  @media (max-width: 639px) { .ly-piece.ph:nth-child(2) { display: none; } .ly-collage .ly-piece.ph:first-child { width: min(80cqw, calc(94cqh * 9 / 19.5)); }
    /* tall enough for one phone at 80% of the width */
    .hero--triptych .ly-collage { min-height: calc((100vw - 2 * var(--gut, 24px)) * .8 * 19.5 / 9 / .94); } }

  /* app-frame (sections-library.md, app-frame) */
  .app.ui { display: grid; grid-template-columns: minmax(0, 5fr) minmax(0, 7fr); margin-top: var(--s-7); background: var(--u-bg); border: 1px solid var(--u-border); border-radius: var(--u-r); box-shadow: var(--u-shadow); }
  .app > .app__pane { position: relative; min-width: 0; min-height: 0; overflow: hidden; }
  .app > .app__pane + .app__pane { border-left: 1px solid var(--u-line); }
  .app > .app__pane > .ui-panel { width: 100%; border: 0; border-radius: 0; box-shadow: none; }
  .app > .app__pane::after { content: ""; position: absolute; inset: auto 0 0 0; height: 64px; background: linear-gradient(transparent, var(--u-bg)); pointer-events: none; }
  @media (min-width: 1024px) and (max-width: 1279px) { .app.ui { grid-template-columns: minmax(0, 4fr) minmax(0, 8fr); } }
  @media (max-width: 1023px) {
    .app.ui { grid-template-columns: minmax(0, 1fr); grid-template-rows: auto minmax(0, 1fr); }
    .app > .app__pane--list { max-height: 252px; }
    .app > .app__pane--list::after { display: none; }
    .app .ui-win__side { display: none; }
    .app .ui-win__body { grid-template-columns: minmax(0, 1fr); }
    .app > .app__pane + .app__pane { border-left: 0; border-top: 1px solid var(--u-line); }
  }
  @media (max-width: 639px) {
    .app.ui { grid-template-rows: minmax(0, 1fr); }
    .app > .app__pane--list { display: none; }
    .app > .app__pane + .app__pane { border-top: 0; }
  }

  #recursos .panel { --p-h: 336px; }
  .hero--triptych .hero__cta .btn { height: var(--btn-h); font-size: var(--t-body); }
  .closing .close--word .close__word { font-size: min(calc((100vw - 2 * var(--gut)) / 5.4), 230px); white-space: nowrap; line-height: 1.12; padding-top: .06em; }
  @media (max-width: 639px) { #recursos .panel { --p-h: 168px; } }

  /* como-funciona (moment pinned-chapters): each step's visual is one app screen in a 19.5:9 phone.
     The stage is a container 1/.45 of the phone, so the phone's cqw sizes match the hero's phones. */
  .mo-pc__visual--lit { container-type: size; }
  .ph-stage { container-type: inline-size; flex: none; width: calc(min(70cqw, 90cqh * 9 / 19.5) / .45); display: flex; justify-content: center; }
  .ph-stage .ph { width: 45cqw; aspect-ratio: 9 / 19.5; overflow: hidden; }
  /* a frame tall enough for the phone to read at the hero phones' size */
  /* (main first: moment.css loads after this sheet with the same selectors) */
  main [data-moment="pinned-chapters"] .mo-pc__visual.mo-pc__visual--lit { aspect-ratio: 4 / 5; }
  main [data-moment="pinned-chapters"].is-live .mo-pc__visuals { aspect-ratio: 4 / 5; }
  main [data-moment="pinned-chapters"].is-live .mo-pc__visual.mo-pc__visual--lit { aspect-ratio: auto; }
  @media (max-width: 639px) {
    main [data-moment="pinned-chapters"] .mo-pc__visual.mo-pc__visual--lit { aspect-ratio: 1 / 2; }
    .mo-pc__visual--lit .ph-stage { width: calc(min(84cqw, 92cqh * 9 / 19.5) / .45); }
  }
  .ph-hero { grid-template-columns: auto 1fr; align-items: center; }
  .ph-av--lg { width: 11cqw; height: 11cqw; font-size: max(13px, 2.6cqw); }
  .ph-stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1.6cqw; }
  .ph-stats span { display: grid; gap: .4cqw; background: var(--app-el); border: 1px solid var(--app-border); border-radius: 2.6cqw; padding: 2cqw; }
  .ph-stats small, .ph-field > small { color: var(--app-muted); font-size: max(12px, 1.6cqw); }
  .ph-thumbs { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1.2cqw; }
  .ph-thumbs i { aspect-ratio: 9 / 12; border-radius: 1.6cqw; background: var(--app-sel); }
  .ph-btn { display: block; text-align: center; border-radius: 99px; padding: 2cqw; background: var(--app-primary); color: var(--app-on-primary); font-weight: 700; }
  .ph-btn--outline { background: none; color: var(--app-text); border: 1px solid var(--app-border); }
  .ph-field { display: grid; gap: 1cqw; }
  .ph-input { display: block; border: 1px solid var(--app-border); background: var(--app-el); border-radius: 2cqw; padding: 2cqw 2.4cqw; }
  .ph-badge { margin-left: auto; align-self: center; border-radius: 99px; padding: .6cqw 2cqw; background: var(--app-sel); color: var(--app-primary); font-weight: 700; font-size: max(12px, 1.6cqw); }
  .ph-msg { max-width: 82%; justify-self: start; background: var(--app-el); border: 1px solid var(--app-border); border-radius: 3cqw 3cqw 3cqw 1cqw; padding: 2cqw 2.6cqw; }
  .ph-msg--me { justify-self: end; background: var(--app-primary); border-color: var(--app-primary); color: var(--app-on-primary); border-radius: 3cqw 3cqw 1cqw 3cqw; }
</style>
<script>document.documentElement.classList.add('mo-js');setTimeout(()=>document.documentElement.classList.add('mo-safe'),2500)</script>
<link rel="modulepreload" href="assets/moments/moments.js"><link rel="modulepreload" href="assets/moments/lib.js"><link rel="modulepreload" href="assets/moments/pinned-chapters/moment.js">
</head>
<body>
<!-- type: software (an app for clubs and youth football players)
     deal: hero triptych, nav nav-centre (dealt nav-cards: no photographs for its panel, so re-dealt to nav-centre),
     close close-word, footer footer-statement, rhythm space, buttons tonal, mark arc, ground sky, palette white-signal-red, type funnel
     effect: none (left out on purpose; the hero collage is the one timed loop); moments: none; scene: none
     plan: hero (1) · peneiras app-frame (1) · perguntas faq (1) · recursos features-2x2 (2, .g2) · footer.closing (1) -->

<header class="nav nav--centre">
  <div class="wrap nav__bar">
    <a class="nav__logo" href="#top"><span class="wordmark" aria-label="Rise Eleven">RISE<b>11</b></span></a>
    <nav class="nav__links" aria-label="Principal">
      <div class="nav__item"><a class="nav__link" href="#peneiras">Peneiras</a></div>
      <div class="nav__item"><a class="nav__link" href="#recursos">Recursos</a></div>
      <div class="nav__item"><a class="nav__link" href="#seguranca">Segurança</a></div>
      <div class="nav__item"><a class="nav__link" href="#perguntas">Perguntas</a></div>
    </nav>
    <div class="nav__actions">
      <a class="btn btn--primary" href="https://rise-eleven.expo.app">Abrir o app</a>
    </div>
    <button class="nav__toggle" type="button" aria-expanded="false" aria-label="Menu">
      <svg width="18" height="18" viewBox="0 0 18 18" aria-hidden="true"><path d="M2 5h14M2 13h14" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/></svg>
    </button>
  </div>
</header>

<main id="top">
  <section class="hero hero--triptych" aria-labelledby="hero-h">
    <div class="wrap"><div class="hero-text grid hero__grid">
      <div class="span-3">
        <h1 id="hero-h">Peneiras de futebol de base no app</h1>
        <p class="hero__lead">Jogadores mostram vídeos jogando, e menores de 18 anos entram com autorização do responsável. Clubes e olheiros encontram talentos e publicam peneiras verificadas. Grátis no lançamento.</p>
      </div>
      <div class="ly-collage span-6 start-4" role="img" aria-label="Duas telas do Rise Eleven: o modo jogador, com vídeo de jogada e uma peneira aberta verificada pelo clube, e o modo olheiro, com a busca de atletas por posição, categoria e estado">
        <div class="ly-collage__set is-on">
          <div class="ly-piece ph">
            <div class="ph__screen">
              <div class="ph__top"><span class="wordmark">RISE<b>11</b></span><span class="ph__icons"><i></i><i></i></span></div>
              <div class="ph__body">
                <div class="ph-card">
                  <div class="ph-who"><span class="ph-av">MS</span><span><b>Miguel Santos</b><br><small>Centroavante · Campinas, SP</small></span></div>
                  <div class="ph-video"><span><svg viewBox="0 0 16 16"><path d="M4 2.5v11l9-5.5z" fill="#0b0b0c"/></svg></span><small>0:42 · Finalização</small></div>
                  <span><em>Novo vídeo.</em> Treino de finalização de curta distância.</span>
                </div>
                <div class="ph-card">
                  <div class="ph-who"><span class="ph-av ph-av--club">EC</span><span><b>Peneira EC Jardim</b><br><small>Sáb, 14 dez · 08:00</small></span></div>
                  <span><em>Peneira aberta.</em> Sub-15, atacantes e laterais. Verificada pela organização do clube.</span>
                </div>
              </div>
              <div class="ph__tabs" aria-hidden="true"><span class="on">Início</span><span>Peneiras</span><span class="rec"><svg viewBox="0 0 16 16" fill="none" stroke="#FFFFFF" stroke-width="1.8" stroke-linejoin="round"><rect x="1.5" y="4" width="9" height="8" rx="1.5"/><path d="m10.5 7 4-2.2v6.4l-4-2.2z"/></svg></span><span>Mensagens</span><span>Perfil</span></div>
            </div>
          </div>
          <div class="ly-piece ph">
            <div class="ph__screen">
              <div class="ph__top"><span><small>Modo olheiro</small><strong>Buscar atletas</strong></span></div>
              <div class="ph__body">
                <div class="ph-search">Nome, posição, cidade</div>
                <div class="ph-chips"><span class="on">Centroavante</span><span>Sub-15</span><span>SP</span></div>
                <div class="ph-row"><span class="ph-av">MS</span><span><b>Miguel Santos</b><br><small>1,68 m · Campinas</small></span><span class="n">3<small>vídeos</small></span></div>
                <div class="ph-row"><span class="ph-av">GA</span><span><b>Gabriel Alves</b><br><small>1,71 m · Hortolândia</small></span><span class="n">5<small>vídeos</small></span></div>
                <div class="ph-row"><span class="ph-av">BR</span><span><b>Bruno Ribeiro</b><br><small>1,74 m · Indaiatuba</small></span><span class="n">2<small>vídeos</small></span></div>
                <div class="ph-row"><span class="ph-av">DS</span><span><b>Davi Souza</b><br><small>1,69 m · Campinas</small></span><span class="n">4<small>vídeos</small></span></div>
              </div>
              <div class="ph__tabs" aria-hidden="true"><span class="on">Buscar</span><span>Favoritos</span><span>Mensagens</span><span>Peneiras</span><span>Perfil</span></div>
            </div>
          </div>
        </div>
      </div>
      <div class="span-3 start-10 hero__cta"><a class="btn btn--primary" href="#como-funciona">Ver como funciona<svg class="m-arrow" viewBox="0 0 12 12" aria-hidden="true"><path class="m-arrow__stem" d="M1 6h8"/><path class="m-arrow__head" d="M4.5 2.5 8 6l-3.5 3.5"/></svg></a></div>
    </div></div>
  </section>

  <section class="section" id="como-funciona" style="--h: 2" data-moment="pinned-chapters" aria-labelledby="como-h">
    <div class="wrap mo-pc__grid">
      <div class="mo-pc__text">
        <h2 id="como-h">Do perfil ao contato do clube</h2>
        <ol class="mo-pc__steps">
          <li class="mo-pc__step">
            <h3>Monte seu perfil de atleta</h3>
            <p>Posição, altura, peso e cidade. Com o perfil pronto, o app mostra as peneiras da sua categoria.</p>
            <figure class="mo-pc__visual mo-pc__visual--lit" role="img" aria-label="A tela de perfil do atleta Miguel Santos: centroavante Sub-15 de Campinas, com altura, peso, idade e os vídeos dele"><div class="ph-stage"><div class="ph"><div class="ph__screen"><div class="ph__top"><span><small>Seu perfil</small><strong>Miguel Santos</strong></span></div><div class="ph__body"><div class="ph-card ph-hero"><span class="ph-av ph-av--lg">MS</span><span><b>Centroavante</b><br><small>Sub-15 · Campinas, SP</small></span></div><div class="ph-stats"><span><small>Altura</small><b>1,68 m</b></span><span><small>Peso</small><b>58 kg</b></span><span><small>Idade</small><b>14</b></span></div><div class="ph-card"><small>Meus vídeos</small><div class="ph-thumbs"><i></i><i></i><i></i></div></div><span class="ph-btn ph-btn--outline">Editar perfil</span></div><div class="ph__tabs" aria-hidden="true"><span>Início</span><span>Peneiras</span><span class="rec"><svg viewBox="0 0 16 16" fill="none" stroke="#FFFFFF" stroke-width="1.8" stroke-linejoin="round"><rect x="1.5" y="4" width="9" height="8" rx="1.5"/><path d="m10.5 7 4-2.2v6.4l-4-2.2z"/></svg></span><span>Mensagens</span><span class="on">Perfil</span></div></div></div></div></figure>
          </li>
          <li class="mo-pc__step">
            <h3>Publique seus lances</h3>
            <p>Escolha o vídeo, marque o fundamento e toque em Publicar. O olheiro vê como você joga, não só seus números.</p>
            <figure class="mo-pc__visual mo-pc__visual--lit" role="img" aria-label="A tela de novo vídeo: um lance de 42 segundos marcado como finalização, com a legenda e o botão Publicar"><div class="ph-stage"><div class="ph"><div class="ph__screen"><div class="ph__top"><span><small>Publicar</small><strong>Novo vídeo</strong></span></div><div class="ph__body"><div class="ph-video"><span><svg viewBox="0 0 12 12" fill="#0B0F14"><path d="M3 1.8v8.4L10 6z"/></svg></span><small>0:42</small></div><div class="ph-field"><small>Fundamento</small><div class="ph-chips"><span class="on">Finalização</span><span>Drible</span><span>Passe</span><span>Defesa</span></div></div><div class="ph-field"><small>Legenda (opcional)</small><span class="ph-input">Treino de finalização de curta distância</span></div><span class="ph-btn">Publicar</span></div><div class="ph__tabs" aria-hidden="true"><span>Início</span><span>Peneiras</span><span class="rec"><svg viewBox="0 0 16 16" fill="none" stroke="#FFFFFF" stroke-width="1.8" stroke-linejoin="round"><rect x="1.5" y="4" width="9" height="8" rx="1.5"/><path d="m10.5 7 4-2.2v6.4l-4-2.2z"/></svg></span><span>Mensagens</span><span>Perfil</span></div></div></div></div></figure>
          </li>
          <li class="mo-pc__step">
            <h3>Inscreva-se nas peneiras</h3>
            <p>Encontre a peneira pela cidade, categoria ou posição e toque em Quero participar. O clube vê você na lista de inscritos.</p>
            <figure class="mo-pc__visual mo-pc__visual--lit" role="img" aria-label="A aba de peneiras: a peneira do EC Jardim com o botão Quero participar e outra peneira já com o selo Inscrito"><div class="ph-stage"><div class="ph"><div class="ph__screen"><div class="ph__top"><span><small>Sub-15 · Campinas</small><strong>Peneiras</strong></span></div><div class="ph__body"><div class="ph-card"><div class="ph-who"><span class="ph-av ph-av--club">EC</span><span><b>Peneira EC Jardim</b><br><small>Sáb, 14 dez · 08:00</small></span></div><small>Sub-15 · atacantes e laterais · 12 vagas</small><span class="ph-btn">Quero participar</span></div><div class="ph-card"><div class="ph-who"><span class="ph-av">AS</span><span><b>Peneira Atlético Serrano</b><br><small>Dom, 15 dez · 09:00</small></span><span class="ph-badge">Inscrito</span></div></div><div class="ph-card"><div class="ph-who"><span class="ph-av">UC</span><span><b>Peneira União Campineira</b><br><small>Sáb, 21 dez · 08:30</small></span></div></div></div><div class="ph__tabs" aria-hidden="true"><span>Início</span><span class="on">Peneiras</span><span class="rec"><svg viewBox="0 0 16 16" fill="none" stroke="#FFFFFF" stroke-width="1.8" stroke-linejoin="round"><rect x="1.5" y="4" width="9" height="8" rx="1.5"/><path d="m10.5 7 4-2.2v6.4l-4-2.2z"/></svg></span><span>Mensagens</span><span>Perfil</span></div></div></div></div></figure>
          </li>
          <li class="mo-pc__step">
            <h3>O clube fala com você</h3>
            <p>Olheiros de clubes aprovados mandam mensagem pelo app. Menores de 18 anos conversam só com a autorização do responsável.</p>
            <figure class="mo-pc__visual mo-pc__visual--lit" role="img" aria-label="Uma conversa entre o EC Jardim e o atleta, combinando o que levar na peneira de sábado"><div class="ph-stage"><div class="ph"><div class="ph__screen"><div class="ph__top"><span><small>Conversa</small><strong>EC Jardim</strong></span></div><div class="ph__body"><div class="ph-msg">Vimos seu vídeo de finalização. Você vem na peneira de sábado?</div><div class="ph-msg ph-msg--me">Vou sim! Preciso levar alguma coisa?</div><div class="ph-msg">Chuteira, caneleira e o documento. Leve também o atestado médico.</div><div class="ph-msg ph-msg--me">Combinado, até sábado.</div></div><div class="ph__tabs" aria-hidden="true"><span>Início</span><span>Peneiras</span><span class="rec"><svg viewBox="0 0 16 16" fill="none" stroke="#FFFFFF" stroke-width="1.8" stroke-linejoin="round"><rect x="1.5" y="4" width="9" height="8" rx="1.5"/><path d="m10.5 7 4-2.2v6.4l-4-2.2z"/></svg></span><span class="on">Mensagens</span><span>Perfil</span></div></div></div></div></figure>
          </li>
        </ol>
      </div>
      <div class="mo-pc__visuals frame"></div>
    </div>
  </section>

  <section class="section" id="peneiras" aria-labelledby="peneiras-h">
    <div class="wrap">
      <div class="grid">
        <h2 id="peneiras-h" class="span-6">Todos os inscritos da peneira em uma tela</h2>
        <p class="span-6 start-7">O clube ou olheiro publica a peneira, vê cada inscrito com posição, idade e cidade, abre o vídeo e conversa com o jogador ali mesmo.</p>
      </div>
      <div class="ui app fill" role="img" aria-label="A tela do clube no Rise Eleven: a lista de inscritos da peneira Sub-15 ao lado da conversa com o jogador Miguel Santos" data-ui-pause-host>
        <div class="app__pane app__pane--list">
          <div class="ui ui-panel ui-win" data-ui="window" data-open="Nova" data-work="Vendo vídeo" data-done="Favorito">
            <div class="ui-win__bar">
              <span class="ui-dots" aria-hidden="true"><i></i><i></i><i></i></span>
              <span class="ui-win__crumb"><svg class="ui-brand" aria-hidden="true" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><path d="M3 13a5 5 0 0 1 10 0"/></svg>Peneiras <i>/</i> <b>Sub-15</b></span>
              <span class="ui-spacer"></span>
              <span class="ui-win__search" aria-hidden="true"><svg class="ui-icon" viewBox="0 0 16 16"><circle cx="7" cy="7" r="4.5"/><path d="m10.5 10.5 3 3"/></svg><span class="ui-win__ph">Buscar</span><span class="ui-kbd">/</span></span>
            </div>
            <div class="ui-win__body">
              <nav class="ui-win__side" aria-hidden="true">
                <span class="ui-win__nav is-on"><svg class="ui-icon" viewBox="0 0 16 16"><circle cx="8" cy="6" r="2.5"/><path d="M3.5 13.5c.6-2.3 2.4-3.5 4.5-3.5s3.9 1.2 4.5 3.5"/></svg>Inscritos<em>22</em></span>
                <span class="ui-win__nav"><svg class="ui-icon" viewBox="0 0 16 16"><path d="M8 13.2 3.2 8.6a2.9 2.9 0 0 1 4.8-3.2 2.9 2.9 0 0 1 4.8 3.2z"/></svg>Favoritos<em>6</em></span>
                <span class="ui-win__nav"><svg class="ui-icon" viewBox="0 0 16 16"><circle cx="8" cy="8" r="5.5"/><path d="M8 5v3l2 1.5"/></svg>Aguardando<em>3</em></span>
                <span class="ui-win__nav"><svg class="ui-icon" viewBox="0 0 16 16"><path d="M2.5 4.5h11v8h-11z"/><path d="m2.5 5 5.5 4 5.5-4"/></svg>Mensagens</span>
                <span class="ui-win__label">Peneiras</span>
                <span class="ui-win__nav"><span class="ui-win__hash">#</span>Sub-15</span>
                <span class="ui-win__nav"><span class="ui-win__hash">#</span>Sub-17</span>
              </nav>
              <div class="ui-win__main">
                <div class="ui-win__cols" aria-hidden="true"><span>Jogador</span><span>Posição</span><span>Status</span><span>Há</span></div>
                <ul class="ui-win__rows">
                  <li class="ui-win__row" data-row><span class="ui-avatar" data-tone="1">MS</span><span class="ui-win__who">Miguel Santos</span><span class="ui-win__sub">Atacante · 14 anos · Campinas</span><span class="ui-win__st" data-st>Nova</span><span class="ui-win__age">2 min</span></li>
                  <li class="ui-win__row" data-row><span class="ui-avatar" data-tone="3">JP</span><span class="ui-win__who">João Pedro Lima</span><span class="ui-win__sub">Lateral · 15 anos · Sumaré</span><span class="ui-win__st" data-st>Nova</span><span class="ui-win__age">9 min</span></li>
                  <li class="ui-win__row" data-row><span class="ui-avatar" data-tone="4">GA</span><span class="ui-win__who">Gabriel Alves</span><span class="ui-win__sub">Atacante · 14 anos · Hortolândia</span><span class="ui-win__st" data-st>Nova</span><span class="ui-win__age">16 min</span></li>
                  <li class="ui-win__row" data-row><span class="ui-avatar" data-tone="2">RC</span><span class="ui-win__who">Rafael Costa</span><span class="ui-win__sub">Lateral · 15 anos · Campinas</span><span class="ui-win__st is-you" data-st>Aguardando</span><span class="ui-win__age">24 min</span></li>
                  <li class="ui-win__row" data-row><span class="ui-avatar" data-tone="5">LO</span><span class="ui-win__who">Lucas Oliveira</span><span class="ui-win__sub">Atacante · 15 anos · Valinhos</span><span class="ui-win__st is-done" data-st>Favorito</span><span class="ui-win__age">41 min</span></li>
                  <li class="ui-win__row" data-row><span class="ui-avatar" data-tone="3">TF</span><span class="ui-win__who">Thiago Ferreira</span><span class="ui-win__sub">Lateral · 14 anos · Paulínia</span><span class="ui-win__st is-done" data-st>Favorito</span><span class="ui-win__age">1 h</span></li>
                  <li class="ui-win__row" data-row><span class="ui-avatar" data-tone="1">BR</span><span class="ui-win__who">Bruno Ribeiro</span><span class="ui-win__sub">Atacante · 15 anos · Indaiatuba</span><span class="ui-win__st" data-st>Nova</span><span class="ui-win__age">1 h</span></li>
                  <li class="ui-win__row" data-row><span class="ui-avatar" data-tone="4">EM</span><span class="ui-win__who">Enzo Martins</span><span class="ui-win__sub">Lateral · 14 anos · Americana</span><span class="ui-win__st is-you" data-st>Aguardando</span><span class="ui-win__age">2 h</span></li>
                  <li class="ui-win__row" data-row><span class="ui-avatar" data-tone="2">DS</span><span class="ui-win__who">Davi Souza</span><span class="ui-win__sub">Atacante · 15 anos · Campinas</span><span class="ui-win__st is-done" data-st>Favorito</span><span class="ui-win__age">3 h</span></li>
                  <li class="ui-win__row" data-row><span class="ui-avatar" data-tone="5">HC</span><span class="ui-win__who">Heitor Carvalho</span><span class="ui-win__sub">Lateral · 15 anos · Jundiaí</span><span class="ui-win__st" data-st>Nova</span><span class="ui-win__age">4 h</span></li>
                </ul>
              </div>
            </div>
          </div>
        </div>
        <div class="app__pane app__pane--open" data-ui-loop>
          <div class="ui ui-panel ui-chat" data-ui="chat">
            <div class="ui-head">
              <span class="ui-avatar" data-tone="1" aria-hidden="true">MS</span>
              <div class="ui-title">Miguel Santos · Sub-15</div>
              <span class="ui-chat__online">Responsável confirmado</span>
              <span class="ui-spacer"></span>
            </div>
            <div class="ui-chat__body">
              <div class="ui-chat__day">Hoje, 9:41</div>
              <div class="ui-msg ui-msg--me"><span class="ui-msg__text">Oi, Miguel. Vimos seu vídeo do gol de falta. Você consegue vir à peneira de sábado?</span></div>
              <div class="ui-msg ui-msg--me" data-chat-q><span class="ui-msg__text">É no campo do clube, às 8h. Precisa de algo para confirmar?</span></div>
              <div class="ui-msg ui-msg--bot" data-chat-a>
                <span class="ui-avatar ui-msg__av" data-tone="1" aria-hidden="true">MS</span>
                <span class="ui-msg__col">
                  <span class="ui-chat__typing" aria-hidden="true"><i></i><i></i><i></i></span>
                  <span class="ui-chat__answer">Consigo, sim. Minha mãe, <b>Carla Santos</b>, já autorizou pelo app. Levo chuteira de campo e RG.</span>
                  <span class="ui-chat__srcs">
                    <span class="ui-chat__src"><svg class="ui-icon" viewBox="0 0 16 16"><path d="m3.5 8.4 2.8 2.8 6.2-6.4"/></svg>Autorização do responsável</span>
                    <span class="ui-chat__src"><svg class="ui-icon" viewBox="0 0 16 16"><path d="M3 3.5h10v9H3z"/><path d="M3 6.5h10M6 2v3M10 2v3"/></svg>Peneira Sub-15, 14 dez</span>
                  </span>
                </span>
              </div>
              <div class="ui-msg ui-msg--me"><span class="ui-msg__text">Perfeito. Leve também atestado médico. Te esperamos às 8h.</span></div>
              <div class="ui-msg ui-msg--bot">
                <span class="ui-avatar ui-msg__av" data-tone="1" aria-hidden="true">MS</span>
                <span class="ui-msg__col"><span class="ui-chat__answer">Combinado. Obrigado pela chance.</span></span>
              </div>
              <div class="ui-msg ui-msg--me"><span class="ui-msg__text">Endereço e horário já estão na página da peneira.</span></div>
            </div>
            <div class="ui-chat__input">
              <span class="ui-chat__ph">Escreva uma mensagem</span>
              <span class="ui-chat__send" aria-hidden="true"><svg viewBox="0 0 16 16" width="14" height="14"><path d="M8 13V3.5M3.8 7.5 8 3.3l4.2 4.2" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round"/></svg></span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>

  <section class="section" id="perguntas" style="--hm: 2" aria-labelledby="perguntas-h">
    <div class="wrap">
      <div class="grid grid--top">
        <div class="span-6 stack-4">
          <h2 id="perguntas-h">Perguntas sobre o Rise Eleven</h2>
          <p>O que jogadores, responsáveis e clubes mais querem saber antes do lançamento.</p>
        </div>
        <div class="faq span-6 start-7">
          <div class="faq__item"><h3>Quando o app vai lançar?</h3><p>Já dá para usar no navegador, no celular ou no computador. Os apps para iPhone e Android vêm em seguida.</p></div>
          <div class="faq__item"><h3>Menores de 18 anos podem usar?</h3><p>Sim, com autorização do responsável legal. Ele recebe um código de 6 dígitos por e-mail; até confirmar, o perfil fica fora da busca e não troca mensagens.</p></div>
          <div class="faq__item"><h3>Como me inscrevo numa peneira?</h3><p>Monte seu perfil, encontre a peneira pela cidade, categoria ou posição e toque em Inscrever-se. O clube vê sua inscrição na lista de inscritos.</p></div>
          <div class="faq__item"><h3>Quanto custa o Rise Eleven?</h3><p>Nada. No lançamento o app é grátis para jogadores, responsáveis, clubes e olheiros.</p></div>
          <div class="faq__item"><h3>Como sei que uma peneira é real?</h3><p>Clubes e organizadores que publicam peneiras passam por verificação, e cada peneira verificada mostra o selo do clube.</p></div>
        </div>
      </div>
    </div>
  </section>

  <section class="section g2" id="recursos" aria-labelledby="recursos-h">
    <div class="wrap">
      <div class="group">
        <div class="section-head">
          <h2 id="recursos-h">Do perfil à peneira. <span class="muted">Tudo o que o jogador precisa para ser visto.</span></h2>
        </div>
        <div class="grid">
          <article class="card card--row card--open span-6 m-nolift">
            <h3>Encontra peneiras perto de casa</h3>
            <p>Busque por cidade, categoria ou posição e veja data, endereço e vagas restantes.</p>
            <div class="panel">
              <div class="stage stage--fade-b"><div class="at-rise fit to-edge" style="--fit-max:380px;--ax:max(24px, 50% - 190px);--ay:56px;--ay-sm:56px" data-ui-loop>
                <div class="ui ui-panel ui-search" data-ui="search">
                  <div class="ui-search__field">
                    <svg class="ui-icon" viewBox="0 0 16 16" aria-hidden="true"><circle cx="7" cy="7" r="4.5"/><path d="m10.5 10.5 3 3"/></svg>
                    <span class="ui-search__q"><span data-search-q>peneira sub-15 campinas</span><i class="ui-search__caret"></i></span>
                  </div>
                  <div class="ui-search__group">Peneiras</div>
                  <ul class="ui-search__list" data-search-list>
                    <li class="ui-search__row is-sel"><span class="ui-search__ico"><svg class="ui-icon" viewBox="0 0 16 16"><path d="M3 3.5h10v9H3z"/><path d="M3 6.5h10M6 2v3M10 2v3"/></svg></span><span class="ui-search__t"><b>Sub-15</b> em <b>Campinas</b></span><span class="ui-search__m">14 dez</span></li>
                    <li class="ui-search__row"><span class="ui-search__ico"><svg class="ui-icon" viewBox="0 0 16 16"><path d="M3 3.5h10v9H3z"/><path d="M3 6.5h10M6 2v3M10 2v3"/></svg></span><span class="ui-search__t"><b>Sub-15</b> em Sumaré</span><span class="ui-search__m">21 dez</span></li>
                    <li class="ui-search__row"><span class="ui-search__ico"><svg class="ui-icon" viewBox="0 0 16 16"><path d="M3 3.5h10v9H3z"/><path d="M3 6.5h10M6 2v3M10 2v3"/></svg></span><span class="ui-search__t"><b>Sub-15</b> em Valinhos</span><span class="ui-search__m">11 jan</span></li>
                  </ul>
                  <div class="ui-search__foot"><span>3 peneiras a menos de 40 km</span></div>
                </div>
              </div></div>
            </div>
          </article>
          <article class="card card--row card--open span-6 m-nolift">
            <h3>Avisa quando um clube se interessa</h3>
            <p>Um favorito ou uma mensagem de clube chega na hora, com o nome do clube e da peneira.</p>
            <div class="panel">
              <div class="stage stage--fade-b"><div class="at-rise fit" style="--fit-max:380px;--ax:32px;--ay:calc(100% - 180px);--ay-sm:calc(100% - 180px)">
                <div class="ui ui-notify" data-ui="notify">
                  <ul class="ui-notify__list" data-notify-list>
                    <li class="ui-note"><span class="ui-avatar ui-note__av" data-tone="2">EC</span><span class="ui-note__text"><span class="ui-note__title">EC Jardim favoritou você<time>agora</time></span><span class="ui-note__body">Depois de ver seu vídeo de jogadas</span></span></li>
                    <li class="ui-note"><span class="ui-avatar ui-note__av" data-tone="4">AV</span><span class="ui-note__text"><span class="ui-note__title">Nova mensagem do AA Vila<time>12 min</time></span><span class="ui-note__body">Sobre a peneira Sub-15 de sábado</span></span></li>
                    <li class="ui-note"><span class="ui-note__ico"><svg class="ui-icon" viewBox="0 0 16 16"><path d="m3.5 8.4 2.8 2.8 6.2-6.4"/></svg></span><span class="ui-note__text"><span class="ui-note__title">Inscrição confirmada<time>1 h</time></span><span class="ui-note__body">Peneira Sub-15, 14 dez, Campinas</span></span></li>
                  </ul>
                </div>
              </div></div>
            </div>
          </article>
        </div>
      </div>
      <div class="grid">
        <article class="card card--row card--open span-6 m-nolift">
          <h3>Monta o perfil do jogador</h3>
          <p>Posição, altura, cidade e os vídeos das melhores jogadas, num perfil que os clubes encontram.</p>
          <div class="panel">
            <div class="stage stage--fade-b"><div class="at-rise fit" style="--fit-max:340px;--ax:max(24px, 50% - 170px);--ay:40px;--ay-sm:40px">
              <div class="ui ui-panel ui-checks" data-ui="checklist">
                <div class="ui-checks__head">
                  <div class="ui-title">Perfil de Miguel Santos</div>
                  <span class="ui-spacer"></span>
                  <div class="ui-meta" data-checks-count>3 de 5</div>
                </div>
                <div class="ui-checks__bar"><i data-checks-bar style="--k:0.6"></i></div>
                <ul class="ui-checks__list">
                  <li class="ui-check is-done"><span class="ui-check__box"><svg viewBox="0 0 16 16"><path d="m4.5 8.2 2.3 2.3 4.7-5"/></svg></span><span class="ui-check__label">Posição e pé dominante</span></li>
                  <li class="ui-check is-done"><span class="ui-check__box"><svg viewBox="0 0 16 16"><path d="m4.5 8.2 2.3 2.3 4.7-5"/></svg></span><span class="ui-check__label">Altura e peso</span></li>
                  <li class="ui-check is-done"><span class="ui-check__box"><svg viewBox="0 0 16 16"><path d="m4.5 8.2 2.3 2.3 4.7-5"/></svg></span><span class="ui-check__label">Cidade e estado</span></li>
                  <li class="ui-check"><span class="ui-check__box"><svg viewBox="0 0 16 16"><path d="m4.5 8.2 2.3 2.3 4.7-5"/></svg></span><span class="ui-check__label">Vídeo de jogadas</span></li>
                  <li class="ui-check"><span class="ui-check__box"><svg viewBox="0 0 16 16"><path d="m4.5 8.2 2.3 2.3 4.7-5"/></svg></span><span class="ui-check__label">Autorização do responsável</span></li>
                </ul>
              </div>
            </div></div>
          </div>
        </article>
        <article class="card card--row card--open span-6 m-nolift" id="seguranca">
          <h3>Protege jogadores menores de idade</h3>
          <p>O responsável confirma por um código no e-mail. Antes disso, o perfil fica oculto e sem mensagens.</p>
          <div class="panel">
            <div class="stage stage--fade-b"><div class="at-right to-edge" style="--ax:32px;--ay:56px;--ay-sm:56px">
              <div class="ui ui-panel ui-toggles" data-ui="toggles">
                <div class="ui-head">
                  <svg class="ui-icon ui-toggles__ico" viewBox="0 0 16 16" aria-hidden="true"><path d="M8 1.8 13 4v3.6c0 3-2.1 5.4-5 6.6-2.9-1.2-5-3.6-5-6.6V4z"/><path d="m5.8 8 1.6 1.6 3-3.2"/></svg>
                  <div class="ui-title">Regras para menores de 18</div>
                  <span class="ui-spacer"></span>
                  <div class="ui-meta" data-tg-count>4 ativas</div>
                </div>
                <ul class="ui-toggles__list">
                  <li class="ui-tg"><span class="ui-tg__text"><span class="ui-tg__label">Perfil oculto até a confirmação</span><span class="ui-tg__sub">Clubes não encontram na busca</span></span><button class="ui-switch is-on" type="button" role="switch" aria-checked="true" aria-label="Perfil oculto até a confirmação" tabindex="-1"><i></i></button></li>
                  <li class="ui-tg"><span class="ui-tg__text"><span class="ui-tg__label">Mensagens só depois da confirmação</span><span class="ui-tg__sub">Nada entra nem sai antes</span></span><button class="ui-switch is-on" type="button" role="switch" aria-checked="true" aria-label="Mensagens só depois da confirmação" tabindex="-1"><i></i></button></li>
                  <li class="ui-tg"><span class="ui-tg__text"><span class="ui-tg__label">Código vale por 24 horas</span><span class="ui-tg__sub">Até 5 tentativas</span></span><button class="ui-switch is-on" type="button" role="switch" aria-checked="true" aria-label="Código vale por 24 horas" tabindex="-1"><i></i></button></li>
                  <li class="ui-tg"><span class="ui-tg__text"><span class="ui-tg__label">Denúncia de conversa</span><span class="ui-tg__sub">Em qualquer conversa com clube</span></span><button class="ui-switch is-on" type="button" role="switch" aria-checked="true" aria-label="Denúncia de conversa" tabindex="-1"><i></i></button></li>
                </ul>
              </div>
            </div></div>
          </div>
        </article>
      </div>
    </div>
  </section>

</main>

<footer class="footer closing footer--statement" id="contact" aria-labelledby="close-h">
  <div class="wrap">
    <div class="closing__cta close--word">
      <div class="section-head--split">
        <div><h2 id="close-h">Use o Rise Eleven no navegador</h2><p class="t-lead">Crie sua conta de jogador ou de clube agora. Os apps para iPhone e Android vêm em seguida.</p></div>
        <a class="btn btn--primary" href="https://rise-eleven.expo.app">Abrir o app<svg class="m-arrow" viewBox="0 0 12 12" aria-hidden="true"><path class="m-arrow__stem" d="M1 6h8"/><path class="m-arrow__head" d="M4.5 2.5 8 6l-3.5 3.5"/></svg></a>
      </div>
      <p class="close__word" aria-hidden="true">RISE<b>11</b></p>
    </div>
    <div class="closing__links">
      <div class="grid">
        <div class="span-6 stack-4">
          <a class="nav__logo" href="#top"><span class="wordmark" aria-label="Rise Eleven">RISE<b>11</b></span></a>
          <p class="footer__lede">Peneiras, olheiros e perfis de jogadores para o futebol de base.</p>
        </div>
        <div class="span-3 start-7 footer__col"><h4>Produto</h4><ul><li><a href="#peneiras">Peneiras</a></li><li><a href="#recursos">Recursos</a></li></ul></div>
        <div class="span-3 footer__col"><h4>Ajuda</h4><ul><li><a href="#seguranca">Segurança</a></li><li><a href="#perguntas">Perguntas</a></li></ul></div>
      </div>
      <div class="footer__legal">
        <span>&copy; 2026 Rise Eleven</span>
        <div class="socials"></div>
      </div>
    </div>
  </div>
</footer>

<script type="module" src="assets/system.js"></script>
<script type="module" src="assets/motion/motion.js"></script>
<script type="module" src="assets/layout/layout.js"></script>
</body>
</html>
```
