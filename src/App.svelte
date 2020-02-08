<script>
    import Tailwindcss from "./Tailwindcss.svelte";
    import { fly } from "svelte/transition";

    const getPointOnCircle = (angle, center, rayon) => {
        const radAngle = (angle * Math.PI) / 180;
        return {
            x: center.x + rayon * Math.cos(radAngle),
            y: center.y - rayon * Math.sin(radAngle)
        };
    };

    const createArc = (start, end, center, ray) => {
        if (start === end) return "null";
        const startP = getPointOnCircle(start, center, ray);
        const endP = getPointOnCircle(end, center, ray);
        const fa = Math.abs(end - start) > 180 ? 1 : 0;
        const fs = end - start > 0 ? 1 : 0;
        return `M${endP.x} ${endP.y} A${ray} ${ray} 0 ${fa} ${fs} ${startP.x} ${startP.y}`;
    };

    const handleRestTimeChange = e => {
        const value = parseInt(e.target.value);
        if (value < workTime && value > 0) {
            restTime = e.target.value;
        } else if (value <= 0) {
            openPopup("You should rest at least a little.", ":(")
            tmpRest = restTime;
        } else {
            openPopup("You should work more than you rest.", "Relou")
            tmpRest = restTime;
        }
    };

    const handleWorkTimeChange = e => {
        if (e.target.value !== 0 && e.target.value > restTime) {
            workTime = e.target.value;
        } else {
            openPopup("You should work more than you rest.", "Relou")
            tmpWork = workTime;
        }
    };

    let outerAngle = -270;
    let startOuterAngle = 225;
    let restTime = 10;
    let workTime = 30;
    $: tmpWork = workTime;
    $: tmpRest = restTime;
    $: innerAngle = (outerAngle * restTime) / workTime;
    $: startInnerAngle = startOuterAngle + (outerAngle - innerAngle) / 2;

    $: outerPath = createArc(
        startOuterAngle,
        startOuterAngle + outerAngle,
        { x: 200, y: 200 },
        150
    );
    $: innerPath = createArc(
        startInnerAngle,
        startInnerAngle + innerAngle,
        { x: 200, y: 200 },
        110
    );

    let disabled = false;
    let running = false;

    let workedPeriod = 0;
    let restedPeriod = 0;

    let timer = "";
    let timer2 = "";

    let interval = 50;
    $: workPas = interval / (workTime * 60 * 1000);
    $: restPas = interval / (restTime * 60 * 1000);
    let restProgression = 1;
    let workProgression = 1;
    let working = true;

    $: workLeft = Math.round(workTime * workProgression);
    $: restLeft = Math.round(restTime * restProgression);

    $: currentWorkPath = createArc(
        startOuterAngle,
        startOuterAngle + outerAngle * workProgression,
        { x: 200, y: 200 },
        150
    );

    $: currentRestPath = createArc(
        startInnerAngle,
        startInnerAngle + innerAngle * restProgression,
        { x: 200, y: 200 },
        110
    );

    const begin = () => {
        if (running) return;
        running = true;
        startWorking();
    };

    const startWorking = () => {
        running = true;
        disabled = true;
        working = true;
        timer = window.setInterval(() => {
            workProgression = workProgression - workPas;
            if (workProgression <= 0) {
                workedPeriod = workedPeriod + 1;
                restProgression = 1;
                window.clearInterval(timer);
                startResting();
            }
        }, interval);
    };

    const startResting = () => {
        working = false;
        timer = window.setInterval(() => {
            restProgression = restProgression - restPas;
            if (restProgression <= 0) {
                restedPeriod = restedPeriod + 1;
                running = false;
                workProgression = 1;
                window.clearInterval(timer);
                startWorking();
            }
        }, 50);
    };

    let paused = false;
    const pause = () => {
        if (!paused) {
            paused = true;
            window.clearInterval(timer);
            window.clearInterval(timer2);
        } else {
            paused = false;
            if (workedPeriod > restedPeriod) {
                startResting();
            } else {
                startWorking();
            }
        }
    };

    const end = () => {
        window.clearInterval(timer);
        window.clearInterval(timer2);
        workProgression = 1;
        restProgression = 1;
        workedPeriod = 0;
        restedPeriod = 0;
        paused = false;
        running = false;
        disabled = false;
        openPopup("Today was a good work session.", "Bye !")
    };

    let popup = false;
    let popupText = '';
    let popupButton = '';
    const openPopup = (txt, btn) => {
        popupText = txt;
        popupButton = btn;
        popup = true;
    }
    const closePopup = () => popup = false;
