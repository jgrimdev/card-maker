<script>
  import { onMount } from "svelte";

  // --- KONFIGURACE (Odpovídá tvému Pythonu) ---
  // Pro obrazovku používáme menší DPI, aby se to vešlo, při tisku/exportu by se to násobilo
  const SCREEN_DPI = 96;
  const MM_TO_PX = SCREEN_DPI / 25.4;

  const PAPER_SIZES = {
    A3: { w: 297, h: 420 },
    A4: { w: 210, h: 297 },
    A5: { w: 148, h: 210 },
  };

  // --- STAV APLIKACE (Proměnné) ---
  let canvas; // Odkaz na <canvas> element
  let ctx; // Kreslící kontext

  // Nastavení (Bindováno na inputy)
  let settings = {
    pageSize: "A4",
    cardWidth: 63,
    cardHeight: 88,
    gap: 3,
    bleed: 0,
    showMarks: true,
    mirror: false,
  };

  // Data (Simulace DataFrame)
  let data = [];
  let generatedPages = []; // Pole polí (co stránka, to seznam karet)
  let currentPageIndex = 0;
  let totalPages = 0;

  // Simulace načtení dat (místo load_file)
  function loadDemoData() {
    // Vytvoříme dummy data jako kdybychom načetli CSV
    data = [];
    for (let i = 1; i <= 15; i++) {
      data.push({
        file: `karta_${i}.png`,
        count: 1, // Každá karta 1x, zkus si změnit
        color: `hsl(${Math.random() * 360}, 70%, 80%)`, // Náhodná barva pro vizualizaci
        name: `Karta ${i}`,
      });
    }
    // Přidáme jednu kartu víckrát (jako v Python logice)
    data.push({
      file: "special.png",
      count: 4,
      color: "#ffcccb",
      name: "Special (4ks)",
    });

    calculateAndRender();
  }

  // --- JÁDRO: VÝPOČET ROZLOŽENÍ (Odpovídá generate_sheets) ---
  function calculateLayout() {
    const paper = PAPER_SIZES[settings.pageSize];

    // Převod mm na pixely
    const sw = paper.w * MM_TO_PX;
    const sh = paper.h * MM_TO_PX;
    const cw = settings.cardWidth * MM_TO_PX;
    const ch = settings.cardHeight * MM_TO_PX;
    const gap = settings.gap * MM_TO_PX;
    const bleed = settings.bleed * MM_TO_PX;

    // Výpočet mřížky
    const cols = Math.floor(sw / (cw + gap));
    const rows = Math.floor(sh / (ch + gap));

    // Centrování (Margins)
    const mx = (sw - cols * (cw + gap) + gap) / 2;
    const my = (sh - rows * (ch + gap) + gap) / 2;

    return { sw, sh, cw, ch, gap, bleed, cols, rows, mx, my };
  }

  // --- JÁDRO: GENERATOR STRÁNEK ---
  function generatePagesData() {
    const layout = calculateLayout();
    let pages = [];
    let currentPageCards = [];

    let col = 0;
    let row = 0;

    // Iterace přes data (jako df.iterrows)
    data.forEach((item) => {
      const count = item.count || 0;

      for (let i = 0; i < count; i++) {
        // Pozice
        const finalCol = settings.mirror ? layout.cols - 1 - col : col;
        const x = layout.mx + finalCol * (layout.cw + layout.gap);
        const y = layout.my + row * (layout.ch + layout.gap);

        // Přidat kartu do seznamu pro vykreslení
        currentPageCards.push({
          x,
          y,
          w: layout.cw,
          h: layout.ch,
          bleed: layout.bleed,
          color: item.color,
          name: item.name,
        });

        // Posun v mřížce
        col++;
        if (col >= layout.cols) {
          col = 0;
          row++;
          if (row >= layout.rows) {
            // Stránka je plná, uložíme a jedeme dál
            pages.push(currentPageCards);
            currentPageCards = [];
            row = 0;
          }
        }
      }
    });

    // Přidat zbytek poslední stránky
    if (currentPageCards.length > 0) {
      pages.push(currentPageCards);
    }

    generatedPages = pages;
    totalPages = pages.length;
    if (currentPageIndex >= totalPages) currentPageIndex = 0;
  }

  // --- VYKRESLOVÁNÍ (Odpovídá draw a render_card_image) ---
  function drawCanvas() {
    if (!canvas) return;
    ctx = canvas.getContext("2d");
    const layout = calculateLayout();

    // 1. Nastavit velikost Canvasu (Papír)
    canvas.width = layout.sw;
    canvas.height = layout.sh;

    // Vyčistit - bílé pozadí
    ctx.fillStyle = "white";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Pokud nemáme stránky, končíme
    if (generatedPages.length === 0) return;

    const cardsOnPage = generatedPages[currentPageIndex];

    // 2. Vykreslit karty
    cardsOnPage.forEach((card) => {
      // A) Spadávka (Bleed) - jen vizuálně rámeček
      if (card.bleed > 0) {
        ctx.strokeStyle = "red";
        ctx.lineWidth = 1;
        ctx.strokeRect(
          card.x - card.bleed,
          card.y - card.bleed,
          card.w + card.bleed * 2,
          card.h + card.bleed * 2
        );
      }

      // B) Karta samotná (Zatím barevný obdélník místo obrázku)
      ctx.fillStyle = card.color;
      ctx.fillRect(card.x, card.y, card.w, card.h);

      // Text uvnitř karty (název)
      ctx.fillStyle = "black";
      ctx.font = "12px Arial";
      ctx.textAlign = "center";
      ctx.fillText(card.name, card.x + card.w / 2, card.y + card.h / 2);

      // C) Ořezové značky
      if (settings.showMarks) {
        drawCropMarks(ctx, card.x, card.y, card.w, card.h);
      }
    });
  }

  function drawCropMarks(ctx, x, y, w, h) {
    const len = 10; // délka značky
    ctx.strokeStyle = "black";
    ctx.lineWidth = 1;
    ctx.beginPath();

    // Levý horní
    ctx.moveTo(x, y);
    ctx.lineTo(x - len, y);
    ctx.moveTo(x, y);
    ctx.lineTo(x, y - len);
    // Pravý horní
    ctx.moveTo(x + w, y);
    ctx.lineTo(x + w + len, y);
    ctx.moveTo(x + w, y);
    ctx.lineTo(x + w, y - len);
    // Levý dolní
    ctx.moveTo(x, y + h);
    ctx.lineTo(x - len, y + h);
    ctx.moveTo(x, y + h);
    ctx.lineTo(x, y + h + len);
    // Pravý dolní
    ctx.moveTo(x + w, y + h);
    ctx.lineTo(x + w + len, y + h);
    ctx.moveTo(x + w, y + h);
    ctx.lineTo(x + w, y + h + len);

    ctx.stroke();
  }

  // Hlavní funkce spouštěná tlačítkem nebo změnou
  function calculateAndRender() {
    generatePagesData();
    // Potřebujeme počkat, až se aktualizuje DOM (Svelte tick), ale zde stačí requestAnimationFrame
    requestAnimationFrame(drawCanvas);
  }

  // Navigace
  function nextPage() {
    if (currentPageIndex < totalPages - 1) {
      currentPageIndex++;
      drawCanvas();
    }
  }
  function prevPage() {
    if (currentPageIndex > 0) {
      currentPageIndex--;
      drawCanvas();
    }
  }

  // Tisk pomocí prohlížeče
  function printCanvas() {
    const dataUrl = canvas.toDataURL();
    const windowContent = `<!DOCTYPE html>
            <html>
            <head><title>Tisk</title></head>
            <body style="margin:0; padding:0; display:flex; justify-content:center; align-items:center;">
            <img src="${dataUrl}" style="width:100%; height:auto;">
            </body>
            </html>`;
    const printWin = window.open("", "", "width=800,height=600");
    printWin.document.open();
    printWin.document.write(windowContent);
    printWin.document.close();
    printWin.focus();
    setTimeout(() => {
      printWin.print();
      printWin.close();
    }, 500);
  }

  // Inicializace
  onMount(() => {
    loadDemoData(); // Načte defaultní data při startu
  });
