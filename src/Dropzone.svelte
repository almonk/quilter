<script>
  import { onMount } from "svelte";
  import { scale, fly } from "svelte/transition";
  import { quintOut } from "svelte/easing";
  import domtoimage from "dom-to-image";

  let dragOver = false;
  let imageData = [];
  let imageHeight = 100;
  let imageBg = "#fff";
  let files = []

  function saveImage() {
    domtoimage
      .toPng(document.getElementById("quilt"), { quality: 1 })
      .then(function(dataUrl) {
        var link = document.createElement("a");
        link.download = "quilt.png";
        link.href = dataUrl;
        link.click();
      });
  }

  onMount(async () => {
    let dropArea = document.getElementById("drop-area");

    dropArea.addEventListener(
      "dragover",
      function(e) {
        dragOver = true;
        e.preventDefault();
      },
      false
    );

    dropArea.addEventListener(
      "dragleave",
      function(e) {
        dragOver = false;
        e.preventDefault();
      },
      false
    );

    dropArea.addEventListener(
      "drop",
      function(e) {
        let dt = e.dataTransfer;
        files = dt.files;

        console.log(files);

        if (FileReader && files && files.length) {
          for (let index = 0; index < files.length; index++) {
            var fr = new FileReader();
            fr.addEventListener("load", event => {
              console.log(event.target.result);
              imageData.push(event.target.result);
              imageData = imageData;
            });
            fr.readAsDataURL(files[index]);
          }
        }

        dragOver = false;
        e.preventDefault();
        return false;
      },
      false
    );
  });
</script>

<main>
  {#if imageData.length > 0}
    <div class="bg-gray-100 border-b px-4 py-2 flex items-center text-sm" transition:fly={{ y: -200, easing: quintOut }}>
      <div class="flex flex-col mr-8">
        <div class="mb-1 text-gray-700">Image height</div>
        <div class="flex">
          <input type="number" bind:value={imageHeight} class="w-20 mr-2 p-1 border rounded"/>
          <input type="range" bind:value={imageHeight} min="100" max="500" class="p-1 border rounded w-12"/>
        </div>
        
      </div>

      <div class="flex flex-col mr-4">
        <div class="mb-1 text-gray-700">Quilt background</div>
        <input type="text" bind:value={imageBg} class="p-1 border rounded font-mono"/>
      </div>
      
      <div class="flex-auto"></div>

      <button on:click={saveImage} class="text-white bg-black px-4 py-2 rounded">Download as image</button>
    </div>
  {/if}

  {#if imageData.length > 0}
    <div class="flex flex-wrap bg-white m-4 shadow" style="background: {imageBg}" id="quilt">
      {#each imageData as image}
        <img
          src={image}
          alt=""
          style="height: {imageHeight}px"
          transition:scale={{ duration: 500, delay: 500, opacity: 0.5, start: 0.5, easing: quintOut }} />
      {/each}
    </div>
  {:else}
    <div class="text-center text-gray-500 p-16 h-screen flex items-center justify-center">
      Drag and drop some images to create your quilt
    </div>
  {/if}
</main>
