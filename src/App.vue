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

        <hr class="h-px my-8 bg-gray-200 border-0 dark:bg-gray-700" />
        <label class="flex flex-col">
          <span class="mb-auto">Logo (SVG)</span>
          <input
            type="file"
            name="logo"
            accept="image/svg+xml"
            class="mt-1 block w-full"
            @change="handleFileChange"
          />
        </label>
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
let configData = reactive<LabelDataType>({
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
  logo: '<path d="M19.673 2.804V1.543h3.251c.451.007.883.098 1.262.261a2.192 2.192 0 0 1 .968 3.373l-.026.036c.459.384.752.963.752 1.61a2.093 2.093 0 0 1-2.059 2.095l-.033.002-.033-.002H18.07V3.421l1.603-.617Zm0 0V4.55h3.373a.856.856 0 0 0 .365-.081.798.798 0 0 0 .486-.61.874.874 0 0 0-.532-.994.726.726 0 0 0-.286-.061h-3.406Zm3.629 4.884a.94.94 0 0 0 .568-1.69.932.932 0 0 0-.568-.193h-3.629v1.883h3.629Zm7.43 4.644c.862 0 1.561.695 1.561 1.56l-.001 4.107h-6.033a1.564 1.564 0 0 1-1.562-1.566v-.44c0-.864.699-1.566 1.562-1.566h4.569l.002-.719a.157.157 0 0 0-.156-.157h-5.12l-.459-1.219h5.637Zm.096 4.435v-1.116h-4.512a.156.156 0 0 0-.156.156v.803c0 .087.07.157.156.157h4.512Zm8.161-4.435c.862 0 1.562.701 1.562 1.565v2.536c0 .865-.7 1.566-1.562 1.566h-5.282v-7.081h1.603v1.414h3.679Zm-.011 4.28v-2.88a.156.156 0 0 0-.156-.156H35.31v3.192h3.512c.087 0 .156-.07.156-.156ZM44.009 3.25c.863 0 1.562.701 1.562 1.565v4.102h-1.568V4.65a.156.156 0 0 0-.156-.156h-2.078a.156.156 0 0 0-.156.156v4.267h-1.565V4.65a.156.156 0 0 0-.157-.156h-2.107a.156.156 0 0 0-.156.156v4.267H36.07V3.25h7.939Zm16.525 0h1.464v4.101c0 .865-.699 1.566-1.561 1.566h-3.878a1.564 1.564 0 0 1-1.561-1.566V3.25h1.475v4.28c0 .086.07.156.156.156h3.75c.086 0 .155-.07.155-.156V3.25ZM28.622 8.917a1.564 1.564 0 0 1-1.561-1.566v-.44c0-.865.699-1.565 1.561-1.565h4.57l.002-.72a.156.156 0 0 0-.157-.156h-5.119l-.46-1.22h5.637a1.56 1.56 0 0 1 1.562 1.56l-.002 4.107h-6.033Zm-.098-2.192v.804c0 .086.07.156.156.156h4.512V6.569H28.68a.156.156 0 0 0-.156.156Zm23.602-3.479c.862 0 1.561.701 1.561 1.566v2.536c0 .864-.699 1.565-1.561 1.565h-5.282V1.832h1.603v1.414h3.679Zm-.011 4.28v-2.88a.156.156 0 0 0-.156-.156h-3.512v3.192h3.512c.086 0 .156-.07.156-.156ZM7.395 17.999V6.719l6.4 2.527v8.753h-6.4ZM7.395 0h6.4v8.169l-6.4-2.529V0ZM0 0h6.4l.001 8.753L0 11.28V0Zm0 17.997v-5.64l6.401-2.529-.001 8.169H0ZM18.07.629l1.603-.617V1.54l-1.603.618V.629ZM17.998 11h1.639v5.672h3.848L23.998 18h-6v-7Z" />',
});

// Function to handle file input change
const handleFileChange = async (event: Event) => {
  const target = event.target as HTMLInputElement;
  if (target.files && target.files[0]) {
    const file = target.files[0];

    // Read the file as text (SVG markup)
    const reader = new FileReader();
    reader.onload = (e) => {
      if (e.target?.result) {
        let svgContent = e.target.result as string;

        // Remove the outer <svg> tags
        const svgStartTag = svgContent.indexOf("<svg");
        if (svgStartTag !== -1) {
          // Find the closing tag of the <svg> element
          const svgEndTag = svgContent.indexOf(">", svgStartTag);
          if (svgEndTag !== -1) {
            // Remove the opening <svg> tag
            svgContent = svgContent.slice(svgEndTag + 1);

            // Remove the closing </svg> tag
            const svgCloseTag = svgContent.lastIndexOf("</svg>");
            if (svgCloseTag !== -1) {
              svgContent = svgContent.substring(0, svgCloseTag);
            }
          }
        }

        // Remove any `fill` attribute`
        svgContent = svgContent.replace(/fill="[^"]*"/g, "");
        // Remove any `style fill` attribute`
        svgContent = svgContent.replace(/fill:[^"]*;/g, "");

        // Set the modified SVG content as the logo
        configData.logo = svgContent;
      }
    };
    reader.readAsText(file);
  }
};

// TODO: Implement method to adjust the logo scale

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
  console.log(configData.logo);
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

<style scoped>
/* Scoped CSS styles (if any) go here */
</style>
