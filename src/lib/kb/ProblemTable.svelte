<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rest = $derived(merged.filter((r) => !notes.oneDayMove.includes(r.id)));
  const span = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return { firstSeen: days[0], lastSeen: days[days.length - 1], tenants: new Set(rs.map((r) => r.tenant)).size };
  };
  const groups = $derived([
    ...groupByCause(rest, tenantOf).map((g) => ({ cause: g.cause as string, reports: g.reports, ...span(g.reports) })),
    ...(moved.length ? [{ cause: 'one-day', reports: moved, ...span(moved) }] : []),
  ]);
  const total = $derived(groups.reduce((n, g) => n + g.reports.length, 0));
</script>

<p class="sum"><strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{groups.length}</strong> problems · counted <strong>{total}</strong></p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr><td><code>{g.cause}</code></td><td>{g.reports.length}</td><td>{g.tenants}</td><td>{g.firstSeen} → {g.lastSeen}</td></tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
