---
title: "応用②：今回使用するHTML文"
---

## 本項について

ここでは、これから JavaScript で操作していくための `html` 要素を掲載します。

:::message

スタイルに関する記述や説明は、本書の趣旨から外れるため除外します。

:::

:::details HTML全文（長いので格納しています）

```html
<div>
  <section aria-label="ディスプレイ">
    <span id="b01js_label"></span>
    <output id="b01js_output"></output>
  </section>
  <section aria-label="機能">
    <button type="button">閉じる</button>
    <button type="button" id="b01js_AllClear">全消去</button>
    <button type="button" id="b01js_BackSpace">1字消去</button>
    <button type="button" id="b01js_Copy">コピー</button>
  </section>
  <section aria-label="テンキー・実行">
    <button type="button" id="b01js_9" data-calc="9">9</button>
    <button type="button" id="b01js_8" data-calc="8">8</button>
    <button type="button" id="b01js_7" data-calc="7">7</button>
    <button type="button" id="b01js_6" data-calc="6">6</button>
    <button type="button" id="b01js_5" data-calc="5">5</button>
    <button type="button" id="b01js_4" data-calc="4">4</button>
    <button type="button" id="b01js_3" data-calc="3">3</button>
    <button type="button" id="b01js_2" data-calc="2">2</button>
    <button type="button" id="b01js_1" data-calc="1">1</button>
    <button type="button" id="b01js_0" data-calc="0">0</button>
    <button type="button" id="b01js_DecimalPoint" data-calc="." aria-label="小数点">.</button>
    <button type="button" id="b01js_Equal" data-calc="=" aria-label="計算実行">=</button>
  </section>
  <section aria-label="四則演算">
    <button type="button" id="b01js_Add" data-calc="+" aria-label="足す">＋</button>
    <button type="button" id="b01js_Subtract" data-calc="-" aria-label="引く">−</button>
    <button type="button" id="b01js_Multiply" data-calc="*" aria-label="掛ける">×</button>
    <button type="button" id="b01js_Divide" data-calc="/" aria-label="割る">÷</button>
  </section>
</div>
```

:::

:::details HTML全文（スタイル付き）

