<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  // The desk read these reports as a one-day move, not a day/month flip: a third problem, filed apart from the tenant's cause.
  const oneDay = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rest = $derived(merged.filter((r) => !notes.oneDayMove.includes(r.id)));
  const span = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return `${days[0]} → ${days[days.length - 1]}`;
  };

  const rows = $derived([
    ...groupByCause(rest, tenantOf).map((g) => ({
      problem: g.cause as string,
      reports: g.reports.length,
      tenants: g.tenants,
      seen: `${g.firstSeen} → ${g.lastSeen}`,
    })),
    ...(oneDay.length
      ? [{ problem: 'one-day move', reports: oneDay.length, tenants: new Set(oneDay.map((r) => r.tenant)).size, seen: span(oneDay) }]
      : []),
  ]);
  const total = $derived(rows.reduce((n, r) => n + r.reports, 0));
</script>

<p class="sum"><strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{rows.length}</strong> problems · counted <strong>{total}</strong></p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th></tr>
  </thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr><td><code>{r.problem}</code></td><td>{r.reports}</td><td>{r.tenants}</td><td>{r.seen}</td></tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
