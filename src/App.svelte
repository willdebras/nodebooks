<script>
	import InfiniteViewer from "svelte-infinite-viewer"
	import { storeFE, idIncrement } from './store.js'
	import 'ace-builds/src-noconflict/ace'

	import Controls from "./components/Controls.svelte"
	import Node from "./components/Node.svelte"
	import LeaderLine from 'leader-line'

	let selectionOfCheckboxes = [];

	let startNode, elmPoint, endNode
	let connectMode = false, line

	function updateLine(e) {
		elmPoint.style.left = `${e.clientX}px`;
		elmPoint.style.top = `${e.clientY}px`;
		line.position();
	}

	function connectANode(e) {
		if(e.target.nodeName === 'INPUT') return;
		if (connectMode) {
			const elmsEnd = Array.from(document.querySelectorAll('.topNodeSpan'))
			const elmTarget = elmsEnd.find(elm => elm === e.target)
			if (elmTarget) {
				line.setOptions({end: elmTarget, endPlug: 'behind', dash: false});
				// const viewerContainer = document.querySelector('div.infinite-viewer-wrapper>div.viewer')
				// const lineDefs = document.getElementById('leader-line-defs')
				// const lineElem = document.querySelector('.leader-line')
				// viewerContainer.appendChild(lineDefs)
				// viewerContainer.appendChild(lineElem)
			} else { // Abort
				line.remove();
			}
			// todo connectMode should be on if you drag an editor around 
			// because then we can 
			connectMode = false;
		} else {
			const elmsStart = Array.from(document.querySelectorAll('.bottomNodeSpan'))
			const elmTarget = elmsStart.find(elm => elm === e.target)
			if(elmTarget) {
				line = new LeaderLine(elmTarget, elmPoint, {
					endPlug: 'disc',
					dash: {animation: true}
				});
				connectMode = true
				updateLine(e)
			}
		}
	}

	$storeFE = [
		{ id:0, name: 'First Node', otherattrib: "additional note", top: 100, left: 100 },
		// other items can go here
	];
	idIncrement.set(1)	// this is a crude way to increment the id for new items
	export function addItem(){
			var l = $storeFE.length;	// get our current items list count
			$storeFE[l] = { id:$idIncrement, name: 'Another Node', otherattrib: "additional note", top: 100, left: 100 }
			console.log($storeFE)
			$idIncrement++	// increment our id to add additional items
	}

	let pannable = false

	function allowDrag(e) {
		if(e.key === 'Control') {
			pannable = true
		} else {
			pannable = false
		}
	}

	function preventDrag(e) {
		pannable = false
	}

	function onMouseMove(e) {
		// handles the leader lines
		if (connectMode) { updateLine(e); }
	}

</script>

<svelte:window on:keydown={allowDrag} on:keyup={preventDrag} on:click={connectANode} on:mousemove={onMouseMove} />

