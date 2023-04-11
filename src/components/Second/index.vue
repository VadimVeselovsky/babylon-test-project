<template>
  <div class="second">
    <canvas ref="canvas" />
  </div>
</template>

<script>
import {
  Engine,
  Scene,
  ArcRotateCamera,
  HemisphericLight,
  Vector3,
  MeshBuilder,
  Quaternion,
  ActionManager,
  Animation,
} from "babylonjs";
import "babylonjs-loaders";

function getKeyframes(frames, bumps, amplitude) {
  let output = [
    { time: 0, y: 1 },
    { time: 1 / 3, y: 1 - (1 / 3) ** 2 },
    { time: 2 / 3, y: 1 - (2 / 3) ** 2 },
    { time: 1, y: 0 },
  ];
  const rate = 3 / 4;

  for (var i = 1; i < bumps; i++) {
    const prevTime = output.length === 0 ? 0 : output[output.length - 1].time,
      bounceTime = Math.sqrt(rate ** i),
      h = rate ** i;

    if (h > 0.15)
      output.push(
        { time: prevTime + bounceTime / 6, y: (1 - (2 / 3) ** 2) * h },
        { time: prevTime + (bounceTime * 2) / 6, y: (1 - (1 / 3) ** 2) * h },
        { time: prevTime + (bounceTime * 3) / 6, y: h },
        { time: prevTime + (bounceTime * 4) / 6, y: (1 - (1 / 3) ** 2) * h },
        { time: prevTime + (bounceTime * 5) / 6, y: (1 - (2 / 3) ** 2) * h },
        { time: prevTime + bounceTime, y: 0 }
      );
    else if (h > 0.05)
      output.push(
        { time: prevTime + bounceTime / 4, y: (1 - (1 / 2) ** 2) * h },
        { time: prevTime + bounceTime / 2, y: h },
        { time: prevTime + (bounceTime * 3) / 4, y: (1 - (1 / 2) ** 2) * h },
        { time: prevTime + bounceTime, y: 0 }
      );
    else
      output.push(
        { time: prevTime + bounceTime / 2, y: h },
        { time: prevTime + bounceTime, y: 0 }
      );
  }

  output = output.map((el) => ({
    value: el.y * amplitude,
    frame: Math.round((el.time * frames) / output[output.length - 1].time),
  }));

  // убрать дублирующиеся фреймы, если такие есть
  for (var i = 0; i < output.length - 1; i++) {
    if (output[i].frame === output[i + 1].frame) {
      output.splice(i + 1, 1);
      i--;
    }
  }

  return output;
}

function applyBouncing(node, amplitude, duration, scene) {
  const fps = 60,
    bouncingAnimation = new Animation(
      "bounce",
      "position.y",
      fps,
      Animation.ANIMATIONTYPE_FLOAT,
      Animation.ANIMATIONLOOPMODE_CYCLE
    ),
    frames = getKeyframes(duration * fps, 20, amplitude);

  bouncingAnimation.setKeys(frames);

  node.animations.push(bouncingAnimation);

  setTimeout(() => scene.beginAnimation(node, 0, duration * fps, true), 2500);
}

export default {
  mounted() {
    this.initCanvas();
  },

  methods: {
    prepareMeshes(scene) {
      const sphere = MeshBuilder.CreateSphere("sphere", {}, scene);

      const ground = MeshBuilder.CreateGround("ground", { width: 10, height: 10 }, scene);
      ground.position.y = -0.5;

      applyBouncing(sphere, 1, 4, scene);
    },

    initCanvas() {
      const canvas = this.$refs.canvas;

      const engine = new Engine(canvas, true, {});
      const scene = new Scene(this.engine);

      this.prepareScene(canvas, scene);

      this.prepareMeshes(scene);

      this.runScene(engine, scene);

      // scene.debugLayer.show();
    },

    runScene(engine, scene) {
      engine.runRenderLoop(() => {
        scene.render();
      });

      window.addEventListener("resize", () => {
        engine.resize();
      });
    },

    prepareScene(canvas, scene) {
      // Camera
      const camera = new ArcRotateCamera(
        "camera",
        Math.PI / 2,
        Math.PI / 2.5,
        4,
        new Vector3(0, 0, 0),
        scene
      );
      camera.attachControl(canvas, true);

      // Light
      new HemisphericLight("light", new Vector3(0.5, 1, 0.8).normalize(), scene);
    },
  },
};
</script>

<style lang="scss" scoped>
canvas {
  width: 100%;
  height: 100vh;
  display: block;
}

.second {
  position: relative;
  height: 100vh;
}
</style>
