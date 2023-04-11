<template>
  <div class="first">
    <div class="first__interface">
      <ParametersChangeWindow
        v-if="!!changedFigure"
        :key="changedFigure"
        :windowName="changedFigureModalProps.name"
        :inputs="changedFigureModalProps.inputs"
        @close="changedFigure = null"
      />
    </div>
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
} from "babylonjs";
import "babylonjs-loaders";
import ParametersChangeWindow from "./ParametersChangeWindow.vue";

export default {
  components: { ParametersChangeWindow },

  data() {
    return {
      changedFigure: null,
      figures: {},
      figuresData: {
        icosphere: {
          subdivisions: 4,
        },
      },
      scene: null,
    };
  },

  mounted() {
    this.initCanvas();
  },

  computed: {
    changedFigureModalProps() {
      const figuresToProps = {
        plane: {
          name: "Edit Plane Parameters",
          inputs: [
            {
              name: "width",
              getter: (value) => this.figures.plane.scaling.x,
              setter: (value) => (this.figures.plane.scaling.x = value),
              min: 0.1,
              max: 2,
              step: 0.1,
            },
            {
              name: "height",
              getter: (value) => this.figures.plane.scaling.y,
              setter: (value) => (this.figures.plane.scaling.y = value),
              min: 0.1,
              max: 2,
              step: 0.1,
            },
            {
              name: "depth",
              getter: (value) => this.figures.plane.scaling.z,
              setter: (value) => (this.figures.plane.scaling.z = value),
              min: 0.1,
              max: 2,
              step: 0.1,
            },
          ],
        },
        icosphere: {
          name: "Edit IcoSphere Parameters",
          inputs: [
            {
              name: "diameter",
              getter: (value) => this.figures.icosphere.scaling.x * 2,
              setter: (value) => {
                const radius = value / 2;
                this.figures.icosphere.scaling.set(radius, radius, radius);
              },
              min: 0.1,
              max: 2,
              step: 0.1,
            },
            {
              name: "subdivisions",
              getter: (value) => this.figuresData.icosphere.subdivisions,
              setter: (value) => {
                if (value % 1 === 0) {
                  this.figuresData.icosphere.subdivisions = value;
                  this.createIcoSphere(this.scene, value);
                }
              },
              min: 1,
              max: 10,
              step: 1,
            },
          ],
        },
        cylinder: {
          name: "Edit Cylinder Parameters",
          inputs: [
            {
              name: "diameter",
              getter: (value) => this.figures.cylinder.scaling.x * 2,
              setter: (value) => {
                const radius = value / 2;
                this.figures.cylinder.scaling.x = radius;
                this.figures.cylinder.scaling.z = radius;
              },
              min: 0.1,
              max: 2,
              step: 0.1,
            },
            {
              name: "height",
              getter: (value) => this.figures.cylinder.scaling.y * 2,
              setter: (value) => (this.figures.cylinder.scaling.y = value / 2),
              min: 0.1,
              max: 2,
              step: 0.1,
            },
          ],
        },
      };

      return figuresToProps[this.changedFigure] || null;
    },
  },

  methods: {
    createIcoSphere(scene, subdivisions) {
      let radius = 1;
      if (this.figures.icosphere) {
        radius = this.figures.icosphere.scaling.x;
        this.figures.icosphere.dispose();
      }
      this.figures.icosphere = MeshBuilder.CreateIcoSphere(
        "IcoSphere",
        { subdivisions },
        scene
      );
      this.figures.icosphere.position.set(-2, 0, 0);
      this.figures.icosphere.scaling.set(radius, radius, radius);
      this.figures.icosphere.actionManager = new ActionManager(scene);
      this.figures.icosphere.actionManager.registerAction(
        new BABYLON.ExecuteCodeAction(
          ActionManager.OnPickTrigger,
          () => (this.changedFigure = "icosphere")
        )
      );

      window.temp1 = this.figures.icosphere;
    },

    prepareMeshes(scene) {
      this.figures.plane = MeshBuilder.CreateBox("Plane", {}, scene);
      this.figures.plane.rotationQuaternion = Quaternion.FromEulerAngles(0, Math.PI, 0);
      this.figures.plane.actionManager = new ActionManager(scene);
      this.figures.plane.actionManager.registerAction(
        new BABYLON.ExecuteCodeAction(
          ActionManager.OnPickTrigger,
          () => (this.changedFigure = "plane")
        )
      );

      this.createIcoSphere(scene, 4);

      this.figures.cylinder = MeshBuilder.CreateCylinder("Cylinder", {}, scene);
      this.figures.cylinder.position.set(2, 0, 0);
      this.figures.cylinder.actionManager = new ActionManager(scene);
      this.figures.cylinder.actionManager.registerAction(
        new BABYLON.ExecuteCodeAction(
          ActionManager.OnPickTrigger,
          () => (this.changedFigure = "cylinder")
        )
      );
    },

    initCanvas() {
      const canvas = this.$refs.canvas;

      const engine = new Engine(canvas, true, {});
      this.scene = new Scene(this.engine);

      this.prepareScene(canvas, this.scene);

      this.prepareMeshes(this.scene);

      this.runScene(engine, this.scene);

      // this.scene.debugLayer.show();
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

.first {
  position: relative;
  height: 100vh;

  &__interface {
    position: absolute;
    top: 0;
    left: 50%;
    top: 40%;
    left: calc(66% - 150px);
  }
}
</style>
