<script lang="ts">
    let streamer: string;
    export let fileContainerBackground: string;

    function search(e: SubmitEvent) {
        e.preventDefault();

        const headers = {
            "Client-ID": "kimne78kx3ncx6brgo4mv6wki5h1ko"
        };

        const query = `query { user(login: "${streamer}") { stream() { previewImageURL(width: 1920, height: 1080) } } }`;

        const requestOptions = {
            method: "POST",
            headers: headers,
            body: JSON.stringify({query})
        };

        fetch("https://gql.twitch.tv/gql", requestOptions)
            .then(response => response.json())
            .then(data => {
                try {
                    fileContainerBackground = data.data.user.stream.previewImageURL;
                } catch (e) {
                    console.log(e);
                }
            })
            .catch(error => {
                console.log(error);
            });
    }
</script>

<form id="streamerForm" class="join" on:submit={search} >
    <input id="streamer" type="text" placeholder="Streamer" class="input input-bordered w-full max-w-xs join-item" bind:value={streamer}/>
    <button class="btn btn-primary join-item">
        <span class="material-symbols-outlined">search</span>
    </button>
</form>
