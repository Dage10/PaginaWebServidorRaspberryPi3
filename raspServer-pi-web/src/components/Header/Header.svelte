<script>
  export let title = "El meu lloc web";
  import './Header.css';
</script>

<header class="flex items-center justify-between bg-[#454430] p-4 text-white">
  <nav class="flex space-x-4">
    <a href="/" class="hover:underline">Inici</a>
    <a href="/scoreboard" class="hover:underline">Scoreboard</a>
  </nav>

  <h1 class="font-bold text-xl">{title}</h1>
</header>