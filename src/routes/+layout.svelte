<!-- <script>import "../app.css";
import favicon from '$lib/assets/favicon.svg';

let { children } = $props();</script>

<svelte:head>
	<link rel="icon" href="{favicon}" />
</svelte:head>

{@render children()} -->
<script>
  import "../app.css"; // Načte styly pro celou appku
</script>

<div class="min-h-screen flex flex-col font-sans bg-gray-50">
  <nav class="bg-gray-900 text-white p-4 shadow-md print:hidden">
    <div class="max-w-6xl mx-auto flex justify-between items-center">
      <div class="text-xl font-bold tracking-wider">OpenDeck</div>
      <div class="flex gap-4">

		<a href="/" class="hover:text-blue-400 transition">Home</a>
<a href="/designer" class="hover:text-blue-400 transition text-yellow-300">Designer (Nový)</a> <a href="/print" class="hover:text-blue-400 transition">Tisk & Layout</a>
      </div>
    </div>
  </nav>

  <main class="flex-grow">
    <slot />
  </main>
</div>
