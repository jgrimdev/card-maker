<script>
  import { onMount } from "svelte";

  // --- 1. DATA (Tabulka) ---
  // Sloupce naší "databáze"
  let columns = ["name", "cost", "type", "description", "image_url"];

  // Defaultní data
  let rows = [
    {
      name: "Rytíř Světla",
      cost: "5",
      type: "Human",
      description: "Za každý útok se vyléčí o 2.",
      image_url: "https://placecats.com/300/300",
    },
    {
      name: "Temný Čaroděj",
      cost: "3",
      type: "Undead",
      description: "Zničí náhodnou kartu soupeře.",
      image_url: "https://placecats.com/300/301",
    },
    {
      name: "Drak Zkázy",
      cost: "10",
      type: "Dragon",
      description: "Tato karta nemůže být zničena.",
      image_url: "https://placecats.com/300/302",
    },
  ];
  let selectedRowIndex = 0;

  // --- 2. DESIGN (Vrstvy) ---
  // Defaultní šablona "Fantasy Karty"
  let elements = [
    // 1. Pozadí (Textura papíru)
    {
      id: 1,
      type: "box",
      x: 0,
      y: 0,
      w: 300,
      h: 420,
      color: "#f3e5ab",
      border: 0,
      borderRadius: 10,
      zIndex: 0,
    },

    // 2. Obrázek Monstra (Portrét)
    {
      id: 2,
      type: "image",
      x: 25,
      y: 50,
      w: 250,
      h: 200,
      src: "",
      bindColumn: "image_url",
      zIndex: 1,
      border: 2,
      borderColor: "#4a3b2a",
    },

    // 3. Rámeček kolem textu dole
    {
      id: 3,
      type: "box",
      x: 20,
      y: 270,
      w: 260,
      h: 130,
      color: "#ffffff",
      border: 1,
      borderColor: "#000",
      borderRadius: 5,
      zIndex: 2,
      opacity: 0.8,
    },

    // 4. Název (Hlavička)
    {
      id: 4,
      type: "text",
      x: 20,
      y: 15,
      label: "Název",
      bindColumn: "name",
      fontSize: 18,
      color: "#2c1a0b",
      bold: true,
      fontFamily: "serif",
      zIndex: 3,
    },

    // 5. Cena (Mana krystal vpravo nahoře)
    {
      id: 5,
      type: "box",
      x: 250,
      y: 10,
      w: 30,
      h: 30,
      color: "#00aaff",
      border: 2,
      borderColor: "#005588",
      borderRadius: 50,
      zIndex: 3,
    },
    {
      id: 6,
      type: "text",
      x: 258,
      y: 14,
      label: "0",
      bindColumn: "cost",
      fontSize: 16,
      color: "#ffffff",
      bold: true,
      zIndex: 4,
    },

    // 6. Popis (Lore)
    {
      id: 7,
      type: "text",
      x: 30,
      y: 280,
      w: 240,
      label: "Popis",
      bindColumn: "description",
      fontSize: 12,
      color: "#333333",
      bold: false,
      fontFamily: "sans-serif",
      zIndex: 3,
    },
  ];

  let selectedElementId = null;

  // --- LOGIKA DRAG & DROP ---
  let isDragging = false;
  let dragOffsetX = 0,
    dragOffsetY = 0;

  function startDrag(event, element) {
    selectedElementId = element.id;
    isDragging = true;
    dragOffsetX = event.clientX - element.x;
    dragOffsetY = event.clientY - element.y;
  }

  function onMouseMove(event) {
    if (!isDragging || selectedElementId === null) return;
    const el = elements.find((e) => e.id === selectedElementId);
    if (el) {
      // Snap to grid (volitelné, teď po 5px)
      let rawX = event.clientX - dragOffsetX;
      let rawY = event.clientY - dragOffsetY;
      el.x = Math.round(rawX / 5) * 5;
      el.y = Math.round(rawY / 5) * 5;
      elements = elements;
    }
  }

  function stopDrag() {
    isDragging = false;
  }

  // --- POMOCNÉ FUNKCE ---
  function getDisplayValue(element) {
    if (element.bindColumn)
      return rows[selectedRowIndex][element.bindColumn] || "";
    return element.label;
  }

  function getImageSrc(element) {
    if (element.bindColumn)
      return (
        rows[selectedRowIndex][element.bindColumn] ||
        "https://via.placeholder.com/150"
      );
    return element.src || "https://via.placeholder.com/150";
  }

  function addElement(type) {
    const newId = Math.max(...elements.map((e) => e.id)) + 1;
    let newEl = { id: newId, x: 50, y: 50, zIndex: 10 };

    if (type === "text") {
      newEl = {
        ...newEl,
        type: "text",
        label: "Nový Text",
        fontSize: 14,
        color: "#000",
      };
    } else if (type === "box") {
      newEl = {
        ...newEl,
        type: "box",
        w: 100,
        h: 50,
        color: "#cccccc",
        border: 1,
        borderColor: "#000",
      };
    } else if (type === "image") {
      newEl = {
        ...newEl,
        type: "image",
        w: 100,
        h: 100,
        src: "https://placecats.com/100/100",
      };
    }
    elements = [...elements, newEl];
    selectedElementId = newId;
  }

  function deleteSelected() {
    if (selectedElementId !== null) {
      elements = elements.filter((e) => e.id !== selectedElementId);
      selectedElementId = null;
    }
  }

  function moveLayer(direction) {
    if (selectedElementId === null) return;
    const el = elements.find((e) => e.id === selectedElementId);
    if (direction === "up") el.zIndex++;
    if (direction === "down") el.zIndex--;
    elements = elements;
  }

  // --- GENERÁTOR DAT ---
  function generateDummyData() {
    const adjectives = [
      "Ohnivý",
      "Ledový",
      "Zuřivý",
      "Starodávný",
      "Elektrický",
    ];
    const nouns = ["Drak", "Skřet", "Mág", "Golem", "Přízrak"];
    let newRows = [];
    for (let i = 0; i < 5; i++) {
      const name =
        adjectives[Math.floor(Math.random() * adjectives.length)] +
        " " +
        nouns[Math.floor(Math.random() * nouns.length)];
      newRows.push({
        name: name,
        cost: Math.floor(Math.random() * 10) + 1,
        type: "Monster",
        description: `Když tato karta zaútočí, udělí ${Math.floor(Math.random() * 5) + 1} poškození navíc.`,
        image_url: `https://placecats.com/300/30${i}`, // Placeholder kočky
      });
    }
    rows = newRows;
    selectedRowIndex = 0;
  }
