<script lang="javascript">
  let video, canvas, ctx, worker;
  let running = false;
  let output = "";

  const start = async () => {
	console.log('start');
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: true,
      });
      video.srcObject = stream;
      video.play();
      video.onresize = () => {
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        running = true;
      };
    } catch (error) {
      /* Do you have a webcam? */
      console.log(error);
    }

    // Initialize Tesseract worker
	worker = await Tesseract.createWorker('deu', 1, {
      logger: (m) => console.log(m),
    });
  };

  const capture = async () => {
	ctx = canvas.getContext("2d");
    ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
    const img = canvas.toDataURL("image/png");
	const { data: { text } } = await worker.recognize(img);
	output = text;
	
	setTimeout(capture, 500);
  };

  // Start the webcam
  start();
</script>

<main>
  <video bind:this={video} style="display: none">
	<track kind="captions">
  </video>
  <canvas bind:this={canvas} />
  {#if running}
    <button on:click={capture}>Capture</button>
    <div><b>output:</b> {output}</div>
  {:else}
    <div>Loading...</div>
  {/if}
</main>


<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>