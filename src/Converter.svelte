

<script>
  import { createFFmpeg, fetchFile } from '@ffmpeg/ffmpeg';
  import Dropzone from "svelte-file-dropzone";
  import { fade } from 'svelte/transition';
  import { Button } from 'svelte-chota';


  let files = {
      accepted: [],
      rejected: []
  };

  let urldata = '';
  let compatible = ( window.SharedArrayBuffer === undefined ) ? false : true;
  const audio = document.getElementById('player');

  function logMessage(msg) {
      const message = document.getElementById('message');
      message.innerHTML = msg;
      message.classList.add("alert")
  }

  function handleDropZone(e) {
      const {acceptedFiles, fileRejections } = e.detail;
      removeAll();
      files.accepted = [...files.accepted, ...acceptedFiles];
      files.rejected = [...files.rejected, ...fileRejections];
  }

  function removeAll() {
      files.accepted = [];
      files.rejected = [];
      urldata = '';
      let player = document.getElementById('player');
      player.style.display = "none";
      URL.revokeObjectURL(player.src);

      let mezzage = document.getElementById('message');
      mezzage.innerHTML = "";
      mezzage.classList.remove("alert");
  }

  const ffmpeg = createFFmpeg({ log: false,
        progress: ({ ratio }) => {console.log(`Complete: ${(ratio * 100.0).toFixed(2)}%`);  },
    });

  function download(filename, url) {
    var element = document.createElement('a');
    element.setAttribute('href', url);
    element.setAttribute('download', filename);

    element.style.display = 'none';
    document.body.appendChild(element);

    element.click();

    document.body.removeChild(element);      
    }

  async function convertFile(e, index) {
    const f = files.accepted[index];
    e.target.disable = true;

    const filename = f.path;
    if (!ffmpeg.isLoaded()) {
        logMessage( "Initializing... (first load takes longer.)") ;
        await ffmpeg.load();
    }
    ffmpeg.FS('writeFile', filename, await fetchFile(f));
    await ffmpeg.run('-i', filename, '-c:a', 'libmp3lame', 'audio.mp3');
    const data = ffmpeg.FS('readFile', 'audio.mp3');

    urldata = URL.createObjectURL(new Blob([data.buffer], {type: 'audio/mp3'}));
    const audio = document.getElementById('player');
    audio.src = urldata;
    audio.style.display = "block";

    const mp3filename = filename.substr(0, filename.lastIndexOf(".")) + ".mp3";
    download(mp3filename, urldata );

    logMessage("<span class='down-arrow'>&#8681;</span> Downloaded " + mp3filename + " (size:" + Math.round(data.length/1024) + "kb).");
  }

</script>

<div class="converter">

    {#if compatible}
    <Dropzone on:drop={handleDropZone} accept=".wav" multiple="false" containerClasses="wavzone" >
        <p>Drag 'n' drop .wav file here, or click for file selector</p>
    </Dropzone>
    {:else}
    <p class="sad alert">This browser doesn't support the features necessary for WAV to MP3 conversion. 
        Please try a recent version of Google Chrome or Microsoft Edge.</p>
    {/if}

    <!-- svelte-ignore a11y-media-has-caption -->
    <audio id="player" style="display: none; margin: 0px auto;" controls></audio>

    {#if files.accepted.length > 0}
    <div style="margin: 10px auto;" transition:fade>

        <span class="file-text">{files.accepted[0].name}</span>
        <span class="file-text">{Math.round(files.accepted[0].size/1024)}kb</span>
        <Button primary on:click={e => convertFile(e, 0)}>Convert</Button>
        <Button secondary on:click={removeAll}>Reset</Button>
    </div>
    {/if}

    <p id="message" transition:fade></p>
</div>


<style>
    .alert { 
        background-color: #acebb1;
        color: rgb(17, 90, 7);
        padding: 1rem;
        margin: 10px 15vw;
        align-self:center;
        border-radius: 20px;
    }
    .sad {
        background-color: #dfa6a6;
        color: rgb(133, 8, 8);
    }
    .converter {
        margin: 0 auto;
    }
    :global(wavzone) {
        background-color: aquamarine;
        color: black;
        border-color: red;
    }
    #message {
        border: 5px red;
    }
    .primary-btn {
        background: rgb(60, 120, 231);
        color: white;
    }
    button {
        margin: 0em 0.25em;
        border-radius: 5px;
    }
    button:hover {
        background-color: #9f9f9f;
        color: black;
    }
    .down-arrow {
        font-size: x-large;
    }
    .file-text {
        margin-right:0.5em;

    }
</style>