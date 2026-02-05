<script>
    import { onMount } from "svelte"

    const mouse = { x: 0, y: 0 }

    // Default values
    const defaults = {
        shape: 'circle',
        style: 'needle',
        densityMode: 'auto', // 'auto' | 'uniform' | 'proportional'
        cursorMode: 'default', // 'default' | 'none' | 'crosshair' | 'dot' | 'ring' | 'glow'
        rings: 6,
        needlesPerRing: 48,
        ringSpacing: 30,
        effectRadius: 300,
        brightnessRadius: 500,
        needleLength: 10,
        needleWidth: 1,
        gridCols: 20,
        gridRows: 10,
        gridGap: 10,
        starPoints: 8,
        hexRings: 5,
        waveAmplitude: 50,
        waveCycles: 3
    }

    // Tunable parameters
    let shape = $state(defaults.shape)
    let style = $state(defaults.style)
    let densityMode = $state(defaults.densityMode)
    let cursorMode = $state(defaults.cursorMode)
    let rings = $state(defaults.rings)
    let needlesPerRing = $state(defaults.needlesPerRing)
    let ringSpacing = $state(defaults.ringSpacing)
    let effectRadius = $state(defaults.effectRadius)
    let brightnessRadius = $state(defaults.brightnessRadius)
    let needleLength = $state(defaults.needleLength)
    let needleWidth = $state(defaults.needleWidth)
    let gridCols = $state(defaults.gridCols)
    let gridRows = $state(defaults.gridRows)
    let gridGap = $state(defaults.gridGap)
    let starPoints = $state(defaults.starPoints)
    let hexRings = $state(defaults.hexRings)
    let waveAmplitude = $state(defaults.waveAmplitude)
    let waveCycles = $state(defaults.waveCycles)

    let panelOpen = $state(true)
    let sectionEl
    let cursorEl

    // Derived cursor style for body
    let bodyCursor = $derived(
        cursorMode === 'none' || cursorMode === 'dot' || cursorMode === 'ring' || cursorMode === 'glow'
            ? 'none'
            : cursorMode === 'crosshair'
                ? 'crosshair'
                : 'default'
    )

    function reset() {
        shape = defaults.shape
        style = defaults.style
        densityMode = defaults.densityMode
        cursorMode = defaults.cursorMode
        rings = defaults.rings
        needlesPerRing = defaults.needlesPerRing
        ringSpacing = defaults.ringSpacing
        effectRadius = defaults.effectRadius
        brightnessRadius = defaults.brightnessRadius
        needleLength = defaults.needleLength
        needleWidth = defaults.needleWidth
        gridCols = defaults.gridCols
        gridRows = defaults.gridRows
        gridGap = defaults.gridGap
        starPoints = defaults.starPoints
        hexRings = defaults.hexRings
        waveAmplitude = defaults.waveAmplitude
        waveCycles = defaults.waveCycles
    }

    // Calculate elements per ring based on density mode
    function getCountForRing(ring, totalRings) {
        const useProportional = densityMode === 'proportional' ||
            (densityMode === 'auto' && style !== 'needle')

        if (useProportional) {
            // Scale count with circumference - inner rings get fewer elements
            return Math.max(6, Math.round(needlesPerRing * ring / totalRings))
        }
        return needlesPerRing
    }

    function generateNeedles() {
        const needles = []

        if (shape === 'circle') {
            for (let ring = 1; ring <= rings; ring++) {
                const radius = ring * ringSpacing
                const count = getCountForRing(ring, rings)
                for (let i = 0; i < count; i++) {
                    const angle = (i / count) * Math.PI * 2
                    needles.push({ x: Math.cos(angle) * radius, y: Math.sin(angle) * radius })
                }
            }
        } else if (shape === 'rectangle') {
            const totalWidth = (gridCols - 1) * (20 + gridGap)
            const totalHeight = (gridRows - 1) * (20 + gridGap)
            for (let row = 0; row < gridRows; row++) {
                for (let col = 0; col < gridCols; col++) {
                    needles.push({
                        x: col * (20 + gridGap) - totalWidth / 2,
                        y: row * (20 + gridGap) - totalHeight / 2
                    })
                }
            }
        } else if (shape === 'spiral') {
            const totalNeedles = rings * needlesPerRing
            for (let i = 0; i < totalNeedles; i++) {
                const angle = (i / needlesPerRing) * Math.PI * 2
                const radius = (i / totalNeedles) * rings * ringSpacing
                needles.push({ x: Math.cos(angle) * radius, y: Math.sin(angle) * radius })
            }
        } else if (shape === 'star') {
            for (let ring = 1; ring <= rings; ring++) {
                const radius = ring * ringSpacing
                const basePoints = Math.round(starPoints * getCountForRing(ring, rings) / needlesPerRing)
                const points = Math.max(3, basePoints)
                for (let i = 0; i < points * 2; i++) {
                    const angle = (i / (points * 2)) * Math.PI * 2
                    const r = i % 2 === 0 ? radius : radius * 0.5
                    needles.push({ x: Math.cos(angle) * r, y: Math.sin(angle) * r })
                }
            }
        } else if (shape === 'hexagon') {
            for (let ring = 0; ring <= hexRings; ring++) {
                if (ring === 0) {
                    needles.push({ x: 0, y: 0 })
                } else {
                    const sideLength = ring
                    for (let side = 0; side < 6; side++) {
                        const angleStart = (side / 6) * Math.PI * 2 - Math.PI / 2
                        const angleEnd = ((side + 1) / 6) * Math.PI * 2 - Math.PI / 2
                        for (let i = 0; i < sideLength; i++) {
                            const t = i / sideLength
                            const x = Math.cos(angleStart) * ring * ringSpacing * (1 - t) + Math.cos(angleEnd) * ring * ringSpacing * t
                            const y = Math.sin(angleStart) * ring * ringSpacing * (1 - t) + Math.sin(angleEnd) * ring * ringSpacing * t
                            needles.push({ x, y })
                        }
                    }
                }
            }
        } else if (shape === 'wave') {
            const cols = gridCols
            const rows = gridRows
            const width = cols * 25
            const height = rows * 25
            for (let row = 0; row < rows; row++) {
                for (let col = 0; col < cols; col++) {
                    const x = (col / (cols - 1)) * width - width / 2
                    const baseY = (row / (rows - 1)) * height - height / 2
                    const waveOffset = Math.sin((col / cols) * Math.PI * 2 * waveCycles) * waveAmplitude
                    needles.push({ x, y: baseY + waveOffset })
                }
            }
        } else if (shape === 'diamond') {
            for (let ring = 1; ring <= rings; ring++) {
                const size = ring * ringSpacing
                const count = getCountForRing(ring, rings)
                const pointsPerEdge = Math.max(2, Math.ceil(count / 4))

                for (let i = 0; i < pointsPerEdge; i++) {
                    const t = i / pointsPerEdge
                    // Top to Right edge
                    needles.push({ x: size * t, y: size * (1 - t) })
                    // Right to Bottom edge
                    needles.push({ x: size * (1 - t), y: -size * t })
                    // Bottom to Left edge
                    needles.push({ x: -size * t, y: -size * (1 - t) })
                    // Left to Top edge
                    needles.push({ x: -size * (1 - t), y: size * t })
                }
            }
        } else if (shape === 'scatter') {
            const count = rings * needlesPerRing
            const radius = rings * ringSpacing
            for (let i = 0; i < count; i++) {
                const r = Math.sqrt(Math.random()) * radius
                const angle = Math.random() * Math.PI * 2
                needles.push({ x: Math.cos(angle) * r, y: Math.sin(angle) * r })
            }
        }

        return needles
    }

    let needles = $derived(generateNeedles())

    $effect(() => {
        document.body.style.cursor = bodyCursor
    })

    onMount(() => {
        function step() {
            const divs = sectionEl?.querySelectorAll('div.needle')
            if (!divs) {
                window.requestAnimationFrame(step)
                return
            }

            divs.forEach((arrow) => {
                const rect = arrow.getBoundingClientRect()
                const x = rect.x + rect.width/2
                const y = rect.y + rect.height/2

                const dx = mouse.x - x
                const dy = mouse.y - y

                const rot = Math.atan2(dy, dx) * (180 / Math.PI)
                const dist = Math.sqrt(dx ** 2 + dy ** 2)

                const scale = 1 + Math.max(0, 1 - dist / effectRadius)
                arrow.children[0].style.transform = `rotate(${rot + 90}deg) scale(${scale})`
                arrow.style.color = `rgb(255 255 255 / ${Math.max(0.15, (1-dist/brightnessRadius))})`
            })

            window.requestAnimationFrame(step)
        }

        window.requestAnimationFrame(step)
    })

    function handleMouseMove(e) {
        mouse.x = e.pageX
        mouse.y = e.pageY
        if (cursorEl) {
            cursorEl.style.transform = `translate(${e.clientX}px, ${e.clientY}px)`
        }
    }
