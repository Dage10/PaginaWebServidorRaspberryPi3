<script>
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
	import Header from '../components/Header/Header.svelte';
	import Footer from '../components/Footer/Footer.svelte';
	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<Header title="LaserTag" />

<main>
  <slot />
</main>

<Footer/>


