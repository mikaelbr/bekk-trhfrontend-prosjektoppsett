## Oppsett av JavaScript

1. Oppsett av `project.json`: `npm init -y`
2. Sett opp filstruktur
  * `src/app.js`: `mkdir src; touch src/app.js`
3. Lag `index.html` som peker til `src/app.js`.
4. Verifiser at det fungerer med å åpne `index.html`
5. Sette oppp andre avhengigheter
  * `src/lib/math.js`: `mkdir src/lib; touch src/lib/math.js`
6. Verifiser at det *ikke* fungerer med å åpne `index.html`
7. Sett opp Webpack:
  1. Se versjon: `npm show webpack version` og `npm show webpack@beta version`
  2. Installer 2 beta: `npm i -D webpack@beta`
  3. Sett opp NPM script:
    * `build`: `webpack -d src/app.js dist/build.js`
    * Endre `index.html` til `dist/build.js`
    * Åpne i browser å se at det fungerer som forventet. Se på kildekoden.
    * Legg til en prod-build av det: `build-prod`: `webpack -p src/app.js dist/build.js`
    * Verifiser innholdet av `build.js` for å se endret output til minifisert med UglifyJS2.
8. Endre innhold i `app.js` til å bruke en ES2015 feature:
  ```js
  import add from './lib/math';

  const printAdd = (a, b) =>
    console.log(`Adding ${a} + ${b}: ${add(a, b)}`);
  console.log('Hello, World!');
  printAdd(1, 2);
  ```
9. Kjør `npm run build` for å se at det fungerer.
10. Sjekk innholdet av `build.js` og se at det er ikke transpilert på noen måte, men det fungerer i Chrome.
11. Prøv å kjør `npm run build-prod` for å se at det failer. UglifyJS2 fungerer ikke på ES2015 syntaks. Heller ikke alle nettlesere.
12. Transpillering av ES2015:
  1. Installer `babel-cli` og `babel-preset-es2015`: `npm i -D babel-cli babel-preset-es2015`
  2. Opprett `.babelrc`:
    ```json
    {
      "presets": ["es2015"]
    }
    ```
  3. Legg til babel i NPM Scripts: `"transpile": "babel src/app.js -o dist/build.js"`
  4. Kjør `npm run transpile` for å se at den bundler ikke flere filer, bare transpilerer.
13. Slå sammen `babel` og `webpack`:
  1. Installer `babel-loader`: `npm i -D babel-loader`
  2. Lag en webpack.config.js:
  ```js
  module.exports = {
    module: {
      rules: [
        {
          test: /\.js$/,
          exclude: /node_modules/,
          use: ['babel-loader'],
        },
      ],
    }
  };
  ```
  3. Kjør bygging: `npm run build`
  4. Verifiser resultatet i `index.html`
  5. Se innhold av pakken at det nå er transpilert.
  6. Fjern `transpile` NPM Script.
  7. Sjekk at `npm run build-prod` fungerer
14. Se at `import` blir transpilert av babel som gjør at three shaking ikke lengre fungerer.
  1. For å få dette til å fungere så må vi la Babel håndtere moduler. Endre babelrc til å ha:
  ```json
  {
    "presets": [
      ["es2015", { "modules": false }]
    ]
  }
  ```
  2. Verifiser at det fungerer med å se i `bundle.js` at nå blir harmony import brukt.
15. Bruk `-- -w` som en del av `npm run build` dersom man ønsker watch, eller legg det til som egen script.


---


## Oppsett av LESS

1. Sett opp `src/app.less`: `touch src/app.less`
2. Sett opp `body { color: red; }` for å kunne vise til om det fungerer.
3. Sett opp less:
  1. `npm i -D less`
  2. Sett opp `npm script`: `"build:less": "lessc src/app.less dist/bundle.css"`
4. Verifiser `bundle.less` at den har innhold.
5. Legg til `src/lib/reset.less`: `touch src/lib/reset.less`
  ```less
  * { padding: 0; margin: 0; }
  ```
6. Verifiser at `bundle.less` har `reset.less`.
7. Legg til `dist/bundle.css` til `index.html`
8. Åpne `index.html` for å verifisere at det fungerer.
9. Legg til transition til `:hover` på h1 i `app.less`:
  ```less
  h1 {
    transition: transform 300ms ease-in;
    &:hover {
      transform: translateY(10px);
    }
  }
  ```
10. Kjør `npm run build:less` for å kompillere CSS.
11. Sjekk at h1 animeres på hover i `index.html`
12. Se at transition er ikke prefixed by vendor. Vil ikke fungere i eldre nettlesere. Særlig relevant for flex osv.
13. Installer `autoprefixer`: `npm i -D less-plugin-autoprefix`
14. Aktiver autoprefixer plugin:
  ```json
    "build:less": "lessc --autoprefix src/app.less dist/bundle.css"
  ```
15. Slitsomt å kompillere selv hver gang. Sett opp watching:
  1. Installed onchange `npm i -D onchange`
  2. Sett opp watch NPM script
  ```json
  "watch:less": "onchange \"src/**/*.{less,css}\" -- npm run build:less"
  ```
  3. Start med `npm run less:watch` og verifiser at det fungerer.

Da skal LESS og WebPack være satt opp og fungere. Kan ta et skritt til å slå
sammen bygg og watch oppgaver til en oppgave ved hjelp av [npm-run-all](http://npm.im/npm-run-all):

```
npm-run-all --parallel watch:*
npm-run-all --parallel build:*
```