検証しやすいよう、スタイルを含めたHTMLデータも掲載しておきます。空のHTMLファイルを作成し、そのままこの内容をコピーするだけで、スタイリングされた状態になるようにしています。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Calculator</title>
  <style>
    :root {
      font-family: "Noto Sans JP", "Segoe UI", "Hiragino Kaku Gothic ProN", "BIZ UDPGothic", meiryo, sans-serif;
      line-height: 1.5;

      /* スペース */
      --space_0: 0.25rem;
      --space_1: 0.5rem;
      --space_2: 1rem;
      --space_3: 1.5rem;
      --space_5: 2.5rem;
      --space_8: 4rem;
      --space_13: 6.5rem;

      /* プリミティブカラー */
      --white: #ffffff;
      --sea-1200: #00004F;
      --sea-1100: #000060;
      --sea-1000: #000071;
      --sea-900: #0000be;
      --sea-800: #0024ce;
      --sea-700: #0031d8;
      --sea-600: #003ee5;
      --sea-500: #0946f1;
      --sea-400: #4979f5;
      --sea-300: #7096f8;
      --sea-200: #9db7f9;
      --sea-100: #c5d7fb;
      --sea-50: #e8f1fe;
      --sumi-1200: #000000;
      --sumi-1100: #080808;
      --sumi-1000: #111111;
      --sumi-900: #1a1a1c;
      --sumi-800: #414143;
      --sumi-700: #626264;
      --sumi-600: #757578;
      --sumi-500: #949497;
      --sumi-400: #b4b4b7;
      --sumi-300: #d8d8db;
      --sumi-200: #e8e8eb;
      --sumi-100: #f1f1f4;
      --sumi-50: #f8f8fb;
      --forest-1200: #032213;
      --forest-1100: #08351f;
      --forest-1000: #0c472a;
      --forest-900: #115a36;
      --forest-800: #197a4b;
      --forest-700: #1a8b56;
      --forest-600: #259d63;
      --forest-500: #2cac6e;
      --forest-400: #51b883;
      --forest-300: #71c598;
      --forest-200: #9bd4b5;
      --forest-100: #c2e5d1;
      --forest-50: #e6f5ec;
      --wood-1200: #662e00;
      --wood-1100: #833b00;
      --wood-1000: #9c4600;
      --wood-900: #b65200;
      --wood-800: #c16800;
      --wood-700: #c87504;
      --wood-600: #cd820a;
      --wood-500: #d18d0f;
      --wood-400: #d69c2b;
      --wood-300: #dcac4d;
      --wood-200: #e5c47f;
      --wood-100: #efdbb1;
      --wood-50: #f8f1e0;
      --sun-1200: #640101;
      --sun-1100: #890101;
      --sun-1000: #af0000;
      --sun-900: #d50000;
      --sun-800: #ec0000;
      --sun-700: #fa1606;
      --sun-600: #ff220d;
      --sun-500: #ff4b36;
      --sun-400: #ff5838;
      --sun-300: #ff7b5c;
      --sun-200: #ffa28b;
      --sun-100: #ffc8b8;
      --sun-50: #ffe7e6;

      /* セマンティックカラー */
      --sc_txt_body: var(--sumi-900);
      --sc_txt_body__small: var(--sumi-1200);  /* 小さな文字用 */
      --sc_txt_desc: var(--sumi-700);
      --sc_txt_placeholder: var(--sumi-600);
      --sc_txt_onFill: var(--white);
      --sc_txt_link: var(--sea-800);
      --sc_txt_link__hover: var(--sea-900);
      --sc_txt_link__active: var(--sea-900);
      --sc_txt_link__visited: var(--sea-900);
      --sc_txt_alert: var(--sun-800);
      --sc_txt_disabled: var(--sumi-500);
      --sc_txt_icon: var(--sumi-900);
      --sc_txt_btn__primary: var(--white);
      --sc_txt_btn__secondary: var(--sea-800);
      --sc_txt_btn__tertiary: var(--sea-800);
      --sc_border_field: 1px solid var(--sumi-900);
      --sc_border_divider: 1px solid var(--sumi-300);
      --sc_border_focused: 2px solid var(--wood-600);
      --sc_border_selected: 2px solid var(--sea-800);
      --sc_border_disabled: 1px solid var(--sumi-300);
      --sc_border_alert: 2px solid var(--sun-800);
      --sc_bg_body__primary: var(--sumi-50);
      --sc_bg_body__secondary: var(--sumi-200);
      --sc_bg_body__tertiary: var(--sumi-100);
      --sc_bg_btn__primary: var(--sea-800);
      --sc_bg_btn__secondary: var(--sea-50);
      --sc_bg_disabled: var(--sumi-500);
      --sc_bg_alert: var(--sun-800);
      --sc_bg_shadow: var(--sumi-700);
    }

    /* セマンティックカラー（ダークモード時） */
    @media (prefers-color-scheme: dark) {
      :root {
        --sc_txt_body: var(--white);
        --sc_txt_body__small: var(--white);
        --sc_txt_desc: var(--sumi-400);
        --sc_txt_placeholder: var(--sumi-500);
        --sc_txt_onFill: var(--white);
        --sc_txt_link: var(--sea-300);
        --sc_txt_link__hover: var(--sea-200);
        --sc_txt_link__active: var(--sea-200);
        --sc_txt_link__visited: var(--sea-200);
        --sc_txt_alert: var(--sun-500);
        --sc_txt_disabled: var(--sumi-600);
        --sc_txt_icon: var(--white);
        --sc_txt_btn__primary: var(--white);
        --sc_txt_btn__secondary: var(--sea-300);
        --sc_txt_btn__tertiary: var(--sea-300);
        --sc_border_field: 1px solid var(--white);
        --sc_border_divider: 1px solid var(--sumi-700);
        --sc_border_focused: 2px solid var(--wood-600);
        --sc_border_selected: 2px solid var(--sea-300);
        --sc_border_disabled: 1px solid var(--sumi-300);
        --sc_border_alert: 2px solid var(--sun-500);
        --sc_bg_body__primary: var(--sumi-900);
        --sc_bg_body__secondary: var(--sumi-700);
        --sc_bg_body__tertiary: var(--sumi-800);
        --sc_bg_btn__primary: var(--sea-300);
        --sc_bg_btn__secondary: var(--sea-1200);
        --sc_bg_disabled: var(--sumi-600);
        --sc_bg_alert: var(--sun-500);
        --sc_bg_shadow: var(--white);
      }
    }

    *,
    ::after,
    ::before {
      box-sizing: border-box
    }

    h1, h2, h3, h4, h5, h6,
    li, th, td, div, span,
    code, pre, figcaption, caption,
    time, address, small,
    button, label, summary, output,
    p {
      color: var(--sc_txt_body);
    }

    body {
      background-color: var(--sc_bg_body__primary);
    }

    img {
      max-width: 100%;
    }

    svg {
      fill: currentcolor;
    }

    small {
      font-size: 0.8em;
    }


    /* フォーカス時のスタイル */
    *:focus-visible {
      outline: var(--sc_border_focused);
      outline-offset: 2px;
      animation-name: focusEffect;
      animation-duration: 0.3s;
    }

    @keyframes focusEffect {
      from {
        outline-width: 1px;
        outline-offset: 8px;
      }

      to {
        outline-width: 2px;
        outline-offset: 2px;
      }
    }

    .hp_srOnly {
      position: absolute !important;
      clip: rect(0 0 0 0) !important;
      clip-path: inset(100%) !important;
      width: 1px !important;
      height: 1px !important;
      margin: -1px !important;
      overflow: hidden !important;
      padding: 0 !important;
      border: 0 !important;
    }

    .b01ly_calc {
      --ly_btnArray:
        "nav main main main aside"
        "nav main main main aside"
        "nav main main main aside"
        "nav main main main aside";
      --ly_numArray:
        "b7 b8 b9"
        "b4 b5 b6"
        "b1 b2 b3"
        "b0 bd be";
      --num_hBox: 4;
      --num_wBox: 5;
      --size_btn: min(100dvw / var(--num_wBox), 100dvh / (var(--num_hBox) + 2));
      --flexFlow_nav: column nowrap;
      --gap: 0.1rem;
      display: flex;
      flex-flow: row wrap;
      align-items: center;
      justify-content: center;
      background-color: var(--sc_bg_body__primary);
      padding: 8px;
      margin: 0;
      width: 100dvw;
      height: 100dvh;
      border: var(--sc_border_field);
    }

    .is_leftMode .b01ly_calc {
      --ly_btnArray:
        "aside main main main nav"
        "aside main main main nav"
        "aside main main main nav"
        "aside main main main nav";
      --ly_numArray:
        "b7 b8 b9"
        "b4 b5 b6"
        "b1 b2 b3"
        "be bd b0";
    }

    @media (max-aspect-ratio: 5/6) {
      .b01ly_calc {
        --ly_btnArray:
          "nav  nav  nav  nav  "
          "main main main aside"
          "main main main aside"
          "main main main aside"
          "main main main aside";
        --num_hBox: 5;
        --num_wBox: 4;
        --flexFlow_nav: row nowrap;
        --width_nav: 100dvw;
      }

      .is_leftMode .b01ly_calc {
        --ly_btnArray:
          "nav  nav  nav  nav  "
          "aside main main main"
          "aside main main main"
          "aside main main main"
          "aside main main main";
      }
    }

    .b01ly_calc_header {
      --base_h: calc((100dvh - (var(--size_btn) * var(--num_hBox))) / 2);
      width: 98dvw;
      min-height: calc(var(--size_btn) * 2);
    }

    #b01js_label {
      width: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
      height: calc(var(--base_h) * 0.5);
      font-size: clamp(1rem, 5dvw, calc(var(--base_h) * 0.4));
      text-align: center;
      border: var(--sc_border_field);
      border-bottom: dotted;
    }

    #b01js_output {
      width: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
      height: var(--base_h);
      font-size: clamp(1rem, 5dvw, calc(var(--base_h) * 0.8));
      text-align: center;
      border: var(--sc_border_field);
      border-top: none;
      text-wrap: nowrap;
    }

    .b01ly_calc_btnArray {
      display: grid;
      gap: var(--gap);
      grid-template-areas: var(--ly_btnArray);
    }

    .b01ly_calc_nav {
      grid-area: nav;
      display: flex;
      gap: var(--gap);
      flex-flow: var(--flexFlow_nav);
    }

    .b01ly_calc_main {
      grid-area: main;
      display: grid;
      gap: var(--gap);
      grid-template-areas: var(--ly_numArray);
    }

    #b01js_1 { grid-area: b1; }
    #b01js_2 { grid-area: b2; }
    #b01js_3 { grid-area: b3; }
    #b01js_4 { grid-area: b4; }
    #b01js_5 { grid-area: b5; }
    #b01js_6 { grid-area: b6; }
    #b01js_7 { grid-area: b7; }
    #b01js_8 { grid-area: b8; }
    #b01js_9 { grid-area: b9; }
    #b01js_0 { grid-area: b0; }
    #b01js_DecimalPoint { grid-area: bd; }
    #b01js_Equal { grid-area: be; }

    .b01ly_calc_aside {
      grid-area: aside;
      display: flex;
      gap: var(--gap);
      flex-flow: column nowrap;
    }

    .b01el_calcBtn {
      appearance: none;
      height: calc(var(--size_btn) * 0.9);
      width: calc(var(--size_btn) * 0.9);
      margin: 0;
      padding: 0;
      font-size: calc(var(--size_btn) * 0.4);
      font-style: normal;
      font-weight: 700;
      line-height: 1;
      letter-spacing: 0;
      text-align: center;
      cursor: pointer;
      background: none;
      border-radius: 25%;
      border: 2px solid transparent;
    }

    .b01el_calcBtn.b01el_calcBtn__primary {
      color: var(--sc_txt_btn__primary);
      background-color: var(--sc_bg_btn__primary);
    }

    .b01el_calcBtn.b01el_calcBtn__primary:hover,
    .b01el_calcBtn.b01el_calcBtn__primary:active {
      filter: brightness(87%);
    }

    .b01el_calcBtn.b01el_calcBtn__primary:disabled {
      background-color: var(--sc_bg_disabled);
      position: relative;
    }

    .b01el_calcBtn.b01el_calcBtn__secondary {
      color: var(--sc_txt_btn__secondary);
      background-color: var(--sc_bg_btn__secondary);
      border-color: currentcolor;
    }

    .b01el_calcBtn.b01el_calcBtn__secondary:hover,
    .b01el_calcBtn.b01el_calcBtn__secondary:active {
      background-color: var(--sc_bg_btn__secondary);
    }

    .b01el_calcBtn.b01el_calcBtn__secondary:disabled {
      color: var(--sc_txt_disabled);
      position: relative;
    }

    .b01el_calcBtn.b01el_calcBtn__tertiary {
      color: var(--sc_txt_body);
      background-color: transparent;
      border: var(--sc_border_field);
    }

    .b01el_calcBtn.b01el_calcBtn__tertiary:hover,
    .b01el_calcBtn.b01el_calcBtn__tertiary:active {
      color: var(--sc_txt_link__active);
    }

    .b01el_calcBtn.b01el_calcBtn__tertiary:disabled {
      color: var(--sc_txt_disabled);
      position: relative;
    }

    .b01el_calcBtn.b01el_calcBtn__iconLabel {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: space-evenly;
    }

    .b01el_calcBtn.b01el_calcBtn__iconLabel>svg {
      color: var(--sc_txt_btn__secondary);
      height: calc(var(--size_btn) * 0.5);
      width: calc(var(--size_btn) * 0.5);
    }

    .b01el_calcBtn.b01el_calcBtn__iconLabel>span {
      font-size: calc(var(--size_btn) * 0.2);
      color: var(--sc_txt_btn__secondary);
    }

    .b01el_calcBtn:disabled::after {
      width: 50%;
      height: 50%;
      content: "";
      position: absolute;
      top: 50%;
      left: 50%;
      background: linear-gradient(-45deg,
          transparent 0%,
          transparent 49%,
          currentcolor 49%,
          currentcolor 51%,
          transparent 51%,
          transparent 100%);
    }
  </style>
