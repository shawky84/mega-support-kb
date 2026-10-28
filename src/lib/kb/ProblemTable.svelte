<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { splitTicket, type Problem } from '#lib/kb/problems';

  // `tickets`: the issue that now owns each problem, as a link, with its state in words.
  let { ticket, tickets }: { ticket: number; tickets: Record<Problem, { label: string; href: string; state?: string }> } = $props();

  const split = $derived(splitTicket(reports as Report[], ticket, tenantOf, notes.oneDayMove));
  const counted = $derived(split.groups.reduce((n, g) => n + g.reports.length, 0));
</script>

<p class="sum">
  <strong>{split.merged.length}</strong> reports merged into #{ticket} ·
  <strong>{split.groups.length}</strong>
  {split.groups.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each split.groups as g (g.problem)}
      <tr class:unplaced={g.problem === 'unplaced'}>
        <td><code>{g.problem}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>
          {#if g.problem !== 'unplaced' && tickets[g.problem]}<a href={tickets[g.problem].href}>{tickets[g.problem].label}</a>{tickets[g.problem].state ? ` · ${tickets[g.problem].state}` : ''}{:else}none yet{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

{#if counted !== split.merged.length}
  <p class="check">The problems count {counted} reports, but {split.merged.length} were merged. Check the split before you rely on it.</p>
{/if}

<style>
  .sum { font-size: 1.1rem; }
  .unplaced td { background: var(--warn-bg); }
  .check { color: var(--warn); font-weight: 600; }
</style>
