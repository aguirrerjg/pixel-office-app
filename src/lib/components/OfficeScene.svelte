<script lang="ts">
	import { TEAMS, milesStatus, pmoStatus } from '$lib/stores';
	import type { AgentState, TeamStatus } from '$lib/types';
	import PixelCharacter from './PixelCharacter.svelte';

	interface Props {
		teamKey: string;
		teamName: string;
		teamColor: string;
		platform: string;
	}

	let { teamKey, teamName, teamColor, platform }: Props = $props();

	const team = $derived(TEAMS.find((t) => t.key === teamKey));
	const agentCount = $derived(team?.agents.length ?? 0);
	const teamStatus: TeamStatus = $derived(teamKey === 'miles' ? $milesStatus : $pmoStatus);

	function getAgentState(agentId: string): AgentState {
		const status = teamStatus[agentId];
		return (status?.state as AgentState) ?? 'idle';
	}

	function getAgentActivity(agentId: string): string {
		return teamStatus[agentId]?.activity ?? '';
	}
</script>

<div class="office-panel">
	<header class="office-header">
		<div class="header-left">
			<span class="team-name" style="color: {teamColor}">{teamName}</span>
			<span class="platform-badge">{platform}</span>
		</div>
		<div class="header-right">
			<div class="office-stats">
				{#if team}
					{@const working = team.agents.filter(a => getAgentState(a.id) === 'working').length}
					{@const thinking = team.agents.filter(a => getAgentState(a.id) === 'thinking').length}
					<span class="o-stat"><span class="s-pip working"></span> {working} working</span>
					<span class="o-stat"><span class="s-pip thinking"></span> {thinking} thinking</span>
				{/if}
			</div>
		</div>
	</header>
	<div class="ws-area">
		<!-- ═══ BACK WALL ═══ -->
		<div class="back-wall">
			<!-- Window (right side) -->
			<div class="window">
				<div class="window-frame">
					<div class="window-pane left">
						<div class="window-view"></div>
					</div>
					<div class="window-pane right">
						<div class="window-view"></div>
					</div>
					<div class="window-divider-v"></div>
					<div class="window-divider-h"></div>
				</div>
			</div>
			<!-- World map / tech pattern (left side) -->
			<div class="wall-art">
				<div class="circuit-line c1"></div>
				<div class="circuit-line c2"></div>
				<div class="circuit-line c3"></div>
				<div class="circuit-dot d1"></div>
				<div class="circuit-dot d2"></div>
				<div class="circuit-dot d3"></div>
				<div class="circuit-dot d4"></div>
			</div>
			<!-- Bookshelf left -->
			<div class="bookshelf left">
				<div class="shelf-board"></div>
				<div class="book b1"></div>
				<div class="book b2"></div>
				<div class="book b3"></div>
				<div class="book b4"></div>
				<div class="book b5"></div>
			</div>
			<!-- Bookshelf right -->
			<div class="bookshelf right">
				<div class="shelf-board"></div>
				<div class="book b1"></div>
				<div class="book b2"></div>
				<div class="book b3"></div>
				<div class="book b4"></div>
			</div>
		</div>

		<!-- ═══ FLOOR ═══ -->
		<div class="floor">
			<div class="floor-plank p1"></div>
			<div class="floor-plank p2"></div>
			<div class="floor-plank p3"></div>
			<div class="floor-plank p4"></div>
		</div>

		<!-- ═══ SHARED DESK ═══ -->
		<div class="shared-desk">
			<div class="desk-surface"></div>
			<div class="desk-front"></div>
			<div class="desk-leg dl"></div>
			<div class="desk-leg dr"></div>
		</div>

		<!-- ═══ PLANTS ═══ -->
		<div class="plant left">
			<div class="pot"></div>
			<div class="trunk"></div>
			<div class="foliage f1"></div>
			<div class="foliage f2"></div>
			<div class="foliage f3"></div>
		</div>
		<div class="plant right">
			<div class="pot"></div>
			<div class="trunk"></div>
			<div class="foliage f1"></div>
			<div class="foliage f2"></div>
			<div class="foliage f3"></div>
		</div>

		<!-- ═══ SERVER RACKS (bottom) ═══ -->
		<div class="server-area">
			{#each Array(Math.min(agentCount, 5)) as _, i}
				<div class="server-rack">
					<div class="rack-unit ru1"></div>
					<div class="rack-unit ru2"></div>
					<div class="rack-unit ru3"></div>
					<div class="rack-led l1"></div>
					<div class="rack-led l2"></div>
					<div class="rack-led l3"></div>
				</div>
			{/each}
		</div>

		<!-- ═══ AGENTS AT DESK ═══ -->
		{#if team}
			{#each team.agents as agent, idx}
				{@const pct = agentCount === 1 ? 50 : (12 + idx * (76 / (agentCount - 1)))}
				{@const agentState = getAgentState(agent.id)}
				<div
					class="workstation-wrap {agentState}"
					style="left:{pct}%;--agent-color:{agent.color}"
				>
					<!-- Monitor -->
					<div class="monitor">
						<div class="monitor-screen">
							<div class="screen-glow"></div>
							<div class="screen-lines">
								{#each Array(5) as _, i}
									<div
										class="screen-line"
										style="width:{40+Math.random()*50}%;background:{agent.color};animation-delay:{i*0.15}s"
									></div>
								{/each}
							</div>
						</div>
						<div class="monitor-stand"></div>
						<div class="monitor-base"></div>
					</div>
					<!-- Work pulse -->
					<div class="work-pulse"></div>
					<!-- Character -->
					<PixelCharacter {agent} agentState={agentState} activity={getAgentActivity(agent.id)} />
				</div>
			{/each}
		{/if}
	</div>
</div>

<style>
	.office-panel {
		flex: 1;
		display: flex;
		flex-direction: column;
		background: #1a130d;
		border: 1px solid rgba(180, 140, 80, 0.15);
		border-radius: 8px;
		overflow: hidden;
		min-height: 280px;
	}

	.office-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 8px 16px;
		background: linear-gradient(180deg, #2a1f14 0%, #1e1610 100%);
		border-bottom: 1px solid rgba(180, 140, 80, 0.15);
	}
	.header-left { display: flex; align-items: center; gap: 8px; }
	.header-right { display: flex; align-items: center; }
	.team-name {
		font-family: 'Chakra Petch', monospace;
		font-size: 13px; font-weight: 700;
		letter-spacing: 1px; text-transform: uppercase;
	}
	.platform-badge {
		font-family: 'Fira Code', monospace;
		font-size: 9px; color: rgba(255, 220, 160, 0.5);
		background: rgba(255, 200, 100, 0.06);
		padding: 2px 6px; border-radius: 3px;
	}
	.office-stats { display: flex; gap: 14px; font-family: 'Fira Code', monospace; font-size: 10px; }
	.o-stat { display: flex; align-items: center; gap: 5px; color: #8a7a60; }
	.s-pip { width: 5px; height: 5px; border-radius: 50%; display: inline-block; }
	.s-pip.working { background: #00e676; }
	.s-pip.thinking { background: #b388ff; }

	/* ═══ MAIN SCENE ═══ */
	.ws-area {
		flex: 1;
		position: relative;
		overflow: hidden;
		min-height: 220px;
		background: linear-gradient(180deg, #3a2d1e 0%, #2e2318 40%, #251c12 100%);
	}

	/* ═══ BACK WALL ═══ */
	.back-wall {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		height: 52%;
		background:
			linear-gradient(180deg, #5a4530 0%, #4d3a28 60%, #3a2d1e 100%);
		border-bottom: 3px solid #2a1f14;
		z-index: 0;
	}
	/* Wood grain texture on wall */
	.back-wall::before {
		content: '';
		position: absolute;
		inset: 0;
		background:
			repeating-linear-gradient(90deg, transparent 0px, transparent 60px, rgba(0,0,0,0.03) 60px, rgba(0,0,0,0.03) 61px),
			repeating-linear-gradient(0deg, transparent 0px, transparent 8px, rgba(255,220,160,0.02) 8px, rgba(255,220,160,0.02) 9px);
	}
	/* Wainscoting / lower wall panel */
	.back-wall::after {
		content: '';
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		height: 45%;
		background: linear-gradient(180deg, #3d3025 0%, #352a1e 100%);
		border-top: 2px solid #4a3928;
	}

	/* ── Window ── */
	.window {
		position: absolute;
		top: 8%;
		right: 8%;
		width: 35%;
		height: 70%;
		z-index: 2;
	}
	.window-frame {
		width: 100%;
		height: 100%;
		background: #3a2a18;
		border: 3px solid #5a4530;
		border-radius: 2px;
		position: relative;
		overflow: hidden;
		box-shadow: inset 0 0 10px rgba(0,0,0,0.3);
	}
	.window-pane {
		position: absolute;
		top: 3px;
		bottom: 3px;
		overflow: hidden;
	}
	.window-pane.left { left: 3px; right: 52%; }
	.window-pane.right { left: 52%; right: 3px; }
	.window-view {
		width: 100%;
		height: 100%;
		background:
			linear-gradient(180deg,
				#87ceeb 0%,
				#a8d8ea 30%,
				#c8e6c9 55%,
				#81c784 65%,
				#4caf50 75%,
				#388e3c 85%,
				#2e7d32 100%
			);
	}
	.window-divider-v {
		position: absolute;
		top: 0; bottom: 0;
		left: 50%; width: 4px;
		transform: translateX(-50%);
		background: #5a4530;
		z-index: 1;
	}
	.window-divider-h {
		position: absolute;
		left: 0; right: 0;
		top: 50%; height: 3px;
		transform: translateY(-50%);
		background: #5a4530;
		z-index: 1;
	}

	/* ── Circuit pattern on wall ── */
	.wall-art {
		position: absolute;
		top: 10%;
		left: 5%;
		width: 45%;
		height: 60%;
		z-index: 1;
	}
	.circuit-line {
		position: absolute;
		background: rgba(180, 150, 100, 0.12);
	}
	.circuit-line.c1 { top: 30%; left: 10%; width: 60%; height: 1px; }
	.circuit-line.c2 { top: 50%; left: 20%; width: 40%; height: 1px; }
	.circuit-line.c3 { top: 30%; left: 40%; width: 1px; height: 40%; }
	.circuit-dot {
		position: absolute;
		width: 4px; height: 4px;
		border-radius: 50%;
		background: rgba(180, 150, 100, 0.15);
		border: 1px solid rgba(180, 150, 100, 0.1);
	}
	.circuit-dot.d1 { top: 28%; left: 10%; }
	.circuit-dot.d2 { top: 28%; left: 70%; }
	.circuit-dot.d3 { top: 48%; left: 20%; }
	.circuit-dot.d4 { top: 68%; left: 40%; }

	/* ── Bookshelves ── */
	.bookshelf {
		position: absolute;
		bottom: 50%;
		height: 30px;
		z-index: 2;
	}
	.bookshelf.left { left: 6%; width: 18%; }
	.bookshelf.right { left: 50%; width: 14%; }
	.shelf-board {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		height: 4px;
		background: linear-gradient(180deg, #6b5540 0%, #4a3828 100%);
		border-radius: 1px;
		box-shadow: 0 2px 4px rgba(0,0,0,0.3);
	}
	.book {
		position: absolute;
		bottom: 4px;
		border-radius: 1px 1px 0 0;
	}
	/* Book variations */
	.bookshelf.left .book.b1 { left: 5%; width: 7px; height: 18px; background: #c62828; }
	.bookshelf.left .book.b2 { left: 18%; width: 9px; height: 20px; background: #1565c0; }
	.bookshelf.left .book.b3 { left: 34%; width: 6px; height: 16px; background: #2e7d32; }
	.bookshelf.left .book.b4 { left: 48%; width: 10px; height: 22px; background: #e65100; }
	.bookshelf.left .book.b5 { left: 68%; width: 8px; height: 17px; background: #6a1b9a; }
	.bookshelf.right .book.b1 { left: 8%; width: 8px; height: 19px; background: #01579b; }
	.bookshelf.right .book.b2 { left: 30%; width: 10px; height: 21px; background: #bf360c; }
	.bookshelf.right .book.b3 { left: 55%; width: 7px; height: 16px; background: #004d40; }
	.bookshelf.right .book.b4 { left: 75%; width: 9px; height: 20px; background: #4a148c; }

	/* ═══ FLOOR ═══ */
	.floor {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		height: 48%;
		background: linear-gradient(180deg, #3a2d1e 0%, #2e2318 30%, #261c12 100%);
		z-index: 0;
	}
	/* Wood plank lines */
	.floor-plank {
		position: absolute;
		left: 0;
		right: 0;
		height: 1px;
		background: rgba(80, 60, 35, 0.3);
	}
	.floor-plank.p1 { top: 20%; }
	.floor-plank.p2 { top: 45%; }
	.floor-plank.p3 { top: 65%; }
	.floor-plank.p4 { top: 82%; }
	.floor::before {
		content: '';
		position: absolute;
		inset: 0;
		background:
			repeating-linear-gradient(90deg, transparent 0px, transparent 80px, rgba(0,0,0,0.04) 80px, rgba(0,0,0,0.04) 81px);
	}
	/* Floor shadow from desk area */
	.floor::after {
		content: '';
		position: absolute;
		top: 0;
		left: 15%;
		right: 15%;
		height: 30px;
		background: radial-gradient(ellipse 100% 100%, rgba(0,0,0,0.15) 0%, transparent 70%);
	}

	/* ═══ SHARED DESK ═══ */
	.shared-desk {
		position: absolute;
		bottom: 32%;
		left: 10%;
		right: 10%;
		height: 14px;
		z-index: 3;
	}
	.desk-surface {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		height: 8px;
		background: linear-gradient(180deg, #c8a87a 0%, #b89868 50%, #a88858 100%);
		border-radius: 3px 3px 0 0;
		border: 1px solid rgba(180, 150, 100, 0.3);
		border-bottom: none;
		box-shadow: 0 2px 8px rgba(0,0,0,0.2);
	}
	.desk-front {
		position: absolute;
		top: 7px;
		left: 0;
		right: 0;
		height: 18px;
		background: linear-gradient(180deg, #8a7050 0%, #6b5540 100%);
		border-radius: 0 0 2px 2px;
		border: 1px solid rgba(100, 80, 50, 0.3);
		border-top: none;
	}
	.desk-leg {
		position: absolute;
		bottom: -20px;
		width: 6px;
		height: 20px;
		background: linear-gradient(180deg, #6b5540 0%, #4a3828 100%);
		border-radius: 0 0 2px 2px;
	}
	.desk-leg.dl { left: 8%; }
	.desk-leg.dr { right: 8%; }

	/* ═══ PLANTS ═══ */
	.plant {
		position: absolute;
		bottom: 30%;
		z-index: 4;
	}
	.plant.left { left: 2%; }
	.plant.right { right: 2%; }
	.pot {
		position: absolute;
		bottom: 0;
		left: 50%;
		transform: translateX(-50%);
		width: 20px;
		height: 18px;
		background: linear-gradient(180deg, #a0522d 0%, #8b4513 100%);
		border-radius: 2px 2px 4px 4px;
		border: 1px solid #6d3a1e;
	}
	.pot::after {
		content: '';
		position: absolute;
		top: -3px;
		left: -2px;
		right: -2px;
		height: 4px;
		background: #8b4513;
		border-radius: 2px;
	}
	.trunk {
		position: absolute;
		bottom: 16px;
		left: 50%;
		transform: translateX(-50%);
		width: 4px;
		height: 25px;
		background: linear-gradient(180deg, #5d4037 0%, #795548 100%);
		border-radius: 2px;
	}
	.foliage {
		position: absolute;
		border-radius: 50%;
	}
	.foliage.f1 {
		bottom: 35px;
		left: 50%;
		transform: translateX(-50%);
		width: 30px;
		height: 28px;
		background: radial-gradient(ellipse, #4caf50 20%, #2e7d32 70%, #1b5e20 100%);
	}
	.foliage.f2 {
		bottom: 48px;
		left: 50%;
		transform: translateX(-65%);
		width: 22px;
		height: 20px;
		background: radial-gradient(ellipse, #66bb6a 20%, #388e3c 70%);
	}
	.foliage.f3 {
		bottom: 45px;
		left: 50%;
		transform: translateX(-20%);
		width: 18px;
		height: 18px;
		background: radial-gradient(ellipse, #43a047 20%, #2e7d32 70%);
	}

	/* ═══ SERVER RACKS ═══ */
	.server-area {
		position: absolute;
		bottom: 2%;
		left: 8%;
		right: 8%;
		height: 18%;
		display: flex;
		gap: 4px;
		justify-content: center;
		z-index: 1;
	}
	.server-rack {
		width: 40px;
		height: 100%;
		background: linear-gradient(180deg, #2a2a30 0%, #1e1e24 100%);
		border: 1px solid #3a3a44;
		border-radius: 2px;
		position: relative;
	}
	.rack-unit {
		position: absolute;
		left: 3px;
		right: 3px;
		height: 20%;
		background: #18181e;
		border: 1px solid #333340;
		border-radius: 1px;
	}
	.rack-unit.ru1 { top: 8%; }
	.rack-unit.ru2 { top: 36%; }
	.rack-unit.ru3 { top: 64%; }
	.rack-led {
		position: absolute;
		width: 3px;
		height: 3px;
		border-radius: 50%;
		right: 6px;
		animation: ledBlink 3s ease-in-out infinite;
	}
	.rack-led.l1 { top: 14%; background: #00e676; animation-delay: 0s; }
	.rack-led.l2 { top: 42%; background: #ffb300; animation-delay: 1s; }
	.rack-led.l3 { top: 70%; background: #00e5ff; animation-delay: 0.5s; }
	@keyframes ledBlink { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }

	/* ═══ WORKSTATION WRAP ═══ */
	.workstation-wrap {
		position: absolute;
		bottom: 35%;
		width: 120px;
		display: flex;
		flex-direction: column;
		align-items: center;
		transition: all 0.5s ease;
		z-index: 5;
		transform: translateX(-50%);
	}
	.workstation-wrap::before {
		content: '';
		position: absolute;
		bottom: -4px;
		left: 50%;
		transform: translateX(-50%);
		width: 80px;
		height: 20px;
		border-radius: 50%;
		background: var(--agent-color);
		opacity: 0;
		filter: blur(15px);
		transition: opacity 0.5s;
		z-index: -1;
	}
	.workstation-wrap.working::before { opacity: 0.2; }
	.workstation-wrap.thinking::before { opacity: 0.12; background: #b388ff; }

	/* ── Monitor (desktop style) ── */
	.monitor {
		position: absolute;
		bottom: 28px;
		left: 50%;
		transform: translateX(-50%);
		z-index: 6;
	}
	.monitor-screen {
		width: 34px;
		height: 24px;
		background: #0a0f1e;
		border: 2px solid #333;
		border-radius: 2px 2px 0 0;
		overflow: hidden;
		position: relative;
	}
	.monitor-stand {
		width: 8px;
		height: 6px;
		background: #444;
		margin: 0 auto;
	}
	.monitor-base {
		width: 18px;
		height: 3px;
		background: #555;
		border-radius: 1px;
		margin: 0 auto;
	}
	.screen-glow {
		position: absolute;
		inset: 2px;
		background: linear-gradient(180deg, rgba(50, 80, 140, 0.3) 0%, rgba(30, 50, 100, 0.15) 100%);
		border-radius: 1px;
	}
	.workstation-wrap.working .screen-glow {
		background: linear-gradient(180deg, color-mix(in srgb, var(--agent-color) 60%, transparent) 0%, color-mix(in srgb, var(--agent-color) 25%, transparent) 100%);
		animation: screenFlicker 1.5s ease-in-out infinite;
	}
	.workstation-wrap.thinking .screen-glow {
		background: linear-gradient(180deg, rgba(179, 136, 255, 0.4) 0%, rgba(179, 136, 255, 0.15) 100%);
		animation: screenFlicker 2.5s ease-in-out infinite;
	}
	@keyframes screenFlicker { 0%, 100% { opacity: 0.6; } 50% { opacity: 1; } }
	.screen-lines {
		position: absolute; inset: 3px;
		display: flex; flex-direction: column;
		gap: 2px; padding: 1px;
	}
	.screen-lines :global(.screen-line) { height: 1.5px; border-radius: 1px; opacity: 0.3; }
	.workstation-wrap.working .screen-lines :global(.screen-line) { animation: typeLine 1s ease-in-out infinite; opacity: 0.7; }
	.workstation-wrap.thinking .screen-lines :global(.screen-line) { animation: typeLine 3s ease-in-out infinite; opacity: 0.4; }
	@keyframes typeLine { 0% { width: 0; } 30% { width: 100%; } 60% { width: 40%; } 100% { width: 80%; } }

	/* Monitor glow on desk */
	.workstation-wrap.working .monitor::after {
		content: '';
		position: absolute;
		bottom: -6px;
		left: 50%;
		transform: translateX(-50%);
		width: 50px;
		height: 14px;
		border-radius: 50%;
		background: var(--agent-color);
		opacity: 0.2;
		filter: blur(8px);
		animation: deskGlow 1.5s ease-in-out infinite;
	}
	.workstation-wrap.thinking .monitor::after {
		content: '';
		position: absolute;
		bottom: -6px;
		left: 50%;
		transform: translateX(-50%);
		width: 50px;
		height: 14px;
		border-radius: 50%;
		background: #b388ff;
		opacity: 0.15;
		filter: blur(8px);
		animation: deskGlow 2.5s ease-in-out infinite;
	}
	@keyframes deskGlow { 0%, 100% { opacity: 0.1; } 50% { opacity: 0.25; } }

	/* Work pulse */
	.work-pulse {
		position: absolute;
		bottom: 38px;
		left: 50%;
		transform: translateX(-50%);
		width: 50px; height: 50px;
		border-radius: 50%;
		border: 1px solid var(--agent-color);
		opacity: 0;
		z-index: 0;
		pointer-events: none;
	}
	.workstation-wrap.working .work-pulse { animation: pulseRing 2s ease-out infinite; }
	@keyframes pulseRing { 0% { opacity: 0.5; transform: translateX(-50%) scale(0.6); } 100% { opacity: 0; transform: translateX(-50%) scale(2); } }
</style>