<InfiniteViewer
	className="viewer {pannable ?  'cursorgrab' : ''}"
	margin={0}
	threshold={0}
	rangeX={[-Infinity, Infinity]}
	rangeY={[-Infinity, Infinity]}
	zoom=1
	useMouseDrag={pannable}
	useAutoZoom=true
	wheelScale=0.001
	onScroll={e => {
		console.log(e);
	}}
	>
	<div class="viewer">
		<!-- <svg xmlns="http://www.w3.org/2000/svg" version="1.1" id="leader-line-defs"><style><![CDATA[.leader-line{position:absolute;overflow:visible!important;pointer-events:none!important;font-size:16px}#leader-line-defs{width:0;height:0;position:absolute;left:0;top:0}.leader-line-line-path{fill:none}.leader-line-mask-bg-rect{fill:white}.leader-line-caps-mask-anchor,.leader-line-caps-mask-marker-shape{fill:black}.leader-line-caps-mask-anchor{stroke:black}.leader-line-caps-mask-line,.leader-line-plugs-face{stroke:rgba(0,0,0,0)}.leader-line-line-mask-shape{stroke:white}.leader-line-line-outline-mask-shape{stroke:black}.leader-line-plug-mask-shape{fill:white;stroke:black}.leader-line-plug-outline-mask-shape{fill:black;stroke:white}.leader-line-areaAnchor{position:absolute;overflow:visible!important}]]></style><defs><circle id="leader-line-disc" cx="0" cy="0" r="5"></circle><rect id="leader-line-square" x="-5" y="-5" width="10" height="10"></rect><polygon id="leader-line-arrow1" points="-8,-8 8,0 -8,8 -5,0"></polygon><polygon id="leader-line-arrow2" points="-4,-8 4,0 -4,8 -7,5 -2,0 -7,-5"></polygon><polygon id="leader-line-arrow3" points="-4,-5 8,0 -4,5"></polygon><g id="leader-line-hand"><path style="fill: #fcfcfc" d="M9.19 11.14h4.75c1.38 0 2.49-1.11 2.49-2.49 0-.51-.15-.98-.41-1.37h1.3c1.38 0 2.49-1.11 2.49-2.49s-1.11-2.53-2.49-2.53h1.02c1.38 0 2.49-1.11 2.49-2.49s-1.11-2.49-2.49-2.49h14.96c1.37 0 2.49-1.11 2.49-2.49s-1.11-2.49-2.49-2.49H16.58C16-9.86 14.28-11.14 9.7-11.14c-4.79 0-6.55 3.42-7.87 4.73H-2.14v13.23h3.68C3.29 9.97 5.47 11.14 9.19 11.14L9.19 11.14Z"></path><path style="fill: black" d="M13.95 12c1.85 0 3.35-1.5 3.35-3.35 0-.17-.02-.34-.04-.51h.07c1.85 0 3.35-1.5 3.35-3.35 0-.79-.27-1.51-.72-2.08 1.03-.57 1.74-1.67 1.74-2.93 0-.59-.16-1.15-.43-1.63h12.04c1.85 0 3.35-1.5 3.35-3.35 0-1.85-1.5-3.35-3.35-3.35H17.2C16.26-10.93 13.91-12 9.7-12 5.36-12 3.22-9.4 1.94-7.84c0 0-.29.33-.5.57-.63 0-3.58 0-3.58 0C-2.61-7.27-3-6.88-3-6.41v13.23c0 .47.39.86.86.86 0 0 2.48 0 3.2 0C2.9 10.73 5.29 12 9.19 12L13.95 12ZM9.19 10.28c-3.46 0-5.33-1.05-6.9-3.87-.15-.27-.44-.44-.75-.44 0 0-1.81 0-2.82 0V-5.55c1.06 0 3.11 0 3.11 0 .25 0 .44-.06.61-.25l.83-.95c1.23-1.49 2.91-3.53 6.43-3.53 3.45 0 4.9.74 5.57 1.72h-4.3c-.48 0-.86.38-.86.86s.39.86.86.86h22.34c.9 0 1.63.73 1.63 1.63 0 .9-.73 1.63-1.63 1.63H15.83c-.48 0-.86.38-.86.86 0 .47.39.86.86.86h2.52c.9 0 1.63.73 1.63 1.63s-.73 1.63-1.63 1.63h-3.12c-.48 0-.86.38-.86.86 0 .47.39.86.86.86h2.11c.88 0 1.63.76 1.63 1.67 0 .9-.73 1.63-1.63 1.63h-3.2c-.48 0-.86.39-.86.86 0 .47.39.86.86.86h1.36c.05.16.09.34.09.51 0 .9-.73 1.63-1.63 1.63C13.95 10.28 9.19 10.28 9.19 10.28Z"></path></g><g id="leader-line-crosshair"><path d="M0-78.97c-43.54 0-78.97 35.43-78.97 78.97 0 43.54 35.43 78.97 78.97 78.97s78.97-35.43 78.97-78.97C78.97-43.54 43.55-78.97 0-78.97ZM76.51-1.21h-9.91v-9.11h-2.43v9.11h-11.45c-.64-28.12-23.38-50.86-51.5-51.5V-64.17h9.11V-66.6h-9.11v-9.91C42.46-75.86 75.86-42.45 76.51-1.21ZM-1.21-30.76h-9.11v2.43h9.11V-4.2c-1.44.42-2.57 1.54-2.98 2.98H-28.33v-9.11h-2.43v9.11H-50.29C-49.65-28-27.99-49.65-1.21-50.29V-30.76ZM-30.76 1.21v9.11h2.43v-9.11H-4.2c.42 1.44 1.54 2.57 2.98 2.98v24.13h-9.11v2.43h9.11v19.53C-27.99 49.65-49.65 28-50.29 1.21H-30.76ZM1.22 30.75h9.11v-2.43h-9.11V4.2c1.44-.42 2.56-1.54 2.98-2.98h24.13v9.11h2.43v-9.11h19.53C49.65 28 28 49.65 1.22 50.29V30.75ZM30.76-1.21v-9.11h-2.43v9.11H4.2c-.42-1.44-1.54-2.56-2.98-2.98V-28.33h9.11v-2.43h-9.11V-50.29C28-49.65 49.65-28 50.29-1.21H30.76ZM-1.21-76.51v9.91h-9.11v2.43h9.11v11.45c-28.12.64-50.86 23.38-51.5 51.5H-64.17v-9.11H-66.6v9.11h-9.91C-75.86-42.45-42.45-75.86-1.21-76.51ZM-76.51 1.21h9.91v9.11h2.43v-9.11h11.45c.64 28.12 23.38 50.86 51.5 51.5v11.45h-9.11v2.43h9.11v9.91C-42.45 75.86-75.86 42.45-76.51 1.21ZM1.22 76.51v-9.91h9.11v-2.43h-9.11v-11.45c28.12-.64 50.86-23.38 51.5-51.5h11.45v9.11h2.43v-9.11h9.91C75.86 42.45 42.45 75.86 1.22 76.51Z"></path><path d="M0 83.58-7.1 96 7.1 96Z"></path><path d="M0-83.58 7.1-96-7.1-96"></path><path d="M83.58 0 96 7.1 96-7.1Z"></path><path d="M-83.58 0-96-7.1-96 7.1Z"></path></g></defs></svg> -->
		<h2 class="description" data-moveable>Infinite canvas IDE</h2>
		<div>Testing new div</div>
		<ul>
			{#each $storeFE as item}
				<svelte:component this={Node} objAttributes={item} bind:selectionOfCheckboxes />
			{/each}
		</ul>
		<div id='elm-point' bind:this={elmPoint}></div>
	</div>
</InfiniteViewer>

<Controls itemFunc={() => addItem()}></Controls>



<style>

#leader-line-defs {
	width:0;
	height:0;
	position:absolute;
	left:0;
	top:0;
	overflow:hidden;
}

#elm-point {
	position: absolute;
	width: 0;
	height: 0;
	/* display: none; */
	}


:global(.viewer) {
    position: absolute !important;
    width: 100%;
    height: 100%;
}

:global(.cursorgrab) {
	cursor:grabbing;
}

:global(body) {
		overflow: hidden;
	}

h2 {
	margin: 12px 24px;
}

</style>