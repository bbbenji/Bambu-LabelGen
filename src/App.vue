<script setup lang="ts">
// Importing necessary Vue composition API functions and external libraries
import { onMounted, watch, reactive } from "vue";
import { debounce } from "lodash-es";
import { Canvg } from "canvg";
import downloadURI from "@/utils/downloadURI";

// Import custom type definitions and components
import LabelDataType from "@/types/LabelData";
import Label from "@/components/Label.vue";
import TextInput from "@/components/TextInput.vue";

// Reactive state for label configuration
const configData = reactive<LabelDataType>({
  // Initial values for label text and colour properties
  text: {
    brand: "Bambu",
    type: "PLA Aero",
    diameter: "2.75",
    tolerance: "0.03",
    length: "340",
    temp: "220 - 260",
    weight: "1",
    units: "kg",
    code: "14103",
    colour_name: "Black",
  },
  colour: {
    background: "#FFFFFF",
    text: "#2D2D2D",
    outline: "#A5AAA9",
    filament: "#000000",
  },
});

// Create a debounced function to update URL parameters for performance optimization
const debouncedUpdateURLParameters = debounce(updateURLParameters, 500);

// Function to update URL parameters based on configData
function updateURLParameters() {
  try {
    const params = new URLSearchParams();

    // Loop through text and colour properties to set them as URL parameters
    for (const [key, value] of Object.entries(configData.text)) {
      if (typeof value === "string") {
        params.set(`text.${key}`, value);
      }
    }
    for (const [key, value] of Object.entries(configData.colour)) {
      if (typeof value === "string") {
        params.set(`colour.${key}`, value);
      }
    }

    // Update the browser URL without reloading the page
    window.history.replaceState(
      {},
      "",
      `${window.location.pathname}?${params}`
    );
  } catch (error) {
    console.error("Error updating URL parameters:", error);
  }
}

// Function to parse URL parameters and update configData accordingly
function parseURLParameters() {
  try {
    const params = new URLSearchParams(window.location.search);

    // Update text and colour properties from URL parameters if available
    for (const [key] of Object.entries(configData.text)) {
      const paramValue = params.get(`text.${key}`);
      if (paramValue !== null) {
        configData.text[key] = paramValue;
      }
    }
    for (const [key] of Object.entries(configData.colour)) {
      const paramValue = params.get(`colour.${key}`);
      if (paramValue !== null) {
        configData.colour[key] = paramValue;
      }
    }
  } catch (error) {
    console.error("Error parsing URL parameters:", error);
  }
}

// Watch for changes in configData and update URL parameters using the debounced function
watch(configData, debouncedUpdateURLParameters, { deep: true });

// Parse URL parameters when the component is mounted
onMounted(parseURLParameters);

// Function to download the label as a PNG file
async function downloadLabel(e: Event) {
  // Select the SVG element and its dimensions
  const svgElement = document.querySelector(
    "#full-sized-label"
  ) as SVGSVGElement;
  const { width, height } = svgElement.viewBox.baseVal;

  // Set up a canvas and its context for rendering
  const canvas = document.querySelector("canvas") as HTMLCanvasElement;
  const ctx = canvas.getContext("2d") as CanvasRenderingContext2D;
  canvas.width = width;
  canvas.height = height;

  // Convert SVG to Canvas using Canvg library
  const v = await Canvg.from(ctx, svgElement.outerHTML);
  v.render();

  // Trigger download of the rendered image
  downloadURI(canvas.toDataURL("image/png"), "Label.png");
}
</script>

<template>
  <div class="container mx-auto p-5 h-full">
    <h1 class="text-center text-3xl pt-6 pb-12">Bambu Lab-elGen</h1>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-12">
      <div>
        <!-- Input fields for label text configuration with data binding to reactive state -->
        <div class="grid grid-cols-2 gap-6">
          <TextInput
            labelText="Brand"
            placeholder=""
            v-model:value="configData.text.brand"
          />
          <TextInput
            labelText="Type"
            placeholder=""
            v-model:value="configData.text.type"
          />
        </div>

        <hr class="h-px my-8 bg-gray-200 border-0 dark:bg-gray-700" />

        <div class="mt-8 grid grid-cols-2 gap-6">
          <TextInput
            labelText="Diameter (mm)"
            placeholder=""
            v-model:value="configData.text.diameter"
          />
          <TextInput
            labelText="Tolerance (mm)"
            placeholder=""
            v-model:value="configData.text.tolerance"
          />
          <TextInput
            labelText="Length (m)"
            placeholder=""
            v-model:value="configData.text.length"
          />
          <TextInput
            labelText="Temp (ºC)"
            placeholder=""
            v-model:value="configData.text.temp"
          />
          <TextInput
            labelText="Weight"
            placeholder=""
            v-model:value="configData.text.weight"
          />

          <label class="flex flex-col">
            <span>Units</span>
            <div class="flex items-center h-[stretch]">
              <div class="flex items-center me-4">
                <input
                  id="kilograms"
                  type="radio"
                  value="kg"
                  name="units"
                  class="w-4 h-4"
                  v-model="configData.text.units"
                />
                <label for="kilograms" class="ms-2">Kilograms</label>
              </div>
              <div class="flex items-center me-4">
                <input
                  id="grams"
                  type="radio"
                  value="g"
                  name="units"
                  class="w-4 h-4"
                  v-model="configData.text.units"
                />
                <label for="grams" class="ms-2">Grams</label>
              </div>
            </div>
          </label>

          <TextInput
            labelText="Code"
            placeholder=""
            v-model:value="configData.text.code"
          />
          <TextInput
            labelText="Color Name"
            placeholder=""
            v-model:value="configData.text.colour_name"
          />
        </div>

        <hr class="h-px my-8 bg-gray-200 border-0 dark:bg-gray-700" />

        <!-- Input fields for label colour configuration -->
        <div class="mt-8 grid grid-cols-2 gap-6">
          <TextInput
            inputType="color"
            labelText="Label Background"
            placeholder=""
            v-model:value="configData.colour.background"
          />
          <TextInput
            inputType="color"
            labelText="Label Text"
            placeholder=""
            v-model:value="configData.colour.text"
          />
          <TextInput
            inputType="color"
            labelText="Filament"
            placeholder=""
            v-model:value="configData.colour.filament"
          />
          <TextInput
            inputType="color"
            labelText="Spool Outline"
            placeholder=""
            v-model:value="configData.colour.outline"
          />
        </div>
      </div>

      <div class="flex flex-col justify-center items-center gap-6">
        <!-- Label preview section -->
        <Label
          v-model:data="configData"
          height="245"
          class="border-black border-8"
        />

        <button
          @click="downloadLabel"
          class="rounded border border-primary bg-primary px-12 py-3 text-white hover:bg-transparent hover:text-primary focus:outline-none focus:ring active:text-primary"
        >
          Download
        </button>

        <!-- Hidden section for full-sized label used in download -->
        <div class="hidden">
          <canvas id="canvas"></canvas>
          <Label id="full-sized-label" v-model:data="configData" />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Scoped CSS styles (if any) go here */
</style>
