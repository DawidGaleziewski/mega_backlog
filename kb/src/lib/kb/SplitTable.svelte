<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem merged into the ticket, counted the desk's way: a report is counted under its
  // tenant's cause (causes.ts). The desk's one-day-move note (desk-notes.json) is a problem of its own
  // with its own ticket, but its reports stay counted where they are filed, as CauseTable does.
  type Problem = 'parser' | 'hint' | 'oneDayMove' | 'unplaced';
  const LABEL: Record<Problem, string> = {
    parser: 'day and month swap: the parser reads month first',
    hint: 'day and month swap: the booking-v3 hint says MM/DD',
    oneDayMove: 'the date moves by one day',
    unplaced: 'no cause on file',
  };

  // `status`: what engineering decided per problem, in the desk's words; a problem without one is still open.
  let {
    ticket,
    issues,
    status = {},
  }: { ticket: number; issues: Partial<Record<Problem, string>>; status?: Partial<Record<Problem, string>> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const causeRow = (r: Report): Problem => causeOf(r, tenantOf(r)) ?? 'unplaced';

  const rows = $derived.by(() => {
    const by = new Map<Problem, Report[]>();
    for (const r of merged) by.set(causeRow(r), [...(by.get(causeRow(r)) ?? []), r]);
    const moved = merged.filter((r) => notes.oneDayMove.includes(r.id));
    if (moved.length) by.set('oneDayMove', moved);
    return [...by.entries()]
      .map(([problem, rs]) => {
        const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
        const channels = [...new Set(rs.map((r) => r.channel))].sort().join(', ');
        return { problem, n: rs.length, tenants: new Set(rs.map((r) => r.tenant)).size, channels, first: days[0], last: days[days.length - 1] };
      })
      .sort((a, b) => b.n - a.n);
  });
  const movedFrom = $derived(
    [...new Set(merged.filter((r) => notes.oneDayMove.includes(r.id)).map(causeRow))].map((c) => LABEL[c]),
  );
  const issueNo = (url?: string) => url?.match(/(\d+)$/)?.[1];
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong> problems · <strong>{rows.filter((r) => !status[r.problem]).length}</strong> still open
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>channels</th><th>seen</th><th>status</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr class:open={!status[r.problem]}>
        <td>{LABEL[r.problem]}</td>
        <td>{r.n}</td>
        <td>{r.tenants}</td>
        <td>{r.channels}</td>
        <td>{r.first} → {r.last}</td>
        <td>{status[r.problem] ?? 'open'}</td>
        <td>
          {#if issues[r.problem]}<a href={issues[r.problem]}>#{issueNo(issues[r.problem])}</a>{:else}none yet{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

{#if rows.some((r) => r.problem === 'oneDayMove')}
  <p class="note">
    The {rows.find((r) => r.problem === 'oneDayMove')?.n} one-day reports are also counted in the row they are filed under
    ({movedFrom.join(', ')}), as the desk counts them; the rows add up to more than {merged.length}.
  </p>
{/if}

<style>
  .sum { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.85em; }
  .open td { background: var(--warn-bg); }
  .open td:nth-last-child(2) { color: var(--warn); font-weight: 600; }
</style>
