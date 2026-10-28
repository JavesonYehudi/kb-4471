<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));

  // The one-day move is its own problem: counted here, not under the tenant's cause.
  const movedIds = new Set(notes.oneDayMove);
  const moved = $derived(merged.filter((r) => movedIds.has(r.id)));
  const byCause = (c: string) => merged.filter((r) => causeOf(r, tenantOf(r)) === c && !movedIds.has(r.id));

  const span = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return days.length ? `${days[0]} → ${days[days.length - 1]}` : '—';
  };
  const tenantsOf = (rs: Report[]) => new Set(rs.map((r) => r.tenant)).size;

  const rows = $derived([
    {
      key: 'parser',
      what: 'month and day flip: the parser reads month first',
      covers: byCause('parser'),
      filed: `the filed cause of #${ticket} — its group of ${byCause('parser').length + moved.length} includes the ones below`,
      ids: '',
    },
    {
      key: 'hint',
      what: 'month and day flip: the booking-v3 form prints month first',
      covers: byCause('hint'),
      filed: `merged into #${ticket} — no ticket of its own`,
      ids: '',
    },
    {
      key: 'one-day-move',
      what: 'lands one day off, not a month off',
      covers: moved,
      filed: 'footnoted under the tenant’s cause — never filed',
      ids: moved.map((r) => r.id).join(' '),
    },
  ]);
</script>

<table>
  <thead><tr><th>problem</th><th>reports it covers</th><th>tenants</th><th>seen</th><th>where filed</th></tr></thead>
  <tbody>
    {#each rows as row (row.key)}
      <tr>
        <td><code>{row.key}</code><br><span class="what">{row.what}</span></td>
        <td>{row.covers.length}{row.ids ? ` · ${row.ids}` : ''}</td>
        <td>{tenantsOf(row.covers)}</td>
        <td>{span(row.covers)}</td>
        <td class="filed">{row.filed}</td>
      </tr>
    {/each}
  </tbody>
</table>
<p class="sum">{rows[0].covers.length} + {rows[1].covers.length} + {rows[2].covers.length} = {merged.length} — one report, one problem</p>

<style>
  .what { color: var(--muted); font-size: 0.85em; }
  .filed { color: var(--muted); font-size: 0.85em; }
  .sum { font-size: 1.1rem; }
</style>
