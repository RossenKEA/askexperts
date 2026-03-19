# AskExperts – Refleksion

## Udfordringer og succeser

Noget af det første vi stødte på var at den samme sektion skulle bruges to gange på about-siden, men spejlvendt. I stedet for at kopiere koden lavede vi en `reverse`-prop på `WhatToExpect`-komponenten. Tricket var at bruge `direction: rtl` på containeren og sætte `direction: ltr` tilbage på children – så vender layoutet uden at HTML-strukturen ændres:

```css
#wte_inner[data-reverse="true"] {
  direction: rtl;
}
#wte_inner[data-reverse="true"] > * {
  direction: ltr;
}
```

En anden ting der tog lidt tid var `CoreValues`, som skulle se forskellig ud på forsiden (mørk) og about-siden (lys). Problemet var at Astros scoped styles ikke kan overskrives udefra. Vi løste det ved at sende baggrunden som en inline style og så tjekke den i CSS:

```css
#cv[style*="030712"] .cv_heading { color: var(--color-white); }
#cv[style*="030712"] .cv_btn { background: var(--color-white); color: var(--color-black); }
```

Det er ikke den reneste løsning, men den virker og holder stylingen samlet i ét komponent.

Noget vi var mere tilfredse med var den dynamiske profilside. Vi samlede al data om teammedlemmer i én fil og brugte Astros `getStaticPaths()` til at generere en side per person:

```js
export function getStaticPaths() {
  return members.map((m) => ({
    params: { member: m.slug },
    props: { member: m },
  }));
}
```

Det betød at vi ikke skulle lave seks separate sider i hånden, og at data kun er ét sted.

---

## Defensive CSS

Vi har tænkt defensive CSS ind løbende, mest fordi vi brændte fingrene et par gange på billeder der strakte sig ud af deres containere. Løsningen var `overflow: hidden` på wrapperen og `object-fit: cover` på selve billedet:

```css
.team__photowrap {
  border-radius: 20px;
  overflow: hidden;
}
.team__photo {
  width: 100%;
  height: 392px;
  object-fit: cover;
}
```

Knapper og ikoner i flex-layouts fik `flex-shrink: 0` så de ikke komprimeres når pladsen bliver trang:

```css
.cv_btn { flex-shrink: 0; }
```

På case-study-sidens sorte meta-boks brugte vi `min()` frem for `width` + `max-width`, da det er mere kompakt og gør det samme:

```css
.cs_image_meta {
  width: min(1028px, 90%);
}
```

Og tekstblokke har `max-width` så linjerne ikke bliver for lange på brede skærme:

```css
.hero__subhead { max-width: 700px; }
```

---

## Progressive Enhancement

Grundprincippet var at siden skal fungere uden JavaScript, og så lægge interaktion ovenpå bagefter.

Det gælder især burger-menuen. Uden JS vises desktop-navigationen normalt. JS tilføjer kun toggle-funktionen til mobilmenuen – man kan stadig navigere siden uden det:

```js
burger?.addEventListener("click", () => {
  const open = burger.getAttribute("aria-expanded") === "true";
  burger.setAttribute("aria-expanded", String(!open));
  menu?.classList.toggle("open", !open);
});
```

Vi brugte `aria-expanded` og `aria-hidden` til at kommunikere menuens tilstand til skærmlæsere, uafhængigt af det visuelle.

Login-popup'en er et andet eksempel. Den bruger HTMLs native `popover`-API, som fungerer i moderne browsere helt uden JavaScript:

```html
<Button popovertarget="login-popover" />
<div popover id="login-popover">...</div>
```

---

## CSS-organisering

Vi har holdt en klar opdeling: globalt i `tokens.css`, komponent-specifikt scoped i hvert `.astro`-fil.

`tokens.css` indeholder kun design tokens – farver, spacing og typografi. Det betyder at vi ikke hardcoder værdier rundt omkring, og at det er nemt at ændre et token ét sted og have det slå igennem overalt:

```css
:root {
  --color-black:  #030712;
  --color-yellow: #fbc336;
  --step-0:  clamp(0.94rem, 0.84rem + 0.47vw, 1.11rem);
  --space-m: clamp(1.5rem,  1.38rem + 0.65vw, 1.88rem);
}
```

`clamp()` på typografi og spacing var en bevidst beslutning – det giver flydende skalering frem for hårde breakpoints, og det reducerer mængden af media queries.

Al øvrig CSS sidder scoped i det komponent den hører til. Astro håndterer det automatisk med data-attributter, så `.cv_heading` i `CoreValues` og `.cv_heading` i et andet komponent aldrig blander sig. Det gør det nemt at arbejde på ét komponent uden at bekymre sig om utilsigtede sideeffekter andre steder på siden.
