# D3.js Custom Linear Scale Vue Component 📊

This project demonstrates a highly configurable linear scale visualization built with D3.js and integrated as a Vue 3 component within a Vite project. It features a custom non-linear scale distribution, various tick mark types, a dynamic indicator, a confidence range, and a simulation mode.

-----

## Features ✨

* **Custom Piecewise Linear Scale:** The scale's domain is defined by `majorTicks`, with segments proportionally sized based on `weights`.
* **Explicit Tick Mark Control:**
  * `majorTicks`: Numerical values for the main, numbered, and bolded tick marks.
  * `minorTicks`: Numerical values for smaller, unnumbered tick marks.
  * `intermediateTicks`: Numerical values for medium-sized, unnumbered tick marks (e.g., for 0.5 increments).
* **Configurable Major Tick Text Offset:** Controls the distance of numbers from major tick marks.
* **Dynamic Indicator:** A red triangle indicator with configurable size, color, opacity, and distance from the scale.
* **Confidence Range:** An opaque box representing a confidence range, with configurable width, perpendicular dimension, opacity, and color.
* **Orientation Toggle:** Switch between horizontal and vertical scale layouts.
* **Configurable Padding:** Adjusts space at the ends of the scale.
* **Smooth Transitions:** CSS transitions with configurable duration for visually appealing movements.
* **Simulation Mode:** Updates indicator values and confidence ranges randomly at a configurable frequency (1-10 Hz).
* **Responsive Design:** The scale component adapts to its container's size.
* **Vue 3 Component:** Encapsulated as a reusable Vue component.
* **Vite & Tailwind CSS v4:** Modern development setup.

-----

## Project Structure 📁

```text
my-d3-scale-app/
├── public/
│   └── vite.svg
├── src/
│   ├── components/
│   │   └── LinearScale.vue  <-- The D3.js scale Vue component
│   ├── App.vue              <-- Main application component with controls
│   ├── main.js              <-- Vue app entry point
│   └── style.css            <-- Tailwind CSS import and global styles
├── index.html               <-- Main HTML file
├── package.json
├── postcss.config.js        <-- PostCSS configuration for Tailwind
├── README.md                <-- This file
├── tailwind.config.js       <-- Tailwind CSS configuration
└── vite.config.js           <-- Vite configuration
```

-----

## Setup and Installation 🛠️

To get this project up and running locally, follow these steps:

1. **Clone the repository (if applicable) or navigate to your project directory:**

    ```bash
    # If you created the project using `npm create vite@latest`
    cd my-d3-scale-app
    ```

2. **Install project dependencies:**

    ```bash
    npm install
    ```

3. **Install Tailwind CSS and its peer dependencies:**

    ```bash
    npm install -D tailwindcss postcss autoprefixer
    ```

4. **Initialize Tailwind CSS config files:**

    ```bash
    npx tailwindcss init -p
    ```

      * *Troubleshooting:* If you encounter an `npm error: could not determine executable to run`, this is typically a Node.js/npm environment issue. Try `npm cache clean --force`, `npm install -g npm@latest`, or reinstalling Node.js.

5. **Verify/Update Configuration Files:**

    Ensure your `tailwind.config.js`, `postcss.config.js`, `src/style.css`, and `src/main.js` files match the content provided in the Canvas.

      * **Crucially for Tailwind CSS v4:** Make sure `src/style.css` **DOES NOT** contain `@tailwind` directives.

-----

## Running the Application 🚀

Once set up, start the development server:

```bash
npm run dev
```

This will typically open the application in your browser at `http://localhost:5173/`.

-----

## Usage ⚙️

The application provides a custom linear scale and a set of controls:

* **Adjust Indicator Value:** Manually set the indicator's position using the slider.
* **Scale Container Dimensions:** Set the SVG container's width and height in pixels.
* **Major Ticks (comma-separated, sorted):** Input the numerical values for the main, numbered tick marks.
* **Minor Ticks (comma-separated):** Input the numerical values for smaller, unnumbered tick marks.
* **Intermediate Ticks (comma-separated):** Input the numerical values for medium-sized, unnumbered tick marks (e.g., 0.5 increments).
* **Segment Weights (comma-separated, sum to 1):** Proportional lengths for segments between major ticks. The array length must be `(major ticks count - 1)`, and values should sum to approximately `1.0`.
* **Indicator Settings:** Configure the triangle's **Size (px)**, **Color**, **Opacity (0-1)**, and **Distance (% of SVG)** from the scale line.
* **Confidence Box Settings:** Configure the box's **Width (% of scale)**, **Cross Dimension (px)**, **Opacity (0-1)**, and **Color**.
* **Transition Duration (seconds):** Modify the smoothness of animations.
* **Orientation:** Toggle between `horizontal` and `vertical` scale layouts.
* **Scale Padding (pixels):** Add space at the ends of the scale.
* **Major Tick Text Offset (pixels):** Adjust the distance of major tick numbers from their respective lines.
* **Update Scale Button:** Apply changes made to the scale's configuration.
* **Simulation Controls:**
  * **Simulation Frequency (Hz):** Adjust how fast simulated values update.
  * **Start/Stop Simulation Button:** Toggle automatic simulation. Manual controls are disabled when active.

### Vue Component Usage Example

The `LinearScale.vue` component is designed for easy integration into any Vue 3 application.

