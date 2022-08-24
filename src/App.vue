

<script setup>
import Sidebar from './components/Sidebar.vue'
//import TheWelcome from './components/TheWelcome.vue'

import { reactive, ref, onMounted } from 'vue'
import shaders from './assets/shaders/shaders.json';

const el = ref()
const models = reactive([

]);

const materials = reactive({});


const selectedObject = reactive({
    selected: false,
    hovered: false,
    selectedObject: undefined,
    hoveredObject: undefined,
});

onMounted(() => {

    var uploadedModel = undefined;
    var selectedMaterial = undefined;


    var totalModels = 1;

    BABYLON.STLFileLoader.DO_NOT_ALTER_FILE_COORDINATES = true;
    BABYLON.SceneLoader.RegisterPlugin(new BABYLON.STLFileLoader());

    const canvas = document.getElementById('renderCanvas');
    const engine = new BABYLON.Engine(canvas, true);
    const scene = new BABYLON.Scene(engine);

    const gizmoManager = new BABYLON.GizmoManager(scene);

    gizmoManager.positionGizmoEnabled = true;
    gizmoManager.rotationGizmoEnabled = true;
    gizmoManager.boundingBoxGizmoEnabled = true;

    gizmoManager.clearGizmoOnEmptyPointerEvent = true;
    gizmoManager.usePointerToAttachGizmos = true;

    gizmoManager.gizmos.positionGizmo.snapDistance = 0.1;
    gizmoManager.gizmos.boundingBoxGizmo.scaleBoxSize = 1;
    gizmoManager.gizmos.boundingBoxGizmo.rotationSphereSize = 0;

    gizmoManager.onAttachedToMeshObservable.add((gizmo) => {
        if(gizmo) {
            console.log(gizmo)
            selectedObject.selectedObject = gizmo;
            selectedObject.selected = true;
        }
        else {
             console.log("null gizmo")
            selectedObject.selected = false;

        }
    });

    const camera = createCamera();
    createPipeline();


    function createCamera() {
        const camera = new BABYLON.ArcRotateCamera("Camera", 0, 0, 0, new BABYLON.Vector3(0, 0, 0), scene);
        camera.setPosition(new BABYLON.Vector3(70, 150, 200));
        camera.attachControl(canvas, true);
        return camera;
    }

    function createPipeline() {
        var pipeline = new BABYLON.DefaultRenderingPipeline(
            "defaultPipeline", // The name of the pipeline
            true, // Do you want the pipeline to use HDR texture?
            scene, // The scene instance
            [camera] // The list of cameras to be attached to
        );

        pipeline.samples = 8;
        pipeline.imageProcessingEnabled = false;

    }

    function loadShaders() {

        // get keys from shaders object
        const keys = Object.keys(shaders);

        // loop through keys and load each shader
        keys.forEach(key => {

            var shader = shaders[key];

            var shaderMaterial = new BABYLON.ShaderMaterial(
                key,
                scene,
                {
                    vertexElement: shader.vertexElement,
                    fragmentElement: shader.fragmentElement,
                },
                {
                    attributes: shader.attributes,
                    uniforms: shader.uniforms,
                },
            );
            shaderMaterial.MATERIAL_ALPHABLEND = true;
            materials[key] = shaderMaterial;
        });
    }

    function createGUI() {
        var advancedTexture = BABYLON.GUI.AdvancedDynamicTexture.CreateFullscreenUI(
            "UI"
        );

        var selectBox = new BABYLON.GUI.SelectionPanel("selectBox");
        selectBox.width = 0.125;
        selectBox.height = 0.2;
        selectBox.horizontalAlignment = BABYLON.GUI.Control.HORIZONTAL_ALIGNMENT_LEFT;
        selectBox.verticalAlignment = BABYLON.GUI.Control.VERTICAL_ALIGNMENT_BOTTOM;

        advancedTexture.addControl(selectBox);
    }


    function createPlane(size) {
                
        const plane = BABYLON.MeshBuilder.CreatePlane("plane", { size: size }, scene);
        plane.position.x = 0;
        plane.position.z = 0;
        plane.position.y = 0;
        plane.rotation.x = Math.PI / 2;
        plane.isPickable = false;
        
        // Material
        const material = new BABYLON.GridMaterial("groundMaterial", scene);
        material.mainColor = new BABYLON.Color4(0.0, 0.0, 0.8, 1.0);
        material.opacity = 0.999;
        plane.material = material;
        plane.material.backFaceCulling = false;

        return plane;
    }

    $("#upload-model").click(function (e) {
        $("#upload-model-form").toggle();
    });

    // check if the model is uploaded
    $("#model-file").change(function (e) {

        uploadedModel = e.target.files[0];

        if ($("#model-name").val() == "") {
            $("#model-name").val(uploadedModel.name);
        }

        readSingleFile(uploadedModel, function (datastring) {

            var model_data = btoa([].reduce.call(new Uint8Array(datastring), function (p, c) { return p + String.fromCharCode(c) }, ''))
            var base64_model_content = "data:;base64," + model_data;
            var raw_content = BABYLON.FileTools.DecodeBase64UrlToBinary(base64_model_content);

            var blob = new Blob([raw_content]);
            var url = URL.createObjectURL(blob);

            var name = $("#model-name").val();

            models.push({
                name: name,
            });

            BABYLON.SceneLoader.Append("", url, scene, function (baby) {
                
                let model = baby.meshes[models.length];
                model.material = new BABYLON.StandardMaterial("model-material", scene);
                model.material.alpha = 1;
                model.material.backFaceCulling = false;
                model.name = name;
                model.isPickable = true;

             
               // reactive(selectedObject.model)

            }, undefined, undefined, ".stl");
           
            $("#upload-model-form").toggle();
            $("#model-name").val("");

        });
    });


    // read single file as base64 string
    function readSingleFile(file, cb) {
        var reader = new FileReader();
        reader.addEventListener("load", function (e) {
            cb(e.target.result);
        })
        reader.readAsArrayBuffer(file);
    }



    function main_loop() {
        // set clear color to black
        scene.clearColor = new BABYLON.Color3(0.1, 0.1, 0.13);

        const dirLight = new BABYLON.DirectionalLight("dirLight", new BABYLON.Vector3(0.15, -0.8, 0), scene);

        loadShaders();

        createPlane(200);
        
      

        engine.runRenderLoop(function () {
            scene.render();
        });

        window.addEventListener('resize', function () {
            engine.resize();
        });
    }

    main_loop();

})
</script>


