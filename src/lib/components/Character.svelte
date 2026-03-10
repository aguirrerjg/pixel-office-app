<script lang="ts">
	import { T, useTask } from '@threlte/core';
	import type { AgentDef, AgentState } from '$lib/types';

	interface Props {
		agent: AgentDef;
		agentState?: AgentState;
	}

	let { agent, agentState = 'idle' }: Props = $props();

	// Animation state
	let breatheT = $state(0);
	let typeT = $state(0);
	let headBobT = $state(0);
	let armLRot = $state(0.2);
	let armRRot = $state(-0.2);
	let bodyY = $state(0);
	let headY = $state(0);
	let headRot = $state(0);

	// Idle micro-action
	let microAction = $state<string | null>(null);
	let microTimer = $state(0);
	let nextMicroAt = $state(3 + Math.random() * 5);

	const MICRO_ACTIONS = ['look-left', 'look-right', 'stretch', 'lean-back', 'nod-off'];

	useTask((delta) => {
		if (agentState === 'idle') {
			// Breathing
			breatheT += delta * 2;
			bodyY = Math.sin(breatheT) * 0.02;
			headY = Math.sin(breatheT) * 0.03;
			armLRot = 0.2 + Math.sin(breatheT * 0.8) * 0.05;
			armRRot = -0.2 - Math.sin(breatheT * 0.8) * 0.05;
			headRot = 0;

			// Micro-actions
			microTimer += delta;
			if (!microAction && microTimer > nextMicroAt) {
				microAction = MICRO_ACTIONS[Math.floor(Math.random() * MICRO_ACTIONS.length)];
				microTimer = 0;
			}
			if (microAction) {
				const mt = microTimer;
				if (microAction === 'look-left') {
					headRot = Math.sin(mt * 1.5) * 0.15;
					if (mt > 2.5) { microAction = null; microTimer = 0; nextMicroAt = 4 + Math.random() * 6; }
				} else if (microAction === 'look-right') {
					headRot = -Math.sin(mt * 1.5) * 0.15;
					if (mt > 2.5) { microAction = null; microTimer = 0; nextMicroAt = 4 + Math.random() * 6; }
				} else if (microAction === 'stretch') {
					armLRot = 0.2 - Math.sin(mt * 1.2) * 1.2;
					armRRot = -0.2 + Math.sin(mt * 1.2) * 1.2;
					bodyY = Math.sin(mt * 1.2) * 0.05;
					if (mt > 3) { microAction = null; microTimer = 0; nextMicroAt = 5 + Math.random() * 8; }
				} else if (microAction === 'lean-back') {
					bodyY = Math.sin(mt * 0.8) * -0.06;
					headY = Math.sin(mt * 0.8) * -0.04;
					if (mt > 3.5) { microAction = null; microTimer = 0; nextMicroAt = 4 + Math.random() * 6; }
				} else if (microAction === 'nod-off') {
					headRot = Math.sin(mt * 0.6) * 0.25;
					headY = Math.sin(mt * 0.6) * -0.08;
					if (mt > 4) { microAction = null; microTimer = 0; nextMicroAt = 6 + Math.random() * 10; }
				}
			}
		} else if (agentState === 'working') {
			// Typing animation
			typeT += delta * 8;
			armLRot = 0.3 + Math.sin(typeT) * 0.3;
			armRRot = -0.3 - Math.sin(typeT + 1.5) * 0.3;
			headBobT += delta * 3;
			headY = Math.sin(headBobT) * 0.02;
			headRot = 0;
			bodyY = -0.04; // leaning forward
			microAction = null;
			microTimer = 0;
		} else if (agentState === 'thinking') {
			// One arm up, sway
			breatheT += delta * 1.5;
			armLRot = 0.2 + Math.sin(breatheT * 0.5) * 0.08;
			armRRot = -1.2; // arm up to chin
			headRot = Math.sin(breatheT * 0.4) * 0.08;
			bodyY = -0.15; // standing taller
			headY = -0.1;
			microAction = null;
			microTimer = 0;
		}
	});
</script>

