<script lang="ts">
	import { onMount } from "svelte";

    const fontSize = 100;

    let ip: string;
    let amountNeededHorizontal: number;
    let amountNeededVertical: number;

    let background: HTMLDivElement;
    let mainText: HTMLDivElement;
    let copied: HTMLParagraphElement;

    let loadingPromise: Promise<void> = new Promise(() => {});

    onMount(() => {
        fetch('https://api.ipify.org?format=json')
            .then(response => response.json())
            .then(data => {
                ip = data.ip;

                loadingPromise = Promise.resolve();
                document.title = ip;

                amountNeededHorizontal = window.innerWidth / (ip.length * (fontSize * 0.2));
                amountNeededVertical = window.innerHeight / fontSize;

            });
    });

    function randomize(node: HTMLElement) {
        let children = node.children;
        for (let i = 0; i < children.length; i++) {
            let row = children[i];
            for (let j = 0; j < row.children.length; j++) {
                let randomOffset = Math.random() * 255;
                (row as HTMLDivElement).style.transform = `translateX(${randomOffset}px)`;
            }
        }
    }

    function fadeInOneAtATime(node: HTMLElement) {
        let children = node.children;
        for (let i = 0; i < children.length; i++) {
            let char = children[i] as HTMLHeadingElement;

            if (char.classList.contains("invisible-selectable")) {
                continue;
            }

            if (char.textContent === ".") {
                setTimeout(() => {
                    char.style.opacity = "1";
                    char.style.transform = "translateY(0)";
                }, (i + 1) * 60);
            } else {
                setTimeout(() => {
                    char.style.opacity = "1";
                    char.style.transform = "translateY(0)";
                }, (i + 1) * 30);
            }
        }
    }

    let currentTimeout: number;

    function copyToClipboard() {
        navigator.clipboard.writeText(ip);

        copied.style.opacity = "1";
        copied.style.transform = "scale(1)";

        clearTimeout(currentTimeout);

        currentTimeout = setTimeout(() => {
            copied.style.opacity = "0";
            copied.style.transform = "scale(0.8)";
        }, 2000);

        let children = background.children;
        for (let i = 0; i < children.length; i++) {
            let row = children[i];
            for (let j = 0; j < row.children.length; j++) {
                let randomOffset = Math.random() * 255;
                (row as HTMLDivElement).style.transform = `translateX(${randomOffset}px)`;
            }
        }

        let textChildren = mainText.children;
        for (let i = 0; i < textChildren.length; i++) {
            let char = textChildren[i] as HTMLHeadingElement;

            if (char.classList.contains("invisible-selectable")) {
                continue;
            }

            setTimeout(() => {
                char.style.transform = "translateY(-15px)";

                setTimeout(() => {
                    char.style.transform = "translateY(0)";
                }, 200);
            }, i * 30);
        }
    }
</script>

{#await loadingPromise then}
    <div class="container">
        <div class="text-container" use:fadeInOneAtATime bind:this={mainText} on:click={copyToClipboard} role="button" tabindex="0" on:keydown={() => {}}> 
            {#each ip.split("") as char, i}
                <h1>{char}</h1>
            {/each}
            <h1 class="invisible-selectable">
                {ip}
            </h1>
        </div>

        <svg width="800px" height="800px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" bind:this={copied}>
            <path d="M8 5.00005C7.01165 5.00082 6.49359 5.01338 6.09202 5.21799C5.71569 5.40973 5.40973 5.71569 5.21799 6.09202C5 6.51984 5 7.07989 5 8.2V17.8C5 18.9201 5 19.4802 5.21799 19.908C5.40973 20.2843 5.71569 20.5903 6.09202 20.782C6.51984 21 7.07989 21 8.2 21H15.8C16.9201 21 17.4802 21 17.908 20.782C18.2843 20.5903 18.5903 20.2843 18.782 19.908C19 19.4802 19 18.9201 19 17.8V8.2C19 7.07989 19 6.51984 18.782 6.09202C18.5903 5.71569 18.2843 5.40973 17.908 5.21799C17.5064 5.01338 16.9884 5.00082 16 5.00005M8 5.00005V7H16V5.00005M8 5.00005V4.70711C8 4.25435 8.17986 3.82014 8.5 3.5C8.82014 3.17986 9.25435 3 9.70711 3H14.2929C14.7456 3 15.1799 3.17986 15.5 3.5C15.8201 3.82014 16 4.25435 16 4.70711V5.00005M12 11H9M15 15H9" stroke="white" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
    </div>
    <div id="background" bind:this={background} use:randomize>
        {#each {length: amountNeededVertical} as _, i}
        <div class="row">
            {#each {length: amountNeededHorizontal} as _, j}
                <span>{ip}</span>
            {/each}
            <br>
        </div>
        {/each} 
    </div>
{/await}


<style lang="scss">
    :global(body) {
        background-color: black;
    }
    :global(html) {
        overflow: hidden;
    }
    span {
        font-family: Gabarito;
        font-weight: 1000;
        font-size: 100px;
        width: fit-content;
    }
    .container {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        height: 100vh;
    }
    .text-container {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
        align-items: center;

        color: white;
        font-family: Gabarito;

        cursor: pointer;
        transition: transform 0.2s ease;

        &:hover {
            transform: translateY(-5px);
        }
    }
    .invisible-selectable {
        color: transparent;
        position: absolute;
        font-weight: 1000;
        opacity: 1;

        transform: translateY(0);
    }
    h1 {
        font-size: 15vw;
        font-weight: 1000;
        margin: -1px;
        padding: 0;
        display: inline-block;
        opacity: 0;
        transform: translateY(100%);
        transition: opacity 0.5s ease, transform 0.5s ease;
    }
    #background {
        position: fixed;
        top: 0;
        left: 0;
        z-index: -1;
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        transform: rotate(30deg) scale(1.5) translateX(-50%);
    }
    .row {
        height: 100px;

        transition: transform 0.5s ease;
    }
    .row span {
        color: rgb(255, 255, 255, 0.05);
        margin-right: 40px;
    }
    svg {
        height: min(5vw, 50px);
        width: min(5vw, 50px);
        margin: 0;
        padding: 0;
        position: absolute;
        bottom: 100px;

        opacity: 0;

        transform: scale(0.8);

        transition: opacity 0.3s ease, transform 0.5s ease;
    }
</style>