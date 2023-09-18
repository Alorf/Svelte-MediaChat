<script lang="ts">
    import type Moveable from "svelte-moveable";

    export type TextData = {
        text: string | null,
        color: string,
        font: string,
        fontSize: number
    };

    let data: TextData = {
        text: "",
        color: "#FFFFFF",
        font: "Arial",
        fontSize: 62,
    };

    export let textData: TextData;
    export let moveableText: Moveable;

    function update() {
        textData = data;

        if(moveableText){
            setTimeout(() => {
                moveableText.updateRect();
            }, 20);
        }
    }

    function updateColor() {
        textData = data;
    }

    $: update(), [data.text, data.font, data.fontSize];
    $: updateColor(), [data.color];
</script>

<div>
    <label for="text" class="label">
        <span class="label-text">Votre message</span>
    </label>
    <textarea id="text" class="textarea textarea-bordered h-12 w-full" placeholder="Bio" bind:value={data.text}></textarea>
</div>


<div>
    <label for="color" class="label">
        <span class="label-text">Couleur</span>
    </label>
    <input id="color" type="color" placeholder="Type here" class="input input-bordered w-full max-w-xs" bind:value={data.color}/>
</div>

<div>
    <label for="font" class="label">
        <span class="label-text">Police d'écriture</span>
    </label>
    <input id="font" type="text" placeholder="Type here" class="input input-bordered w-full max-w-xs" bind:value={data.font}/>
</div>

<div>
    <label for="fontSize" class="label">
        <span class="label-text">Taille</span>
    </label>
    <input id="fontSize" type="number" placeholder="Type here" class="input input-bordered w-full max-w-xs" bind:value={data.fontSize}/>
</div>