<template>
    <div class="container-fluid">
        <header>
            <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
                <a class="navbar-brand" href="/">Product Configurator</a>
                <ul class="navbar-nav mr-auto">
                    <li class="nav-item">
                        <a class="nav-link" href="#" id="upload-model">Upload model</a>
                    </li>
                </ul>
            </nav>
        </header>
        <div id="modals">
            <div id="upload-modal">
                <div id="upload-model-form" class="modal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel"
                    aria-hidden="true">
                    <div class="modal-dialog">
                        <div class="modal-content">
                            <div class="modal-header">
                                <button type="button" class="close" data-dismiss="modal" aria-hidden="true">×</button>
                                <h4 class="modal-title" id="myModalLabel">Upload Model</h4>
                            </div>
                            <div class="modal-body">
                                <div class="container">
                                    <form action="{{ url('/upload-model') }}" method="post"
                                        enctype="multipart/form-data">
                                        <div class="form-group">
                                            <label for="model-name">Model Name</label>
                                            <input type="text" class="form-control" id="model-name" name="model-name"
                                                placeholder="Enter model name">
                                        </div>
                                        <div class="form-group">
                                            <label for="model-file">Model File</label>
                                            <input type="file" class="form-control" id="model-file" name="model-file">
                                        </div>

                                        <button type="submit" class="btn btn-primary">Upload</button>
                                    </form>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <main>
            <div class="container-fluid">
                <div class="row ">

                  

                    <div class="col-md-12">
                        <div class="ratio ratio-16x9">
                            <canvas id="renderCanvas"></canvas>
                        </div>
                    </div>

                  
                </div>
            </div>
        </main>
    </div>
</template>

<style scoped>
#renderCanvas {

    height: 100%;
    width: 100%;
    margin: auto;
}

#renderCanvas:focus-visible {
    outline: none;
}
</style>