</head>
<body class="b01ly_calc">
  <h1 class="hp_srOnly" id="b01_title">簡易電卓</h1>
  <section class="b01ly_calc_header" aria-label="ディスプレイ"><!-- Display -->
    <span id="b01js_label"></span>
    <output id="b01js_output"></output>
  </section>
  <div class="b01ly_calc_btnArray">
    <section class="b01ly_calc_nav" aria-label="機能"><!-- 4 buttons([AC,%,BackSpace,Copy]) -->
      <button type="button" class="b01el_calcBtn b01el_calcBtn__secondary b01el_calcBtn__iconLabel js_btnCloseDialog"
        aria-controls="b01js_dialog_calculator" aria-describedby="b01_title">
        <svg role="graphics-symbol img" aria-hidden="true" version="1.1" xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24" width="24" height="24">
          <path transform="rotate(45)"
            d="m4.9705-2.0012v4.0023h9.9989v9.9989h4.0023v-9.9989h9.9989v-4.0023h-9.9989v-9.9989h-4.0023v9.9989h-9.9989z" />
        </svg>
        <span>閉じる</span>
      </button>
      <button type="button" id="b01js_AllClear" class="b01el_calcBtn b01el_calcBtn__secondary b01el_calcBtn__iconLabel">
        <svg role="graphics-symbol img" aria-hidden="true" version="1.1" xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 48 48" width="48" height="48">
          <path
            d="m24 5.25a21 21 0 0 0-21 21 21 21 0 0 0 4.7812 13.219l-3.2812 3.2812h9v-9l-3.6016 3.6016a18 18 0 0 1-3.8984-11.102 18 18 0 0 1 18-18 18 18 0 0 1 18 18 18 18 0 0 1-5.3047 12.695l2.1172 2.1172a21 21 0 0 0 6.1875-14.812 21 21 0 0 0-21-21z"
            stroke-linecap="round" stroke-linejoin="round" />
        </svg>
        <span>全消去</span>
      </button>
      <button type="button" id="b01js_BackSpace"
        class="b01el_calcBtn b01el_calcBtn__secondary b01el_calcBtn__iconLabel">
        <svg role="graphics-symbol img" aria-hidden="true" version="1.1" xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 48 48" width="48" height="48">
          <path d="m45.75 22.5v3h-36l12 12-3 3-16.5-16.5 16.5-16.5 3 3-12 12z" />
        </svg>
        <span>1字消去</span>
      </button>
      <button type="button" id="b01js_Copy" class="b01el_calcBtn b01el_calcBtn__secondary b01el_calcBtn__iconLabel">
        <svg role="graphics-symbol img" aria-hidden="true" version="1.1" xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 48 48" width="48" height="48">
          <path
            d="m17.032 8.7h19.336c2.6216 0 4.7322 2.1106 4.7322 4.7322v26.536c0 2.6216-2.1106 4.7322-4.7322 4.7322h-19.336c-2.6216 0-4.7322-2.1106-4.7322-4.7322v-26.536c0-2.6216 2.1106-4.7322 4.7322-4.7322zm-5.4002-5.4c-2.6216 0-4.732 2.1104-4.732 4.732v26.536c0 2.6216 2.1104 4.732 4.732 4.732h0.66797v-25.868c0-2.6216 2.1104-4.732 4.732-4.732h18.668v-0.66797c0-2.6216-2.1104-4.732-4.732-4.732z"
            fill="none" stroke="currentColor" stroke-linejoin="round" stroke-width="2.4" />
        </svg>
        <span>コピー</span>
      </button>
    </section>
    <section class="b01ly_calc_main" aria-label="テンキー・実行"><!-- 12 buttons([0 - 9],[.] ,[=]) -->
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_9" data-calc="9">9</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_8" data-calc="8">8</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_7" data-calc="7">7</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_6" data-calc="6">6</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_5" data-calc="5">5</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_4" data-calc="4">4</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_3" data-calc="3">3</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_2" data-calc="2">2</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_1" data-calc="1">1</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_0" data-calc="0">0</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__tertiary" id="b01js_DecimalPoint" data-calc="."
        aria-label="小数点">.</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__primary" id="b01js_Equal" data-calc="="
        aria-label="計算実行">=</button>
    </section>
    <section class="b01ly_calc_aside" aria-label="四則演算"><!-- 4 buttons([+,-,*,/]) -->
      <button type="button" class="b01el_calcBtn b01el_calcBtn__primary" id="b01js_Add" data-calc="+"
        aria-label="足す">＋</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__primary" id="b01js_Subtract" data-calc="-"
        aria-label="引く">−</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__primary" id="b01js_Multiply" data-calc="*"
        aria-label="掛ける">×</button>
      <button type="button" class="b01el_calcBtn b01el_calcBtn__primary" id="b01js_Divide" data-calc="/"
        aria-label="割る">÷</button>
    </section>
  </div>
  <!-- /.b01ly_calc_btnArray -->
  <script></script>
