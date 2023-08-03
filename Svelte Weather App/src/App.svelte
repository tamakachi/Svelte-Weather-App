<script>
// Developed by Kent Van Heerden


import Weather from "./lib/Weather.svelte";
import { onMount } from "svelte";


function zoomOut() {
  var zoom = window.document.body.style.transform;
  if (zoom === undefined) {
    zoom = "scale(1)";
  }
  window.document.body.style.transform = "scale(0)";
}

onMount(() => {
    zoomOut()
});

</script>

<main>

 <Weather />


</main>

<style>
 
</style>
