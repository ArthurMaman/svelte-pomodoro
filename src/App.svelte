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
</style>

<script>
    import Tailwindcss from "./Tailwindcss.svelte";
    import { fly } from "svelte/transition";

    function typewriter(node, { speed = 50 }) {
        const valid =
            node.childNodes.length === 1 && node.childNodes[0].nodeType === 3;

        if (!valid) {
            throw new Error(
                `This transition only works on elements with a single text node child`
            );
        }

        const text = node.textContent;
        const duration = text.length * speed;

        return {
            duration,
            tick: t => {
                const i = ~~(text.length * t);
                node.textContent = text.slice(0, i);
            }
        };
    }

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
        if (e.target.value !== 0 && e.target.value < workTime) {
            restTime = e.target.value;
        } else {
            alert("You need to work more than you rest :(");
            tmpRest = restTime;
        }
    };

    const handleWorkTimeChange = e => {
        if (e.target.value !== 0 && e.target.value > restTime) {
            workTime = e.target.value;
        } else {
            alert("You need to work more than you rest :(");
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
    let workedPeriod = 0;
    let restedPeriod = 0;
    $: innerPathAnimated = innerPath;
    $: outerPathAnimated = outerPath;
    $: currentAngle = outerAngle;
    let deltaAngle = 0;
    let timeLeft = workTime;

    let timer = "";
    let timer2 = "";
    let running = false;
    const startWorking = currentAngleP => {
        if (running === true) return;
        timeLeft = workTime;
        running = true;
        innerPathAnimated = innerPath;
        disabled = true;
        deltaAngle = (outerAngle * 50) / (workTime * 60000);
        currentAngle = currentAngleP;
        timer2 = window.setInterval(() => {
            timeLeft = timeLeft - 1;
            console.log(timeLeft);
        }, 60000);
        timer = window.setInterval(() => {
            currentAngle = currentAngle - deltaAngle;
            outerPathAnimated = createArc(
                startOuterAngle,
                startOuterAngle + currentAngle,
                { x: 200, y: 200 },
                150
            );
            if (currentAngle >= 0) {
                workedPeriod = workedPeriod + 1;
                window.clearInterval(timer);
                startResting(innerAngle);
            }
        }, 50);
    };

    const startResting = currentAngleP => {
        outerPathAnimated = outerPath;
        timeLeft = restTime;
        deltaAngle = (innerAngle * 50) / (restTime * 60000);
        currentAngle = currentAngleP;
        timer = window.setInterval(() => {
            currentAngle = currentAngle - deltaAngle;
            innerPathAnimated = createArc(
                startInnerAngle,
                startInnerAngle + currentAngle,
                { x: 200, y: 200 },
                110
            );
            if (currentAngle >= 0) {
                restedPeriod = restedPeriod + 1;
                window.clearInterval(timer);
                running = false;
                startWorking(outerAngle);
            }
        }, 50);
    };

    let paused = false;
    const pause = () => {
        if (!paused) {
            paused = true;
            window.clearInterval(timer);
        } else {
            paused = false;
            if (workedPeriod > restedPeriod) {
                startResting(currentAngle);
            } else {
                running = false;
                startWorking(currentAngle);
            }
        }
    };

    const end = () => {
        window.clearInterval(timer);
        window.clearInterval(timer2);
        outerPathAnimated = outerPath;
        innerPathAnimated = innerPath;
        workedPeriod = 0;
        restedPeriod = 0;
        paused = false;
        running = false;
        disabled = false;
        alert("Good work !");
    };
</script>

<Tailwindcss />
<div class="flex justify-around">
    <div class="flex flex-col max-w-lg align-middle object-right">
        <h1 class="object-top self-center pb-5 pt-4 font-bold text-2xl italic">
            Svelte Pomodoro
        </h1>
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
        <svg viewBox="0 0 400 400" class="self-center">
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
                d="{innerPathAnimated !== 'null' ? innerPathAnimated : 'M0 0'}"
                stroke="#FE5F55"
                fill="none"
                stroke-width="25px"
                stroke-linecap="round"
            ></path>
            <path
                d="{outerPathAnimated || ''}"
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
                <h1>
                    <span>{timeLeft}</span>
                    min left
                </h1>
                <h1>You're doing great!</h1>
            </div>
        {/if}
        <div class="self-center p-10">
            <button class="w-20" on:click="{() => startWorking(outerAngle)}">
                Start
            </button>
            <button class="w-20" on:click="{pause}">
                {!paused ? 'Pause' : 'Resume'}
            </button>
            <button class="w-20" on:click="{end}">End</button>
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