</body>
</html>
```

:::

## 各セクション毎の詳細

ここでは 各 `section` ごとに、中身を簡単に解説していきます。

主に `htmlCalculator` クラスを操作するための `button` 要素を格納しているものが多いです。

:::details button要素についてのメモ

電卓ボタンは、（`span` や `div` などでなく）できれば `button` 要素で実装すべきです。

`button` 要素は、インタラクティブ要素として、下記の特徴を持っています。

- （Tabキーなどで）フォーカスできる
- スペースキーで `click` イベントを発火させられる
- `disabled` 属性の付与で、フォーカスを不可に、スクリーンリーダーの読み上げから除外できる

また その特徴（インタラクティブ要素であること）を強調するため、ブラウザに備わっているデフォルトスタイルが特徴的な表現になっています。自由なスタイリングには `appearance` による[固有のスタイル除去](https://developer.mozilla.org/ja/docs/Web/CSS/appearance)が必要になってきますが、アクセシビリティ上のメリットが大きいため、`button` 要素の使用をおすすめします。

:::

### ディスプレイ部

![ディスプレイ部のスクリーンショット](/images/books/howto-create-calculator/ui-display-01.png)

```html
<section aria-label="ディスプレイ">
  <span id="b01js_label"></span>
  <output id="b01js_output"></output>
