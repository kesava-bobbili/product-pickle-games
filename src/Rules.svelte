<script>
  import { Check } from "lucide-svelte";

  export let title;
  export let rules = [];
</script>

<section class="rules">
  <h3>{title}</h3>
  <div>
    {#each rules as rule}
      <p><Check size={15} /> {rule}</p>
    {/each}
  </div>
</section>
