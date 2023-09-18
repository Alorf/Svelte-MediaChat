<script lang="ts">
    import TextData from "./Textes.svelte";

    export let textData: TextData;
</script>

<span id="moveable" style="color: {textData.color}; font-family: {textData.font}; font-size: {textData.fontSize}px;">
    {textData.text}
</span>

<style>
    #moveable {
        position: absolute;
        /* text nowrap */
        /*padding-left: 1px;*/
        overflow-wrap: anywhere;
        z-index: 99;
        font-weight: bold;
        -webkit-text-stroke-width: 3px;
        -webkit-text-stroke-color: black;
        line-height: 100%;
    }
</style>