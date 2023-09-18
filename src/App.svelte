<script lang="ts">
    import Streamer from "./lib/Streamer.svelte"
    import Textes from "./lib/Textes.svelte";
    import Moveable from "svelte-moveable";
    import TexteValue from "./lib/TexteValue.svelte";
    import axios from "axios";
    import TextData from "./lib/Textes.svelte";
    import nostreamer from "./assets/nostreamer.png";

    let dev : boolean = false;
    let envurl : string = "";

    if (dev){
        envurl = "http://localhost:3000";
    }

    let fileContainerBackground: string = nostreamer;
    let fileContainer: HTMLElement;

    let preview: HTMLElement;
    let moveable: Moveable;
    let moveableText: Moveable;

    let video: HTMLVideoElement;
    let image: HTMLImageElement;

    let isVideo: boolean = false;
    let isImage: boolean = false;
    let isText: boolean = false;
    let greenScreen: boolean = false;

    let textData: TextData = {text: "", font: "Arial", color: "#FFFFF", fontSize: 62};
    let username: string;
    let ratio: number;

    let files: FileList;
    let file: File;
    let input: HTMLInputElement;

    let link: string = "";
    let isLink: boolean = false;
    let linkData: any;

    // load media
    $: if (files) {
        file = files[0];

        try {
            if (file.type.includes("image")) {
                isVideo = false;
                isImage = true;
            } else if (file.type.includes("video")) {
                isVideo = true;
                isImage = false;
            }
        } catch (e) {
            console.log(e);

            isVideo = false;
            isImage = false;
        }

    }


    function updateLink() {
        let data : any;

        try {
            new URL(link);
        }catch (e) {
            return;
        }

        axios.get(envurl + "/video-url?url=" + link).then(response => {
            data = response.data;

            if (data['url'] === "none") {
                //writeInToast(fileToSend['message'])
                console.log("erreur")
            }

            isImage = data['type'].indexOf('image') !== -1 || data['type'].indexOf('png') !== -1;
            isLink = true;

            linkData = data;

            if (isImage) {

            } else {
                isVideo = true;
            }
        }).catch(error => {
            console.log(error)
        });
    }

    // load video
    $: if (video && isVideo) {
        video.src = isLink ? linkData["url"] : URL.createObjectURL(file);

        video.addEventListener('loadedmetadata', function () {
            preview.style.height = "";
            preview.style.width = "";

            preview.style.height = video.videoHeight + "";
            preview.style.width = video.videoWidth + "";

            if (video.videoHeight > fileContainer.offsetHeight) {
                preview.style.height = fileContainer.getBoundingClientRect().height + "px";
                preview.style.width = "auto";
            }

            moveable.updateRect();
        }, {once: true});
    }

    // load image
    $: if (image && isImage) {
        preview.style.height = "";
        preview.style.width = "";

        image.src = URL.createObjectURL(file);

        image.addEventListener('load', () => {
            preview.style.height = image.height + "";
            preview.style.width = image.width + "";

            if (image.height > fileContainer.offsetHeight) {
                preview.style.height = fileContainer.getBoundingClientRect().height + "px";
                preview.style.width = "auto";
            }

            moveable.updateRect();
        }, {once: true});
    }

    addEventListener("resize", () => {
        ratio = 1920 / fileContainer.offsetWidth;
    });

    $: (() => {
        if (textData.text === undefined) {
            isText = false;
            return;
        }

        isText = textData.text.length > 0;
    })(), [textData.text];

    async function generateData() {
        ratio = 1920 / fileContainer.offsetWidth;

        let text = {data: "", haveFile: isImage || isVideo};
        let textPositions = null;

        if (isText) {

            textPositions = moveableText.getRect();
            text = {
                "data": textData.text,
                "left": textPositions.left,
                "top": textPositions.top,
                "font_size": textData.fontSize,
                "font_family": textData.font,
                "font_color": textData.color,
                "haveFile": isImage || isVideo,
                "ratio": ratio
            };

            if (!isVideo && !isImage){
                return {"text" : text};
            }
        }

        let positions = moveable.getRect();

        let buffer = isLink ? linkData["url"] : await file.arrayBuffer();

        return {
            "data": isLink ? buffer : Array.from(new Uint8Array(buffer)),
            "fileSize": isLink ? 0 : file.size,
            "fullscreen": false,
            "width": positions.width,
            "height": positions.height,
            "left": positions.left,
            "top": positions.top,
            "timestamp": isImage ? "" : video.currentTime,
            "muted": isImage ? "" : video.muted,
            "ratio": ratio,
            "isLink": isLink,
            "haveText": isText,
            "typeFile": isLink ? linkData["type"] : file.type,
            "anonymous": true,
            "greenscreen": greenScreen,
            "user": username,
            "text": text
        };
    }

    async function send() {
        if (isImage || isVideo) {
            let data = await generateData();

            console.log(data);

            await axios.post(envurl + "/sendFile", data, {
                headers: {
                    'Content-Type': 'application/json'
                }
            });
        }else if (isText) {
            let data = await generateData();

            console.log(data);

            await axios.post(envurl + "/sendText", data, {
                headers: {
                    'Content-Type': 'application/json'
                }
            });
        }
    }

    async function skip() {
        let data = {"user": username};

        await axios.post(envurl + "/skip", data, {
            headers: {
                'Content-Type': 'application/json'
            }
        });
    }

    async function flush() {
        let data = {"user": username};

        await axios.post(envurl + "/flush", data, {
            headers: {
                'Content-Type': 'application/json'
            }
        });

        clean();
    }

    function clean() {
        console.log("clean");

        input.value = "";

        isImage = false;
        isVideo = false;
        textData = {};
    }

