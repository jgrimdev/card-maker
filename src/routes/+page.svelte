<script>
  // Tady je logika (JavaScript)
  // Definujeme proměnné pro naši kartu
  let cardName = "Ohnivý Drak";
  let cardCost = 5;
  let cardText = "Tato karta zničí vše v okolí.";
  let cardColor = "#ffcccc"; // Světle červená
</script>

<h1>Welcome to SvelteKit</h1>
<p>
  Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the
  documentation
</p>

<div class="min-h-screen bg-gray-100 p-8 font-sans">
  <h1 class="text-3xl font-bold mb-8 text-center text-gray-800">
    Můj Card Editor
  </h1>

  <div class="max-w-6xl mx-auto grid grid-cols-1 md:grid-cols-2 gap-10">
    <div class="bg-white p-6 rounded-xl shadow-lg">
      <h2 class="text-xl font-bold mb-4 border-b pb-2">Data Karty</h2>

      <div class="mb-4">
        <label class="block text-sm font-bold mb-1">Název karty</label>
        <input
          type="text"
          bind:value={cardName}
          class="w-full border p-2 rounded focus:ring-2 focus:ring-blue-500 outline-none"
        />
      </div>

      <div class="flex gap-4 mb-4">
        <div class="w-1/2">
          <label class="block text-sm font-bold mb-1">Cena</label>
          <input
            type="number"
            bind:value={cardCost}
            class="w-full border p-2 rounded"
          />
        </div>
        <div class="w-1/2">
          <label class="block text-sm font-bold mb-1">Barva pozadí</label>
          <input
            type="color"
            bind:value={cardColor}
            class="w-full h-10 cursor-pointer"
          />
        </div>
      </div>

      <div class="mb-4">
        <label class="block text-sm font-bold mb-1">Efekt karty</label>
        <textarea
          bind:value={cardText}
          rows="4"
          class="w-full border p-2 rounded focus:ring-2 focus:ring-blue-500 outline-none"
        ></textarea>
      </div>
    </div>

    <div
      class="flex items-center justify-center bg-gray-200 rounded-xl border-2 border-dashed border-gray-400 p-10"
    >
      <div
        class="w-[300px] h-[420px] rounded-lg shadow-2xl flex flex-col relative overflow-hidden border-8 border-gray-800"
        style="background-color: {cardColor};"
      >
        <div
          class="bg-white/90 p-3 flex justify-between items-center border-b-2 border-black"
        >
          <span class="font-bold text-lg truncate">{cardName}</span>
          <span
            class="bg-blue-600 text-white w-8 h-8 rounded-full flex items-center justify-center font-bold shadow"
          >
            {cardCost}
          </span>
        </div>

        <div
          class="h-40 bg-gray-300 flex items-center justify-center border-b-2 border-black m-2 border-2"
        >
          <span class="text-gray-500 italic">Obrázek</span>
        </div>

        <div
          class="p-4 flex-grow bg-white/80 m-2 mt-0 border-2 border-black rounded"
        >
          <p class="text-sm text-gray-900 leading-relaxed">{cardText}</p>
        </div>

        <div class="p-1 text-center text-xs text-black/50">© 2025 My Game</div>
      </div>
    </div>
  </div>
</div>