</section>
```

ここは、`htmlCalculator` クラスで管理している「式」「ラベル」を表示出力させる要素です。

`output` 要素は 暗黙のロールとして `status` を持ち、多くのブラウザで `aria-live="polite"` 領域として認識されます。これによりスクリーンリーダーは、中身が変化した際（例：式への入力時 など）、読み上げに余裕があるタイミング（≒アイドル状態）で中身を読み上げてくれます。

https://developer.mozilla.org/ja/docs/Web/HTML/Element/output

### 各種ボタン

![ボタン配列のスクリーンショット](/images/books/howto-create-calculator/ui-button-01.png)

#### 機能ボタン部

```html
<section aria-label="機能">
  <button type="button">閉じる</button>
  <button type="button" id="b01js_AllClear">全消去</button>
  <button type="button" id="b01js_BackSpace">1字消去</button>
  <button type="button" id="b01js_Copy">コピー</button>
</section>
```

ここでは、`htmlCalculator` クラスの `reset()`, `back()` 処理を行うためのボタン要素を格納しています。  
（※閉じるボタンとコピーボタンもありますが、本書の趣旨から外れる部分のため説明を除外します）

#### 数値ボタン部

```html
<section aria-label="テンキー・実行">
  <button type="button" id="b01js_9" data-calc="9">9</button>
  <button type="button" id="b01js_8" data-calc="8">8</button>
  <button type="button" id="b01js_7" data-calc="7">7</button>
  <button type="button" id="b01js_6" data-calc="6">6</button>
  <button type="button" id="b01js_5" data-calc="5">5</button>
  <button type="button" id="b01js_4" data-calc="4">4</button>
  <button type="button" id="b01js_3" data-calc="3">3</button>
  <button type="button" id="b01js_2" data-calc="2">2</button>
  <button type="button" id="b01js_1" data-calc="1">1</button>
  <button type="button" id="b01js_0" data-calc="0">0</button>
  <button type="button" id="b01js_DecimalPoint" data-calc="." aria-label="小数点">.</button>
  <button type="button" id="b01js_Equal" data-calc="=" aria-label="計算実行">=</button>