</script>

<main>
    <div class="flex overflow-x-hidden">
        <div class="p-10 h-screen w-1/4 min-h-full max-h-full bg-base-200 text-base-content flex overflow-auto flex-col form-control justify-between">

            <div>
                <label for="file" class="label">
                    <span class="label-text">Fichier</span>
                </label>
                <input id="file" bind:this={input} bind:files type="file"
                       class="file-input file-input-bordered w-full max-w-xs"/>
            </div>


            <div>
                <label for="link" class="label">
                    <span class="label-text">Lien d'un média</span>
                </label>
                <input id="link" type="text" bind:value={link} on:blur={updateLink} placeholder="Lien d'un média"
                       class="input input-bordered w-full max-w-xs"/>
            </div>

            <Textes bind:textData={textData} {moveableText}/>

            <div>
                <label for="discord" class="label">
                    <span class="label-text">Username Discord</span>
                </label>
                <input id="discord" type="text" placeholder="Username discord" bind:value={username}
                       class="input input-bordered w-full max-w-xs"/>
            </div>

            <div>
                <label for="greenscreen" class="label cursor-pointer">
                    <span class="label-text">GreenScreen</span>
                    <input id="greenscreen" type="checkbox" bind:checked={greenScreen} class="checkbox"/>
                </label>
            </div>

            <Streamer bind:fileContainerBackground={fileContainerBackground}/>
        </div>
        <div class="flex flex-col w-full h-screen justify-between">
            <div>
                <div id="file-container" bind:this={fileContainer} class="container aspect-video"
                     style="background-image: url({fileContainerBackground})">

                    {#if isText}
                        <TexteValue {textData}/>
                    {/if}

                    <div class="target" bind:this={preview} id="preview">
                        {#if isVideo}
                            <video bind:this={video} class="w-full h-full" controls></video>
                        {/if}

                        {#if isImage}
                            <img bind:this={image} class="w-full h-full" alt="" src=""/>
                        {/if}
                    </div>

                    {#if isText}
                        <Moveable
                                bind:this={moveableText}
                                target={"#moveable"}
                                draggable={true}
                                throttleDrag={1}
                                edgeDraggable={false}
                                startDragRotate={0}
                                throttleDragRotate={0}
                                rotatable={false}
                                snappable={true}
                                bounds={{"left": 0, "top": 0, "right": 0, "bottom": 0, "position": "css"}}
                                resizable={false}
                                scalable={false}
                                origin={false}
                                keepRatio={true}
                                renderDirections={["nw","n","ne","w","e","sw","s","se"]}

                                on:drag={({ detail: e })=> { e.target.style.transform = e.transform; }}
                                on:rotate={({ detail: e }) => { e.target.style.transform = e.afterTransform; }}
                                on:resize={({ detail: e }) => {
                                    e.target.style.width = `${e.width}px`;
                                    e.target.style.height = `${e.height}px`;
                                    e.target.style.transform = e.drag.transform;
                                }}
                                on:scale={({ detail: e }) => {
                e.target.style.transform = e.drag.transform;
            }}

                        />
                    {/if}

                    {#if isVideo || isImage}
                        <Moveable
                                bind:this={moveable}
                                target={".target"}
                                draggable={true}
                                throttleDrag={1}
                                edgeDraggable={false}
                                startDragRotate={0}
                                throttleDragRotate={0}
                                rotatable={false}
                                snappable={true}
                                bounds={{"left": 0, "top": 0, "right": 0, "bottom": 0, "position": "css"}}
                                resizable={true}
                                scalable={false}
                                origin={false}
                                keepRatio={true}
                                renderDirections={["nw","n","ne","w","e","sw","s","se"]}

                                on:drag={({ detail: e })=> { e.target.style.transform = e.transform; }}
                                on:rotate={({ detail: e }) => { e.target.style.transform = e.afterTransform; }}
                                on:resize={({ detail: e }) => {
                                    e.target.style.width = `${e.width}px`;
                                    e.target.style.height = `${e.height}px`;
                                    e.target.style.transform = e.drag.transform;
                                }}
                                on:scale={({ detail: e }) => {
                e.target.style.transform = e.drag.transform;
            }}

                        />
                    {/if}

                </div>
            </div>

            <div class="flex h-full justify-around items-center">
                <!--Clean-->
                <div class="tooltip" data-tip="Clean">
                    <button class="btn btn-primary" on:click={clean}>
                        <span class="material-symbols-outlined">mop</span>
                    </button>
                </div>
                <!--Flush-->
                <div class="tooltip" data-tip="Flush">
                    <button class="btn btn-primary" on:click={flush}>
                        <span class="material-symbols-outlined">delete</span>
                    </button>
                </div>
                <!--Send-->
                <div class="tooltip" data-tip="Send">
                    <button class="btn btn-primary" on:click={send}>
                        <span class="material-symbols-outlined">send</span>
                    </button>
                </div>
                <!--Skip-->
                <div class="tooltip" data-tip="Skip">
                    <button class="btn btn-primary" on:click={skip}>
                        <span class="material-symbols-outlined">skip_next</span>
                    </button>
                </div>
            </div>
        </div>
    </div>
</main>

<style>
    #file-container {
        position: relative;
        background-size: cover;
        width: 100%;
    }

    #preview {
        position: absolute;
    }

    #file {
        width: 100%;
    }
</style>