</script>

<div
  class="h-[calc(100vh-64px)] flex flex-col font-sans select-none"
  on:mousemove={onMouseMove}
  on:mouseup={stopDrag}
  role="application"
>
  <div class="flex-grow flex overflow-hidden">
    <div
      class="w-16 bg-gray-800 flex flex-col items-center py-4 gap-4 z-20 shadow-xl"
    >
      <button
        on:click={() => addElement("text")}
        class="w-10 h-10 bg-white rounded hover:bg-blue-200 flex items-center justify-center font-bold"
        title="Přidat Text">T</button
      >
      <button
        on:click={() => addElement("box")}
        class="w-10 h-10 bg-white rounded hover:bg-blue-200 flex items-center justify-center"
        title="Přidat Obdélník">⬜</button
      >
      <button
        on:click={() => addElement("image")}
        class="w-10 h-10 bg-white rounded hover:bg-blue-200 flex items-center justify-center"
        title="Přidat Obrázek">🖼️</button
      >
      <div class="h-px w-8 bg-gray-600 my-2"></div>
      <button
        on:click={deleteSelected}
        class="w-10 h-10 bg-red-500 text-white rounded hover:bg-red-600 flex items-center justify-center font-bold"
        title="Smazat">🗑️</button
      >
    </div>

    <div
      class="flex-grow bg-gray-600 flex items-center justify-center relative overflow-hidden"
    >
      <div class="text-white absolute top-2 left-2 text-xs opacity-50">
        Editor V2.0
      </div>

      <div
        class="relative bg-white shadow-2xl overflow-hidden ring-4 ring-black/20"
        style="width: 300px; height: 420px;"
      >
        {#each elements as el (el.id)}
          <div
            on:mousedown={(e) => startDrag(e, el)}
            class="absolute cursor-move group hover:outline hover:outline-2 hover:outline-blue-500"
            style="
                            left: {el.x}px; 
                            top: {el.y}px; 
                            z-index: {el.zIndex};
                            {el.type !== 'text'
              ? `width:${el.w}px; height:${el.h}px;`
              : ''}
                        "
          >
            {#if el.type === "text"}
              <div
                style="
                                font-size: {el.fontSize}px;
                                color: {el.color};
                                font-weight: {el.bold ? 'bold' : 'normal'};
                                font-family: {el.fontFamily || 'sans-serif'};
                                width: {el.w ? el.w + 'px' : 'auto'};
                            "
              >
                {getDisplayValue(el)}
              </div>
            {:else if el.type === "box"}
              <div
                style="
                                width: 100%; height: 100%;
                                background: {el.color};
                                border: {el.border ||
                  0}px solid {el.borderColor || '#000'};
                                border-radius: {el.borderRadius || 0}px;
                                opacity: {el.opacity || 1};
                            "
              ></div>
            {:else if el.type === "image"}
              <img
                src={getImageSrc(el)}
                alt="card asset"
                class="w-full h-full object-cover"
                style="
                                    border: {el.border ||
                  0}px solid {el.borderColor || '#000'};
                                    border-radius: {el.borderRadius || 0}px;
                                  "
                draggable="false"
              />
            {/if}

            {#if selectedElementId === el.id}
              <div
                class="absolute -top-1 -right-1 w-2 h-2 bg-blue-500 rounded-full"
              ></div>
            {/if}
          </div>
        {/each}
      </div>
    </div>

    <div class="w-72 bg-white border-l flex flex-col overflow-y-auto">
      <div class="p-4 border-b bg-gray-50">
        <h2 class="font-bold text-gray-800">Vlastnosti</h2>
        <div class="flex gap-2 mt-2">
          <button
            on:click={() => moveLayer("up")}
            class="text-xs bg-gray-200 px-2 py-1 rounded"
            >Vrstvu Nahoru ▲</button
          >
          <button
            on:click={() => moveLayer("down")}
            class="text-xs bg-gray-200 px-2 py-1 rounded">Vrstvu Dolů ▼</button
          >
        </div>
      </div>

      <div class="p-4 space-y-4">
        {#if selectedElementId !== null}
          {@const activeEl = elements.find((e) => e.id === selectedElementId)}

          <div class="grid grid-cols-2 gap-2">
            <div>
              <label class="text-xs font-bold text-gray-500">X</label>
              <input
                type="number"
                bind:value={activeEl.x}
                class="w-full border p-1 rounded"
              />
            </div>
            <div>
              <label class="text-xs font-bold text-gray-500">Y</label>
              <input
                type="number"
                bind:value={activeEl.y}
                class="w-full border p-1 rounded"
              />
            </div>
            <div>
              <label class="text-xs font-bold text-gray-500">Šířka</label>
              <input
                type="number"
                bind:value={activeEl.w}
                class="w-full border p-1 rounded"
              />
            </div>
            <div>
              <label class="text-xs font-bold text-gray-500">Výška</label>
              <input
                type="number"
                bind:value={activeEl.h}
                class="w-full border p-1 rounded"
              />
            </div>
          </div>

          <hr />

          {#if activeEl.type === "text"}
            <div>
              <label class="text-xs font-bold text-gray-500 uppercase"
                >Data Sloupec</label
              >
              <select
                bind:value={activeEl.bindColumn}
                class="w-full border p-2 rounded bg-blue-50"
              >
                <option value="">(Statický text)</option>
                {#each columns as col}
                  <option value={col}>{col}</option>
                {/each}
              </select>
            </div>
            {#if !activeEl.bindColumn}
              <input
                type="text"
                bind:value={activeEl.label}
                class="w-full border p-2 rounded"
              />
            {/if}
            <div class="flex gap-2">
              <input
                type="number"
                bind:value={activeEl.fontSize}
                class="w-16 border p-1 rounded"
              />
              px
              <input
                type="color"
                bind:value={activeEl.color}
                class="w-8 h-8 cursor-pointer"
              />
            </div>
            <label class="flex items-center gap-2">
              <input type="checkbox" bind:checked={activeEl.bold} /> Bold
            </label>
          {:else}
            <div>
              <label class="text-xs font-bold text-gray-500"
                >Barva / Pozadí</label
              >
              {#if activeEl.type === "box"}
                <input
                  type="color"
                  bind:value={activeEl.color}
                  class="w-full h-8 cursor-pointer"
                />
              {/if}
            </div>

            {#if activeEl.type === "image"}
              <div>
                <label class="text-xs font-bold text-gray-500 uppercase"
                  >Zdroj Obrázku</label
                >
                <select
                  bind:value={activeEl.bindColumn}
                  class="w-full border p-2 rounded bg-blue-50 mb-1"
                >
                  <option value="">(URL Adresa)</option>
                  {#each columns as col}
                    <option value={col}>{col}</option>
                  {/each}
                </select>
                {#if !activeEl.bindColumn}
                  <input
                    type="text"
                    bind:value={activeEl.src}
                    class="w-full border p-1 text-xs"
                    placeholder="https://..."
                  />
                {/if}
              </div>
            {/if}

            <div class="grid grid-cols-2 gap-2">
              <div>
                <label class="text-xs font-bold text-gray-500"
                  >Rámeček (px)</label
                >
                <input
                  type="number"
                  bind:value={activeEl.border}
                  class="w-full border p-1 rounded"
                />
              </div>
              <div>
                <label class="text-xs font-bold text-gray-500">Barva rámu</label
                >
                <input
                  type="color"
                  bind:value={activeEl.borderColor}
                  class="w-full h-8"
                />
              </div>
              <div>
                <label class="text-xs font-bold text-gray-500">Radius</label>
                <input
                  type="number"
                  bind:value={activeEl.borderRadius}
                  class="w-full border p-1 rounded"
                />
              </div>
            </div>
          {/if}
        {:else}
          <div class="text-center text-gray-400 mt-10">
            Klikni na prvek na kartě pro úpravu.
          </div>
        {/if}
      </div>
    </div>
  </div>

  <div class="h-1/3 bg-gray-100 border-t flex flex-col">
    <div
      class="bg-gray-200 p-2 flex justify-between items-center border-b px-4"
    >
      <div class="font-bold text-sm text-gray-700">Data Kartiček</div>
      <button
        on:click={generateDummyData}
        class="bg-white border px-3 py-1 rounded text-xs hover:bg-blue-50 shadow-sm"
      >
        🎲 Vygenerovat Náhodná Data
      </button>
    </div>

    <div class="overflow-auto flex-grow p-4">
      <table class="w-full bg-white border collapse text-sm">
        <thead>
          <tr class="bg-gray-800 text-white">
            <th class="p-2 w-10">#</th>
            {#each columns as col}
              <th class="p-2 border border-gray-600 text-left capitalize"
                >{col}</th
              >
            {/each}
          </tr>
        </thead>
        <tbody>
          {#each rows as row, i}
            <tr
              class="cursor-pointer hover:bg-blue-100 {selectedRowIndex === i
                ? 'bg-blue-200'
                : ''}"
              on:click={() => (selectedRowIndex = i)}
            >
              <td class="p-2 border text-center text-gray-400">{i + 1}</td>
              {#each columns as col}
                <td class="p-0 border relative min-w-[100px]">
                  <input
                    type="text"
                    bind:value={row[col]}
                    class="w-full h-full p-2 outline-none bg-transparent focus:bg-white focus:ring-2 focus:ring-blue-500"
                  />
                </td>
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>
</div>