</script>

<div class="flex h-[calc(100vh-64px)] overflow-hidden font-sans text-sm">
  <div
    class="w-80 bg-gray-100 border-r border-gray-300 flex flex-col shadow-lg z-10 shrink-0"
  >
    <div class="flex-grow overflow-y-auto p-4 space-y-6">
      <div>
        <h3 class="font-bold bg-gray-300 px-2 py-1 mb-2">
          1. Nastavení papíru
        </h3>
        <div class="flex justify-between items-center mb-2">
          <label>Formát:</label>
          <select
            bind:value={settings.pageSize}
            on:change={calculateAndRender}
            class="border p-1 rounded w-24"
          >
            {#each Object.keys(PAPER_SIZES) as size}
              <option value={size}>{size}</option>
            {/each}
          </select>
        </div>

        <div class="grid grid-cols-2 gap-2 mb-2">
          <div>
            <label class="text-xs text-gray-500">Šířka (mm)</label>
            <input
              type="number"
              bind:value={settings.cardWidth}
              on:input={calculateAndRender}
              class="w-full border p-1"
            />
          </div>
          <div>
            <label class="text-xs text-gray-500">Výška (mm)</label>
            <input
              type="number"
              bind:value={settings.cardHeight}
              on:input={calculateAndRender}
              class="w-full border p-1"
            />
          </div>
          <div>
            <label class="text-xs text-gray-500">Mezera (mm)</label>
            <input
              type="number"
              bind:value={settings.gap}
              on:input={calculateAndRender}
              class="w-full border p-1"
            />
          </div>
          <div>
            <label class="text-xs text-gray-500">Spadávka (mm)</label>
            <input
              type="number"
              bind:value={settings.bleed}
              on:input={calculateAndRender}
              class="w-full border p-1"
            />
          </div>
        </div>

        <div class="space-y-1">
          <label class="flex items-center gap-2 cursor-pointer">
            <input
              type="checkbox"
              bind:checked={settings.showMarks}
              on:change={calculateAndRender}
            />
            <span>Ořezové značky</span>
          </label>
          <label class="flex items-center gap-2 cursor-pointer text-blue-600">
            <input
              type="checkbox"
              bind:checked={settings.mirror}
              on:change={calculateAndRender}
            />
            <span>Zrcadlit (Zadní str.)</span>
          </label>
        </div>
      </div>

      <hr class="border-gray-300" />

      <div>
        <h3 class="font-bold bg-gray-300 px-2 py-1 mb-2">2. Data a Zdroje</h3>
        <button
          on:click={loadDemoData}
          class="w-full bg-gray-200 border py-1 mb-2 hover:bg-gray-300"
        >
          🔄 Reset / Načíst Demo Data
        </button>

        <div class="mb-2">
          <label class="text-xs text-gray-500 block">Soubor dat (CSV):</label>
          <input
            type="text"
            value="demo_data.csv"
            disabled
            class="w-full bg-white border p-1 text-gray-400 cursor-not-allowed"
          />
        </div>
        <div class="mb-2">
          <label class="text-xs text-gray-500 block">Složka obrázků:</label>
          <div class="flex">
            <input
              type="text"
              value="/images/"
              disabled
              class="w-full bg-white border p-1 text-gray-400 cursor-not-allowed rounded-l"
            />
            <button class="px-2 bg-gray-200 border border-l-0 rounded-r"
              >...</button
            >
          </div>
        </div>
      </div>

      <hr class="border-gray-300" />

      <div>
        <h3 class="font-bold bg-gray-300 px-2 py-1 mb-2">3. Prohlížení</h3>
        <div
          class="flex justify-center items-center gap-4 bg-white p-2 border rounded"
        >
          <button
            on:click={prevPage}
            class="px-3 py-1 bg-gray-200 rounded hover:bg-gray-300 font-bold"
            >&lt;</button
          >
          <span class="font-mono"
            >{currentPageIndex + 1} / {totalPages || 1}</span
          >
          <button
            on:click={nextPage}
            class="px-3 py-1 bg-gray-200 rounded hover:bg-gray-300 font-bold"
            >&gt;</button
          >
        </div>
      </div>
    </div>

    <div class="bg-gray-200 p-4 border-t border-gray-400 space-y-2">
      <button
        on:click={calculateAndRender}
        class="w-full bg-blue-200 hover:bg-blue-300 text-blue-900 py-2 rounded font-bold border border-blue-400"
      >
        Generovat Náhled
      </button>
      <button
        on:click={printCanvas}
        class="w-full bg-green-200 hover:bg-green-300 text-green-900 py-2 rounded font-bold border border-green-400"
      >
        Tisk / Uložit PDF
      </button>
    </div>
  </div>

  <div
    class="flex-1 bg-gray-600 overflow-auto flex items-center justify-center p-8 relative"
  >
    <canvas
      bind:this={canvas}
      class="shadow-2xl bg-white transition-all duration-300"
      style="max-width: 100%; max-height: 100%;"
    ></canvas>
  </div>
</div>
