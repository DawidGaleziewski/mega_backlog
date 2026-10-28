<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem merged into the ticket. The desk's one-day-move note takes a report out of
  // its tenant's cause: a date one day off is not a day/month swap, whatever the tenant writes.
  type Problem = 'parser' | 'hint' | 'oneDayMove' | 'unplaced';
  const LABEL: Record<Problem, string> = {
    parser: 'day and month swap: the parser reads month first',
    hint: 'day and month swap: the booking-v3 hint says MM/DD',
    oneDayMove: 'the date moves by one day',
    unplaced: 'no cause on file',
  };

  let { ticket, issues }: { ticket: number; issues: Partial<Record<Problem, string>> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const problemOf = (r: Report): Problem =>
    notes.oneDayMove.includes(r.id) ? 'oneDayMove' : (causeOf(r, tenantOf(r)) ?? 'unplaced');

  const rows = $derived.by(() => {
    const by = new Map<Problem, Report[]>();
    for (const r of merged) by.set(problemOf(r), [...(by.get(problemOf(r)) ?? []), r]);
    return [...by.entries()]
      .map(([problem, rs]) => {
        const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
        const channels = [...new Set(rs.map((r) => r.channel))].sort().join(', ');
        return { problem, n: rs.length, tenants: new Set(rs.map((r) => r.tenant)).size, channels, first: days[0], last: days[days.length - 1] };
      })
      .sort((a, b) => b.n - a.n);
  });
  const issueNo = (url?: string) => url?.match(/(\d+)$/)?.[1];
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong> problems · <strong>{rows.filter((r) => issues[r.problem]).length}</strong> with a ticket of their own
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>channels</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr class:open={!issues[r.problem]}>
        <td>{LABEL[r.problem]}</td>
        <td>{r.n}</td>
        <td>{r.tenants}</td>
        <td>{r.channels}</td>
        <td>{r.first} → {r.last}</td>
        <td>
          {#if issues[r.problem]}<a href={issues[r.problem]}>#{issueNo(issues[r.problem])}</a>{:else}none yet{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .open td { background: var(--warn-bg); }
  .open td:last-child { color: var(--warn); font-weight: 600; }
</style>