<!-- Character group -->
<T.Group position.y={bodyY}>
	<!-- Torso -->
	<T.Mesh position.y={0.55}>
		<T.BoxGeometry args={[0.35, 0.4, 0.25]} />
		<T.MeshStandardMaterial color={agent.jacket} />
	</T.Mesh>

	<!-- Shirt V-neck -->
	<T.Mesh position={[0, 0.62, 0.126]}>
		<T.BoxGeometry args={[0.15, 0.2, 0.01]} />
		<T.MeshStandardMaterial color={agent.shirt} />
	</T.Mesh>

	<!-- Left arm -->
	<T.Group position={[-0.24, 0.7, 0]}>
		<T.Group rotation.z={armLRot}>
			<T.Mesh position.y={-0.18}>
				<T.BoxGeometry args={[0.1, 0.32, 0.12]} />
				<T.MeshStandardMaterial color={agent.jacket} />
			</T.Mesh>
			<!-- Hand -->
			<T.Mesh position.y={-0.36}>
				<T.SphereGeometry args={[0.055, 6, 6]} />
				<T.MeshStandardMaterial color={agent.skinColor} />
			</T.Mesh>
		</T.Group>
	</T.Group>

	<!-- Right arm -->
	<T.Group position={[0.24, 0.7, 0]}>
		<T.Group rotation.z={armRRot}>
			<T.Mesh position.y={-0.18}>
				<T.BoxGeometry args={[0.1, 0.32, 0.12]} />
				<T.MeshStandardMaterial color={agent.jacket} />
			</T.Mesh>
			<!-- Hand -->
			<T.Mesh position.y={-0.36}>
				<T.SphereGeometry args={[0.055, 6, 6]} />
				<T.MeshStandardMaterial color={agent.skinColor} />
			</T.Mesh>
		</T.Group>
	</T.Group>

	<!-- Neck -->
	<T.Mesh position.y={0.78}>
		<T.CylinderGeometry args={[0.06, 0.07, 0.08, 6]} />
		<T.MeshStandardMaterial color={agent.skinColor} />
	</T.Mesh>

	<!-- Head -->
	<T.Group position.y={headY}>
		<T.Group rotation.z={headRot}>
			<T.Mesh position.y={1.0}>
				<T.SphereGeometry args={[0.2, 8, 8]} />
				<T.MeshStandardMaterial color={agent.skinColor} />
			</T.Mesh>

			<!-- Eyes -->
			<T.Mesh position={[-0.065, 1.02, 0.17]}>
				<T.SphereGeometry args={[0.03, 6, 6]} />
				<T.MeshStandardMaterial color="#1a1a2e" />
			</T.Mesh>
			<T.Mesh position={[0.065, 1.02, 0.17]}>
				<T.SphereGeometry args={[0.03, 6, 6]} />
				<T.MeshStandardMaterial color="#1a1a2e" />
			</T.Mesh>

			<!-- Eye highlights -->
			<T.Mesh position={[-0.055, 1.03, 0.195]}>
				<T.SphereGeometry args={[0.012, 4, 4]} />
				<T.MeshStandardMaterial color="white" emissive="white" emissiveIntensity={0.5} />
			</T.Mesh>
			<T.Mesh position={[0.075, 1.03, 0.195]}>
				<T.SphereGeometry args={[0.012, 4, 4]} />
				<T.MeshStandardMaterial color="white" emissive="white" emissiveIntensity={0.5} />
			</T.Mesh>

			<!-- Hair -->
			<T.Mesh position={[0, 1.12, -0.02]}>
				<T.BoxGeometry args={[0.42, 0.18, 0.38]} />
				<T.MeshStandardMaterial color={agent.hairColor} />
			</T.Mesh>

			<!-- Mouth -->
			<T.Mesh position={[0, 0.92, 0.19]}>
				<T.BoxGeometry args={[0.06, 0.02, 0.01]} />
				<T.MeshStandardMaterial color="#8b6f5e" />
			</T.Mesh>
		</T.Group>
	</T.Group>

	<!-- Accessories -->
	{#if agent.acc === 'glasses'}
		<T.Group position.y={headY}>
			<T.Mesh position={[0, 1.02, 0.2]}>
				<T.TorusGeometry args={[0.06, 0.008, 4, 8]} />
				<T.MeshStandardMaterial color="#9eaab8" metalness={0.6} roughness={0.3} />
			</T.Mesh>
		</T.Group>
	{/if}

	{#if agent.acc === 'headphones'}
		<T.Group position.y={headY}>
			<!-- Headband -->
			<T.Mesh position={[0, 1.18, 0]} rotation.x={0.1}>
				<T.TorusGeometry args={[0.22, 0.015, 4, 12, Math.PI]} />
				<T.MeshStandardMaterial color="#444" metalness={0.5} />
			</T.Mesh>
			<!-- Ear pads -->
			<T.Mesh position={[-0.22, 1.02, 0]}>
				<T.BoxGeometry args={[0.06, 0.1, 0.08]} />
				<T.MeshStandardMaterial color="#333" />
			</T.Mesh>
			<T.Mesh position={[0.22, 1.02, 0]}>
				<T.BoxGeometry args={[0.06, 0.1, 0.08]} />
				<T.MeshStandardMaterial color="#333" />
			</T.Mesh>
		</T.Group>
	{/if}

	{#if agent.acc === 'dogtags'}
		<T.Mesh position={[0, 0.6, 0.13]}>
			<T.BoxGeometry args={[0.04, 0.06, 0.01]} />
			<T.MeshStandardMaterial color="#9e9e9e" metalness={0.8} roughness={0.2} />
		</T.Mesh>
	{/if}
</T.Group>
