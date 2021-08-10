<script>
  import { onMount } from "svelte";

  import * as tf from "@tensorflow/tfjs";

  import * as tmImage from "@teachablemachine/image";

  let videoEl;
  let errorMessage;
  let model;
  let loading = true;
  let percentage = "";
  let name = "";
  let backgroundColor = "#fff";
  let fontColor = "#000000";

  const URL = "model/";
  const modelURL = URL + "model.json";
  const metadataURL = URL + "metadata.json";

  onMount(async () => {
    try {
      model = await tmImage.load(modelURL, metadataURL);
      const stream = await navigator.mediaDevices.getUserMedia({ video: true });

      videoEl.srcObject = stream;
      videoEl.play();
      setInterval(predict, 1000);
      loading = false;
    } catch (e) {
      console.error(e, "camera access denied");
      errorMessage = "Camera Access Denied";
    }
  });

  $: if (name === "Healthy") {
    backgroundColor = "#42edb1";
    fontColor = "#045187";
  } else if (name === "Potato healthy") {
    backgroundColor = "#42edb1";
    fontColor = "#045187";
  } else {
    backgroundColor = "#fff";
    fontColor = "#000000";
  }

  async function predict() {
    const predictions = await model.predict(videoEl);
    const [choosenPrediction] = predictions.sort(
      (a, b) => b.probability - a.probability
    );

    if (choosenPrediction) {
      percentage = (choosenPrediction.probability * 100).toFixed(2) + "%";
      name = classNameToLabel(choosenPrediction.className);
    }
  }

  function classNameToLabel(className) {
    switch (className) {
      case "Common_rust":
        return "Common rust";
      case "healthy":
        return "Healthy";
		  case "Northen_leaf_blight":
        return "Northen leaf blight";
		  case "Gray_leaf_spot":
        return "Gray leaf spot";
		  case "Potato_healthy":
        return "Potato healthy";
		  case "Potato_Late_blight":
        return "Potato Late blight";
		  case "Potato_Early_blight":
        return "Potato Early blight";
      default:
        return "Nothing";
    }
  }
</script>

<style>
  main {
    width: 100%;
    height: 100vh;
    padding: 0;
    box-sizing: border-box;
    position: absolute;
  }

  video {
    display: block;
    margin: 20px auto;
  }

  h1,
  h2 {
    text-align: center;
  }

  h1 {
    font-size: 40px;
  }

  h2 {
    font-size: 20px;
  }
</style>

<main style="background-color: {backgroundColor}; color: {fontColor};">
  <h1>Machine Learning</h1>
  <video bind:this={videoEl} width="600" height="480" />

  {#if errorMessage}
    <h2 style="color: red;">{errorMessage}</h2>
  {:else if loading}
    <h2>Loading ...</h2>
  {:else if percentage && name}
    <h2>AI {percentage} certain it's a {name}</h2>
  {/if}
</main>