</script>

<svelte:head>
    <title>Svelte Pomodoro</title>
    <meta
        name="description"
        content="A simple, fast, no troubles, pomodoro timer"
    />
    <meta name="keywords" content="Productivity, Procrastination" />
    <meta name="author" content="Arthur Maman" />
</svelte:head>
<Tailwindcss />
<div class="flex justify-around">
    <div class="flex flex-col w-full max-w-lg align-middle object-right">
        <h1 class="object-top self-center pb-5 pt-5 font-bold text-2xl italic">
            Svelte Pomodoro
        </h1>
        {#if popup}
        <div transition:fly="{{ y: -100, duration: 300 }}" class="bg-white rounded-lg flex flex-row content-between max-w-sm w-full self-center items-center justify-around h-20 mb-5">
            <div class="whitespace-pre-wrap text-center font-mono text-base m-6">{popupText}</div>
            <button class="h-10 w-16 m-6" on:click="{closePopup}">{popupButton}</button>
        </div>
        {/if}
        <div class="p-2 self-center">
            <h2 class="inline-block w-40">Insert working time</h2>
            <input
                {disabled}
                class="inline-block w-10 text-right pr-2 work_time"
                type="value"
                bind:value="{tmpWork}"
                on:change="{handleWorkTimeChange}"
            />
            <h2 class="inline-block w-10">min</h2>
        </div>
        <div class="p-2 self-center">
            <h2 class="inline-block w-40">Insert resting time</h2>
            <input
                {disabled}
                class="inline-block w-10 text-right pr-2 rest_time"
                type="value"
                bind:value="{tmpRest}"
                on:change="{handleRestTimeChange}"
            />
            <h2 class="inline-block w-10">min</h2>
        </div>
        <svg viewBox="0 0 400 400" class="self-center svgBox">
            <path
                d="{innerPath}"
                stroke="#fe8f88"
                fill="none"
                stroke-width="25px"
                stroke-linecap="round"
            ></path>
            <path
                d="{outerPath}"
                stroke="#a5a9c5"
                fill="none"
                stroke-width="25px"
                stroke-linecap="round"
            ></path>
            <path
                d="{currentRestPath}"
                stroke="#FE5F55"
                fill="none"
                stroke-width="25px"
                stroke-linecap="round"
            ></path>
            <path
                d="{currentWorkPath}"
                stroke="#777DA7"
                fill="none"
                stroke-width="25px"
                stroke-linecap="round"
            ></path>
        </svg>
        {#if running}
            <div
                class="font-mono font-extrabold text-3xl text-center"
                in:fly="{{ y: 100, duration: 750 }}"
            >
                {#if working}
                    <h1>
                        <span>{workLeft === 0 ? 'Less than one' : workLeft}</span>
                        min left
                    </h1>
                    <h1>You're doing great!</h1>
                {:else}
                    <h1>
                        <span>{restLeft === 0 ? 'Less than one' : restLeft}</span>
                        min left
                    </h1>
                    <h1>Breathe in. Breathe out.</h1>
                {/if}
            </div>
        {/if}
        <div class="self-center p-5">
            <button class="w-20" on:click="{begin}" disabled="{running}">Start</button>
            <button class="w-20" on:click="{pause}">
                {!paused ? 'Pause' : 'Resume'}
            </button>
            <button class="w-20" on:click="{end}" disabled="{!running}">End</button>
        </div>
        <h2 class="p-1">
            You worked for {workedPeriod} period{workedPeriod > 1 ? 's' : ''}.
            <br />
            {workedPeriod > 0 ? `That's ${workedPeriod * workTime} minutes !` : ''}
        </h2>
        <h2 class="p-1">
            You rested for {restedPeriod} period{restedPeriod > 1 ? 's' : ''}.
            <br />
            {restedPeriod > 0 ? `It's only ${restedPeriod * restTime} minutes !` : ''}
        </h2>
        <h2 class="p-1 text-center">You nice, keep going.</h2>
    </div>
</div>
<style>
    :global(body) {
        background: #e1f4e3;
        color: black;
    }

    button {
        background-color: white;
    }

    input {
        border: none;
        border-bottom: black 1px solid;
        background-color: #e1f4e3;
        border-radius: 0;
        font-weight: bold;
    }

    .work_time {
        color: #777da7;
    }

    .rest_time {
        color: #fe5f55;
    }

    .svgBox {
        max-height: 400px;
    }
</style>