</section>
```

ここでは 数値に加え、小数点やイコールボタンを格納しています。

後述する演算子ボタンを含め、式への入力ボタンについては `data-calc` という独自の属性をつけています。これは、ユーザーに表示する記号とコード上で扱う記号に差異があるためです。

例えば「掛ける」演算子は、ラベルとしては `✕` を使っていますが、コード上では `*` を使います。このような差異から表示する内容とは別に、コードで扱うための属性を設けています。ボタンが押された際は、ここに格納されている内容が `htmlCalculator` へ送られるというわけです。

::::details 「=」の aria-label を「イコール」にしていない理由

`img` の `alt` にも共通することですが、私は スクリーンリーダーを使用する方へのアクセシビリティを下記のように考えています。

- **文脈に基づく情報提供**：記号や画像情報は単なる外観でなく、前後の文脈を含めて認知されたものであるべきです
- **適切な情報量**：晴眼者と比較し、情報量は過剰でも不足でもいけません
- **晴眼者と同等の体験**：晴眼者と「同じ情報」より「同じ体験」を目指すべきです

その上で、`=` 記号は**その役割である**「計算実行」の方をラベルにすべきと考えました。

以下の格納されたメモについては、拙いながら私の考えを書いたものになります。

:::details 「外観でなく認知された情報」について

ここでは、ある記号と それを受け取った際の認識について考えてみます。

例えば、UI上に `💾`, `≡`, `⚙` マークがあったとします。この時 晴眼者は「フロッピー」「三本線」「歯車」とは認識しないでしょう。意識にも登らない段階で、「保存」「メニュー」「設定」と認識するはずです。

これは前後のコンテキスト（文化的に共有された認識、言ってしまえば「お約束」のようなもの？）によるものです。

- メニュー欄には通常「保存」や「設定」の項目がある
- 長年多くの Web サイトで3本線のマークをメニューとして扱ってきた経緯がある

画像や記号を説明する際には、こうしたコンテキストを踏まえるべきなのだと思います（最も、フロッピーは昨今使われなくなってきたので、「保存」を表す意味合いもだんだん薄れていくとは思いますが・・・）

:::

:::details 「過剰な情報量」について

ここでは、プログレスバーと [progress 要素](https://developer.mozilla.org/ja/docs/Web/HTML/Element/progress)による、同じ内容を繰り返してしまう例を挙げてみます。

状況として、視覚的なプログレスバーとして `div` を使ったローディングスピナーを用意し、別途スクリーンリーダー用に視覚から隠した `progress` 要素も用意したとします。

この場合、`div` 要素を AOM（Accessibility Object Model）上から消していない場合、同じ内容を2回読み上げることになります。同じ内容を複数回読み上げると、ユーザーは冗長に感じ、集中力が削がれてしまいます。情報が伝わらないケースよりは良いのでしょうが、削れる部分は削ったほうが より良くなります。

スクリーンリーダーの読み上げは、マウスやタップ操作と違い、コンテンツの順番通りに進むという制限が課されます。そのため ユーザーが必要とする情報へ素早くアクセスできるよう、心がけておくべきです。

:::

:::details 「晴眼者と同等の体験」について

ここではその例として、`select` 要素を挙げてみます。これはデフォルトのスタイルとしてボックスに `▼` が出ることが多いですが、これを除去して自作するケースが多いです。

その際、セレクトボックスにある `▼` は、晴眼者にとっては「展開可能」であることを示す視覚情報です。しかしスクリーンリーダーは `select` 要素を「展開可能なボックス」と認識しており、要素にたどり着いた段階でユーザーへその旨を伝えてくれます。

つまり両者は、（`▼` と `select` で）箇所は違うけど同じ体験ができるようになっているわけです。これを「同じ情報を提供しなければならない」という考えに嵌まると、`▼` を「下方向の三角」や「展開可能」とスクリーンリーダに読み上げさせなければなりません。けれどもこれは、恐らくユーザーにとって混乱を生じさせるのではないでしょうか？

同じ体験でにあることを重視するなら、`▼` 要素は AOM 上から消しておき、スクリーンリーダーが読み上げないようした方がいいでしょう。

:::

::::

#### 演算子ボタン部

```html
<section aria-label="四則演算">
  <button type="button" id="b01js_Add" data-calc="+" aria-label="足す">＋</button>
  <button type="button" id="b01js_Subtract" data-calc="-" aria-label="引く">−</button>
  <button type="button" id="b01js_Multiply" data-calc="*" aria-label="掛ける">×</button>
  <button type="button" id="b01js_Divide" data-calc="/" aria-label="割る">÷</button>
</section>
```

ここでは四則演算子のボタンを格納しています。スクリーンリーダーの読み上げを統一するため、記号それぞれに `aria-label` を割り振ってます。
