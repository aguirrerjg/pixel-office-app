<script lang="ts">
	import { T } from '@threlte/core';
	import { TEAMS, milesStatus, pmoStatus } from '$lib/stores';
	import type { AgentState, TeamStatus } from '$lib/types';
	import Workstation from './Workstation.svelte';

	interface Props {
		teamKey: string;
	}

	let { teamKey }: Props = $props();

	const team = $derived(TEAMS.find((t) => t.key === teamKey));
	const agentCount = $derived(team?.agents.length ?? 0);
	const teamStatus: TeamStatus = $derived(teamKey === 'miles' ? $milesStatus : $pmoStatus);

	// Calculate scene width based on agent count
	const sceneWidth = $derived(agentCount * 2.8 + 2);

	function getAgentState(agentId: string): AgentState {
		const status = teamStatus[agentId];
		return (status?.state as AgentState) ?? 'idle';
	}

	function getAgentActivity(agentId: string): string {
		return teamStatus[agentId]?.activity ?? '';
	}
</script>

<!-- Orthographic camera for 2D-like view -->
<T.OrthographicCamera
	makeDefault
	position={[0, 4, 8]}
	zoom={agentCount > 4 ? 48 : 60}
	near={0.1}
	far={100}
	oncreate={(ref) => ref.lookAt(0, 0.7, 0)}
/>

<!-- Lighting -->
<T.AmbientLight intensity={0.5} color="#c0d0e8" />
<T.DirectionalLight position={[5, 12, 8]} intensity={0.6} color="#e8f0ff" castShadow />
<T.DirectionalLight position={[-3, 6, -2]} intensity={0.2} color="#80a0c0" />
<!-- Subtle rim light from behind -->
<T.DirectionalLight position={[0, 4, -8]} intensity={0.15} color="#6080b0" />

<!-- ═══ FLOOR ═══ -->
<T.Mesh rotation.x={-Math.PI / 2} position.y={0} receiveShadow>
	<T.PlaneGeometry args={[sceneWidth + 4, 8]} />
	<T.MeshStandardMaterial color="#1a1f29" roughness={0.9} />
</T.Mesh>

<!-- Floor grid lines (subtle) -->
{#each Array.from({ length: Math.ceil(sceneWidth / 2) + 3 }) as _, i}
	{@const x = (i - Math.ceil(sceneWidth / 4) - 1) * 2}
	<T.Mesh rotation.x={-Math.PI / 2} position={[x, 0.001, 0]}>
		<T.PlaneGeometry args={[0.01, 8]} />
		<T.MeshBasicMaterial color="#252b35" transparent opacity={0.5} />
	</T.Mesh>
{/each}

<!-- Back wall -->
<T.Mesh position={[0, 1.5, -2.5]}>
	<T.PlaneGeometry args={[sceneWidth + 4, 3]} />
	<T.MeshStandardMaterial color="#13171e" roughness={0.95} />
</T.Mesh>

<!-- ═══ WORKSTATIONS ═══ -->
{#if team}
	{#each team.agents as agent, idx}
		{@const xPos = (idx - (agentCount - 1) / 2) * 2.8}
		<Workstation
			{agent}
			position={[xPos, 0, 0]}
			state={getAgentState(agent.id)}
			activity={getAgentActivity(agent.id)}
		/>
	{/each}
{/if}