```vue
<script setup>
import { ref } from 'vue';
import LinearScale from './components/LinearScale.vue';

const myValue = ref(0.5);
const myIndicatorColor = ref('#1e40af'); // A blue color
const myOrientation = ref('vertical');

// Example tick and weight configuration
const myMajorTicks = ref([-15, -10, -5, 0, 5, 10, 15]);
const myMinorTicks = ref([-14, -13, -12, -11, -9, -8, -7, -6, -4, -3, -2, -1, 1, 2, 3, 4, 6, 7, 8, 9, 11, 12, 13, 14]);
const myIntermediateTicks = ref([-12.5, -7.5, -2.5, 2.5, 7.5, 12.5]);
const myWeights = ref([0.1, 0.15, 0.2, 0.2, 0.15, 0.2]); // Must sum to ~1.0 and match segments

const myMajorTickTextOffset = ref(20);
</script>

<template>
  <div style="width: 400px; height: 600px; border: 1px solid #ccc;">
    <LinearScale
      :value="myValue"
      :indicatorColor="myIndicatorColor"
      :orientation="myOrientation"
      :scalePadding="60"
      :indicatorSize="25"
      :confidenceRangePercent="10"
      :confidenceBoxCrossDimension="30"
      :transitionDuration="0.5"
      :majorTicks="myMajorTicks"
      :minorTicks="myMinorTicks"
      :intermediateTicks="myIntermediateTicks"
      :weights="myWeights"
      :majorTickTextOffset="myMajorTickTextOffset"
    />
  </div>
</template>
```

-----

## Deployment to GitHub Pages 🌐

To deploy your Vite project to GitHub Pages, configure Vite and then build and push your project.

### 1\. Configure Vite for GitHub Pages

Edit your `vite.config.js`:

**Example `vite.config.js`:**

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  base: '/my-d3-scale-app/', // Replace 'my-d3-scale-app' with your GitHub repository name
})
```

### 2\. Build the Project

This creates an optimized `dist` folder:

```bash
npm run build
```

### 3\. Deploy to GitHub Pages

#### Method A: Manual Deployment (using `gh-pages` branch)

1. **Install `gh-pages` package:**

    ```bash
    npm install gh-pages --save-dev
    ```

2. **Add a deploy script to `package.json`:**

    ```json
    {
      "name": "my-d3-scale-app",
      "private": true,
      "version": "0.0.0",
      "type": "module",
      "scripts": {
        "dev": "vite",
        "build": "vite build",
        "preview": "vite preview",
        "deploy": "gh-pages -d dist"
      },
      "dependencies": {
        "d3": "^7.9.0",
        "vue": "^3.4.21"
      },
      "devDependencies": {
        "@vitejs/plugin-vue": "^5.0.4",
        "autoprefixer": "^10.4.19",
        "gh-pages": "^6.1.1",
        "postcss": "^8.4.38",
        "tailwindcss": "^4.0.0-next-11",
        "vite": "^5.2.0"
      }
    }
    ```

    *Make sure `gh-pages` is listed under `devDependencies` after installation.*

3. **Run the deploy script:**

    ```bash
    npm run deploy
    ```

    This command will create/update a `gh-pages` branch, commit `dist` contents, and push to your repo.

4. **Configure GitHub Pages in Repository Settings:**

      * Go to your GitHub repository on `github.com`.
      * Navigate to **Settings** \> **Pages**.
      * Under "Build and deployment", select **Deploy from a branch**.
      * Choose `gh-pages` as the branch and `/ (root)` as the folder.
      * Click **Save**.
      * Your site should be live at `https://<your-github-username>.github.io/<your-repository-name>/`. (Might take a few minutes.)

#### Method B: Automated Deployment with GitHub Actions (Recommended)

For continuous deployment, use GitHub Actions.

1. **Create a workflow file:**
    `my-d3-scale-app/.github/workflows/deploy.yml`

    ```text
    .github/
    └── workflows/
        └── deploy.yml
    ```

2. **Add the workflow content to `deploy.yml`:**

    ```yaml
    name: Deploy to GitHub Pages

    on:
      push:
        branches:
          - main # or 'master' if that's your default branch

    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout 🛎️
            uses: actions/checkout@v4

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20' # Use a recent LTS Node.js version

          - name: Install dependencies 📦
            run: npm install

          - name: Build project 🏗️
            run: npm run build

          - name: Deploy to GitHub Pages 🚀
            uses: peaceiris/actions-gh-pages@v4
            with:
              github_token: ${{ secrets.GITHUB_TOKEN }}
              publish_dir: ./dist
              # For project pages (https://<username>.github.io/<repo-name>/),
              # the default branch is usually 'gh-pages'.
              # For user/organization pages (https://<username>.github.io/),
              # the default branch is usually 'main' or 'master'.
              publish_branch: gh-pages # This is the branch where your built site will be pushed
    ```

3. **Commit and Push:**

    ```bash
    git add .github/workflows/deploy.yml
    git commit -m "feat: Add GitHub Pages deployment workflow"
    git push origin main # or master
    ```

4. **Configure GitHub Pages in Repository Settings:**

      * Go to your GitHub repository on `github.com`.
      * Navigate to **Settings** \> **Pages**.
      * Under "Build and deployment", select **GitHub Actions**.
      * GitHub will automatically detect the workflow.
      * Your site should be live at `https://<your-github-username>.github.io/<your-repository-name>/` after the workflow completes. Monitor progress in the "Actions" tab.

-----
