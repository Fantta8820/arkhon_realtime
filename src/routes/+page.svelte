<script lang="ts">
  import { onMount } from "svelte";
  import { Canvas, Rect } from "fabric";
  import io from "socket.io-client";

  let canvas: Canvas;
  const socket = io("http://localhost:3000");    

  let isUpdating: boolean = false;

  let projectID = "1";

  // Inicializar o canvas com Fabric.js
  onMount(() => {
    canvas = new Canvas("canvas");

    socket.emit("joinProject", projectID);
    
    canvas.add(
      new Rect({
        width: 100,
        height: 100,
        strokeWidth: 2,
      })
    );

    addEventListener("keydown", (e) => {
      if (e.key == "x") {
        if (canvas.getActiveObjects().length > 0) {
          canvas.remove(canvas.getActiveObjects()[0]);
          canvas.requestRenderAll();

          broadcastUpdate();
        }
      }
    });

    // Listener para mudanças no canvas (enviar para o servidor)
    canvas.on("object:added", () => {
      if(!isUpdating){
        broadcastUpdate();
      }
    });  

    canvas.on("object:modified", broadcastUpdate);

    canvas.on("object:moving", broadcastUpdate);

    canvas.on("object:scaling", broadcastUpdate);

    canvas.on("object:rotating", broadcastUpdate);

    // Listener para receber atualizações do servidor
    socket.on("updateCanvas", (data) => {
      updateCanvasFromServer(data);
    });
  });

  // Enviar atualizações para o servidor
  function broadcastUpdate() {
    const canvasData = canvas.toJSON();    

    // Definir isUpdating como true antes de emitir os dados para evitar loop
    isUpdating = false;    
    socket.emit("updateCanvas", projectID, canvasData );
  }

  // Atualizar o canvas com os dados recebidos do servidor
  function updateCanvasFromServer(data: any) {    
    canvas.loadFromJSON(data);
    canvas.requestRenderAll();        

    isUpdating = true;    
  }

  function addRect() {
    canvas.add(
      new Rect({
        width: 100,
        height: 100,
        strokeWidth: 2,
      })
    );

    broadcastUpdate();

    isUpdating = false;
  }
</script>

<canvas id="canvas" width="800" height="600"></canvas>
<button on:click={addRect}>Adicionar Rect</button>

<style>
  canvas {
    border: 1px solid #ccc;
  }
</style>