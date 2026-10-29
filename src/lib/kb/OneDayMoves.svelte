<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  // The desk's own list: reports read as a one-day move. Counted where they're filed, footnoted here.
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
</script>

<p class="sum"><strong>{moved.length}</strong> reports the desk reads as a one-day move</p>

<table>
  <thead><tr><th>id</th><th>tenant</th><th>via</th><th>received</th><th>what it says</th></tr></thead>
  <tbody>
    {#each moved as r (r.id)}
      <tr>
        <td><code>{r.id}</code></td>
        <td>{r.tenant} ({tenantOf(r)?.locale})</td>
        <td>{r.channel}</td>
        <td>{r.receivedAt.slice(0, 10)}</td>
        <td>“{r.body}”</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
