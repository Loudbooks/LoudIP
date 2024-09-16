<script lang="ts">
	import { onMount } from "svelte";

    const fontSize = 100;

    let ip: string;
    let location: string;
    let isp: string;
    let amountNeededHorizontal: number;
    let amountNeededVertical: number;

    let background: HTMLDivElement;
    let mainText: HTMLDivElement;
    let locationBlock: HTMLHeadingElement;
    let ispBlock: HTMLHeadingElement;
    let copied: SVGSVGElement;

    let loadingPromise: Promise<void> = new Promise(() => {});

    onMount(async () => {
        let ipPromise = fetch('https://api64.ipify.org?format=json')
            .then(response => response.json())
            .then(async data => {
                ip = data.ip;

                let ispPromise = fetch(`https://ip.loudbook.dev/api/${ip}`)
                    .then(response => response.json())
                    .then(data => {
                        console.log(data);
                        location = `${data.city}, ${data.regionName}`;
                        isp = data.isp;
                    });

                document.title = ip;

                amountNeededHorizontal = window.innerWidth / (ip.length * (fontSize * 0.2));
                amountNeededVertical = window.innerHeight / fontSize;

                await ispPromise;
            });

        Promise.all([ipPromise]).then(() => {
            loadingPromise = Promise.resolve();
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

        setTimeout(() => {
            locationBlock.style.opacity = "1";
            locationBlock.style.transform = "translateY(0)";
        }, children.length * 30 + 200);

        setTimeout(() => {
            ispBlock.style.opacity = "1";
            ispBlock.style.transform = "translateY(0)";
        }, children.length * 30 + 400);
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
        <div bind:this={locationBlock} class="info-block" id="location-block">
            <svg width="800px" height="800px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" class="info-icon">
                <path d="M19.5 6L18.0333 7.1C17.6871 7.35964 17.2661 7.5 16.8333 7.5H13.475C12.8775 7.5 12.3312 7.83761 12.064 8.37206V8.37206C11.7342 9.03161 11.9053 9.83161 12.476 10.2986L14.476 11.9349C16.0499 13.2227 16.8644 15.22 16.6399 17.2412L16.5936 17.6577C16.5314 18.2177 16.4102 18.7695 16.232 19.304L16 20" stroke="#FFFFFF" stroke-width="2"/>
                <path d="M2.5 10.5L5.7381 9.96032C7.09174 9.73471 8.26529 10.9083 8.03968 12.2619L7.90517 13.069C7.66434 14.514 8.3941 15.9471 9.70437 16.6022V16.6022C10.7535 17.1268 11.2976 18.3097 11.0131 19.4476L10.5 21.5" stroke="#FFFFFF" stroke-width="2"/>
                <circle cx="12" cy="12" r="9" stroke="#FFFFFF" stroke-width="2"/>
            </svg>
            <h1 class="info-text">{location}</h1>
        </div>

        <div bind:this={ispBlock} class="info-block" id="isp-block">
            <svg width="800px" height="800px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" class="info-icon">
                <g id="Navigation / Building_04">
                    <path id="Vector" d="M2 20H4M4 20H14M4 20V6.2002C4 5.08009 4 4.51962 4.21799 4.0918C4.40973 3.71547 4.71547 3.40973 5.0918 3.21799C5.51962 3 6.08009 3 7.2002 3H10.8002C11.9203 3 12.4796 3 12.9074 3.21799C13.2837 3.40973 13.5905 3.71547 13.7822 4.0918C14 4.5192 14 5.07899 14 6.19691V12M14 20H20M14 20V12M20 20H22M20 20V12C20 11.0681 19.9999 10.6024 19.8477 10.2349C19.6447 9.74481 19.2557 9.35523 18.7656 9.15224C18.3981 9 17.9316 9 16.9997 9C16.0679 9 15.6019 9 15.2344 9.15224C14.7443 9.35523 14.3552 9.74481 14.1522 10.2349C14 10.6024 14 11.0681 14 12M7 10H11M7 7H11" stroke="#FFFFFF" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                </g>
            </svg>
            <h1 class="info-text">{isp}</h1>
        </div>

        <svg width="800px" height="800px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" bind:this={copied} class="copied-icon">
            <path d="M8 5.00005C7.01165 5.00082 6.49359 5.01338 6.09202 5.21799C5.71569 5.40973 5.40973 5.71569 5.21799 6.09202C5 6.51984 5 7.07989 5 8.2V17.8C5 18.9201 5 19.4802 5.21799 19.908C5.40973 20.2843 5.71569 20.5903 6.09202 20.782C6.51984 21 7.07989 21 8.2 21H15.8C16.9201 21 17.4802 21 17.908 20.782C18.2843 20.5903 18.5903 20.2843 18.782 19.908C19 19.4802 19 18.9201 19 17.8V8.2C19 7.07989 19 6.51984 18.782 6.09202C18.5903 5.71569 18.2843 5.40973 17.908 5.21799C17.5064 5.01338 16.9884 5.00082 16 5.00005M8 5.00005V7H16V5.00005M8 5.00005V4.70711C8 4.25435 8.17986 3.82014 8.5 3.5C8.82014 3.17986 9.25435 3 9.70711 3H14.2929C14.7456 3 15.1799 3.17986 15.5 3.5C15.8201 3.82014 16 4.25435 16 4.70711V5.00005M12 11H9M15 15H9" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
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
        flex-wrap: nowrap;
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

    .info-block {
        opacity: 0;
        transform: translateY(100%);
        transition: opacity 0.5s ease, transform 0.5s ease;

        display: flex;
        align-items: center;
        gap: 10px;
    }

    .info-text {
        font-family: Gabarito;
        font-size: 2vw;
        font-weight: 1;
        color: white;
        opacity: 1;
        transform: none;
        transition: opacity 0.5s ease, transform 0.5s ease;
    }

    h1 {
        font-size: 15vw;
        font-weight: 1000;
        padding: 0;
        display: inline-block;
        white-space: nowrap;
        opacity: 0;
        transform: translateY(100%);
        transition: opacity 0.5s ease, transform 0.5s ease, margin 0.5s ease;
        
        margin: -1px;
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

    .info-icon {
        height: 2.2vw;
        width: 2.2vw;
        margin: 0;
        padding: 0;
    }

    .copied-icon {
        height: 2.2vw;
        width: 2.2vw;
        min-height: 30px;
        min-width: 30px;

        margin: 0;
        padding: 0;
        position: absolute;
        bottom: 100px;
        opacity: 0;
        transform: scale(0.8);

        transition: opacity 0.3s ease, transform 0.5s ease;
    }
</style>