</script>

<style>
    :global(body) {
        background: #000;
        display: grid;
        place-items: center;
        min-height: 100vh;
        margin: 0;
    }

    .custom-cursor {
        position: fixed;
        top: 0;
        left: 0;
        pointer-events: none;
        z-index: 9999;
        will-change: transform;
    }

    .cursor-dot-style {
        width: 8px;
        height: 8px;
        background: #fff;
        border-radius: 50%;
        margin: -4px 0 0 -4px;
    }

    .cursor-ring-style {
        width: 24px;
        height: 24px;
        border: 1px solid rgba(255, 255, 255, 0.8);
        border-radius: 50%;
        margin: -12px 0 0 -12px;
    }

    .cursor-glow-style {
        width: 40px;
        height: 40px;
        background: radial-gradient(circle, rgba(255,255,255,0.3) 0%, transparent 70%);
        border-radius: 50%;
        margin: -20px 0 0 -20px;
    }

    @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600&display=swap');

    .panel {
        position: fixed;
        top: 16px;
        right: 16px;
        z-index: 100;
        font-family: 'JetBrains Mono', monospace;
        font-size: 10px;
        -webkit-font-smoothing: antialiased;
    }

    .panel-toggle {
        position: absolute;
        top: 0;
        right: 0;
        width: 24px;
        height: 24px;
        background: rgba(255, 255, 255, 0.03);
        border: 1px solid rgba(255, 255, 255, 0.2);
        color: rgba(255, 255, 255, 0.4);
        cursor: pointer;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 12px;
        transition: all 0.2s ease;
        backdrop-filter: blur(12px);
    }

    .panel-toggle:hover {
        background: rgba(255, 255, 255, 0.06);
        border-color: rgba(255, 255, 255, 0.35);
        color: rgba(255, 255, 255, 0.7);
    }

    .controls {
        background: rgba(12, 12, 12, 0.9);
        border: 1px solid rgba(255, 255, 255, 0.15);
        padding: 10px;
        color: rgba(255, 255, 255, 0.7);
        width: 164px;
        margin-top: 32px;
        backdrop-filter: blur(20px);
    }

    .controls.hidden {
        display: none;
    }

    .section-title {
        font-size: 8px;
        font-weight: 600;
        text-transform: uppercase;
        letter-spacing: 0.8px;
        color: rgba(255, 255, 255, 0.25);
        margin: 10px 0 6px 0;
        padding-left: 2px;
    }

    .section-title:first-child {
        margin-top: 0;
    }

    .labeled-select {
        display: flex;
        align-items: center;
        justify-content: space-between;
        margin-bottom: 6px;
    }

    .labeled-select .select-label {
        font-size: 10px;
        color: rgba(255, 255, 255, 0.5);
    }

    .labeled-select select {
        width: 90px;
        margin-bottom: 0;
    }

    .control-row {
        display: flex;
        align-items: center;
        justify-content: space-between;
        height: 20px;
        padding: 0 2px;
    }

    .control-row label {
        color: rgba(255, 255, 255, 0.5);
        font-size: 10px;
    }

    .control-row .value {
        color: rgba(255, 255, 255, 0.3);
        font-variant-numeric: tabular-nums;
        font-size: 9px;
        min-width: 32px;
        text-align: right;
    }

    input[type="range"] {
        width: 100%;
        height: 1px;
        background: rgba(255, 255, 255, 0.15);
        outline: none;
        -webkit-appearance: none;
        margin: 2px 0 8px 0;
    }

    input[type="range"]::-webkit-slider-thumb {
        -webkit-appearance: none;
        width: 4px;
        height: 12px;
        background: rgba(255, 255, 255, 0.85);
        cursor: pointer;
        transition: all 0.15s ease;
    }

    input[type="range"]::-webkit-slider-thumb:hover {
        background: #fff;
        height: 14px;
    }

    input[type="range"]::-webkit-slider-thumb:active {
        height: 10px;
    }

    select {
        width: 100%;
        padding: 5px 6px;
        background: rgba(255, 255, 255, 0.04);
        border: 1px solid rgba(255, 255, 255, 0.15);
        color: rgba(255, 255, 255, 0.7);
        font-size: 10px;
        font-family: inherit;
        cursor: pointer;
        margin-bottom: 6px;
        transition: all 0.15s ease;
    }

    select:hover {
        background: rgba(255, 255, 255, 0.06);
        border-color: rgba(255, 255, 255, 0.25);
    }

    select:focus {
        outline: none;
        border-color: rgba(255, 255, 255, 0.35);
        background: rgba(255, 255, 255, 0.08);
    }

    select option {
        background: #111;
        color: #fff;
        padding: 4px;
    }

    .hint {
        font-size: 8px;
        color: rgba(255, 255, 255, 0.2);
        margin: -2px 0 6px 2px;
        line-height: 1.3;
    }

    .reset-btn {
        width: 100%;
        padding: 6px;
        background: transparent;
        border: 1px solid rgba(255, 255, 255, 0.15);
        color: rgba(255, 255, 255, 0.35);
        font-size: 8px;
        font-weight: 500;
        font-family: inherit;
        text-transform: uppercase;
        letter-spacing: 1px;
        cursor: pointer;
        transition: all 0.15s ease;
        margin-top: 8px;
    }

    .reset-btn:hover {
        background: rgba(255, 255, 255, 0.04);
        border-color: rgba(255, 255, 255, 0.3);
        color: rgba(255, 255, 255, 0.6);
    }

    .logo {
        position: fixed;
        top: 20px;
        left: 20px;
        z-index: 100;
        font-family: 'JetBrains Mono', monospace;
        font-size: 10px;
        font-weight: 500;
        letter-spacing: 1.5px;
        color: rgba(255, 255, 255, 0.7);
        text-transform: uppercase;
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid rgba(255, 255, 255, 0.1);
        padding: 8px 12px;
        user-select: none;
    }

    .credits {
        position: fixed;
        bottom: 20px;
        left: 24px;
        z-index: 100;
        font-family: 'JetBrains Mono', monospace;
        display: flex;
        align-items: center;
        gap: 8px;
        font-size: 10px;
    }

    .credits-text {
        color: rgba(255, 255, 255, 0.25);
    }

    .credits .heart {
        color: #e84057;
        font-size: 9px;
    }

    .credits-name {
        color: rgba(255, 255, 255, 0.5);
        text-decoration: none;
        transition: color 0.2s ease;
        border-bottom: 1px solid transparent;
    }

    .credits-name:hover {
        color: rgba(255, 255, 255, 0.9);
        border-bottom-color: rgba(255, 255, 255, 0.3);
    }

    .github-link {
        display: flex;
        align-items: center;
        color: rgba(255, 255, 255, 0.25);
        margin-left: 4px;
        transition: color 0.2s ease;
    }

    .github-link:hover {
        color: rgba(255, 255, 255, 0.8);
    }

    section {
        position: relative;
        width: 800px;
        height: 800px;
    }

    main * {
        pointer-events: none;
    }

    .panel, .panel * {
        pointer-events: auto;
        cursor: default !important;
    }

    .needle {
        position: absolute;
        left: 50%;
        top: 50%;
    }

    .needle span {
        display: block;
    }

    .shape-needle {
        background: currentColor;
    }

    .shape-dot {
        background: currentColor;
        border-radius: 50%;
    }

    .shape-ring {
        background: transparent;
        border: solid currentColor;
        border-radius: 50%;
        box-sizing: border-box;
    }

    .shape-cross {
        width: var(--size);
        height: var(--size);
        position: relative;
    }

    .shape-cross::before,
    .shape-cross::after {
        content: '';
        position: absolute;
        background: currentColor;
    }

    .shape-cross::before {
        width: var(--thickness);
        height: 100%;
        left: 50%;
        transform: translateX(-50%);
    }

    .shape-cross::after {
        width: 100%;
        height: var(--thickness);
        top: 50%;
        transform: translateY(-50%);
    }

    .shape-square {
        background: currentColor;
    }

    .shape-dash {
        background: currentColor;
    }
</style>

<svelte:window onmousemove={handleMouseMove}/>

{#if cursorMode === 'dot' || cursorMode === 'ring' || cursorMode === 'glow'}
    <div class="custom-cursor" bind:this={cursorEl}>
        <div class={cursorMode === 'dot' ? 'cursor-dot-style' : cursorMode === 'ring' ? 'cursor-ring-style' : 'cursor-glow-style'}></div>
    </div>
{/if}

<div class="panel">
    <button class="panel-toggle" onclick={() => panelOpen = !panelOpen}>
        {panelOpen ? '×' : '≡'}
    </button>

    <div class="controls" class:hidden={!panelOpen}>
        <div class="section-title">Appearance</div>
        <div class="labeled-select">
            <span class="select-label">Shape</span>
            <select bind:value={shape}>
                <option value="circle">Circle</option>
                <option value="rectangle">Grid</option>
                <option value="spiral">Spiral</option>
                <option value="star">Star</option>
                <option value="hexagon">Hex</option>
                <option value="wave">Wave</option>
                <option value="diamond">Diamond</option>
                <option value="scatter">Scatter</option>
            </select>
        </div>
        <div class="labeled-select">
            <span class="select-label">Style</span>
            <select bind:value={style}>
                <option value="needle">Needle</option>
                <option value="dot">Dot</option>
                <option value="ring">Ring</option>
                <option value="cross">Cross</option>
                <option value="square">Square</option>
                <option value="dash">Dash</option>
            </select>
        </div>
        <div class="labeled-select">
            <span class="select-label">Cursor</span>
            <select bind:value={cursorMode}>
                <option value="default">Default</option>
                <option value="none">Hidden</option>
                <option value="crosshair">Cross</option>
                <option value="dot">Dot</option>
                <option value="ring">Ring</option>
                <option value="glow">Glow</option>
            </select>
        </div>
        <div class="labeled-select">
            <span class="select-label">Density</span>
            <select bind:value={densityMode}>
                <option value="auto">Auto</option>
                <option value="uniform">Uniform</option>
                <option value="proportional">Scaled</option>
            </select>
        </div>

        <div class="section-title">Layout</div>
        {#if shape === 'circle' || shape === 'spiral' || shape === 'star' || shape === 'diamond' || shape === 'scatter'}
            <div class="control-row"><label>Rings</label><span class="value">{rings}</span></div>
            <input type="range" min="2" max="15" bind:value={rings}>
            <div class="control-row"><label>Count</label><span class="value">{needlesPerRing}</span></div>
            <input type="range" min="12" max="72" step="6" bind:value={needlesPerRing}>
            <div class="control-row"><label>Gap</label><span class="value">{ringSpacing}</span></div>
            <input type="range" min="15" max="60" bind:value={ringSpacing}>
        {/if}
        {#if shape === 'star'}
            <div class="control-row"><label>Points</label><span class="value">{starPoints}</span></div>
            <input type="range" min="3" max="16" bind:value={starPoints}>
        {/if}
        {#if shape === 'hexagon'}
            <div class="control-row"><label>Rings</label><span class="value">{hexRings}</span></div>
            <input type="range" min="2" max="10" bind:value={hexRings}>
            <div class="control-row"><label>Gap</label><span class="value">{ringSpacing}</span></div>
            <input type="range" min="20" max="80" bind:value={ringSpacing}>
        {/if}
        {#if shape === 'rectangle' || shape === 'wave'}
            <div class="control-row"><label>Cols</label><span class="value">{gridCols}</span></div>
            <input type="range" min="5" max="30" bind:value={gridCols}>
            <div class="control-row"><label>Rows</label><span class="value">{gridRows}</span></div>
            <input type="range" min="3" max="20" bind:value={gridRows}>
        {/if}
        {#if shape === 'rectangle'}
            <div class="control-row"><label>Gap</label><span class="value">{gridGap}</span></div>
            <input type="range" min="5" max="30" bind:value={gridGap}>
        {/if}
        {#if shape === 'wave'}
            <div class="control-row"><label>Amp</label><span class="value">{waveAmplitude}</span></div>
            <input type="range" min="10" max="100" bind:value={waveAmplitude}>
            <div class="control-row"><label>Waves</label><span class="value">{waveCycles}</span></div>
            <input type="range" min="1" max="6" bind:value={waveCycles}>
        {/if}

        <div class="section-title">Effect</div>
        <div class="control-row"><label>Force</label><span class="value">{effectRadius}</span></div>
        <input type="range" min="100" max="800" step="50" bind:value={effectRadius}>
        <div class="control-row"><label>Glow</label><span class="value">{brightnessRadius}</span></div>
        <input type="range" min="200" max="1000" step="50" bind:value={brightnessRadius}>

        <div class="section-title">Element</div>
        <div class="control-row"><label>Size</label><span class="value">{needleLength}</span></div>
        <input type="range" min="3" max="30" bind:value={needleLength}>
        <div class="control-row"><label>Weight</label><span class="value">{needleWidth}</span></div>
        <input type="range" min="1" max="8" bind:value={needleWidth}>

        <button class="reset-btn" onclick={reset}>Reset</button>
    </div>
</div>

<div class="logo">30 DAYS × AI</div>

<div class="credits">
    <span class="credits-text">made with <span class="heart">♥</span> by</span>
    <a href="https://linkedin.com/in/oyindamolaakinleye" target="_blank" rel="noopener" class="credits-name">Oyindamola Akinleye</a>
    <a href="https://github.com/Dammyjay93/needle-field" target="_blank" rel="noopener" class="github-link" aria-label="View source on GitHub">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
            <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
        </svg>
    </a>
</div>

<main>
    <section bind:this={sectionEl}>
        {#each needles as needle}
            <div class="needle" style="transform: translate({needle.x}px, {needle.y}px)">
                {#if style === 'needle'}
                    <span class="shape-needle" style="height: {needleLength}px; width: {needleWidth}px;"></span>
                {:else if style === 'dot'}
                    <span class="shape-dot" style="width: {needleLength}px; height: {needleLength}px;"></span>
                {:else if style === 'ring'}
                    <span class="shape-ring" style="width: {needleLength}px; height: {needleLength}px; border-width: {needleWidth}px;"></span>
                {:else if style === 'cross'}
                    <span class="shape-cross" style="--size: {needleLength}px; --thickness: {needleWidth}px;"></span>
                {:else if style === 'square'}
                    <span class="shape-square" style="width: {needleLength}px; height: {needleLength}px;"></span>
                {:else if style === 'dash'}
                    <span class="shape-dash" style="width: {needleLength}px; height: {needleWidth}px;"></span>
                {/if}
            </div>
        {/each}
    </section>
</main>
